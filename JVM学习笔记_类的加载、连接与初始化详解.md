---
title: JVM学习笔记_类的加载、连接与初始化详解
date: 2018-02-04 15:30:35
categories:
- JVM学习笔记
tags:
- JVM学习笔记
---

# 类加载 #
 
1. 在java代码中，类型的加载、连接与初始化过程都是在程序运行期间完成的。

2. 提供了更大的灵活性，增加了更多的可能性。

# 类的加载、连接与初始化 #

1. **加载**：查找并加载类的二进制数据

	>类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在内存中创建一个java.lang.Class对象（规范并未说明Class对象位于哪里，HotSpot虚拟机将其放在了方法区中）用来封装类在方法区内的数据。

  + 加载.class文件的方式

	 ①从本地系统中直接加载
	
	 ②通过网络下载.class文件
	
	 ③从zip，jar等归档文件中加载.class文件
	
	 ④从专有数据库中提取.class文件
	
	 ⑤<font color ="red">将Java源文件动态编译为.class文件</font>


2. **连接**

   ① 验证：确保被加载的类的正确性
   
   ②准备：为类的<font color="red">静态变量</font>分配内存，并将其初始化为<font color = "red">默认值</font>
   
   ③解析：<font color="red">把类中的符号引用转换为直接引用</font>
   
3. <font color="red">**初始化**:为类的静态变量赋予正确的初始值</font>


**从类被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期分为5个阶段，加载(Loading)、连接（Linkging）、初始化(Initialization)、使用(Using)、卸御(Unloading)**


![](http://upload-images.jianshu.io/upload_images/9584733-6e0168c0a2c48be4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**JAVA虚拟机与程序的生命周期**
 
  - 在如下几种情况下，Java虚拟机将结束生命周期
    
      ①执行了System.exit()方法
      
      ②程序正常执行结束
      
      ③程序在执行过程中遇到了异常或者错误而异常终止
      
      ④由于操作系统出现错误而导致Jaava虚拟机进程终止

# Java程序对类的使用方式 #

1. 主动使用（<font color="red">七种</font>）
	
 ① 创建类的实例 

 ② 访问某个类或接口的静态变量，或者对该静态变量赋值

 ③ 调用类的静态方法

 ④ 反射（如：Class.forName("com.test.Test"))

 ⑤ 初始化一个类的子类

 ⑥ Java虚拟机启动时被标明为启动类的类（Java Test）

 ⑦ JDK1.7开始提供了动态语言支持：java.lang.invoke.MethodHandle实例的解析结果REF_getStatic，REF_putStatic,REF_invokeStatic句柄对应的类没有初始化，则初始化。
	
2. 被动使用
	
 ① 除了以上七种情况，其他使用Java类的方式都被看作是对类的<font color="red">被动使用</font>，都不会导致类的<font color="red">初始化</font>
 
 ② 所有的Java虚拟机实现必须在每个类或接口被Java"<font color="red">首次主动使用</font>时才初始化他们

# 示例 #

- 示例1

```
class MyParent1 {

    public static String str = "hello world";

    static {
        System.out.println("MyParent1 static block");
    }
}

class MyChild1 extends MyParent1 {

    public static String str2 = "welcome";

    static {
        System.out.println("MyChild1 static block");
    }
}

```

该main方法执行后，会输入什么结果呢？

```
public class MyTest1 {

    public static void main(String[] args) {
        System.out.println(MyChild1.str);
    }
}

```
输出：

```
MyParent1 static block
hello world

```

为什么MyChild1的静态代码块没实例化呢？
>对于静态字段来说，只有直接定义了该字段的类才会被初始化.


该main方法执行后，又会输入什么结果呢？

```
public class MyTest1 {

    public static void main(String[] args) {
        System.out.println(MyChild1.str2);
    }
}
```
输出：

```
MyParent1 static block
MyChild1 static block
welcome
```

这次为什么子类与父类的静态代码块都实例化了呢？
>当一个类在初始化时，要求其父类全部都已经初始化完毕了。


**JVM参数**

`-XX:+TraceClassLoading`,用于追踪类的加载信息并打印出来

` -XX:+<option>`,表示开启option选项

` -XX:-<option>`,表示关闭option选项

` -XX:<option>=<value>`,表示将option选项的值设置为value

