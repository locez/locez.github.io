title: URLClassLoader加载类
id: 123
categories:
  - JAVA
date: 2013-09-27 23:18:17
tags:
  - java
---

ClassLoader提供了一个URLClassLoader实现类，该类是系统类加载器和扩展类加载器的父类。
URLClassLoader可以从本地二进制文件加载类，也可以从远程主机获取二进制文件加载类。

构造器

``` java
URLClassLoader(URL[] urls):使用parent（父类）加载器创建一个ClassLoader对象，该对象将从urls所指定的系列路径查询并获取类。

URKClassLoader(URL[] urls,ClassLoader paremt):使用指定父类加载器创建一个ClassLoader对象。

```
<!--more-->
接下来通过一个例子具体演示：

``` java
//定义一个Test接口，该接口包含一个write方法
public interface Test {
         void write();
}
```


定义一个mytest类，并实现Test接口,把该文件打包成jar，名字为TestClassloader.jar，把该文件放入远程主机或本地

``` java
public class mytest implements Test{
      public void write(){
         System.out.print("我是从服务器加载的");
      }
}
```

下面是测试类：

``` java
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;

public class testurl{

    public static void main(String[] args) throws Exception
    {
        //我这里使用本地服务器做示范，http:为前缀，表示从互联网通过HTTP访问加载
        //相同的，使用file:前缀，从本地加载；还可以使用ftp:前缀从互联网通过FTP访问加载。
        URL[] urls={new URL("http://127.0.0.1/TestClassloader.jar")};
        System.out.println("starting....");
        //获取当前时间
        long time=System.currentTimeMillis();
        //通过URLClassLoader构造器生成myclassloader对象
        URLClassLoader myclassloader=new URLClassLoader(urls);
        //使用URLClassLoader的loadClass动态加载类mytest，并调用Class对象的newInstance()方法创建了该类的对象。
        //使用接口声明一个Test引用类型的变量m，并把newInstance()生成的对象强制类型转换为Test
        Test m=(Test) myclassloader.loadClass("mytest").newInstance();
        //输出加载的耗时
        System.out.println("need time:"+(System.currentTimeMillis()-time));
        System.out.println("load end start method....");
        //因编译时类型和运行时类型都有write方法，因此可以使用编译时类型直接调用运行时类型的方法。
        m.write();
    }
}
```

运行testurl，结果输出如下：

``` java
starting....
need time:312
load end start method....
我是从服务器加载的
```

动态加载类可以灵活的处理各种需要的情况，同时动态加载类常与反射共同使用，因为既然是动态，那么类的类型是未知的，那么如何能调用该类的方法呢？这就需要运用反射执行方法了。