---
layout: post
title: 设计新的Data Structure
date: 2016-07-01
tag: Leetcode
---
##### 1 Stack Min
> 设计一个stack,实现O(1)时间的push, pop 和 min.

解答: 普通的stack就已经能够实现O(1)时间的push和pop了,但是min不行.那么怎样找min呢?而且pop和push的时候能更新min.如果只记录一个global最小的值的话,它被pop就没法O(1)找到下一个min了.

其实有现成的解决方案,就是递增递减stack.我们把push进主stack的值再push进辅助stack.这个stack中的值是递减的.这样保证它的top是最小值,而且即使被pop了的话下一个还是最小值.

##### 2 Queue via Stack
> 用两个stack实现一个queue.

解答: 建立两个stack,一个叫做input,只管push操作;一个叫做output,只管pop操作.如果output为空,而这时产生了pop操作,就把input中的所有数字都push到output中就好了.


##### 3 Random Node:
> 建立一个binary tree,实现insert, find,和delete功能,而且它还有一个操作是getRandomNode(),能够以相同的概率随机返回tree中的一个node.

解答:这明显是要建立一个binary search tree. getRandomNode()可以现将所有node traverse一遍再来选,但是这样很慢.我们还可以将每个node建立成这样的模式[val, left, right, left_count, right_count],然后根据left_count,right_count来随机选择是往左还是往右走.

##### 4 实现LRU（Least Recently Used)
> 当内存满时，删除的是最早使用的unit

建立一个双向链表，并记录链表长度，以及使用hashmap记录值到node的关系。当链表达到最大长度时，如果再来一个新的node，则将其放入链表最前的位置，记录到hashmap中，并从链表中pop出链表最后为止的node；如果不是新的node，则找到值所对应的node，讲其移到第一位即可。

###### 5 Serialize and Deserialize N-ary Tree
> 如何序列化和反序列化一个N-ary tree ?

使用#来标注最后的child已经被处理了.和trie tree的思路是一样的.而且使用的是preorder的方式来serialize.

而deserialize 就用的是deque了当遇到#时,就pop,然后把最后的那个元素的children后边加上下一个遇到的元素.

```
class Codec:
    def serialize(self, root):
        serial = []

        def preorder(node):

            if not node:
                return

            serial.append(str(node.val))

            for child in node.children:
                preorder(child)

            serial.append("#")      # indicates no more children, continue serialization from parent

        preorder(root)
        return " ".join(serial)

    def deserialize(self, data):
        if not data:
            return None

        tokens = deque(data.split())
        root = Node(int(tokens.popleft()), [])

        def helper(node):

            if not tokens:
                return

            while tokens[0] != "#": # add child nodes with subtrees
                value = tokens.popleft()
                child = Node(int(value), [])
                node.children.append(child)
                helper(child)

            tokens.popleft()        # discard the "#"

        helper(root)
        return root
```

###### 6 设计一个Data Structure 使得insert, delete和getrandome的时间都是O(1).

思路: 设计一个array,一个map, array用来store values, map 用来map一个int到storage上.

insert就是把值放到array的最后.

remove就是把找到的那个值和array的最后的那个值互换,更新map,并删去.

getrandom就是在array中随机找一个数.

```
import random

class RandomizedSet(object):

    def __init__(self):
        self.nums, self.pos = [], {}

    def insert(self, val):
        if val not in self.pos:
            self.nums.append(val)
            self.pos[val] = len(self.nums) - 1
            return True
        return False


    def remove(self, val):
        if val in self.pos:
            idx, last = self.pos[val], self.nums[-1]
            self.nums[idx], self.pos[last] = last, idx
            self.nums.pop(); self.pos.pop(val, 0)
            return True
        return False

    def getRandom(self):
        return self.nums[random.randint(0, len(self.nums) - 1)]
```

###### 7 设计一个frequency stack,每次pop出来的是frequency最大的那个.
class FreqStack(object):

    def __init__(self):
        self.q = []
        self.sc = {}
        self.mark = 0             <- mark用来表示输入的顺序大小.
        import heapq
        heapq.heapify(self.q)

    def push(self, x):
        self.sc[x] = self.sc.get(x, 0) + 1
        heapq.heappush(self.q, [-self.sc[x], -self.mark, x])
        self.mark += 1      

    def pop(self):
        x = heapq.heappop(self.q)
        self.sc[x[2]] -= 1
        return x[2]
