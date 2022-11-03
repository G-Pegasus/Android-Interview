# Android 面试题整理

## 一、JVM

[JVM底层原理最全知识总结](https://doocs.github.io/jvm/)

### 1. GC Root 有哪些？

- 虚拟机栈中引用的对象：

  如下代码所示，a 是栈帧中的本地变量，当 a = null 时，由于此时 a 充当了 **GC Root** 的作用，a 与原来指向的实例 **new Test()** 断开了连接，所以对象会被回收。

  ```java
  public class Test {
      public static void main(String[] args) {
  		  Test a = new Test();
  		  a = null;
      }
  }
  ```

- 方法区中类静态属性引用的对象：

  如下代码所示，当栈帧中的本地变量 a = null 时，由于 a 原来指向的对象与 GC Root (变量 a) 断开了连接，所以 a 原来指向的对象会被回收，而由于我们给 s 赋值了变量的引用，s 在此时是类静态属性引用，充当了 GC Root 的作用，它指向的对象依然存活。

  ```java
  public class Test {
      public static Test s;
      public static  void main(String[] args) {
  		  Test a = new Test();
  		  a.s = new Test();
  		  a = null;
      }
  }
  ```

- 方法区中常量引用的对象

  如下代码所示，常量 s 指向的对象并不会因为 a 指向的对象被回收而回收

  ```java
  public class Test {
  	public static final Test s = new Test();
      public static void main(String[] args) {
  		  Test a = new Test();
  		  a = null;
      }
  }
  ```

- 本地方法栈中 JNI 引用的对象

  比如 java 调用 C++ 的方法这些。

### 2. JVM 内存模型

分为五个部分，程序计数器，Java 虚拟机栈，本地方法栈，堆，方法区。

- 程序计数器：是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。每个线程都有独立的程序计数器，用来在线程切换后能恢复到正确的执行位置。各线程之间的计数器互不影响，独立存储。线程私有。
- 虚拟机栈：线程私有的一块内存区域，它描述的是 java 方法执行的内存模型，每个方法执行的同时都会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从调用直至完成的过程都对应这一个栈帧从入栈到出栈的过程。每当一个方法执行完毕时，该栈帧就会弹出栈帧的元素作为这个方法的返回值，并且清除这个栈帧。Java 栈的栈顶的栈帧就是当前执行的方法。方法的调用过程也是由栈帧切换来产生结果。
- 本地方法栈：本地方法栈为虚拟机使用到的Native方法服务，涉及到 JNI。
- 堆：Heap是OOM故障最主要的发源地，它存储着几乎所有的实例对象，堆由垃圾收集器自动回收，堆区由各子线程共享使用；通常情况下，它占用的空间是所有内存区域中最大的，但如果无节制地创建大量对象，也容易消耗完所有的空间。
- 方法区：方法区是被所有线程共享的内存区域，用来存储已被虚拟机加载的类信息、常量、静态变量、JIT（just in time，即时编译技术）编译后的代码等数据。运行时常量池是方法区的一部分，用于存放编译期间生成的各种字面常量和符号引用。

![内存模型](https://upload-images.jianshu.io/upload_images/10006199-a4108d8fb7810a71.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 3. 垃圾回收策略及算法

### 4. 类加载机制

[学习来源](https://juejin.cn/post/6865572557329072141)

类从被加载到虚拟机内存开始，到卸载出内存为止，它的整个生命周期包括：加载，验证，准备，解析，初始化，使用，卸载。**验证、准备、解析**统称为连接。

![类生命周期](https://tva1.sinaimg.cn/large/007S8ZIlly1gi5eqpkg4cj312m0dyjvn.jpg)

- 加载：查找并加载类的二进制数据，通过一个类的全限定名来获取定义此类的二进制字节流，将这个字节流所代表的静态存储结构转化为**方法区**的运行时数据结构。在内存（**堆**）中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的访问入口。

![类加载](https://tva1.sinaimg.cn/large/007S8ZIlly1gi57jiyydxj30ya0icdiw.jpg)

- 验证：验证是连接阶段的第一步，这一阶段的目的是为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。主要有四个检验动作。

  - 文件格式验证：验证字节流是否符合 Class 文件格式的规范。
  - 元数据验证：对字节码描述的信息进行语义分析（比如说看看这个类有没有父类）
  - 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的
  - 符号引用验证：确保解析动作能正确执行

  验证阶段是很重要的，但不是必须的。可以考虑关闭一些类验证措施，以缩短虚拟机类加载时间。

- 准备：为类的**静态变量**（不包括实例变量）分配内存，并将其初始化为默认值。

  准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。（**其他详情请看原文章**）

- 解析：把类中的符号引用转换为直接引用

  **解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程**，解析动作主要针对`类`或`接口`、`字段`、`类方法`、`接口方法`、`方法类型`、`方法句柄`和`调用点`限定符7类符号引用进行。符号引用就是一组符号来描述目标，可以是任何字面量。

  `直接引用`就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

  [不懂的来这篇文章](https://cana.space/%E8%AF%A6%E8%A7%A3%E7%AC%A6%E5%8F%B7%E5%BC%95%E7%94%A8%E8%BD%AC%E7%9B%B4%E6%8E%A5%E5%BC%95%E7%94%A8/)

- 初始化：对类的静态变量，静态代码块执行初始化操作

  初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。

  **类初始化**：

  - 假如这个类还没有被加载和连接，则程序会先加载并连接该类
  - 假如该类的直接父类还没有被初始化，则先初始化其直接父类
  - 假如类中有初始化语句，则系统依次执行这些初始化语句

  **类初始化时机**：当对类主动使用的时候

  - 使用 new 关键字实例化对象
  - 调用一个类的静态方法
  - 当初始化该类时，如果其父类没有被初始化，则需要先触发其父类的初始化
  - 使用反射进行调用时，如果类型还没有被初始化，则需要先触发其初始化
  - 读取或设置一个类型的静态字段的时候

- 使用：类访问方法区内的数据结构和接口，对象是 Heap 区的数据

- 卸载：Java 虚拟机将结束生命周期的几种情况

  - 执行了 System.exit() 方法
  - 程序正常执行结束
  - 程序在执行过程中遇到了异常或错误而异常终止
  - 由于操作系统出现错误而导致 Java 虚拟机进程终止

### 5. 类加载器

虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。 实现这个动作的代码模块称为“类加载器”。

![类加载器的层次](https://tva1.sinaimg.cn/large/007S8ZIlly1gi5f30yg36j30vx0u0n9z.jpg)

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。

- 启动类加载器：这个类将器负责将存放在＜JAVA_HOME＞\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的（按照文件名识别，如rt.jar、tools.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。
- 扩展类加载器：这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载＜JAVA_HOME＞\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。
- 应用程序类加载器：它负责加载用户类路径（ClassPath）上所有的类库，开发者同样可以直接在代码中使用这个类加载器。如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

JVM 类加载特点：

- 全盘负责：当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。
- 缓存机制：缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效。
- 双亲委派机制：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

## 二、Java基础

### 1. synchronized 给普通方法，静态方法以及代码块加锁的区别？

- 给类中的普通方法加锁，属于对象锁，锁的是当前对象，即同一个对象可以同步访问。但是创建了两个对象对同步方法进行访问，则不能保持同步。
- 给类中的静态方法加锁，属于类锁，作用域是整个类。即使是两个不同的对象对同步方法进行访问也可以保持同步。
- 给类中的方法块加锁，这其中也分为对象锁和类锁，和上面的区别差不多。类锁的形式是

```java
synchronized(A.class) {
    ...
}
```

所有对象都可以保持同步。对象锁的形式是

```java
synchronized(this) {
    ...
}
```

对于同一个对象可以保持同步，但不同对象不能同步。

### 2. synchronized 和 volatile 的区别

[锁原理](https://segmentfault.com/a/1190000023315634)

https://zhuanlan.zhihu.com/p/55167585

### 3. 面向对象七大原则

- 单一职责原则：即一个类只负责一个功能，核心就是解耦和增强内聚性。
- 里氏替换原则：即子类不可以重写父类方法的功能，如果要增加新功能，可以新建一个方法。
- 开闭原则：当软件需要变化时，尽量通过扩展软件实体的行为来实现变化，而不是通过修改已有的代码来实现变化。
- 合成/复合原则：要尽量使用组合，尽量不要使用继承。
- 接口分离原则：每一个接口的职责单一明确，所以当我们要实现一个复杂的类时，尽量去实现多个接口，而不要把多个接口的功能放到一个接口。
- 依赖倒置原则：程序要依赖于抽象接口，不要依赖于具体实现。
- 迪米特（最少知道）原则：一个类对于其他类知道的越少越好，就是说一个对象应当对其他对象有尽可能少的了解。

### 4. 面向对象三大特性

https://zhuanlan.zhihu.com/p/27912079

## 三、设计模式

### 1. 单例模式（通常需要手撕，所以重点讲述）

使用单例模式就可以避免一个全局使用的类，频繁的创建与销毁，耗费系统资源。

[学习来源](https://segmentfault.com/a/1190000039746895)

#### 1. 懒汉式

```java
public class Singleton {
    private static Singleton uniqueInstance;
    
    private Sinleton() {}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        
        return uniqueInstance;
    }
}
```

**说明：** 先不创建实例，当第一次被调用时，再创建实例，所以被称为懒汉式。

**优点：** 延迟了实例化，如果不需要使用该类，就不会被实例化，节约了系统资源。

**缺点：** 线程不安全，多线程环境下，如果多个线程同时进入了 if (uniqueInstance == null) ，若此时还未实例化，也就是uniqueInstance == null，那么就会有多个线程执行 uniqueInstance = new Singleton(); ，就会实例化多个实例；

#### 2. 饿汉式 （线程安全）

```java
public class Singleton {
    private static Singleton uniqueInstance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return uniqueInstance;
    }
}
```

**说明：** 先不管需不需要使用这个实例，直接先实例化好实例 (饿死鬼一样，所以称为饿汉式)，然后当需要使用的时候，直接调方法就可以使用了。

**优点：** 提前实例化好了一个实例，避免了线程不安全问题的出现。

**缺点：** 直接实例化好了实例，不再延迟实例化；若系统没有使用这个实例，或者系统运行很久之后才需要使用这个实例，都会操作系统的资源浪费。

#### 3. 懒汉式（线程安全）

```java
public class Singleton {
    private static Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static synchronized Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        
        return uniqueInstance;
    }
}
```

**说明：** 实现和线程不安全的懒汉式 几乎一样，唯一不同的点是，在get方法上加了一把锁。如此一来，多个线程访问，每次只有拿到锁的的线程能够进入该方法，避免了多线程不安全问题的出现。

**优点：** 延迟实例化，节约了资源，并且是线程安全的。

**缺点：** 虽然解决了线程安全问题，但是性能降低了。因为，即使实例已经实例化了，既后续不会再出现线程安全问题了，但是锁还在，每次还是只能拿到锁的线程进入该方法，会使线程阻塞，等待时间过长。

#### 4. 双重校验锁（线程安全）

```java
public class Singleton {
    private static volatile Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        
        return uniqueInstance;
    }
}
```

**说明:** 双重检查数相当于是改进了线程安全的懒汉式。线程安全的懒汉式的缺点是性能降低了，造成的原因是因为即使实例已经实例化，依然每次都会有锁。而现在，我们将锁的位置变了，并且多加了一个检查。 也就是，先判断实例是否已经存在，若已经存在了，则不会执行判断方法内的有锁方法了。 而如果，还没有实例化的时候，多个线程进去了，也没有事，因为里面的方法有锁，只会让一个线程进入最内层方法并实例化实例。如此一来，最多最多，也就是第一次实例化的时候，会有线程阻塞的情况，后续便不会再有线程阻塞的问题。

**优点：** 延迟实例化，节约了资源；线程安全；并且相对于 线程安全的懒汉式，性能提高了。

**缺点：** volatile 关键字，对性能也有一些影响。

#### 5. 静态内部类实现（线程安全）

```java
public class Singleton {
    private Singleton() {}
    
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

**说明：** 首先，当外部类 Singleton 被加载时，静态内部类 SingletonHolder 并没有被加载进内存。当调用 getUniqueInstance() 方法时，会运行 return SingletonHolder.INSTANCE; ，触发了 SingletonHolder.INSTANCE ，此时静态内部类 SingletonHolder 才会被加载进内存，并且初始化 INSTANCE 实例，而且 JVM 会确保 INSTANCE 只被实例化一次。

**优点：** 延迟实例化，节约了资源；**且线程安全**，因为取得都是同一个 INSTANCE 实例；性能也提高了。

#### 6. 枚举类实现

```java
public enum Singleton {
    
    INSTANCE;

    //添加自己需要的操作
    public void doSomeThing() {
    
    }
}
```

**说明：** 默认枚举实例的创建就是线程安全的，且在任何情况下都是单例。

**优点：** 写法简单，线程安全，天然防止反射和反序列化调用。

### 2. 其他设计模式

[详情请看这篇文章，讲得很细](https://juejin.cn/post/6844903638503161870)

ps：我认为代理模式不是很好理解，所以专门看了很多博文才弄懂，下面这篇是我认为比较好理解的。还有，都具体学习代理模式了，建议去看一下AOP编程思想。

[代理模式详解](https://www.zhihu.com/question/20794107)

[Retrofit中的动态代理](
