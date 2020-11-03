SE生僻点梳理
=====

# 对象引用

## 强引用（StrongReference）
强引用就是指在程序代码之中普遍存在的，如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。比如下面这段代码中的object和str都是强引用：
```java
    Object object = new Object();
    String str = "hello";
```

## 软引用（SoftReference）
软引用是用来描述一些有用但并不是必需的对象，在Java中用java.lang.ref.SoftReference类来表示。对于软引用关联着的对象，只有在内存不足的时候JVM才会回收该对象。因此，这一点可以很好地用来解决OOM的问题，并且这个特性很适合用来实现缓存：比如网页缓存、图片缓存等。
软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中。下面是一个使用示例：
```java
import java.lang.ref.SoftReference;

public class Main {
    public static void main(String[] args) {

        SoftReference<String> sr = new SoftReference<String>(new String("hello"));
        System.out.println(sr.get());
    }
}
```

## 弱引用（WeakReference）
弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。在java中，用java.lang.ref.WeakReference类来表示。下面是使用示例：
```java
import java.lang.ref.WeakReference;

public class Main {
    public static void main(String[] args) {

        WeakReference<String> sr = new WeakReference<String>(new String("hello"));

        System.out.println(sr.get());
        System.gc();                //通知JVM的gc进行垃圾回收
        System.out.println(sr.get());
    }
}
```
第二个输出结果是null，这说明只要JVM进行垃圾回收，被弱引用关联的对象必定会被回收掉。不过要注意的是，这里所说的被弱引用关联的对象是指只有弱引用与之关联，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象（软引用也是如此）。弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中。

## 虚引用（PhantomReference）
虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期。在java中用java.lang.ref.PhantomReference类表示。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。

要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;


public class Main {
    public static void main(String[] args) {
		String hello = new String("Hello");
        ReferenceQueue<String> queue = new ReferenceQueue<String>();
        PhantomReference<String> pr = new PhantomReference<String>(hello, queue);
        System.out.println(pr.get());
		System.out.println(pr.get());
		System.gc();



    }
}
```


# JVM数据监控
ManagementFactory
```java
package com.fei;

import java.lang.management.ClassLoadingMXBean;
import java.lang.management.CompilationMXBean;
import java.lang.management.GarbageCollectorMXBean;
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;
import java.lang.management.MemoryManagerMXBean;
import java.lang.management.MemoryPoolMXBean;
import java.lang.management.MemoryUsage;
import java.lang.management.OperatingSystemMXBean;
import java.lang.management.RuntimeMXBean;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.List;

public class JvmInfo {

	static final long MB = 1024 * 1024;

	public static void main(String[] args) {


		//打印系统信息
		System.out.println("===========打印系统信息==========");
		printOperatingSystemInfo();
		//打印编译信息
		System.out.println("===========打印编译信息==========");
		printCompilationInfo();
		//打印类加载信息
		System.out.println("===========打印类加载信息==========");
		printClassLoadingInfo();
		//打印运行时信息
		System.out.println("===========打印运行时信息==========");
		printRuntimeInfo();
		//打印内存管理器信息
		System.out.println("===========打印内存管理器信息==========");
		printMemoryManagerInfo();
		//打印垃圾回收信息
		System.out.println("===========打印垃圾回收信息==========");
		printGarbageCollectorInfo();
		//打印vm内存
		System.out.println("===========打印vm内存信息==========");
		printMemoryInfo();
		//打印vm各内存区信息
		System.out.println("===========打印vm各内存区信息==========");
		printMemoryPoolInfo();
		//打印线程信息
		System.out.println("===========打印线程==========");
		printThreadInfo();

	}


