# 线段树及其扩展


<!--more-->

# 线段树及其扩展

线段树是一种维护区间性质，满足符合结合律的运算的区间修改的数据结构。

## 基本实现  

### 朴素线段树

```cpp
#include<iostream>
using namespace std;
using ll=long long;
#define int ll
const int maxn=1e5+10;
int a[maxn],tree[maxn<<2],mark[maxn<<2];
void build(int node,int l,int r)
{
    if(l==r)tree[node]=a[l];
    else{
        int mid=(l+r)>>1;
        build(node<<1,l,mid);
        build(node<<1|1,mid+1,r);
        tree[node]=tree[node<<1]+tree[node<<1|1];
    }
}
void push_down(int node,int len)
{
    mark[node<<1]+=mark[node];
    mark[node<<1|1]+=mark[node];
    tree[node<<1]+=mark[node]*(len-(len>>1));
    tree[node<<1|1]+=mark[node]*(len>>1);
    mark[node]=0;
}
void update(int node,int l,int r,int L,int R,int d)
{
    if(L>r||R<l)return;
    else if(l>=L&&r<=R){
        tree[node]+=(r-l+1)*d;
        if(r>l)mark[node]+=d;
    }
    else{
        int mid=(l+r)>>1;
        push_down(node,r-l+1);
        update(node<<1,l,mid,L,R,d);
        update(node<<1|1,mid+1,r,L,R,d);
        tree[node]=tree[node<<1]+tree[node<<1|1];
    }
}
int query(int node,int l,int r,int L,int R)
{
    if(L>r||R<l)return 0;
    else if(l>=L&&r<=R)return tree[node];
    else{
        int mid=(l+r)>>1;
        push_down(node,r-l+1);
        return query(node<<1,l,mid,L,R)+query(node<<1|1,mid+1,r,L,R);
    }
}
signed main()
{
    int n,m;
    cin>>n>>m;
    for(int i=1;i<=n;i++)cin>>a[i];
    build(1,1,n);
    int oper,x,y,k;
    for(int i=1;i<=m;i++){
        cin>>oper>>x>>y;
        if(oper==1){
            cin>>k;
            update(1,1,n,x,y,k);
        }
        else{
            cout<<query(1,1,n,x,y)<<endl;
        }
    }
    return 0;
}
```

查询与修改代码类似，三步

1. 查询（修改）区间与当前区间无交集，直接返回
2. 当前区间完全被包含在查询（修改）区间内，对当前树节点（和懒标记）修改或直接返回当前树节点
3. 当前区间与查询（修改）区间有交集，**下推懒标记后**递归处理左右儿子。

#### 双tag线段树

有两种操作，需要两个懒标记，需要定义优先级。注意两种操作对值的影响和先后顺序，并且做出修改。

