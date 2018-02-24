---
title: JVM学习笔记_类加载机制详解二
date: 2018-02-24 15:30:35
categories:
- JVM学习笔记
tags:
- JVM学习笔记
---
# 编译期常量与运行期常量的区别 #

如下代码执行后会输出什么结果？
```
public class MyTest3 {
    public static void main(String[] args) {
        System.out.println(MyParent3.str);
    }
}

class MyParent3{

    public static final String str = "hello";

    static {
        System.out.println("MyParent3 static code");
    }
}
```
输出
```
hello
```
修改后的代码执行后会输出什么结果？
```
public class MyTest3 {
    public static void main(String[] args) {
        System.out.println(MyParent3.str);
    }
}

class MyParent3{

    public static final String str = UUID.randomUUID().toString();

    static {
        System.out.println("MyParent3 static code");
    }
}
```
输出
```
MyParent3 static code
fa328c0d-a230-4126-a720-c04d101f3dc2
```
**小结**
>当一个常量的值并非编译期间可以确定的，那么其值就不会被放到调用类的常量池中,这时在运行时，会导致主动使用这个常量所在的类，显然会导致这个类被初始化。


# 数组创建本质分析 #

如下代码执行后会输出什么呢？静态代码块是否会执行？
```
public class MyTest4 {
    public static void main(String[] args) {
        MyParent4 myParent4 = new MyParent4();//首次主动使用
    }
}

class MyParent4 {

    static {
        System.out.println("MyParent4 static block");
    }
}
```
输出：
```
MyParent4 static block
```
上述代码是对类主动使用情况之一：创建类的实例（首次主动使用）

我们在创建一个类的实例看下执行结果，静态代码块会执行几次？
```
public class MyTest4 {
    public static void main(String[] args) {
        MyParent4 myParent4 = new MyParent4();//首次主动使用
        System.out.println("============");
        MyParent4 myParent41 = new MyParent4();
    }
}

class MyParent4 {

    static {
        System.out.println("MyParent4 static block");
    }
}
```
输出
```
MyParent4 static block
============
```
非首次对类的主动使用不会导致类的初始化。

如下示例代码会输出什么结果？静态代码块是否会执行？数组类型是什么？
```
public class MyTest4 {
    public static void main(String[] args) {
        MyParent4[] myParent4s = new MyParent4[1];
        System.out.println(myParent4s.getClass());

        MyParent4[][] myParent4s1 = new MyParent4[1][1];
        System.out.println(myParent4s1.getClass());

        System.out.println(myParent4s.getClass().getSuperclass());
        System.out.println(myParent4s1.getClass().getSuperclass());
    }
}

class MyParent4 {

    static {
        System.out.println("MyParent4 static block");
    }
}

```
输出
```
class [Lcom.leofight.jvm.classloader.MyParent4;
class [[Lcom.leofight.jvm.classloader.MyParent4;
class java.lang.Object
class java.lang.Object
```


