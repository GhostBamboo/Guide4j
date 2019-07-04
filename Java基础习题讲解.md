## Java基础习题讲解

### 1. String的intern（）方法

```java
String s = new String("1");
s.intern();
String s2 = "1";
System.out.println(s == s2);

String s3 = new String("1") + new String("1");
s3.intern();
String s4 = "11";
System.out.println(s3 == s4);
```

jdk1.6中输出结果为 false false

　　解释：

　　　　String s = new String("1"); 在字符串常量池中创建"1"对象，在堆中创建s对象。

　　　　s.intern(); 由于字符串常量池中已经有"1"对象，因此该句并无实际意义。 s2指向字符串常量池中"1"对象。

　　　　因此s指向堆中的引用，s2指向字符串常量池中的引用，返回 false。

　　　　String s3 = new String("1") + new String("1"); 在堆中创建s3对象。

　　　　s3.intern(); 将字符串"11"复制到字符串常量池中。s4指向字符串常量池中"11"对象。

　　　　因此s3指向堆中的引用，s4指向字符串常量池中的引用，返回 false。

jdk1.7中输出结果为 false true

　　解释：

　　　　s与s2的情况与jdk1.6中一样，返回 false。

　　　　s3与s4有所不同的是s3.intern(); 先在字符串常量池中查找是否存在"11"，再从堆中查找，

　　　　然后将堆中s3的引用存储到字符串常量池中。

　　　　String s4 = "11"; 创建的时候发现字符串常量池中有了“11”（s3），然后指向s3引用的对象。

　　　　因此s3与s4的引用相同，返回 true。

### 2.**指出下面程序的运行结果。**

```java
class A {

    static {
        System.out.print("1");
    }

    public A() {
        System.out.print("2");
    }
}

class B extends A{

    static {
        System.out.print("a");
    }

    public B() {
        System.out.print("b");
    }
}

public class Hello {

    public static void main(String[] args) {
        A ab = new B();
        ab = new B();
    }

}
```

执行结果：1a2b2b。创建对象时构造器的调用顺序是：先初始化静态成员，然后调用父类构造器，再初始化非静态成员，最后调用自身构造器。

### 3.**如何递归实现字符串的反转及替换？**

```java
    public static String reverse(String originStr) {
        if(originStr == null || originStr.length() <= 1) 
            return originStr;
        return reverse(originStr.substring(1)) + originStr.charAt(0);
    }
```

