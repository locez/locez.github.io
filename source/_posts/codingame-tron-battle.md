title: CodinGame-Tron Battle
id: 223
categories:
  - JAVA
date: 2014-05-25 15:35:42
tags:
  - java
---

![IDE CodinGame](https://image.locez.com/blog/codingame-tron-battle-00.png)

在viz的叙说下玩了一下，写了一段烂代码，但是还是有点用途的，至少不会马上就挂啊！！

Tron Battle，需要Player编写一段程序来移动，有点类似于贪吃蛇，不可以碰墙，也不可以碰自己，只能往空位的地方走，最大的不同在于Player是自己编写程序来进行移动，并且每次都有判断时限哦（100ms=0.1s），否则就TimeOut了，我这代码也会TimeOut，具体的有的地方还可以改进！这游戏如果可以的话应该是可以进行攻击以及防御的，根据对方的路线限制其行动。这样说来这其实应该和AI（人工智能）能扯上关系，有兴趣的可以去玩玩哦。

地址：www.codingame.com

上不去的可以试下翻墙！

下面贴上自己的Java版代码，写的不好，判断什么都是最简单的，实在没有时间想复杂的了！

<!--more-->

``` java
import java.util.*;
import java.io.*;
import java.math.*;

class Player {
	int[][] board;
	public void initBoard(){
		board=new int[30][20];
		for(int i=0;i&lt;30;i++){
			for(int j=0;j&lt;20;j++){
				board[i][j]=0;
			}
		}
	}
	public void add(int x,int y){
		board[x][y]=1;
	}
	public boolean isEmpty(int x,int y){
		if(x&gt;=29||x&lt;0||y&gt;=19||y&lt;0){
			return false;
		}
		if(board[x][y]==0){
			return true;
		}
		return false;
	}

    public static void main(String args[]) {

        // Read init information from standard input, if any
        Scanner in = new Scanner(System.in);
        Player player=new Player();
        player.initBoard();
        Random rand=new Random();

        int sum=0;
        while (true) {
        	int mX=0,mY=0;
        	// Read information from standard input
            int N = in.nextInt();
            int P= in.nextInt();
            // Compute logic here
            for(int i=1;i&lt;=N;i++){
            	int x1=in.nextInt(),y1=in.nextInt(),x2=in.nextInt(),y2=in.nextInt();
            	player.add(x1, y1);
            	player.add(x2, y2);
            	if(P==i-1){
            		mX=x2;mY=y2;
            	}
            }

            if(player.isEmpty(mX+1, mY)){
            	System.out.println("RIGHT");
            	continue;
            }
            if(player.isEmpty(mX-1, mY)){
            	System.out.println("LEFT");
            	continue;
            }

            if(player.isEmpty(mX, mY-1)){
            	System.out.println("UP");
            	continue;
            }
            if(player.isEmpty(mX+1, mY+1)){
            	System.out.println("DOWN");
            	continue;
            }

            // System.err.println("Debug messages...");

            // Write action to standard output

        }
    }
}
```