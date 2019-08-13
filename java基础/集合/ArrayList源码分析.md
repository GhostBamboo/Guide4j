### ArrayList源码分析

##### 一、ArrayList关系链相关

|-----Collection（interface）：顶级接口，不提供该接口的实现类，只提供该接口的子接口，提供实现该接口的集合类通用操作方法，存储一组对象；

​	|-----List（interface）：存储有序的，可重复的元素，并可通过其索引访问元素；

​		|-----ArrayList（class）：底层是数组，线程不安全；查找快，增删慢；

​		|-----LinkedList（class）：底层是双向链表，线程不安全；查找慢，增删快；

​		|-----Vector（class）：底层是数组，线程安全，效率低。JDK1.0时推出，几乎弃用。

#### 二、JDK7与JDK8中ArrayList对比

**JDK7：**

1. 初始化时，默认创建了一个容量为10的Object数组；ArrayList list = new ArrayList();
2. 如果本次添加造成数组空间不足，则触发扩容操作，并将数据复制到扩容后的新数组里。

**JDK8：**

1. 初始化时，默认创建一个**{}**，只有在执行add()时才会创建一个长度为10的数组；（tips：在JDK8中对很多类的优化都是如此，从而节省内存空间）

```java
private static final Object[] EMPTY_ELEMENTDATA = {}; //源码120行

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}; //源码127行

public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) { //指定容量立即创建一个该容量大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) { //为0时创建{}
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

public ArrayList() { //使用无参构造时直接初始化一个{}
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

public ArrayList(Collection<? extends E> c) {//通过原有集合创建，不作分析
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

2. 其他操作同JDK7，接下来详细讲述ArrayList的扩容操作。

#### 三、ArrayList源码解析

添加元素时提供了add()方法，添加元素时会对数组容量进行检查，本次添加若造成数组空间不足就会触发扩容。

```java
//不指定索引位置直接在末尾对数组进行赋值
public boolean add(E e) { 
        ensureCapacityInternal(size + 1);  
        elementData[size++] = e;
        return true;
    }

//在索引位置添加当前元素，之后的索引位置执行移位操作（该操作是导致ArrayList执行增删操作时性能下降的罪魁祸首）
public void add(int index, E element) { 
    	//判断当前索引是否超出范围，若超出抛 IndexOutOfBoundsException
        rangeCheckForAdd(index); 

    	//之前的数组容量加1进行比较看是否执行扩容操作
        ensureCapacityInternal(size + 1);  
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index); //执行移位操作
        elementData[index] = element;
        size++;
    }
```

接下来看看是如何检查容量并决定是否进行扩容的。每次给数组赋值前都会检查容量并判断是否执行扩容操作，该步骤执行了 `ensureCapacityInternal(size + 1)`，然后通过`calculateCapacity(Object[] elementData, int minCapacity)`计算出最小需要容量，最后通过`ensureExplicitCapacity(int minCapacity)`来决定是否需要扩容。

```java
private void ensureCapacityInternal(int minCapacity) { //源码230行
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

private static int calculateCapacity(Object[] elementData, int minCapacity) { //源码223行
        //数组为{}时与默认容量10做比较选择最大值作为数组需要的最小容量值，否则返回传入值（当前数组容量加1）
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) { 
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

private void ensureExplicitCapacity(int minCapacity) { //源码234行
        modCount++;

        //最小需要的容量值如果大于当前数组容量值执行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

需要扩容的时候会走到这一步，来决定扩容后容量大小，该操作执行`grow(minCapacity)`

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; 

private void grow(int minCapacity) {
    	//当前数组长度
        int oldCapacity = elementData.length; 
    	//新容量为当前数组长度的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1); 
    	//如果新容量小于最小需要容量，则赋值最小需要容量给新容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
    	//如果新的容量值大于数组最大允许长度，则对最小需要容量进行下一步判断来选择新容量
        if (newCapacity - MAX_ARRAY_SIZE > 0) 
            newCapacity = hugeCapacity(minCapacity);
        //复制原数组的元素到一个经过重重选拔出来的新容量大小的数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

 private static int hugeCapacity(int minCapacity) {
     	//超出支持的数据范围抛出OOM异常
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
     	//如果最小需要容量大于数组最大允许长度返回Integer的极限值，否则返回数组最大允许长度
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

#### 四、总结

1. ArrayList在索引位置添加元素时需要执行移位操作，这是导致ArrayList增删性能降低的根源。
2. JDK8中ArrayList在初始化后为一个空数组，第一次添加元素时才会扩容为容量为10的数组。其他情况默认扩容为原来容量的1.5倍。
3. 如果可以的话最好在初始化时就指定数组容量，从而避免不断的扩容导致性能的损耗。
4. 扩容的实质就是创建一个新数组并将原来的值复制到新数组中。

注：因为复制源码时代码跨度太大，所以部分地方标注了代码的位置。