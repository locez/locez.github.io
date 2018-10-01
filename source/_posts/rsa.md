---
title: RSA 算法
date: 2018-10-01 12:52:11
categories:
 - linux
 - cryptography
 - algorithm
tags:
 - linux
 - algorithm
 - rsa
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1

### RSA 是什么？
---
RSA 是以发明者 **Ron Rivest**、**Adi Shamir**、**Leonard Adleman** 名字的首字母命名的一种非对称加密算法。在公钥密码系统中，加密密钥与解密密钥不同，由加密密钥推导出解密密钥在计算上是不可行的，系统的加密算法和加密密钥可以公开，只要保存好解密密钥即可。而 RSA 则是基于整数因子分解问题的，对极大数做因数分解的难度决定了 RSA 算法的可靠性。
<!--more-->

### RSA 密钥生成的一般步骤
---
1.首先选取两个大素数 `p` 和 `q`，然后计算它们的乘积：

$$
N=pq
$$

2.然后计算乘积 `N` 的欧拉函数 $\phi(N)$，其中欧拉函数指的是对于正整数 N，所有小于或等于 N 的正整数中，与 N 互质的数的个数，它的数学公式为：

$$
\phi(N)=N(1-\frac{1}{p_1})(1-\frac{1}{p_2})\cdots(1-\frac{1}{p_k})
$$

即：

$$
\phi(N) =\prod_{i=1}^{k}\quad N(1-\frac{1}{p_i})
$$

其中 $ p_i $ 为 N 的所有质因数

我们还会用到以下两个性质：
I:
$$
若 p,q 互质，则有:
\phi(pq)=\phi(p)\phi(q)
$$
II:
$$
若有质数 p 且  p>1 ，则有：
\phi(p)=p-1
$$


3.接着随机选择选择一个整数 `e`，满足 $ 1 < e <\phi(N) $，且 $ e  与  \phi(N) $ 互质，通常选择 `65537`

4.最后计算 `e` 对于 $ \phi(N) $ 的模反元素 `d`，其中模反元素的定义为：如果有两个正整数 `a` 和 `n` 互质，那么一定可以找到整数 `b`，使得 $ n|ab-1 $，`b` 称为 `a` 模反元素，可用扩展欧几里德算法求得

5.将 (e,N) 组装成公钥，将 (d,N) 组装成私钥

### RSA 加密解密的一般步骤
---
设 `m` 为明文，`c` 为密文，`e` 上述生成的公钥指数，`d` 为 `e` 对于 $ \phi(N) $ 的模反元素
加密：

$$
c=m^e mod N
$$

解密：

$$
m=c^d mod N
$$

### 编程实现
---
在本例子中，是比较简单的加密实现，没有涉及适当的分组操作，对字符串加密采用了最低效率的按字节加密，仅仅作为例子。实际上的 RSA 算法，会采用分组操作，每个分组的长度 `k` 应该满足$ 2^k\leq N $，最后也会跟下文的实现一样，形成多个密文输出。

``` golang
generating p and q:
p: 4637 q: 5591
N: 25925467 eulerN: 25915240
calculating e and d
e: 2649 d:2484889
generate random integer:8608
c: 5853546 m:8608
encrypt sting:

encrypt sting to []int:[14773902 6929511 3881662 3881662 17177132 1441672 8018696 20570993 1197689 8018696 8990288 7523664 8018696 9478268 19565276 5933296 21454753 23718142 1994912 18323106 9394526 20490471 8041615 25550957 8044600 21575371 24665880 4225706 13781244 13667208 8044600 13781244 18578049 8044600 7523664 6485992 5933296 1581769 20457713]
decrypt string:hello,我是按照逐个字节加密的 
```

最后是编程实现代码，用 golang 实现：

``` golang
package main

import (
	"fmt"
	"math"
	"math/rand"
	"time"
)

const LIMIT = 10000

func main() {
	fmt.Println("generating p and q:")
	p, q := genPQ()
	fmt.Printf("p: %v q: %v\n", p, q)
	N := p * q
	eulerN := euler(p, q)
	fmt.Printf("N: %v eulerN: %v\n", N, eulerN)
	fmt.Println("calculating e and d")
	e, d := genED(eulerN)
	fmt.Printf("e: %v d:%v\n", e, d)
	r := rand.New(rand.NewSource(time.Now().UnixNano()))

	random := r.Intn(LIMIT)
	fmt.Printf("generate random integer:%v\n", random)

	c := encrypt(random, e, N)
	m := decrypt(c, d, N)

	fmt.Printf("c: %v m:%v\n", c, m)

	fmt.Println("encrypt sting:\n")
	en := encryptString("hello,我是按照逐个字节加密的", e, N)
	fmt.Printf("encrypt sting to []int:%v\n", en)
	fmt.Printf("decrypt string:%v \n", decryptString(en, d, N))
}

func isPrime(n int) bool {
	sn := int(math.Sqrt(float64(n)))
	for i := 2; i <= sn; i++ {
		if n%i == 0 {
			return false
		}
	}
	return true
}

func genPQ() (int, int) {
	primechan := make(chan int)
	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	count := 0
	for i := 0; i < 2; i++ {
		go func() {
			for {
				p := r.Intn(LIMIT)
				if isPrime(p) {
					primechan <- p
					count++
					break
				}
			}
			if count == 2 {
				close(primechan)
			}
		}()
	}
	pq := make([]int, 0, 2)
	for {
		p, ok := <-primechan
		if !ok {
			break
		}
		pq = append(pq, p)
	}
	return pq[0], pq[1]
}

func euler(p int, q int) int {
	return (p - 1) * (q - 1)
}

func exgcd(a int, b int, x *int, y *int) int {
	if b == 0 {
		*x, *y = 1, 0
		return a
	}
	r := exgcd(b, a%b, x, y)
	*x, *y = *y, *x-a/b*(*y)
	return r
}

func genED(eulerN int) (int, int) {
	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	var d, y int
	for {
		e := r.Intn(LIMIT)
		if e < eulerN && exgcd(e, eulerN, &d, &y) == 1 && d > 0 {
			return e, d
		}
	}
}

func encrypt(m int, e int, N int) int {
	c := 1
	m = m % N
	for i := 1; i <= e; i++ {
		c = (c * m) % N
	}
	c = c % N
	return c
}

func decrypt(c int, d int, N int) int {
	return encrypt(c, d, N)
}

func encryptString(data string, e int, N int) []int {
	en := make([]int, 0, 10)
	databyte := []byte(data)
	length := len(databyte)
	for i := 0; i < length; i++ {
		c := encrypt(int(databyte[i]), e, N)
		en = append(en, c)
	}
	return en
}

func decryptString(en []int, d int, N int) string {
	de := make([]byte, 0, 10)
	length := len(en)
	for i := 0; i < length; i++ {
		m := decrypt(en[i], d, N)
		de = append(de, byte(m))
	}
	return string(de)
}
```
