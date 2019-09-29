---
title: 中国剩余定理
date: 2019-09-29 18:42:11
categories:
 - algorithm
tags:
 - algorithm
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

### 什么是中国剩余定理？
---

中国剩余定理是数论中关于一元线性同余方程组的定理，说明了一元线性同于方程组有结的准则以及求解方法。《孙子算经》中有一题：

```
有物不知其数，三三数之剩二，五五数之剩三，七七数之剩二。问物几何？
```

这个题目翻译过来的意思就是：

```
一个整数除以 3 余 2， 除以 5 余 3， 除以 7 余 2，求这个整数是什么？
```

同时这也是韩信点兵的故事的来源，韩信让士兵们依次按 3 人一排，5 人一排， 7 人一排进行排列整队，每次只数最后一排的人，便可算出总人数

而韩信点兵与上面的题目同理，数最后一排的人数，也即数余数。3， 5， 7 这种小的数可以枚举，也可以有口诀，但是当这些除数很大，要怎么求解这些一元线性同余方程组，就是中国剩余定理要解决的了。

<!--more-->
### 中国剩余定理的现代数学描述
---

中国剩余定理其实是在求解以下形式的同余方程组：



$$
\begin{cases}
    x \equiv a_1 \pmod {m_1}  \\
    x \equiv a_2 \pmod {m_2}  \\  
    x \equiv a_3 \pmod {m_3}  \\
    \cdot \cdot \cdot \\
    x \equiv a_n \pmod {m_n} 
\end{cases}
$$

求解方法：

首先求得所有模的乘积：

$$
M = m_1 * m_2 * m3 * \cdot\cdot\cdot m_n = \prod_{i=1}^n m_i
$$

然后求得每个除了 $m_i$ 以外的 $n-1$ 个模的乘积，即：

$$
M_i = \frac M m_i
$$

再求得 $M_i$ 关于模 $m_i$ 的数论倒数 (用扩展欧几里得算法求):

$$
M_i^{-1}M_i \equiv 1 \pmod {m_i}
$$

最终方程解的通解为：

$$
x =  a_1M_1^{-1}M_1 + a_2M_2^{-1}M_2 + \cdot\cdot\cdot + a_nM_n^{-1}M_n + kM  = kM + \sum_{i=1}^n a_iM_i^{-1}M_i \\ k \in Z
$$

编程实现：

```
please input a number means n row, each row contains 2 numbers, the first is mod, second is remainder                   
example:
3
3 2
5 3
7 2                                                                             

3
3 2
5 3
7 2
23
```

``` cpp
#include<iostream>
#include<algorithm>
using namespace std;

const int size = 100;
int mod[size];
int a[size];
int Mi[size];
int MM_i[size];

int exgcd(int a, int b, int &x, int &y){
	if(b==0){
		x = 1;
		y = 0;
		return a;
	}
	int r = exgcd(b, a%b, x, y);
	int tmp = x;
	x = y;
	y = tmp - a/b * y;
	return r;
}

int main(){
	int n;
	long long M=1;
	cout << "please input a number means n row, each row contains 2 numbers, the first is mod, second is remainder" << endl;
	cout << "example:" << endl;
	cout << "3" << endl;
	cout << "3 2\n5 3\n7 2" <<endl;
	cin >> n;
	for(int i = 0; i < n; i++){
		cin >> mod[i];
		cin >> a[i];
		M *= mod[i];
	}
	for(int i = 0; i < n; i++){
		Mi[i]=M/mod[i];
		int mm_i, y;
		exgcd(Mi[i], mod[i], mm_i, y);
		MM_i[i] = mm_i;
	}
	int sum = 0;
	for(int i=0; i < n; i++){
		sum += a[i]*MM_i[i]*Mi[i];
	}
	cout << sum << endl;
	return 0;
}
```
