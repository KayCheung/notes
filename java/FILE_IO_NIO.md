# IO 

> FileInputStream.c

<pre><code>
<font color="read">//<i>单字节读取</i></font>
JNIEXPORT jint JNICALL
Java_java_io_FileInputStream_read(JNIEnv *env, jobject this) {
    return readSingle(env, this, fis_fd);
}

<font color="read">//<i>读取字节数组</i></font>
JNIEXPORT jint JNICALL
Java_java_io_FileInputStream_readBytes(JNIEnv *env, jobject this,
        jbyteArray bytes, jint off, jint len) {
    return readBytes(env, this, bytes, off, len, fis_fd);
}
</code></pre>

> FileOutputStream_md.c

<font color="read">//<i>写单字节</i></font>
<pre><code>
JNIEXPORT void JNICALL
Java_java_io_FileOutputStream_write(JNIEnv *env, jobject this, jint byte, jboolean append) {
    writeSingle(env, this, byte, append, fos_fd);
}

<font color="read">//<i>写字节数组</i></font>
JNIEXPORT void JNICALL
Java_java_io_FileOutputStream_writeBytes(JNIEnv *env,
    jobject this, jbyteArray bytes, jint off, jint len, jboolean append)
{
    writeBytes(env, this, bytes, off, len, append, fos_fd);
}
</code></pre>

> io_util.c

<font color="read">//<i>单字节读取</i></font>
<pre><code>
jint
readSingle(JNIEnv *env, jobject this, jfieldID fid) {
    jint nread;
    char ret; <font color="read">//<i>定义一个变量,接收读取到的数据</i></font>
    FD fd = GET_FD(this, fid); <font color="read">//<i>获取文件句柄</i></font>
    if (fd == -1) {
        JNU_ThrowIOException(env, "Stream Closed");
        return -1;
    }
    nread = IO_Read(fd, &ret, 1);<font color="read">//<i>调用系统接口</i></font>
    if (nread == 0) { /* EOF */
        return -1;
    } else if (nread == -1) { /* error */
        JNU_ThrowIOExceptionWithLastError(env, "Read error");
    }
    return ret & 0xFF;<font color="read">//<i>因为char是1个字节8位，所以转int,高24位取0, 后8位保持原样</i></font>
}

<font color="read">//<i>读取字节数组</i></font>

<font color="read">//<i>Block IO默认非单字节读写为8KB, 操作系统默认的数据块大小一般在4KB</i></font>
<code>#define BUF_SIZE 8192</code>

jint
readBytes(JNIEnv *env, jobject this, jbyteArray bytes,
          jint off, jint len, jfieldID fid)
{
    jint nread;
    char stackBuf[BUF_SIZE];<font color="read">//<i>定义一个固定长度的缓冲区,栈上分配</i></font>
    char *buf = NULL;<font color="read">//<i>定义空的缓冲区,待分配空间</i></font>
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
    } else if (len > BUF_SIZE) {<font color="read">//<i>如果读取的内容大于默认的BUF_SIZE,则在内存中开辟一块len长度的空间</i></font>
        buf = malloc(len);<font color="read">//<i>在内存中开辟一块len长度的空间,内存空间不够返回NULL,最后需要配合free进行内存回收</i></font>
        if (buf == NULL) {
            JNU_ThrowOutOfMemoryError(env, NULL);
            return 0;
        }
    } else {
        buf = stackBuf;<font color="read">//<i>如果len小于默认的缓冲大小,则直接使用默认的缓冲区</i></font>
    }
    fd = GET_FD(this, fid);<font color="read">//<i>获取文件句柄</i></font>
    if (fd == -1) {
        JNU_ThrowIOException(env, "Stream Closed");
        nread = -1;
    } else {
        nread = IO_Read(fd, buf, len);<font color="read">//<i>调用系统接口</i></font>
        if (nread > 0) {
            (*env)->SetByteArrayRegion(env, bytes, off, nread, (jbyte *)buf);<font color="read">//<i>将buff中的数据拷贝到bytes中</i></font>
        } else if (nread == -1) {
            JNU_ThrowIOExceptionWithLastError(env, "Read error");
        } else { /* EOF */
            nread = -1;
        }
    }
    if (buf != stackBuf) {
        free(buf);<font color="read">//<i>释放buf</i></font>
    }
    return nread;
}

<font color="read">//<i>写单字节</i></font>
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

