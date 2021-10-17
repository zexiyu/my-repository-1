 



# JVM的体系结构



## JVM的简单介绍

**JDK、JRE、JVM的区别**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816211946554.png" alt="image-20210816211946554" style="zoom:33%;" />

**常见的JVM**

- Hotspot

  －oracle官方："我们做实验用的JVM..."

- Jrockit

  －BEA，曾经号称世界上最快的JVM，被Oracle收购，合并于hotspot

  －hotspot深度定制版（收费）
  
- J9-IBM

- Microsoft VM

- TaobaoVM

  －hotspot深度定制版（多租户JVM，SessionBased JVM）

- LiqidVM
	－直接针对硬件
	
- azul zing
	－最新垃圾回收业界标杆

**JVM---从跨平台的语言到跨语言的平台**

一次编译，到处运行  : 当Java源代码成功编译成字节码后，如果想在不同的平台上面运行，则无须再次编译

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816213005620.png" alt="image-20210816213005620" style="zoom:50%;" />

虽然名字叫做Java Virtual Machine，Java虚拟机可不仅仅支持Java语言。任何语言只要编译成.class字节码文件，就可以被JVM识别





## JVM体系架构(Hotspo)

![image-20211001190047189](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211001190047189.png)

程序计数器、本地方法栈、虚拟机栈（灰色区域）是每个线程独有的，不存在垃圾回收。





### Native Interface本地方法接口

在内存中专门开辟的一块区域处理标记为native的c语言本地库



### Native Method Stack本地方法栈和JVM Stack虚拟机栈

栈是用于描述方法执行的内存模型。其生命周期随着线程结束而结束

二者不同的是本地方法栈服务的对象是JVM执行的native方法，而虚拟机栈服务的是JVM执行的java方法



#### Native Method Stack(本地方法栈)



#### JVM Stack(虚拟机栈)

 虚拟机栈中存的是每个方法对应的一个栈帧（Frame） ，每个方法的栈帧包含

1. 本地变量表（Local Variable Table）本地变量表就放各种具体类型的数值

2. 操作数栈（Operand Stack ）
   对于long的处理（store and load），多数虚拟机的实现都是原子的，没必要加volatile

3. 动态链接 （Dynamic Linking）

   将类属性和方法的符号引用解析为直接引用并指向常量池

4. 返回地址（return address）
   a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方

   

   

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820211007268.png" alt="image-20210820211007268" style="zoom:39%;" />

   

注：注意本地变量表和局部变量表的区别，局部变量表是这个

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820212944060.png" alt="image-20210820212944060" style="zoom:50%;" />

### Execution Engine执行引擎

用来执行字节码，对字节码进行优化，然后执行GC清理垃圾



### PC程序计数器

**用来记录每个线程中的指令执行的位置信息**，在多线程的情况下存在CPU时间片的切换和线程的调度，如果没有记录位置会导致线程执行错乱

> 虚拟机的运行，类似于这样的循环：
>
> while( not end ) {
>
> ​	取PC中的位置，找到对应位置的指令；
>
> ​	执行该指令；
>
> ​	PC ++;
>
> }



### 方法区

​				注:图中JVM的其他结构略	

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210826222249618.png" alt="image-20210826222249618" style="zoom:39%;" />

方法区在所有Java虚拟机线程之间共享，**它存储每个类的结构信息，各种class**，常量池也在其中。

**方法区是逻辑概念，实际的JVM内存中不存在方法区**。在HotSpot虚拟机中，jdk8之前方法区是永久代，jdk8及之后是元空间

**永久带使用的JVM的堆内存，元空间并不在虚拟机中而是使用本机物理内存。**

- 永久代Perm Space (JDK version<1.8)
  字符串常量位于永久代
  不会触发FGC清理
  永久代大小启动的时候指定，不能变

  类的方法的信息大小难以确定，因此给永久带的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出
  
  
  
- 元空间Meta Space (JDK version>=1.8)
  字符串常量位于堆
  会触发FGC清理
  元空间不设定的话，最大就是物理内存


元空间对比永久代最大的好处就是受限于物理内存，永久代非常容易溢出，元空间解决了这个问题

**永久代和元空间**

  1. 永久代 存放各种字节码class信息，方法和代码编译后的信息，字节码指令等

  2. 永久代必须指定大小限制 ；元数据可以设置，也可以不设置，无上限，因为受限于物理内存。

  3. 字符串常量 1.7 - 在永久代，1.8 - 在堆

  4. 方法区是逻辑上的概念 ，其实它是：1.7 - 永久代、1.8 - 元数据

     

  

**字节码常量池，运行时常量池，字符串常量池的区别详见：**https://blog.csdn.net/darkLight2017/article/details/117090705



### heap堆

**堆内存被所有线程共享**，主要存放使用**new**关键字创建的对象。所有**对象实例以及数组**都要在堆上分配。垃圾收集器根据GC算法收集堆上对象所占用的内存空间（收集的是对象占用的空间而不是对象本身）。

**堆内存逻辑上分为两部分：新生代，老年代**

 

### JVM Direct Memory

> JVM可以直接访问的内核空间的内存 (OS 管理的内存)
>
> 通过NIO实现零拷贝 ， 提高效率

思考：

> 如何证明1.7字符串常量位于PermSpace，而1.8位于Heap？
>
> 提示：结合GC， 一直创建字符串常量，观察堆，和Metaspace



# Class字节码



## 字节码文件结构



Class文件的总体结构如下： 

![image-20210816224255839](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816224255839.png)

- 魔数 （cafe babe）
- Class文件版本 
- 常量池 （字节码常量池）
- 访问标识(或标志) 
- 类索引，父类索引，接口索引集合 
- 字段表集合 
- 方法表集合 
- 属性表集合 

**魔数**

每个字节码文件的开头为 ca fe ba be 永远是这四个字节的固定格式，代表字节码文件的标识符，有了它才能被虚拟机识别



