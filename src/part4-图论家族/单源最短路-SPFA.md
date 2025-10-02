## 📖 算法讲解

### 1. 问题模型
同 Bellman-Ford：  
给定带权有向图 `G=(V,E)`，源点 `s`，求 `s` 到所有其它顶点的最短路径长度。  
**允许负权边**，**不允许负权环**；若存在从 `s` 可达的负权环，算法能够检测并报告。

---

### 2. SPFA 思想（Shortest Path Faster Algorithm）
1. 只维护一个**普通队列** `q`，初始放入源点 `s`。  
2. 用 `inQueue[u]` 标记点 `u` 是否已在队列中，避免重复入队。  
3. 每次弹出队首 `u`，对其**所有出边**做松弛：  
   若 `dist[u]+w(u,v) < dist[v]`，则更新 `dist[v]`；  
   若 `v` 不在队列，则把 `v` 入队。  
4. 用 `cnt[v]` 记录**当前最短路径包含的边数**；  
   若某次更新后 `cnt[v] ≥ n`，说明存在负权环。  

正确性依旧依赖 Bellman-Ford 的松弛规则，但**只把真正发生更新的点入队**，平均复杂度 `O(m)`，最坏仍为 `O(n·m)`。

---

### 3. 代码实现
风格继续仿照原 Dijkstra 示例：  
- `i64` 类型、`vector<char>` 标记、`priority_queue` 改为 `queue`  
- 注释、变量命名、输入输出格式保持一致  
- 额外返回「是否存在负权环」信息

```c++
#include <bits/stdc++.h>
using i64 = long long;
struct Edge {
    int to;
    i64 w;
    Edge(int t, i64 c) : to(t), w(c) {}
};
int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    i64 n, m, s;
    std::cin >> n >> m >> s;

    std::vector<std::vector<Edge>> conn(n + 1);
    for (int i = 0; i < m; ++i) {
        int a, b, c;
        std::cin >> a >> b >> c;
        conn[a].emplace_back(b, c);
    }

    const i64 INF = std::numeric_limits<i64>::max();
    std::vector<i64> dist(n + 1, INF);
    std::vector<char> inQueue(n + 1, 0);
    std::vector<int> cnt(n + 1, 0);   // 记录最短路径边数
    std::queue<int> q;

    dist[s] = 0;
    q.push(s);
    inQueue[s] = 1;

    bool hasNegCycle = false;
    while (!q.empty() && !hasNegCycle) {
        int u = q.front();
        q.pop();
        inQueue[u] = 0;

        for (auto &[v, w] : conn[u]) {
            if (dist[u] != INF && dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n) {          // 负环检测
                    hasNegCycle = true;
                    break;
                }
                if (!inQueue[v]) {
                    q.push(v);
                    inQueue[v] = 1;
                }
            }
        }
    }

    if (hasNegCycle) {
        std::cout << "NEGATIVE CYCLE DETECTED\n";
    } else {
        for (int i = 1; i <= n; ++i) {
            if (dist[i] == INF) std::cout << "INF ";
            else                std::cout << dist[i] << ' ';
        }
        std::cout << '\n';
    }
    return 0;
}
```

---

### 4. 使用小贴士
- 平均时间复杂度 `O(m)`，网格图、随机图常远优于 `O(n·m)`。  
- 若只需判断**全图任意负环**，可从**虚拟超级源**向所有点连 `0` 边，再跑 SPFA；若某点 `cnt[v] ≥ n+1` 即说明负环存在。