### abstract class 和 interface 有什么区别?

声明方法的存在而不去实现它的类叫抽象类。

- 不能创建抽象类的实例；
- 可以创建一个变量，其类型是一个抽象类，并让他指向具体子类的一个实例；
- 不能有抽象构造函数或抽象静态方法。 

接口是抽象类的变体。

- 接口中所有方法都是抽象的；
- 多继承性可通过实现这样的接口而获得；
- 接口只可以定义 static final 成员变量。

### **switch 是否能作用在byte 上，是否能作用在long 上，是否能作用在String上？**

在Java 5以前，switch(expr)中，expr只能是byte、short、char、int。从Java 5开始，Java中引入了枚举类型，expr也可以是enum类型，从Java 7开始，expr还可以是字符串（String），但是长整型（long）在目前所有的版本中都是不可以的。

### String、StringBuilder、StringBuffer的区别

- String 是不可变的对象，每次对 String 类型进行改变的时候其实是产生了一个新的 String 对象，然后指针指向新的 String 对象； 
- StringBuffer 是线程安全的可变字符序列，需要同步，则使用。 
- StringBuilder 线程不安全，速度更快，单线程使用。 

String 是一个类，但却是不可变的，所以 String 创建的算是一个字符串常量， StringBuffer 和 StringBuilder 都是可变的。所以每次修改 String 对象的值都是新建一个对象再指向这个对象。而使用 StringBuffer 则是对 StringBuffer 对象本身进行 操作。所以在字符串经常改变的情况下，使用 StringBuffer 要快得多。

### Servlet和Filter

##### 过程：

```
Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。
```

##### Filter有如下几个用处： 

```
	Filter可以进行对特定的url请求和响应做预处理和后处理。 在HttpServletRequest到达Servlet之前，拦截客户的 HttpServletRequest。 根据需要检查HttpServletRequest，也可以修改HttpServletRequest头和数据。 在HttpServletResponse到达客户端之前，拦截HttpServletResponse。 根据需要检查HttpServletResponse，也可以修改HttpServletResponse头和数据。 
	实际上Filter和Servlet极其相似，区别只是Filter不能直接对用户生成响应。实际上Filter里doFilter()方法里的代码就是从多个Servlet的service()方法里 抽取的通用代码，通过使用Filter可以实现更好的复用。 
```

##### Filter和Servlet的生命周期：

1. Filter在web服务器启动时初始化 
2. 如果某个Servlet配置了 <load-on-startup >1 </load-on-startup >，该Servlet也是在 Tomcat（Servlet容器）启动时初始化。 
3. 如果Servlet没有配置<load-on-startup >1 </load-on-startup >，该Servlet不会在Tomcat启动时初始化，而是在请求到来时初始化。 
4. 每次请求， Request都会被初始化，响应请求后，请求被销毁。 
5. Servlet初始化后，将不会随着请求的结束而注销。 
6. 关闭Tomcat时，Servlet、Filter依次被注销。

### **抽象的（abstract）方法是否可同时是静态的（static）,是否可同时是本地方法（native），是否可同时被synchronized修饰？** 

都不能。抽象方法需要子类重写，而静态的方法是无法被重写的，因此二者是矛盾的。本地方法是由本地代码（如C代码）实现的方法，而抽象方法是没有实现的，也是矛盾的。synchronized和方法的实现细节有关，抽象方法不涉及实现细节，因此也是相互矛盾的。

### **如何实现对象克隆？**