通过`-XX:+TraceClassLoading`参数我们来看下调用`MyChild1.str`类的加载情况

```
[Opened /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Object from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.Serializable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Comparable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.CharSequence from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.String from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.AnnotatedElement from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.GenericDeclaration from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Type from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Cloneable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.System from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Throwable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Error from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ThreadDeath from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Exception from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.RuntimeException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.SecurityManager from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.ProtectionDomain from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.AccessControlContext from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.SecureClassLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ReflectiveOperationException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassNotFoundException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.LinkageError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.NoClassDefFoundError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassCastException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ArrayStoreException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.VirtualMachineError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.OutOfMemoryError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StackOverflowError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.IllegalMonitorStateException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.Reference from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.SoftReference from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.WeakReference from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.FinalReference from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.PhantomReference from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Cleaner from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.Finalizer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Runnable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Thread from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Thread$UncaughtExceptionHandler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ThreadGroup from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Map from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Dictionary from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Hashtable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Properties from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.AccessibleObject from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Member from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Field from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Parameter from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Executable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Method from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Constructor from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.MagicAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.MethodAccessor from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.MethodAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.ConstructorAccessor from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.ConstructorAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.DelegatingClassLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.ConstantPool from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.FieldAccessor from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.FieldAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.UnsafeFieldAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.UnsafeStaticFieldAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.annotation.Annotation from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.CallerSensitive from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandle from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.DirectMethodHandle from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MemberName from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleNatives from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.LambdaForm from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodType from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.BootstrapMethodError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.CallSite from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.ConstantCallSite from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MutableCallSite from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.VolatileCallSite from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Appendable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.AbstractStringBuilder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StringBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StringBuilder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Unsafe from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.AutoCloseable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.Closeable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.InputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.ByteArrayInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.File from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URLClassLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URL from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.Manifest from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Launcher from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Launcher$AppClassLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Launcher$ExtClassLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.CodeSource from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StackTraceElement from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.Buffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Boolean from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Character from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Number from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Float from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Double from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Byte from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Short from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Integer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Long from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.NullPointerException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ArithmeticException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.ObjectStreamField from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Comparator from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.String$CaseInsensitiveComparator from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.Guard from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.Permission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.BasicPermission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.RuntimePermission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.AccessController from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.ReflectPermission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.PrivilegedAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.ReflectionFactory$GetReflectionFactoryAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.cert.Certificate from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Iterable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.List from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.RandomAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.AbstractCollection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.AbstractList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Vector from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Stack from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.ReflectionFactory from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.Reference$Lock from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.Reference$ReferenceHandler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.ReferenceQueue from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.ReferenceQueue$Null from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.ReferenceQueue$Lock from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ref.Finalizer$FinalizerThread from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.VM from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Map$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Hashtable$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Math from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.Charset from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.spi.CharsetProvider from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.FastCharsetProvider from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.StandardCharsets from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.AbstractMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.PreHashedMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.StandardCharsets$Aliases from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.StandardCharsets$Classes from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.StandardCharsets$Cache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ThreadLocal from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.atomic.AtomicInteger from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.IncompatibleClassChangeError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.NoSuchMethodError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.ArrayList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Set from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.AbstractSet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$EmptySet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$EmptyList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$EmptyMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$UnmodifiableCollection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$UnmodifiableList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$UnmodifiableRandomAccessList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.Reflection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.HashMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.HashMap$Node from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$ReflectionData from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$Atomic from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.generics.repository.AbstractRepository from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.generics.repository.GenericDeclRepository from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.generics.repository.ClassRepository from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$AnnotationData from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.annotation.AnnotationType from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.WeakHashMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassValue$ClassValueMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Modifier from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.LangReflectAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.ReflectAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Arrays from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.HistoricallyNamedCharset from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.Unicode from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.UTF_8 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.ReflectionFactory$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.NativeConstructorAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.DelegatingConstructorAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StringCoding from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ThreadLocal$ThreadLocalMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ThreadLocal$ThreadLocalMap$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StringCoding$StringDecoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ArrayDecoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CharsetDecoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.UTF_8$Decoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CodingErrorAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Hashtable$EntrySet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$SynchronizedCollection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$SynchronizedSet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Objects from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Enumeration from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Iterator from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Hashtable$Enumerator from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Runtime from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Version from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileDescriptor from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaIOFileDescriptorAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileDescriptor$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.SharedSecrets from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.Flushable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.OutputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileOutputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FilterInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.BufferedInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.atomic.AtomicReferenceFieldUpdater from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.atomic.AtomicReferenceFieldUpdater$AtomicReferenceFieldUpdaterImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.PrivilegedExceptionAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.atomic.AtomicReferenceFieldUpdater$AtomicReferenceFieldUpdaterImpl$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.misc.ReflectUtil from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FilterOutputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.PrintStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.BufferedOutputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.Writer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.OutputStreamWriter from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.StreamEncoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.security.action.GetPropertyAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ArrayEncoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CharsetEncoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.UTF_8$Encoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.ByteBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.HeapByteBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.Bits from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.ByteOrder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaNioAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.Bits$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.BufferedWriter from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.DefaultFileSystem from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileSystem from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.UnixFileSystem from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.ExpiringCache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.LinkedHashMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.ExpiringCache$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Enum from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.File$PathStatus from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.file.Watchable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.file.Path from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.StringCoding$StringEncoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassLoader$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.ExpiringCache$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.LinkedHashMap$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassLoader$NativeLibrary from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Terminator from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.SignalHandler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Terminator$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Signal from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.NativeSignalHandler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Integer$IntegerCache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.OSEnvironment from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaLangAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.System$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.IllegalArgumentException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Compiler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Compiler$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URLStreamHandlerFactory from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Launcher$Factory from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.security.util.Debug from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassLoader$ParallelLoaders from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.WeakHashMap$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Collections$SetFromMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.WeakHashMap$KeySet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaNetAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URLClassLoader$7 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.StringTokenizer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Launcher$ExtClassLoader$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.MetaIndex from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Readable from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.Reader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.BufferedReader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.InputStreamReader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileReader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.StreamDecoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.CharBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.HeapCharBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CoderResult from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CoderResult$Cache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CoderResult$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.CoderResult$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.Array from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.HashMap$TreeNode from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileInputStream$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.www.ParseUtil from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.BitSet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Locale from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.locale.LocaleObjectCache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Locale$Cache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.locks.Lock from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.locks.ReentrantLock from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$Segment from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$Node from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$CounterCell from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$CollectionView from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$KeySetView from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$ValuesView from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$EntrySetView from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.locale.BaseLocale from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.locale.BaseLocale$Cache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.locale.BaseLocale$Key from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.locale.LocaleObjectCache$CacheEntry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Locale$LocaleKey from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.util.locale.LocaleUtils from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.CharacterData from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.CharacterDataLatin1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Parts from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URLStreamHandler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.www.protocol.file.Handler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaSecurityAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.ProtectionDomain$JavaSecurityAccessImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaSecurityProtectionDomainAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.ProtectionDomain$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.ProtectionDomain$Key from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.Principal from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.HashSet from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.www.protocol.jar.Handler from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Launcher$AppClassLoader$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.SystemClassLoaderAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.InternalError from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.instrument.Instrumentation from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.instrument.InstrumentationImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.instrument.TransformerManager from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.instrument.TransformerManager$TransformerInfo from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URLClassLoader$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.util.URLUtil from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$Loader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$JarLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipConstants from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipFile from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaUtilZipFileAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipFile$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$JarLoader$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.FileURLMapper from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.JarFile from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JavaUtilJarAccess from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.JavaUtilJarAccessImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.charset.StandardCharsets from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.US_ASCII from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ISO_8859_1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.UTF_16BE from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.UTF_16LE from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.UTF_16 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Queue from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Deque from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.ArrayDeque from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipCoder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.PerfCounter from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Perf$GetPerfAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Perf from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.PerfCounter$CoreCounters from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.ch.DirectBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.MappedByteBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.DirectByteBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.LongBuffer from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.nio.DirectLongBufferU from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.JarIndex from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipEntry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.JarEntry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.JarFile$JarFileEntry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipFile$ZipFileInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.Inflater from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZStreamRef from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.InflaterInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.zip.ZipFile$ZipFileInflaterInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.AbstractSequentialList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.LinkedList from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.LinkedList$Node from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.ExtensionDependency from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.IOUtils from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$FileLoader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ThreadLocalCoders from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ThreadLocalCoders$Cache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ThreadLocalCoders$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.cs.ThreadLocalCoders$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.Resource from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$JarLoader$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.Attributes from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.Manifest$FastInputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.Attributes$Name from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.ASCIICaseInsensitiveComparator from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.JarVerifier from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.CodeSigner from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.jar.JarVerifier$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.ByteArrayOutputStream from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Package from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.security.util.SignatureFileVerifier from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.security.util.ManifestEntryVerifier from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.nio.ByteBuffered from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.PermissionCollection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.Permissions from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URLConnection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.www.URLConnection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.www.protocol.file.FileURLConnection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.www.MessageHeader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FilePermission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FilePermission$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FilePermissionCollection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.AllPermission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.UnresolvedPermission from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.security.BasicPermissionCollection from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded com.intellij.rt.execution.application.AppMainV2$Agent from file:/Applications/IntelliJ%20IDEA.app/Contents/lib/idea_rt.jar]
[Loaded sun.instrument.InstrumentationImpl$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.NativeMethodAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.reflect.DelegatingMethodAccessorImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded com.intellij.rt.execution.application.AppMainV2 from file:/Applications/IntelliJ%20IDEA.app/Contents/lib/idea_rt.jar]
[Loaded com.intellij.rt.execution.application.AppMainV2$1 from file:/Applications/IntelliJ%20IDEA.app/Contents/lib/idea_rt.jar]
[Loaded java.util.concurrent.ConcurrentHashMap$ForwardingNode from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.reflect.InvocationTargetException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.NoSuchMethodException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Socket from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocketAddress from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetSocketAddress from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddress from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetSocketAddress$InetSocketAddressHolder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.security.action.GetBooleanAction from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddress$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddress$InetAddressHolder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddress$Cache from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddress$Cache$Type from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddressImplFactory from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleImpl$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.function.Function from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleImpl$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleImpl$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddressImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Inet6AddressImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassValue from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleImpl$4 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassValue$Entry from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassValue$Identity from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.ClassValue$Version from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.spi.nameservice.NameService from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.InetAddress$2 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MemberName$Factory from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.util.IPAddressUtil from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleStatics from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Inet4Address from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.invoke.MethodHandleStatics$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.PostVMInitHook from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.usagetracker.UsageTrackerClient from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.concurrent.atomic.AtomicBoolean from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocksConsts from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.usagetracker.UsageTrackerClient$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocketOptions from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocketImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.usagetracker.UsageTrackerClient$4 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.AbstractPlainSocketImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.PlainSocketImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocksSocketImpl from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.usagetracker.UsageTrackerClient$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.AbstractPlainSocketImpl$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.IOException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocketException from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.SocksSocketImpl$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.ProxySelector from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.spi.DefaultProxySelector from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.io.FileOutputStream$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.spi.DefaultProxySelector$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.NetProperties from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.NetProperties$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.Properties$LineReader from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.launcher.LauncherHelper from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Inet6Address from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.misc.URLClassPath$FileLoader$1 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.URI from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded com.leofight.jvm.classloader.MyTest1 from file:/Users/lizhi/IdeaProjects/jvm_lecture/out/production/classes/]
[Loaded java.net.URI$Parser from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.launcher.LauncherHelper$FXHelper from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$MethodArray from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.spi.DefaultProxySelector$NonProxyInfo from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Void from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.spi.DefaultProxySelector$3 from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Proxy from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Proxy$Type from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.util.ArrayList$Itr from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.NetHooks from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.NetHooks$Provider from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded sun.net.sdp.SdpProvider from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded com.leofight.jvm.classloader.MyParent1 from file:/Users/lizhi/IdeaProjects/jvm_lecture/out/production/classes/]
[Loaded com.leofight.jvm.classloader.MyChild1 from file:/Users/lizhi/IdeaProjects/jvm_lecture/out/production/classes/]
MyParent1 static block
hello world
[Loaded java.lang.Shutdown from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Shutdown$Lock from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.Inet6Address$Inet6AddressHolder from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.net.NetworkInterface from /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre/lib/rt.jar]
```
从打印的加载信息可看出MyChild1已被加载。


