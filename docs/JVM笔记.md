## JVM笔记

一 分区及流程图

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/JVM%E6%B5%81%E7%A8%8B.png)

**ClassLoader:**

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8.png)

作用：负责加载class文件，class文件在文件开头有特定的文字标识（cafe babe），将class文件字节码内容加载到内存中，并将这些内容转换成方法中的运行时数据结构，并且ClassLoader只负责class文件的加载至于它是否可以运行，则由Execution Engine决定。

**ClassLoader的双亲委派机制:**

 作用: 主要是能够保障Java平台的安全性, 防止内存中出现多份同样的字节码。

 实现:  

- 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。
- 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。
- 如果BootStrapClassLoader加载失败（例如在$JAVA_HOME/jre/lib里未查找到该class），会使用ExtClassLoader来尝试加载；
- 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

**沙箱安全机制：**

沙箱机制是由基于双亲委派机制上采取的一种JVM的自我保护机制, 假设你要写一个java.lang.String 的类, 由于双亲委派机制的原理, 此请求会先交给Bootstrap试图进行加载, 但是Bootstrap在加载类时首先通过包和类名查找rt.jar中有没有该类, 有则优先加载rt.jar包中的类, 因此就保证了java的运行机制不会被破坏.。

**程序计数器（PC寄存器）：**

1. PC寄存器（ PC register ）：每个线程启动的时候，都会创建一个PC（Program Counter，程序计数器）寄存器。PC寄存器里保存有当前正在执行的JVM指令的地址。 每一个线程都有它自己的PC寄存器，也是该线程启动时创建的。保存下一条将要执行的指令地址的寄存器是 ：PC寄存器。PC寄存器的内容总是指向下一条将被执行指令的地址，这里的地址可以是一个本地指针，也可以是在方法区中相对应于该方法起始指令的偏移量。
2. 每个线程都有一个程序计数器，是线程私有的,就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址,也即将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不记。
3. 这块内存区域很小，它是当前线程所执行的字节码的行号指示器，字节码解释器通过改变这个计数器的值来选取下一条需要执行的字节码指令。
4. 如果执行的是一个Native方法，那这个计数器是空的。

**方法区：**

作用：供各线程共享的运行时内存区域。它**存储了每一个类的结构信息**，例如运行时常量池、字段和方法数据、构造方法和普通方法的字节码内容。这是规范，但在不同虚拟机里面实现是不一样的，最典型的就是永久代（PermGen Space）和元空间（Metaspace）。但是，**实例变量存在堆内存中，与方法区无关**。

**栈区：**

作用：栈管运行，堆管存储。栈也叫栈内存，主管Java程序的运行，是在线程创建时创建，它的生命周期跟随线程的生命周期。线程结束时栈内存也就释放，**对于栈来说不存在垃圾回收问题**，是线程私有的。**8种基本数据类型的变量+对象的引用变量+实例方法都是在栈内存中分配**。

栈存储：栈帧中主要保存三类数据：

- 本地变量（Local variables）：输入参数和输出参数以及方法内的变量。
- 栈操作（OPerate Stack）：记录入栈、出栈的操作。
- 栈帧数据（Frame Data）：包括类文件、方法等。

栈运行原理：栈中的数据都是以栈帧（Stack Frame）的格式存在，栈帧是一个内存区块，是一个数据集，是一个有关方法和运行期数据的数据集；

当一个方法A被调用时就产生了一个栈帧F1，并被压入到栈中，

A方法又调用了B方法，于是产生栈帧F2也被压入栈，

B方法又调用了C方法，于是产生栈帧F3也被压入栈，

。。。。。。

执行完毕后按照栈FILO的特性依次弹出F3、F2、F1栈帧。

**每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息**，每一个方法从调用直至执行完毕的过程就对应着一个栈帧从入栈到出栈的过程。栈的大小和具体的JVM实现有关，通常在256K~756K之间，约等于1Mb左右。

