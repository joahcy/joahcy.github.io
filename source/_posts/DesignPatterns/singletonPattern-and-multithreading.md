---
title: 单例模式与多线程
date: 2017-06-05
comments: false
categories: 设计模式
tags: [设计模式,多线程]

---

## 什么是单例模式?
单例模式是为确保一个类只有一个实例,并为整个系统提供一个全局访问点的一种模式方法.从概念中体现出了单例的一些特点:
- 在任何情况下,单例类永远只有一个实例存在
- 单例需要有能力为整个系统提供这一唯一实例

下面来详细看看多线程中的各版本单例模式是怎么实现的吧!

## 立即加载/"饿汉模式"
调用方法之前实例已经被创建了  
实现代码:

```
class MyObject{
    
    private static MyObject myObject = new MyObject();
    
    private MyObject(){}
    
    public static MyObject getInstance(){
        return myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
668661813
668661813
668661813
```
从结果可以看出,饿汉式可以实现线程安全的单例模式.

## 延迟加载/"懒汉模式"
调用get()方法时才创建实例   
实现代码:

```
class MyObject {

    private static MyObject myObject = null;
    private MyObject(){}
    public static MyObject getInstance(){
        if(myObject==null){
            myObject = new MyObject();
        }
        return myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
1928052572
417166340
1928052572
```
从结果可以发现,hashcode并不是唯一的,也就是说创建出了多例,存在非线程安全问题.那怎么办呢?有4种解决方案:
- 同步方法,使用synchronized关键字
- 同步代码块
- 针对重要代码进行同步
- 使用DCL(Double-Check Locking)双检查机制,且DCL是大多数多线程结合单例模式使用的解决方案.(***注:jdk1.5之前不能使用***)

### 方案一: 同步方法,使用synchronized关键字
实现代码:

```
class MyObject {

    private static MyObject myObject = null;
    private MyObject(){}
    public static synchronized MyObject getInstance(){
        if(myObject==null){
            myObject = new MyObject();
        }
        return myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
417166340
417166340
417166340
```
结果显示,此方案可行.但是运行效率低下,下一个线程必须等上一个线程释放锁后,才可以继续执行.

### 方案二: 同步代码块
实现代码:

```
class MyObject {
    private static MyObject myObject = null;

    private MyObject(){}

    public static MyObject getInstance() {
        try {
            synchronized (MyObject.class) {

                if (myObject == null) {
                    //模拟在对象创建之前做一些准备性的工作
                    Thread.sleep(3000);
                    myObject = new MyObject();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
1928052572
1928052572
1928052572
```
结果显示该方案也可行,写法等同于方案一,全部代码都被上锁,运行效率也很低

### 方案三: 针对重要代码进行同步
实现代码:

```
class MyObject {
    private static MyObject myObject = null;

    private MyObject(){}

    public static MyObject getInstance() {
        try {
            if (myObject == null) {
                //模拟在对象创建之前做一些准备性的工作
                Thread.sleep(3000);
                synchronized (MyObject.class) {
                    myObject = new MyObject();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
1928052572
1599065238
417166340
```
从结果看出,此方案并不能解决非线程安全问题.看看下一个解决方案.

### 方案四: 使用DCL(Double-Check Locking)双检查机制
实现代码:

