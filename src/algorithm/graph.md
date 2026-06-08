# 图算法

## 图算法简介
### 什么是图算法？
图算法是处理图数据结构的算法，图由顶点（节点）和边组成，用于表示对象之间的关系。图算法广泛应用于社交网络、路由规划、推荐系统等领域。

### 图的表示方式
- **邻接矩阵**：二维数组表示顶点间的连接关系
- **邻接表**：链表或数组表示每个顶点的邻居
- **边列表**：存储所有边的集合

### 常见图算法分类
- **遍历算法**：BFS、DFS
- **最短路径**：Dijkstra、Floyd-Warshall、Bellman-Ford
- **最小生成树**：Prim、Kruskal
- **拓扑排序**：Kahn算法、DFS

---

## 1. 深度优先搜索 (DFS)

### 原理
从起始节点出发，尽可能深地访问节点，直到无法继续再回溯。

### 实现
```go
type Graph struct {
    adj map[int][]int
}

func NewGraph() *Graph {
    return &Graph{adj: make(map[int][]int)}
}

func (g *Graph) AddEdge(u, v int) {
    g.adj[u] = append(g.adj[u], v)
    g.adj[v] = append(g.adj[v], u)
}

func (g *Graph) DFS(start int) []int {
    visited := make(map[int]bool)
    result := []int{}
    g.dfsHelper(start, visited, &result)
    return result
}

func (g *Graph) dfsHelper(node int, visited map[int]bool, result *[]int) {
    visited[node] = true
    *result = append(*result, node)
    
    for _, neighbor := range g.adj[node] {
        if !visited[neighbor] {
            g.dfsHelper(neighbor, visited, result)
        }
    }
}
```

### 复杂度
- **时间复杂度**：O(V + E)
- **空间复杂度**：O(V)

---

## 2. 广度优先搜索 (BFS)

### 原理
从起始节点出发，逐层访问所有相邻节点。

### 实现
```go
func (g *Graph) BFS(start int) []int {
    visited := make(map[int]bool)
    queue := []int{start}
    visited[start] = true
    result := []int{}
    
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        result = append(result, node)
        
        for _, neighbor := range g.adj[node] {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(V + E)
- **空间复杂度**：O(V)

---

## 3. Dijkstra 算法

### 原理
用于计算从单个源节点到所有其他节点的最短路径，适用于非负权边。

### 实现
```go
import "container/heap"

type Edge struct {
    to     int
    weight int
}

type WeightedGraph struct {
    adj map[int][]Edge
}

func NewWeightedGraph() *WeightedGraph {
    return &WeightedGraph{adj: make(map[int][]Edge)}
}

func (g *WeightedGraph) AddEdge(u, v, w int) {
    g.adj[u] = append(g.adj[u], Edge{to: v, weight: w})
}

type PriorityQueue []*Item

type Item struct {
    node     int
    distance int
    index    int
}

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].distance < pq[j].distance
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    old[n-1] = nil
    item.index = -1
    *pq = old[0 : n-1]
    return item
}

func (g *WeightedGraph) Dijkstra(start int) map[int]int {
    distances := make(map[int]int)
    for node := range g.adj {
        distances[node] = int(^uint(0) >> 1) // 无穷大
    }
    distances[start] = 0
    
    pq := make(PriorityQueue, 0)
    heap.Init(&pq)
    heap.Push(&pq, &Item{node: start, distance: 0})
    
    visited := make(map[int]bool)
    
    for pq.Len() > 0 {
        item := heap.Pop(&pq).(*Item)
        node := item.node
        
        if visited[node] {
            continue
        }
        visited[node] = true
        
        for _, edge := range g.adj[node] {
            if !visited[edge.to] && distances[edge.to] > distances[node]+edge.weight {
                distances[edge.to] = distances[node] + edge.weight
                heap.Push(&pq, &Item{node: edge.to, distance: distances[edge.to]})
            }
        }
    }
    
    return distances
}
```

### 复杂度
- **时间复杂度**：O((V + E) log V)
- **空间复杂度**：O(V)

---

## 4. Floyd-Warshall 算法

### 原理
计算所有节点对之间的最短路径，适用于有负权边但无负权环的图。

### 实现
```go
func FloydWarshall(graph [][]int) [][]int {
    n := len(graph)
    dist := make([][]int, n)
    
    for i := range dist {
        dist[i] = make([]int, n)
        copy(dist[i], graph[i])
    }
    
    for k := 0; k < n; k++ {
        for i := 0; i < n; i++ {
            for j := 0; j < n; j++ {
                if dist[i][k]+dist[k][j] < dist[i][j] {
                    dist[i][j] = dist[i][k] + dist[k][j]
                }
            }
        }
    }
    
    return dist
}
```

### 复杂度
- **时间复杂度**：O(V³)
- **空间复杂度**：O(V²)

---

## 5. Prim 算法

### 原理
构建最小生成树，从一个顶点开始，每次添加权值最小的边。

### 实现
```go
func (g *WeightedGraph) Prim(start int) (int, []Edge) {
    visited := make(map[int]bool)
    pq := make(PriorityQueue, 0)
    heap.Init(&pq)
    
    totalWeight := 0
    mst := []Edge{}
    
    visited[start] = true
    for _, edge := range g.adj[start] {
        heap.Push(&pq, &Item{node: edge.to, distance: edge.weight})
    }
    
    for pq.Len() > 0 {
        item := heap.Pop(&pq).(*Item)
        node := item.node
        weight := item.distance
        
        if visited[node] {
            continue
        }
        
        visited[node] = true
        totalWeight += weight
        
        for _, edge := range g.adj[node] {
            if !visited[edge.to] {
                heap.Push(&pq, &Item{node: edge.to, distance: edge.weight})
            }
        }
    }
    
    return totalWeight, mst
}
```

### 复杂度
- **时间复杂度**：O((V + E) log V)
- **空间复杂度**：O(V)

---

## 6. Kruskal 算法

### 原理
构建最小生成树，按权值从小到大排序边，使用并查集避免环。

### 实现
```go
type UnionFind struct {
    parent []int
    rank   []int
}

