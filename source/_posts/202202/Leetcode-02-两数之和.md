title: Leetcode-02-两数之和
date: 2022-02-12 22:23:00
toc: true
category:
- leetcode
tags:
- leetcode
---

## 暴力枚举

枚举数组中的每一个数 x，寻找数组中是否存在 target-x。


<!-- more -->


```go
func twoSum(nums []int, target int) []int {
	for i := 0; i < len(nums); i++ {
		for j := i + 1; j < len(nums); j++ {
			if nums[j] == target-nums[i] {
				return []int{i, j}
			}
		}
	}
	return nil
}
```

![image.png](https://b3logfile.com/file/2022/02/image-11576454.png)

**最坏情况下数组中任意两个数都要被匹配一次。**

## 哈希表

暴力枚举方式在于寻找 target - x 的时间复杂度过高，因此可以创建哈希表解决该问题。

```go
func twoSum(nums []int, target int) []int {
	result := make(map[int]int, 0)
	for index, value := range nums {
		if val, ok := result[target-value]; ok {
			return []int{index, val}
		}
		result[value] = index
	}
	return nil
}
```

![image.png](https://b3logfile.com/file/2022/02/image-51e4a586.png)

## 参考资料

- 原题地址：[1. 两数之和 - 力扣（LeetCode）](https://leetcode-cn.com/problems/two-sum/)
