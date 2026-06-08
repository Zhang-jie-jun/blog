# 树算法

## 树算法简介
### 什么是树算法？
树算法是处理树形数据结构的算法，树由节点和边组成，具有层次结构。树算法广泛应用于文件系统、数据库索引、编译器设计等领域。

### 树的基本概念
- **根节点**：树的最顶层节点
- **叶子节点**：没有子节点的节点
- **深度**：从根节点到该节点的路径长度
- **高度**：从该节点到最远叶子节点的路径长度

### 常见树类型及设计目的

#### 1. 二叉树 (Binary Tree)
**产生原因**：在计算机中，树形结构是表达层次关系的自然方式。二叉树是最简单的树形结构，每个节点最多有两个子节点。

**解决的问题**：
- 表达层次化数据（如文件系统目录）
- 作为更复杂树结构的基础

#### 2. 二叉搜索树 (Binary Search Tree, BST)
**产生原因**：普通二叉树的查找效率为O(n)，无法满足高效查找需求。BST通过约束节点值的顺序，将查找效率提升到O(log n)。

**核心性质**：左子树所有节点 < 根节点 < 右子树所有节点

**解决的问题**：
- 高效的动态集合操作（插入、删除、查找）
- 有序数据的维护

**存在的问题**：
- 最坏情况下退化为链表（如插入有序数据），时间复杂度变为O(n)

#### 3. 平衡二叉搜索树 (Balanced BST)

##### AVL树
**产生原因**：解决BST可能退化为链表的问题。

**核心性质**：任意节点的左右子树高度差不超过1

**解决的问题**：
- 保证O(log n)的查找效率
- 适用于查找操作频繁的场景

**缺点**：插入和删除时旋转操作较多

##### 红黑树 (Red-Black Tree)
**产生原因**：在AVL树的基础上减少旋转次数，提高插入和删除性能。

**核心性质**：通过颜色约束（红/黑）保证树的近似平衡

**解决的问题**：
- 插入和删除操作更高效（平均旋转次数少于AVL树）
- 综合性能更优，是实际应用最广泛的平衡树

#### 4. B树 (B-tree)
**产生原因**：磁盘I/O操作比内存访问慢得多，需要减少树的高度以减少磁盘访问次数。

**核心性质**：多路搜索树，每个节点可以有多个子节点

**解决的问题**：
- 减少磁盘I/O次数
- 提高外存数据的访问效率

#### 5. B+树 (B+ Tree)
**产生原因**：在B树基础上优化范围查询性能，更适合数据库索引场景。

**核心性质**：所有数据存储在叶子节点，叶子节点形成链表

**解决的问题**：
- 高效的范围查询
- 更适合作为数据库索引
- 空间利用率更高

### 树结构对比

| 树类型 | 产生原因 | 核心解决的问题 | 时间复杂度 |
|--------|---------|---------------|-----------|
| 二叉树 | 表达层次关系 | 基础数据结构 | O(n) |
| BST | 高效查找 | 动态集合操作 | O(log n)~O(n) |
| AVL树 | 解决BST退化 | 保证查找效率 | O(log n) |
| 红黑树 | 平衡与效率兼顾 | 综合性能最优 | O(log n) |
| B树 | 减少磁盘I/O | 外存数据访问 | O(log_m n) |
| B+树 | 优化范围查询 | 数据库索引 | O(log_m n) |

---

## 1. 二叉树遍历

### 1.1 前序遍历 (Preorder)

**顺序**：根 -> 左 -> 右

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func PreorderTraversal(root *TreeNode) []int {
    result := []int{}
    preorderHelper(root, &result)
    return result
}

func preorderHelper(node *TreeNode, result *[]int) {
    if node == nil {
        return
    }
    *result = append(*result, node.Val)
    preorderHelper(node.Left, result)
    preorderHelper(node.Right, result)
}
```

### 1.2 中序遍历 (Inorder)

**顺序**：左 -> 根 -> 右

```go
func InorderTraversal(root *TreeNode) []int {
    result := []int{}
    inorderHelper(root, &result)
    return result
}

