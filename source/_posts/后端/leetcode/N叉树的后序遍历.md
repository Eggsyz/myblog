---
title: N叉树的后序遍历
date:  2020-04-21T22:10:03
categories: leetcode
tags:
- 每日一题
- 树
- 递归
---

给定一个 N 叉树，返回其节点值的后序遍历。

例如，给定一个 3叉树 :

 <img src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png" alt="img" style="zoom:50%;" />

返回其后序遍历: [5,6,3,2,4,1].

说明: 递归法很简单，你可以使用迭代法完成此题吗?

## 题目详解

### 递归求解

此题比较简单，采用递归方式即可。这是后序遍历N叉树，则需要遍历节点的所有子节点，然后在添加当前节点。例如上述3叉树。首先遍历根节点的每个子节点：

1. 如果节点存在叶子节点，则继续遍历其叶子节点直到节点没有叶子节点，即添加当前节点
2. 如果没有叶子节点，则添加当前节点

### 迭代求解



## 代码

### 递归代码

```go
func postorder(root *Node) []int {
	var res []int
	help(root, &res)
	return res
}

func help(root *Node, res *[]int) {
	if root == nil {
		return
	}
	if len(root.Children) == 0 {
		*res = append(*res, root.Val)
		return
	}
	for _, node := range root.Children {
		help(node, res)
	}
	*res = append(*res, root.Val)
}
```

### 迭代代码