**文件版本**

用四个字节表示

前两个字节：minor version（副版本）

后两个字节：major version（主版本）

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816224638053.png" alt="image-20210816224638053" style="zoom:39%;" />

例如jdk1.8中就会用 00 00 00 34 表示



**常量池**

首先const_pool_count 用两个字节表示的是常量池中常量池的数量

例如 00 10，十六进制的10表示成十进制为16，实际上表示有15个常量。因为系统将第0项常量空出来留作后提拓展

demo：

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816231645761.png" alt="image-20210816231645761" style="zoom:39%;" />

常量池表：

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816231716559.png" alt="image-20210816231716559" style="zoom:50%;" />

**访问标识**

使用两个字节表示，用于识别一些类或者接口层次的访问信息。例如：是Class还是Interface；是否定义为 public 类型；是否定义为 abstract 类型；如果是类的话，是否被声明为 final 等。各种访问标记如下所示：

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210816231946708.png" alt="image-20210816231946708" style="zoom:50%;" />

。。。。。





## 字节码指令

常用指令

istore 操作数栈中的变量赋值

- istore_1 :将操作数栈中的变量，弹出放在索引为1的局部变量表中

iload 加载与存储

- iload_1:将局部变量表中索引为1位置上的数据压入操作数栈中。 
- iload_2:将局部变量表中索引为2位置上的数据压入操作数栈中。 

Xpush： 将单个数压入操作数栈

Xload：将赋值给局部变量表（赋值后）的变量压入操作数栈  

  **仔细体会Xpush和Xload的区别**



pop

计算指令

- mul
- sub

invoke

1. InvokeStatic

2. InvokeVirtual

3. InvokeInterface

4. InovkeSpecial
   可以直接定位，不需要多态的方法
   private 方法 ， 构造方法
   
5. InvokeDynamic
   JVM最难的指令
   lambda表达式或者反射或者其他动态语言scala kotlin，或者CGLib ASM，动态产生的class，会用到的指令
   
   

Demo1:

分析 i= 8这个指令

```java
public class Test {
    public static void main(String[] args) {
        int i = 8;
    }
}
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211001224236878.png" alt="image-20211001224236878" style="zoom:50%;" />

```
0 bipush 8        # 将8这个数压入操作数栈
2 istore_1        # 将8弹出放在下标为1的局部变量表中（将8赋值给i)
3 return
```



`int i = 8` 这个指令就是将8这个数压入操作数栈(`bipush 8`)，然后将8弹出放在下标为1的局部变量表中（`istore1`）

为什么下标为1？因为下标为0的局部变量表为this



Demo2：

```java
public class Test {
    public static void main(String[] args) {
        int i = 8;
        i= i++;
        System.out.println(i);
    }
}
```

```
 0 bipush 8			# 将8压入操作数栈
 2 istore_1			# 将8弹出放在下标为1的局部变量表中（将8赋值给i)
 3 iload_1			# 将变量i = 8再压入操作数栈
 4 iinc 1 by 1			# 将变量i的8变为9
 7 istore_1			# 将8弹出放在下标为1的局部变量表中 （此时是将变量从9又变成了8）
 8 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
11 iload_1			# 将i压入操作数栈
12 invokevirtual #3 <java/io/PrintStream.println : (I)V>
15 return
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820220742720.png" alt="image-20210820220742720" style="zoom:30%;" />

Demo3：

```java
public class Test {
    public static void main(String[] args) {
        int i = 8;
        i= ++i;
        System.out.println(i);
    }
}
```

```
 0 bipush 8			# 将8压入操作数栈
 2 istore_1			# 将8弹出放在下标为1的局部变量表中（将8赋值给i)
 3 iinc 1 by 1			# 将变量i的8变为9
 6 iload_1			# 将变量 i= 9 压入操作数栈
 7 istore_1			# 将再次压入栈的9再次赋值给变量i
 8 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
11 iload_1		# 将变量i = 9 再次压入操作数栈 
12 invokevirtual #3 <java/io/PrintStream.println : (I)V>
15 return
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820221253961.png" alt="image-20210820221253961" style="zoom:50%;" />

Demo4:

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820223634567.png" alt="image-20210820223634567" style="zoom:50%;" />

# 类加载



## 类加载器和双亲委派

ClassLoader负责将类的二进制数据加载到虚拟机中。

### 类加载器

所有的classloader都有一个顶级的父类：Classloader

类加载到内存之后，内存创建了两块内容：

- 将类loading成二进制文件放入磁盘。 
- 从磁盘中读取文件放入内存
- 在方法区中将二进制字节流代表的静态存储结构转换为方法区的运行时数据结构（class类的对象，Java类的模型），class对象指向二进制文件

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211001231925528.png" alt="image-20211001231925528" style="zoom:50%;" />

**类加载器的分类**

- BootStrapClassLoader ： 引导类加载器，负责加载一些核心的jar，例如rt.jar（runtime.jar）、charset.jar。。如果通过反射拿到的类加载器是空值的话，就代表已经拿到了这个加载器，最顶层的加载器，因为这个加载器使用C/C++语言实现的，嵌套在JVM内部

- ExtentionClassLoader ：ext扩展类加载器，负责加载扩展包，jre/lib/ext下面的jar包。Java语言编写，继承于ClassLoader

- AppClassLoader：系统类加载器，加载classpath目录下的内容，是加载我们自己写好的程序

- custonClassLoader：自定义类的加载器

  

  类加载器的范围：  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817231358415.png" alt="image-20210817231358415" style="zoom:33%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817213721428.png" alt="image-20210817213721428" style="zoom:50%;" />

### 双亲委派

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817221354447.png" alt="image-20210817221354447" style="zoom:50%;" />

一个类申请加载 --> 从自定义加载器开始自底向上从缓存检查该类是否加载（find in cache从缓存检查，缓存可以理解为类内部维护的容器),如果自底向上的过程从哪个类加载器缓存中找到类被加载，就返回。如果没找到，就自顶向下进行加载，这个类归哪个类加载器加载就返回结果，如果到最下面都没加载到，就抛出异常ClassNotFoundException



**双亲委派是子加载器向父加载器方向委派，然后父加载器向子加载器方向的 双亲委派过程**，即子到父 + 父到子的过程 



思考：为什么要搞双亲委派

1. 主要为了安全，特定的类由特定的类加载器进行加载,如果任何类加载器都能加载任何类，就会替换掉核心类库

   试想一下你自己写的java.lang.String类由自定义类加载器加载到。就完全可以肆无忌惮的搜集客户密码了

2. 避免类的重复加载，加载器进行向上委托的过程就是在缓存中检查类是否已经加载过。

   



#### 沙箱安全机制

**什么是沙箱？**

  Java安全模型的核心就是Java沙箱（sandbox），沙箱**主要限制系统资源访问**，不同级别的沙箱对这些资源访问的限制也可以不一样，所谓双亲委派的沙箱安全也就是为了让不同的类加载器加载不同的类

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

举例

创建一个自定义的java.lang.String类

```java
public class String {
    public static void main(String[] args) {
        System.out.println("hello!");
    }
}
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817233708345.png" alt="image-20210817233708345" style="zoom:50%;" />

