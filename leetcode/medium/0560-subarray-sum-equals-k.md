# 560. 和为 K 的子数组

题目链接：https://leetcode.cn/problems/subarray-sum-equals-k/

难度：Medium

标签：数组、前缀和、哈希表、分治

## 题目内容

给定一个整数数组 `nums` 和一个整数 `k`，统计并返回该数组中和为 `k` 的连续子数组个数。

注意：数组中可能有正数、负数和 `0`。

## 我的第一反应

这题和第 209 题“长度最小的子数组”很像，都是连续子数组的和。

但这题有一个关键变化：

```text
nums 里可能有负数。
```

所以普通滑动窗口不成立。

滑动窗口依赖的是：

```text
右边界扩大，sum 只会变大。
左边界收缩，sum 只会变小。
```

但如果数组里有负数：

```text
加入一个数，sum 可能变小。
移出一个数，sum 可能变大。
```

窗口没有单调性，就不能放心地用滑动窗口。

最直接的暴力想法是枚举所有子数组：

```text
枚举左端点
枚举右端点
判断子数组和是不是 k
```

这样时间复杂度是 `O(n^2)`。

## 分治思路：来之不易的一步

我想到可以用分治：

```text
整个数组的答案 =
左半边内部的答案
+ 右半边内部的答案
+ 跨过中点的答案
```

这个想法很重要，也很来之不易。

它说明我不是只在背模板，而是在主动思考：

```text
能不能把一个大区间拆成两个小区间？
能不能避免枚举所有子数组？
能不能只把“跨中心”的部分单独加速？
```

这是很多区间统计题都会用到的思维方式。虽然这题最优解还有更直接的前缀和哈希，但分治思路本身很值得保留。

## 分治的三部分

对于区间 `[left, right]`，取中点 `mid`：

```text
[left ... mid] [mid + 1 ... right]
```

所有和为 `k` 的连续子数组只可能有三类：

```text
1. 完全在左半边
2. 完全在右半边
3. 跨过 mid 和 mid + 1
```

所以递归处理：

```python
left_count = divide(left, mid)
right_count = divide(mid + 1, right)
cross_count = count_cross(left, mid, right)
```

最后：

```python
return left_count + right_count + cross_count
```

## cross_count：像两数之和的快速版

跨中心的子数组一定长这样：

```text
左半边某个位置 ... mid, mid + 1 ... 右半边某个位置
```

也就是：

```text
左边一段 + 右边一段
```

如果左边这段的和是 `left_sum`，右边这段的和是 `right_sum`，那么需要：

```text
left_sum + right_sum = k
```

这就非常像两数之和：

```text
两数之和：
a + b = target
已知 b，查 target - b

cross_count：
left_sum + right_sum = k
已知 right_sum，查 k - right_sum
```

区别是：

```text
两数之和是 数字 + 数字。
cross_count 是 左半段子数组和 + 右半段子数组和。
```

所以可以先把左边所有可能的 `left_sum` 存进字典，再枚举右边的 `right_sum`，查有没有 `k - right_sum`。

## 分治代码

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        def divide(left, right):
            if left > right:
                return 0

            if left == right:
                if nums[left] == k:
                    return 1
                return 0

            mid = (left + right) // 2

            left_count = divide(left, mid)
            right_count = divide(mid + 1, right)
            cross_count = count_cross(left, mid, right)

            return left_count + right_count + cross_count

        def count_cross(left, mid, right):
            count = 0

            # 统计所有“以 mid 结尾”的左半边子数组和
            left_sums = {}
            total = 0

            for i in range(mid, left - 1, -1):
                total += nums[i]
                left_sums[total] = left_sums.get(total, 0) + 1

            # 枚举所有“以 mid + 1 开头”的右半边子数组和
            total = 0

            for j in range(mid + 1, right + 1):
                total += nums[j]

                need = k - total

                if need in left_sums:
                    count += left_sums[need]

            return count

        return divide(0, len(nums) - 1)
```

## 分治复杂度

- 每一层递归里，所有 `count_cross` 加起来是 `O(n)`。
- 递归大约有 `O(log n)` 层。

所以：

```text
时间复杂度：O(n log n)
空间复杂度：O(n)
```

空间复杂度主要来自递归栈和每层 `count_cross` 中使用的字典。

## 最优解：前缀和 + 哈希表

这题还有更直接的 `O(n)` 解法。

连续子数组和可以用前缀和表示：

```text
nums[i ... j] 的和 = prefix[j] - prefix[i - 1]
```

如果子数组和等于 `k`：

```text
prefix[j] - prefix[i - 1] = k
```

移项：

```text
prefix[i - 1] = prefix[j] - k
```

意思是：

```text
当我走到当前位置 j，当前前缀和是 prefix[j]。
我只需要知道：之前有没有出现过 prefix[j] - k。
```

这也是两数之和思想：

```text
已知当前 prefix[j]，
去字典里找需要的旧前缀和 prefix[j] - k。
```

## 最优代码

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        count = 0
        prefix = 0
        seen = {0: 1}

        for num in nums:
            prefix += num

            need = prefix - k

            if need in seen:
                count += seen[need]

            seen[prefix] = seen.get(prefix, 0) + 1

        return count
```

## 为什么 `seen = {0: 1}`

`0` 表示一个“空前缀和”。

它用来处理从下标 `0` 开始的子数组。

比如：

```text
nums = [3, 1, 2]
k = 3
```

走到第一个元素时：

```text
prefix = 3
need = prefix - k = 0
```

如果字典里提前有一个 `0`，就能找到：

```text
nums[0 ... 0] = 3
```

所以 `seen = {0: 1}` 是必要的。

## 最优解复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## 复盘

这题最重要的收获不是只记住最优代码。

更重要的是，我先想到了：

```text
有负数，所以不能滑动窗口。
暴力是 O(n^2)。
能不能用分治拆成左边、右边、跨中心？
```

这个分治思路来之不易，值得记录下来。

它说明我已经开始从“套模板”进入“自己设计减少复杂度的方法”。

然后在 `cross_count` 里，又发现它其实很像两数之和：

```text
left_sum + right_sum = k
```

再往前走一步，就能得到最优解：

```text
prefix[j] - prefix[i] = k
```

所以这题可以这样记：

```text
分治 cross_count 是“两数之和”的区间和版本。
前缀和哈希是“两数之和”的前缀和版本。
```

## 记忆口诀

```text
连续子数组和等于 k，
有负数不能滑窗口，
前缀和差值查哈希。
```