func inorderHelper(node *TreeNode, result *[]int) {
    if node == nil {
        return
    }
    inorderHelper(node.Left, result)
    *result = append(*result, node.Val)
    inorderHelper(node.Right, result)
}
```

### 1.3 后序遍历 (Postorder)

**顺序**：左 -> 右 -> 根

```go
func PostorderTraversal(root *TreeNode) []int {
    result := []int{}
    postorderHelper(root, &result)
    return result
}

func postorderHelper(node *TreeNode, result *[]int) {
    if node == nil {
        return
    }
    postorderHelper(node.Left, result)
    postorderHelper(node.Right, result)
    *result = append(*result, node.Val)
}
```

### 1.4 层序遍历 (Level Order)

**顺序**：按层从上到下，从左到右

```go
func LevelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }
    
    queue := []*TreeNode{root}
    
    for len(queue) > 0 {
        levelSize := len(queue)
        level := []int{}
        
        for i := 0; i < levelSize; i++ {
            node := queue[0]
            queue = queue[1:]
            
            level = append(level, node.Val)
            
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        
        result = append(result, level)
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(n)（层序遍历）

---

## 2. 二叉树的最大深度

### 原理
递归计算左右子树的最大深度。

```go
func MaxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    
    leftDepth := MaxDepth(root.Left)
    rightDepth := MaxDepth(root.Right)
    
    return max(leftDepth, rightDepth) + 1
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(h)（h为树高）

---

## 3. 二叉树的最小深度

### 原理
递归计算左右子树的最小深度，注意叶子节点判断。

```go
func MinDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    
    if root.Left == nil && root.Right == nil {
        return 1
    }
    
    minD := int(^uint(0) >> 1)
    
    if root.Left != nil {
        minD = min(minD, MinDepth(root.Left))
    }
    if root.Right != nil {
        minD = min(minD, MinDepth(root.Right))
    }
    
    return minD + 1
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(h)

---

## 4. 验证二叉搜索树

### 原理
利用中序遍历有序的特性，或者递归检查范围。

```go
func IsValidBST(root *TreeNode) bool {
    return isValidBSTHelper(root, nil, nil)
}

func isValidBSTHelper(node *TreeNode, min, max *int) bool {
    if node == nil {
        return true
    }
    
    if min != nil && node.Val <= *min {
        return false
    }
    if max != nil && node.Val >= *max {
        return false
    }
    
    return isValidBSTHelper(node.Left, min, &node.Val) &&
           isValidBSTHelper(node.Right, &node.Val, max)
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(h)

---

## 5. 二叉树的最近公共祖先

### 原理
递归查找，若左右子树都找到则当前节点是LCA。

```go
func LowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || root == p || root == q {
        return root
    }
    
    left := LowestCommonAncestor(root.Left, p, q)
    right := LowestCommonAncestor(root.Right, p, q)
    
    if left != nil && right != nil {
        return root
    }
    
    if left != nil {
        return left
    }
    return right
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(h)

---

## 6. 二叉树的直径

### 原理
直径 = 左子树深度 + 右子树深度。

```go
func DiameterOfBinaryTree(root *TreeNode) int {
    diameter := 0
    diameterHelper(root, &diameter)
    return diameter
}

func diameterHelper(node *TreeNode, diameter *int) int {
    if node == nil {
        return 0
    }
    
    left := diameterHelper(node.Left, diameter)
    right := diameterHelper(node.Right, diameter)
    
    *diameter = max(*diameter, left+right)
    
    return max(left, right) + 1
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(h)

---

## 7. 二叉搜索树的插入

### 原理
根据BST性质找到合适位置插入。

```go
func InsertIntoBST(root *TreeNode, val int) *TreeNode {
    if root == nil {
        return &TreeNode{Val: val}
    }
    
    if val < root.Val {
        root.Left = InsertIntoBST(root.Left, val)
    } else {
        root.Right = InsertIntoBST(root.Right, val)
    }
    
    return root
}
```

### 复杂度
- **时间复杂度**：O(h)
- **空间复杂度**：O(h)

---

## 8. 红黑树 (Red-Black Tree)

### 红黑树简介
红黑树是一种自平衡二叉搜索树，通过颜色约束保证树的高度始终为 O(log n)。

### 红黑树的性质
1. **每个节点要么是红色，要么是黑色**
2. **根节点是黑色**
3. **所有叶子节点（NIL节点）是黑色**
4. **如果一个节点是红色，则它的两个子节点都是黑色**
5. **从任意节点到其所有后代叶子节点的路径上，黑色节点的数量相同**

### 节点结构
```go
const (
    RED   = true
    BLACK = false
)

type RBNode struct {
    Val    int
    Color  bool
    Left   *RBNode
    Right  *RBNode
    Parent *RBNode
}

type RBTree struct {
    Root *RBNode
    NIL  *RBNode // 哨兵节点
}

func NewRBTree() *RBTree {
    nilNode := &RBNode{Color: BLACK}
    return &RBTree{NIL: nilNode, Root: nilNode}
}
```

### 左旋操作
```go
func (t *RBTree) LeftRotate(x *RBNode) {
    y := x.Right
    x.Right = y.Left
    
    if y.Left != t.NIL {
        y.Left.Parent = x
    }
    
    y.Parent = x.Parent
    
    if x.Parent == t.NIL {
        t.Root = y
    } else if x == x.Parent.Left {
        x.Parent.Left = y
    } else {
        x.Parent.Right = y
    }
    
    y.Left = x
    x.Parent = y
}
```

### 右旋操作
```go
func (t *RBTree) RightRotate(y *RBNode) {
    x := y.Left
    y.Left = x.Right
    
    if x.Right != t.NIL {
        x.Right.Parent = y
    }
    
    x.Parent = y.Parent
    
    if y.Parent == t.NIL {
        t.Root = x
    } else if y == y.Parent.Left {
        y.Parent.Left = x
    } else {
        y.Parent.Right = x
    }
    
    x.Right = y
    y.Parent = x
}
```

### 插入操作
```go
func (t *RBTree) Insert(val int) {
    z := &RBNode{Val: val, Color: RED, Left: t.NIL, Right: t.NIL, Parent: t.NIL}
    
    // 找到插入位置
    y := t.NIL
    x := t.Root
    
    for x != t.NIL {
        y = x
        if z.Val < x.Val {
            x = x.Left
        } else {
            x = x.Right
        }
    }
    
    z.Parent = y
    
    if y == t.NIL {
        t.Root = z
    } else if z.Val < y.Val {
        y.Left = z
    } else {
        y.Right = z
    }
    
    // 修复红黑树性质
    t.InsertFixup(z)
}

func (t *RBTree) InsertFixup(z *RBNode) {
    for z.Parent.Color == RED {
        if z.Parent == z.Parent.Parent.Left {
            // 父节点是祖父节点的左孩子
            y := z.Parent.Parent.Right // 叔叔节点
            
            if y.Color == RED {
                // 情况1：叔叔是红色
                z.Parent.Color = BLACK
                y.Color = BLACK
                z.Parent.Parent.Color = RED
                z = z.Parent.Parent
            } else {
                if z == z.Parent.Right {
                    // 情况2：叔叔是黑色，当前节点是右孩子
                    z = z.Parent
                    t.LeftRotate(z)
                }
                // 情况3：叔叔是黑色，当前节点是左孩子
                z.Parent.Color = BLACK
                z.Parent.Parent.Color = RED
                t.RightRotate(z.Parent.Parent)
            }
        } else {
            // 父节点是祖父节点的右孩子（对称情况）
            y := z.Parent.Parent.Left
            
            if y.Color == RED {
                z.Parent.Color = BLACK
                y.Color = BLACK
                z.Parent.Parent.Color = RED
                z = z.Parent.Parent
            } else {
                if z == z.Parent.Left {
                    z = z.Parent
                    t.RightRotate(z)
                }
                z.Parent.Color = BLACK
                z.Parent.Parent.Color = RED
                t.LeftRotate(z.Parent.Parent)
            }
        }
    }
    
    t.Root.Color = BLACK
}
```

### 删除操作
```go
func (t *RBTree) Delete(val int) {
    z := t.Search(val)
    if z == t.NIL {
        return
    }
    
    y := z
    yOriginalColor := y.Color
    var x *RBNode
    
    if z.Left == t.NIL {
        x = z.Right
        t.Transplant(z, z.Right)
    } else if z.Right == t.NIL {
        x = z.Left
        t.Transplant(z, z.Left)
    } else {
        y = t.Minimum(z.Right)
        yOriginalColor = y.Color
        x = y.Right
        
        if y.Parent == z {
            x.Parent = y
        } else {
            t.Transplant(y, y.Right)
            y.Right = z.Right
            y.Right.Parent = y
        }
        
        t.Transplant(z, y)
        y.Left = z.Left
        y.Left.Parent = y
        y.Color = z.Color
    }
    
    if yOriginalColor == BLACK {
        t.DeleteFixup(x)
    }
}

func (t *RBTree) Transplant(u, v *RBNode) {
    if u.Parent == t.NIL {
        t.Root = v
    } else if u == u.Parent.Left {
        u.Parent.Left = v
    } else {
        u.Parent.Right = v
    }
    v.Parent = u.Parent
}

func (t *RBTree) Minimum(x *RBNode) *RBNode {
    for x.Left != t.NIL {
        x = x.Left
    }
    return x
}

func (t *RBTree) DeleteFixup(x *RBNode) {
    for x != t.Root && x.Color == BLACK {
        if x == x.Parent.Left {
            w := x.Parent.Right // 兄弟节点
            
            if w.Color == RED {
                // 情况1：兄弟是红色
                w.Color = BLACK
                x.Parent.Color = RED
                t.LeftRotate(x.Parent)
                w = x.Parent.Right
            }
            
            if w.Left.Color == BLACK && w.Right.Color == BLACK {
                // 情况2：兄弟的两个孩子都是黑色
                w.Color = RED
                x = x.Parent
            } else {
                if w.Right.Color == BLACK {
                    // 情况3：兄弟的右孩子是黑色
                    w.Left.Color = BLACK
                    w.Color = RED
                    t.RightRotate(w)
                    w = x.Parent.Right
                }
                // 情况4：兄弟的右孩子是红色
                w.Color = x.Parent.Color
                x.Parent.Color = BLACK
                w.Right.Color = BLACK
                t.LeftRotate(x.Parent)
                x = t.Root
            }
        } else {
            // 对称情况
            w := x.Parent.Left
            
            if w.Color == RED {
                w.Color = BLACK
                x.Parent.Color = RED
                t.RightRotate(x.Parent)
                w = x.Parent.Left
            }
            
            if w.Right.Color == BLACK && w.Left.Color == BLACK {
                w.Color = RED
                x = x.Parent
            } else {
                if w.Left.Color == BLACK {
                    w.Right.Color = BLACK
                    w.Color = RED
                    t.LeftRotate(w)
                    w = x.Parent.Left
                }
                w.Color = x.Parent.Color
                x.Parent.Color = BLACK
                w.Left.Color = BLACK
                t.RightRotate(x.Parent)
                x = t.Root
            }
        }
    }
    
    x.Color = BLACK
}

func (t *RBTree) Search(val int) *RBNode {
    current := t.Root
    for current != t.NIL && current.Val != val {
        if val < current.Val {
            current = current.Left
        } else {
            current = current.Right
        }
    }
    return current
}
```

### 复杂度
- **时间复杂度**：O(log n)（插入、删除、查找）
- **空间复杂度**：O(n)

---

## 9. B+树 (B+ Tree)

### B+树简介
B+树是一种多路搜索树，广泛应用于数据库索引和文件系统。

### B+树的特点
1. **所有数据存储在叶子节点**
2. **叶子节点形成链表，便于范围查询**
3. **内部节点只存储键，不存储数据**
4. **每个节点的子节点数在 [m/2, m] 之间（m为阶数）**

### B+树与B树的区别
| 特性 | B树 | B+树 |
|------|-----|------|
| 数据存储 | 所有节点都可存储 | 仅叶子节点存储 |
| 叶子节点 | 独立 | 形成链表 |
| 范围查询 | 效率较低 | 高效 |
| 空间利用率 | 较低 | 较高 |

### B+树节点结构
```go
const M = 3 // B+树的阶数

type BPlusTreeNode struct {
    Keys     []int
    Children []*BPlusTreeNode
    IsLeaf   bool
    Next     *BPlusTreeNode // 叶子节点的链表指针
}

type BPlusTree struct {
    Root  *BPlusTreeNode
    Order int
}

func NewBPlusTree(order int) *BPlusTree {
    return &BPlusTree{
        Order: order,
        Root: &BPlusTreeNode{
            Keys:     []int{},
            Children: []*BPlusTreeNode{},
            IsLeaf:   true,
            Next:     nil,
        },
    }
}
```

### 插入操作
```go
func (t *BPlusTree) Insert(val int) {
    root := t.Root
    
    if len(root.Keys) == t.Order-1 {
        // 根节点已满，需要分裂
        newRoot := &BPlusTreeNode{
            Keys:     []int{},
            Children: []*BPlusTreeNode{root},
            IsLeaf:   false,
            Next:     nil,
        }
        
        t.splitChild(newRoot, 0)
        t.insertNonFull(newRoot, val)
        t.Root = newRoot
    } else {
        t.insertNonFull(root, val)
    }
}

func (t *BPlusTree) insertNonFull(node *BPlusTreeNode, val int) {
    i := len(node.Keys) - 1
    
    if node.IsLeaf {
        // 叶子节点，直接插入
        node.Keys = append(node.Keys, 0)
        for i >= 0 && val < node.Keys[i] {
            node.Keys[i+1] = node.Keys[i]
            i--
        }
        node.Keys[i+1] = val
    } else {
        // 内部节点，找到合适的子节点
        for i >= 0 && val < node.Keys[i] {
            i--
        }
        i++
        
        if len(node.Children[i].Keys) == t.Order-1 {
            t.splitChild(node, i)
            if val > node.Keys[i] {
                i++
            }
        }
        
        t.insertNonFull(node.Children[i], val)
    }
}

func (t *BPlusTree) splitChild(parent *BPlusTreeNode, index int) {
    fullNode := parent.Children[index]
    newNode := &BPlusTreeNode{
        Keys:     make([]int, (t.Order-1)/2),
        Children: []*BPlusTreeNode{},
        IsLeaf:   fullNode.IsLeaf,
        Next:     nil,
    }
    
    // 复制后半部分键到新节点
    copy(newNode.Keys, fullNode.Keys[(t.Order)/2:])
    
    if !fullNode.IsLeaf {
        // 如果不是叶子节点，复制子节点
        newNode.Children = make([]*BPlusTreeNode, (t.Order+1)/2)
        copy(newNode.Children, fullNode.Children[(t.Order)/2:])
        fullNode.Children = fullNode.Children[:(t.Order)/2]
    } else {
        // 叶子节点，更新链表
        newNode.Next = fullNode.Next
        fullNode.Next = newNode
    }
    
    // 截断原节点
    fullNode.Keys = fullNode.Keys[:(t.Order-1)/2]
    
    // 在父节点中插入新节点
    parent.Keys = append(parent.Keys, 0)
    parent.Children = append(parent.Children, nil)
    
    for i := len(parent.Keys) - 1; i > index; i-- {
        parent.Keys[i] = parent.Keys[i-1]
        parent.Children[i+1] = parent.Children[i]
    }
    
    parent.Keys[index] = newNode.Keys[0]
    parent.Children[index+1] = newNode
}
```

### 查找操作
```go
func (t *BPlusTree) Search(val int) (*BPlusTreeNode, int) {
    current := t.Root
    
    for !current.IsLeaf {
        i := 0
        for i < len(current.Keys) && val > current.Keys[i] {
            i++
        }
        current = current.Children[i]
    }
    
    // 在叶子节点中查找
    for i, key := range current.Keys {
        if key == val {
            return current, i
        }
    }
    
    return nil, -1
}
```

### 范围查询
```go
func (t *BPlusTree) RangeQuery(start, end int) []int {
    result := []int{}
    
    // 找到起始节点
    current := t.Root
    for !current.IsLeaf {
        i := 0
        for i < len(current.Keys) && start > current.Keys[i] {
            i++
        }
        current = current.Children[i]
    }
    
    // 遍历叶子节点链表
    for current != nil {
        for _, key := range current.Keys {
            if key > end {
                return result
            }
            if key >= start {
                result = append(result, key)
            }
        }
        current = current.Next
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(log_m n)（m为阶数）
- **空间复杂度**：O(n)

---

## 树算法对比

| 算法 | 时间复杂度 | 空间复杂度 | 描述 |
|------|----------|-----------|------|
| 前序遍历 | O(n) | O(h) | 根-左-右 |
| 中序遍历 | O(n) | O(h) | 左-根-右 |
| 后序遍历 | O(n) | O(h) | 左-右-根 |
| 层序遍历 | O(n) | O(n) | 按层遍历 |
| 最大深度 | O(n) | O(h) | 递归计算 |
| 最小深度 | O(n) | O(h) | 递归计算 |
| 验证BST | O(n) | O(h) | 范围检查 |
| LCA | O(n) | O(h) | 递归查找 |
| BST插入 | O(h) | O(h) | 二叉搜索树插入 |
| 红黑树 | O(log n) | O(n) | 自平衡BST |
| B+树 | O(log_m n) | O(n) | 多路搜索树 |

## 常见面试题

### 1. 二叉树的三种深度优先遍历有什么区别？

**答案：**
- **前序**：根-左-右，适用于复制树、前缀表达式
- **中序**：左-根-右，BST中序遍历是有序的
- **后序**：左-右-根，适用于释放树、后缀表达式

### 2. 如何判断一棵二叉树是否是平衡树？

**答案：**
检查每个节点的左右子树高度差是否不超过1，同时递归检查左右子树是否平衡。

```go
func IsBalanced(root *TreeNode) bool {
    return checkBalance(root) != -1
}

func checkBalance(node *TreeNode) int {
    if node == nil {
        return 0
    }
    
    left := checkBalance(node.Left)
    if left == -1 {
        return -1
    }
    
    right := checkBalance(node.Right)
    if right == -1 {
        return -1
    }
    
    if abs(left-right) > 1 {
        return -1
    }
    
    return max(left, right) + 1
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### 3. 如何将二叉搜索树转换为有序双向链表？

**答案：**
使用中序遍历，记录前驱节点，将当前节点的左指针指向前驱，前驱的右指针指向当前节点。

### 4. 什么是完全二叉树？

**答案：**
完全二叉树是除最后一层外，其他层的节点数都达到最大，且最后一层的节点都集中在左侧。

### 5. B树和B+树的区别？

**答案：**
- **B树**：数据存储在所有节点中
- **B+树**：数据只存储在叶子节点，叶子节点形成链表，更适合范围查询

### 6. 如何在二叉树中找到第k个最大的节点？

**答案：**
使用逆中序遍历（右-根-左），遍历到第k个节点即为答案。

```go
func KthLargest(root *TreeNode, k int) int {
    count := 0
    result := 0
    kthLargestHelper(root, &count, k, &result)
    return result
}

func kthLargestHelper(node *TreeNode, count *int, k int, result *int) {
    if node == nil {
        return
    }
    
    kthLargestHelper(node.Right, count, k, result)
    
    *count++
    if *count == k {
        *result = node.Val
        return
    }
    
    kthLargestHelper(node.Left, count, k, result)
}
```

### 7. 如何判断两棵二叉树是否相同？

**答案：**
递归比较两个树的结构和节点值。

```go
func IsSameTree(p, q *TreeNode) bool {
    if p == nil && q == nil {
        return true
    }
    if p == nil || q == nil {
        return false
    }
    if p.Val != q.Val {
        return false
    }
    return IsSameTree(p.Left, q.Left) && IsSameTree(p.Right, q.Right)
}
```

### 8. 如何判断二叉树是否对称？

**答案：**
递归比较左子树的左节点与右子树的右节点，左子树的右节点与右子树的左节点。

```go
func IsSymmetric(root *TreeNode) bool {
    if root == nil {
        return true
    }
    return isMirror(root.Left, root.Right)
}

func isMirror(p, q *TreeNode) bool {
    if p == nil && q == nil {
        return true
    }
    if p == nil || q == nil {
        return false
    }
    return p.Val == q.Val && isMirror(p.Left, q.Right) && isMirror(p.Right, q.Left)
}
```

### 9. 如何从有序数组构建高度平衡的二叉搜索树？

**答案：**
每次取中间元素作为根节点，递归构建左右子树。

```go
func SortedArrayToBST(nums []int) *TreeNode {
    if len(nums) == 0 {
        return nil
    }
    
    mid := len(nums) / 2
    root := &TreeNode{Val: nums[mid]}
    root.Left = SortedArrayToBST(nums[:mid])
    root.Right = SortedArrayToBST(nums[mid+1:])
    
    return root
}
```

### 10. 如何合并两棵二叉搜索树？

**答案：**
方法一：将两棵树的中序遍历结果合并成一个有序数组，再构建平衡BST。

```go
func MergeTrees(root1, root2 *TreeNode) *TreeNode {
    // 获取两棵树的中序遍历
    list1 := inorder(root1)
    list2 := inorder(root2)
    
    // 合并有序数组
    merged := mergeSortedLists(list1, list2)
    
    // 构建平衡BST
    return sortedArrayToBST(merged)
}

func inorder(root *TreeNode) []int {
    result := []int{}
    if root != nil {
        result = append(result, inorder(root.Left)...)
        result = append(result, root.Val)
        result = append(result, inorder(root.Right)...)
    }
    return result
}

func mergeSortedLists(a, b []int) []int {
    result := []int{}
    i, j := 0, 0
    
    for i < len(a) && j < len(b) {
        if a[i] < b[j] {
            result = append(result, a[i])
            i++
        } else {
            result = append(result, b[j])
            j++
        }
    }
    
    result = append(result, a[i:]...)
    result = append(result, b[j:]...)
    
    return result
}
```

### 11. 如何在二叉树中找到路径和等于给定值的所有路径？

**答案：**
使用回溯法，遍历所有路径并记录路径和。

```go
func PathSum(root *TreeNode, targetSum int) [][]int {
    result := [][]int{}
    path := []int{}
    pathSumHelper(root, targetSum, path, &result)
    return result
}

func pathSumHelper(node *TreeNode, targetSum int, path []int, result *[][]int) {
    if node == nil {
        return
    }
    
    path = append(path, node.Val)
    
    if node.Left == nil && node.Right == nil && targetSum == node.Val {
        *result = append(*result, append([]int{}, path...))
    }
    
    pathSumHelper(node.Left, targetSum-node.Val, path, result)
    pathSumHelper(node.Right, targetSum-node.Val, path, result)
    
    path = path[:len(path)-1]
}
```

### 12. 红黑树的插入操作可能破坏哪些性质？如何修复？

**答案：**
插入操作可能破坏的性质：
1. 根节点必须是黑色
2. 如果一个节点是红色，则它的两个子节点都是黑色

修复方法（4种情况）：
- **情况1**：叔叔节点是红色 → 父节点和叔叔节点变黑，祖父节点变红
- **情况2**：叔叔节点是黑色，当前节点是右孩子 → 左旋父节点
- **情况3**：叔叔节点是黑色，当前节点是左孩子 → 父节点变黑，祖父节点变红，右旋祖父节点
- **对称情况**：处理右子树的镜像情况

### 13. B+树为什么适合作为数据库索引？

**答案：**
1. **范围查询高效**：叶子节点形成链表，顺序遍历即可
2. **空间利用率高**：内部节点不存储数据，可存储更多键
3. **磁盘友好**：高度低，减少磁盘I/O次数
4. **查询稳定**：所有查询都要到叶子节点，时间复杂度稳定

### 14. 如何实现二叉树的序列化和反序列化？

**答案：**
使用层序遍历进行序列化，用特殊符号表示空节点。

```go
type Codec struct{}

func (c *Codec) serialize(root *TreeNode) string {
    if root == nil {
        return "[]"
    }
    
    result := []string{}
    queue := []*TreeNode{root}
    
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        
        if node == nil {
            result = append(result, "null")
            continue
        }
        
        result = append(result, strconv.Itoa(node.Val))
        queue = append(queue, node.Left)
        queue = append(queue, node.Right)
    }
    
    return "[" + strings.Join(result, ",") + "]"
}

func (c *Codec) deserialize(data string) *TreeNode {
    if data == "[]" {
        return nil
    }
    
    values := strings.Split(data[1:len(data)-1], ",")
    rootVal, _ := strconv.Atoi(values[0])
    root := &TreeNode{Val: rootVal}
    queue := []*TreeNode{root}
    
    i := 1
    for len(queue) > 0 && i < len(values) {
        node := queue[0]
        queue = queue[1:]
        
        if values[i] != "null" {
            val, _ := strconv.Atoi(values[i])
            node.Left = &TreeNode{Val: val}
            queue = append(queue, node.Left)
        }
        i++
        
        if i < len(values) && values[i] != "null" {
            val, _ := strconv.Atoi(values[i])
            node.Right = &TreeNode{Val: val}
            queue = append(queue, node.Right)
        }
        i++
    }
    
    return root
}
```

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