String 默认情况下是由BootStrap启动类加载器进行加载的。假设我也自定义一个String 。会发现自定义的String 可以正常编译，但是永远无法被加载运行。

双亲委派机制可以确保Java核心类库所提供的类，不会被自定义的类所替代。 





## 类加载过程

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817203303999.png" alt="image-20210817203303999" style="zoom:0%;" />

**loading装载阶段**

- 将类loading生成二进制字节码文件存入磁盘，并从磁盘中读取放入内存。 

- 在方法区中将二进制字节流代表的静态存储结构转换为方法区的运行时数据结构（class类的对象，Java类的模型）

- class对象指向二进制文件

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211001231925528.png" alt="image-20211001231925528" style="zoom:50%;" />



LazyLoading的五种情况

- new getstatic putstatic invokestatic指令，访问final变量除外
- java.lang.reflect对类进行反射调用时
- 初始化子类的时候，父类首先初始化
- 虚拟机启动时，被执行的主类必须初始化
- 动态语言支持java.lang.invoke.MethodHandle解析的结果为REF_getstatic REF_putstatic REF_invokestatic的方法句柄时，该类必须初始化



**linking阶段**

- verification验证 : 验证文件是否符合JVM规范

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817205914589.png" alt="image-20210817205914589" style="zoom:39%;" />

- preparation准备 : 虚拟机为类的静态成员变量赋默认值。如int,short,byte赋默认值0

- resolution解析：解析运行时常量池中的符号引用为具体引用的过程

  

  ![image-20210817210845432](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210817210845432.png)

**initializing阶段**

调用类初始化代码 <clinit>，给静态成员变量赋初始值（你定义的值）

**gc垃圾回收阶段**

类的生命周期结束被垃圾回收



demo：

```java
public class T001_ClassLoadingProcedure {
    public static void main(String[] args) throws Exception {
        System.out.println(T.count);
    }
}

class T {
    public static T t = new T(); // null
    public static int count = 2; //0


    private T() {
        count++;
        System.out.println("-----" + count);
    }
}
```

运行结果

```
-----1
2
```

先将类加载到内存 --> 然后verify --> initializing将t设为null，count 设为null --->  resolution  --> prepareing 将两个静态成员变量设初始值此时new T走构造器，此时count为0，加加后变为1，然后输出，最后count又在静态成员变量中被prepareing为2，在mian方法中输出



**注：**

- 静态成员变量的初始化过程：随着类被类加载器load进内存，静态变量通过preparation被赋默认值，随后通过initializing附初始值

  load - 默认值 - 初始值

- 非静态成员变量的初始化：   随着对象的new申请内存在堆中开辟新的空间，先赋默认值，在赋初始值

  new - 申请内存 - 默认值 - 初始值



#### 双亲委派的打破


重写loadClass（）



## 类加载器源码

```java
// 从内存中找类是否已经加载到内存	
// Launcher类中
protected final Class<?> findLoadedClass(String name) {
        if (!checkName(name))
            return null;
        return findLoadedClass0(name);
    }

private native final Class<?> findLoadedClass0(String name);
// ============================================================================================

// ClassLoader类中
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded 先从内存中找
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order 如果最终仍未找到调用findClass()
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);  

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);  // 加载类
            }
            return c;
        }
    }	
// ============================================================================================
 	protected final void resolveClass(Class<?> c) {
        resolveClass0(c);
    }

    private native void resolveClass0(Class<?> c);
```



**类加载器源码都在Launcher.class中**

通过ClassLoader的源码发现加载类的步骤为：

findInCache从缓存找 -> parent.loadClass -> findClass()



**如何自定义类加载器**

1. 继承 ClassLoader
2. 重写 findClass( )方法 



**如何打破双亲委派**

重写loadClass( )

何时打破过双亲委派

1. 在JDK1.2之前，自定义ClassLoader都必须重写loadClass()
2. ThreadContextClassLoader可以实现基础类调用实现类代码，通过thread.setContextClassLoader指定
3. 热启动，热部署 例如tomcat有自己模块指定的classloader





## 编译器

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/f0618c9e262974cae3e47f6b37f16ca3-2478331.png" alt="img" style="zoom:30%;" />

Java是解释型语言，将一个类load成二进制文件后，由Java的解释器去执行，但是Java有个JIT编译器（Just In Time compiler）它可以将一部分Java代码编译成本地代码执行（native代码），因此Java即是编译型语言也是解释型语言，**默认是混合模式：解释器+ 热点代码编译**。当一段代码在单位时间重复执行超过一定次数时（多次被调用，多次被循环），JVM会将其热点代码编译成本地代码来执行（通过方法计数器，循环计数器检测）。