有两种方式： 
  1). 实现Cloneable接口并重写Object类中的clone()方法； 
  2). 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆，代码如下。

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class MyUtil {

    private MyUtil() {
        throw new AssertionError();
    }

    @SuppressWarnings("unchecked")
    public static <T extends Serializable> T clone(T obj) throws Exception {
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bout);
        oos.writeObject(obj);

        ByteArrayInputStream bin = new ByteArrayInputStream(bout.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bin);
        return (T) ois.readObject();

        // 说明：调用ByteArrayInputStream或ByteArrayOutputStream对象的close方法没有任何意义
        // 这两个基于内存的流只要垃圾回收器清理对象就能够释放资源，这一点不同于对外部资源（如文件流）的释放
    }
}
```



### **Java**的四种引用，强弱软虚，以及用到的场景 

a.利用软引用和弱引用解决OOM问题：用一个HashMap来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，从而有效地避免了OOM的问题。 

b.通过软可及对象重获方法实现Java对象的高速缓存:比如我们创建了一Employee的类，如果每次需要查询一个雇员的信息。哪怕是几秒中之前刚刚查询过的，都要重新构建一个实例，这是需要消耗很多时间的。我们可以通过软引用和 HashMap 的结合，先是保存引用方面：以软引用的方式对一个Employee对象的实例进行引用并保存该引用到HashMap 上，key 为此雇员的 id，value为这个对象的软引用，另一方面是取出引用，缓存中是否有该Employee实例的软引用，如果有，从软引用中取得。如果没有软引用，或者从软引用中得到的实例是null，重新构建一个实例，并保存对这个新建实 例的软引用。 

c.强引用：如果一个对象具有强引用，它就不会被垃圾回收器回收。即使当前内存空间不足，JVM也不会回收它，而是抛出 OutOfMemoryError 错误，使程序异常终止。如果想中断强引用和某个对象之间的关联，可以显式地将引用赋值为null，这样一来的话，JVM在合适的时间就会回收该对象。 

d.软引用：在使用软引用时，如果内存的空间足够，软引用就能继续被使用，而不会被垃圾回收器回收，只有在内存不足时，软引用才会被垃圾回收器回收。 

e.弱引用：具有弱引用的对象拥有的生命周期更短暂。因为当 JVM 进行垃圾回收，一旦发现弱引用对象，无论当前内存空间是否充足，都会将弱引用回收。不过由于垃圾回收器是一个优先级较低的线程，所以并不一定能迅速发现弱引用对象。 

f.虚引用：顾名思义，就是形同虚设，如果一个对象仅持有虚引用，那么它相当于没有引用，在任何时候都可能被垃圾回收器回收。

### **JAVA**多态的实现原理 

a.抽象的来讲，多态的意思就是同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用） 

b.实现的原理是动态绑定，程序调用的方法在运行期才动态绑定，追溯源码可以发现，JVM 通过参数的自动转型来找到合适的办法。 

### **Hashcode**的作用，与 **equal** 有什么区别？ 

同样用于鉴定2个对象是否相等的，java集合中有 list 和 set 两类，其中 set不允许元素重复实现，那个这个不允许重复实现的方法，如果用 equal 去比较的话，如果存在1000个元素，你 new 一个新的元素出来，需要去调用1000次 equal 去逐个和他们比较是否是同一个对象，这样会大大降低效率。hashcode实际上是返回对象的存储地址，如果这个位置上没有元素，就把元素直接存储在上面，如果这个位置上已经存在元素，这个时候才去调用equal方法与新元素进行比较，相同的话就不存了，散列到其他地址上。

### 数据连接池的工作机制

J2EE 服务器启动时会建立一定数量的池连接，并一直维持不少于此数目的池连接。客户端程序需要连接时，池驱动程序会返回一个未使用的池连接并将其表记为忙。如果当前没有空闲连接，池驱动程序就新建一定数量的连接，新建连接的数量有配置参数决定。当使用的池连接调用完成后，池驱动程序将此连接表记为空闲，其他调用就可以使用这个连接。

### JDBC中的 **PreparedStatement** 、 Statement、CallableStatement 

预编译语句java.sql.PreparedStatement ,扩展自Statement,不但具有Statement的所有能 力而且具有更强大的功能。不同的是，PreparedStatement 是在创建语句对象的同时给出要 执行的sql 语句。这样，sql 语句就会被系统进行预编译，执行的速度会有所增加，尤其是在执行大语句的时候，效果更加理想。CallableStatement 执行存储过程，预编译的，带参数的。

### **ClassLoader** **的分类及加载顺序**

主要分4类，见下图：

- JVM类（根）加载器（BootStrap）：这个模式会加载JAVA_HOME/lib下的jar包 ；
- 扩展类加载器（Extension）：会加载JAVA_HOME/lib/ext下的jar包 ；
- 系统类加载器（System）：这个会去加载指定了classpath参数指定的jar 文件 ；
- 用户自定义类加载器：sun提供的ClassLoader是可以被继承 的，允许用户自己实现类加载器 。

类加载器的加载顺序如图所示：

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/ClassLoader加载顺序.png?dkey=cb6a844e-88fb-4161-aa26-21943adfc44f>)

JVM并不是把所有的类一次性全部加载到JVM中的，也不是每次用到一个类的时候都去查找，对于JVM级别的类加载器在启 动时就会把默认的JAVA_HOME/lib里的class文件加载到JVM 中，因为这些是系统常用的类，对于其他的第三方类，则采用用到时就去找，找到了就缓存起来的，下次再用到这个类的时 候就可以直接用缓存起来的类对象了，ClassLoader之间也是有父子关系的，每个ClassLoader都有一个父ClassLoader,在加载类 时ClassLoader与其父ClassLoader的查找顺序如下图所示：

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/ClassLoader查找顺序.png?dkey=04793c11-9a89-4422-9694-5a84545d7d68>)

### TCP的优势

从传输数据来讲，TCP/UDP以及其他协议都可以完成数据的传输，从一端传输到另外一端，TCP比较出众的一点就是提供一个**可靠** 的，**流控**的数据传输，所以实现起来要比其他协议复杂的多，先来看下这两个修饰词的意义： 

1. Reliability ，提供TCP的可靠性，TCP的传输要保证数据能够准确到达目的地，如果不能，需要能检测出来并且重新发送数据。 

2. Data Flow Control，提供TCP的**流控**特性，管理发送数据的速 率，不要超过设备的承载能力 。

为了能够实现以上2点，TCP实现了很多细节的功能来保证数据传输，比如说 滑动窗口适应系统，超时重传机制，累计ACK等。

### 三次握手、四次挥手

##### 三次握手：

在基于 TCP 通信中，双方要进行通信，则需要建立一个物理连接，建立 时需要双方进行三次握手，成功即可完成连接建立。 

**原因**： 

```
在网络通信中，网络存在拥塞，发送的报文可能会由于网络拥塞的原因，导致对方收不到。若采用直接开启连接，当客户端发送连接建立请求后，不等待确认服务器可以打开连接就直接打开连接，这样如果服务器收不到报文，根本不知道客户端，那么客户端的打开的物理连接是无效的，但客户端不知道， 还一直发送数据，做无用的工作。
```

**过程：**

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/TCP三次握手.png?dkey=613e1c67-4ada-4b83-a838-84538a542a72>)

##### 四次挥手：

当双方通信结束时，需要四次挥手来关闭连接。

**原因：**

```
TCP 连接是双向的，一个是从客户端到服务端，另一个是从服务端到客户端。假设当前客户端已经发送完所有数据到服务 器，则此时可以告知服务器，我已经发送完数据了，可以关闭我这端到另一端的通道，服务器收到关闭报文则可发送一个确认，确认关闭；但此时由于服务器可能还需要发送数据到客户端，因此并不会关闭从服务端到客户端方向的通道；等服务端发完了，才发送一个 FIN 报文给客户端，客户端收到之后发送确认，则此时 TCP 连接才正式关闭。
```

**过程：**

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/TCP四次挥手.png?dkey=6f17d439-7e77-43c6-8a97-611cb08c2ee0>)

**TIME_WAIT 状态：**

```
从四次挥手过程可以看到，当服务器像客户端发送 FIN 报文后，客户端响应确认报文时，客户端处于 TIME_WAIT 状态，而不是处于 CLOSE 状态。之所以会这样主要是因为客户端发送确认报文后，不能立刻关闭连接。因为如果服务端收不到确认报文，会将 FIN 报文重传，但此时客户端已经关闭连接了，这样会导致客户端收不到，而服务端则一直苦苦等待客户端发送确认报文，不断重传 FIN 报文。因此客户端在响应确认报文后，需要等待两个报文往返时间，以此来确保服务端能够正常收到确认报文关闭连接。
```

### HTTPS是如何解决通信安全问题的？

HTTPS 要使客户端与服务器端的通信过程得到安全保证，必须 使用的对称加密算法，但是协商对称加密算法的过程，需要使用非对称加密算法来保证安全，然而直接使用非对称加密的过程本身也不安全，会有中间人篡改公钥的可能性，所以客户端与服务器不直接使用公钥，而是使用数字证书签发机构颁发的证书来保证非对称加密过程本身的安全。这样通过这些机制协商出一个对称加密算法，就此双方使用该算法进行加密解密。从而解决了客户端与服务 器端之间的通信安全问题。

### 一个Web请求操作的全过程

一个Http请求 DNS域名解析 --> 发起TCP的三次握手 --> 建立TCP连接后发起http请求 --> 服务器响应http请求，浏览器得到html代码 --> 浏览器解析 html代码，并请求html代码中的资源（如javascript、css、图片等） --> 浏览器对页面进行渲染呈现给用户