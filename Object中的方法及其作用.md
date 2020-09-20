# Object中的方法及其作用

## 一、引言

Object类是Java所有类的**始祖**，在Java中**每个类都扩展了Object类**。

## 二、Object类的方法详解

类中包含有：`registerNatives()、getClass()、hashCode()、equals()、clone()、toString()、notify()、notifyAll()、wait(long)、wait(long,int)、wait()、finalize()`共十二个方法。

### 1.1 registerNatives()

这个方法的作用是**注册该类所包含的除本方法以外的native方法**，以便当程序有调用到native方法时，JVM可以找到相应的**本地**方法进行调用。	

````java
private static native void registerNatives();
static {
    registerNatives();
}
````



### 1.2 getClass()

`public final native Class<?> getClass();`

在类加载的第一阶段就是将.class文件加载到内存中，并生成一个`java.lang.Class`对象的过程。而**getClass()**的作用就是获取这个对象，获取的是当前类在运行时的所有信息。

而对于getClass和.class的区别：

getClass方法，有多态能力，运行时可以返回子类的类型信息，.class是没有多态的，是静态解析的，编译时可以确定类型信息。



### 1.3 hashCode()

`public native int hashCode();`子类可以重写该方法，该方法返回的是当前对象的hashCode的值。这个值是一个整数范围内（-2^31~2^31-1）的数。

对于hashCode有以下几点约束：

* 如果两个对象x.equals(y) 方法返回true，则x、y这两个对象的hashCode必须相等。
* 如果两个两个对象x.equals(y) 方法返回false，则x、y这两个对象的hashCode可以相等也可以不等。

一般重写hashCode的时候，都会选择**31**作为基础乘数，一是因为**result * 31 = (result<<5) - result**。JVM底层可以自动做优化为位运算，效率很高。而是因为还有因为31计算的hashCode冲突较少。

```
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```



### 1.4 equals()

用于比较当前对象与目标对象是否相等，默认比较的是两个对象的地址。此方法可以被重写。

```
public boolean equals(Object obj) {
    return (this == obj);
}
```

为什么要重写equals方法？

  这是因为如果不重写equals方法，将自定义的对象放到map或者set中时，若果这两个对象的hashcode相同，就会调用equals方法，而默认的方法只是比较两个对象是否指向了同一个对象，显然大多数时候都不会指向，这样就会将重复对象存入map或者set中。这就 破坏了map与set不能存储重复对象的特性。

重写equals方法的几条约定：

1. 自反性：即x.equals(x)返回true，x不为null
2. 对称性：即x.equals(y)与y.equals(x）的结果相同，x与y不为null
3. 传递性：即x.equals(y)结果为true, y.equals(z)结果为true，则x.equals(z)结果也必须为true
4. 一致性：即x.equals(y)返回true或false，在未更改equals方法使用的参数条件下，多次调用返回的结果也必须一致。x与y不为null。

  **建议equals及hashCode两个方法，需要重写时，两个都要重写，一般都是将自定义对象放至Set中，或者Map中的key时，需要重写这两个方法。**

### 1.5  clone()

```
protected native Object clone() throws CloneNotSupportedException;
```

该方法返回的是当前对象的一个副本，支持子类重写，如果子类重写该方法，**必须实现Cloneable接口**，如果没有实现当前接口当调用object.clone()方法，**会抛出CloneNotSupportedException**。

```
@Override
protected CloneTest clone() throws CloneNotSupportedException {
    return (CloneTest) super.clone();
}

public static void main(String[] args) throws CloneNotSupportedException {

    CloneTest wang = new CloneTest("小王", 23);
    CloneTest li = wang.clone();
    System.out.println(wang == li);
    System.out.println(wang.getAge() == li.getAge());
    System.out.println(wang.getName() == li.getName());
}

// 输出结果
false
true
true
```

**上述输出可以看出，虽然克隆的对象是一个新对象，但是原对象与clone的对象的String类型 的name却是同一个引用，这表明，super.clone方法对成员变量如果是引用类型，进行是浅拷贝。**

**浅拷贝**：被复制对象的所有变量都含有和原对象相同的值，而所有对其他对象的引用仍然指向原来的对象。

**深拷贝**：深拷贝是一个整个独立对象的拷贝，深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。**简而言之就是，深拷贝就是把要复制对象所引用的对象都复制了一遍。**

**要想实现深拷贝，如果要拷贝的对象的成员变量有引用类型，则成员变量也要实现Cloneable接口，重写clone方法。**



### 1.6 toString()

```
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

  这是一个public方法，子类可重写， 建议所有子类都重写toString方法，默认的toString方法，只是将**当前类的全限定性类名+@+十六进制的hashCode值**。

  为什么要重写toString方法，可以根据需要返回当前对象的字符串信息，方便调试。



### 1.7 wait()/ wait(long)/ waite(long,int)

  这三个方法是用来 **线程间通信用** 的，作用是 **阻塞当前线程** ，等待其他线程调用**notify()/notifyAll()**方法将其唤醒。这些方法都是public final的，不可被重写。

注意：**此方法只能在当前线程获取到对象的锁监视器之后才能调用，否则会抛出IllegalMonitorStateException异常。**



### 1.8 notify()/notifyAll()

   如果当前线程获得了当前对象锁，调用wait方法，将锁释放并阻塞；这时另一个线程获取到了此对象锁，并调用此对象的notify()/notifyAll()方法将之前的线程唤醒。 这些方法都是public final的，不可被重写。

### 1.9 finalize()

```
protected void finalize() throws Throwable { }
```

