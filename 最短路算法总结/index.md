# 最短路算法总结


<!--more-->

# 最短路算法总结
最短路应该是图论最基本的问题了，之后在各种问题中也会作为基本算法组合使用。这里整理一下五种常用最短路算法（我也不知道还有啥了）的思想和用处。  
**前置知识：存图，图的定义**
## 所有结点对的最短路径问题
### 基于动态规划的矩阵乘法求解
由于时间复杂度比floyd都高（为$O(n^3lgn)$，还是用快速幂优化了），所以没人用。只是作为一个思想在此简述。
### floyd
使用$O(n^3)$的时间，使用每一个点对整张图进行松弛。使用场景：任意两点间最短路，传递闭包。
> 从数学上来说，传递闭包是在集合$X$上求包含关系$R$的最小传递关系。从关系图的角度来说，就是如果原关系图上有$i$到$j$的路径，则其传递闭包的关系图上就应有从$i$到$j$的边。通俗地讲，就是确定每个点是否能到达其他每个点。
```cpp
for (int k = 1; k <= n; ++k)
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= n; ++j)
            if (E[i][k] && E[k][j])
                E[i][j] = 1;
```
### Johnson
用于大型稀疏图，时间复杂度$O(n^2lgn+nv)$。使用dijkstra和bellman-ford作为子程序。*因为挺麻烦的所以其实挺少用。*   
思想很简单，使用的技术称为**重新赋予权重**。如果图中无负权边，那么跑n-1次dijkstra就能得到任意两点间的最短距离。   
但如果存在负权边，由于dijkstra无法处理负权边，很简单的想法是给所有边加上一个值使得所有边大于0，算出最短路后再一起减去这个值，但这样是不对的，可以举出反例。  
应当赋予的权重$\hat\omega$必须满足两个性质：
1. $\hat\omega$不改变原来的最短路径。
2. $\hat\omega$非负。

正确的做法是添加虚拟节点到图中，它到所有点的距离都为0。以虚拟节点为起点执行Bellman-Ford，求出到每个节点的最短距离，这个距离为每个节点的值，再去更新图里的所有边，最后再对每个点执行dijkstra。       
算法步骤：
1. 添加虚拟节点到这个图里，并添加指向所有节点的虚拟边，这些边的权重为0
2. 以虚拟节点节点为起点，运行 Bellman-Ford 算法，求出到每个节点的最短距离，这些最短距离为每个节点的值
3. 用上面求出的h值去更新图里的边，使得$eg[u,v]=eg[u,v]+h[u]-h[v] $
4. 移除添加的虚拟节点和边
5. 在每个节点运行 Dijkstra 计算到其它节点的最短距离
《算法导论》说使用斐波那契堆实现Dijkstra效率会比使用普通二叉堆高，不清楚具体的，反正差不多。  