	private static void printOperatingSystemInfo(){
		OperatingSystemMXBean system = ManagementFactory.getOperatingSystemMXBean();
		//相当于System.getProperty("os.name").
		System.out.println("系统名称:"+system.getName());
		//相当于System.getProperty("os.version").
		System.out.println("系统版本:"+system.getVersion());
		//相当于System.getProperty("os.arch").
		System.out.println("操作系统的架构:"+system.getArch());
		//相当于 Runtime.availableProcessors()
		System.out.println("可用的内核数:"+system.getAvailableProcessors());

		if(isSunOsMBean(system)){
			long totalPhysicalMemory = getLongFromOperatingSystem(system,"getTotalPhysicalMemorySize");
			long freePhysicalMemory = getLongFromOperatingSystem(system, "getFreePhysicalMemorySize");
			long usedPhysicalMemorySize =totalPhysicalMemory - freePhysicalMemory;

			System.out.println("总物理内存(M):"+totalPhysicalMemory/MB);
			System.out.println("已用物理内存(M):"+usedPhysicalMemorySize/MB);
			System.out.println("剩余物理内存(M):"+freePhysicalMemory/MB);

			long  totalSwapSpaceSize = getLongFromOperatingSystem(system, "getTotalSwapSpaceSize");
			long freeSwapSpaceSize = getLongFromOperatingSystem(system, "getFreeSwapSpaceSize");
			long usedSwapSpaceSize = totalSwapSpaceSize - freeSwapSpaceSize;

			System.out.println("总交换空间(M):"+totalSwapSpaceSize/MB);
			System.out.println("已用交换空间(M):"+usedSwapSpaceSize/MB);
			System.out.println("剩余交换空间(M):"+freeSwapSpaceSize/MB);
		}
	}

	private static long getLongFromOperatingSystem(OperatingSystemMXBean operatingSystem, String methodName) {
		try {
			final Method method = operatingSystem.getClass().getMethod(methodName,
					(Class<?>[]) null);
			method.setAccessible(true);
			return (Long) method.invoke(operatingSystem, (Object[]) null);
		} catch (final InvocationTargetException e) {
			if (e.getCause() instanceof Error) {
				throw (Error) e.getCause();
			} else if (e.getCause() instanceof RuntimeException) {
				throw (RuntimeException) e.getCause();
			}
			throw new IllegalStateException(e.getCause());
		} catch (final NoSuchMethodException e) {
			throw new IllegalArgumentException(e);
		} catch (final IllegalAccessException e) {
			throw new IllegalStateException(e);
		}
	}

	private static void printCompilationInfo(){
		CompilationMXBean compilation = ManagementFactory.getCompilationMXBean();
		System.out.println("JIT编译器名称："+compilation.getName());
		//判断jvm是否支持编译时间的监控
		if(compilation.isCompilationTimeMonitoringSupported()){
			System.out.println("总编译时间："+compilation.getTotalCompilationTime()+"秒");
		}
	}

	private static void printClassLoadingInfo(){
		ClassLoadingMXBean classLoad= ManagementFactory.getClassLoadingMXBean();
		System.out.println("已加载类总数："+classLoad.getTotalLoadedClassCount());
		System.out.println("已加载当前类："+classLoad.getLoadedClassCount());
		System.out.println("已卸载类总数："+classLoad.getUnloadedClassCount());

	}

	private static void printRuntimeInfo(){
		RuntimeMXBean runtime = ManagementFactory.getRuntimeMXBean();
		System.out.println("进程PID="+runtime.getName().split("@")[0]);
		System.out.println("jvm规范名称:"+runtime.getSpecName());
		System.out.println("jvm规范运营商:"+runtime.getSpecVendor());
		System.out.println("jvm规范版本:"+runtime.getSpecVersion());
		//返回虚拟机在毫秒内的开始时间。该方法返回了虚拟机启动时的近似时间
		System.out.println("jvm启动时间（毫秒）:"+runtime.getStartTime());
		//相当于System.getProperties
		System.out.println("获取System.properties:"+runtime.getSystemProperties());
		System.out.println("jvm正常运行时间（毫秒）:"+runtime.getUptime());
		//相当于System.getProperty("java.vm.name").
		System.out.println("jvm名称:"+runtime.getVmName());
		//相当于System.getProperty("java.vm.vendor").
		System.out.println("jvm运营商:"+runtime.getVmVendor());
		//相当于System.getProperty("java.vm.version").
		System.out.println("jvm实现版本:"+runtime.getVmVersion());
		List<String> args = runtime.getInputArguments();
		if(args != null && !args.isEmpty()){
			System.out.println("vm参数:");
			for(String arg : args){
				System.out.println(arg);
			}
		}
		System.out.println("类路径:"+runtime.getClassPath());
		System.out.println("引导类路径:"+runtime.getBootClassPath());
		System.out.println("库路径:"+runtime.getLibraryPath());
	}

