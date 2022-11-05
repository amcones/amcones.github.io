# Tarjan强连通分量与缩点


<!--more-->

# Tarjan强连通分量与缩点

之前就没有很搞懂dfs搜索树的顺序，这次好好搞一下。

## 概念

强连通：两个点互相可达称为强连通
连通分量：一个集合中的顶点互相可达，称为连通分量
强连通分量：每个集合中最大的连通分量。

---

dfs搜索树上边分为四大类（jls的集训队论文也提到了）

1. 树枝边（父节点与子节点之间的边、特殊的前向边）
2. 前向边（每个点到后代的点）
3. 后向边（每个点到祖先的点）
4. **横叉边**（往其它分支搜索的时候产生）

---

判是否在强连通分量中：

1. 存在后向边指向祖先节点
2. 先走到横叉边，再走到祖先。

## Tarjan

引入时间戳的概念判断在dfs中被搜到的顺序（是否和dfs序是一个东西？）

对每个点定义两个时间戳：

1. dfn[u]遍历到u的时间戳
2. low[u]从u开始走，所能遍历到的最小时间戳

u是其所在的强连通分量的最高点，等价于dfn[u]==low[u]

如果还能往上走，就说明强连通分量还没搜完。

```cpp
void tarjan(int u) {
    dfn[u] = low[u] = ++timestamp;
    stk.push(u), in_stk[u] = true;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
        } else if (in_stk[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    if (dfn[u] == low[u]) {
        int y;
        scc_cnt++;
        do {
            y = stk.top();
            stk.pop();
            in_stk[y] = false;
            id[y] = scc_cnt;
            sum[scc_cnt] += a[y - 1];
        } while (y != u);
    }
}
```

## 缩点
```
遍历所有点
    遍历 i 的所有邻点 j
        如果 i 与 j 不在同一SCC中
            加一条新边 id(i)->id(j)
```
**连通分量编号递减的顺序一定是拓扑序，不需要缩点后再写拓扑排序**

感性认识一下，编号越大的越靠上，那么从大到小是拓扑序也挺合理的（


---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.cn/tarjan%E5%BC%BA%E8%BF%9E%E9%80%9A%E5%88%86%E9%87%8F%E4%B8%8E%E7%BC%A9%E7%82%B9/  

