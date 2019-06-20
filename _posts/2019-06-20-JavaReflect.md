---
layout: post
title: Java的反射机制
date: 2019-06-20
tag: Reflect
---

### Java的反射机制

Java 反射机制在程序运行时，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。

这种 动态的获取信息 以及 动态调用对象的方法 的功能称为 java 的反射机制。

反射机制很重要的一点就是“运行时”，其使得我们可以在程序运行时加载、探索以及使用编译期间完全未知的 .class 文件。

换句话说，Java 程序可以加载一个运行时才得知名称的 .class 文件，然后获悉其完整构造，并生成其对象实体、或对其 fields（变量）设值、或调用其 methods（方法）。

### 通过反射来获取类的信息

```java

/**
 * 通过发射来获取类的所有变量 
 */
//获取类的名称
String name = Class.getName();

 //获取所有 public 访问权限的变量
Field[] fields = Class.getFields();
//获取所有public访问权限的方法
Method[] Methods = Class.getMethods();

//获取本类声明的所有变量，不问访问权限
Field[] fields = Class.getDeclaredFields();
//获取本类声明的所有变量，不问访问权限
Method[] Methods = Class.getDeclaredMethods();

//获取到私有变量需要操作私有变量时
private.setAccessible(true);//获取私有变量的访问权,private 暂指私有变量或方法

//获取访问权限
int modifiers = field.getModifiers();//method.getModifiers();
System.out.print(Modifier.toString(modifiers));//输出访问的权限

//获取方法的返回值类型
Class returnType = method.getReturnType();
//获取方法的参数
Parameter[] parameters = method.getParameters();
//获取方法抛出的异常
Class[] exceptionTypes = method.getExceptionTypes();



```
### 修改私有常量

**对于基本类型的静态常量，JVM 在编译阶段会把引用此常量的代码替换成具体的常量值。**

```java

/*
我们在程序运行时刻依然可以使用反射修改常量的值（后面会代码验证），但是 JVM 在编译阶段得到的 .class 文件已经将常量优化为具体的值，在运行阶段就直接使用具体的值了，所以即使修改了常量的值也已经毫无意义了。

无论直接为常量赋值 、 通过构造函数为常量赋值 还是 使用三目运算符，实际上我们都能通过反射成功修改常量的值。而我在上面说的修改"成功"与否是指：我们在程序运行阶段通过反射肯定能修改常量值，但是实际执行优化后的 .class 文件时，修改的后值真的起到作用了吗？换句话说，就是编译时是否将常量替换为具体的值了？如果替换了，再怎么修改常量的值都不会影响最终的结果了
*/

```