**栈、堆及方法区的交互关系：**

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%A0%86%E6%A0%88%E5%8F%8A%E6%96%B9%E6%B3%95%E5%8C%BA%E7%9A%84%E5%85%B3%E7%B3%BB.png)

Hotspot是使用指针的方式来访问对象:

Java堆中会存放访问**类元数据**的地址,  reference存储的就是对象的地址。

**堆区：**

一个JVM实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件后，需要把类、方法、常变量放到堆内存中，保存所有引用类型的真实信息，以方便执行器执行，堆内存分为三部分：

- Young Generation Space  新生代                    Young/New
- Tenure generation space   老年代                     Old/ Tenure
- Permanent Space              永久代/元空间                         Perm

新生区：

新生区是类的诞生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器收集，结束生命。新生区又分为两部分： 伊甸区（Eden space）和幸存者区（Survivor pace） ，所有的类都是在伊甸区被new出来的。幸存区有两个： 0区（Survivor 0 space / From Survivor）和1区（Survivor 1 space / To Survivor）。当伊甸园的空间用完时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收(Minor GC)，将伊甸园区中的不再被其他对象所引用的对象进行销毁。然后将伊甸园中的剩余对象移动到幸存 0区。若幸存 0区也满了，再对该区进行垃圾回收，然后移动到 1 区。那如果1 区也满了呢？再移动到养老区。若养老区也满了，那么这个时候将产生MajorGC（FullGC），进行养老区的内存清理。若养老区执行了Full GC之后发现依然无法进行对象的保存，就会产生OOM异常“**OutOfMemoryError**”。

如果出现java.lang.OutOfMemoryError: Java heap space异常，说明Java虚拟机的堆内存不够。原因有二：
（1）Java虚拟机的堆内存设置不够，可以通过参数-Xms、-Xmx来调整。
（2）代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）。

**MinorGC的过程（复制->清空->互换：**

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%A0%86%E5%8C%BA%E7%9A%84%E5%88%92%E5%88%86.png>)

1. eden、SurvivorFrom 复制到 SurvivorTo，年龄+1 
   首先，当Eden区满的时候会触发第一次GC,把还活着的对象拷贝到SurvivorFrom区，当Eden区再次触发GC的时候会扫描Eden区和From区域,对这两个区域进行垃圾回收，经过这次回收后还存活的对象,则直接复制到To区域（如果有对象的年龄已经达到了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1。
2. 清空 eden、SurvivorFrom 
   然后，清空Eden和SurvivorFrom中的对象，也即复制之后有交换，谁空谁是to。
3. SurvivorTo和 SurvivorFrom 互换 
   最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区。部分对象会在From和To区域中复制来复制去,如此交换15次(由JVM参数MaxTenuringThreshold决定,这个参数默认是15),最终如果还是存活,就存入到老年代。

实际而言，方法区（Method Area）和堆一样，是各个线程共享的内存区域，它用于**存储虚拟机加载的：类信息+普通常量+静态常量+编译器编译后的代码等等，虽然JVM规范将方法区描述为堆的一个逻辑部分，但它却还有一个别名叫做Non-Heap(非堆)，目的就是要和堆分开。**

对于HotSpot虚拟机，很多开发者习惯将方法区称之为“永久代(Parmanent Gen)” ，但严格本质上说两者不同，或者说使用永久代来实现方法区而已，**永久代是方法区(相当于是一个接口interface)的一个实现，jdk1.7的版本中，已经将原本放在永久代的字符串常量池移走。**

**堆区参数调优：**

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/java8%E5%A0%86%E5%8C%BA%E6%9E%84%E6%88%90.png)

在Java8中，永久代已经被移除，被一个称为元空间的区域所取代。元空间的本质和永久代类似。

元空间与永久代之间最大的区别在于：
永久代使用的JVM的堆内存，但是java8以后的**元空间并不在虚拟机中而是使用本机物理内存。**