<font color="read">//<i>写字节数组</i></font>
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

---
 
# NIO

两种方式读取文件内容，且两种都是以DirectByteBuffer来获取内容

* FileChannel.read -> IOUtil.read -> FileDispatcher.read
* Filechannel.map -> DirectByteBuffer.get -> Unsafe.getByte

> FileChannelImpl.java

<pre><code>
public int read(ByteBuffer dst) throws IOException {
    ......
    synchronized (positionLock) {
        .......
        if (!isOpen())
            return 0;
        do {
            n = IOUtil.read(fd, dst, -1, nd);
        } while ((n == IOStatus.INTERRUPTED) && isOpen());
        return IOStatus.normalize(n);
        ......
    }
    ......
}

public int write(ByteBuffer src) throws IOException {
    ......
    synchronized (positionLock) {
        ......
        if (!isOpen())
            return 0;
        do {
            n = IOUtil.write(fd, src, -1, nd);
        } while ((n == IOStatus.INTERRUPTED) && isOpen());
        return IOStatus.normalize(n);
        ......
    }
}

public MappedByteBuffer map(MapMode mode, long position, long size)
        throws IOException
{
    ......
	do {
		rv = nd.truncate(fd, position + size);
	} while ((rv == IOStatus.INTERRUPTED) && isOpen());
    ......
	......
	try {
		// If no exception was thrown from map0, the address is valid
		addr = map0(imode, mapPosition, mapSize);
	} catch (OutOfMemoryError x) {
		// An OutOfMemoryError may indicate that we've exhausted memory
		// so force gc and re-attempt map
		System.gc();
		try {
			Thread.sleep(100);
		} catch (InterruptedException y) {
			Thread.currentThread().interrupt();
		}
		try {
			addr = map0(imode, mapPosition, mapSize);
		} catch (OutOfMemoryError y) {
			// After a second OOME, fail
			throw new IOException("Map failed", y);
		}
	}
	......
	......
	Unmapper um = new Unmapper(addr, mapSize, isize, mfd);
	if ((!writable) || (imode == MAP_RO)) {
		return Util.newMappedByteBufferR(isize,
										 addr + pagePosition,
										 mfd,
										 um);
	} else {
		return Util.newMappedByteBuffer(isize,
										addr + pagePosition,
										mfd,
										um);
	}
	......
}

private static class Unmapper
        implements Runnable
{
    ......
	public void run() {
		if (address == 0)
			return;
		unmap0(address, size);
		address = 0;
		// if this mapping has a valid file descriptor then we close it
		if (fd.valid()) {
			try {
				nd.close(fd);
			} catch (IOException ignore) {
				// nothing we can do
			}
		}
		synchronized (Unmapper.class) {
			count--;
			totalSize -= size;
			totalCapacity -= cap;
		}
	}
}
</code></pre>

> IOUtil.java

<pre><code>
static int read(FileDescriptor fd, ByteBuffer dst, long position,  NativeDispatcher nd) throws IOException
{
    if (dst.isReadOnly())
        throw new IllegalArgumentException("Read-only buffer");
    if (dst instanceof DirectBuffer)
        return readIntoNativeBuffer(fd, dst, position, nd);
    // Substitute a native buffer
    ByteBuffer bb = Util.getTemporaryDirectBuffer(dst.remaining());
    try {
        int n = readIntoNativeBuffer(fd, bb, position, nd);
        bb.flip();
        if (n > 0)
            dst.put(bb);
        return n;
    } finally {
        Util.offerFirstTemporaryDirectBuffer(bb);
    }
}

private static int readIntoNativeBuffer(FileDescriptor fd, ByteBuffer bb, long position, NativeDispatcher nd) throws IOException
{
    int pos = bb.position();
    int lim = bb.limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);
    if (rem == 0)
        return 0;
    int n = 0;
    if (position != -1) {
        n = nd.pread(fd, ((DirectBuffer)bb).address() + pos,
                     rem, position);
    } else {
        n = nd.read(fd, ((DirectBuffer)bb).address() + pos, rem);
    }
    if (n > 0)
        bb.position(pos + n);
    return n;
}