- Xmixed：混合模式 ，hotspot版JVM默认使用这个模式，热点代码编译，非热点代码解释执行
- Xint：纯解释模式，启动速度快，执行速度稍慢
- Xcomp：纯编译模式，启动很慢，执行速度快

一个参数 ：检测热点代码   -XX:CompileThreshold = 10000   循环或调用超过这些次JVM就会将热点代码编译成本地代码



## 对象的创建

**对象的创建过程**

 1. class loading
 2. class linking (verification, preparation, resolution) 
 3.  class initializing 
 4. 申请对象内存
 5. 成员变量赋默认值
 6. 调用构造方法＜init＞
    - 成员变量顺序赋初始值
    
    - 执行构造方法语句
    



创建一个简单对象的字节码指令

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210829002144773.png" alt="image-20210829002144773" style="zoom:50%;" />

  注：前三步静态成员变量完成初始化并赋值 ，后三步非静态成员变量完成初始化并赋值



**对象在内存中的存储布局**

普通对象

1. 对象头： 8字节

   markword :4字节

   ClassPointer指针：指向类的元数据，虚拟机通过指针确定是哪个类的实例，开启指针压缩`-xxUseCompressedClassPointers`为4字节，不开启为8字节

2. 实例数据 ：存储对象真正的有效信息。 开启指针压缩`-xxUseCompressedClassOops`为4字节，不开启为8字节

4. Padding对齐，**对象大小必须是8的倍数**，因此需要对其填充（比如对象大小为15，实际上就要填充为16）

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211007001243020.png" alt="image-20211007001243020" style="zoom:39%;" />

数组对象

1. 对象头
2. ClassPointer指针
3. 数组长度
4. 数组真实数据 
5. Padding对齐



- 一个Object对象大小为16字节 ，因为一个markword对象头8字节，一个class指针，本来8字节打开指针压缩变为4字节，后面4字节padding对齐

- int[] 16字节 markword 8字节， classpointer 4字节，长度4字节

- 一个这样的对象： 32字节

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/fcc13f1a482a376c5aa65e8805c4b91e-539514.png" alt="img" style="zoom: 25%;" />



有两个JVM参数：

- xxUseCompressedClassPointers:  指针压缩（class对象的指针，对象头下面的那个指向class的指针），默认指针8字节压缩成4字节

- xxUseCompressedClassOops ：压缩指针（引用类型成员变量的指针），默认8 压缩成4。

  **oops就是ordinary object pointers 成员变量指针，小心网上的文章是错的**

  

### Java对象头 (MarkWord)

​	对象头源码如下：（JDK1.8）

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210819215556084.png" alt="image-20210819215556084" style="zoom:50%;" />



对象头markword基本组成结构

着重说 **有2位代表锁状态的标志位，和GC标记（标记分代的年龄），对象头和对象的锁状态有关，不同的状态对象头会记录不同的数据**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210819215746970.png" alt="image-20210819215746970" style="zoom:50%;" />
为什么GC年龄默认为15： 因为对象头的分代年龄标记只有4bit位



### 对象定位

https://blog.csdn.net/clover_lily/article/details/80095580

1. 句柄池
2. 直接指针







# GC（Garbage Collector）



## GC基础知识

### Java中的垃圾和C语言中的垃圾的区别

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820224845492.png" alt="image-20210820224845492" style="zoom:50%;" />

C语言处理垃圾的劣势就是需要手动处理垃圾，这样就会导致经常出现垃圾忘记回收导致内存泄漏，或者垃圾被回收系统多次将被其他人正在使用的对象当做垃圾回收，开发效率低



### 什么是垃圾

没有任何引用指向的一个对象，或者多个对象（循环引用是一团垃圾）都是垃圾

单个垃圾：<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820225555973.png" alt="image-20210820225555973" style="zoom:39%;" />

一团垃圾：<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820225130831.png" alt="image-20210820225130831" style="zoom:50%;" />

### 如何定位垃圾

1. 引用计数（ReferenceCount）

   当引用计数为0就变成垃圾

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820225737335.png" alt="image-20210820225737335" style="zoom:39%;" />

    

   - 优点： 实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性。 

   - 缺点： 

     1. 它需要单独的字段存储计数器，这样的做法增加了存储空间的开销。

     2. 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销。

     3. **无法处理循环引用**。因此在Java的垃圾回收器中没有使用这类算法。 

        

2. 根可达算法(RootSearching)

   首先要知道什么是根对象：线程栈变量，静态变量，常量池，JNI指针（Java Native Interface指针)

   根可达算法：通过根对象的引用链一直往下搜找能往下找到的就是有用的，没有与引用链相连的对象就是垃圾

   **解决了循环引用的问题**
   
   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820230305912.png" alt="image-20210820230305912" style="zoom:50%;" />

### 常见的垃圾回收算法

1. 标记清除(mark sweep)  **缺点：位置不连续 产生碎片 **

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821113627965.png" alt="image-20210821113627965" style="zoom:50%;" />

2. 拷贝算法 (copying) - 没有碎片，但是浪费空间

   将存活的对象内存空间拷贝相同的一份，在垃圾回收时将存活的对象复制到拷贝后的内存空间，之后清除旧的内存块空间中所有对象，交换两个内存的角色，最后完成垃圾回收。 

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821113555642.png" alt="image-20210821113555642" style="zoom:50%;" />

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820232140321.png" alt="image-20210820232140321" style="zoom:50%;" />

   **适用于垃圾对象比较多的情况**，只扫描一次，便可形成没有碎片的连续空间

   缺点是浪费空间，移动复制对象，需要调整对象引用

3. 标记压缩(mark compact) - 没有碎片，效率偏低（两遍扫描，指针需要调整）

   标记的是所有存活对象。步骤：标记 ->清除 -> 压缩。比标记清除算法多了个压缩步骤

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821114234474.png" alt="image-20210821114234474" style="zoom:50%;" />

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820232632699.png" alt="image-20210820232632699" style="zoom:35%;" />