因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入 native memory, 字符串池和类的静态变量放入 java 堆中，这样可以加载多少类的元数据就不再由MaxPermSize 控制, 而由系统的实际可用空间来控制。 

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%A0%86%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E.png>)

```java
public static void main(String[] args){
long maxMemory = Runtime.getRuntime().maxMemory() ;//返回 Java 虚拟机试图使用的最大内存量。
long totalMemory = Runtime.getRuntime().totalMemory() ;//返回 Java 虚拟机中的内存总量。
System.out.println("MAX_MEMORY = " + maxMemory + "（字节）、" + (maxMemory / (double)1024 / 1024) + "MB");
System.out.println("TOTAL_MEMORY = " + totalMemory + "（字节）、" + (totalMemory / (double)1024 / 1024) + "MB");
}
```

**VM参数：	-Xms1024m -Xmx1024m -XX:+PrintGCDetails**

****

**垃圾回收机制（GC）：**

JVM在进行GC时，并非每次都对上面三个内存区域一起回收的，大部分时候回收的都是指新生代。
因此GC按照回收的区域又分了两种类型，一种是普通GC（minor GC），一种是全局GC（major GC or Full GC）

**Minor GC和Full GC的区别：**

- 普通GC（minor GC）：只针对新生代区域的GC,指发生在新生代的垃圾收集动作，因为大多数Java对象存活率都不高，所以Minor GC非常频繁，一般回收速度也比较快。 
- 全局GC（major GC or Full GC）：指发生在老年代的垃圾收集动作，出现了Major GC，经常会伴随至少一次的Minor GC（但并不是绝对的）。Major GC的速度一般要比Minor GC慢上10倍以上 。

GC作用域：方法区和堆区，大多数情况下发生在堆区。

常见的垃圾回收算法：

- **引用计数法** ：JVM不推荐使用这种实现方式。缺点是：

1. 每次对对象赋值时均要维护引用计数器，且计数器本身也有一定的消耗。

2. 较难处理循环引用的问题。

- **复制算法：**用于新生代，**产生MinorGC的过程（复制 -> 清空 -> 互换）**

过程：

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95%E6%B5%81%E7%A8%8B.png>)

缺点：

1. 它浪费了一半的内存，这太要命了。 
2. 如果对象的存活率很高，我们可以极端一点，假设是100%存活，那么我们需要将所有对象都复制一遍，并将所有引用地址重置一遍。复制这一工作所花费的时间，在对象存活率达到一定程度时，将会变的不可忽视。 所以从以上描述不难看出，复制算法要想使用，最起码对象的存活率要非常低才行，而且最重要的是，我们必须要克服50%内存的浪费。

- **标记清除法：**老年代一般是由标记清除或者是标记清除与标记整理的混合实现。

过程：

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4%E6%B3%95%E6%B5%81%E7%A8%8B.png>)

用通俗的话解释一下标记清除算法，就是当程序运行期间，若可以使用的内存被耗尽的时候，GC线程就会被触发并将程序暂停，随后将要回收的对象标记一遍，最终统一回收这些对象，完成标记清理工作接下来便让应用程序恢复运行。

主要进行两项工作，第一项则是标记，第二项则是清除。  
  标记：从引用根节点开始标记遍历所有的GC Roots， 先标记出要回收的对象。
  清除：遍历整个堆，把标记的对象清除。 

缺点：此算法需要暂停整个应用，会产生内存碎片 。

1. 首先，它的缺点就是效率比较低（递归与全堆对象遍历），而且在进行GC的时候，需要停止应用程序，这会导致用户体验非常差劲
2. 其次，主要的缺点则是这种方式清理出来的空闲内存是不连续的，这点不难理解，我们的死亡对象都是随即的出现在内存的各个角落的，现在把它们清除之后，内存的布局自然会乱七八糟。而为了应付这一点，JVM就不得不维持一个内存的空闲列表，这又是一种开销。而且在分配数组对象的时候，寻找连续的内存空间会不太好找。 

