---
title: java 存储只拥有基本数据类型字段的对象
date: 2017-11-11 01:10:23
categories:
    - java
tags:
    - java
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

有时候我们需要存储一个 Java 对象的信息，以便软件在下次打开的时候还能获取到原来的属性。通常在这种情况下，我们可以实现接口 `Serializable` 对该类的对象进行序列化，并用 `ObjectOutputStream` 将对象写入文件，那么下次就可以从文件中把这个对象读取出来。但是序列化有一个问题就是静态成员不能被序列化，因为序列化是保存的对象的信息，静态成员理论上是属于类信息，因此无法采用序列化保存。

以下是我在作业过程中遇到的题目，要求将字段属性存入到文件中，然后能从文件中读取出来。于是我使用了 `泛型` 与 `反射`，将这两个方法写成通用的了。
```java
public static <T> void saveFields(T t, String path, char separated)
public static <T> void getFields(T t, String path, char separated)
```
<!-- more -->
### Main
***
上面两个方法只能处理 Java 基本数据类型对应的包装类以及 `String` 类型，别的类型不支持，主要是因为这几个类型的数据是可以直接阅读观察的，因此很容易从文本中通过反射重新建立对象。
接下来看一个使用的例子：
```java
public class Main {
    static Person person = new Person();

    public static void main(String[] args) {
        // write your code here.
        person.setIdNo("20171111");
        person.setName("Locez");
        person.setAge(0);
        person.setSex("Male");
        person.setIsMerried(false);
        //保存对象的属性
        saveFields(person, "data.txt", ':');
        Person testPerson = new Person();
        //将保存的属性赋值给新对象
        getFields(testPerson, "data.txt", ':');
        System.out.println(testPerson.getName());
        System.out.println(testPerson.getAge());
        System.out.println(testPerson.getIsMerried());
        System.out.println(testPerson.score);
    }
}
```

### data.txt
***
写入后的文件内容应该如下：
```
name:Locez
sex:Male
age:0
idNo:20171111
isMerried:false
score:2333.0
```

### saveFields
***
```java
/**
  * @param t    对象实例
  * @param path 存储路径
  * @param separated 分隔符
  */
 public static <T> void saveFields(T t, String path, char separated) {
     Class<T> clazz = (Class<T>) t.getClass();
     Field[] fields = clazz.getDeclaredFields();
     String result = "";
     for (Field field : fields) {
         field.setAccessible(true);
         try {
             result += field.getName() + separated + field.get(t) + "\n";
         } catch (IllegalAccessException e) {
             e.printStackTrace();
         }
     }
     try (OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(new File(path)))) {
         osw.write(result);
         osw.flush();
     } catch (Exception e) {
         e.printStackTrace();
     }
 }
 ```

 ### getFields
 ***
 ```java
 /**
  * @param t         对象实例
  * @param path      存储路径
  * @param separated 分隔符
  */
 public static <T> void getFields(T t, String path, char separated) {
     Class<T> clazz = (Class<T>) t.getClass();
     try (Scanner scanner = new Scanner(new InputStreamReader(new FileInputStream(new File(path))))) {
         while (scanner.hasNext()) {
             String string = scanner.nextLine();
             String fieldName = string.split(String.valueOf(separated))[0];
             String value = string.split(String.valueOf(separated))[1];
             Field field = clazz.getDeclaredField(fieldName);
             //获取字段类型的 Class 信息，下面根据字段类型的不同做不同的处理。
             Class<?> fieldClass = Class.forName(field.getType().getCanonicalName());
             field.setAccessible(true);
             if (! (field.getType().getSimpleName().equals("String"))) {
                 //非 String 类则通过类的 valueOf 方法将 String 类型转为目标类型
                 if (! value.equals("null")) {
                     Method method = fieldClass.getDeclaredMethod("valueOf", String.class);
                     field.set(t, method.invoke(fieldClass, value));
                 }
             } else {
                 field.set(t, value);
             }
         }
     } catch (Exception e) {
         e.printStackTrace();
     }
 }
```
