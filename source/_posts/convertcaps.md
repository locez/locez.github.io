title: 数字转换成人民币大写读法
id: 80
categories:
  - JAVA
date: 2013-06-22 23:57:55
tags:
  - java
---

``` java
{
    //汉字数字
    public static String[] han={"零","壹","贰","叁","肆","伍","陆","柒","捌","玖"};
    //单位
    public static String[] unit={"元","十","百","千"};
    //单位
    public static String[] xunit={"角","分"};
    //分割整数与小数部分，返回字符串数组
    public static String[] divide(double num)
    {
	long zheng = (long)num;
	long xiao = Math.round((num-zheng)*100);
	if(xiao!=0)
	{
	    return new String[]{zheng+"",xiao+""};
	}
	else
	{
	    return new String[]{zheng+"",null};
	}
    }
    //转换成汉字，4位数
    public static String toHan(double number)
    {
        //使用divide方法得到分割后的整数与小数部分
	String[] arr=Convert.divide(number);
	//整数部分
        String numStr=arr[0];
        //小数部分
        String xiaoStr=arr[1];
        String result="";
        //大写字符串长度
        int numlen=numStr.length();
        for (int i=0;i&lt;numlen;i++)
        {
            //在ASCII码中，字符串数字的ASCII码-48得到相应的数字ASCII码
	    //String.charAt（int num）方法得到指定位置的字符值
	    int num=numStr.charAt(i)-48;
	    //数字不为0，则添加单位，否则不添加
	    if(num!=0)
	    {
	        result+=han[num]+unit[numlen-1-i];
	    }
	    else
	    {
	        result+=han[num];
	     }
        }
        //若存在小数部分则进行处理
        if(xiaoStr!=null)
        {
            for (int i=0;i&lt;2;i++)
	    {
	        int num=xiaoStr.charAt(i)-48;
	        result+=han[num]+xunit[i];
	    }
        }
        return result;
     }

     public static void main(String[] args)
     {
         System.out.println(toHan(6521.1));
     }
}
/*运行结果输出如下：
陆千伍百贰十壹元壹角零分
*/
```