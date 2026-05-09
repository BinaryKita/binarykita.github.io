---
title: 线性优化策略（更新中）
description: 前缀和、差分、双指针、离散化
tags:
 - 前缀和
 - 差分
 - SOS DP
categories: 
 - 算法
---

{% note %}
本文部分内容参考自 [OI-WIKI](https://oi-wiki.org/).
{% endnote %}

{% note %}
本文使用GPT进行润色.
{% endnote %}

# 维护操作类型

在维护数据结构的过程中，我们通常会进行一些操作。大体上，可以将这些操作分为以下几类：

- **单点查询/修改**：对单个位置上的值进行查询/修改
- **区间查询/修改**：对一个区间上的值进行查询/修改

# 前缀和与差分

设一个二元操作为 $+$，其逆操作为 $-$. 数据为 $a_0,a_1,\dots,a_n$.

## 前缀和

当我们需要 $O(1)$ 进行如 $a_l + a_{l+1} + \dots + a_r$ 这样的 **区间查询** 时，可以使用前缀和维护。

我们预先用 $O(n)$ 维护一个数组
$$
S_i = a_0 + a_1 + \dots + a_i
$$

每次查询 $[l,r]$ 区间的二元运算和时，直接计算
$$
S_r - S_{l-1}
$$
即可。

## 差分

当我们需要 $O(1)$ 对数据进行如“给 $[l, r]$ 区间内的每个值都加上 $k$”这样的 **区间修改** 操作，且最后只需要 **很少** 的 **单点查询** 时，可以使用差分维护。

我们预先维护一个差分数组
$$
D_i = a_i - a_{i-1}
$$

每次区间修改时，进行操作
$$
D_l \leftarrow D_l + k,\quad D_{r+1} \leftarrow D_{r+1} - k
$$

单点查询第 $i$ 个值时，只需计算
$$
D_0 + D_1 + \dots + D_i
$$

所有区间修改完成后，对差分数组做一次前缀和即可还原原数组。

{% note %}
显然，前缀和与差分是一对互逆操作。
{% endnote %}

## 条件

并非所有二元运算都能满足前缀和与差分操作。

- 可满足的运算如：加法，模意义下加法，非零数集合上的乘法，异或等；
- 不可满足的运算如：取最大/最小值，最大公因数，最小公倍数，减法等。

结论是，一个二元运算可满足前缀和与差分操作的充要条件为：该运算在对应集合上构成一个 **群**。即满足：

- 结合律
- 单位元
- 逆元

## 二维前缀和/差分

给定一个二维矩阵 $a_{i,j}$，当我们需要 $O(1)$ 求左上角为 $(x_l,y_l)$、右下角为 $(x_r,y_r)$ 的子矩阵之和时，可以使用二维前缀和维护。

类似一维前缀和，我们维护
$$
S_{x,y} = \sum_{0 \le i \le x,\ 0 \le j \le y} a_{i,j}
$$

具体地，递推式为
$$
S_{x,y} = a_{x,y} + S_{x-1,y} + S_{x,y-1} - S_{x-1,y-1}
$$

查询时，我们通过容斥计算
$$
S_{x_r,y_r} - S_{x_l-1, y_r} - S_{x_r, y_l-1} + S_{x_l-1,y_l-1}
$$

{% asset_img two-dimension-sum.png %}

二维差分即二维前缀和逆操作，维护

$$D_{x,y} = a_{x,y} - a_{x-1,y} - a_{x,y-1} + a_{x-1,y-1}$$

若要给左上角为 $(x_l,y_l)$、右下角为 $(x_r,y_r)$ 的子矩阵整体加上 $k$，则：

$$D_{x_l,y_l} \leftarrow D_{x_l,y_l}+k$$

$$D_{x_r+1,y_l} \leftarrow D_{x_r+1,y_l}-k$$

$$D_{x_l,y_r+1} \leftarrow D_{x_l,y_r+1}-k$$

$$D_{x_r+1,y_r+1} \leftarrow D_{x_r+1,y_r+1}+k$$

查询即求 $D$ 的二维前缀和：

$$a_{x,y} = \sum_{0 \le i \le x,\ 0 \le j \le y} D_{i,j}$$

## 树上前缀和

给定一棵树，我们需要很快地求出 $(u,v)$ 两节点构成的简单路径上点权的和。下面我们用 $(u,v)$ 表示 $(u,v)$ 简单路径上点权的和，$fa(x)$ 表示 $x$ 的父节点。

类似一维前缀和，我们维护
$$
S_p = (root,p)
$$

每次查询时，先求出 $lca(u,v)$，答案即
$$
S_u + S_v - S_{lca} - S_{fa(lca)}
$$

这里讨论的是点权路径和。如果维护的是边权，答案为

$$
S_u+S_v-2S_{lca}
$$

树上差分即树上前缀和的逆操作。

## 高维前缀和

对于二维以上的前缀和，即 $k(k > 2)$ 维前缀和，当我们需要求
$$
\sum_{l_1 \le i_1 \le r_1} \sum_{l_2 \le i_2 \le r_2} \cdots \sum_{l_k \le i_k \le r_k} a_{i_1,i_2,\dots,i_k}
$$
时，若依然直接使用容斥法维护，转移复杂度会达到 $O(N2^k)$，其中 $N$ 为数组总大小。

注意到上式本质上是沿着 $k$ 个维度分别做求和。于是我们可以每次固定一维进行求和，做若干次一维前缀和即可。这样预处理复杂度为 $O(kN)$，一般可以接受。

查询时依旧需要容斥，复杂度为 $O(2^k)$。

高维差分即高维前缀和逆操作。每次固定一维进行差分，做若干次一维差分即可。

### 子集和/超集和 DP(Fast Zeta Transform)

当 $k$ 的值很大时，会出现一类 **子集和** (Sum Over Subsets, SOS) 问题，这是高维前缀和的一个特例。

问题描述：考虑大小为 $n$ 的集合的全体子集上定义的函数 $f$，现在要求出其子集和函数 $g$，其满足
$$
g(S) = \sum_{T \subseteq S} f(T)
$$
即 $g(S)$ 等于其所有子集 $T \subseteq S$ 上的函数值 $f(T)$ 之和。

首先，子集和问题可以写成高维前缀和的形式。注意到，$S$ 的子集可以通过状态压缩表示为长度为 $n$ 的 $0/1$ 串 $s$。将串的每一位看作一个维度，则子集的包含关系等价于下标的大小关系，即
$$
T \subseteq S \iff \forall i,\ t_i \le s_i
$$

因此，对子集求和，本质上就是对这个 $n$ 维数组求高维前缀和，时间复杂度为
$$
O(n2^n)
$$

其逆操作依旧可以通过容斥进行。

超集和即对 $n$ 维数组求高维后缀和。

#### 模板

```cpp
//子集和
for (int i = 0; i < n; i++) {
    for (int S = 0; S < (1 << n); S++) {
        if (S & (1 << i)) {
            f[S] += f[S ^ (1 << i)];
        }
    }
}

// 超集和
for (int i = 0; i < n; i++) {
    for (int S = 0; S < (1 << n); S++) {
        if (!(S & (1 << i))) {
            f[S] += f[S ^ (1 << i)];
        }
    }
}
```

#### 例题

解决 FZT 问题的关键在于将问题的约束条件转化为子集/超集关系，从而求解。

更进一步的说，可以将题目条件转化为仅含 $\And, \mid$ 运算的表达式。在很多按位运算计数题中，$\And$ 约束常常可以转化为超集关系，$\mid$ 约束常常可以转化为子集关系。

[CF165E](https://codeforces.com/problemset/problem/165/E)

题意：给定一个数组，对其中每个数，在数组中找到一个与他按位与为 0 的数.

题解：问题可转化为对数组中每个数，在数组中找到一个是他补集的子集的数.

直接进行子集和求解即可.

```cpp
        int n;
        cin >> n;
        vector<int> a(n + 1);
        vector<int> f((1 << 22));
        for(int i = 1; i <= n; i++){
            cin >> a[i];
            f[a[i]] = a[i];
        }
        for(int i = 0; i <= 21; i++){
            for(int j = 1; j < (1 << 22); j++){
                if(j & (1 << i)) f[j] = max(f[j], f[j ^ (1 << i)]);
            }
        }
        for(int i = 1; i <= n; i++){
            if(f[((1 << 22) - 1) ^ a[i]] == 0)
                cout << -1 << " ";
            else
                cout << f[((1 << 22) - 1) ^ a[i]] << " ";
        }
```

[CF449D](https://codeforces.com/problemset/problem/449/D)

题意：给定数组 $a$，求有多少非空子序列使得 $a_{i_1} \And a_{i_2} \And ... \And a_{i_k} = 0$

题解：

考虑求 连与 结果为 $mask$ 的方案数，发现无法求解，转而求解 连与 结果为 **$mask$ 超集** 的方案数.

设 $f_{mask}$ 为 $a$ 中 $mask$ 超集的个数. 通过超集和求解.

设 $g_{mask}$ 为 连与 结果为 $mask$ 超集的方案数，$g_{mask} = 2^{f_{mask}}-1$.

我们对 $g$ 进行高维差分（Fast Möbius Transform）容斥即可得到 连与 结果为 $mask$ 的方案数，答案即 $g_0$.

```cpp
        int n;
        cin >> n;
        vector<int> a(n + 1);
        vector<i64> f(1 << 20);
        for(int i = 1; i <= n; i++){
            cin >> a[i];
            f[a[i]]++;
        }
        for(int i = 0; i < 20; i++){
            for(int j = (1 << 20) - 1; j >= 0; j--){
                if(!(j & (1 << i))) addto(f[j ^ (1 << i)], f[j]);
            }
        }
        vector<i64> pw(n + 1);
        pw[0] = 1;
        for(int i = 1; i <= n; i++) 
            pw[i] = (pw[i - 1] << 1) % mod;
        for(int i = (1 << 20) - 1; i >= 0; i--)
            f[i] = sub(pw[f[i]], 1);
        for(int i = 0; i < 20; i++){
            for(int j = (1 << 20) - 1; j >= 0; j--)
                if(!(j & (1 << i)))  f[j] = sub(f[j], f[j ^ (1 << i)]); 
        }
        cout << f[0] << endl;
```

[CF383E](https://codeforces.com/problemset/problem/383/E)

题意：

给定一个字典，里面有 $n$ 个长度恰好为 3 的单词。
每个单词只由前 24 个小写字母 `a` 到 `x` 组成。

现在要从这 24 个字母里任选一些字母当作 **元音**，其余当作 **辅音**。
一个单词只要 **至少包含一个元音字母**，就算是合法单词。

求所有可能的元音集合对应的“合法单词数的平方”的异或值。

题解：

将元音集合状态压缩成 24 位的 0/1 串 $mask$. 第 $i$ 位为 1 表示第 $i$ 个字母为元音，反之则为辅音.

再将每个单词状压，第 $i$ 位为 1 表示该单词含第 $i$ 个字母.

若一个单词合法，即该单词状压后与 $mask$ 有交集. 进一步转化为单词状压后不为 $mask$ 补集的子集. 由于遍历所有状态时，$mask$ 与其补集一一对应，因此直接遍历补集状态 $i$，累加 $n-f_i$ 即可。

我们求出单词状压后为 $mask$ 补集的子集的数量，最后从总单词数中减去即可.

```cpp
        int n;
        cin >> n;
        vector<string> s(n + 1);
        vector<int> f((1 << 24));
        for(int i = 1; i <= n; i++){
            cin >> s[i];
            int tmp = 0;
            for(auto j : s[i]){
                tmp |= (1 << (j - 'a'));
            }
            f[tmp]++;
        }
        for(int i = 0; i < 24; i++){
            for(int j = 0; j < (1 << 24); j++){
                if(j & (1 << i))    f[j] += f[j ^ (1 << i)];
            }
        }
        i64 ans = 0;
        for(int i = 0; i < (1 << 24); i++)
            ans ^= ((n - f[i]) * (n - f[i]));
        cout << ans << endl;
```

[ABC349F](https://atcoder.jp/contests/abc349/tasks/abc349_f)

题意：

给定一个长度为 $N$ 的正整数序列 $A=(A_1,A_2,\dots,A_N)$ 和一个正整数 $M$，求有多少个 **非空子序列**（不要求连续），其所有元素的 **最小公倍数** 恰好等于 $M$.

题解：

对子序列 $(A_{j_1},A_{j_2},...A_{j_k})$ 和 $M$ 进行质因数分解. 对于所有数产生的质因数集合 $p$, $A_{j_s}$ 的 $p_i$ 次数为 $c_{i_s}$，$M$ 的 $p_i$ 次数为 $c_i$.

由唯一分解定理，子序列所有元素的最小公倍数恰好等于 $M$ 当且仅当：

$$\forall i, \max_{1 \le s \le k} c_{i_s} = c_i$$

$M$ 的质因子最多为 14 个，考虑按将每个质因子的次数是否产生贡献进行状压，得到 14 位 0/1 串. 第 $i$ 位为 1 当且仅当 $c_{i_s} = c_i$. 目标状态为所有位均为 1.

我们先将不能整除 $M$ 的 $A$ 直接排除掉，这样保证 $\forall i,c_{i_s} \le c_i$. 假设 $A$ 状压后为 $a$, 问题转化为求 $a_{j_1} \mid a_{j_2} \mid ... \mid a_{j_k} = 1$ 的方案数.

CF449D 通过求超集和来计算 连与 的方案数，此时求子集和即可求 连或 的方案数. 方法相似，此处不再赘述.

```cpp
        i64 n, m;
        cin >> n >> m;
        vector<i64> a(n + 1);
        for(int i = 1; i <= n; i++)
            cin >> a[i];
        vector<pair<i64, int>> p;
        i64 mm = m;
        for(i64 i = 2; i * i <= m; i++){
            if(m % i)   continue;
            int cnt = 0;
            while(m % i == 0){
                m /= i;
                cnt++;
            }
            p.push_back(make_pair(i, cnt));
        }
        if(m > 1)   p.push_back(make_pair(m, 1));
        int tot = p.size();
        vector<i64> f((1 << tot));
        for(int i = 1; i <= n; i++){
            if(mm % a[i] != 0)   continue;
            int tmp = 0;
            for(int j = 0; j < tot; j++){
                int cnt = 0;
                while(a[i] % p[j].first == 0){
                    a[i] /= p[j].first;
                    cnt++;
                }
                if(cnt == p[j].second)
                    tmp |= (1 << j);
            }
            f[tmp]++;
        }
        for(int i = 0; i < tot; i++){
            for(int j = 0; j < (1 << tot); j++){
                if(j & (1 << i)) addto(f[j ^ (1 << i)], f[j]);
            }
        }
        vector<i64> pw(n + 1);
        pw[0] = 1;
        for(int i = 1; i <= n; i++)
            pw[i] = (pw[i - 1] << 1ll) % mod;
        for(int i = 0; i < (1 << tot); i++)
            f[i] = sub(pw[f[i]], 1ll);
        for(int i = 0; i < tot; i++){
            for(int j = 0; j < (1 << tot); j++){
                if(j & (1 << i)) f[j] = sub(f[j], f[j ^ (1 << i)]);
            }
        }
        cout << f[(1 << tot) - 1] << endl;
```

[CF1208F](https://codeforces.com/problemset/problem/1208/F)

题意：给定数组 $a$, 求最大化 $a_i|(a_j \And a_k)$ 的 $i,j,k(i < j < k)$.

题解：题目中含三元和两种运算，考虑降维或降运算.

固定 $i$，由于最后一层是按位或运算，$a_i$ 为 1 的位必为 1，我们仅需考虑 $a_i$ 不为 1 的位.

贪心地从高到低考虑 $a_i$ 不为 1 的位，判断是否存在 $j,k$ 使得该位为 1. 

我们通过超集和预处理出 $mx1_{mask}$ 表示为 $mask$ 超集的最大 $a$ 下标， $mx2_{mask}$ 表示为 $mask$ 超集的次大 $a$ 下标. 若 $mx2_{mask} > i$ 则表示存在 $j,k$ 使得 $a_j \And a_k$ 为 $mask$ 的超集，也就能判断是否能使得贪心到的该位为 1.

```cpp
        int n;
        cin >> n;
        vector<int> a(n + 1);
        vector<int> mx1((1 << 21)), mx2((1 << 21));
        for(int i = 1; i <= n; i++){
            cin >> a[i];
            if(mx1[a[i]]){
                mx2[a[i]] = mx1[a[i]];
                mx1[a[i]] = i;
            }else{
                mx1[a[i]] = i;
            }
        }
        for(int i = 0; i < 21; i++)
            for(int j = (1 << 21) - 1; j >= 0; j--){
                if(!(j & (1 << i))){
                    vector<int> tmp(4);
                    tmp[0] = mx1[j], tmp[1] = mx2[j], tmp[2] = mx1[j ^ (1 << i)], tmp[3] = mx2[j ^ (1 << i)];
                    sort(tmp.begin(), tmp.end());
                    mx1[j] = tmp[3];
                    mx2[j] = tmp[2];
                }
            }
        int ans = 0;
        for(int i = 1; i <= n - 2; i++){
            int s = ((1 << 21) - 1) ^ a[i];
            int res = 0;
            for(int j = 20; j >= 0; j--){
                if(((1 << j) & s) && mx2[(res | (1 << j))] > i){
                    res |= (1 << j);
                }
            }
            ans = max(ans, (res | a[i]));
        }
        cout << ans << endl;
```

