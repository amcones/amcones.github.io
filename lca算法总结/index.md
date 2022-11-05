# LCA算法总结


<!--more-->

# LCA算法总结
LCA（最近公共祖先）一般有三种算法解决这类问题。为什么要求树上结点的公共祖先？这是由于求解树上两点间最近距离、以及做树上差分等问题时需要。（其它得做题才知道）
## 向上标记法
这是最容易理解的一种做法，时间复杂度$O(n)$，在多次查询时就不够用了，所以很少用。思想就是从要查询的点向上标记直到根节点，再从另一点出发向上标记，直到标记重合，那个点就是LCA。
## 倍增LCA（在线）
使用ST表的倍增思想和dp（ST表本来就是dp）优化向上跳的速度，时间复杂度预处理$O(nlogn)$+查询$O(logn)$。
先用一次bfs（不用dfs因为递归容易爆栈）预处理所有结点向上跳2^k的父亲结点是谁（使用dp）。   
然后使用LCA算法，不妨设a结点更深，先将a结点向上跳到和b一样高，此时如果a与b是同一个结点则这个点就是LCA，否则他们就是同一深度不同的两点。   
此时再让他们一起向上跳，加上约束条件：他们向上跳的父节点不同。因为第一个出现父节点相同的可能只是公共祖先而不是最近公共祖先，那么只要父节点不同就是还没跳到LCA。循环结束后保证俩个结点的父节点就是LCA。
为什么能确保循环结束后父节点是LCA？因为满足条件时，两个点一起向上跳2^k。众所周知，任何一个十进制正数都能被表示为数个2^k的和。（十进制的二进制表示）那么，如果跳过根节点了，那么他们的父节点都为0不满足约束条件，不跳；如果跳过LCA了，他们的父节点必是一个点（因为经过LCA后，必定跳的位置在同一条子链上）；因此满足约束条件，必定是在LCA下方。此时使用二进制拼凑，得到两个结点的深度与LCA深度的差就是找到了。同时不直接跳到LCA也是由于这个约束条件的关系，更根本的原因其实是约束条件是判断LCA把两个子树合并在一起的性质是否满足（我是这么认为的）。
```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
#include<queue>
using namespace std;
const int maxn=4e4+10;
int dep[maxn],fa[maxn][20];
int head[maxn],tot;
struct edge{
    int to,next;
}eg[maxn<<1];
void add(int u,int v)
{
    eg[tot].to=v;
    eg[tot].next=head[u];
    head[u]=tot++;
}
void bfs(int s)
{
    memset(dep,0x3f,sizeof dep);
    dep[0]=0,dep[s]=1;
    queue<int>q;
    q.push(s);
    while(!q.empty())
    {
        int t=q.front();
        q.pop();
        for(int i=head[t];~i;i=eg[i].next)
        {
            int j=eg[i].to;
            if(dep[j]>dep[t]+1){
                dep[j]=dep[t]+1;
                q.push(j);
                fa[j][0]=t;
                for(int k=1;k<=15;k++)fa[j][k]=fa[fa[j][k-1]][k-1];
            }
        }
    }
}
int lca(int a, int b)
{
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = 15; k >= 0; k -- )
        if (dep[fa[a][k]] >= dep[b])
            a = fa[a][k];
    if (a == b) return a;
    for (int k = 15; k >= 0; k -- )
        if (fa[a][k] != fa[b][k])
        {
            a = fa[a][k];
            b = fa[b][k];
        }
    return fa[a][0];
}
int main()
{
    memset(head,-1,sizeof head);
    int n;
    cin>>n;
    int a,b,root=0;
    for(int i=1;i<=n;i++){
        cin>>a>>b;
        if(b==-1)root=a;
        else add(a,b),add(b,a);
    }
    bfs(root);
    int m,x,y;
    cin>>m;
    for(int i=1;i<=m;i++){
        cin>>x>>y;
        int ans=lca(x,y);
        if(ans==x)cout<<1<<endl;
        else if(ans==y)cout<<2<<endl;
        else cout<<0<<endl;
    }
    return 0;
}
```
## Tarjan（离线）
需要注意的是这里的Tarjan算法是指离线求LCA的一种做法，而不是著名的求强连通分量的那个Tarjan。
时间复杂度$O(n+m)$,需要存下所有询问。至于和倍增写法的适用差别尚不明确，时间复杂度其实差的也不多？不过听说常数小一点。
Tarjan是基于dfs的算法，在搜索过程中把点分为三类：未遍历过的点0，正在遍历的点1，已经遍历且回溯的点2。   
任选一点u开始搜索。 
对于与u相连且未搜索的点，递归搜索它并标记为1，回溯时使用并查集将其全部合并到当前点。其实在树中，深度比u深的且和u相连的点就是u的儿子，所以是把儿子都合并到父亲上。     
然后处理与当前点有关的询问，如果另一个点有标记2，那么当前点与另一个点的LCA就是另一个点的根节点。（其实就是这两个子树中节点的LCA都是那个点）       
最后把这个根节点标记为2。       
为什么标记2的点的根节点就是LCA？因为标记为2的点已经被合并完毕，它在并查集中的fa不会有变化了，这个fa也就是它所在的子树的根节点。（其实又利用了LCA合并子树的性质，我觉得）而这个根节点是LCA与dfs的遍历顺序有关，标记为2的点一定是先被遍历，而标记为1的则是之后才被遍历。因为在回溯的时候才会把路径压缩到根节点，所以会保证标记2的点的fa是LCA，即是两个查询节点所在子树的根节点。
```cpp
#include<iostream>
#include<vector>
#include<cstring>
using namespace std;
using PII=pair<int,int>;
const int maxn=2e4+10;
int head[maxn],tot,n,m;
int dis[maxn];
int fa[maxn];
int st[maxn],res[maxn];
struct edge{
    int to,w,next;
}eg[maxn<<1];
vector<PII> query[maxn];
void add(int u,int v,int w)
{
    eg[tot].to=v;
    eg[tot].w=w;
    eg[tot].next=head[u];
    head[u]=tot++;
}
void dfs(int u,int fath)
{
    for(int i=head[u];~i;i=eg[i].next)
    {
        int j=eg[i].to;
        if(j==fath)continue;
        dis[j]=dis[u]+eg[i].w;
        dfs(j,u);
    }
}
int find(int x)
{
    return x==fa[x]?x:fa[x]=find(fa[x]);
}
void tarjan(int u)
{
    st[u]=1;
    for(int i=head[u];~i;i=eg[i].next)
    {
        int j=eg[i].to;
        if(!st[j]){
            tarjan(j);
            fa[j]=u;
        }
    }
    for(auto item:query[u])
    {
        int y=item.first,id=item.second;
        if(st[y]==2){
            int anc=find(y);
            res[id]=dis[u]+dis[y]-2*dis[anc];
        }
    }
    st[u]=2;
}
int main()
{
    memset(head,-1,sizeof head);
    cin>>n>>m;
    int u,v,w;
    for(int i=1;i<n;i++){
        cin>>u>>v>>w;
        add(u,v,w),add(v,u,w);
    }
    for(int i=0;i<m;i++){
        cin>>u>>v;
        if(u!=v){
            query[u].push_back({v,i});
            query[v].push_back({u,i});
        }
    }
    for(int i=1;i<=n;i++)fa[i]=i;
    dfs(2,-1);
    tarjan(2);
    for(int i=0;i<m;i++)cout<<res[i]<<endl;
    return 0;
}
```
## 应用
### 次小生成树
这是人做的？
```cpp
#include<iostream>
#include<algorithm>
#include<vector>
#include<cstring>
#include<queue>
using namespace std;
using ll=long long;
const int N=1e5+10,M=3e5+10,inf=0x3f3f3f3f;
int n,m;
struct edge{
    int a,b,w;
    bool used;
}eg[M];
int p[N];
int h[N],e[M],w[M],ne[M],idx;
int depth[N],fa[N][17],d1[N][17],d2[N][17];
void add(int a,int b,int c)
{
    e[idx]=b,ne[idx]=h[a],w[idx]=c,h[a]=idx++;
}
int find(int x)
{
    return x==p[x]?x:p[x]=find(p[x]);
}
ll kruskal()
{
    for(int i=1;i<=n;i++)p[i]=i;
    sort(eg,eg+m,[](edge a,edge b){return a.w<b.w;});
    ll res=0;
    for(int i=0;i<m;i++)
    {
        int a=find(eg[i].a),b=find(eg[i].b);
        if(a!=b){
            p[a]=b;
            res+=eg[i].w;
            eg[i].used=true;
        }
    }
    return res;
}
void build()
{
    memset(h,-1,sizeof h);
    for(int i=0;i<m;i++)
    {
        if(eg[i].used)
        {
            int a=eg[i].a,b=eg[i].b,w=eg[i].w;
            add(a,b,w),add(b,a,w);
        }
    }
}
void bfs()
{
    memset(depth,inf,sizeof depth);
    depth[0]=0,depth[1]=1;
    queue<int>q;
    q.push(1);
    while(!q.empty())
    {
        int t=q.front();
        q.pop();
        for(int i=h[t];~i;i=ne[i])
        {
            int j=e[i];
            if(depth[j]>depth[t]+1){
                depth[j]=depth[t]+1;
                q.push(j);
                fa[j][0]=t;
                d1[j][0]=w[i],d2[j][0]=-inf;
                for(int k=1;k<=16;k++)
                {
                    int anc=fa[j][k-1];
                    fa[j][k]=fa[anc][k-1];
                    int distance[4]={d1[j][k-1],d2[j][k-1],d1[anc][k-1],d2[anc][k-1]};
                    d1[j][k]=d2[j][k]=-inf;
                    for(int d:distance)
                    {
                        if(d>d1[j][k])d2[j][k]=d1[j][k],d1[j][k]=d;
                        else if(d!=d1[j][k]&&d>d2[j][k])d2[j][k]=d;
                    }
                }
            }
        }
    }
}
int lca(int a,int b,int w)
{
    static int distance[N<<1];
    int cnt=0;
    if(depth[a]<depth[b])swap(a,b);
    for(int k=16;k>=0;k--)
    {
        if(depth[fa[a][k]]>=depth[b])
        {
            distance[cnt++]=d1[a][k];
            distance[cnt++]=d2[a][k];
            a=fa[a][k];
        }
    }
    if(a!=b)
    {
        for(int k=16;k>=0;k--)
        {
            if(fa[a][k]!=fa[b][k])
            {
                distance[cnt++]=d1[a][k];
                distance[cnt++]=d2[a][k];
                distance[cnt++]=d1[b][k];
                distance[cnt++]=d2[b][k];
                a=fa[a][k],b=fa[b][k];
            }
        }
        distance[cnt++]=d1[a][0];
        distance[cnt++]=d1[b][0];
    }
    int dis1=-inf,dis2=-inf;
    for(int i=0;i<cnt;i++)
    {
        int d=distance[i];
        if(d>dis1)dis2=dis1,dis1=d;
        else if(d!=dis1&&d>dis2)dis2=d;
    }
    if(w>dis1)return w-dis1;
    if(w>dis2)return w-dis2;
    return inf;
}
int main()
{
    cin>>n>>m;
    for(int i=0;i<m;i++){
        int a,b,c;
        cin>>a>>b>>c;
        eg[i]={a,b,c};
    }
    ll sum=kruskal();
    build();
    bfs();
    ll res=2e18;
    for(int i=0;i<m;i++)
    {
        if(!eg[i].used)
        {
            int a=eg[i].a,b=eg[i].b,w=eg[i].w;
            res=min(res,sum+lca(a,b,w));
        }
    }
    cout<<res<<endl;
    return 0;
}
```
除了LCA和最小生成树不用说，主要的问题在于存路径上的最长边和次长边用于之后的替换。
### 树上差分
```cpp
#include<bits/stdc++.h>
using namespace std;
#define N 500100
int n,m,tot,ans,p[N],f1[N],lin[N],f[N][25],d[N],vis[N];
struct gg {
    int x,y,next;
}a[N<<1];
inline void add(int x,int y) {
    a[++tot].x=x;
    a[tot].y=y;
    a[tot].next=lin[x];
    lin[x]=tot;
}
void bfs(int x) {
    queue<int> q;
    q.push(1);
    d[1]=1;
    while(q.size()) {
        int x=q.front();
        q.pop();
        for(int i=lin[x];i;i=a[i].next) {
            int y=a[i].y;
            if(d[y]) continue;
            d[y]=d[x]+1;
            f[y][0]=x;
            for(int j=1;j<=23;j++){
                f[y][j]=f[f[y][j-1]][j-1];
            }
            q.push(y);
        }
    }
}
inline int lca(int x,int y) {
    if(d[x]>d[y]) swap(x,y);
    for(int i=23;i>=0;i--) {
        if(d[f[y][i]]>=d[x]) y=f[y][i];
    }
    if(x==y) return x;
    for(int i=23;i>=0;i--) {
        if(f[x][i]!=f[y][i]) x=f[x][i],y=f[y][i];
    }
    return f[x][0];
}
inline void dfs(int x) {
    vis[x]=1;
    for(int i=lin[x];i;i=a[i].next) {
        int y=a[i].y;
        if(vis[y]) continue;
        dfs(y);
        p[x]+=p[y];
    }
}
int main() {
    cin>>n>>m;
    int x,y;
    for(int i=1;i<n;i++) {
        cin>>x>>y;
        add(x,y);
        add(y,x);
    }
    bfs(1);
    for(int i=1;i<=m;i++) {
        cin>>x>>y;
        p[x]++,p[y]++;
        p[lca(x,y)]-=2;
    }
    dfs(1);
    for(int i=2;i<=n;i++) {
        if(!p[i]) ans+=m;
        if(p[i]==1) ans+=1;
    }
    cout<<ans<<endl;
    return 0;
}
```
## 总结
LCA算是偏基础的算法了，不是说他好写，是需要用它作为其它算法的基础才能解决问题。裸的LCA没啥问题，但是用LCA去解决在树上找边、删边、换边等问题就比较麻烦了。

---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.cn/lca%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93/  

