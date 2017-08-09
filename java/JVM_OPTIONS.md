# JVM Options

> http://jvm-options.tech.xebia.fr/
> http://www.oracle.com/technetwork/java/tuning-139912.html#section4.2.5

#### VM Options参数定义
	1. 以 ‘-’ 开头的是标准VM选项，VM规范的选项
	2. 以 ‘-X’ 开头的都是非标准的（这些参数并不能保证在所有的JVM上都被实现），而且如果在新版本有什么改动也不会发布通知。
	3. 以 ‘-XX’ 开头的都是不稳定的并且不推荐在生产环境中使用。这些参数的改动也不会发布通知。
	4. Bool 型参数选项：-XX:+ 打开， -XX:- 关闭。（比如-XX:+PrintGCDetails）
	5. 数字型参数选项通过-XX:=设定。数字可以是 m/M(兆字节)，k/K(千字节)，g/G(G字节)。比如：32K表示32768字节。（比如-XX:MaxPermSize=64m）
	6. String参数选项通过-XX:=设定，通常用来指定一个文件，路径，或者一个命令列表。（比如-XX:HeapDumpPath=./java_pid.hprof）
	7. 命令 java -help可以列出java 应用启动时标准选项（见附录标准VM参数表，不同的JVM实现是不同的）。java -X可以列出不标准的参数（这是JVM的扩展特性）。-X相关的选项不是标准的，被改变也不会通知。如果你想查看当前应用使用的JVM参数，你可以使用：ManagementFactory.getRuntimeMXBean().getInputArguments()。

#### VM Options 参数详细介绍

参数|默认值|描述
----|------|----
|1. 类|
-Xbootclasspath|						|指定加载不需要校验的类(跳过必要的类加载前的校验，能够减少加载时间，但是不安全)
-XX:+FailOverToOldVerifier|				|如果新的Class校验器检查失败，则使用老的校验器。(Java6新引入选项默认启用。 JDK6最高向下兼容到JDK1.2，而JDK1.2的class info 与JDK6的info存在较大的差异，所以新校验器可能会出现校验失败的情况。关联选项：-XX:+UseSplitVerifier)
-XX:+UseSplitVerifier|					|使用新的Class类型校验器 (新Class类型校验器，将老的校验步骤拆分成了两步：1.类型推断。2.类型校验。新类型校验器通过在javac编译时嵌入类型信息到bytecode中，省略了类型推断这一步，从而提升了classloader的性能。关联选项：-XX:+FailOverToOldVerifier)
-XX:-RelaxAccessControlCheck|			|在Class校验器中，放松对访问控制的检查。作用与reflection里的setAccessible类似。(Java1.6引入)