以下是证明。  
**引理**：给定带权重的有向图$G=(V,E)$ ，其权重函数为$\omega:E\rightarrow R$ ，设$h:V\rightarrow R$ 为任意函数，该函数将结点映射到实数上。对于每条边$(u,v)\in E$ ，定义
$$
\hat\omega(u,v)=\omega(u,v)+h(u)-h(v)
$$
设$p=\langle v_0,v_1,\cdots,v_k\rangle$ 为从结点$v_0$ 到$v_k$ 的任意一条路径，那么$p$是在使用权重函数$\omega$时从结点$v_0$ 到$v_k$ 的一条最短路径，当且仅当$p$ 是在使用权重函数$\hat\omega$ 时从结点$v_0$ 到$v_k$ 的一条最短路径，即$\omega(p)=\delta(v_0,v_k)$ 当且仅当$\hat\omega(p)=\hat\delta(v_0,v_k)$ 。而且，图$G$ 在使用权重函数$\omega$ 时不包含权重为负值的环路，当且仅当$p$ 在使用权重函数$\hat\omega$ 也不包含权重为负值的环路。   
**证明**：
首先来证明
$$
\hat\omega(p)=\omega(p)+h(v_0)-h(v_k)
$$
我们有：    
$$
\begin{aligned}
\hat\omega(p) & = \sum\limits^k_{i=1}\hat\omega(v_{i-1},v_i)\\\ 
& = \sum\limits^k_{i=1}(\omega(v_{i-1},v_i)+h(v_{i-1})-h(v_i))\\\ 
& = \sum\limits^k_{i=1}\omega(v_{i-1},v_i)+h(v_0)-h(v_k)\text{(裂项相消)} \\\ 
& = \omega(p)+h(v_0)-h(v_k)
\end{aligned}
$$
因为 $h(v_0)$ 和 $h(v_k)$ 不依赖于任何具体的路径，如果从结点 $v_0$ 到 $v_k$ 的一条路径在使用权重函数 $\omega$ 时比另一条路径短，则其在使用权重函数 $\hat\omega$ 时也比另一条短。因此 $\omega(p)=\delta(v_0,v_k)$ 当且仅当 $\hat\omega(p)=\hat\delta(v_0,v_k)$ 。  
最后，我们来证明图 $G$ 在使用权重函数 $\omega$ 时包含一个权重为负值的环路当且仅当 $p$ 在使用权重函数 $\hat\omega$ 时也包含一个权重为负值的环路。考虑任意环路 $c=\langle v_0,v_1,\cdots,v_k\rangle$ ，其中 $v_0=v_k$ 。根据 $\hat\omega(c)=\omega(c)+h(v_0)-h(v_k)=\omega(c)$ ，因此，环路 $c$ 在使用权重函数 $\omega$ 时为负值当且仅当 $c$ 在使用权重函数 $\hat\omega$ 时也为负值。    
接下来证明第二个性质保持成立，即对于所有的边 $(u,v)\in E$ ，$\hat\omega(u,v)$ 为非负值。给定带权重的有向图 $G=(V,E)$ ，其权重函数为 $\omega:E\rightarrow R$ 。制作一张新图 $G'=(V',E')$ ，这里 $V'=V\cup\{s\}$ ，$s$ 是一个新结点，$s\notin V$ ，$E'=E\cup\{(s,v):v\in V\}$ 。我们对权重函数$\omega$ 进行扩展，使得对于所有的结点$v\in V,\omega(s,v)=0$ 。因为$s$ 入度为0，除了以$s$ 为源结点的最短路径外，图$G'$中没有其他包含结点$s$ 的最短路径。而且，图$G'$ 不包含权重为负值的环路当且仅当图$G$ 不包含权重为负值的环路。   
现在假定图$G$ 和图$G'$ 都不包含权重为负值的环路。让我们定义，对于所有的结点$v\in V',h(v)=\delta(s,v)$ 。根据三角不等式，对于所有的边$(u,v)\in E'$ ，有$h(v)\leqslant h(u)+\omega(u,v)$ 。因此，如果我们根据$\hat\omega(p)=\omega(p)+h(v_0)-h(v_k)$ 来定义新的权重$\hat\omega$ ，则有$\hat\omega(u,v)=\omega(u,v)+h(u)-h(v)\geqslant 0$ ，至此我们满足了第二条性质。     
Q.E.D.
## 单源最短路径
### dijkstra
思想是从起点开始，每次选择离起点最近的且未访问过的（即未曾被作为松弛点）点，将与此点相连的所有点的距离松弛。  
在选择离起点最近的点时，朴素dijkstra使用暴力$O(n)$找出最近点，而堆优化dijkstra则能够$O(logv)$找出最近点。但由于堆优化dijkstra需要将所有边加入堆进行松弛，所以复杂度与边有关。
使用场景：单源最短路，无负权边。
#### 朴素dijkstra
时间复杂度$O(n^2)$，使用场景稠密图。
```cpp
int dis1[maxn],dis2[maxn],vis[maxn],mp[maxn][maxn];
void dij(int s)
{
    memset(vis,0,sizeof vis);
    memset(dis1,0x3f,sizeof dis1);
    dis1[s]=0;
    for(int i=1;i<=n;i++){
        int t=-1;
        for(int j=1;j<=n;j++){
            if(!vis[j]&&(t==-1||dis1[j]<dis1[t]))t=j;
        }
        vis[t]=1;
        for(int j=1;j<=n;j++){
            dis1[j]=min(dis1[j],dis1[t]+mp[t][j]);
        }
    }
}
```
#### 堆优化dijkstra
时间复杂度$O(nvlogv)$（或写为$O(Elogv)$）。一般情况下都用它。
```cpp
void dij(int s)
{
    fill(dis,dis+maxn,inf);
    priority_queue<Pair,vector<Pair>,greater<Pair>> q;
    dis[s]=0;
    q.push({0,s});
    while(!q.empty())
    {
        int p=q.top().second;
        q.pop();
        if(vis[p])continue;
        vis[p]=true;
        for(int i=head[p];i!=-1;i=eg[i].next){
            int to=eg[i].to;
            dis[to]=min(dis[to],eg[i].w+dis[p]);
            if(!vis[to])q.push({dis[to],to});
        }
    }
}
```
### Bellman-Ford
思想是把所有边松弛n-1遍。时间复杂度$O(nv)$。为什么要用这么暴力的算法？因为使用场景有边数限制的最短路。    
由于Bellman-Ford每松弛一遍，相当于在最短路中插入了一条边，所以只要运行k次就是只有k条边的最短路了。
实现时，为了避免串联效应，即边数限制为1时却把要经过2条边才能到达的点也给更新了，需要一个备份数组，备份更新之前的距离，再用备份数组去更新其它点的距离。
```cpp
int bellmanford()
{
    memset(dis,0x3f,sizeof dis);
    dis[1]=0;
    for(int i=1;i<=k;i++){
        memcpy(backup,dis,sizeof dis);
        for(int j=0;j<m;j++){
            dis[eg[j].v]=min(dis[eg[j].v],backup[eg[j].u]+eg[j].w);
        }
    }
    if(dis[n]>0x3f3f3f3f/2)return -1;
    return dis[n];
}
```
### SPFA
Bellman-Ford的队列优化版本，最坏情况下复杂度能卡到和Bellman-Ford一致。优化的思路是每次不用松弛所有点，只松弛被松弛的点能够直接到达的点，因为其它点不可能被更新。使用场景判负环、正环。（只要判断一个点是否入队超过n次即可）
```cpp
bool spfa()
{
    queue<int>q;
    for(int i=1;i<=n;i++)q.push(i);
    while(!q.empty())
    {
        int t=q.front();
        q.pop();
        st[t]=false;
        for(int i=head[t];~i;i=eg[i].next)
        {
            int j=eg[i].to;
            if(dis[j]>dis[t]+eg[i].w){
                dis[j]=dis[t]+eg[i].w;
                cnt[j]=cnt[t]+1;
                if(cnt[j]>=n)return true;
                if(!st[j]){
                    q.push(j);
                    st[j]=true;
                }
            }
        }
    }
    return false;
}
```
## 差分约束
[差分约束 - OI Wiki](https://oi-wiki.org/graph/diff-constraints/)       
~~什么鸟语~~    
人话就是找出一组解，使得n元一次不等式组成立。任何最短路问题和差分约束问题都可以相互转化。       
原理：约束条件$x_i-x_j\leqslant c_k$都可以变形为$x_i\leqslant x_j+c_k$，与$dis[y]\leqslant dis[x]+z$最短路三角不等式非常相似。因此，我们可以把每个变量$x_i$看做图中的一个结点，对于每个约束条件$x_i-x_j\leqslant c_k$，从结点$j$向结点$i$连一条长度为$c_k$的有向边。    
建立超级源点，令$dis[0]=0$，并向所有点连长度为最低限制的边（比如所有x都要大于1），确保可以访问所有点和边，然后跑SPFA。当图中存在负环则无解（可使用反证法证明），这也是需要使用SPFA而不是dijkstra的原因，不存在负环则一定存在一组解。   
求最少转化为最长路（因为是求距离的下界），求最大则转化为最短路（因为是求距离的上界）。     
注：有些时候把SPFA中的队列改成栈会跑得更快，因为存在负环的话会直接被判出。（反复入栈n+1次）但SPFA的复杂度太玄学了，不能随便改。   
[#2436. 「SCOI2011」糖果 - Problem - LibreOJ](https://loj.ac/p/2436)
```cpp
#include<iostream>
#include<cstring>
#include<stack>
using namespace std;
using ll=long long;
const int N=1e5+10,M=3e5+10;
int head[N],cnt[N],st[N],tot,n,m;
struct edge{
    int to,w,next;
}eg[M];
ll dis[N];
void add(int u,int v,int w)
{
    eg[tot].to=v;
    eg[tot].w=w;
    eg[tot].next=head[u];
    head[u]=tot++;
}
bool spfa()
{
    memset(dis,-0x3f,sizeof dis);
    stack<int>s;
    s.push(0);
    dis[0]=0;
    st[0]=true;
    while(!s.empty())
    {
        int t=s.top();
        s.pop();
        st[t]=false;
        for(int i=head[t];~i;i=eg[i].next){
            int j=eg[i].to;
            if(dis[j]<dis[t]+eg[i].w){
                dis[j]=dis[t]+eg[i].w;
                cnt[j]=cnt[t]+1;
                if(cnt[j]>=n+1)return false;
                if(!st[j]){
                    s.push(j);
                    st[j]=true;
                }
            }
        }
    }
    return true;
}
int main()
{
    memset(head,-1,sizeof head);
    cin>>n>>m;
    while(m--){
        int x,a,b;
        cin>>x>>a>>b;
        if(x==1)add(a,b,0),add(b,a,0);
        else if(x==2)add(a,b,1);
        else if(x==3)add(b,a,0);
        else if(x==4)add(b,a,1);
        else if(x==5)add(a,b,0);
    }
    for(int i=1;i<=n;i++)add(0,i,1);
    if(!spfa())puts("-1");
    else{
        ll res=0;
        for(int i=1;i<=n;i++)res+=dis[i];
        cout<<res<<endl;
    }
    return 0;
}
```
## 总结
最短路算法本身没什么东西，顶多改变建图的方法，边的类型（比如路径从加变为乘，这种时候可以全部取log变乘为加）等等。但可以与矩阵转置、矩阵快速幂连用，这就比较麻烦了。

---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.github.io/%E6%9C%80%E7%9F%AD%E8%B7%AF%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93/  