	private static void printMemoryManagerInfo(){
		List<MemoryManagerMXBean> managers = ManagementFactory.getMemoryManagerMXBeans();
		if(managers != null && !managers.isEmpty()){
			for(MemoryManagerMXBean manager : managers){
				System.out.println("vm内存管理器：名称="+manager.getName()+",管理的内存区="
			+Arrays.deepToString(manager.getMemoryPoolNames())+",ObjectName="+manager.getObjectName());
			}
		}
	}

	private static void printGarbageCollectorInfo(){
		List<GarbageCollectorMXBean> garbages = ManagementFactory.getGarbageCollectorMXBeans();
		for(GarbageCollectorMXBean garbage : garbages){
			System.out.println("垃圾收集器：名称="+garbage.getName()+",收集="+garbage.getCollectionCount()+",总花费时间="
		+garbage.getCollectionTime()+",内存区名称="+Arrays.deepToString(garbage.getMemoryPoolNames()));
		}
	}

	private static void printMemoryInfo(){
		MemoryMXBean memory = ManagementFactory.getMemoryMXBean();
		MemoryUsage headMemory = memory.getHeapMemoryUsage();
		System.out.println("head堆:");
		System.out.println("\t初始(M):"+headMemory.getInit()/MB);
		System.out.println("\t最大(上限)(M):"+headMemory.getMax()/MB);
		System.out.println("\t当前(已使用)(M):"+headMemory.getUsed()/MB);
		System.out.println("\t提交的内存(已申请)(M):"+headMemory.getCommitted()/MB);
		System.out.println("\t使用率:"+headMemory.getUsed()*100/headMemory.getCommitted()+"%");

		System.out.println("non-head非堆:");
		MemoryUsage nonheadMemory = memory.getNonHeapMemoryUsage();
		System.out.println("\t初始(M):"+nonheadMemory.getInit()/MB);
		System.out.println("\t最大(上限)(M):"+nonheadMemory.getMax()/MB);
		System.out.println("\t当前(已使用)(M):"+nonheadMemory.getUsed()/MB);
		System.out.println("\t提交的内存(已申请)(M):"+nonheadMemory.getCommitted()/MB);
		System.out.println("\t使用率:"+nonheadMemory.getUsed()*100/nonheadMemory.getCommitted()+"%");
	}

	private static void printMemoryPoolInfo(){
		List<MemoryPoolMXBean> pools = ManagementFactory.getMemoryPoolMXBeans();
		if(pools != null && !pools.isEmpty()){
			for(MemoryPoolMXBean pool : pools){
				//只打印一些各个内存区都有的属性，一些区的特殊属性，可看文档或百度
				//最大值，初始值，如果没有定义的话，返回-1，所以真正使用时，要注意
				System.out.println("vm内存区:\n\t名称="+pool.getName()+"\n\t所属内存管理者="+Arrays.deepToString(pool.getMemoryManagerNames())
						+"\n\t ObjectName="+pool.getObjectName()+"\n\t初始大小(M)="+pool.getUsage().getInit()/MB
						+"\n\t最大(上限)(M)="+pool.getUsage().getMax()/MB
						+"\n\t已用大小(M)="+pool.getUsage().getUsed()/MB
						+"\n\t已提交(已申请)(M)="+pool.getUsage().getCommitted()/MB
						+"\n\t使用率="+(pool.getUsage().getUsed()*100/pool.getUsage().getCommitted())+"%");

			}
		}
	}

	private static void printThreadInfo(){
		ThreadMXBean thread = ManagementFactory.getThreadMXBean();
		System.out.println("ObjectName="+thread.getObjectName());
		System.out.println("仍活动的线程总数="+thread.getThreadCount());
		System.out.println("峰值="+thread.getPeakThreadCount());
		System.out.println("线程总数（被创建并执行过的线程总数）="+thread.getTotalStartedThreadCount());
		System.out.println("当初仍活动的守护线程（daemonThread）总数="+thread.getDaemonThreadCount());

		//检查是否有死锁的线程存在
		long[] deadlockedIds =  thread.findDeadlockedThreads();
		if(deadlockedIds != null && deadlockedIds.length > 0){
			ThreadInfo[] deadlockInfos = thread.getThreadInfo(deadlockedIds);
			System.out.println("死锁线程信息:");
			System.out.println("\t\t线程名称\t\t状态\t\t");
			for(ThreadInfo deadlockInfo : deadlockInfos){
				System.out.println("\t\t"+deadlockInfo.getThreadName()+"\t\t"+deadlockInfo.getThreadState()
						+"\t\t"+deadlockInfo.getBlockedTime()+"\t\t"+deadlockInfo.getWaitedTime()
						+"\t\t"+deadlockInfo.getStackTrace().toString());
			}
		}
		long[] threadIds = thread.getAllThreadIds();
		if(threadIds != null && threadIds.length > 0){
			ThreadInfo[] threadInfos = thread.getThreadInfo(threadIds);
			System.out.println("所有线程信息:");
			System.out.println("\t\t线程名称\t\t\t\t\t状态\t\t\t\t\t线程id");
			for(ThreadInfo threadInfo : threadInfos){
				System.out.println("\t\t"+threadInfo.getThreadName()+"\t\t\t\t\t"+threadInfo.getThreadState()
						+"\t\t\t\t\t"+threadInfo.getThreadId());
			}
		}

	}