```cpp
#include<iostream>
using namespace std;
using ll=long long;
const int maxn=1e5+10;
ll a[maxn],tree[maxn<<2],add[maxn<<2],mul[maxn<<2],p;
void build(int node,int l,int r)
{
    add[node]=0,mul[node]=1;
    if(l==r)tree[node]=a[l];
    else{
        int mid=(l+r)>>1;
        build(node<<1,l,mid);
        build(node<<1|1,mid+1,r);
        tree[node]=(tree[node<<1]+tree[node<<1|1])%p;
    }
}
void push_down(int node,int len)
{
    tree[node<<1]=(tree[node<<1]*mul[node]+add[node]*(len-(len>>1)))%p;
    tree[node<<1|1]=(tree[node<<1|1]*mul[node]+add[node]*(len>>1))%p;
    mul[node<<1]=(mul[node<<1]*mul[node])%p;
    mul[node<<1|1]=(mul[node<<1|1]*mul[node])%p;
    add[node<<1]=(add[node<<1]*mul[node]+add[node])%p;
    add[node<<1|1]=(add[node<<1|1]*mul[node]+add[node])%p;
    mul[node]=1,add[node]=0;
}
void mulupdate(int node,int l,int r,int L,int R,int k)
{
    if(L>r||R<l)return;
    else if(L<=l&&R>=r){
        tree[node]=(tree[node]*k)%p;
        mul[node]=(mul[node]*k)%p;
        add[node]=(add[node]*k)%p;
    }
    else{
        int mid=(l+r)>>1;
        push_down(node,r-l+1);
        mulupdate(node<<1,l,mid,L,R,k);
        mulupdate(node<<1|1,mid+1,r,L,R,k);
        tree[node]=(tree[node<<1]+tree[node<<1|1])%p;
    }
}
void addupdate(int node,int l,int r,int L,int R,int k)
{
    if(L>r||R<l)return;
    else if(L<=l&&R>=r){
        add[node]=(add[node]+k)%p;
        tree[node]=(tree[node]+k*(r-l+1))%p;
    }
    else{
        int mid=(l+r)>>1;
        push_down(node,r-l+1);
        addupdate(node<<1,l,mid,L,R,k);
        addupdate(node<<1|1,mid+1,r,L,R,k);
        tree[node]=(tree[node<<1]+tree[node<<1|1])%p;
    }
}
ll query(int node,int l,int r,int L,int R)
{
    if(L>r||R<l)return 0;
    else if(L<=l&&R>=r){
        return tree[node];
    }
    else{
        int mid=(l+r)>>1;
        push_down(node,r-l+1);
        return (query(node<<1,l,mid,L,R)+query(node<<1|1,mid+1,r,L,R))%p;
    }
}
int main()
{
    ios::sync_with_stdio(false);
    int n,m;
    cin>>n>>m>>p;
    for(int i=1;i<=n;i++)cin>>a[i];
    build(1,1,n);
    int oper,x,y,k;
    for(int i=1;i<=m;i++)
    {
        cin>>oper>>x>>y;
        if(oper==1){
            cin>>k;
            mulupdate(1,1,n,x,y,k);
        }
        else if(oper==2){
            cin>>k;
            addupdate(1,1,n,x,y,k);
        }
        else{
            cout<<query(1,1,n,x,y)<<endl;
        }
    }
    return 0;
}
```

#### jls线段树

同时维护两个值，在修改时比较两个值，如果修改不会影响结果则直接退出，否则分类讨论修改区间和。
![](/images/segment_tree_1.png)
![](/images/segment_tree_2.png)

```cpp
#include<iostream>
using namespace std;
const int maxn=1e6+10;
int a[maxn];
struct node{
    int l,r,sum,mx,se,c;
#define l(i) tree[i].l
#define r(i) tree[i].r
#define c(i) tree[i].c
#define mx(i) tree[i].mx
#define se(i) tree[i].se
#define sum(i) tree[i].sum
#define ls rt<<1
#define rs rt<<1|1
}tree[maxn<<2];
void push_up(int rt)
{
    sum(rt)=sum(ls)+sum(rs);
    mx(rt)=max(mx(ls),mx(rs));
    se(rt)=max(se(ls),se(rs));
    c(rt)=0;
    if(mx(ls)!=mx(rs))se(rt)=max(se(rt),min(mx(ls),mx(rs)));
    if(mx(rt)==mx(ls))c(rt)+=c(ls);
    if(mx(rt)==mx(rs))c(rt)+=c(rs);
}
void build(int rt,int l,int r)
{
    l(rt)=l,r(rt)=r;
    if(l==r){
        mx(rt)=sum(rt)=a[l],se(rt)=-1;
        c(rt)=1;
        return;
    }
    int mid=(l+r)>>1;
    build(ls,l,mid);
    build(rs,mid+1,r);
    push_up(rt);
}
void update(int rt,int k){
    if(k>=mx(rt))return;
    sum(rt)-=(mx(rt)-k)*c(rt);
    mx(rt)=k;
}
void push_down(int rt){
    update(ls,mx(rt)),update(rs,mx(rt));
}
void modify(int rt,int x,int y,int k)
{
    if(k>=mx(rt))return;
    if(l(rt)>=x&&r(rt)<=y&&k>se(rt)) {
        update(rt,k);
        return;
    }
    int mid=l(rt)+r(rt)>>1;
    push_down(rt);
    if(x<=mid)modify(ls,x,y,k);
    if(y>mid)modify(rs,x,y,k);
    push_up(rt);
}
int getmax(int rt,int x,int y)
{
    if(l(rt)>=x&&r(rt)<=y)return mx(rt);
    push_down(rt);
    int ans=0;
    int mid=l(rt)+r(rt)>>1;
    if(x<=mid)ans=max(ans,getmax(ls,x,y));
    if(y>mid)ans=max(ans,getmax(rs,x,y));
    return ans;
}
int getsum(int rt,int x,int y)
{
    if(l(rt)>=x&&r(rt)<=y)return sum(rt);
    push_down(rt);
    int ans=0;
    int mid=l(rt)+r(rt)>>1;
    if(x<=mid)ans+=getsum(ls,x,y);
    if(y>mid)ans+=getsum(rs,x,y);
    return ans;
}
int main()
{
    int t;
    cin>>t;
    while(t--)
    {
        int n,q;
        cin>>n>>q;
        for(int i=1;i<=n;i++)cin>>a[i];
        build(1,1,n);
        int op,x,y,z;
        while(q--){
            cin>>op>>x>>y;
            if(op==0)cin>>z,modify(1,x,y,z);
            else if(op==1)cout<<getmax(1,x,y)<<endl;
            else if(op==2)cout<<getsum(1,x,y)<<endl;
        }
    }
    return 0;
}
```

