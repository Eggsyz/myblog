---
title: N叉树的最大深度
date:  2020-04-24T21:20:03
categories: leetcode
tags:
- 每日一题
- 树 
---

## N叉树的最大深度

给定一个 N 叉树，找到其最大深度。

最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。例如，给定一个 3叉树 :

示例：

<img src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png" alt="img" style="zoom:50%;" />

我们应返回其最大深度，3。

**说明:**

1. 树的深度不会超过 `1000`。
2. 树的节点总不会超过 `5000`。

## 题目详解

此题比较简单，采用递归方式即可。

1. 如果节点为nil，则返回0
2. 如果节点没有子节点，则返回1
3. 如果节点存在子节点，则返回1+子节点最大高度

## 代码

```go
func maxDepth(root *Node) int {
	if root == nil {
		return 0
	}
	if len(root.Children) == 0 {
		return 1
	}
	var max int
	for _, node := range root.Children {
		depth := maxDepth(node)
		if depth > max {
			max = depth
		}
	}
	return 1 + max
}
```