- **标记压缩法: **老年代一般是由标记清除或者是标记清除与标记整理的混合实现。

  ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E6%B3%95.png)

在整理压缩阶段，不再对标记的对像做回收，而是通过所有存活对像都向一端移动，然后直接清除边界以外的内存。
可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。 标记/整理算法不仅可以弥补标记/清除算法当中，内存区域分散的缺点，也消除了复制算法当中，内存减半的高额代价。

缺点：

标记/整理算法唯一的缺点就是效率也不高，不仅要标记所有存活对象，还要整理所有存活对象的引用地址。
从效率上来说，标记/整理算法要低于复制算法。

**整体对比：**

- 内存效率：复制算法>标记清除算法>标记整理算法（此处的效率只是简单的对比时间复杂度，实际情况不一定如此）。 
- 内存整齐度：复制算法=标记整理算法>标记清除算法。 
- 内存利用率：标记整理算法=标记清除算法>复制算法。 

### GC Roots

要进行垃圾回收，需要判断一个对象是否可以被回收，JVM是是通过枚举根节点做可达性分析来实现这个过程的。

**Java可以做GC ROOTS的对象：**

- 虚拟机栈(栈帧中的局部变量区,也叫做局部变量表；
- 方法区中的类静态属性引用的对象；
- 方法区中常量引用的对象；
- 本地方法栈中N( Native方法)引用的对象。

### 四大引用 - 强、软、弱、虚

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%9B%9B%E5%A4%A7%E5%BC%95%E7%94%A8%E6%9E%B6%E6%9E%84.png)

**强引用（Strong Reference）：**

Java中默认声明的就是强引用，比如：

```java
public class StrongRef {
    public static void main(String[] args) {

        Object obj = new Object(); //强引用赋值,只要obj还指向Object对象，Object对象就不会被回收
        Object obj1 = obj; //引用赋值
        obj = null; //置空

        System.gc();
        System.out.println(obj1);
    }
}
```

只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足时，JVM也会直接抛出OutOfMemoryError，不会去回收。如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了。

**软引用（Soft Reference）：**

软引用是用来描述一些非必需但仍有用的对象。**在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常**。这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等。在 JDK1.2 之后，用java.lang.ref.SoftReference类来表示软引用。

```java
/**
 * 内存够用就保留，不够就回收
 */
public class SoftRef {
    public static void main(String[] args) {

        Object obj = new Object();
        SoftReference<Object> softReference = new SoftReference(obj);
        System.out.println(obj);
        System.out.println(softReference.get());

        System.gc();
        System.out.println(obj);
        System.out.println(softReference.get());
    }
}
```

**弱引用（Weak Reference）：**

弱引用的引用强度比软引用要更弱一些，**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收**。在 JDK1.2 之后，用 java.lang.ref.WeakReference 来表示弱引用。

**虚引用（）：**

虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它**随时可能会被回收**，在 JDK1.2 之后，用 PhantomReference 类来表示，通过查看这个类的源码，发现它只有一个构造函数和一个 get() 方法，而且它的 get() 方法**仅仅是返回一个null**，也就是说将永远无法通过虚引用来获取对象，**虚引用必须要和 ReferenceQueue 引用队列一起使用**。

**引用队列（ReferenceQueue）：**

引用队列可以与软引用、弱引用以及虚引用一起配合使用，当垃圾回收器准备回收一个对象时，如果发现它还有引用，那么就会在回收对象之前，把这个引用加入到与之关联的引用队列中去。程序可以通过判断引用队列中是否已经加入了引用，来判断被引用的对象是否将要被垃圾回收，这样就可以在对象被回收之前采取一些必要的措施。

与软引用、弱引用不同，虚引用必须和引用队列一起使用。

### GC收集器