**使用建议：新生代大量死去，少量存活，采用复制算法Copying；老年代存活率高，回收较少，采用MC或MS**

### JVM内存分代模型（用于分代垃圾回收算法）

**为什么要分代：** 

- 垃圾性质不同，有的垃圾回收的较早，有的回收的较晚
- 算法不同

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210820234115940.png" alt="image-20210820234115940" style="zoom: 39%;" />

#### 部分垃圾回收器的分代模型

> 除Epsilon ZGC Shenandoah之外的GC都是使用逻辑分代模型
>
> G1是逻辑分代，物理不分代
>
> 除此之外其他垃圾收集器不仅逻辑分代，而且物理分代
>

#### 新生代 + 老年代 + 永久代（1.7）Permanent Generation/ 元数据区(1.8) Metaspace

**永久代/元空间**

1. 永久代 存放各种字节码class信息，方法和代码编译后的信息，字节码指令等
2. 永久代必须指定大小限制 ；元数据可以设置，也可以不设置，无上限，因为受限于物理内存。
3. 字符串常量 1.7 - 在永久代，1.8 - 在堆
4. 方法区是逻辑上的概念 ，其实它是：1.7 - 永久代、1.8 - 元数据

**新生代、老年代**

什么是YGC 什么是FGC?

- YGC：对新生代范围进行GC

- FGC：FULL GC全堆范围的GC，FCG触发时，年轻代和老年代分别根据自身的垃圾收集器的算法和规则进行相应的垃圾回收。FGC尽量让它少触发，一般一周一次、十天一次。

  什么时候触发FGC?老年代内存不够了就会触发FGC，JVM默认是 PS + PO 可以设置为PN + CMS
  



1. 新生代 = Eden + 2个suvivor幸存者区 （幸存者0区和幸存者1区）

   1. YGC回收之后，大多数的对象会被回收，活着的进入s0
   2. 再次YGC，活着的对象eden + s0 -> s1
   3. 再次YGC，eden + s1-> s0
   4. 年龄足够 进入 -> 老年代 （15次GC后）
   5. 幸存者区装不下 -> 老年代

    s0 - s1 之间的复制年龄超过限制时，进入old区通过参数：－XX:MaxTenuringThreshold配置

   **注：s0和s1有时会被叫成to和from，空的是to**

2. 老年代

   1. 顽固分子
   2. 老年代满了执行FGC （Full GC）

3. GC Tuning (Generation)

   1. 尽量减少FGC

   2. MinorGC = YGC

   3. MajorGC = FGC

      

4. 对象分配过程图
   ![](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/%E5%AF%B9%E8%B1%A1%E5%88%86%E9%85%8D%E8%BF%87%E7%A8%8B%E8%AF%A6%E8%A7%A3.png)

   注：次数上 **频繁对新生代GC，较少对老年代GC，对老年代而言只有FGC**

5. 动态年龄：（不重要）

   https://www.jianshu.com/p/989d3b06a49d

6. 分配担保：（不重要）
   YGC期间 在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中 (默认15) 要求的年龄。

   因为YGC使用的是复制算法清除垃圾，每次YGC后必须要有一半以上的剩余空间
   
   survivor区空间不够了 空间担保直接进入老年代
   参考：https://cloud.tencent.com/developer/article/1082730

**栈上分配内存和线程本地分配内存**

如果开启栈上内存分配的话，一些小的对象是可以分配到栈上的。每次创建对象会优先检查是否可以在栈上进行内存分配，如果可以且内存足够就分配到栈上，如果不能分配到栈上才会将对象new到Eden区，见上图



逃逸分析：逃逸分析其实就是分析java对象的动态作用域

1. 如果一个对象被定义之后，被外部对象引用，则称之为`方法逃逸`
2. 如果被其它线程所引用，则称之为`线程逃逸`

TLAB：Thread Local Allocation Buffer，即线程本地分配缓存区，这是一个线程专用的内存分配区域。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821112332310.png" alt="image-20210821112332310" style="zoom:50%;" />

栈上分配内存要比线程本地分配效率高，因为分配到栈上不需要进行垃圾回收，随着栈的弹出对象即消失



 **内存溢出和内存泄漏**

**内存溢出：**（out of memory）就是内存不够用了，使用大程序在小内存机器上运行，程序在申请内存时，没有足够的内存空间供其使用，肯定内存溢出。比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出

**内存泄漏：**（Memory Leak）是指程序在申请内存后，由于种种原因无法释放已申请的内存空间，造成系统内存的浪费。一次内存泄露危害可以忽略，但内存泄露堆积后果很严重最终会导致内存溢出

Java的内存泄露多半是因为对象存在无效的引用，得不到释放，如果发现Java应用程序占用的内存出现了泄露的迹象，那么我们一般采用下面的步骤分析：

1. 使用jps命令找到对应java进程

2. 使用Java heap分析工具（如jamp，jvisualvm，arthas），找出内存占用超出预期的嫌疑对象

3. 根据情况，分析嫌疑对象和其他对象的引用关系。

4. 分析程序的源代码，找出嫌疑对象数量过多的原因。

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210823202539167-20210826001401536.png" alt="image-20210823202539167" style="zoom:25%;" />




### 常见的垃圾回收器

随着JDK诞生出现了第一款Serial GC ，为了提高效率，诞生了PS，为了配合CMS，诞生了PN，CMS是1.4版本后期引入，CMS是里程碑式的GC，它开启了并发回收的过程，但是CMS毛病较多
并发垃圾回收是因为无法忍受STW

![image-20211002124956175](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211002124956175.png)

查看你的JDK的默认垃圾收集器

```bash
java -XX:+PrintCommandLineFlags -version
```

![image-20210821110422446](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821110422446.png)

