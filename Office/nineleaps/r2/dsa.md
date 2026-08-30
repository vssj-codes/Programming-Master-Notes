# DSA — Round 2 Prep (Uber Client Round)

---

## 1. Two Sum Problem

### Problem
Given an array of integers and a target, return indices of two numbers that add up to the target.

```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]  (because nums[0] + nums[1] = 2 + 7 = 9)
```

---

### Approach 1: Brute Force — O(n²)
Check every pair.

```python
def two_sum(nums, target):
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]
```

Too slow for large inputs.

---

### Approach 2: HashMap — O(n)
For each number, check if its **complement** (target - num) is already in the map.

```python
def two_sum(nums, target):
    seen = {}  # value -> index

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i
```

**Walk through:**
```
nums = [2, 7, 11, 15], target = 9

i=0, num=2, complement=7 → not in seen → seen={2:0}
i=1, num=7, complement=2 → 2 IS in seen → return [0, 1]
```

---

### Key Insight
Instead of looking forward (brute force), look backward — "have I seen the complement already?"

---

### Complexity
| | Time | Space |
|---|---|---|
| Brute Force | O(n²) | O(1) |
| HashMap | O(n) | O(n) |

---

### Variations to Know
- **Sorted array** → use two pointers (left, right) — O(n) time, O(1) space
- **All pairs** → return all pairs, not just indices
- **Three Sum** → fix one number, two sum on the rest

**Two Pointers (sorted array):**
```python
def two_sum_sorted(nums, target):
    left, right = 0, len(nums) - 1

    while left < right:
        total = nums[left] + nums[right]
        if total == target:
            return [left, right]
        elif total < target:
            left += 1
        else:
            right -= 1
```

---

## 2. DFS — Word Search Problem

### Problem
Given a 2D grid of characters and a word, return True if the word exists in the grid.
The word must be formed by sequentially adjacent cells (up, down, left, right). Cannot reuse a cell.

```
board = [
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]
word = "ABCCED" → True
word = "SEE"    → True
word = "ABCB"   → False (can't reuse B)
```

---

### What is DFS?
Depth-First Search — explore as far as possible down one path before backtracking.

Think of it like navigating a maze:
- Try one direction
- Keep going until you hit a dead end or find the exit
- Backtrack and try another direction

---

### Approach: DFS + Backtracking

```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def dfs(r, c, index):
        # base case: all characters matched
        if index == len(word):
            return True

        # out of bounds or wrong character or already visited
        if (r < 0 or r >= rows or
            c < 0 or c >= cols or
            board[r][c] != word[index]):
            return False

        # mark cell as visited
        temp = board[r][c]
        board[r][c] = "#"

        # explore all 4 directions
        found = (dfs(r+1, c, index+1) or
                 dfs(r-1, c, index+1) or
                 dfs(r, c+1, index+1) or
                 dfs(r, c-1, index+1))

        # restore cell (backtrack)
        board[r][c] = temp

        return found

    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True

    return False
```

---

### Walk Through (word = "ABC")
```
board:
A B C
S F C

Start at A (0,0), index=0 → match
  Go right to B (0,1), index=1 → match
    Go right to C (0,2), index=2 → match
      index=3 == len("ABC") → return True
```

---

### Why Mark as Visited?
Without marking, you could revisit the same cell:
```
word = "AA"
board = [['A']]
Without marking: A → A (same cell) → True (wrong!)
With marking:    A → # (blocked)   → False (correct)
```

---

### Backtracking
After DFS returns (success or failure), **restore the cell** so other paths can use it.

```python
temp = board[r][c]
board[r][c] = "#"    # mark visited
# ... explore ...
board[r][c] = temp   # restore
```

---

### Complexity
- **Time:** O(rows × cols × 4^len(word)) — for each cell, explore 4 directions recursively
- **Space:** O(len(word)) — recursion stack depth

---

## 3. Binary Tree DFS Problems

### Binary Tree Structure
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

