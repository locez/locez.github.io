title: java 实现简单的 ATM 类
date: 2015-10-01 21:09:32
categories:
 - JAVA
tags:
 - java
---

首先自己也很久没有写 **java** ，最近给别人出了一道 **java** 的题目，要求也就是实现一个简单的 **ATM** ，当作复习，我也自己写了一个这样的 **ATM** 类。

 - 需要验证密码
 - 查询功能
 - 存款功能
 - 转载功能

真的 **ATM** 使用 6 位的数字密码，但我自己做的使用了 `String` ，其中一个原因是考虑到 `String` 的 `equals` 方法的运用。当然为了测试，我也增加了一些自己的想法，例如缓存一个 `instance` 来模拟转帐操作。下面的代码，基本逻辑算是非常简单的（因此注释什么的我也没写了），但还是可以继续优化，例如判断输入是否符合 `int` 、 `String` ,此处我只是给出一个简单的答案而已。

<!--more-->

``` java
import java.util.Scanner;

public class ATM{
    private String person="";
    private final static String PASSWORD="MYPASSWORD";
    private int times=0,money=3000;
    private static ATM atmcache=new ATM();
    public ATM()
    {}
    public ATM(String person)
    {
        this.person=person;
    }

    //只是为了测试而设计的缓存对象
    public static ATM getATM(String name)
    {
        if(atmcache.getPerson().equals(name)){
            return atmcache;
        }
        else{
            atmcache=new ATM(name);
            return atmcache;
        }
    }

    //验证密码
    boolean verifyPassword(String password)
    {
        if(times>=3){
            return false;
        }
        else if(!PASSWORD.equals(password)){
            times++;
            System.out.println("Your password is incorrect,please check again;");
            return false;
        }
        else{
            return true;
        }
    }
	
    public String getPerson()
    {
        return person;
    }

    public int getTimes()
    {
        return times;
    }

    int getMoney()
    {
        return money;
    }

    int drawMoney(int money)
    {
        if(money<=this.money){
            //System.out.println(person+": Do you wan to draw "+money+"￥?");
            this.money=this.money-money;
            System.out.println(person+": Your accout now has "+this.money+"￥.\n");
            return this.money;
        }
        else{
            System.out.println(person+": Your accout is not enough,please check again.\n");
            return 0;
        }
    }

    int depositMoney(int money)
    {
        this.money+=money;
        System.out.println(person+": Your accout now has "+this.money+"￥.\n");
        return this.money;
    }

    int transferAccounts(ATM others,int money)
    {
        if(money<=this.money){
            drawMoney(money);
            others.depositMoney(money);
            return this.money;
        }
        else{
            System.out.println(person+": Your account is not enough,please check again.");
        return 0;
        }
    }
    public static void main(String[] args)
    {
        Scanner sc=new Scanner(System.in);
        ATM atm=new ATM("Locez");
        System.out.println("Please enter your password");
        while(true){
            if(atm.verifyPassword(sc.next()))
                break;
            if(atm.getTimes()>=3){
                System.out.println("Your account has benn locked.");
                System.exit(0);
            }
        }
        while(true)
        {	
            System.out.println("********************************");
            System.out.println(" Hello "+atm.person+"   Money : "+atm.getMoney()+"￥");
            System.out.println("********************************");
            System.out.println("1.Deposit money;\n2.Draw money;\n3.Transfer accouts;\n4.Exit.");
            int i=sc.nextInt();
            switch(i)
            {
                case 1:{
                    int flag;
                    System.out.println("Please enter the amount you want to deposit.");
                    int temp=sc.nextInt();
                    System.out.println("Please make sure you have the amount you want to deposit.\n"+temp+"￥\n1.ok    2.exit\n");
                    flag=sc.nextInt();
                    if(flag==1){
                        atm.depositMoney(temp);
                        break;
                    }
                    else{
                        System.out.println("You hava cancelled your action.\n");
                        break;
                    }
                }
                case 2:{
                    int flag;
                    System.out.println("Please enter the amount you want to draw.");
                    int temp=sc.nextInt();
                    System.out.println("Please make sure you have the amount you want to draw.\n"+temp+"￥\n1.ok    2.exit\n");
                    flag=sc.nextInt();
                    if(flag==1){
                        atm.drawMoney(temp);
                        break;
                    }
                    else{
                        System.out.println("You hava cancelled your action.\n");
                        break;
                    }
                }
                
                case 3:{
                    int flag;
                    System.out.println("Please enter the account name.");
                    String account_name=sc.next();
                    System.out.println("Please enter the amount you want to transfer.");
                    int temp=sc.nextInt();
                    System.out.println("Please make sure the information \nAcconu:"+account_name+"\nMoney:"+temp+"￥\n1.ok    2.exit\n");
                    flag=sc.nextInt();
                    if(flag==1){
                        atm.transferAccounts(ATM.getATM(account_name),temp);
                        break;
                    }
                    else{
                        System.out.println("You hava cancelled your action.\n");
                        break;
                    }
                }
                
                case 4:System.out.println("Please take away your card.Good Bye!\n");System.exit(0);
            }
        }
    }
}
```
