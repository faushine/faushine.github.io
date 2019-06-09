---
layout: post
title: "Binary Tree"
date:   2019-06-08
author: Faushine
tags: 
- Leetcode
---

## Preorder 

### Divide Conquer

```java
public class Solution {
    public ArrayList<Integer> preorderTraversal(TreeNode root) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        // null or leaf
        if (root == null) {
            return result;
        }

        // Divide
        ArrayList<Integer> left = preorderTraversal(root.left);
        ArrayList<Integer> right = preorderTraversal(root.right);

        // Conquer
        result.add(root.val);
        result.addAll(left);
        result.addAll(right);
        return result;
    }
}
```

### Non-Recursion

```java
public class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        Stack<TreeNode> stack = new Stack<TreeNode>();
        List<Integer> preorder = new ArrayList<Integer>();
        
        if (root == null) {
            return preorder;
        }
        
        stack.push(root);
        while (!stack.empty()) {
            TreeNode node = stack.pop();
            preorder.add(node.val);
            if (node.right != null) {
                stack.push(node.right);
            }
            if (node.left != null) {
                stack.push(node.left);
            }
        }
        
        return preorder;
    }
}
```

## Inorder

### Non-Recursion

```java
class Solution {
        public List<Integer> inorderTraversal(TreeNode root) {
            List<Integer> list = new ArrayList<>();
            Stack<TreeNode> stack = new Stack<>();
            TreeNode cur = root;
            while (cur != null || !stack.isEmpty()) {
                if (cur != null) {
                    stack.push(cur);
                    cur = cur.left;
                } else {
                    cur = stack.pop();
                    list.add(cur.val);
                    cur = cur.right;
                }
            }
            return list;
        }
}
```

## Divide Conquer

### Maximum Depth of Binary Tree

```java
public class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        return Math.max(left, right) + 1;
    }
}
```

###  Minimum Subtree

```java
public class Solution {
    private TreeNode subtree = null;
    private int subtreeSum = Integer.MAX_VALUE;
    /**
     * @param root the root of binary tree
     * @return the root of the minimum subtree
     */
    public TreeNode findSubtree(TreeNode root) {
        helper(root);
        return subtree;
    }
    
    private int helper(TreeNode root) {
        if (root == null) {
            return 0;
        }
        
        int sum = helper(root.left) + helper(root.right) + root.val;
        if (sum <= subtreeSum) {
            subtreeSum = sum;
            subtree = root;
        }
        return sum;
    }
}
```

###  Balanced Binary Tree

```java
public class Solution {
    /**
     * @param root: The root of binary tree.
     * @return: True if this Binary tree is Balanced, or false.
     */
    public boolean isBalanced(TreeNode root) {
        // write your code here
       return helper(root)!=-1;
    }
    private int helper(TreeNode root){
        if(root == null){
            return 0;
        }
        int leftDepth = helper(root.left);
        int rightDepth = helper(root.right);
        if(leftDepth==-1 || rightDepth==-1 || Math.abs(rightDepth-leftDepth)>1){
            return -1;
        }
        return Math.max(leftDepth,rightDepth)+1;
    }
    
}
```
###  Subtree with Maximum Average

```java
class ResultType{
    int sum;
    int size;
    public ResultType(int sum, int size){
        this.sum = sum;
        this.size = size;
    }
}
public class Solution {
    /**
     * @param root: the root of binary tree
     * @return: the root of the maximum average of subtree
     */
    private int maxSum, minSize;
    private TreeNode node=null;
    public TreeNode findSubtree2(TreeNode root) {
        // write your code here
        helper(root);
        return node;
    }
    private ResultType helper(TreeNode root){
        if(root == null){
            return new ResultType(0,0);
        }
        ResultType leftR = helper(root.left);
        ResultType rightR = helper(root.right);
        int sum = leftR.sum+rightR.sum+root.val;
        int size = leftR.size+rightR.size+1;
        if(sum*minSize>maxSum*size || node==null){
            maxSum = sum;
            minSize = size;
            node = root;
        }
        return new ResultType(sum,size);
    }
}
```

###  Lowest Common Ancestor

```java
public class Solution {
    /*
     * @param root: The root of the binary search tree.
     * @param A: A TreeNode in a Binary.
     * @param B: A TreeNode in a Binary.
     * @return: Return the least common ancestor(LCA) of the two nodes.
     */
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode A, TreeNode B) {
        // write your code here
       if(root==null||root==A||root==B){
           return root;
       }
       TreeNode leftNode = lowestCommonAncestor(root.left,A,B);
       TreeNode rightNode = lowestCommonAncestor(root.right,A,B);
       
       if(leftNode!=null&&rightNode!=null){
           return root;
       }
       if(leftNode!=null){
           return leftNode;
       }
       if(rightNode!=null){
           return rightNode;
       }
       return null;
    }
    
}
```

### Validate Binary Search Tree

```java
class ResultType{
    boolean isValidBST;
    int maxValue,minValue;
    public ResultType(boolean isValidBST, int maxValue, int minValue){
        this.isValidBST = isValidBST;
        this.maxValue = maxValue;
        this.minValue = minValue;
    }
}
public class Solution {
    /**
     * @param root: The root of binary tree.
     * @return: True if the binary tree is BST, or false
     */
    public boolean isValidBST(TreeNode root) {
        // write your code here
        ResultType result = helper(root);
        return result.isValidBST;
    }
    private ResultType helper(TreeNode root){
        if (root==null){
            return new ResultType(true,Integer.MIN_VALUE,Integer.MAX_VALUE);
        }
        ResultType leftR = helper(root.left);
        ResultType rightR = helper(root.right);
        if(root.left ! = null && leftR.maxValue >= root.val || root.right!=null && rightR.minValue <= root.val){
            return new ResultType(false,0,0);
        }
        if(!leftR.isValidBST||!rightR.isValidBST){
            return new ResultType(false,0,0);
        }
        return new ResultType(true,Math.max(rightR.maxValue,root.val),Math.min(leftR.minValue,root.val));
    }
}
```