**小结**
>对于数组实例来说，其类型是由JVM在运行期动态生成的(类似动态代理)，表示为[Lcom.leofight.jvm.classloader.MyParent4这种形式。动态生成的类型，其父类型就是Object。
>
 >   对于数组来说，JavaDoc经常将构成数组的元素为Component，实际上就是将数组降低一个维度后的类型。

原生数据类型数组对应的数组类型，示例代码

```
public class MyTest4 {
    public static void main(String[] args) {
        int[] ints = new int[1];
        System.out.println(ints.getClass());
        System.out.println(ints.getClass().getSuperclass());

        char[] chars = new char[1];
        System.out.println(chars.getClass());

        boolean[] booleans = new boolean[1];
        System.out.println(booleans.getClass());

        short[] shorts = new short[1];
        System.out.println(shorts.getClass());

        byte[] bytes = new byte[1];
        System.out.println(bytes.getClass());
    }
}
```
输出：
```
class [I
class java.lang.Object
class [C
class [Z
class [S
class [B
```
**助记符补充**
>anewarray:表示创建一个引用类型的（如类、接口、数组）数组，并将其引用值压入栈顶。
    newarray:表示创建一个指定的原始类型（如int、float、char等）的数组，并将其引用值压入栈顶

# 接口初始化规则 #
```
public class MyTest5 {

    public static void main(String[] args) {
        System.out.println(MyChild5.b);
    }
}

interface MyParent5 {
    public static final int a = 5;
}

interface MyChild5 extends MyParent5 {
    public static final int b = 6;
}
```
编译后删除MyParent5和MyChild5的class文件，执行输出
```
6
```
修改代码如下：
```
public class MyTest5 {

    public static void main(String[] args) {
        System.out.println(MyChild5.b);
    }
}

interface MyParent5 {

    public static final int a = 5;

}

interface MyChild5 extends MyParent5 {

    public static final int b = new Random().nextInt(2);
}
```
编译后删除MyParent5和MyChild5的class文件，执行输出
```
Exception in thread "main" java.lang.NoClassDefFoundError: com/leofight/jvm/classloader/MyChild5
Exception in thread "main" java.lang.NoClassDefFoundError: com/leofight/jvm/classloader/MyParent5
```
**小结**
>当一个接口在初始化时，并不要求父接口都完成了初始化
>
>只有在真正使用到父类接口的时候（如引用接口中所定义的常量时），才会初始化

# 类加载器准备阶段与初始化阶段的重要意义分析 #

示例
```
public class MyTest6 {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getSingleton();
        System.out.println("counter1: " + Singleton.counter1);
        System.out.println("counter2: " + Singleton.counter2);
    }
}

class Singleton {

    public static int counter1;

    public static int counter2 = 0;

    private static Singleton singleton = new Singleton();

    private Singleton() {
        counter1++;
        counter2++;
    }

    public static Singleton getSingleton() {
        return singleton;
    }
}
```
输出结果会是什么呢？
```
counter1: 1
counter2: 1
```
分析：
执行`Singleton.getSingleton()`会调用` public static Singleton getSingleton() {
        return singleton;
    }`获取`Singleton`的实例，会调用代码`private static Singleton singleton = new Singleton();`接下来就会调用` private Singleton() {counter1++;
        counter2++;
    }`,在调用构造方法之前会给静态变量赋值counter1=0，counter2=0；所以执行为都为1。

调整上述代码的顺序，修改后代码如下：
```
package com.leofight.jvm.classloader;

public class MyTest6 {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getSingleton();
        System.out.println("counter1: " + Singleton.counter1);
        System.out.println("counter2: " + Singleton.counter2);
    }
}

class Singleton {

    public static int counter1;

    private static Singleton singleton = new Singleton();

    private Singleton() {
        counter1++;
        counter2++;//准备阶段的重要意义
    }

    public static int counter2 = 0;

    public static Singleton getSingleton() {
        return singleton;
    }
}
```
输出结果
```
counter1: 1
counter2: 0
```
分析：
静态变量初始化是按照声明的顺序初始化的，` public static int counter1` counter1初始化的初值为0（准备阶段给的默认值），然后`private static Singleton singleton = new Singleton();`初始化会调用构造方法`private Singleton() {
        counter1++;
        counter2++;
    }`这里引用到了counter2，counter2在准备阶段赋了默认值0，所以在这个阶段，counter1，counter2 都为1，继续初始化`public static int counter2 = 0;`显式的给counter2赋初值为0，所以counter1 =1，counter=0.

# 类加载器深入解析及重要特性剖析 #

**加载**：就是把二进制形式的java类型读入java虚拟机中

**验证**：类文件的结构检查、语义检查、字节码验证、二进制兼容性的验证
**准备**：为类变量分配内存，设置默认值。但是在到达初始化之前，类变量没有初始化为真正的初始化值
**解析**：解析过程就是类型的常理池中寻找类、接口、字段和方法的符号引用，把这些符号引用替换成直接引用的过程

**初始化**：为类变量赋予正确的初始值

**类实例化**：
为新的对象分配内存
为实例变量赋默认值
为实例变量赋正确的初始值
java编译器为它编译的每一个类都至少生成一个实例初始化方法，在java的class文件中，这个实例初始化方法被称为“<init>"。针对源代码中每一个类的构造方法，java编译器都产生一个<init>方法。

**类的加载的最终产品是位于内存中的Class对象**
**Class对象封装了类的方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。**

**有两种类型的类加载器**
1. Java虚拟机自带的加载器
          ①根类加载器（Bootstrap）
          ②扩展类加载器（Extension）
          ③系统（应用）类加载器（System)
2.   用户自定义的类加载器
       ①java.lang.ClassLoader的子类
       ② 用户可以定制类的加载方式


**类加载器并不需要等到某个类被“首次主动使用”时再加载它**

JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或者存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）

如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

**类的验证**

类被加载后，就进入连接阶段，连接就是将已经读入到内存的类的二进制数据合并到虚拟机的允许时环境中去。

