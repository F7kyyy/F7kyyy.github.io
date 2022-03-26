---
title: 计算小于等于N的质数的并行算法
date: 2022-03-17 13:35:05
tags: 
    - 算法
    - 笔记
    - C++
categories: 并行计算
math: true
index_img: /img/article/prime.jpg
---

## 1. 遍历每一个数，判断是否是质数

###  朴素方法 是否可以被整除

我们判断$N$是否是质数时，不选要考虑$\leq N$的所有情况,只需要考虑$N$是否可以被小于等于$\sqrt{N}$的数整除就可以了，因此代码如下(此时不考虑并行情况)

```c++
bool ifprime(int n)
{
    for (int k = 2; k <= sqrt(n); k++)
    {
        if (n % k == 0)
            return false;
    }
    return true;
}

void getPrime(vector<long> &prime)
{
    for (int i = 2; i <= SIZE; i++)
    {
        if (ifprime(i))
            prime.emplace_back(i);
    }
}

int main()
{
    vector<long> prime;
    double t = omp_get_wtime();
    // add your codes begin
    getPrime(prime);
    // add your codes end
    t = omp_get_wtime() - t;
    printf("time %f %ld\n", t, long(SIZE));
    printf("\nsize %ld\n", prime.size());
}
```

**此时运行时间为**

>time=0.477831

时间后的1000000时参数SIZE，含义是$\leq 1000000$中有78498个质数

![202203171647230](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261347360.png)

而在使用openmp并行后，从2-SIZE，每个线程分配一部分，判断是否为质数。

```c++
bool ifprime(int n)
{
    for (int k = 2; k <= sqrt(n); k++)
    {
        if (n % k == 0)
            return false;
    }
    return true;
}

void getPrime(vector<long> &prime)
{
    // 并行的写入vector
#pragma omp parallel
    {
        vector<int> vec_private;
#pragma omp for nowait schedule(static)
        for (int i = 2; i <= SIZE; i++)
        {
            if (ifprime(i))
                vec_private.emplace_back(i);
        }
#pragma omp critical
        prime.insert(prime.end(), vec_private.begin(), vec_private.end());
    }
}
```

**此时运行时间为**

> time=0.033235

![202203171651834](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261348208.png)

### 优化判断质数函数

首先，我们应该知道一个关于质数分布的规律：**大于等于5的质数一定和6的倍数相邻**。例如5和7，11和13,17和19,**反之是不一定成立的**

证明：令x≥1，将大于等于5的自然数表示如下：··· 6x-1，6x，6x+1，6x+2，6x+3，6x+4，6x+5，6(x+1），6(x+1)+1 ···

可以看到，不和6的倍数相邻的数为6x+2，6x+3，6x+4，由于2(3x+1)，3(2x+1)，2(3x+2)，所以它们一定不是素数，再除去6x本身，显然，素数要出现只可能出现在6x的相邻两侧。因此在5到$\sqrt{n}$中每6个数只判断2个，时间复杂度O($\frac{\sqrt{n}}{3}$),C++代码如下

```c++
bool ifprime(int n)
{
    if (n == 2 || n == 3)
        return 1;
    if (n % 6 != 1 && n % 6 != 5)
        return 0;
    for (int k = 5; k <= floor(sqrt(n)); k += 6)
        if (n % k == 0 || n % (k + 2) == 0)
            return 0;
    return 1;
}
```

串行 **此时运行时间为**

>time=0.156363

![202203171659060](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261348573.png)

并行 **此时运行时间为**

>time=0.015401

![202203171713716](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261348471.png)

## 2. 使用筛法

我们可以标记所有合数，然后求质数

```c++
long sign[SIZE + 1];

void getPrime(vector<long> &prime)
{
  int N = sqrt(SIZE);
  for (int i = 2; i <= N; i++)
  {
    if (!sign[i])
      for (int j = i; j <= SIZE / i; j++)
        sign[i * j] = true;
}
```

sign[i]=true表示i为合数，当一个数i是质数时，我们标记$i*j$为合数，$i*j\leq SIZE$

### 串行

**此时运行时间为**

>time=0.021131

![202203171712942](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261349127.png)

### 在遍历时并行

```c++
  int N = sqrt(SIZE);
  // 并行的使用筛法找出所有合数
#pragma omp parallel for num_threads(NUM_THREADS)
  for (int i = 2; i <= N; i++)
  {
    if (!sign[i])
      for (int j = i; j <= SIZE / i; j++)
        sign[i * j] = true;
  }
```

**此时运行时间为**

>time=0.012204

![202203171716363](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261349515.png)

### 在寻找合数时并行

```c++
  int N = sqrt(SIZE);
// 并行的使用筛法找出所有合数
  for (int i = 2; i <= N; i++)
  {
    if (!sign[i])
    #pragma omp parallel for num_threads(NUM_THREADS)
      for (int j = i; j <= SIZE / i; j++)
        sign[i * j] = true;
  }
```

**此时运行时间为**

> time=0.006109

![202203171718941](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261349256.png)

## 3. 时间对比

| 方法                | 时间     |
| ------------------- | -------- |
| 朴素方法 串行       | 0.477831 |
| 朴素方法 并行       | 0.033235 |
| 优化判断质数 串行   | 0.156363 |
| 优化判断质数 并行   | 0.015401 |
| 筛法 串行           | 0.021131 |
| 筛法 遍历时并行     | 0.012204 |
| 筛法 寻找合数时并行 | 0.006109 |

## 4. 多线程写入vector

在完成上述问题时，遇到过循环中多线程无法对同一个vector进行写操作，有以下几种解决方法

```c++
std::vector<int> vec;
#pragma omp parallel
{
    std::vector<int> vec_private;
    #pragma omp for nowait //fill vec_private in parallel
    for(int i=0; i<100; i++) {
        vec_private.push_back(i);
    }
    #pragma omp critical
    vec.insert(vec.end(), vec_private.begin(), vec_private.end());
}
```

OpenMP 4.0允许使用reduction

```c++
#pragma omp declare reduction (merge : std::vector<int> : omp_out.insert(omp_out.end(), omp_in.begin(), omp_in.end()))

std::vector<int> vec;
#pragma omp parallel for reduction(merge: vec)
for(int i=0; i<100; i++) 
    vec.push_back(i);
```

较为详细的可以这样

```c++
std::vector<int> vec;
#pragma omp parallel
{
    std::vector<int> vec_private;
    #pragma omp for nowait schedule(static)
    for(int i=0; i<N; i++) { 
        vec_private.push_back(i);
    }
    #pragma omp for schedule(static) ordered
    for(int i=0; i<omp_get_num_threads(); i++) {
        #pragma omp ordered
        vec.insert(vec.end(), vec_private.begin(), vec_private.end());
    }
}
```

