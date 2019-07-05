---
title: "DFS 解题模式"
date: 2019-07-01T00:10:14+08:00
---

#### 概述

这篇文章介绍如何解决 Leetcode 常见的 DFS 问题的模式，了解这个基本模式之后，相信面对大部分 DFS 问题（如果 hard 难度的需要一些变形）都能够迎刃而解。阅读之前希望你对图的基础知识有一定的了解，例如什么是图，常见的图有那些，如何遍历图，Leetcode 上 DFS 问题常见的都是无环图，所以我们这里也只讨论无环图的解法。

#### 辨别问题
那么什么样的问题可以用 DFS 来解决呢？在 Leetcode 中，DFS 问题常见的表达形式为：

> “给定一个图（树，字符串，矩阵），找到在遍历图的过程中，符合特定条件的数值或路径。”


（这里我把返回布尔值当成返回默认值或空路径的特殊情况），上面的这个定义有点抽象，举两个例子：

- [Leetcode 113 Path Sum II](https://leetcode.com/problems/path-sum-ii/)

    > "Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum."
    
    > “给定一个有向无环图（非空二叉树），找到在遍历图的过程中，符合特定条件的数值（路径和等于 sum ）”

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
        11110
        11010
        11000
        00000

        Output: 1


#### 解题模式
很多 DFS 问题可以用 DP 来解决，通常效率也会更高。不过 DP 的状态转移方程有时候不好想，那么这时候即使头脑空白也能遵循一个固定的思路，先实现 DFS 的解法然后优化成 DP，首先我们先描述算法中要做的事情。

1. 遍历图

    如果题目要求返回的只是布尔值的话，遍历可以提早结束，不然遍历目标都是遍历整个图，若根节点不能访问所有其他节点，**那么就需要对每个节点进行 DFS 遍历**。
    
2. DFS 函数

    这个函数是一个递归函数，里面包含了 **is_valid** 以及 **match** 两个子函数。分别用作判断子节点是否合法，以及当前状态是否符合条件。
    
3. 返回结果


#### 具体实现
1. 遍历图

注意这里我把 is_valid 的状态判断放到了 DFS 的子节点遍历中，所以主函数需要处理一些边界条件

   1. 从根节点可以访问所有其他节点（例子 1）：
    
        function main_function(graph):
            # 边界条件，例如如果图是空的，或者根节点本身就符合条件
            if not graph:
                return []
            # 如果需要返回数值则创建变量（例如最大值，最小值），返回路径则创建数组：
            res = val // list[]
            element = first element in the graph
            # 对根节点进行 DFS 遍历
            dfs(element, res)
            return res

   2. 根节点不能访问其他所有节点（例子 2)：
    
        function main_function(graph):
            # 边界条件，例如如果图是空的，或者根节点本身就符合条件
            if not graph:
                return []
            res = val // list[]
            # 对图里面每个元素进行 DFS 遍历
            for element in the matrix:
                dfs(element, res)
            return res
              
   3. 如果题目要求的返回值是布尔值的话，遍历图可以提前结束：

        function main_function(graph):
            res = val // list[]
            for in the graph:
                if dfs(element, res) is True:
                    return True
            return False
              
