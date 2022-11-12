# Codeforces 1753B


<!--more-->

## 题意

$\sum_{i=1}^{n}a_i!$能否被 $x!$整除。

## 题解

首先容易想到预处理每个数字$k$ 出现的次数$cnt_k$，$\sum_{i=1}^n a_i!=\sum_{k=1}^x k!\times cnt_k$。

其次对于阶乘有$(k+1)\times k!=(k+1)!$，那么就可以进行一个转化$cnt_k-=k+1,cnt_{k+1}+=1$。

对$\forall k \in [1,x-1]$进行这个操作，使得所有的$cnt_k \leq k$。其中$k=x$不用考虑，因为$x!|x!$。

最后观察一下$\sum_{k=1}^{x-1} k!\times cnt_k$的范围，最大值在$\forall k,cnt_k=k$时取到，原式为$\sum_{k=1}^{x-1} k!\times k$。而$k!\times k=(k+1)!-k!$，通过裂项相消可得结果等于$x!-1$，而$[1,x!-1]$内没有数能被$x!$整除。也就是说，如果原式要被$x!$整除的话，$\sum_{k=1}^{x-1} k!\times cnt_k$只能为0。

那么做法就是所有操作做完后扫一遍，如果$\forall k \in [1,x-1],cnt_k>0$，那么就不能整除，反之可以。

## 总结

这题的风格非常接近数列题，虽然不复杂但容易想偏，让我回想起了高中的日子。

感觉阶乘大多与数列相关，因为只能拆开去寻求代数上的一些性质。


---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.cn/codeforces-1753b/  

