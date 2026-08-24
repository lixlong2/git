# 54. 螺旋矩阵

题目链接：https://leetcode.cn/problems/spiral-matrix/

难度：Medium

标签：数组、矩阵、模拟

## 题目内容

给定一个 `m x n` 的矩阵 `matrix`，按照顺时针螺旋顺序，返回矩阵中的所有元素。

例子：

```text
输入：
[
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
]

输出：[1, 2, 3, 6, 9, 8, 7, 4, 5]
```

## 第一次代码：用 i/j 状态判断方向

第一次的想法是维护四个边界：

- `row1`：上边界
- `row2`：下边界
- `col1`：左边界
- `col2`：右边界

同时用 `i` 和 `j` 判断当前走到了哪一条边，再决定下一步应该往哪个方向遍历。

```python
class Solution(object):
    def spiralOrder(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: List[int]
        """
        row1 = 0
        col1 = 0
        row2 = len(matrix) - 1
        col2 = len(matrix[0]) - 1
        data = []
        i = 0
        j = -1

        while row1 != row2 or col1 != col2:
            if i == row1:
                if j < col1:
                    for k in matrix[row1][col1:col2 + 1]:
                        data.append(k)
                    row1 += 1
                    j = col2
                else:
                    for k in range(row1, row2 + 1):
                        data.append(matrix[k][col2])
                    col2 -= 1
                    i = row2

            if i == row2:
                if j < col1:
                    for k in range(col2, col1 - 1, -1):
                        data.append(matrix[row2][k])
                    row2 -= 1
                    j = col1
                else:
                    for k in range(row2, row1 - 1, -1):
                        data.append(matrix[k][col1])
                    col1 += 1
                    i = row1

        data.append(matrix[row1][col1])
        return data
```

## 第一次问题

这份代码的方向感是对的：已经想到了四个边界，也想到了每走完一条边就收缩边界。

但问题在于状态太复杂：

- `while row1 != row2 or col1 != col2` 这个结束条件不稳定。它假设最后一定只剩一个元素，但矩阵可能最后剩一行或一列。
- `data.append(matrix[row1][col1])` 也是假设最后只剩一个格子，所以容易漏掉或重复处理元素。
- 单行矩阵、单列矩阵、长方形矩阵都容易出问题。
- 用 `i` 和 `j` 判断当前方向，会让“当前位置”和“四个边界”的关系变得绕。
- 原始代码里 `return data` 的缩进如果放到函数外，也会导致代码结构错误。

这类题的难点不是某个算法公式，而是边界收缩时不要重复、不要漏掉。

## 第二次代码：按四条边一圈一圈收缩

第二次写法更稳定：不再用 `i/j` 判断方向，而是每一圈固定做四件事。

1. 从左到右走上边界。
2. 从上到下走右边界。
3. 如果还剩有效行，从右到左走下边界。
4. 如果还剩有效列，从下到上走左边界。

```python
class Solution(object):
    def spiralOrder(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: List[int]
        """
        if not matrix:
            return []

        row1 = 0
        col1 = 0
        row2 = len(matrix) - 1
        col2 = len(matrix[0]) - 1
        data = []

        while row1 <= row2 and col1 <= col2:
            # 从左到右：上边界
            for j in range(col1, col2 + 1):
                data.append(matrix[row1][j])
            row1 += 1

            # 从上到下：右边界
            for i in range(row1, row2 + 1):
                data.append(matrix[i][col2])
            col2 -= 1

            # 从右到左：下边界
            if row1 <= row2:
                for j in range(col2, col1 - 1, -1):
                    data.append(matrix[row2][j])
                row2 -= 1

            # 从下到上：左边界
            if col1 <= col2:
                for i in range(row2, row1 - 1, -1):
                    data.append(matrix[i][col1])
                col1 += 1

        return data
```

## 为什么第二份更稳

第二份代码的核心是：

```text
不要判断当前方向。
每一圈都按固定顺序处理四条边。
```

这样每一步都更清楚：

- 走完上边界，就 `row1 += 1`。
- 走完右边界，就 `col2 -= 1`。
- 走完下边界，就 `row2 -= 1`。
- 走完左边界，就 `col1 += 1`。

其中两个判断很关键：

```python
if row1 <= row2:
```

用来防止只剩一行时重复走下边界。

```python
if col1 <= col2:
```

用来防止只剩一列时重复走左边界。

## 复杂度

- 时间复杂度：`O(m * n)`，每个元素只访问一次。
- 空间复杂度：`O(1)`，不算返回结果数组。

## 边界检查

- 空矩阵：`matrix = []`，需要返回 `[]`。
- 只有一个元素：`[[1]]`。
- 只有一行：`[[1, 2, 3]]`。
- 只有一列：`[[1], [2], [3]]`。
- 行数和列数不相等的长方形矩阵。
- 最后一圈只剩一行或一列时，不能重复访问。

## 复盘

这题第一份代码不是完全错，而是把“方向状态”写得太复杂。

第一份其实已经抓到了关键：

```text
螺旋矩阵需要维护四个边界。
```

但更自然的写法是：

```text
固定顺序走四条边，然后收缩边界。
```

以后遇到矩阵模拟题，可以先问：

```text
我能不能把过程拆成几个固定动作？
每个动作结束后，哪条边界应该收缩？
什么时候需要判断是否还有有效行或有效列？
```

螺旋矩阵的重点不是“当前位置在哪里”，而是“四个边界还剩多少有效区域”。