- 示例2

```
class MyParent2 {

    public static String str = "hello world";

    static {
        System.out.println("MyParent2 static block");
    }
}

```
执行该main方法会输出什么结果

```
public class MyTest2 {

    public static void main(String[] args) {
        System.out.println(MyParent2.str);
    }
}

```

输出

```
MyParent2 static block
hello world
```

>上述代码是对于MyTest2的主动使用，主动使用会导致它初始化，初始化就会使的静态代码块的执行。



对上述代码str属性增加fina修饰，修改后的代码如下：

```
class MyParent2 {

    public static final String str = "hello world";

    static {
        System.out.println("MyParent2 static block");
    }
}
```
再执行main方法会输出什么呢？

```
public class MyTest2 {

    public static void main(String[] args) {
        System.out.println(MyParent2.str);
    }
}

```

输出：

```
hello world

```

为什么静态代码块没执行呢？
>常量在编译阶段会存入到调用这个常量的方法所在类的常量池中，本质上，调用类并没有直接引用到定义常量的类，因此并不会触发定义常量的类的初始化。
    
> 注意：这里指的是将常量存放到MyTest2的常量池中，之后MyTest2与MyParent2就没任何关系了,甚至，我们可以将MyParent2的class文件删除。


反编译该程序`javap -c com.leofight.jvm.classloader.MyTest2`

