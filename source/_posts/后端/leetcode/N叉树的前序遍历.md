---
title: N叉树的前序遍历
date:  2020-04-24T20:20:03
categories: leetcode
tags:
- 每日一题
- 树 
---

## N叉树的前序遍历

给定一个 N 叉树，返回其节点值的*前序遍历*。

例如，给定一个 3叉树 :

示例：

<img src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png" alt="img" style="zoom:50%;" />

返回其前序遍历: `[1,3,5,6,2,4]`。

说明: 递归法很简单，你可以使用迭代法完成此题吗?

## 题目详解

此题比较简单，采用递归方式即可。这是前序遍历N叉树，先添加当前节点，再遍历节点的所有子节点。例如上述3叉树。

## 代码

```go
func preorder(root *Node) []int {
	var res []int
	if root == nil {
		return res
	}
	help(root, &res)
	return res
}

func help(root *Node, res *[]int) {
	if root == nil {
		return
	}
	*res = append(*res, root.Val)
	for _, node := range root.Children {
		help(node, res)
	}
}
```
