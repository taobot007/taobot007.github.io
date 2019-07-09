* 使用stack遍历tree的问题

* 设计customer who bought this item also bought ...

* 怎样实现netflix电影推荐,怎么scale到全世界.

* 实现限流器

* lc 108 Convert  Sorted Array to binary Search Tree

* lc 253 Meeting roos II 问最多需要多少个meeting room来开会

* BQ: 是有没有发生过manager 不在自己做决定的时候，以及most challenge。

* LRU的设计

* list movies with actors info

* 例如队友不给力怎么办，你怎么激发队友，你跟队友的矛盾，你作为领导的经历等。总之很多都是关于处理人与人之间的关系。

* lc212 word search II, 查看一个matrix中是不是有word在dictionary里.

* 设计一个Token Pool，这个之前没听说过，想着应该是跟连接池，线程池差不多的东西。没画图，直接让我写代码实现。

* L13 Roman to integer

* 一轮design，简单的可怕。就是设计一个让用户注册的系统，只有名字和email。简单聊了聊也没咋challenge我

* Meeting room的简单版. 一堆排好序的有time interval的tasks, 在没有交叉的情况下，一共可以跑多少tasks. 这里不是问最多的tasks，从左到右扫一遍，有交叉的直接跳过就行; followup是每个tasks都有权重，找到在没有交叉的情况下，所有tasks的最大权重和。仔细想了下就类似longest increasing subsequence, dp就搞定了。 第三题是给一堆单词，找到出现次数最多的K个单词，这里的input是数据流，所以就heap把.

* OOD：设计cache并代码实现. 主要就是interface，ttl，generic type的input，怎么clean等等。应该是挂在这轮了，答得不太好

* Max length of consecutive numbers in binary tree 国人大哥,给一个binary tree，按升序找出连续数字的最大路径长度，

* Project dependency，返valid order list……就topo sort加cycle detection……没啥好说的

* LRU

* BQ
Move forward or collecting more data
Work with incomplete information
Tough feedback
Disagreement with peers
Good proposal but manager didn't take actions
Transfer project ownership to others

* Word Search

* L419 Battleship in a Board
Given an 2D board, count how many battleships are in it. The battleships are represented with 'X's, empty slots are represented with '.'s. You may assume the following rules:
You receive a valid board, made of only battleships or empty slots.
Battleships can only be placed horizontally or vertically. In other words, they can only be made of the shape 1xN (1 row, N columns) or Nx1 (N rows, 1 column), where N can be of any size.
At least one horizontal or vertical cell separates between two battleships - there are no adjacent battleships.

要求:只scan一次,使用1memory, 不能改变board value
trick: 只有ij位置上为X且x-1,j, 和 x, j-1位置上不为零的时候才能trigger count++

*  system design traffic light control system

* processor depedency， 给一个 list of list， [[a,b],[b,c],[c,d],[h,t]], return one of the correct dependency result e.g "abcdht"

*  design voting system

* ood: file search

* 138 Copy list with random pointer

* prime video watch count. 考虑未登录与登陆情况，考虑bot刷点击率情况，考虑qps超高的情况

* 算法题：数字转罗马数字，刷题网12，这道题一年前刷的，解法全给忘了，只能现场想，有点小慌。所幸解出来了，就是解法没那么简洁。

* * frustrated interaction with user
* tight deadline
* receive bad feedback
* dive deep several layers to figure out a problem
* help other people, ownership
* innovative solution for a problem
* you can do better if you have another chance
* took a big risk
* can not meet a commitment
* a difficult decision
* a simple solution for a problem
* do something bigger than the initial focus

* 设计一个在飞机上的Netflix，就你在坐飞机的时候能看电影啊啥的。其实后来想了一下并没有很确定这个”在飞机上“的意义是什么，可能只是限制user的数量免得让题目太难毕竟一台飞机上最多也就1000人（我反正按照1000人来算storage啊bandwidth啥的）？

* disagree with someone
* have conflicts
* negative feedback.
* challenging project
* need more information. 
* did more than what was required
