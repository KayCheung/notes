# JVM Options

> http://jvm-options.tech.xebia.fr/
> http://www.oracle.com/technetwork/java/tuning-139912.html#section4.2.5

#### 查看当前版本JDK的所有VM Options：java -XX:+PrintFlagsFinal
	1. 以 ‘-’ 开头的是标准VM选项，VM规范的选项
	2. 以 ‘-X’ 开头的都是非标准的（这些参数并不能保证在所有的JVM上都被实现），而且如果在新版本有什么改动也不会发布通知。
	3. 以 ‘-XX’ 开头的都是不稳定的并且不推荐在生产环境中使用。这些参数的改动也不会发布通知。
	4. Bool 型参数选项：-XX:+ 打开， -XX:- 关闭。（比如-XX:+PrintGCDetails）
	5. 数字型参数选项通过-XX:=设定。数字可以是 m/M(兆字节)，k/K(千字节)，g/G(G字节)。比如：32K表示32768字节。（比如-XX:MaxPermSize=64m）
	6. String参数选项通过-XX:=设定，通常用来指定一个文件，路径，或者一个命令列表。（比如-XX:HeapDumpPath=./java_pid.hprof）
	7. 命令 java -help可以列出java 应用启动时标准选项（见附录标准VM参数表，不同的JVM实现是不同的）。java -X可以列出不标准的参数（这是JVM的扩展特性）。-X相关的选项不是标准的，被改变也不会通知。如果你想查看当前应用使用的JVM参数，你可以使用：ManagementFactory.getRuntimeMXBean().getInputArguments()。
	


参数|默认值|描述
----|------|----
1. 类
-Xbootclasspath							指定加载不需要校验的类(跳过必要的类加载前的校验，能够减少加载时间，但是不安全)
-XX:+FailOverToOldVerifier				如果新的Class校验器检查失败，则使用老的校验器。(Java6新引入选项默认启用。 JDK6最高向下兼容到JDK1.2，而JDK1.2的class info 与JDK6的info存在较大的差异，所以新校验器可能会出现校验失败的情况。关联选项：-XX:+UseSplitVerifier)
-XX:+UseSplitVerifier					使用新的Class类型校验器 (新Class类型校验器，将老的校验步骤拆分成了两步：1.类型推断。2.类型校验。新类型校验器通过在javac编译时嵌入类型信息到bytecode中，省略了类型推断这一步，从而提升了classloader的性能。关联选项：-XX:+FailOverToOldVerifier)
-XX:-RelaxAccessControlCheck			在Class校验器中，放松对访问控制的检查。作用与reflection里的setAccessible类似。(Java1.6引入)


2. 编译/JIT
-Xbatch									禁用后台编译(关闭后台代码编译,强制在前台编译,编译完成之后才能进行代码执行,默认情况下,jvm在后台进行编译，若没有编译完成,则前台运行代码时以解释模式运行)
-XX:+AggressiveOpts						开启编译器性能优化。
-XX:+PrintCompilation					往stdout打印方法被JIT编译时的信息。
-XX:CompileThreshold=1000				通过JIT编译器，将方法编译成机器码的触发阀值，可以理解为调用方法的次数，例如调1000次，将方法编译为机器码。 [-client: 1,500]
-XX:ReservedCodeCacheSize=32m			设置代码缓存的最大值，编译时用
-XX:-CITime								打印JIT编译器编译耗时。
-XX:AllocatePrefetchLines=1				在使用JIT生成的预读取指令分配对象后读取的缓存行数。如果上次分配的对象是一个实例则默认值是1，如果是一个数组则是3
-XX:InlineSmallCode=n					当编译的代码小于指定的值时,内联编译的代码。（默认值取决于当前JVM 设置）
-XX:MaxInlineSize=35					内联方法的最大字节数。
-XX:FreqInlineSize=n					内联频繁执行的方法的最大字节码大小。
-XX:AllocatePrefetchStyle=1				预读取指令的生成代码风格  0-无预读取指令生成   1-在每次分配后执行预读取命令   2-当预读取指令执行后使用TLAB()分配水印指针来找回入口
-XX:-UseFastAccessorMethods				优化原始类型的getter方法性能。
-XX:LoopUnrollLimit=n					代表节点数目小于给定值时打开循环体。
-XX:+UseLoopPredicate					编译器对循环的优化