```
public class com.leofight.jvm.classloader.MyTest2 {
  public com.leofight.jvm.classloader.MyTest2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #4                  // String hello world
       5: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}


```

main方法中的ldc的注释是String hello world，也就是说这段代码已经是hello world了。

>ldc表示将int、float或者是String类型的常量值从常量池中推送至栈顶。


对上述程序做修改，修改如下：

```
class MyParent2 {

    public static final String str = "hello world";

    public static final short s = 7;

    static {
        System.out.println("MyParent2 static block");
    }
}

public class MyTest2 {

    public static void main(String[] args) {
        System.out.println(MyParent2.s);
    }
}
```
输出：
```
7
```

反编译结果：

```

public class com.leofight.jvm.classloader.MyTest2 {
  public com.leofight.jvm.classloader.MyTest2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: bipush        7
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
       8: return
}


```
>bipush表示将单字节（-128 ~ 127）的常量值推送至栈顶。

修改‘public static final short s = 127;’，反编译结果：

```
public class com.leofight.jvm.classloader.MyTest2 {
  public com.leofight.jvm.classloader.MyTest2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: bipush        127
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
       8: return
}


```
与上一个程序一样，助记符没变。

继续修改，如下：

```
class MyParent2 {

    public static final String str = "hello world";


    public static final short s = 127;


    public static final int i = 128;

    static {
        System.out.println("MyParent2 static block");
    }
}


public class MyTest2 {

    public static void main(String[] args) {
        System.out.println(MyParent2.i);
    }
}


```

