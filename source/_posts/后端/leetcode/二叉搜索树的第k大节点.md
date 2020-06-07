---
title: 二叉搜索树的第k大节点
date:  2020-05-08T20:20:03
categories: leetcode
tags:
- 每日一题
- 树
---



## 二叉搜索树的第k大节点

给定一棵二叉搜索树，请找出其中第k大的节点。

 

示例 1:

```
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4
```

示例 2:

```
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4
```


限制：

1 ≤ k ≤ 二叉搜索树元素个数

## 题目详解

本题可以采用中序遍历的方式。中序遍历后，二叉搜索树结果为从小到大，因此采用逆序中序遍历，当找到第k个值时，返回结果。

## 代码

+ 中序遍历递归+提前退出解法

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func kthLargest(root *TreeNode, k int) int {
	var res int
	var dfs func(n *TreeNode)
	dfs = func(node *TreeNode) {
		if node == nil {
			return
		}
		dfs(node.Right)
    // 当k==0时直接返回
    if k==0{
      return
    }
    // 找第k个值
		k--
		if k == 0 {
			res = node.Val
			return
		}
		dfs(node.Left)
	}
	dfs(root)
	return res
}
```

