# 常用算法

## 目录结构

| 文档 | 内容 |
|------|------|
| [栈](stack.md) | 栈实现、有效的括号、最小栈、逆波兰表达式求值、用栈实现队列 |
| [队列](queue.md) | 队列实现、循环队列、优先队列、滑动窗口最大值、用队列实现栈 |
| [链表](linkedlist.md) | 单链表、反转链表、合并有序链表、环形链表、相交链表、双链表 |
| [哈希表](hash.md) | 哈希表实现、两数之和、最长无重复子串、字母异位词分组、LRU缓存 |
| [树算法](tree.md) | 二叉树遍历、最大深度、最小深度、验证BST、最近公共祖先、二叉树直径 |
| [图算法](graph.md) | DFS、BFS、Dijkstra、Floyd-Warshall、Prim、Kruskal、拓扑排序 |
| [排序算法](sort.md) | 冒泡排序、选择排序、插入排序、希尔排序、快速排序、归并排序、堆排序、计数排序、桶排序、基数排序 |
| [查找算法](search.md) | 顺序查找、二分查找、插值查找、哈希查找、二叉搜索树查找 |
| [动态规划](dp.md) | 斐波那契数列、最长递增子序列、最长公共子序列、背包问题、爬楼梯、最大子数组和、编辑距离 |
| [贪心算法](greedy.md) | 活动选择、区间调度、哈夫曼编码、硬币找零、跳跃游戏、合并区间 |
| [回溯算法](backtracking.md) | 子集、组合求和、全排列、N皇后、分割回文串、单词搜索 |
| [分治算法](divide.md) | 归并排序、快速排序、二分查找、最大子数组和、合并K链表、快速选择 |
| [字符串算法](string.md) | KMP、Boyer-Moore、Rabin-Karp、最长回文子串、字符串反转、最长公共前缀 |

## 算法分类

### 数据结构相关
- **栈与队列**：后进先出和先进先出的数据结构
- **链表**：线性链式存储结构
- **哈希表**：通过哈希函数实现快速访问
- **树算法**：处理树形结构的遍历和操作
- **图算法**：处理图结构的遍历、路径、生成树等问题

### 基础算法
- **排序算法**：将数据按特定顺序排列
- **查找算法**：在数据集中定位目标元素

### 高级算法
- **动态规划**：通过子问题最优解构建全局最优解
- **贪心算法**：每步选择局部最优以期望全局最优
- **回溯算法**：通过探索所有可能的候选解来找到所有解的算法
- **分治算法**：通过子问题的解合并得到原问题的解

### 字符串处理
- **字符串匹配**：在文本中查找模式串
- **字符串操作**：反转、转换、比较等

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