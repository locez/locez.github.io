title: 按字节截子字符串
id: 59
categories:
  - JAVA
date: 2013-06-22 22:36:17
tags:
  - java
---

/*刚刚测试的时候发现java在Linux与windows下一个中文字符所代表的字节不相同，此处是在Linux下测试的，以下作为字节参考：

--------windows Linux
----中文   2      3
----英文   1      1 */

``` java
//Substr类

public class Substr
{
    //sub方法
    public static String sub(String s,int start,int end)
    {
        //把字符串s转换成字节数组
	byte[] arr=s.getBytes();
	//定义一个字节数组subs，用来接收子字符串
	byte[] subs=new byte[arr.length];
	//使用循环，令subs字节数组接收子字节
	for(int i=start,j=0;i&lt;=end;i++)
	{
	    subs[j]=arr[i];
	    j++;
	}
	//使用subs直接用String构造函数返回子字符串
	String substring = new String(subs);
	return substring;
    }

    public static void main(String [] args)
    {
        String s="hello我是子字符串word";
        System.out.println(s);
        System.out.println("子字符串:");
        System.out.println(Substr.sub(s,5,22));
    }
}
/*运行输出如下：
hello我是子字符串word
子字符串:
我是子字符串*/
```