类的验证的内容
    ①类文件的结构检查
    ②语义检查
    ③字节码验证
    ④二进制兼容性的验证

  在准备阶段，Java虚拟机为类的静态变量分配内存，并设置默认的初始值。例如对于以下Sample类，在准备阶段，将为int类型的静态变量a分配4个字节的内存空间，并且赋予默认值0，为long类型的静态变量b分配8个字节的内存空间，并且赋予默认值0.
```
public class Sample {
    private static int a = 1;
    public static long b;

    static {
        b = 2;
    }
  ...
}
```
在初始化阶段，Java虚拟机执行类的初始化语句，为类的静态变量赋予初始值。在程序中，静态变量的初始化有两种途径：（1）在静态变量的声明处进行初始化；（2）在静态代码块中进行初始化。例如在以下代码中，静态变量a和b都被显式初始化，而静态变量c没有被显式初始化，它将保持默认值0。
```
public class Sample {
    private static int a = 1;//在静态变量的声明处进行初始化
    public static long b;
    public static long c;

    static {
        b = 2;//在静态代码块中进行初始化
    }

  ...
}

```

静态变量的声明语句，以及静态代码块都被看做类的初始化语句，Java虚拟机会按照初始化语句的类文件中的先后顺序来依次执行它们。例如一下Sample类初始化后，它的静态变量a的取值为4.
```
 package com.leofight.jvm.classloader;

public class Sample {
    static int a = 1;

    static {
        a = 2;
    }

    static {
        a = 4;
    }

    public static void main(String args[]) {
        System.out.println("a=" + a);//打印a=4
    }
}
```

**类的初始化**

*类的初始化步骤*
  - 假如这个类还没有被加载和连接，那就先进行加载和连接
 -  假如类存在直接父类，并且这个父类还没有被初始化，那就先初始化直接父类。
 -  假如类中存在初始化语句，那就依次执行这些初始化语句。


**类的初始化时机**
- 主动使用
- 被动使用
详细内容见上一篇文章

当Java虚拟机初始化一个类时，要求它的所有父类都已经被初始化，但是这条规则并不使用于接口。
   - 在初始化一个类时，并不会先初始化它所实现的接口。
   - 在初始化一个接口时，并不会先初始化它的父接口。

因此，一个父接口并不会因为它的子接口或者实现类的初始化而初始化。只有当程序首次使用特点接口的静态变量时，才会导致该接口的初始化。

只有当程序访问的静态变量或者静态方法确实在当前类或者当前接口中定义时，才可以认为是对类或者接口的主动使用。


调用ClassLoader类的loadClass方法加载一个类，并不是对类的主动使用，不会导致类的初始化。


类加载器用来把类加载到Java虚拟机中。从JDK1.2版本开始，类的加载过程采用父亲委托机制，这种机制能更好地保证Java平台的安全。在此委托机制中，除了Java虚拟机自带的根类加载器以外，其余的类加载器都有且只有一个父加载器。当Java程序请求加载器loader1加载Sample类时，loader1首先委托自己的父类加载器去加载Sample类，若父加载器能加载，则由父加载器完成加载任务，否则才由加载器loader1本身加载Sample类。

*Java虚拟机自带了以下几种加载器。*
  - 根（Bootstrap）类加载器：该加载器没有父加载器。它负责加载虚拟机的核心库，如java.lang.*等。例如从例程（Sample.java)可以看出，java.lang.Object就是由根类加载器加载的。根类加载器从系统属性sun.boot.class.path所指定的目录中加载类库。根类加载器的实现依赖于底层操作系统，属性虚拟机的实现的一部分，它并没有继承java.lang.ClassLoader类。

- 扩展（Extension）类加载器：它的父类加载器就是根类加载器。它从java.ext.dirs系统属性所指定的目录中加载类库，或者从JDK的安装目录的jre\lib\ext子目录(扩展目录）下加载类库，如果把用户创建的JAR文件放在这个目录下，也会自动有扩展类加载器加载。扩展类加载器是纯Java类，是java.lang.ClassLoader类的子类。

- 系统（System）类加载器：也称为应用类加载器，它的父加载器为扩展类加载器。它从环境变量classpath或者系统属性java.class.path所指定的目录中加载类，它是用户自定义的类加载器的默认父加载器。系统类加载器是纯Java类，是java.lang.ClassLoader类的子类。

除了以上虚拟机自带的加载器外，用户还可以定制自己的类加载器。Java提供了抽象类java.lang.ClassLoader,所有用户自定义类的加载器都应该继承ClassLoader类。
![类加载器双亲委派模型](http://upload-images.jianshu.io/upload_images/9584733-c2f5c4c652193795.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