#### 动态开点线段树

在需要用到左右儿子的时候再分配下标，在push_down（或update反正是修改时）中开点。

```cpp
#include<iostream>
#include<algorithm>
using namespace std;
const int maxn=1e5+10;
struct node{
    int s,lc,rc;
}tree[maxn<<2];
int tot=1,root=0;
void update(int &node,int l,int r,int x,int k)
{
    if(!node)node=++tot;
    if(l==r)tree[node].s+=k;
    else{
        int mid=(l+r)>>1;
        if(x<=mid)update(tree[node].lc,l,mid,x,k);
        else update(tree[node].rc,mid+1,r,x,k);
        tree[node].s=tree[tree[node].lc].s+tree[tree[node].rc].s;
    }
}
int query(int node,int l,int r,int L,int R)
{
    if(!node)return 0;
    else if(L==l&&R==r)return tree[node].s;
    else{
        int mid=(l+r)>>1;
        if(R<=mid) return query(tree[node].lc,l,mid,L,R);
        else if(L>mid) return query(tree[node].rc,mid+1,r,L,R);
        return query(tree[node].lc,l,mid,L,mid)+query(tree[node].rc,mid+1,r,mid+1,R);
    }
}
int main()
{
    int n;
    cin>>n;
    int ans=0;
    for(int i=1;i<=n;i++){
        int a;
        cin>>a;
        ans+= query(root,1,1e5,a+1,1e5);
        update(root,1,1e5,a,1);
    }
    cout<<ans<<endl;
    return 0;
}
```

由于不用build，而是在修改过程中建树，所以建的位置需要记录，所以update中传入了引用，为了在函数中改变值。

#### 权值线段树

开桶保存每一个数出现的次数，用线段树去维护桶即为权值线段树。由于需要按值域去开数组，所以常常动态开点。可以 $O(logv)$ （ $v$ 为值域）地查询某个范围内的数出现的总次数。不仅如此，它还可以 $O(logv)$ 地求得第 k 大的数。

```cpp
int kth(int k, int p = 1, int cl = L, int cr = R) // 求指定排名的数
{
    if (cl == cr)
        return cl;
    int mid = (cl + cr - 1) / 2;
    if (val(ls(p)) >= k)
        return kth(k, ls(p), cl, mid); // 往左搜
    else
        return kth(k - val(ls(p)), rs(p), mid + 1, cr); // 往右搜
}
```

### zkw线段树

非递归式线段树，把线段树开成满二叉树，从叶子结点往上处理，缩小传统线段树的常数与代码量。**无法处理有运算优先级的问题，如双tag。**