```
        1
       / \
      2   3
     / \
    4   5
```

---

### DFS Traversals
Three ways to traverse — differ only in when you process the current node.

```python
# Preorder: root → left → right
def preorder(node):
    if not node:
        return
    print(node.val)       # process first
    preorder(node.left)
    preorder(node.right)
# Output: 1 2 4 5 3

# Inorder: left → root → right
def inorder(node):
    if not node:
        return
    inorder(node.left)
    print(node.val)       # process in middle
    inorder(node.right)
# Output: 4 2 5 1 3  (sorted order for BST)

# Postorder: left → right → root
def postorder(node):
    if not node:
        return
    postorder(node.left)
    postorder(node.right)
    print(node.val)       # process last
# Output: 4 5 2 3 1
```

---

### Problem 1: Maximum Depth of Binary Tree

```
        1
       / \
      2   3
     / \
    4   5

Max depth = 3
```

```python
def max_depth(root):
    if not root:
        return 0

    left_depth = max_depth(root.left)
    right_depth = max_depth(root.right)

    return 1 + max(left_depth, right_depth)
```

**Think recursively:** depth of tree = 1 + max(depth of left subtree, depth of right subtree)

---

### Problem 2: Path Sum
Given a tree and a target sum, return True if there's a root-to-leaf path that sums to target.

```
        5
       / \
      4   8
     /   / \
    11  13   4
   /  \       \
  7    2       1

target = 22 → True (5→4→11→2)
```

```python
def has_path_sum(root, target):
    if not root:
        return False

    # leaf node
    if not root.left and not root.right:
        return root.val == target

    remaining = target - root.val
    return (has_path_sum(root.left, remaining) or
            has_path_sum(root.right, remaining))
```

---

### Problem 3: Lowest Common Ancestor (LCA)
Find the lowest common ancestor of two nodes p and q.

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

LCA(5, 1) = 3
LCA(5, 4) = 5
```

```python
def lowest_common_ancestor(root, p, q):
    if not root:
        return None

    # if current node is p or q, it's the LCA
    if root == p or root == q:
        return root

    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)

    # p found on one side, q found on other → current node is LCA
    if left and right:
        return root

    # both on same side
    return left if left else right
```

---

### Problem 4: Validate Binary Search Tree (BST)
In a BST: all left nodes < current < all right nodes.

```python
def is_valid_bst(root, min_val=float('-inf'), max_val=float('inf')):
    if not root:
        return True

    if root.val <= min_val or root.val >= max_val:
        return False

    return (is_valid_bst(root.left, min_val, root.val) and
            is_valid_bst(root.right, root.val, max_val))
```

**Key:** Pass bounds down — left subtree must be less than current, right must be greater.

---

### Problem 5: Diameter of Binary Tree
Longest path between any two nodes (doesn't have to pass through root).

```python
def diameter_of_binary_tree(root):
    max_diameter = [0]

    def depth(node):
        if not node:
            return 0

        left = depth(node.left)
        right = depth(node.right)

        # diameter through this node = left depth + right depth
        max_diameter[0] = max(max_diameter[0], left + right)

        return 1 + max(left, right)

    depth(root)
    return max_diameter[0]
```

---

### DFS Pattern — Template to Remember

```python
def dfs(node):
    # base case
    if not node:
        return base_value

    # recurse
    left = dfs(node.left)
    right = dfs(node.right)

    # combine results
    return something(left, right, node.val)
```

Almost every binary tree DFS problem fits this template.

---

## Quick Reference

| Problem | Key Technique |
|---|---|
| Two Sum | HashMap — store complement |
| Two Sum (sorted) | Two pointers |
| Word Search | DFS + backtracking, mark visited |
| Max Depth | Postorder DFS, return 1 + max(left, right) |
| Path Sum | DFS, subtract from target |
| LCA | If found on both sides → current is LCA |
| Validate BST | Pass min/max bounds down |
| Diameter | Track max(left + right) at each node |
