title: Leetcode-01-删除排序数组中的重复项
date: 2022-02-12 22:23:13
toc: true
category:
- leetcode
tags:
- leetcode
- 删除
- 排序
- 数组
- 重复项
---

采用双指针解题（分为 slow 慢指针和 fast 快指针），快指针表示遍历数组到达的下标位置，慢指针表示下一个不同元素要填入的下标位置，初始时两个指针都指向下标 1。

**题干得知传入的数组 nums 是升序的。**

假设 nums 长度为 n，将快指针 fast 快指针由 1 遍历到 n-1，对于每个位置，

如果 nums[fast] != nums[fast-1]，说明 nums[fast] 和上一元素不同，

因此将 nums[fast] 移动到 nums[slow] 的位置，然后将 slow 的值加 1，即指向下一位置。

遍历结束后，从 nums[0] 到 nums[slow-1] 的元素都不相同且包含原数组中每个不同的元素，因此新数组的长度为 slow，返回 slow 即可。

```go
func removeDuplicates(nums []int) int {
	if 0 == len(nums) {
		return 0
	}

	slow := 1
	for fast := 1; fast < len(nums); fast++ {
		if nums[fast] != nums[fast-1] {
			nums[slow] = nums[fast]
			slow++
		}
	}
	return slow
}
```

![image.png](https://b3logfile.com/file/2022/02/image-05a9a15f.png)

## 参考资料

- 原题地址：[26. 删除有序数组中的重复项 - 力扣（LeetCode）](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)