为什么这么多垃圾收集器，因为Java 的使用场景很多，移动端，服务器等。所以我们需要针对不同的场景，选择适合的垃圾收集器，提高垃圾回收效率。 

**STW (stop the world)**

GC事件发生过程中，会产生应用程序的停顿。 **使整个应用程序的所有线程暂停**，没有任何响应。每个垃圾收集器都有STW现象，只是更优秀的垃圾收集器会减少STW的发生

GC时为什么会出现STW ？个人认为垃圾回收的过程中因为对象位置发生变化，重新分配的过程中是不允许程序正常执行的，会出异常

**串行回收和并行回收**

- 串行回收：同一时间段内只能有一个线程进行垃圾回收
- 并行回收:   同一时间段内有多个线程同时进行垃圾回收

并发回收：表示回收垃圾的过程中可以产生新的垃圾

并行回收提高了回收效率，不过还是会引发STW



#### Serial GC

Serial配合和SerialOld使用，最早期的GC版本。Serial收集年轻代，SerialOld收集老年代

Serial和SerialOld为串行回收机制 ，STW，年轻代采用复制算法，老年代采用标记整理算法

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821143039820.png" alt="image-20210821143039820" style="zoom:50%;" />

这种串行垃圾回收机制在单核cpu中效率高。现在都不是单核cpu了，因此这种垃圾回收器逐渐废弃

#### Parallel Scavenge

年轻代垃圾回收期，采用并行回收机制 ，STW，采用复制算法。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821144201643.png" alt="image-20210821144201643" style="zoom:30%;" />

默认线程数为cpu的核数

Parallel Scavenge配合Parallel old一起使用

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821145317444.png" alt="image-20210821145317444" style="zoom:50%;" />

Java8中默认使用PS + PO收集器

#### ParNew

年轻代垃圾回收期，采用并行回收机制 ，STW，采用复制算法



ParNew对比Parallel Scavenge对比

- PN响应时间快，因为配合CMS。

- PS吞吐量优先。

  PN配合CMS并发收集垃圾，会比PS+PO垃圾回收速度慢一些，垃圾回收快吞吐量大





#### CMS

全称ConcurrentMarkSweep ，老年代垃圾收集器 ，**支持并发操作，也就是说垃圾回收和应用程序同时运行**，降低STW的时间(200ms)
CMS问题比较多，会产生内存碎片，所以现在没有一个版本默认是CMS，只能手工指定
CMS使用标记清除，那就一定会有碎片化的问题，碎片到达一定程度，CMS的老年代分配对象分配不下的时候，使用SerialOld对老年代区域标记压缩

CMS只管Full GC

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210821150508236.png" alt="image-20210821150508236" style="zoom:50%;" />

- 初始标记（STW）：**找到GC roots根节点对象**。暂停时间非常短

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825102934977.png" alt="image-20210825102934977" style="zoom:30%;" />

- 并发标记（比较耗时，但是不STW）：从GC Roots开始遍历整个对象图的过程。不会停顿用户线程。该阶段最耗时。

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825103143197.png" alt="image-20210825103143197" style="zoom:33%;" />

- 重新标记：（STW）：**并发标记的同时会出现垃圾引用的改动**，该阶段对改动过的引用进行重新标记

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825103448483.png" alt="image-20210825103448483" style="zoom: 33%;" />

- 并发清理（比较耗时，但是不STW）：清理删除掉标记阶段判断的已经死亡的对象，释放内存空间（标记清除）。用户线程和GC线程同时执行

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825103515367.png" alt="image-20210825103515367" style="zoom: 33%;" />

CMS的优势：使用了并发标记**缩短了STW的时间**。经实验：10g的内存在垃圾回收的过程中会STW 11 秒，这是无法忍受的

CMS的缺点：

- 会产生大量内存碎片，当碎片过多时，年轻代的对象想要往老年代转移发现已经没有足够空间，CMS会将Serial Old这个“老奶奶”请过来进行标记压缩，且SerialOld进行标记压缩的过程是STW的，如果内存空间非常大会导致整个应用线程停止相当长的时间！因此CMS无论如何优化都解决不了STW时间相当长的问题

  

- 会产生浮动垃圾：在并发标记阶段产生的垃圾即为浮动垃圾，这些浮动垃圾会在下一次GC中被回收

  解决方案：降低触发CMS的阈值。CMS中有个参数，并不是在老年代满了才会FCG，当达到设定的阈值就会触发
  
  ```bash
   -XX:CMSInitiatingOccupancyFraction 92%    # 92%是默认的
  ```
  
  降低这个值，保证老年代足够的空间
  
  

demo：硬件升级系统反而卡顿？

有一个50万PV的资料类网站（从磁盘提取文档到内存）原服务器32位，1.5G的堆，用户反馈网站比较缓慢，因此公司决定升级，新的服务器为64位，16G的堆内存，结果用户反馈卡顿十分严重，反而比以前效率更低了，为什么？

答：FGC时STW的时间长了



![image-20210822164445476](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210822164445476.png)

注：PS和PN的区别：PS吞吐量优先，PN响应速度优先





#### G1

G1的核心思想---分而治之。G1物理不分代，逻辑分代

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825144206040.png" alt="image-20210825144206040" style="zoom:30%;" />

G1逻辑分为4个区：

- Eden区
- Survivor区
- Old区
- Humongous大对象区

**G1垃圾收集器的特点**

- 并发收集 
- 压缩空闲空间不会延长GC的暂停时间；
- 更易预测的GC暂停时间；
-  适用不需要实现很高的吞吐量的场景

**说明：**

G1上的分区只是逻辑上的分区，**每个分区都可能是年轻代也可能是老年代，是可变的，但是在同一时刻只能属于某个代**，非常灵活。在物理上不需要连续，则带来了额外的好处－有的分区内垃圾对象特别多，有的分区内垃圾对象很少，**G1会优先回收垃圾对象特别多的分区**，这样可以花费较少的时间来回收这些分区的垃圾，这也就是G1名字的由来，**即首先收集垃圾最多的分区。**