3. 堆
-Xms1024m -Xmx1024m -Xmn2g -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:PermSize=64m -XX:MaxPermSize=64m
-Xms 									堆初始值(物理内存的1/64(<1GB) 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制) 此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
-Xmx									堆最大可用值(物理内存的1/4(<1GB) 默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制)
-Xmn									年轻代大小，一般设为整个堆的1/3到1/4左右。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
-XX:SurvivorRatio=8						年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
-XX:TargetSurvivorRatio=50				实际使用的survivor空间大小占比。默认是47%，最高90%。
-XX:NewRatio=2							设置新生代(包括Eden和两个Survivor区)与老年代的比值(除去永久代)。例如:值为3，则表示年轻代与年老代比值为1 ：3，年轻代占整个年轻代加年老代和的1/4。(Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。)
-XX:MaxNewSize=64m						新生代占整个堆内存的最大值。从Java1.4开始, MaxNewSize成为 NewRatio的一个函数	
-XX:NewSize=2m							新生代预估上限的默认值(新生代对象生成时占用内存的默认值)。(5.0以后: 64 bit Vms 会增大预设值的30%; x86: 1m; x86, 5.0以后: 640k; 其他:2.125m)
-XX:PermSize   							方法区（永久区）初始值（物理内存的1/64）。
-XX:MaxPermSize    						方法区（永久区）最大值（物理内存的1/4）, 占整个堆内存的最大值。 Java1.5以后：64 bit VMs会增大预设值的30%；1.4 amd64:：96m；1.3.1 -client: 32m；其他默认 64m
-XX:LargePageSizeInBytes=4m				设置堆内存的内存页最大值(默认值：4m, amd64位：2m), 内存页的大小不可设置过大， 会影响Perm的大小
-XX:MaxTenuringThreshold=15				设置垃圾最大存活阀值（垃圾最大年龄/次数），并行回收机制默认为15，CMS默认为4。(默认值：15，最大值：15) 如果设置为0的话，则新生代对象不经过Survivor区，直接进入老年代。对于老年代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则新生代对象会在Survivor区进行多次复制，这样可以增加对象再新生代的存活时间，增加在新生代即被回收的概率，该参数只有在串行GC时才有效。
-XX:InitialTenuringThreshold=7			设置初始的对象在新生代中最大存活次数。
-XX:SoftRefLRUPolicyMSPerMB=1s			每兆堆空闲空间中SoftReference的存活时间（单位秒，默认1s）
-XX:PretenureSizeThreshold=0			对象超过多大是直接在老年代分配(新生代采用Parallel Scavenge GC时无效，另一种直接在老年代分配的情况是大的数组对象，且数组中无外部引用对象.)
-XX:TLABWasteTargetPercent=1			TLAB占eden区的百分比
-XX:+AggressiveHeap						试图是使用大量的物理内存(长时间大内存使用的优化，能检查计算资源（内存， 处理器数量，至少需要256MB内存), jdk1.8 不存在
-XX:AllocatePrefetchDistance=n			为对象分配设置预取距离。（默认值取决于当前JVM 设置）
-XX:+AlwaysPreTouch						在JVM 初始化时预先对Java堆进行摸底。
-XX:InitialTenuringThreshold=7			设置初始的对象在新生代中最大存活次数。


4. 栈
-Xss128k -XX:ThreadStackSize=512
-Xss									设置线程堆栈空间大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。根据应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
-XX:ThreadStackSize						与Xss一样，0代表使用默认值，不能带单位
-XX:+UseTLAB							启用线程本地缓存区(Thread Local), Java1.4.2以前和使用-client选项时：默认关闭其余版本默认启用


5. 非堆
-XX:MaxDirectMemorySize=200m
-XX:MaxDirectMemorySize=200m    		直接内存最大可用空间，设置不当可能导致系统OOM，NIO direct memory可申请的空间的上限就是-Xmx减去一个survivor space的预留大小。 当MaxDirectMemorySize参数没被显式设置时它的值就是-1。


6. 字符串
-XX:+UseStringCache						缓存常用字符串。
-XX:+UseCompressedStrings				其中，对于不需要16位字符的字符串，可以使用byte[] 而非char[]。对于许多应用，这可以节省内存，但速度较慢（5％-10％）
-XX:+OptimizeStringConcat				在可能的情况下优化字符串连接操作。