```cpp
#include<iostream>
using namespace std;
using ll=long long;
const int maxn=1e5+10;
int n,m,N=1;
ll tree[maxn<<2],add[maxn<<2];
void build() {
    for(; N <= n+1; N <<= 1);
    for(int i=N+1;i<=N+n;i++)cin>>tree[i];
    for(int i=N-1;i>=1;i--) tree[i] = tree[i << 1] + tree[i << 1 | 1];
}
void update(int s, int t, int k) {
    int lNum=0, rNum=0, nNum=1;
    //lNum:  s一路走来已经包含了几个数
    //rNum:  t一路走来已经包含了几个数
    //nNum:  本层每个节点包含几个数
    for(s = N+s-1, t = N+t+1; s^t^1; s >>= 1, t >>= 1, nNum <<= 1) {
        //更新tree
        tree[s] += k*lNum;
        tree[t] += k*rNum;
        //处理add
        if(~s&1) {add[s^1] += k; tree[s^1] += k*nNum; lNum += nNum;}
        if(t&1) {add[t^1] += k; tree[t^1] += k*nNum; rNum += nNum;}
    }
    //更新上层tree
    for(; s; s >>= 1, t >>= 1) {
        tree[s] += k*lNum;
        tree[t] += k*rNum;
    }
}
ll query(int s, int t){
    ll lNum=0, rNum=0, nNum=1;
    ll ans=0;
    for(s = N+s-1, t = N+t+1; s^t^1; s >>= 1, t >>= 1, nNum <<= 1) {
        //根据标记更新
        if(add[s]) ans += add[s]*lNum;
        if(add[t]) ans += add[t]*rNum;
        //常规求和
        if(~s&1) {ans += tree[s^1]; lNum += nNum;}
        if(t&1) {ans += tree[t^1]; rNum += nNum;}
    }
    //处理上层标记
    for(; s; s >>= 1, t >>= 1) {
        ans += add[s]*lNum;
        ans += add[t]*rNum;
    }
    return ans;
}
int main()
{
    ios::sync_with_stdio(false);
    cin>>n>>m;
    build();
    int oper,x,y,k;
    for(int i=1;i<=m;i++){
        cin>>oper>>x>>y;
        if(oper==1){
            cin>>k;
            update(x,y,k);
        }
        else{
            cout<<query(x,y)<<endl;
        }
    }
    return 0;
}
```

### 主席树（可持久化权值线段树）

为了保存每一次的历史记录，每次修改实际上是新建了一棵线段树，并不对原树做任何修改。但为了合理的时空复杂度，可以发现每次修改只会改一部分结点，剩下的结点可以共用。因此可持久化只需要把修改改为动态开点，只新建需要修改的点。每次修改都会新增一个根节点，因此查询时需要选择查询哪个根节点。
在建立第一个版本的线段树时应当一次建好，之后再动态开点。
虽然我不喜欢用一堆宏，但是这里不用宏代码会显得逻辑不够清晰且啰嗦冗余，所以算了。

#### 单点修改

```cpp
#include<iostream>
#include<algorithm>
using namespace std;
const int maxn=1e6+10;
int a[maxn],roots[maxn],n,m,cnt=1;
#define ls(x) tree[x].ls
#define rs(x) tree[x].rs
#define val(x) tree[x].val
#define mark(x) tree[x].mark;
struct node{
    int val,ls,rs;
}tree[20000000];
void build(int p=1,int l=1,int r=n)
{
    if(l==r)val(p)=a[l];
    else{
        ls(p)=++cnt,rs(p)=++cnt;
        int mid=(l+r)>>1;
        build(ls(p),l,mid);
        build(rs(p),mid+1,r);
        val(p)=val(ls(p))+val(rs(p));
    }
}
void update(int x,int k,int p,int q,int l=1,int r=n)
{
    if(l==r)val(q)=k;
    else{
        ls(q)=ls(p),rs(q)=rs(p);
        int mid=(l+r)>>1;
        if(x<=mid)
            ls(q)=++cnt,update(x,k,ls(p),ls(q),l,mid);
        else
            rs(q)=++cnt,update(x,k,rs(p),rs(q),mid+1,r);
        val(q)=val(ls(q))+val(rs(q));
    }
}
int query(int p,int L,int R,int l=1,int r=n)
{
    if(L>r||R<l)return 0;
    else if(L<=l&&R>=r)return val(p);
    else{
        int mid=(l+r)>>1;
        return query(ls(p),L,R,l,mid)+query(rs(p),L,R,mid+1,r);
    }
}
int main()
{
    ios::sync_with_stdio(false);
    cin>>n>>m;
    for(int i=1;i<=n;i++)cin>>a[i];
    build();
    roots[0]=1;
    int v,oper,loc,k;
    for(int i=1;i<=m;i++)
    {
        cin>>v>>oper>>loc;
        if(oper==1){
            cin>>k;
            roots[i]=++cnt;
            update(loc,k,roots[v],roots[i]);
        }
        else{
            roots[i]=roots[v];
            cout<<query(roots[v],loc,loc)<<endl;
        }
    }
    return 0;
}
```

