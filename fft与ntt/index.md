# FFT与NTT


<!--more-->

# FFT与NTT
大家都知道NTT是FFT的优化，实际上也就是将FFT难搞的复数换成原根。这次来仔细分析一下FFT与NTT。（个人向）

## FFT
### 前言
又是众所周知，FFT是用来加速二项式乘法的，利用精心构造出的分治算法将复杂度优化到$O(nlogn)$。

还是众所周知，$FFT=DFT\rightarrow 点值相乘\rightarrow IDFT$，即将多项式的系数表示法转为点值表示法，算完再转回去。

如果是一般的系数表示法，两个多项式相乘每个系数之间都要乘一次，复杂度显然是$O(nm)$。

多项式$C$的系数$c_i$满足$c_i=\sum^i_{j=0}a_jb_{i-j}$，这是一个加法形式的卷积。

假如要求两个$n$次多项式$F(x),G(x)$的积$H(x)$，用点值表示就是
$$
(x_0,F_{y_0}\times H_{y_0}),\dots,(x_{2n},F_{y_{2n}}\times G_{y_{2n}})
$$
就是横坐标代入两个多项式，纵坐标就是两个系数的积。

回顾一个东西叫**单位根**，也就是在复平面上单位圆上的所有复数。由于所有单位根的模长都为1，所以两个单位根相乘所得复数仍在单位圆上，也就是仍为单位根。而一个$n-1$次多项式的单位根就是满足$x^n=1$的复数$x$。

假设$n$个单位根，幅角都为$\frac{1}{n}\times 360^\circ$。用$\omega^i_n$表示$n$次单位根中幅角为$\frac{1}{n}\times 360^\circ$的单位根，可以得到以下性质。
$$
\begin{align*}
\omega^k_n&=\omega_n^{k \bmod n} \\\
\omega^{2k}_{2n}&=\omega^k_n \\\
\omega^j_n\times\omega^k_n&=\omega^{j+k}_n \\\
(\omega^j_n)^k&=\omega^{jk}_n \\\
(\omega^k_n)^{-1}&=\omega^{-k}_n=\omega^{n-k}_n \\\
\omega^{\frac{n}{2}}_n&=-1 \\\
\omega^{\frac{n}{2}+k}_n&=\omega^k_n\times \omega^{\frac{n}{2}}_n=-\omega^k_n 
\end{align*}
$$
每个单独看都很简单，跟离散一样，放在一起就头疼了。还好只要知道怎么用这些性质推FFT就行了。

### DFT
将特殊的$x$代入两个多项式进行计算，就可以消掉很多项。每次代入不同的$x$就能消去不同的项，从而加速计算。而在FFT中就是代入$\omega_n^k$。

首先将多项式按奇偶分组，将奇数组提出一个$x$就可以得到两边是一样的次数，但系数被分到两边。

对两个分组分别建立新的函数去表示原函数，得到

$$
f(x)=G(x^2)
+x\times H(x^2)
$$

显然$G(x),H(x)$是偶函数，利用单位根性质$\omega^{\frac{n}{2}+k}_n=-\omega^k_n$可以得到$\omega^k_n$和$\omega^{k+\frac{n}{2}}_n$的$G(x^2),H(x^2)$对应相等。

偷个懒，把这两个单位根代入$f(x)$就可以得到范围缩小的表达式。
$$
f(\omega^k_n)=G(\omega^k_{\frac{n}{2}})+\omega^k_n\times H(\omega^k_{\frac{n}{2}})
$$
$$
f(\omega_n^{k+\frac{n}{2}})=G(\omega^k_{\frac{n}{2}})-\omega^k_n\times H( \omega^k_{\frac{n}{2}})
$$

对$G(x),H(x)$分别递归即可。

使用前须将多项式补成$2^m$项形式以满足单位根性质。
### 位逆序置换（bit-reversal permutation），也称蝴蝶变换
为了优化递归的内存，先模拟递归在原数组中拆分系数，再倍增合并算出来的值。

可以发现规律原序列每个数用二进制表示，翻转后就是最终的位置。

有$O(n)$实现的技巧，就记个板子吧。

```cpp
// 同样需要保证 len 是 2 的幂
// 记 rev[i] 为 i 翻转后的值
void change(Complex y[], int len) {
  for (int i = 0; i < len; ++i) {
    rev[i] = rev[i >> 1] >> 1;
    if (i & 1) {  // 如果最后一位是 1，则翻转成 len/2
      rev[i] |= len >> 1;
    }
  }
  for (int i = 0; i < len; ++i) {
    if (i < rev[i]) {  // 保证每对数只翻转一次
      swap(y[i], y[rev[i]]);
    }
  }
  return;
}
```

### IDFT
数学证明比较复杂就不证了，反正就是把$\omega^k_n$转为$\omega^{-k}_n$再除以$n$就行了。

## NTT
解决多项式乘法带模数的情况，因为复数无法取模，所以需要改为原根。

今天主要解决原根是如何取代单位根的问题。

显然，原根需要满足单位根的几条性质。

原根：对于一个素数$P$，若有一个数$G$使得$G^1,G^2,G^3,\dots,G^{p-2}(\mod P)$互不相同，则$G$是$P$的一个原根。

懒得证明了，深入讨论原根性质都要跑到群论去了。

## 结语
应该不会有改算法内部的题目，主要考察运用。目前还没有遇到过什么题可以自己想到要用NTT，慢慢积累吧。

---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.cn/fft%E4%B8%8Entt/  