新生代其实并不是适用于这种算法的，依然是在新生代满了的时候，对整个新生代进行回收-----整个新生代中的对象，要么被回收、要么晋升，至于新生代也采取分区机制的原因，则是因这样跟老年代的策略统一，方便调整代的大小。
G1还是一种带压缩的收集器，在回收老年代的分区时，是将存活的对象从一个分区拷贝到另一个可用分区，这个拷贝的过程就实现了局部的压缩。每个分区的大小从1M到32M不等，但是都是2的冥次方。



**Card Table**

如果在年轻代中想要找到存活的对象，就要遍历老年代中的每一个对象，看是否有老年代中的某个对象作为年轻代某个对象的引用指向年轻代中的对象，说明每次GC都要遍历老年代。优化：使用Card Table标记。

由于做YGC时，需要扫描整个OLD区，效率非常低，所以JVM设计了CardTable，用cardtable将内存分为一些小块块，如果一个OLD区CardTable中有对象指向Y区，就将它设为Dirty，下次扫描时，只需要扫描老年代中的Dirty Card Table。**这样就避免了扫描整个老年代**

Dirty Card Table的标记是基于比特位，的如上图：bitmap [0,1,0,0，0,0,1,0]

因此，使用Card Table + Drity标记是为了使得年轻代的回收效率变高避免扫描整个老年代



**cardtable 和region的关系？**

一个region逻辑上可能是yong、surviver、old、Humomgous

cardtable是比region更小的逻辑单元，每个region又分为很多cardtable



 **CSet（Collection Set)**
表示一组可被回收的分区的集合，用来记录哪些card需要被回收。选择要进行回收的分区放入CSet（G1选择的标准是垃圾最多的分区优先，也就是存活对象率最低的分区优先）
CSet只占用不到整个堆空间的1％大小。



 **RSet（RememberedSet）**

- 在每一个region都有一个hashset哈希表
- 记录着其他region中的对象到本region的引用
- 价值在于使得垃圾收集器不需要扫描整个堆找到谁引用了当前分区中的对象，只需要扫描rset就行
- G1高效高速回收的关键

rset与赋值的效率

- 由于rset的存在，每次修改更新对象的引用时，都会在GC中生成一个写屏障（这个写屏障不等于内存屏障），但这点性能影响对于能避免扫描整个老年代来说微乎其微



![image-20211002220134914](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211002220134914.png)



**G1新老年代比例**

不需要手工指定，G1可以根据young区的STW时间动态调整比例 （5% ~60%）



**G1中的GC方式**

- YGC（注意：G1的YGC是STW的）

  何时触发？ Eden区空间不足，采用多线程并行回收（用户线程和垃圾回收线程并存，多个垃圾回收线程并行）

- MixedGC

  已存活的对象占用堆空间超过45%触发MixedGC（默认）。 通过参数 XX:InitiatingHeapOccupacyPercent

- FGC （标记清除，JDK10之前串行,JDK10以后并行）

  何时触发？ Old区内存空间不足 ，或者System.gc( )触发。

**使用G1和使用CMS的核心目的都是为了减少FGC的发生，尽量不要有！**





**G1垃圾回收器会产生FGC吗？如果G1产生FGC，你应该做什么？**

  1. 扩内存
  2. 提高CPU性能（回收的快，业务逻辑产生对象的速度固定，垃圾回收越快，内存空间越大）
  3. 降低MixedGC触发的阈值，让MixedGC提早发生（默认是45%）

**MixedGC的过程**

- 初始标记（STW)
- 并发标记
- 重新标记(STW)
- 筛选回收（STW + 并行）优先选择垃圾多的区域进行回收并将回收后区域存活的对象放入其他区域中，相当于压缩



##### **G1的三色标记法**

- 黑色：对象自身和成员变量均已标记完成
- 灰色：自身已被标记，成员变量未被标记
- 白色：未被标记的对象

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825154319820.png" alt="image-20210825154319820" style="zoom:20%;" />

如下图：A对象包含域B、C，B对象包含域D。

- A自身和A中的域B、C都已经被标记（黑色)。C中没有别的域，C被标记（黑色）
- B中的域D未被标记（灰色）
- D未被标记（白色）

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825155419394.png" alt="image-20210825155419394" style="zoom:39%;" />

**三色标记可能出现的问题：**

- 浮动垃圾：例如，次轮标记过后B->D的引用消失了，D就成为了浮动垃圾，只能等待下一轮的垃圾回收被清除

- 漏标：黑色对象的引用指向了白色对象，与此同时指向白色对象的其他引用没了（A多了个指向D，B指向D的引用消失）黑色已经是被标记过的， 后面不会再扫描了，因此白色对象永远也不会被扫描到。 

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825162006005.png" alt="image-20210825162006005" style="zoom:50%;" />

  

  漏标终造成的后果：非垃圾会被当成垃圾清理掉。如上图，D已经从垃圾变成了非垃圾了，但是这种情况下D永远都扫描不到，D最终会被清理掉！

  漏标发生在MixGC的并发标记阶段，因为该阶段工作线程和和GC线程同时进行，并发标记的过程其他工作线程也在改变引用的指向，并发标记的过程产生新垃圾，甚至不可达对象变成了可达对象(垃圾变成非垃圾)。所以漏标是这么来的





​	***漏标的解决方案***	

- incremental update  ：**CMS的解决方案**，增量更新，关注引用的增加。如上图：既然A多了个指向的D，那就将A标为灰色好了。

  但是此方案存在巨大的bug,如下图

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211002224553731.png" alt="image-20211002224553731" style="zoom:39%;" />

  所以CMS的remark阶段必须从头扫一遍

- SATB (snapshot at the begining) ：**G1的解决方案**，它关注引用的删除，如上图，当B->D的引用消失时，把这个引用推到GC的堆栈，保证D还能被扫描到