#### 区间修改

使用标记永久化技术，即不再向下传递标记，而是查询时带着标记。

```cpp
ll query(int l, int r, int p, int cl = 1, int cr = n, ll mk = 0) // 区间查询
{
    if (cl > r || cr < l)
        return 0;
    else if (cl >= l && cr <= r)
        return val(p) + mk * (cr - cl + 1); // 加上带的标记
    else
    {
        int mid = (cl + cr) / 2;
        return query(l, r, ls(p), cl, mid, mk + mark(p)) + query(l, r, rs(p), mid + 1, cr, mk + mark(p)); // 带着标记传递
    }
}
void update(int l, int r, int d, int p, int q, int cl = 1, int cr = n) // 区间修改
{
    ls(q) = ls(p), rs(q) = rs(p), mark(q) = mark(p); // 复制节点
    if (cl >= l && cr <= r)
    {
        if (cr > cl)
            mark(q) += d;
    }
    else
    {
        int mid = (cl + cr) / 2;
        if (cl <= r && mid >= l) // 提前进行判断，以免新建不必要的节点
            ls(q) = ++cnt, update(l, r, d, ls(p), ls(q), cl, mid);
        if (mid + 1 <= r && cr >= l)
            rs(q) = ++cnt, update(l, r, d, rs(p), rs(q), mid + 1, cr);
    }
    val(q) = val(p) + (min(cr, r) - max(cl, l) + 1) * d; // 根据需要更新的区间长度计算当前节点的值
}
```

#### 求静态区间第k小数