func NewUnionFind(n int) *UnionFind {
    parent := make([]int, n)
    rank := make([]int, n)
    for i := range parent {
        parent[i] = i
    }
    return &UnionFind{parent: parent, rank: rank}
}

func (uf *UnionFind) Find(x int) int {
    if uf.parent[x] != x {
        uf.parent[x] = uf.Find(uf.parent[x])
    }
    return uf.parent[x]
}

func (uf *UnionFind) Union(x, y int) bool {
    rootX := uf.Find(x)
    rootY := uf.Find(y)
    
    if rootX == rootY {
        return false
    }
    
    if uf.rank[rootX] < uf.rank[rootY] {
        uf.parent[rootX] = rootY
    } else {
        uf.parent[rootY] = rootX
        if uf.rank[rootX] == uf.rank[rootY] {
            uf.rank[rootX]++
        }
    }
    
    return true
}

type EdgeWithWeight struct {
    u, v, w int
}

func Kruskal(edges []EdgeWithWeight, n int) (int, []EdgeWithWeight) {
    sort.Slice(edges, func(i, j int) bool {
        return edges[i].w < edges[j].w
    })
    
    uf := NewUnionFind(n)
    totalWeight := 0
    mst := []EdgeWithWeight{}
    
    for _, edge := range edges {
        if uf.Union(edge.u, edge.v) {
            totalWeight += edge.w
            mst = append(mst, edge)
            if len(mst) == n-1 {
                break
            }
        }
    }
    
    return totalWeight, mst
}
```

### 复杂度
- **时间复杂度**：O(E log E)
- **空间复杂度**：O(E)

---

## 7. 拓扑排序 (Topological Sort)

### 原理
对有向无环图 (DAG) 进行排序，使得所有边从前面的节点指向后面的节点。

### 实现
```go
func (g *Graph) TopologicalSort() []int {
    inDegree := make(map[int]int)
    for node := range g.adj {
        inDegree[node] = 0
    }
    
    for _, neighbors := range g.adj {
        for _, neighbor := range neighbors {
            inDegree[neighbor]++
        }
    }
    
    queue := []int{}
    for node, degree := range inDegree {
        if degree == 0 {
            queue = append(queue, node)
        }
    }
    
    result := []int{}
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        result = append(result, node)
        
        for _, neighbor := range g.adj[node] {
            inDegree[neighbor]--
            if inDegree[neighbor] == 0 {
                queue = append(queue, neighbor)
            }
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(V + E)
- **空间复杂度**：O(V)

---

## 图算法对比

| 算法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|----------|-----------|----------|
| DFS | O(V+E) | O(V) | 连通性检测、路径搜索 |
| BFS | O(V+E) | O(V) | 最短路径（无权图） |
| Dijkstra | O((V+E)logV) | O(V) | 单源最短路径（非负权） |
| Floyd-Warshall | O(V³) | O(V²) | 所有节点对最短路径 |
| Prim | O((V+E)logV) | O(V) | 稠密图最小生成树 |
| Kruskal | O(E log E) | O(E) | 稀疏图最小生成树 |
| 拓扑排序 | O(V+E) | O(V) | DAG排序 |

## 常见面试题

### 1. DFS 和 BFS 的区别？

**答案：**
- **DFS**：深度优先，使用栈或递归，可能找到路径但不一定最短
- **BFS**：广度优先，使用队列，能找到最短路径（无权图）

### 2. Dijkstra 算法为什么不能处理负权边？

**答案：**
Dijkstra 算法假设一旦节点被加入最短路径树，其最短距离就确定了。但负权边可能让已确定的路径变得更长，导致结果错误。

### 3. 如何检测图中是否存在环？

**答案：**
- **无向图**：使用 DFS，记录父节点，发现回边且不是父节点则有环
- **有向图**：使用 DFS，记录递归栈，发现节点在递归栈中则有环

### 4. Prim 和 Kruskal 算法的区别？

**答案：**
- **Prim**：从顶点出发，逐步扩展到相邻顶点
- **Kruskal**：按边权排序，依次添加不形成环的边

### 5. 拓扑排序的应用场景？

**答案：**
任务调度、课程安排、编译顺序等需要依赖关系排序的场景。

<center>...未完待续...</center>
---  
***  

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>
<script>
  var gitalk = new Gitalk({
    "clientID": "44d7c96f948be236a8c9",
    "clientSecret": "fb9fb3178db6640131c4e3eb69f9449e42bba661",
    "repo": "blog",
    "owner": "Zhang-jie-jun",
    "admin": ["Zhang-jie-jun"],
    "id": location.pathname,      
    "distractionFreeMode": false  
  });
  gitalk.render("gitalk-container");
</script>