SATB就是将删除过的引用放入栈中，再次重新标记时，扫描栈中的引用就知道哪些引用发生了改变。

CMS用的是incremenal update，G1使用的是SATB



**为什么G1使用SATB**
灰色 ->白色 引用消失时，如果黑色指向白色的引用消失，该引用会被push到堆栈下次扫描时拿到这个引用（关注删除），由于有Rset的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高
SATB 配合Rset浑然天成！



**使用G1垃圾回收器的优缺点**

- G1 优点：

  1. 停顿时间短，追求吞吐量；
  2. 追求响应时间，用户可以通过参数对STW时间进行控制；
  3. 灵活---优先回收垃圾比例高 花费时间少的区域，年轻代和老年代比例也可以动态调整

- G1 缺点：

  G1 需要记忆集  Cset，Rset，这些数据可能达到整个堆内存容量的 20% 甚至更多。而且 G1 中维护记忆集的成本较高，带来了更高的执行负载，影响效率。







#### 了解其他版本的JVM

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210825145456482.png" alt="image-20210825145456482" style="zoom:50%;" />

# JMM -JAVA内存模型

JMM不是真实存在的，只是一个抽象的概念。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210818220938412.png" alt="image-20210818220938412" style="zoom:50%;" />

## 硬件层数据一致性

### 存储器的层次结构

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210818221119251.png" alt="image-20210818221119251" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210828214901522.png" alt="image-20210828214901522" style="zoom:30%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210818232218070.png" alt="image-20210818232218070" style="zoom:50%;" />



总线锁---只要有一个cpu在访问缓存中的数据，其他线程只能等待不能访问，相当于阻塞。这样效率低下（老的cpu使用这种办法）

而现在的cpu是使用各种协议保证数据一致的，我们常用的 intel 使用的是 ：MESI。

**现代CPU的数据一致性实现是通过缓存锁(MESI) + 总线锁**。 volitile和synchornized正是借助此协议完成数据的可见性的

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210818225940189.png" alt="image-20210818225940189" style="zoom:50%;" />



### cacheline的概念

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210818222334588.png" alt="image-20210818222334588" style="zoom:39%;" />

图中main memory相当于内存，多个cpu中每个cpu都包含单独的L1一级缓存和L2二级缓存，L3相当于cpu的三级缓存（多个cpu的共同缓存）

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210828222019300.png" alt="image-20210828222019300" style="zoom:40%;" />

**cacheline缓存行的伪共享**：读取缓存以cache line为基本单位，**缓存行大小为64bytes**。位于同一缓存行的两个不同数据，被两个不同CPU锁定。如图，两个cpu1，cpu2分别访问同一个缓存行中的相邻数据x， y ，cpu1不可能单独读取缓存行中的x，cpu2不可能单独读取缓存行中的y，这两个cpu将整个缓存行中的数据64bytes都加载进去了。此时若cpu1对的操作是x，导致这段地址上的数据对于其他cpu的缓存失效，cpu2也需要重新去主存中load这段数据，导致了额外的开销。**由于缓存行的特性，当多个cpu修改同一个缓存行中的数据会影响彼此的性能。**

伪共享带来的问题就是：**当处理器发现自己缓存行对应的内存数据被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这**个数据进行修改操作的时候，会强制重新从系统内存里把数据读取到处理器缓存里

解决伪共享办法：

使用缓存行的对齐，也就是说多个cpu修改同一个缓存行中的不同数据，那我把同一个缓存行中的不同数据设置到不同的缓存行中，这样他们就修改不同的缓存行了。省去了通知和同步，提升了效率

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210819104228923.png" alt="image-20210819104228923" style="zoom:50%;" />





## 乱序问题

cpu的执行时间远大于内存读取时间（cpu比内存至少快100倍），因此CPU为了提高指令执行效率，会在一条指令执行过程中，去同时执行另一条指令，前提是，两条指令没有依赖关系。

产生一个问题--指令重排

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210828235725759.png" alt="image-20210828235725759" style="zoom:39%;" />

乱序的存在条件：**只要不影响单线程的最终结果一致性（单线程中指令互不影响）**，单线程的指令是完全可以换顺序的

### 如何保证特定情况下不乱序



**硬件层级：内存屏障（X86）**

- storefence 写屏障:  在sfence指令前的写操作当必须在sfence指令后的写操作前完成。
- loadfence 读屏障： 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。
- modify/mix fence ：读写屏障在modify/mix fence 指令前的读写操作当必须在modify/mix fence 指令后的读写操作前完成。

> 原子指令，如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序

**JVM层级 屏障**

- LoadLoad屏障（读屏障）：对于这样的语句 `Load1;LoadLoad; Load2`， 在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

- StoreStore屏障（写屏障）：对于这样的语句 `Store1; StoreStore; Store2`，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。

- LoadStore读写屏障：对于这样的语句 `Load1; LoadStore; Store2`，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。

- StoreLoad读写屏障：对于这样的语句 `Store1; StoreLoad; Load2`， 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。



**volatile的实现细节**

1. 字节码层面
   ACC_VOLATILE

   
   
2. JVM层面
   volatile内存区的读写 都加屏障

   > StoreStore屏障
   >
   > volatile 写操作
   >
   > StoreLoad屏障

   

   > LoadLoad屏障
   >
   > volatile 读操作
   >
   > LoadStore屏障

   

3. OS和硬件层面
   https://blog.csdn.net/qq_26222859/article/details/52235930
   hsdis - HotSpot Dis Assembler
   windows lock 指令实现 | MESI实现



**synchronized实现细节**

1. 字节码层面
   ACC_SYNCHRONIZED
   monitorenter monitorexit
   
   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210819111902935.png" alt="image-20210819111902935" style="zoom:50%;" />
   
2. JVM层面
   C C++ 调用了操作系统提供的同步机制

3. OS和硬件层面
   X86 : lock cmpxchg指令实现