	private static boolean isSunOsMBean(OperatingSystemMXBean operatingSystem) {
		final String className = operatingSystem.getClass().getName();
		return "com.sun.management.OperatingSystem".equals(className)
				|| "com.sun.management.UnixOperatingSystem".equals(className);
	}
}

```

# 堆 非堆 栈
## 堆<->栈
1.Heap Memory是堆内存，Stack Memory是栈内存。
2.Stack memory内存空间由操作系统自动分配和释放，Heap Memory内存空间手动申请和释放的，Heap Memory内存常用new关键字来分配。
3.Stack Memory内存空间有限，Heap Memor的空间是很大的自由区几乎没有空间限制。
在Java中，声明的对象是先在栈内存中为其分配地址空间，在对其进行实例化后则在堆内存中为其分配地址。
* Person p = null ; 只在Stack Memory中为其分配地址空间
* p = new Person(); 则在Heap Memory中为其分配内存地址

在函数中定义的一些基本类型的变量和对象的引用变量都是在函数的栈内存中分配，当在一段代码块定义一个变量时，Java就在栈中为这个变量分配内存空间，当超过变量的作用域后，Java 会自动释放掉为该变量分配的内存空间，该内存空间可以立即被另作它用

堆内存用来存放由 new 创建的对象和数组，在堆中分配的内存，由 Java 虚拟机的自动垃圾回收器来管理。在堆中产生了一个数组或者对象之后，还可以在栈中定义一个特殊的变量，让栈中的这个变量的取值等于数组或对象在堆内存中的首地址，栈中的这个变量就成了数组或对象的引用变量，以后就可以在程序中使用栈中的引用变量来访问堆中的数组或者对象，引用变量就相当于是为数组或者对象起的一个名称。引用变量是普通的变量，定义时在栈中分配，引用变量在程序运行到其作用域之外后被释放。而数组和对象本身在堆中分配，即使程序运行到使用 new 产生数组或者对象的语句所在的代码块之外，数组和对象本身占据的内存不会被释放，数组和对象在没有引用变量指向它的时候，才变为垃圾，不能在被使用，但仍然占据内存空间不放，在随后的一个不确定的时间被垃圾回收器收走(释放掉)。

### 栈大小分配
分配参数：-Xss
特点：
- 局部变量、参数都分配在栈上。
- 栈的内存通常只有几百k，但是它决定了函数调用的深度；如果程序使用了很深的递归函数，而栈内存又非常小，此时很有可能发生栈溢出(java.lang.StackOverflowError)的情况。
- 每一个线程都有独立的栈空间。如果想尽量多跑一些线程的话，就尽量将栈内存缩小，而不是增大。

## 堆<->非堆
按照官方的说法：“Java 虚拟机具有一个堆，堆是运行时数据区域，所有类实例和数组的内存均从此处分配。堆是在 Java 虚拟机启动时创建的。”“在JVM中堆之外的内存称为非堆内存(Non-heap memory)”。可以看出JVM主要管理两种类型的内存：堆和非堆。简单来说堆就是Java代码可及的内存，是留给开发人员使用的；非堆就是JVM留给 自己用的，所以方法区、JVM内部处理或优化所需的内存(如JIT编译后的代码缓存)、每个类结构(如运行时常数池、字段和方法数据)以及方法和构造方法 的代码都在非堆内存中。

Java 虚拟机管理堆之外的内存（称为非堆内存）。
Java 虚拟机具有一个由所有线程共享的方法区。方法区属于非堆内存。它存储每个类结构，如运行时常数池、字段和方法数据，以及方法和构造方法的代码。它是在 Java 虚拟机启动时创建的。方法区在逻辑上属于堆，但 Java 虚拟机实现可以选择不对其进行回收或压缩。与堆类似，方法区的大小可以固定，也可以扩大和缩小。方法区的内存不需要是连续空间。除了方法区外，Java 虚拟机实现可能需要用于内部处理或优化的内存，这种内存也是非堆内存。例如，JIT 编译器需要内存来存储从 Java 虚拟机代码转换而来的本机代码，从而获得高性能。

### 堆内存分配
JVM初始分配的内存由-Xms指定，默认是物理内存的1/64；JVM最大分配的内存由-Xmx指 定，默认是物理内存的1/4。默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制；空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制。因此服务器一般设置-Xms、-Xmx相等以避免在每次GC 后调整堆的大小。

### 非堆内存分配
JVM使用-XX:PermSize设置非堆内存初始值，默认是物理内存的1/64；由XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4。


## 内存设置参数

|           设置项           | 说明                                                                                          |
| :------------------------: | :-------------------------------------------------------------------------------------------- |
|          -Xms512m          | 表示JVM初始分配的堆内存大小为512m（JVM Heap(堆内存)最小尺寸，初始分配）                       |
|         -Xmx1024m          | JVM最大允许分配的堆内存大小为1024m，按需分配（JVM Heap(堆内存)最大允许的尺寸，按需分配）      |
|           -Xmn5M           | 表示堆内存中新生代内存大小为5M，如果超过则会向老年代申请空间。                                |
|     -XX:PermSize=512M      | JVM初始分配的非堆内存                                                                         |
|   -XX:MaxPermSize=1024M    | JVM最大允许分配的非堆内存，按需分配                                                           |
| -XX:MaxMetaspaceSize=100m  | JDK8以后的非堆内存                                                                            |
| -XX:NewSize/-XX:MaxNewSize | 定义YOUNG段的尺寸，NewSize为JVM启动时YOUNG的内存大小；MaxNewSize为最大可占用的YOUNG内存大小。 |
|       -XX:NewRatio=4       | 新生代与老年代的比值为1：4                                                                    |
|    -XX:SurvivorRatio=8     | 设置YOUNG代中Survivor空间和Eden空间的比例1：1：8                                              |
|          -Xss128K          | 设置栈大小                                                                                    |


# 反射

## java.lang.Class

| 方法                    | 用途                                                                                                                  | 备注                                                              |
| :---------------------- | :-------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------- |
| **字段和方法**          |
| getFields               | 获取public的字段                                                                                                      | 包括从父类中继承的                                                |
| getDeclaredFields       | 获取所有的public, protected, default (package) access, and private字段                                                | 不包括父类继承的                                                  |
| getMethods              | 获取public的方法                                                                                                      | 包括从父类中继承的                                                |
| getDeclaredMethods      | 获取所有的public, protected, default (package) access, and private方法                                                | 不包括从父类接口中继承的                                          |
| **类的种类判断**        |
| isArray                 | 判断是否数组类                                                                                                        |
| isInterface             | 判断是否接口类型                                                                                                      |
| isPrimitive             | 是否为基础数据类型                                                                                                    |
| isAnnotation            | 是否注解                                                                                                              | ANNOTATION标记                                                    |
| isSynthetic             | 是否人造类                                                                                                            | SYNTHETIC标记                                                     |
| isEnum                  | 是否枚举类                                                                                                            | ENUM标记                                                          |
| isAnonymousClass        | 是否匿名类                                                                                                            |
| isLocalClass            | 是否局部类                                                                                                            |
| isMemberClass           | 是否类变量                                                                                                            |
| getModifiers            | 获取修饰符                                                                                                            | public, protected, private, final, static, abstract and interface |
| **继承信息**            |
| getInterfaces           | 获取实现的接口                                                                                                        |
| getSuperclass           | 获取继承的父类                                                                                                        |
| getGenericInterfaces    | 获取实现的全部接口                                                                                                    |
| getGenericSuperclass    | 获取父类                                                                                                              |
| getAnnotatedInterfaces  | 获取实现的接口类型信息                                                                                                |
| getAnnotatedSuperclass  | 获取继承的父类类型信息                                                                                                |
| **注解信息**            |
| getAnnotations          | 获取声明的的注解                                                                                                      | 包含继承的注解，注解类需要加上@Inherited注解                      |
| getDeclaredAnnotations  | 获取声明的的注解                                                                                                      | 不包含继承的注解                                                  |
| getDeclaredAnnotation   | 获取声明的的注解的对象信息                                                                                            |
| **注解信息**            |
| getConstructor          | 获取“参数是parameterTypes”的public的构造函数                                                                          |
| getConstructors         | 获取全部的public的构造函数                                                                                            |
| getDeclaredConstructor  | 获取“参数是parameterTypes”的，并且是类自身声明的构造函数，包含public、protected和private方法。                        |
| getDeclaredConstructors | 获取类自身声明的全部的构造函数，包含public、protected和private方法。                                                  |
| **其他**                |
| getEnclosingConstructor | 如果这个类是“其它类的构造函数中的内部类”，调用getEnclosingConstructor()就是这个类所在的构造函数；若不存在，返回null。 |
| getEnumConstants        | 获取枚举类的所有变量值                                                                                                |
| getEnclosingClass       | 获取匿名类或者内部类的立即封闭类                                                                                      |
| getEnclosingMethod      | 获取匿名类的立即封闭方法                                                                                              |
| getComponentType        | 数组类class获取元素的类型信息，普通类则返回空                                                                         |
| getClassLoader          | 获取classLoader                                                                                                       |

# GC

## GC Root

在Java语言中，可作为GC Roots的对象包含以下几种：

* 虚拟机栈(栈帧中的本地变量表)中引用的对象。(可以理解为:引用栈帧中的本地变量表的所有对象)
* 方法区中静态属性引用的对象(可以理解为:引用方法区该静态属性的所有对象)
* 方法区中常量引用的对象(可以理解为:引用方法区中常量的所有对象)
* 本地方法栈中(Native方法)引用的对象(可以理解为:引用Native方法的所有对象)

可以理解为:

1. 首先第一种是虚拟机栈中的引用的对象，我们在程序中正常创建一个对象，对象会在堆上开辟一块空间，同时会将这块空间的地址作为引用保存到虚拟机栈中，如果对象生命周期结束了，那么引用就会从虚拟机栈中出栈，因此如果在虚拟机栈中有引用，就说明这个对象还是有用的，这种情况是最常见的。

2. 第二种是我们在类中定义了全局的静态的对象，也就是使用了static关键字，由于虚拟机栈是线程私有的，所以这种对象的引用会保存在共有的方法区中，显然将方法区中的静态引用作为GC Roots是必须的。

3. 第三种便是常量引用，就是使用了static final关键字，由于这种引用初始化之后不会修改，所以方法区常量池里的引用的对象也应该作为GC Roots。最后一种是在使用JNI技术时，有时候单纯的Java代码并不能满足我们的需求，我们可能需要在Java中调用C或C++的代码，因此会使用native方法，JVM内存中专门有一块本地方法栈，用来保存这些对象的引用，所以本地方法栈中引用的对象也会被作为GC Roots。

**JVM之判断对象是否存活（引用计数算法、可达性分析算法，最终判定）**

## finalize():

即使在可达性分析算法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历再次标记过程。标记的前提是对象在进行可达性分析后发现没有与GC Roots相连接的引用链。

1.第一次标记并进行一次筛选。
    筛选的条件是此对象是否有必要执行finalize()方法。
    当对象没有覆盖finalize方法，或者finzlize方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”，对象被回收。

2.第二次标记
    如果这个对象被判定为有必要执行finalize（）方法，那么这个对象将会被放置在一个名为：F-Queue的队列之中，并在稍后由一条虚拟机自动建立的、低优先级的Finalizer线程去执行。这里所谓的“执行”是指虚拟机会触发这个方法，但并不承诺会等待它运行结束。这样做的原因是，如果一个对象finalize（）方法中执行缓慢，或者发生死循环（更极端的情况），将很可能会导致F-Queue队列中的其他对象永久处于等待状态，甚至导致整个内存回收系统崩溃。
    Finalize（）方法是对象脱逃死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模标记，如果对象要在finalize（）中成功拯救自己----只要重新与引用链上的任何的一个对象建立关联即可，譬如把自己赋值给某个类变量或对象的成员变量，那在第二次标记时它将移除出“即将回收”的集合。如果对象这时候还没逃脱，那基本上它就真的被回收了。

流程图如下：
![GC Fianlize](/images/2020/03/gc-finalize.jpg "Tip")