GC算法（引用计数/复制/标清/标整）是内存回收的方法论，垃圾收集器就是算法落地实现。

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/JDK8%E6%94%B6%E9%9B%86%E5%99%A8%E6%A6%82%E8%BF%B0.png)

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/JDK8%E6%94%B6%E9%9B%86%E5%99%A8%E4%BD%BF%E7%94%A8.png)

**四种主要的垃圾收集器：**

串行垃圾回收器（Serial）：它为单线程环境设计并且只使用一个线程进行垃圾回收，会暂停所有的用户线程。所以不适合服务器环境。

并行垃圾回收器（Parallel）：多个垃圾回收线程并行工作，此时用户线程是暂停的，适用于科学计算/大数据处理等弱交互场景。

并发垃圾回收器（CMS）：用户线程和垃圾收集线程同时执行（不一定是并行，可能交替执行），不需要停顿用户线程
互联网公司多用它，适用于对响应时间有要求的场景。

G1垃圾回收器：G1垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收。

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8%E5%AF%B9%E6%AF%94%E5%9B%BE.jpg)

查看当前使用的垃圾收集器：

java -XX:+PrintCommandLineFlags -version

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E6%9F%A5%E7%9C%8B%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0.png)

默认的垃圾收集器有哪些:

![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%B8%B8%E7%94%A8%E7%9A%84%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.png)

GC日志中部分参数解释:

| 参数       | 解释说明                |
| ---------- | ----------------------- |
| DefNew     | Default New Generation  |
| Tenured    | Old                     |
| ParNew     | Parallel New Generation |
| PSYoungGen | Parallel Scavenge       |
| ParOldGen  | Parallel Old Generation |

新生代：

1. 串行GC（Serial）/（Serial Coping）

   Serial收集器是一款非常古老的收集器，它使用单线程串行方式回收年轻代，会产生STW（Stop The World，即停止所有用户线程，只有GC线程在运行）。

   ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/Serial%E6%94%B6%E9%9B%86%E5%99%A8%E8%BF%87%E7%A8%8B.jpg)

   每次进行GC时，首先停顿所有的用户线程，然后只有一个GC线程回收年轻代中的死亡对象。在**Java Client模式中，默认仍然使用Serial**，因为Client模式主要针对桌面应用，一般内存较小，在百M范围内，使用单线程收集Serial效率非常高，可以带来很少时间的停顿，用户体验非常好。

   对应JVM参数是：**-XX:+UseSerialGC**

   开启后会使用**Serial(年轻代) + Serial Old（老年代）**的收集器组合。

   **新生代使用复制算法，老年代使用标记-整理算法**。

2. 并行GC(ParNew)

   ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/ParNew%E6%94%B6%E9%9B%86%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

   ①：新生代使用ParNew收集器，可以看到有多条GC线程在进行垃圾回收，采用复制算法，会暂停其他用户线程（STW）专心做垃圾回收。 
   ②：老年代使用Serial Old收集器，采用标记整理算法，会发生STW。

   对应JVM参数：**-XX:+UseParNewGC**

   开启后会使用**ParNew （新生代）+ Serial Old（老年代）**的收集器组合。

   **新生代使用复制算法，老年代使用标记-整理算法**。

   但是，ParNew + Tenured这样的搭配在Java8中已经不再被推荐。

3. 并行回收GC(Parallel)/(Parallel Scavenge)

   ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/Parallel%E6%94%B6%E9%9B%86%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

   Parallel Scavenge 收集器也是新生代收集器，也是使用复制算法的多线程收集器。 
   看上去和ParNew收集器差不多，但是Parallel Scavenge最大的特点是更关注`吞吐量`。 
   吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值：

   > 吞吐量 = 运行用户代码时间 / (运行用户代码时间) + 垃圾收集时间

   打个比方，虚拟机运行了100分钟，垃圾回收用了2分钟，那么吞吐量就是98%。 
   按照公式来看，吞吐量越高的虚拟机，自然是垃圾收集时间也越短，理所当然的用户体验也要更好。`Parallel Scavenge收集器会根据当前系统的运行情况，动态调整某些参数来提供最合适的停顿时间或最大的吞吐量`，这就是GC的**自适应调节策略**，这也是其与ParNew收集器最明显的区别。

   对应JVM参数：**-XX:+UseParallelGC 或 -XX:+UseParallelOldGC**来使用Parallel Scavenge 收集器。

   开启后会使用**Parallel Scavenge（新生代）+ Parallel Old（老年代）**的收集器组合。

   **新生代使用复制算法，老年代使用标记-整理算法**。

