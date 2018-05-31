# IO 

> FileInputStream.c

<pre><code>
JNIEXPORT jint JNICALL
Java_java_io_FileInputStream_read(JNIEnv *env, jobject this) {
    return readSingle(env, this, fis_fd);
}

JNIEXPORT jint JNICALL
Java_java_io_FileInputStream_readBytes(JNIEnv *env, jobject this,
        jbyteArray bytes, jint off, jint len) {
    return readBytes(env, this, bytes, off, len, fis_fd);
}
</code></pre>

> FileOutputStream_md.c

<pre><code>
JNIEXPORT void JNICALL
Java_java_io_FileOutputStream_write(JNIEnv *env, jobject this, jint byte, jboolean append) {
    writeSingle(env, this, byte, append, fos_fd);
}

JNIEXPORT void JNICALL
Java_java_io_FileOutputStream_writeBytes(JNIEnv *env,
    jobject this, jbyteArray bytes, jint off, jint len, jboolean append)
{
    writeBytes(env, this, bytes, off, len, append, fos_fd);
}
</code></pre>

> io_util.c

<pre><code>
jint
readSingle(JNIEnv *env, jobject this, jfieldID fid) {
    jint nread;
    char ret;
    FD fd = GET_FD(this, fid);
    if (fd == -1) {
        JNU_ThrowIOException(env, "Stream Closed");
        return -1;
    }
    nread = IO_Read(fd, &ret, 1);
    if (nread == 0) { /* EOF */
        return -1;
    } else if (nread == -1) { /* error */
        JNU_ThrowIOExceptionWithLastError(env, "Read error");
    }
    return ret & 0xFF;
}

jint
readBytes(JNIEnv *env, jobject this, jbyteArray bytes,
          jint off, jint len, jfieldID fid)
{
    jint nread;
    char stackBuf[BUF_SIZE];
    char *buf = NULL;
    FD fd;

    if (IS_NULL(bytes)) {
        JNU_ThrowNullPointerException(env, NULL);
        return -1;
    }

    if (outOfBounds(env, off, len, bytes)) {
        JNU_ThrowByName(env, "java/lang/IndexOutOfBoundsException", NULL);
        return -1;
    }

    if (len == 0) {
        return 0;
    } else if (len > BUF_SIZE) {
        buf = malloc(len);
        if (buf == NULL) {
            JNU_ThrowOutOfMemoryError(env, NULL);
            return 0;
        }
    } else {
        buf = stackBuf;
    }

    fd = GET_FD(this, fid);
    if (fd == -1) {
        JNU_ThrowIOException(env, "Stream Closed");
        nread = -1;
    } else {
        nread = IO_Read(fd, buf, len);
        if (nread > 0) {
            (*env)->SetByteArrayRegion(env, bytes, off, nread, (jbyte *)buf);
        } else if (nread == -1) {
            JNU_ThrowIOExceptionWithLastError(env, "Read error");
        } else { /* EOF */
            nread = -1;
        }
    }

    if (buf != stackBuf) {
        free(buf);
    }
    return nread;
}

void
writeSingle(JNIEnv *env, jobject this, jint byte, jboolean append, jfieldID fid) {
    // Discard the 24 high-order bits of byte. See OutputStream#write(int)
    char c = (char) byte;
    jint n;
    FD fd = GET_FD(this, fid);
    if (fd == -1) {
        JNU_ThrowIOException(env, "Stream Closed");
        return;
    }
    if (append == JNI_TRUE) {
        n = IO_Append(fd, &c, 1);
    } else {
        n = IO_Write(fd, &c, 1);
    }
    if (n == -1) {
        JNU_ThrowIOExceptionWithLastError(env, "Write error");
    }
}

