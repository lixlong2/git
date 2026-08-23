# 977. 有序数组的平方

题目链接：https://leetcode.cn/problems/squares-of-a-sorted-array/

难度：Easy

标签：数组、双指针、排序

## 题目内容

给定一个按非递减顺序排序的整数数组 `nums`，返回每个数字的平方组成的新数组，要求结果也按非递减顺序排序。

例子：

```text
输入：nums = [-4, -1, 0, 3, 10]
输出：[0, 1, 9, 16, 100]
```

## 第一次代码：我的错误尝试

第一次的思路是：

1. 先把每个数原地平方。
2. 平方之后，数组会变成“前半部分下降，后半部分上升”的形状。
3. 找到下降和上升的分界点。
4. 再从分界点左右两边合并出一个有序数组。

这个方向其实已经接近“归并”的想法了，但边界处理没有写完整，所以会出错。

```python
class Solution(object):
    def sortedSquares(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        n = len(nums)

        for i in range(n):
            nums[i] *= nums[i]

        k = 0
        while k <= n - 2 and nums[k + 1] <= nums[k]:
            k += 1

        nums2 = []

        if k == 0:
            return nums
        elif k == n - 1:
            for i in range(n):
                nums2.append(nums[n - i - 1])
            return nums2
        else:
            j = k + 1
            for i in range(n):
                if nums[k] <= nums[j]:
                    nums2.append(nums[k])
                    k -= 1
                else:
                    nums2.append(nums[j])
                    j += 1

        return nums2
```

## 第一次问题

- `k == 0` 时直接 `return nums` 不一定正确。比如 `nums = [-2, -1, 3]`，平方后是 `[4, 1, 9]`，还没有排好序。
- 合并时只写了 `nums[k] <= nums[j]`，但没有判断 `k` 是否已经小于 `0`。
- 合并时也没有判断 `j` 是否已经等于 `n`，所以可能数组越界。
- `for i in range(n)` 固定循环 `n` 次，但左右两边有一边用完后，应该直接把另一边剩下的元素补进去。
- 这份代码的问题不是思路完全错，而是“归并两个有序部分”时边界没有兜住。

## 第二次代码：基于原方法改进

继续沿用第一次的想法：

1. 先平方。
2. 找到左右两段的分界点。
3. 左边从 `k` 往左走，右边从 `k + 1` 往右走。
4. 每次取更小的平方数放入结果数组。
5. 如果一边走完了，就把另一边剩下的元素全部加入结果。

```python
class Solution(object):
    def sortedSquares(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        n = len(nums)

        for i in range(n):
            nums[i] *= nums[i]

        k = 0
        while k <= n - 2 and nums[k + 1] <= nums[k]:
            k += 1

        left = k
        right = k + 1
        result = []

        while left >= 0 and right < n:
            if nums[left] <= nums[right]:
                result.append(nums[left])
                left -= 1
            else:
                result.append(nums[right])
                right += 1

        while left >= 0:
            result.append(nums[left])
            left -= 1

        while right < n:
            result.append(nums[right])
            right += 1

        return result
```

## 第二次复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## 第三次代码：左右双指针

这题更经典的做法是直接利用原数组已经有序这一点。

原数组从小到大排列，但平方以后，最大的数一定可能来自：

- 最左边的负数，因为绝对值可能很大。
- 最右边的正数。

所以可以用左右双指针：

- `left` 指向数组开头。
- `right` 指向数组末尾。
- 比较 `abs(nums[left])` 和 `abs(nums[right])`。
- 谁的绝对值更大，就把谁的平方放到结果数组的末尾。

```python
class Solution(object):
    def sortedSquares(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        n = len(nums)
        result = [0] * n

        left = 0
        right = n - 1
        pos = n - 1

        while left <= right:
            if abs(nums[left]) > abs(nums[right]):
                result[pos] = nums[left] * nums[left]
                left += 1
            else:
                result[pos] = nums[right] * nums[right]
                right -= 1

            pos -= 1

        return result
```

## 第三次复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## 复盘

这题我一开始没有直接想到左右双指针，是因为注意力放在了“平方之后怎么排序”上。

但更关键的观察应该是：原数组本来就是有序的，平方之后的最大值一定出现在两端，而不是中间。

这和第 27 题“移除元素”的双指针不太一样：

- 第 27 题的快慢指针，是用来原地保留有效元素。
- 第 977 题的左右双指针，是利用有序数组两端的绝对值最大，把结果从后往前填。

以后遇到这类题，可以提醒自己：

- 如果数组已经有序，先想能不能利用有序性。
- 如果平方、绝对值、距离这类操作会改变顺序，就想想最大值或最小值会不会出现在两端。
- 如果结果需要有序，可以考虑从结果数组的末尾开始填。