摘自[算法学习笔记(50): 可持久化线段树 - 知乎](https://zhuanlan.zhihu.com/p/250565583)
先建立一棵全0的树（因为要可持久化所以不能动态开点）：

```cpp
void build(int l = 1, int r = n, int p = 1) // 建树
{
    val(p) = 0;
    if (l != r)
    {
        ls(p) = ++cnt, rs(p) = ++cnt;
        int mid = (l + r) / 2;
        build(l, mid, ls(p));
        build(mid + 1, r, rs(p));
    }
}
```

把原数组离散化：

```cpp
int C[MAXN], L[MAXN], ori[MAXN];
void discretize(int A[], int n)
{
    memcpy(C, A, sizeof(int) * n);     // 复制
    sort(C, C + n);                    // 排序
    int l = unique(C, C + n) - C;      // 去重
    for (int i = 0; i < n; ++i)
    {
        L[i] = lower_bound(C, C + l, A[i]) - C + 1; // 查找
        ori[L[i]] = A[i]; // 保存离散化后的数对应的原数
    }
}
```

然后按原区间中的顺序，把离散化后的数据一个一个地插入可持久化的权值线段树：

```cpp
for (int i = 0; i < n; ++i)
{
    roots[i + 1] = ++cnt;
    update(L[i], 1, roots[i], roots[i + 1]);
}
```

重点来了：我们已经有了求[1, r]内第k小数的方法（查询相应历史版本即可），而求[l, r]内第k小数，只需对kth函数稍作修改。注意，我们在求kth时会用到当前左儿子对应的区间和，现在要排除[1, l-1]，那么把这部分和减去即可。

```cpp
int kth(int k, int p, int q, int cl = 1, int cr = n) // 求指定排名的数
{
    if (cl == cr)
        return ori[cl];
    int mid = (cl + cr) / 2;
    if (val(ls(q)) - val(ls(p)) >= k)
        return kth(k, ls(p), ls(q), cl, mid); // 往左搜
    else
        return kth(k - (val(ls(q)) - val(ls(p))), rs(p), rs(q), mid + 1, cr); // 往右搜
}
// 在main函数中
printf("%d\n", kth(k, roots[l - 1], roots[r]));
```

### 李超线段树

一种用于维护平面直角坐标系内线段关系的数据结构。（也许计算几何能用？某年icpc出到了可以用这个做）

## 基本应用

### 线段树染色问题

-1表示染了多种颜色，0表示未染色，其它数字表示颜色种类。
似乎只要用父节点和懒标记的值区间覆盖就行了。  
修改时要考虑下递归之后的回溯，对于两个子区间的颜色是什么情况来更新父区间的颜色情况。
查询时对于一个区间内颜色的情况，未染色直接返回，染了多种颜色则递归查。跨区间的也递归查。

### 扫描线

经典应用，解决平面矩形覆盖的面积和问题。（也能够扩展到空间和其它图形）。
思想是水平或竖直方向扫描，以下以水平方向扫描为例，也就是矩形的竖边作为扫描线。从左往右扫，将整个图形分成若干个小矩形相加（如果上下两个矩形分开，视为合并在一起的就好）。那么，扫过的距离就是小矩形的宽，只有长在不断变化。使用线段树维护扫过区间内被覆盖的次数和长度，然后相乘即可。  
具体思路如下：首先按照边来读入，对左边的边标记为1，右边的标记为-1，然后离散化，使用二分来查找对应的边。建立线段树，其中的每一个点表示一条线段，维护该线段的长度len与被多少个矩形覆盖的cnt。按照每一条边去更新线段树，每更新一条边在答案上累加。（扫描线）最后输出答案即可。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;
#define p1 (p<<1)
#define p2 (p<<1|1)
const int N=1e5+10;
struct T {
    int l,r,cnt;
    double len;
} t[N*8];
struct A {
    double x,y1,y2;
    int add;
} a[N*2];
int n,len;
double lsh[N*2];
bool cmp(A u,A v) {return u.x<v.x;}
int val(double x) {return lower_bound(lsh+1,lsh+1+len,x)-lsh;}
double raw(int x) {return lsh[x];}
void pushUp(int p) {t[p].len=(t[p].cnt>0)?raw(t[p].r+1)-raw(t[p].l):t[p1].len+t[p2].len;}
void build(int p,int l,int r) {
    t[p]={l,r,0,0};
    if(l==r) return;
    int mid=(l+r)>>1;
    build(p1,l,mid),build(p2,mid+1,r);
}
void upd(int p,int l,int r,int add) {
    if(t[p].l>=l && t[p].r<=r) {t[p].cnt+=add,pushUp(p); return;}
    int mid=(t[p].l+t[p].r)>>1;
    if(l<=mid) upd(p1,l,r,add);
    if(r>mid) upd(p2,l,r,add);
    pushUp(p);
}
int main()
{
    for(int tim=1;;tim++) {
        scanf("%d",&n);
        if(!n) break;
        printf("Test case #%d\n",tim);
        for(int i=1;i<=n;i++) {
            double x1,y1,x2,y2;
            scanf("%lf%lf%lf%lf",&x1,&y1,&x2,&y2);
            a[i]={x1,y1,y2,1},a[i+n]={x2,y1,y2,-1};
            lsh[i]=y1,lsh[i+n]=y2;
        }
        n*=2;
        sort(a+1,a+1+n,cmp),sort(lsh+1,lsh+1+n);
        len=unique(lsh+1,lsh+1+n)-lsh-1;
        build(1,1,len-1);
        double ans=0;
        upd(1,val(a[1].y1),val(a[1].y2)-1,a[1].add);
        for(int i=2;i<=n;i++) {
            ans+=t[1].len*(a[i].x-a[i-1].x);
            upd(1,val(a[i].y1),val(a[i].y2)-1,a[i].add);
        }
        printf("Total explored area: %.2lf\n\n",ans);
    }
    return 0;
}
```


---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.github.io/%E7%BA%BF%E6%AE%B5%E6%A0%91%E5%8F%8A%E5%85%B6%E6%89%A9%E5%B1%95/  

