# 输入一个 二叉树，和一个 整数， 要求输出 path的和等于 这个整数的 所有path. 1point3acres
eg：   5
      3       9
   1   6  7    4    另外一个整数是 9
. From 1point 3acres bbs
输出为  5 3 1， 3 6， 9

# Binary Search Tree Iterator
Implement an iterator over a binary search tree (BST). Your iterator will be initialized with the root node of a BST.

此题的精华在于建立一个path,存储的是一路走下来经历的节点,然后当找下一个的时候,要使用pop,并判断是不是在当前的左节点

# 301 Remove Invalid Parentheses
Remove the minimum number of invalid parentheses in order to make the input string valid. Return all possible results.

Note: The input string may contain letters other than the parentheses ( and ).

使用BFS,每个字母都尝试减去了一遍.

# 286 Walls and Gates

You are given a m x n 2D grid initialized with these three possible values.

-1 - A wall or an obstacle.
0 - A gate.
INF - Infinity means an empty room. We use the value 231 - 1 = 2147483647 to represent INF as you may assume that the distance to a gate is less than 2147483647.
Fill each empty room with the distance to its nearest gate. If it is impossible to reach a gate, it should be filled with INF.

这是一道BFS题吧

# 227 Basic Calculator II

+ - * /

#
Input：一个list，里面是pair<pickup_city, dropoff_city>, 先pick后drop
Output: 输出所有的可能顺序

Ex. <a_pick, b_drop>, <b_pick, a_drop>
output:
a_pick, b_drop, b_pick, a_drop
a_pick, b_pick, a_drop, b_drop
b_pick, a_drop, a_pick, b_drop
b_pick, a_pick, a_drop, b_drop

# 128 Longest Consecutive Sequence
Given an unsorted array of integers, find the length of the longest consecutive elements sequence.

Your algorithm should run in O(n) complexity.

# weight base load balance

# 找出一个list里面第二小的数。

#  merge一个list里的interval。 如input: [[1,2],[2,4],[1,3],[5,6]] 返回[[1,4],[5,6]]。 我用o(nlog(n))的算法做的

# 经典电梯设计问题， follow up 如何在多线程环境 实现

# 给一组有重复的单词，返回按照frequency高低排序的单词组。
先用O(nlgn)方法写出来了，小哥说能不能优化。然后写了个O(n)的。

# 有两个人，判断这两人是否属于同一个social circle。

# 76 Minimum Window Substring, 简化版: target 是一个set没有重复char

# 221. Maximal Square

Given a 2D binary matrix filled with 0's and 1's, find the largest square containing only 1's and return its area.

# 428 Serialize and Deserialize N-ary Tree

使用 # 来表示结束
使用BFS来重建

# 772 Basic Calculator III

# Given k sorted lists, return largest, smallest, second largest, second smallest, ...　

# 332 Reconstruct Itinerary

Given a list of airline tickets represented by pairs of departure and arrival airports [from, to], reconstruct the itinerary in order. All of the tickets belong to a man who departs from JFK. Thus, the itinerary must begin with JFK.

这题有点难度.因为要求的是必须要返回lexial smaller的那组数据.

```
class Solution(object):
    import collections
    def findItinerary(self, tickets):
        targets = collections.defaultdict(list)
        for a, b in sorted(tickets):
            targets[a] += b,
        route = []
        def visit(airport):
            while targets[airport]:
                next_station = targets[airport][0]
                targets[airport] = targets[airport][1:]
                visit(next_station)
            route.append(airport)
        visit('JFK')
        return route[::-1]
```

### 2D array, black and white blocks. Only white blocks 可以走。
给两个点，求从一个点到另一个点的path。followup：求最短的那条path

### 利特口的 岛的个数 + test cases+ followup 岛形状的个数所有代码都要run结果

###