```
class MyObject {
    //使用volatile关键字保证其可见性,并禁止内存重排序来让DCL生效
    private static volatile MyObject myObject = null;

    private MyObject() {}

    public static MyObject getInstance() {
        try {
            if (myObject == null) {
                //模拟在对象创建之前做一些准备性的工作
                Thread.sleep(3000);
                synchronized (MyObject.class) {
                    if(myObject == null){
                        myObject = new MyObject();
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return myObject;
    }
}
```
测试代码:
```
public class Test{
    public static void main(String[] args) {

        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
1398828021
1398828021
1398828021
```
虽然从结果看,DCL可以实现线程安全的单例模式,但这里还需深入讨论理解DCL机制,请看 [有关“双重检查锁定失效”的说明](http://ifeve.com/doublecheckedlocking/) , [Java线程安全兼谈DCL](http://www.iteye.com/topic/875420) ,[单例模式(DCL缺陷以及如何安全发布对象)](http://blog.csdn.net/u014108122/article/details/38352005) 这三篇文章来帮助我们深入理解java里面的DCL.


## 使用静态内置类实现单例模式
实现代码:

```
class MyObject {

    private MyObject(){}
    //内部类
    public static class MyObjectHandler{
        private static MyObject myObject = new MyObject(); 
    }
    
    public static MyObject getInstance(){
        return MyObjectHandler.myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
1879096508
1879096508
1879096508
```
结果显示可以达到线程安全,但是遇到序列化对象时,使用默认的方法会得到多例.

## 序列化与反序列化的单例模式实现
在多线程下,单例对象需要序列化和反序列化时,添加readResolve() 方法,就不会新创建对象.  
实现代码:

```
class MyObject implements Serializable{
    private static final long serialVersionUID = 1L;

    private MyObject(){}
    //内部类
    public static class MyObjectHandler{
        private static MyObject myObject = new MyObject(); 
    }
    
    public static MyObject getInstance(){
        return MyObjectHandler.myObject;
    }
    
    protected Object readResolve() throws ObjectStreamException{
        System.out.println("readrReslove");
        return MyObjectHandler.myObject;
    }
}
```
测试代码:

```
public class Test {

    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
        
    }
    
    //模拟序列化和反序列化
    public static void saveAndRead() {
        
        try {
            MyObject myObject=MyObject.getInstance();
            FileOutputStream fos = new FileOutputStream(new File("myobjectfile.txt"));
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(myObject);
            oos.close();
            fos.close();
            System.out.println(myObject.hashCode());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        try {
            FileInputStream fis = new FileInputStream(new File("myobjectfile.txt"));
            ObjectInputStream ois = new ObjectInputStream(fis);
            MyObject myObject = (MyObject) ois.readObject();
            ois.close();
            fis.close();
            System.out.println(myObject.hashCode());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        
    } 
}

class MyThread extends Thread{
    @Override
    public void run() {
       Test.saveAndRead();
    }
}
```
运行结果:

```
1087933741
readrReslove
1087933741
1087933741
readrReslove
1087933741
1087933741
readrReslove
1087933741
```
结果显示,每次反序列化之前都会调用readResolve()方法,从而实现线程安全的单例模式.

## 使用static代码块实现单例模式
实现代码:

```
public class MyObject {

    private static MyObject myObject = null;

    static {
        myObject = new MyObject();
    }

    public static MyObject getInstance(){
        return myObject;
    }
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
668661813
668661813
668661813
```
结果显示,此单例模式也是线程安全的.

## 使用enum枚举类型实现单例模式
实现代码:

```
public enum MyEnumSingleton {

    singletonFactory;
    
    private MySingleton instance;
    //使用enum时,构造方法被自动调用
    private MyEnumSingleton(){
        instance=new MySingleton();
    }
    public MySingleton getInstance(){
        return instance;
    }
    
}
//需要实现单例的类，比如数据库连接Connection
class MySingleton{
    public MySingleton(){}
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyEnumSingleton.singletonFactory.getInstance().hashCode());
    }
}
```
运行结果:

```
779325750
779325750
779325750
```
结果显示也可行.

## 完善使用enum枚举实现单例模式
前面虽然用enum实现了,但是枚举类型被暴露,违反了"职责单一原则",需要完善一下.
实现代码:

```
public class MyObject{
    
    public enum MyEnumSingleton {

        singletonFactory;
        
        private MySingleton instance;
        //使用enum时,构造方法被自动调用
        private MyEnumSingleton(){
            instance=new MySingleton();
        }
        public MySingleton getInstance(){
            return instance;
        }
        
    }
    
    public static MySingleton getInstance(){
        return MyEnumSingleton.singletonFactory.getInstance();
    }
}
//需要实现单例的类，比如数据库连接Connection
class MySingleton{
    public MySingleton(){}
}
```
测试代码:

```
public class Test{
    public static void main(String[] args) {
        
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
        
    }
}

class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}
```
运行结果:

```
854728855
854728855
854728855
```
结果显示也可行.

## 总结
以上就是线程安全的单例模式的所有版本,最近在看"Java多线程编程核心技术",特此记录一下,以便日后好查看复习.