2. DFS 递归函数

    1. 找到需要遍历的子节点

        遍历的子节点有时候不好找，对于有向图来说，树结构通常是它的子树节点，字符串根据实际情况可以是其他任意一个字符。无向图如某些矩阵则可能是上下左右节点，或者下右节点，这些看题目要求，同时因为要防止重复遍历，所以这里可以使用一个小技巧，把当前节点的值设为无效，DFS 遍历结束再还原。（下面的可选 2），函数中的的 **is_valid** 和 **match** 都是子函数
    
    2. 函数实现
    
        这里有四个重点，**1. 防止点被重复遍历**，**2. 检查节点是否合法**，**3. 检查更新后的状态是否符合要求**，**4. 更新接下来 DFS 的参数**。形式一我们把 match 函数放在子节点的遍历中，这样速度相对比较快，不过主函数需要处理边界情况。形式二则把 match 函数放在 DFS 函数的开头，虽然速度较慢，但是容易实现：
                    
            function dfs_first(element, res, current, target, path):
                # 输入参数中，
                # element 代表需要遍历的节点
                # res 代表保存结果的最终容器
                # current 代表当前状态（可选）
                # target 代表目标状态
                # path 代表遍历路径（可选）
                
                # 遍历每一个子节点
                for each child in element:
                    # 1. 修改图的节点值为非法（可选）
                    graph->val = unvalid value
                    # 2. 检查子节点是否合法，是否已经访问过，是否越界
                    if is_valid(child):
                        # 3. 检查子节点与元素组成的新状态是否符合条件
                        if match(current, child, target):
                            # 更新最终结果
                            res += new_res
                        else:
                            # 4. 遍历所有合法子节点，更新状态
                            dfs_first(child, res, current+child.val, target, path+child)
                    # 恢复图的节点值（可选）
                    graph->val = valid value

            function dfs_second(element, res, current, target, path):
                # 输入参数中，
                # element 代表需要遍历的节点
                # res 代表保存结果的最终容器
                # current 代表当前状态（可选）
                # target 代表目标状态
                # path 代表遍历路径（可选）

                # 如果当前节点符合条件
                if match(current, element, target):
                    # 更新最终结果
                    res += new_res
                    return
                # 遍历每一个子节点
                for each child in element:
                    # 1. 修改图的节点值为非法（可选）
                    graph->val = unvalid value
                    # 2. 检查子节点是否合法，是否已经访问过，是否越界
                    if is_valid(child):
                        # 4. 遍历所有合法子节点，更新状态
                        dfs_first(child, res, current+child.val, target, path+child)
                    # 恢复图的节点值（可选）
                    graph->val = valid value
                    
            function is_valid(child):
                # 如果 child 合法则返回真，否则返回假
                
            function match(current, child, target):
                # 如果当前 child 与 current 的组合满足题目与 target 的要求，则返回真
                
                       
#### 原题分析

按照此解题模式，第一个例子可以这样实现：（形式一，Python 代码）

    class Solution:
        def pathSum(self, root, sum):
            # 边界情况
            if not root:
                return []
            # 边界情况2，因为我们是在遍历中验证是否符合条件，所以这里要处理另外一个边界情况
            if root.val == sum and not root.left and not root.right:
                return [[root.val]]
            # 因为根节点可以访问所有子节点，所以只需要遍历根节点
            return self.dfs(root, [], root.val, sum, [root.val])

        def dfs(self, node, res, current, target, path):
            # 遍历每一个子节点
            for n in [node.left, node.right]:
                # 检查子节点是否合法，是否已经访问过，是否越界
                if self.is_valid(n):
                    # 检查子节点与元素组成的新状态是否符合条件
                    if self.match(current, n, target):
                        # 更新最终结果
                        res.append(path+[n.val])
                    else:
                        # 遍历所有合法子节点，更新状态
                        self.dfs(n, res, current+n.val, target, path+[n.val])
            return res

        def is_valid(self, node):
            # 子节点在不同的 path 中可以重复遍历，所以只要存在则返回真
            if node:
                return True
            return False
            
        def match(self, current, child, target):
            # 题目要求 child 必须为叶子节点，并且与之前值的和等于 target
            if not child.left and not child.right and current + child.val == target:
                return True
            return False
            
         
第二题也是类似的方法，形式一以及形式二都能实现，这里我选择了另一种特殊的方式，把 match 函数提前。

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
                    # 这里也可以使用 match 函数的形式，即当周围节点都不为 1 的时候， count 增加 1
                    grid[k][v] = '#'
                    self.dfs(grid, k, v)
    
        def is_valid(self, i, j, grid):
            # 如果 i，j 合法的话
            if i < 0 or j < 0 or i >= len(grid) or j >= len(grid[0]) or grid[i][j] != '1':
                return False
            return True

#### 总结
解决 DFS 问题最重要的是四点，**1. 防止点被重复遍历**，**2. 检查节点是否合法**，**3. 检查更新后的状态是否符合要求**，**4. 更新接下来 DFS 的参数**，只要按照这个思路，形式怎么写都没关系。