老年代：

1. 串行回收GC(Serial Old)/(Serial MSC)

   Serial Old是Serial的垃圾收集器老年代版本，同样是单线程的垃圾收集器，使用标记-整理算法，这个收集器也主要是运行在Client得JVM默认的老年代垃圾收集器，在java8不能显式的配置。

   在Server模式下，主要有两个用途（JDK版本在8及以后）：

   1. 在JDK1.5之前版本中与新生代的Parallel Scavenge 收集器搭配使用。（Parallel Scavenge  + Serial Pld）

   2. 作为老年代中使用CMS收集器的后备处理方案。

      

2. 并行GC(Parallel Old)/(Parallel MSC)

   是Parallel Scavenge收集器的老年代版本，用于老年代的垃圾回收，但与Parallel Scavenge不同的是，它使用的是“标记-整理算法”。适用于注重于吞吐量及CPU资源敏感的场合。

   使用方式：**-XX:+UseParallelOldGC**，打开该收集器后，将使用**Parallel Scavenge（年轻代）+Parallel Old（老年代）的组合**进行GC。

3. **并发标记清除GC(CMS)**

   ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/CMS%E6%94%B6%E9%9B%86%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

   

   对应JVM参数：**-XX:+UseConcMarkSweepGC 开启该参数后会自动执行 -XX:+UseParNewGC**

   开启后会使用**ParNew（新生代） + CMS（老年代） + Serial Old的收集器组合，Serial Old将作为CMS出错的后备收集器**。
   
   
   
   **工作流程：**
   
   ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/CMS%E6%94%B6%E9%9B%86%E5%99%A8%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png)
   
   
   
   - 初始标记(CMS initial mark)
   
   - 并发标记(CMS concurrent mark)和用户线程一起：进行GC Roots跟踪的过程，和用户线程一起工作，不需要暂停工作线程。
   
   - 重新标记(CMS remark)：为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程。由于并发标记时，用户线程依然运行，因此在正式清理前再做修正。
   
   - 并发清除(CMS concurrent sweep)和用户线程一起：为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正。
   
   优点：
   
   ​	**并发收集低停顿**。
   
   缺点：
   
   ​	并发执行，对CPU资源压力大。由于并发运行，CMS在收集与应用线程同时进行时会增加对堆内存的占用，也就是说，CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，Serial Old收集器将会以STW的方式进行一次GC，从而造成较大停顿时间。
   
   ​	采用的标记-清除算法会导致大量内存碎片。标记清楚算法无法整理空间碎片，老年代空间会随着应用时长被逐渐耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS也提供了参数-XX:CMSFullGCsBeforeCompaction（默认0，即每次都进行内存整理）来指定多少次CMS收集之后，进行一次压缩的Full GC。
   
   
   
   ### 垃圾收集器应用场景分析
   
   ![](https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/JVM/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8%E5%8F%82%E6%95%B0%E5%AF%B9%E6%AF%94.png)

- 单CPU或者小内存、单机程序：

  **-XX:+UseSerialGC**

- 多CPU，需要大吞吐量，如后台计算机应用

  **-XX:+UseParallelGC 或者 -XX:+UseParallelOldGC**

- 多CPU，追求低停顿时间，需快速响应如互联网应用

  **-XX:+UseConcMarkSweepGC 或者 -XX:+UseParNewGC**

