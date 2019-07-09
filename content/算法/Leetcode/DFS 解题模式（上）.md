---
title: "DFS 解题模式（上）"
date: 2019-07-01T00:10:14+08:00
weight: 1
---

#### 概述

这篇文章介绍 **Leetcode 常见 DFS 问题的解题模式**，希望你了解这些模式之后，对大部分 DFS 问题（hard 难度的需要一些变形）都能够迎刃而解。由于 Leetcode 上 DFS 问题中常见的都是无环图，所以我们这里也只讨论无环图的解题模式。阅读本文之前你需要对**图的基础知识**有一定的了解，包括什么是图？常见的图的类型有那些？（有向无环图，有向有环图），如何遍历图？（前序遍历以及后序遍历）。

#### 辨别问题
那么什么样的问题可以用 DFS 来解决呢？，DFS 问题常见的表达形式为：

> “给定一个图（树，字符串，矩阵），找到在遍历图的过程中，符合特定条件的数值或路径。”


上面的这个定义有点抽象，举两个例子：

- [Leetcode 113 Path Sum II](https://leetcode.com/problems/path-sum-ii/)

    > "Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum."
    
    > “给定一个有向无环图（二叉树），找到在遍历图的过程中，符合特定条件的数值（路径和等于 sum ）”

        Given the below binary tree and sum = 22, 

        input:
        
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1

        output:

        [
            [5,4,11,2],
            [5,8,4,5]
        ]

    
- [Leetcode 200 Number of Islands](https://leetcode.com/problems/number-of-islands/)

    > Given a 2d grid map of '1's (land) and '0's (water), count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.
    
    > “给定一个无向无环图（矩阵），找到在遍历图的过程中，符合特定条件的数值（岛的数量）”
        
        Input:
        
        11010
        11000
        11000
        00000

        Output: 2


#### 解题模式
虽然许多 DFS 问题都可以用 DP 来解决，通常效率也更高。不过 DP 的状态转移方程往往不容易想到，所以在面试的时候，**先快速按照解题模式实现 DFS 的解法然后再优化成 DP 也是一个不错的方法**。解决 DFS 问题最重要的是四点，**1. 防止节点被重复遍历**，**2. 遍历前，检查节点是否合法**，**3. 检查遍历后的状态是否符合要求**，**4. 更新接下来 DFS遍历 的参数**，解题模式主要包括三个部分：

1. 主函数

    主函数要做的有两件事：**第一，处理边界情况，例如图为空，第二，遍历整个图**，如果题目要求返回的是布尔值（图中是否存在符合此条件的路径），那么遍历在找到符合条件的路径时就可以结束，**除了这种情况，都需要遍历图中所有可达节点。**若不能通过初始节点访问所有可达节点，**那么在主函数就需要对每个节点进行 DFS 遍历**。上面的例子 **Leetcode 200 Number of Islands**，遍历完初始节点后，因为其他节点都是未知状态，所以需要继续遍历。

        # 左侧为遍历初始节点后递归遍历过的点，右侧为仍需遍历的点
        110       0
        110       0
        110       0
        000       0

    题目                     | 要求                  | 初始节点可以遍历到整个图 |
    ------------------------ | --------------------- | ------------------------ |
    Leetcode 113 Path Sum II | 返回所有符合条件的路径| 是                       |
    Leetcode 200 Number of Islands | 返回符合条件的路径的数量 | 否              |
    
2. DFS 递归函数

    这个函数是一个递归函数，里面调用了辅助函数，DFS 函数只需要对当前节点的子节点（如果是无向图则临近节点）进行遍历即可，其他功能通过辅助函数实现。

3. 辅助函数

    里面包含了 **is_valid** 以及 **match** 两个子函数。分别用作判断子节点是否合法，以及当前状态是否符合条件。

#### 具体实现

1. 主函数伪代码

   1. 从初始节点可以访问所有可达节点（Leetcode 113 Path Sum II）：
    
            function main_function(graph):
                # 边界情况，例如如果图是空的，或者初始节点本身就符合条件
                if graph is empty
                    return empty
                # 如果需要返回数值则创建变量（例如最大值，最小值），返回路径则创建数组：
                res -> a variable or array
                # 只需要遍历初始节点
                element -> first element in the graph
                # 对初始节点进行 DFS 遍历
                dfs(element, res)
                # 返回结果
                return res

   2. 初始节点不能访问其他可达节点（Leetcode 200 Number of Islands)：
    
            function main_function(graph):
                # 边界条件，例如如果图是空的，或者初始节点本身就符合条件
                if graph is empty
                    return empty
                res -> a variable or array
                # 对图里面每个元素进行 DFS 遍历
                for every element in the matrix
                    dfs(element, res)
                # 返回结果
                return res
              
   3. 如果题目要求的返回值是布尔值的话，遍历图可以提前结束：

            function main_function(graph):
                res -> a variable or array
                for every element in the matrix
                    if dfs(element, res) -> True
                        return True
                return False
              
2. DFS 递归函数

    1. 写代码前，先需要遍历节点的层级关系

        对于有向图来说，某一节点需要进行遍历它的子节点（如果是二叉树则是左右子节点，如果是字符则可能是临近字符）。无向图则遍历临近节点（如矩阵，可能遍历上下左右或者下右节点）。我建议大家在面试实现的时候可以绘制出遍历图，这样写代码的时候会比较有把握。以下是 Leetcode 200 Number of Islands 的遍历流程图：


        ![dfs_one](https://raw.githubusercontent.com/EngineGirl/enginegirl.github.io/markdown/images/al/dfs_one.png) 

        上图中，要防止无向图中（1，1）被重复遍历，这里可以使用一个小技巧，先把当前节点的值设为无效值（这样在递归遍历中不会原路返回），DFS 遍历结束再还原。
    
    2. 函数实现
    
        这里有四个重点，**1. 防止点被重复遍历**，**2. 检查节点是否合法**，**3. 检查更新后的状态是否符合要求**，**4. 更新接下来 DFS 遍历的参数**。以下实现了两种形式，形式一把 match 函数放在子节点的遍历中，这样速度相对比较快，不过主函数需要处理边界情况。形式二则把 match 函数放在 DFS 函数的开头，虽然速度较慢，但是容易实现，以下是伪代码：
                    
            function dfs_first(element, res, current, target, path):
                # 输入参数中，
                # element 代表需要遍历的节点
                # res 代表保存结果的最终容器
                # current 代表当前状态
                # target 代表目标状态
                # path 代表遍历路径（可选）
                
                # 遍历每一个子节点
                for each child in element:
                    # 1. 大部分问题中，在同一节点的遍历中都不能重复使用同一节点，所以在无向图中，需要修改图的节点值为非法，
                    graph->val = unvalid value
                    # 2. 检查子节点是否合法，包括是否已经遍历过，是否越界
                    if is_valid(child):
                        # 3. 检查子节点与元素组成的新状态是否符合条件
                        if match(current, child, target):
                            # 更新最终结果
                            res += new_res
                        else:
                            # 4. 遍历所有合法子节点，更新当前状态以及路径
                            dfs_first(child, res, current+child.val, target, path+child)
                    # 恢复图的节点值
                    graph->val = valid value

            function dfs_second(element, res, current, target, path):
                # 输入参数中，
                # element 代表需要遍历的节点
                # res 代表保存结果的最终容器
                # current 代表当前状态
                # target 代表目标状态
                # path 代表遍历路径（可选）

                # 1. 先验证当前状态是否符合条件
                if match(current, element, target):
                    # 更新最终结果
                    res += new_res
                    return
                # 2. 遍历每一个子节点
                for each child in element:
                    graph->val = unvalid value
                    if is_valid(child):
                        dfs_first(child, res, current+child.val, target, path+child)
                    graph->val = valid value
                    
            function is_valid(child):
                # 如果 child 合法则返回真，否则返回假
                # 例如 child 在矩阵范围中
                if 0 <= child.i < length of matrix and 0 <= child.y < length of first row of matrix
                    return True
                return False
                
            function match(current, child, target):
                # 如果当前 child 与 current 的组合满足题目与 target 的要求，则返回真
                if current + child.val equal to target
                    return True
                return False
                       
#### 原题分析

我们试试在例子中运用此解题模式，第一题：（**以下为 Python 代码，形式二只需要把 match 函数移在 DFS 函数开头即可**）

    class Solution:
        # 形式一
        def pathSum(self, root, sum):
            # 边界情况
            if not root:
                return []
            # 边界情况2，因为我们是在遍历中验证是否符合条件，所以要检查初始条件是否已经符合要求
            if root.val == sum and not root.left and not root.right:
                return [[root.val]]
            # 因为初始节点可以访问所有可达节点，所以只需要遍历初始节点
            return self.dfs(root, [], root.val, sum, [root.val])

        def dfs_first(self, node, res, current, target, path):
            # 1. 遍历每一个子节点
            for n in [node.left, node.right]:
                # 2. 检查子节点是否合法，是否已经访问过，是否越界
                if self.is_valid(n):
                    # 3. 检查子节点与元素组成的新状态是否符合条件
                    if self.match(current, n, target):
                        # 4. 更新最终结果
                        res.append(path+[n.val])
                    else:
                        # 5. 遍历所有合法子节点，更新状态
                        self.dfs_first(n, res, current+n.val, target, path+[n.val])
            return res

        def is_valid(self, node):
            # 只要存在则为真
            if node:
                return True
            return False
            
        def match(self, current, child, target):
            # 题目要求 child 必须为叶子节点，并且与之前值的和等于 target
            if not child.left and not child.right and current + child.val == target:
                return True
            return False
 
    114 / 114 test cases passed.
    Status: Accepted
    Runtime: 60 ms
    Memory Usage: 19.1 MB
         
第二题也是类似的方法，形式一以及形式二都能实现，因为形式一以及形式二区别不大，所以这里我选择了另一种特殊的方式，把 match 函数提前。

    class Solution:
        def numIslands(self, grid):
            # 边界情况
            if not grid:
                return 0
            count = 0
            # 因为岛之间可能并不相连，所以需要遍历整个图
            for i in range(len(grid)):
                for j in range(len(grid[0])):
                    # match 函数
                    if grid[i][j] == '1':
                        grid[i][j] = '#'
                        self.dfs(grid, i, j)
                        count += 1
            return count

        def dfs(self, grid, i, j):
            # 遍历可能的子节点
            for k, v in [(i+1, j), (i-1, j), (i, j+1), (i, j-1)]:
                if self.is_valid(k, v, grid):
                    # 我把 match 函数抽离出来放在主函数中了，
                    grid[k][v] = '#'
                    self.dfs(grid, k, v)
    
        def is_valid(self, i, j, grid):
            # 如果 i，j 合法的话
            if i < 0 or j < 0 or i >= len(grid) or j >= len(grid[0]) or grid[i][j] != '1':
                return False
            return True

    47 / 47 test cases passed.
    Status: Accepted
    Runtime: 84 ms
    Memory Usage: 14.1 MB

#### 总结
解决 DFS 问题最重要的是四点，**1. 防止节点被重复遍历**，**2. 遍历前，检查节点是否合法**，**3. 检查遍历后的状态是否符合要求**，**4. 更新接下来 DFS遍历 的参数**，只要按照这个思路，形式怎么写都没关系。
