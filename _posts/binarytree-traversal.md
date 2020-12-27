---
title: 二叉树遍历
date: 2020-09-17 19:11:48
tags: 数据结构
---
二叉树遍历方式有四种：前序遍历、中序遍历、后序遍历、层序遍历。
<!-- more -->
TreeNode定义
```java
public class TreeNode {
	public int data;
	public TreeNode leftChild;
	public TreeNode rightChild;

	public TreeNode(int data) {
		this.data = data;
	}
}
```
# 前序遍历
前序遍历（PreOrder Traversal）是先访问根节点，然后前序遍历左子树，再前序遍历右子树。
```java
/**
 * 前序遍历
 */
public void preOrderTraversal(TreeNode node) {
	if (node == null) {
		return;
	}
	System.out.println(node.data);
	preOrderTraversal(node.leftChild);
	preOrderTraversal(node.rightChild);
}
```
# 中序遍历
中序遍历（InOrder Traversal）是先中序遍历根节点的左子树，然后是访问根节点，最后遍历右子树。
```java
/**
 * 中序遍历
 */
public void inOrderTraversal(TreeNode node) {
	if (node == null) {
		return;
	}
	inOrderTraversal(node.leftChild);
	System.out.println(node.data);
	inOrderTraversal(node.rightChild);
}
```
# 后序遍历
后序遍历（PostOrder Traversal）是先后序遍历左子树，然后遍历右子树，最后访问根节点。
```java
/**
 * 后序遍历
 */
public void postOrderTraversal(TreeNode node) {
	if (node == null) {
		return;
	}
	postOrderTraversal(node.leftChild);
	postOrderTraversal(node.rightChild);
	System.out.println(node.data);
}
```
# 层序遍历
层序遍历是从根节点开始从上往下逐层遍历，在同一层，按从左到右的顺序对节点逐个访问。
层序遍历利用队列实现，首先将根节点入队，然后根节点出队，判断如果根节点有左节点或右节点，如果有，将子节点入队，如果队列不为空，再将队列中的节点出队，继续前面的步骤。
```java
/**
 * 层序遍历
 */
public void levelOrder(TreeNode root) {
	if (root == null) {
		return;
	}
	LinkedList<TreeNode> queue = new LinkedList<>();
	queue.offer(root);
	while (!queue.isEmpty()) {
		TreeNode node = queue.poll();
		System.out.println(node.data);
		if (node.leftChild != null) {
			queue.offer(node.leftChild);
		}
		if (node.rightChild != null) {
			queue.offer(node.rightChild);
		}
	}
}
```