static int write(FileDescriptor fd, ByteBuffer src, long position, NativeDispatcher nd) throws IOException
{
    if (src instanceof DirectBuffer)
        return writeFromNativeBuffer(fd, src, position, nd);
    // Substitute a native buffer
    int pos = src.position();
    int lim = src.limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);
    ByteBuffer bb = Util.getTemporaryDirectBuffer(rem);
    try {
        bb.put(src);
        bb.flip();
        // Do not update src until we see how many bytes were written
        src.position(pos);
        int n = writeFromNativeBuffer(fd, bb, position, nd);
        if (n > 0) {
            // now update src
            src.position(pos + n);
        }
        return n;
    } finally {
        Util.offerFirstTemporaryDirectBuffer(bb);
    }
}

private static int writeFromNativeBuffer(FileDescriptor fd, ByteBuffer bb, long position, NativeDispatcher nd) throws IOException
{
    int pos = bb.position();
    int lim = bb.limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);
    int written = 0;
    if (rem == 0)
        return 0;
    if (position != -1) {
        written = nd.pwrite(fd,
                            ((DirectBuffer)bb).address() + pos,
                            rem, position);
    } else {
        written = nd.write(fd, ((DirectBuffer)bb).address() + pos, rem);
    }
    if (written > 0)
        bb.position(pos + written);
    return written;
}
</code></pre>

> DirectByteBuffer.java

<pre><code>
public byte get() {
    return ((unsafe.getByte(ix(nextGetIndex()))));
}

public ByteBuffer put(byte x) {
    unsafe.putByte(ix(nextPutIndex()), ((x)));
    return this;
}

private static class Deallocator
        implements Runnable
    {
    public void run() {
        if (address == 0) {
            // Paranoia
            return;
        }
        unsafe.freeMemory(address);
        address = 0;
        Bits.unreserveMemory(size, capacity);
    }
}
</code></pre>

> unsafe.cpp

<pre><code>
inline void* addr_from_java(jlong addr) {
  // This assert fails in a variety of ways on 32-bit systems.
  // It is impossible to predict whether native code that converts
  // pointers to longs will sign-extend or zero-extend the addresses.
  //assert(addr == (uintptr_t)addr, "must not be odd high bits");
  return (void*)(uintptr_t)addr;
}

inline jlong addr_to_java(void* p) {
  assert(p == (void*)(uintptr_t)p, "must not be odd high bits");
  return (uintptr_t)p;
}

#define DEFINE_GETSETNATIVE(java_type, Type, native_type) \
 \