void
writeBytes(JNIEnv *env, jobject this, jbyteArray bytes,
           jint off, jint len, jboolean append, jfieldID fid)
{
    jint n;
    char stackBuf[BUF_SIZE];
    char *buf = NULL;
    FD fd;

    if (IS_NULL(bytes)) {
        JNU_ThrowNullPointerException(env, NULL);
        return;
    }

    if (outOfBounds(env, off, len, bytes)) {
        JNU_ThrowByName(env, "java/lang/IndexOutOfBoundsException", NULL);
        return;
    }

    if (len == 0) {
        return;
    } else if (len > BUF_SIZE) {
        buf = malloc(len);
        if (buf == NULL) {
            JNU_ThrowOutOfMemoryError(env, NULL);
            return;
        }
    } else {
        buf = stackBuf;
    }

    (*env)->GetByteArrayRegion(env, bytes, off, len, (jbyte *)buf);

    if (!(*env)->ExceptionOccurred(env)) {
        off = 0;
        while (len > 0) {
            fd = GET_FD(this, fid);
            if (fd == -1) {
                JNU_ThrowIOException(env, "Stream Closed");
                break;
            }
            if (append == JNI_TRUE) {
                n = IO_Append(fd, buf+off, len);
            } else {
                n = IO_Write(fd, buf+off, len);
            }
            if (n == -1) {
                JNU_ThrowIOExceptionWithLastError(env, "Write error");
                break;
            }
            off += n;
            len -= n;
        }
    }
    if (buf != stackBuf) {
        free(buf);
    }
}
</code></pre>

#### `#define IO_Read handleRead`
#### `#define IO_Write handleWrite`
#### `#define IO_Append handleAppend`

> io_util_md.c

## Windows:

<pre><code>
JNIEXPORT
jint
handleRead(FD fd, void *buf, jint len)
{
    DWORD read = 0;
    BOOL result = 0;
    HANDLE h = (HANDLE)fd;
    if (h == INVALID_HANDLE_VALUE) {
        return -1;
    }
    result = ReadFile(h,          /* File handle to read */
                      buf,        /* address to put data */
                      len,        /* number of bytes to read */
                      &read,      /* number of bytes read */
                      NULL);      /* no overlapped struct */
    if (result == 0) {
        int error = GetLastError();
        if (error == ERROR_BROKEN_PIPE) {
            return 0; /* EOF */
        }
        return -1;
    }
    return (jint)read;
}

static jint writeInternal(FD fd, const void *buf, jint len, jboolean append)
{
    BOOL result = 0;
    DWORD written = 0;
    HANDLE h = (HANDLE)fd;
    if (h != INVALID_HANDLE_VALUE) {
        OVERLAPPED ov;
        LPOVERLAPPED lpOv;
        if (append == JNI_TRUE) {
            ov.Offset = (DWORD)0xFFFFFFFF;
            ov.OffsetHigh = (DWORD)0xFFFFFFFF;
            ov.hEvent = NULL;
            lpOv = &ov;
        } else {
            lpOv = NULL;
        }
        result = WriteFile(h,                /* File handle to write */
                           buf,              /* pointers to the buffers */
                           len,              /* number of bytes to write */
                           &written,         /* receives number of bytes written */
                           lpOv);            /* overlapped struct */
    }
    if ((h == INVALID_HANDLE_VALUE) || (result == 0)) {
        return -1;
    }
    return (jint)written;
}
</code></pre>

#Linux:
<pre><code>
ssize_t
handleRead(FD fd, void *buf, jint len)
{
    ssize_t result;
    RESTARTABLE(read(fd, buf, len), result);
    return result;
}

ssize_t
handleWrite(FD fd, const void *buf, jint len)
{
    ssize_t result;
    RESTARTABLE(write(fd, buf, len), result);
    return result;
}
</code></pre>



# NIO

两种方式读取文件内容，且两种都是以DirectByteBuffer来获取内容

* FileChannel.read -> IOUtil.read -> FileDispatcher.read
* Filechannel.map -> DirectByteBuffer.get -> Unsafe.getByte

> FileChannelImpl.java

<pre><code>
public int read(ByteBuffer arg0) throws IOException {
   ......
   if(this.isOpen()) {
      do {
         arg2 = IOUtil.read(this.fd, arg0, -1L, this.nd);
      } while(arg2 == -3 && this.isOpen());

      int arg11 = IOStatus.normalize(arg2);
      return arg11;
   }
   ......

}
</code></pre>

> IOUtil.java

<pre><code>
static int read(FileDescriptor arg, ByteBuffer arg0, long arg1, NativeDispatcher arg3) throws IOException {
  if(arg0.isReadOnly()) {
     throw new IllegalArgumentException("Read-only buffer");
  } else if(arg0 instanceof DirectBuffer) {
     return readIntoNativeBuffer(arg, arg0, arg1, arg3);
  } else {
     ByteBuffer arg4 = Util.getTemporaryDirectBuffer(arg0.remaining());
     int arg6;
     try {
        int arg5 = readIntoNativeBuffer(arg, arg4, arg1, arg3);
        arg4.flip();
        if(arg5 > 0) {
           arg0.put(arg4);
        }
        arg6 = arg5;
     } finally {
        Util.offerFirstTemporaryDirectBuffer(arg4);
     }
     return arg6;
  }
}