7. 线程
-XX:PreBlockSpin=10 -XX:+UseBoundThreads -XX:-UseSpinning -XX:+UseVMInterruptibleIO
-XX:PreBlockSpin 						控制多线程自旋锁优化的自旋次数。 前置选项：-XX:+UseSpinning
-XX:+UseSpinning						启用多线程自旋锁优化。(Java1.4.2和1.5需要手动启用,Java1.6默认已启用)
-XX:+UseThreadPriorities				使用本地线程的优先级
-XX:+UseBiasedLocking					启用偏向锁。


8. 垃圾收集器
-Xincgc									开启增量gc（默认为关闭）；这有助于减少长时间GC时应用程序出现的停顿；但由于可能和应用程序并发执行，所以会降低CPU对应用的处理能力。
-Xnoclassgc								禁用类垃圾收集
-XX:-UseSerialGC						使用串行垃圾收集器。(-client时启用)
-XX:+UseParNewGC						设置新生代为并行收集(可与CMS收集同时使用，JDK 5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值)
-XX:+UseParallelGC						为新生代使用并行GC，年老代使用单线程Mark-Sweep-Compact的垃圾收集器。(-server时启用其他情况下：默认关闭)
-XX:+UseParallelOldGC					为老年代和新生代都使用并行清除的垃圾收集器。开启此选项将自动开启-XX:+UseParallelGC 选项, 这个是JAVA 6出现的参数选项
-XX:ParallelGCThreads					配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。(此值最好配置与处理器数目相等，同样适用于CMS)
-XX:MaxGCPauseMillis=4294967295			设置并行收集最大暂停时间，这是一个理想目标，JVM将尽最大努力来实现它。(如果无法满足此时间，JVM会自动调整新生代大小，以满足此值.)	
-XX:+UseCMSInitiatingOccupancyOnly		使用手动定义初始化定义开始CMS收集(禁止hostspot自行触发CMS GC)
-XX:CMSInitiatingOccupancyFraction=70	使用CMS作为垃圾回收,使用70％后开始CMS收集(该值的设置需要满足以下公式CMSInitiatingOccupancyFraction计算公式)
-XX:CMSInitiatingPermOccupancyFraction	设置Perm Gen使用到达多少比率时触发
-XX:+UseConcMarkSweepGC					启用CMS低停顿垃圾收集器(测试中配置这个以后， -XX:NewRatio=4的配置失效了，原因不明，所以此时新生代大小最好用-Xmn设置)
-XX:+CMSClassUnloadingEnabled			永久代CMS方式GC
-XX:+CMSIncrementalMode					设置为增量模式(用于单CPU情况)
-XX:InitiatingHeapOccupancyPercent=45	启动一个并发垃圾收集周期所需要达到的整堆占用比例。这个比例是指整个堆的占用比例而不是某一个代（例如G1）,如果这个值是0则代表‘持续做GC’。默认值是45
-XX:CMSFullGCsBeforeCompaction			多少次后进行内存压缩(由于并发收集器不对内存空间进行压缩，整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低)
-XX:+UseCMSCompactAtFullCollection		在FullGC的时候对老年代进行压缩(CMS是不会移动内存的， 因此这个非常容易产生碎片，导致内存不够用，因此内存的压缩这个时候就会被启用。增加这个参数是个好习惯。可能会影响性能，但是可以消除碎片)
-XX:+CMSParallelRemarkEnabled			降低标记停顿
-XX:+UseAdaptiveSizePolicy				自动选择新生代区大小和相应的Survivor区比例(设置此选项后，并行收集器会自动选择新生代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开)
-XX:-DisableExplicitGC					禁止在运行期显式地调用 System.gc(),开启该选项后,GC的触发时机将由Garbage Collector全权掌控, 这个参数需要严格的测试.(注意：你熟悉的代码里没调用System.gc()，不代表你依赖的框架工具没在使用。例如RMI就在多数用户毫不知情的情况下，显示地调用GC来防止自身OOM。请仔细权衡禁用带来的影响。)
-XX:+CollectGen0First					FullGC时是否先YGC
-XX:GCTimeRatio							设置垃圾回收时间占程序运行时间的百分比(公式为1/(1+n))
-XX:+ScavengeBeforeFullGC				在Full GC 前触发一次 Minor GC
-XX:ConcGCThreads=n						Number of threads concurrent garbage collectors will use. The default value varies with the platform on which the JVM is running.
-XX:-UseG1GC							使用G1垃圾处理器
-XX:G1ReservePercent=10					设置保留,用来做假天花板以减少晋升（新生代对象晋升到老生代）失败可能性的堆数目。
-XX:G1HeapRegionSize=n					使用G1垃圾回收器，java堆被划分成统一大小的区块。这个选项设置每个区块的大小。最小值是1Mb，最大值是32Mb。
-XX:MaxHeapFreeRatio=70					GC后，如果发现空闲堆内存占到整个预估上限值的70%，则收缩预估上限值。 
-XX:MinHeapFreeRatio=40					GC后，如果发现空闲堆内存占到整个预估上限值的40%，则增大上限值。关联选项：-XX:MaxHeapFreeRatio=70
-XX:+HandlePromotionFailure				关闭新生代收集担保,Java1.5以前默认关闭,Java1.6后默认启用.
-XX:+UseGCOverheadLimit					限制GC的运行时间。如果GC耗时过长，就抛OutOfMemoryError。(Java1.6引入)
-XX:+UseCompressedOops					使用compressed pointers。这个参数默认在64bit的环境下默认启动，但是如果JVM的内存达到32G后，这个参数就会默认为不启动，因为32G内存后，压缩就没有多大必要了，要管理那么大的内存指针也需要很大的宽度了
-XX:-UseLargePages						用大内存分页。(调整内存页的方法和性能提升原理，详见(http://www.oracle.com/technetwork/java/javase/tech/largememory-jsp-137182.html)关联选项：-XX:LargePageSizeInBytes=4m)


9. 垃圾收集信息/内存使用情况
com.sun.management.HotSpotDiagnosticMXBean API 和 jconsole 动态启用部分Print
-Xloggc:filename						把相关日志信息记录到文件以便分析(与上面几个配合使用)
-XX:-UseGCLogFileRotation				开启GC 日志文件切分功能，前置选项 -Xloggc			
-XX:NumberOfGClogFiles=1				设置切分GC 日志文件数量，文件命名格式：.0, .1, ..., .n-1
-XX:GCLogFileSize=8K					GC日志文件切分大小。
-XX:+PrintGC    						每次触发GC的时候打印相关日志,和-verbose:gc一样
-XX:+PrintGCDetails    					更详细的GC日志
-XX:+PrintGCTimeStamps 					打印GC停顿耗时。配合上述PrintGC参数使用, 或者写成-XX:+PrintGC:PrintGCTimeStamps
-XX:+PrintGC:PrintGCTimeStamps			可与-XX:+PrintGC -XX:+PrintGCDetails混合使用
-XX:+PrintGCApplicationStoppedTime		打印垃圾回收期间程序暂停的时间。可与上面混合使用
-XX:+PrintGCApplicationConcurrentTime   打印每次垃圾回收前，程序未中断的执行时间(打印应用程序执行时间)
-XX:+PrintHeapAtGC    					打印GC前后的详细堆栈信息
-XX:+PrintReferenceGC   	 			跟踪系统内的软引用，弱引用，虚引用和finallize队列。
-XX:+PrintClassHistogram				在Windows下, 按ctrl-break或Linux下是执行kill -3（发送SIGQUIT信号）时，打印class柱状图。 jmap -histo pid也实现了相同的功能。
-XX:+PrintClassHistogramBeforeFullGC	FullGC前打印类信息
-XX:+PrintTLAB							查看TLAB空间的使用情况
-XX:+PrintTenuringDistribution			打印对象的存活期限信息。
-XX:-PrintConcurrentLocks				在thread dump的同时，打印java.util.concurrent的锁状态。jstack -l pid 也同样实现了同样的功能。		
-XX:+PrintCommandLineFlags    			Java启动时，往stdout打印当前启用的非稳态jvm options。(例如： -XX:+UseConcMarkSweepGC -XX:+HeapDumpOnOutOfMemoryError -XX:+DoEscapeAnalysis)
-XX:ErrorFile=./hs_err_pid<pid>.log		如果JVM crashed，将错误日志输出到指定文件路径。
-XX:OnError="<cmd args>;<cmd args>"		当java每抛出一个ERROR时，运行指定命令行指令集。指令集是与OS环境相关的，在Linux下多数是.sh脚本，windows下是.bat批处理。
-XX:OnOutOfMemoryError="<cmd args>;<cmd args>" 当第一次发生java.lang.OutOfMemoryError 时，运行指定命令行指令集。指令集是与OS环境相关的，在linux下多数是.sh脚本，windows下是.bat批处理。
-XX:+PerfSaveDataToFile					当java进程因java.lang.OutOfMemoryError 异常或crashed 被强制终止后，生成一个堆快照文件。
-XX:+HeapDumpOnOutOfMemoryError			在java.lang.OutOfMemoryError 异常出现时，输出一个dump.core文件，记录当时的堆内存快照（见 -XX:HeapDumpPath 的描述）。
-XX:HeapDumpPath						堆内存快照的存储文件路径。(配合-XX:+HeapDumpOnOutOfMemoryError使用)


10. 类跟踪信息
-verbose:class    						跟踪类的加载和卸载
-XX:+TraceClassLoading    				打印class装载信息到stdout。记Loaded状态。例如： [Loaded java.lang.Object from /opt/taobao/install/jdk1.6.0_07/jre/lib/rt.jar]
-XX:+TraceClassUnloading    			打印class的卸载信息到stdout。记Unloaded状态。
-XX:+PrintClassHistogram    			查看运行时类的分布情况，使用时在控制台按ctrl+break
-XX:-TraceClassLoadingPreorder			按class的引用/依赖顺序打印类装载信息到stdout。不同于 TraceClassLoading，本选项只记 Loading状态。 例如： [Loading java.lang.Object from /home/confsrv/jdk1.6.0_14/jre/lib/rt.jar]
-XX:-TraceClassResolution				打印所有静态类，常量的代码引用位置。用于debug。 例如： RESOLVE java.util.HashMap java.util.HashMap$Entry HashMap.java:209 说明HashMap类的209行引用了静态类 java.util.HashMap$Entry
-XX:-TraceLoaderConstraints				打印class的装载策略变化信息到stdout。


11. 系统参数查看
java -XX:+PrintFlagsFinal
-XX:+PrintFlagsFinal					打印所有JVM Options


12. 虚拟机工作模式
-client    								默认工作模式
-server    								server工作模式，启动虚拟机时需要显式指定
-Xint									仅解释模式执行(设置jvm以解释模式运行，所有的字节码将被直接执行，而不会编译成本地码。)
-Xmixed           						混合模式执行 (默认)


13. 未分类
-Xcheck:jni								对JNI函数进行附加check；此时jvm将校验传递给JNI函数参数的合法性，在本地代码中遇到非法数据时，jmv将报一个致命错误而终止；使用该参数后将造成性能下降，请慎用。
-Xfuture								让jvm对类文件执行严格的格式检查（默认jvm不进行严格格式检查），以符合类文件格式规范，推荐开发人员使用该参数。
-Xrs									减少 Java/VM 对操作系统信号的使用 (请参阅文档)
-Xprof									性能诊断, 输出 cpu 配置文件数据
-Xrunhprof								性能诊断
-d32									使用 32 位数据模型 (如果可用)
-d64									使用 64 位数据模型 (如果可用)
-server									选择 "server" VM（默认 VM 是 server，因为您是在服务器类计算机上运行。）
-cp										目录和 zip/jar 文件的类搜索路径
-classpath								目录和 zip/jar 文件的类搜索路径（用  ':' 分隔的目录、JAR档案和 ZIP档案列表，用于搜索类文件, 例如：/spring-beans-4.3.3.RELEASE.jar:/spring-context-4.3.3.RELEASE.jar:/spring-context-support-4.2.8.RELEASE.jar:/spring-core-4.3.3.RELEASE.jar:）
-D<名称>=<值>							设置系统属性(例如：-Ddisconf.conf_server_host=10.10.1.1:8101)
-verbose:[class|gc|jni]					启用详细输出
-version								输出jdk版本
-version:<值>							Deprecated
-showversion							输出产品版本并继续
-jre-restrict-search					Deprecated
-no-jre-restrict-search					Deprecated
-? -help								输出此帮助消息
-X										输出非标准选项的帮助
-ea[:<包名、类名>]						断言
-enableassertions[:<包名、类名>]			按指定的粒度启用断言
-da[:<包名、类名>]
-disableassertions[:<包名、类名>]		禁用具有指定粒度的断言
-esa									启用系统断言
-enablesystemassertions					启用系统断言
-dsa									禁用系统断言
-disablesystemassertions				禁用系统断言
-agentlib:<libname>[=<选项>]				加载本机代理库 <libname>(例如 -agentlib:hprof另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help)
-agentpath:<pathname>[=<选项>]			按完整路径名加载本机代理库
-javaagent:<jarpath>[=<选项>]			加载 Java 编程语言代理, 请参阅 java.lang.instrument
-splash:<imagepath>						使用指定的图像显示启动屏幕


14. Solaris
-XX:-AllowUserSignalHandlers			允许为java进程安装信号处理器
-XX:+MaxFDLimit							设置java进程可用文件描述符为操作系统允许的最大值。
-XX:+UseAltSigs							为了防止与其他发送信号的应用程序冲突，允许使用候补信号替代 SIGUSR1和SIGUSR2。
-XX:+UseLWPSynchronization				使用轻量级进程（内核线程）替换线程同步
-XX:+UseVMInterruptibleIO				在solaris中，允许运行时中断线程。
-XX:-UseISM								http://www.oracle.com/technetwork/java/ism-139376.html
-XX:+UseMPSS							启用solaris的MPSS，不能与ISM同时使用。
-XX:-ExtendedDTraceProbes				启用dtrace(http://docs.oracle.com/javase/6/docs/technotes/guides/vm/dtrace.html)诊断
-XX:+UseBoundThreads					绑定所有的用户线程到内核线程。 减少线程进入饥饿状态（得不到任何cpu time）的次数。

15. 废弃
-XX:AltStackSize=16384					备用信号堆栈大小（以字节为单位）


==============================================

-XX:-UseSpinning

自旋锁优化原理
大家知道，Java的多线程安全是基于Lock机制实现的，而Lock的性能往往不如人意。原因是，monitorenter与monitorexit这两个控制多线程同步的bytecode原语，
是JVM依赖操作系统互斥(mutex)来实现的。互斥是一种会导致线程挂起，并在较短的时间内又必须重新调度回原线程的，较为消耗资源的操作。为了避免进入OS互斥，
Java6的开发者们提出了自旋锁优化。
 自旋锁优化的原理是在线程进入OS互斥前，通过CAS自旋一定的次数来检测锁的释放。如果在自旋次数未达到预设值前锁已被释放，则当前线程会立即持有该锁。
 关联选项：
-XX:PreBlockSpin=10



-XX:+HandlePromotionFailure

什么是新生代收集担保？
在一次理想化的minor gc中，Eden和First Survivor中的活跃对象会被复制到Second Survivor。然而，Second Survivor不一定能容纳下所有从E和F区copy过来的活跃对象。
为了确保minor gc能够顺利完成，GC需要在年老代中额外保留一块足以容纳所有活跃对象的内存空间。这个预留操作，就被称之为新生代收集担保（New Generation Guarantee）。
如果预留操作无法完成时，仍会触发major gc(full gc)。
为什么要关闭新生代收集担保？
因为在年老代中预留的空间大小，是无法精确计算的。为了确保极端情况的发生，GC参考了最坏情况下的新生代内存占用，即Eden+First Survivor。这种策略无疑是在浪费年老代内存，
从时序角度看，还会提前触发Full GC。为了避免如上情况的发生，JVM允许开发者手动关闭新生代收集担保。在开启本选项后，minor gc将不再提供新生代收集担保，
而是在出现survior或年老代不够用时，抛出promotion failed异常。

-XX:MaxHeapFreeRatio=70
什么是预估上限值？
JVM在启动时，会申请最大值（-Xmx指定的数值）的地址空间，但其中绝大部分空间不会被立即分配(virtual)。它们会一直保留着，直到运行过程中，JVM发现实际占用接近已分配上限值时，
才从virtual里再分配掉一部分内存。这里提到的已分配上限值，也可以叫做预估上限值。
引入预估上限值的好处是，可以有效地控制堆的大小。堆越小，GC效率越高嘛。注意：预估上限值的大小一定小于或等于最大值。

-XX:HeapDumpPath=./java_pid<pid>.hprof

什么是堆内存快照？
当java进程因OOM或crash被OS强制终止后，会生成一个hprof（Heap PROFling）格式的堆内存快照文件。该文件用于线下调试，诊断，查找问题。
文件名一般为 java_<pid>_<date>_<time>_heapDump.hprof 解析快照文件，可以使用 jhat, eclipse MAT，gdb等工具。