UNSAFE_ENTRY(java_type, Unsafe_GetNative##Type(JNIEnv *env, jobject unsafe, jlong addr)) \
  UnsafeWrapper("Unsafe_GetNative"#Type); \
  void* p = addr_from_java(addr); \
  JavaThread* t = JavaThread::current(); \
  t->set_doing_unsafe_access(true); \
  java_type x = *(volatile native_type*)p; \
  t->set_doing_unsafe_access(false); \
  return x; \
UNSAFE_END \
 \
UNSAFE_ENTRY(void, Unsafe_SetNative##Type(JNIEnv *env, jobject unsafe, jlong addr, java_type x)) \
  UnsafeWrapper("Unsafe_SetNative"#Type); \
  JavaThread* t = JavaThread::current(); \
  t->set_doing_unsafe_access(true); \
  void* p = addr_from_java(addr); \
  *(volatile native_type*)p = x; \
  t->set_doing_unsafe_access(false); \
UNSAFE_END \
 \
// END DEFINE_GETSETNATIVE.

DEFINE_GETSETNATIVE(jbyte, Byte, signed char)
DEFINE_GETSETNATIVE(jshort, Short, signed short);
DEFINE_GETSETNATIVE(jchar, Char, unsigned short);
DEFINE_GETSETNATIVE(jint, Int, jint);
// no long -- handled specially
DEFINE_GETSETNATIVE(jfloat, Float, float);
DEFINE_GETSETNATIVE(jdouble, Double, double);

#undef DEFINE_GETSETNATIVE
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

> FileChannelImpl.c

## windows
<pre>code>
JNIEXPORT jlong JNICALL
Java_sun_nio_ch_FileChannelImpl_map0(JNIEnv *env, jobject this,
                               jint prot, jlong off, jlong len)
{
    void *mapAddress = 0;
    jint lowOffset = (jint)off;
    jint highOffset = (jint)(off >> 32);
    jlong maxSize = off + len;
    jint lowLen = (jint)(maxSize);
    jint highLen = (jint)(maxSize >> 32);
    jobject fdo = (*env)->GetObjectField(env, this, chan_fd);
    HANDLE fileHandle = (HANDLE)(handleval(env, fdo));
    HANDLE mapping;
    DWORD mapAccess = FILE_MAP_READ;
    DWORD fileProtect = PAGE_READONLY;
    DWORD mapError;
    BOOL result;
    if (prot == sun_nio_ch_FileChannelImpl_MAP_RO) {
        fileProtect = PAGE_READONLY;
        mapAccess = FILE_MAP_READ;
    } else if (prot == sun_nio_ch_FileChannelImpl_MAP_RW) {
        fileProtect = PAGE_READWRITE;
        mapAccess = FILE_MAP_WRITE;
    } else if (prot == sun_nio_ch_FileChannelImpl_MAP_PV) {
        fileProtect = PAGE_WRITECOPY;
        mapAccess = FILE_MAP_COPY;
    }
    mapping = CreateFileMapping(
        fileHandle,      /* Handle of file */
        NULL,            /* Not inheritable */
        fileProtect,     /* Read and write */
        highLen,         /* High word of max size */
        lowLen,          /* Low word of max size */
        NULL);           /* No name for object */
    if (mapping == NULL) {
        JNU_ThrowIOExceptionWithLastError(env, "Map failed");
        return IOS_THROWN;
    }
    mapAddress = MapViewOfFile(
        mapping,             /* Handle of file mapping object */
        mapAccess,           /* Read and write access */
        highOffset,          /* High word of offset */
        lowOffset,           /* Low word of offset */
        (DWORD)len);         /* Number of bytes to map */
    mapError = GetLastError();
    result = CloseHandle(mapping);
    if (result == 0) {
        JNU_ThrowIOExceptionWithLastError(env, "Map failed");
        return IOS_THROWN;
    }
    if (mapAddress == NULL) {
        if (mapError == ERROR_NOT_ENOUGH_MEMORY)
            JNU_ThrowOutOfMemoryError(env, "Map failed");
        else
            JNU_ThrowIOExceptionWithLastError(env, "Map failed");
        return IOS_THROWN;
    }
    return ptr_to_jlong(mapAddress);
}

JNIEXPORT jint JNICALL
Java_sun_nio_ch_FileChannelImpl_unmap0(JNIEnv *env, jobject this,
                                 jlong address, jlong len)
{
    BOOL result;
    void *a = (void *) jlong_to_ptr(address);
    result = UnmapViewOfFile(a);
    if (result == 0) {
        JNU_ThrowIOExceptionWithLastError(env, "Unmap failed");
        return IOS_THROWN;
    }
    return 0;
}
</code></pre>


## Linux
<pre><code>
JNIEXPORT jlong JNICALL
Java_sun_nio_ch_FileChannelImpl_map0(JNIEnv *env, jobject this,
                                     jint prot, jlong off, jlong len)
{
    void *mapAddress = 0;
    jobject fdo = (*env)->GetObjectField(env, this, chan_fd);
    jint fd = fdval(env, fdo);
    int protections = 0;
    int flags = 0;
    if (prot == sun_nio_ch_FileChannelImpl_MAP_RO) {
        protections = PROT_READ;
        flags = MAP_SHARED;
    } else if (prot == sun_nio_ch_FileChannelImpl_MAP_RW) {
        protections = PROT_WRITE | PROT_READ;
        flags = MAP_SHARED;
    } else if (prot == sun_nio_ch_FileChannelImpl_MAP_PV) {
        protections =  PROT_WRITE | PROT_READ;
        flags = MAP_PRIVATE;
    }
    mapAddress = mmap64(
        0,                    /* Let OS decide location */
        len,                  /* Number of bytes to map */
        protections,          /* File permissions */
        flags,                /* Changes are shared */
        fd,                   /* File descriptor of mapped file */
        off);                 /* Offset into file */
    if (mapAddress == MAP_FAILED) {
        if (errno == ENOMEM) {
            JNU_ThrowOutOfMemoryError(env, "Map failed");
            return IOS_THROWN;
        }
        return handle(env, -1, "Map failed");
    }
    return ((jlong) (unsigned long) mapAddress);
}


JNIEXPORT jint JNICALL
Java_sun_nio_ch_FileChannelImpl_unmap0(JNIEnv *env, jobject this,
                                       jlong address, jlong len)
{
    void *a = (void *)jlong_to_ptr(address);
    return handle(env,
                  munmap(a, (size_t)len),
                  "Unmap failed");
}
</code></pre>

