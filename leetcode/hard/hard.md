# 5753. 有向图最大颜色值

| 2021.5.9 |      |      |
| -------- | ---- | ---- |

拓扑序DP -> 在拓扑排序的时候转移方程

## 图的邻接表表示

普通邻接表表示

```c++
vector<vector<pair<int, int>>> graph;
// graph[node_start]里面存储的是{ node_end, weight }
```

链式前向星

```c++
struct {
    // 边的终点
    int to;
    // 边权
    int w;
    // 这条边的起点的下一条边在 edges 数组里的下标
    int next;    
} edges[MAXN];

// head[i] 表示顶点i的第一条边的数组下标，-1表示顶点i没有出边
int head[MAXN];
// 初始化
memset(head,-1,sizeof(head));
// 增加边 a->b, 该边的数组下标为 id
inline void addEdge(int a, int b, int id) {
    edges[id].to = b;
    edges[id].next = head[a];
    // 新增的边要成为顶点`a`的第一条边，而不是最后一条边
    head[a] = id;
}

// 遍历a点的所有出边
for (int i = head[a]; i != -1; i = edges[i].next) {
    // edges[i] 就是当前遍历的边 a->edges[i].to
}

```

## BFS 拓扑排序

```c++
int deg[MAXN];	// 记录每个点的入度，每次选择入度为0的节点
queue<int> Q;

// 首先把所有入度为 0 的节点入队
// n 是节点总数
for (int i = 0; i < n; i++) {
    if (deg[i] == 0) {
        Q.push(i);
    }
}
// 出队的总数
int count = 0;
while (!Q.empty()) {
    int x = Q.front();
    Q.pop();
    ++count;
    for (int i = head[x]; i != -1; i = edges[i].next) {
        int u = head[i].to;
        // 由于 x 已经出队，因此把 u 的入度减一
        --deg[u];
        if (deg[u] == 0) Q.push(u);
    }
}
// 有向图中有环
if (count < n) {
    return false;
}
return true;
```

## 思路

**状态：`dp[i][j]`表示以`i`为终点的所有路径中，颜色为 `j` 的节点的数量最大值。**

转移方程：`dp[i][j]=dp[k][j]+(colors[i] == j), k 是 i 的前驱节点`

初始化：最开始时全部初始化为0

遍历顺序：边拓扑排序边遍历

```c++
class Solution {
    struct edge {
        int to;
        int next;
    };
    const static int MAXN = 1e5 + 5;
    edge edges[MAXN];
    int head[MAXN];
    int deg[MAXN];
public:
    void addEdge(int a, int b, int id) {
        edges[id].to = b;
        edges[id].next = head[a];
        head[a] = id;
    }

    int largestPathValue(string colors, vector<vector<int>>& edge_list) {
        memset(head, -1, sizeof(head));
        memset(deg, 0, sizeof(deg));
        int len = edge_list.size();
        int node_num = colors.size();
        for (int i = 0; i < len; i++) {
            addEdge(edge_list[i][0], edge_list[i][1], i);
            ++deg[edge_list[i][1]];
        }
        vector<vector<int>> dp(node_num, vector<int>(27, 0));
        queue<int> Q;
        for (int i = 0; i < node_num; i++) {
            if (deg[i] == 0) Q.push(i);
            dp[i][colors[i] - 'a'] = 1;
        }

        int count = 0;
        while (!Q.empty()) {
            int u = Q.front();
            Q.pop();
            ++count;
            for (int i = head[u]; i != -1; i = edges[i].next) {
                int v = edges[i].to;
                --deg[v];
                if (deg[v] == 0) {
                    Q.push(v);
                }
                for (int j = 0; j < 26; j++) {
                    dp[v][j] = max(dp[v][j], dp[u][j] + (colors[v] - 'a' == j));
                }
            }
        }

        // for (int i = 0; i < node_num; i++) {
        //     for (int j = 0; j < 26; j++) {
        //         cout << dp[i][j] << "\t";
        //     }
        //     cout << endl;
        // }

        if (count < node_num) return -1;
        int maxx = -1;
        for (int i = 0; i < node_num; i++) {
           maxx = max(maxx, *max_element(dp[i].begin(), dp[i].end()));
        }
        return maxx;
    }
};
```



# 943. 最短超级串

| 2021.5.9 (2021.5.7 晚睡) |      |      |
| ------------------------ | ---- | ---- |

状态压缩 DP

本质上是一个旅行商问题

```c++
class Solution {
public:
    string shortestSuperstring(vector<string>& words) {
        // dp[i][j] 表示集合 i 为前缀，j \in i 且 j 为结尾的重叠部分的长度
        // ... k ... j
        // dp[i][j] = dp[i ^ (1 << j)][k] + overlap[k][j] for all k in i and k != j
        int n = words.size();
        int m = 1 << n;
        vector<vector<int>> dp(m, vector<int>(n, 0));
        vector<vector<int>> parent(m, vector<int>(n, -1));
        vector<vector<int>> overlap(n, vector<int>(n, 0));

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                overlap[i][j] = calcOverlap(words[i], words[j]);
            }
        }

        for (int mask = 0; mask < m; mask++) {
            for (int j = 0; j < n; j++) {
                // 说明 mask 的第 j 位是 1
                if ((mask >> j) & 1) {
                    int pmask = mask ^ (1 << j);
                    for (int k = 0; k < n; k++) {
                        if ((pmask >> k) & 1) {
                            int tmp = dp[pmask][k] + overlap[k][j];
                            if (dp[mask][j] <= tmp) {
                                dp[mask][j] = tmp;
                                // cout << "parent[" << mask << "]" << "[" << k << "] = " << k << endl; 
                                parent[mask][j] = k;
                            }
                        }
                    }
                }
            }
        }

        int maxx = -1;
        int end = -1;
        for (int i = 0; i < n; i++) {
            if (maxx < dp[m - 1][i]) {
                maxx = dp[m - 1][i];
                end = i;
            }
        }
        string ans = words[end];
        // parent[mask][j] 是以 j 为末尾的上一个点
        int p = end;
        int mask = m - 1;
        int pre = parent[mask][end];
        while (pre != -1) {
            // cout << pre << endl;
            int len = words[pre].size();
            ans = words[pre].substr(0, len - overlap[pre][p]) + ans;
            mask ^= 1 << p;
            p = pre;
            pre = parent[mask][p];
        }
        return ans;
    }

    int calcOverlap(const string& a, const string& b) {
        int la = a.size(), lb = b.size();
        int l = min(la, lb);
        for (int i = l; i >= 0; i--) {
            // -1 表示
            if (a.compare(la - i, i, b, 0, i) == 0) {
                return i;
            }
        }
        return 0;
    }
};
```