private static int readIntoNativeBuffer(FileDescriptor arg, ByteBuffer arg0, long arg1, NativeDispatcher arg3) throws IOException {
    int arg4 = arg0.position();
    int arg5 = arg0.limit();
    assert arg4 <= arg5;
    int arg6 = arg4 <= arg5?arg5 - arg4:0;
    if(arg6 == 0) {
        return 0;
    } else {
        boolean arg7 = false;
        int arg8;
        if(arg1 != -1L) {
            arg8 = arg3.pread(arg, ((DirectBuffer)arg0).address() + (long)arg4, arg6, arg1);
        } else {
            arg8 = arg3.read(arg, ((DirectBuffer)arg0).address() + (long)arg4, arg6);
        }
        if(arg8 > 0) {
            arg0.position(arg4 + arg8);
        }
        return arg8;
    }
}
</code></pre>

> DirectByteBuffer.java

<pre><code>
public byte get() {
    return ((unsafe.getByte(ix(nextGetIndex()))));
}
</code></pre>


> FileDispatcherImpl.c

## Windows

<pre><code>
JNIEXPORT jint JNICALL
Java_sun_nio_ch_FileDispatcherImpl_read0(JNIEnv *env, jclass clazz, jobject fdo,
                                      jlong address, jint len)
{
    DWORD read = 0;
    BOOL result = 0;
    HANDLE h = (HANDLE)(handleval(env, fdo));

    if (h == INVALID_HANDLE_VALUE) {
        JNU_ThrowIOExceptionWithLastError(env, "Invalid handle");
        return IOS_THROWN;
    }
    result = ReadFile(h,          /* File handle to read */
                      (LPVOID)address,    /* address to put data */
                      len,        /* number of bytes to read */
                      &read,      /* number of bytes read */
                      NULL);      /* no overlapped struct */
    if (result == 0) {
        int error = GetLastError();
        if (error == ERROR_BROKEN_PIPE) {
            return IOS_EOF;
        }
        if (error == ERROR_NO_DATA) {
            return IOS_UNAVAILABLE;
        }
        JNU_ThrowIOExceptionWithLastError(env, "Read failed");
        return IOS_THROWN;
    }
    return convertReturnVal(env, (jint)read, JNI_TRUE);
}

JNIEXPORT jint JNICALL
Java_sun_nio_ch_FileDispatcherImpl_write0(JNIEnv *env, jclass clazz, jobject fdo,
                                          jlong address, jint len, jboolean append)
{
    BOOL result = 0;
    DWORD written = 0;
    HANDLE h = (HANDLE)(handleval(env, fdo));
    if (h != INVALID_HANDLE_VALUE) {
        OVERLAPPED ov;
        LPOVERLAPPED lpOv;
        if (append == JNI_TRUE) {
            ov.Offset = (DWORD)0xFFFFFFFF;
            ov.OffsetHigh = (DWORD)0xFFFFFFFF;
            ov.hEvent = NULL;
            lpOv = &ov;
        } else {
            lpOv = NULL;
        }
        result = WriteFile(h,           /* File handle to write */
                      (LPCVOID)address, /* pointers to the buffers */
                      len,              /* number of bytes to write */
                      &written,         /* receives number of bytes written */
                      lpOv);            /* overlapped struct */
    }
    if ((h == INVALID_HANDLE_VALUE) || (result == 0)) {
        JNU_ThrowIOExceptionWithLastError(env, "Write failed");
    }
    return convertReturnVal(env, (jint)written, JNI_FALSE);
}
</code></pre>

## Linux

<pre><code>
JNIEXPORT jint JNICALL
Java_sun_nio_ch_FileDispatcherImpl_read0(JNIEnv *env, jclass clazz,
                             jobject fdo, jlong address, jint len)
{
    jint fd = fdval(env, fdo);
    void *buf = (void *)jlong_to_ptr(address);

    return convertReturnVal(env, read(fd, buf, len), JNI_TRUE);
}

JNIEXPORT jint JNICALL
Java_sun_nio_ch_FileDispatcherImpl_write0(JNIEnv *env, jclass clazz,
                              jobject fdo, jlong address, jint len)
{
    jint fd = fdval(env, fdo);
    void *buf = (void *)jlong_to_ptr(address);

    return convertReturnVal(env, write(fd, buf, len), JNI_FALSE);
}
</code></pre>

