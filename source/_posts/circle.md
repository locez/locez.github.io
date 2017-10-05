title: 画圆
id: 37
categories:
  - JAVA
date: 2013-06-15 20:56:29
tags:
  - java
---

``` java

//Circle类

public class Circle

{
	public static void main(String[] args)
	{
		//圆的半径
		int r = 6;
		//y为纵坐标变量
		for(int y=0;y&lt;=2*r;y=y+2)
		{
			//求出1/2弦长
			long x=Math.round(Math.sqrt(r*r-(r-y)*(r-y)));
			//输出*号
			for(int i=0;i&lt;=2*r;i++)
			{
				//i为横坐标
				if(i==r-x||i==r+x)
				{
					System.out.print("*");
				}
				else
				{
					System.out.print(" ");
				}
			}
			//转行
			System.out.println("");
		}
	}

}
```
