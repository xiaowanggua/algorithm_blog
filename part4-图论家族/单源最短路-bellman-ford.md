## 📖 算法讲解

### 1. 问题模型
给定带权有向图 `G=(V,E)`，源点 `s`，求 `s` 到所有其它顶点的最短路径长度（权值和最小）。  
**允许负权边**，但**不允许负权环**（否则最短路径无定义）。  
若图中存在从 `s` 可达的负权环，算法能够检测并报告。

---

### 2. Bellman-Ford 思想
1. 维护一个距离数组 `dist[1…n]`，初始时 `dist[s]=0`，其余为正无穷。  
2. 进行 `n-1` 轮「全边松弛」：  
   对每条边 `(u,v,w)`，若 `dist[u]+w < dist[v]`，则更新 `dist[v]`。  
3. 第 `n` 轮再扫描所有边，若仍能松弛，则说明存在负权环。  

正确性证明依赖于「路径边数」归纳：  
第 `k` 轮结束后，所有**最多由 `k` 条边组成**的最短路径都已求出。  
由于简单路径最多 `n-1` 条边，`n-1` 轮即可保证收敛；第 `n` 轮若仍松弛，说明存在无限缩短的负环。

---

### 3. 代码实现
以下代码与给出 Dijkstra 示例风格保持一致：  
- 采用 `i64` 类型  
- 使用 `vector<char>` 做标志  
- 注释风格、变量命名、输入输出格式均仿照原示例  
- 额外返回「是否存在负权环」信息

```c++
#include <bits/stdc++.h>
using i64 = long long;
struct Edge {
    int from, to;
    i64 w;
    Edge(int u, int v, i64 c) : from(u), to(v), w(c) {}
};
int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    i64 n, m, s;
    std::cin >> n >> m >> s;

    std::vector<Edge> edges;
    // 本题暂时不需要使用邻接矩阵
    edges.reserve(m);
    for (int i = 0; i < m; ++i) {
        int a, b, c;
        std::cin >> a >> b >> c;
        edges.emplace_back(a, b, c);
    }

    const i64 INF = std::numeric_limits<i64>::max();
    std::vector<i64> dist(n + 1, INF);
    dist[s] = 0;

    // n-1 轮松弛
    for (int round = 1; round < n; ++round) {
        bool updated = false;
        for (auto &[u, v, w] : edges) {
            if (dist[u] != INF && dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                updated = true;
            }
        }
        if (!updated) break;          // 提前收敛
    }

    // 第 n 轮检测负环
    bool hasNegCycle = false;
    for (auto &[u, v, w] : edges) {
        if (dist[u] != INF && dist[v] > dist[u] + w) {
            hasNegCycle = true;
            break;
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
下面是邻接表版本的实现（仅风格差异）

```c++
#include <bits/stdc++.h>
using i64 = long long;
struct Edge {
    int to;
    i64 w;
    Edge(int v, i64 c) : to(v), w(c) {}
};
int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    i64 n, m, s;
    std::cin >> n >> m >> s;

    std::vector<std::vector<Edge>> edges(n+1);
    for (int i = 0; i < m; ++i) {
        int a, b, c;
        std::cin >> a >> b >> c;
        edges[a].emplace_back({b, c});
    }

    const i64 INF = std::numeric_limits<i64>::max();
    std::vector<i64> dist(n + 1, INF);
    dist[s] = 0;

    bool updated = true;
    // 判断提前收敛
    // n-1 轮松弛
    for (int round = 1; round < n && updated; ++round) {
        updated = false;
        for(int u = 0;u <= n; u++){
            if(dist[u] == INF){
                continue;
            }
            for(auto& [v,w] : edges[u]){
                if(dist[u] + w < dist[v]){
                    dist[v] = dist[u] + w;
                    updated = true;
                }
            }
        }

    }

    // 第 n 轮检测负环
    bool hasNegCycle = false;
    for (int u = 1; u <= n; ++u) {
        if (dist[u] == INF) continue;
        for (auto &[v, w] : edges[u]) {
            if (dist[u] + w < dist[v]) {
                hasNegCycle = true;
                break;
            }
        }
        if (hasNegCycle) break;
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
- 时间复杂度 `O(n·m)`，边数较大时请改用 SPFA（队列优化 Bellman-Ford）。  
- 若只需判断负环，可在第 `n` 轮后继续松弛并记录哪些点被更新，这些点即在负环上或可达负环。