反编译结果：

```
ublic class com.leofight.jvm.classloader.MyTest2 {
  public com.leofight.jvm.classloader.MyTest2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: sipush        128
       6: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
       9: return
}


```
>sipush表示将一个短整型常量值（-32768 ~ 32767）推送至栈顶。


继续修改，代码如下：

```

class MyParent2 {

    public static final String str = "hello world";


    public static final short s = 127;


    public static final int i = 128;

    public static final int m = 1;


    static {
        System.out.println("MyParent2 static block");
    }
}

public class MyTest2 {

    public static void main(String[] args) {
        System.out.println(MyParent2.m);
    }
}


```

反编译结果：

```
public class com.leofight.jvm.classloader.MyTest2 {
  public com.leofight.jvm.classloader.MyTest2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: iconst_1
       4: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
       7: return
}

```

>iconst_1表示将int类型1推送至栈顶(iconst_1 ~ iconst_5)



**助记符总结**
> 1. ldc表示将int、float或者是String类型的常量值从常量池中推送至栈顶。
> 2. bipush表示将单字节（-128 ~ 127）的常量值推送至栈顶。
> 3. sipush表示将一个短整型常量值（-32768 ~ 32767）推送至栈顶。
> 4. iconst_1表示将int类型1推送至栈顶(iconst_1 ~ iconst_5)








      
      

 