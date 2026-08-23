# 27. 移除元素

题目链接：https://leetcode.cn/problems/remove-element/

难度：Easy

标签：数组、双指针、原地修改

## 题目内容

给定一个数组 `nums` 和一个值 `val`，需要原地移除数组中所有等于 `val` 的元素，并返回移除后数组的新长度。

题目只关心返回的新长度 `k`，以及 `nums` 前 `k` 个元素是否是不等于 `val` 的元素。超过 `k` 的部分不需要关心。

## 第一次思路：左右双指针交换

第一次想到的是用两个指针：

- `i` 从左往右找第一个等于 `val` 的元素。
- `j` 从右往左找最后一个不等于 `val` 的元素。
- 如果 `i < j`，就把右边这个有效元素放到左边需要删除的位置。

这种方法的特点是：不保持原数组中元素的相对顺序，但可以减少不必要的移动。

## 第一次代码

```python
class Solution(object):
    def removeElement(self, nums, val):
        """
        :type nums: List[int]
        :type val: int
        :rtype: int
        """
        i = 0
        j = len(nums) - 1

        while i <= j:
            # 找到左边第一个等于 val 的元素
            while i <= j and nums[i] != val:
                i += 1

            # 找到右边最后一个不等于 val 的元素
            while i <= j and nums[j] == val:
                j -= 1

            # 如果 i < j，把右边的有效元素移动到左边
            if i < j:
                nums[i] = nums[j]
                i += 1
                j -= 1

        return i
```

## 第一次复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

## 第二次思路：快慢指针覆盖

第二次写法更直观：

- `i` 负责遍历整个数组。
- `k` 记录下一个应该放入有效元素的位置。
- 如果 `nums[i] != val`，就把它放到 `nums[k]`，然后 `k += 1`。

这种写法会保持剩余元素的相对顺序，逻辑也更容易理解。

## 第二次代码

```python
class Solution(object):
    def removeElement(self, nums, val):
        """
        :type nums: List[int]
        :type val: int
        :rtype: int
        """
        k = 0  # 记录不等于 val 的元素个数

        for i in range(len(nums)):
            if nums[i] != val:
                nums[k] = nums[i]
                k += 1

        return k
```

## 第二次复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

## 复盘

- 这题的关键是理解“原地修改”和“返回新长度”。
- 题目不要求保留 `k` 之后的元素，所以后面的内容不用管。
- 第一种双指针写法适合不要求顺序的情况。
- 第二种快慢指针写法更稳定、更容易写对，也更适合新手复习。
- 遇到数组原地删除问题时，可以优先考虑快慢指针。
