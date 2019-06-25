* 1

给你一个二叉树，再给你一个节点集，这个节点集里的节点可能在这个树上，也可能不在。要求把以这个节点集里的节点从二叉树里删除，要求返回所有未删除的树或者子树的根节点
比如
        1
  2          3
4  5       6   7

set = [2, 3]
那么结果是 [1, 4, 5, 6, 7]，因为删除2, 3节点后，1成了一个单独的节点，4,5,6,7成为4个单独的子树。

* 2
一共两道题，第一道题就是two pointer。给一个list 然后两个目标的int， 找出这两个int的最小距离（这两个int是会重复多次出现的）。第二题是第一题的follow up 在一个grid[][]里面找两个点的最小距离

* 3
考的是island变种
给的是matrix

[[1, 0, 0, 1], [0,0,1,0], [1, 1, 1, 0], [0,0,0,1]]

第一行挨着ceiling，与ceiling接壤的1可以走，它的上下左右如果也有1也可以走   
走完后，matrix里面如果有1没有走到，就是孤独的1
问题是，如果改变某个地方的值，matrix[i][j] 被flip了以后  新的matrix是否还有孤独的1，以及与原来的matrix的孤独的1如果有出入，就输出这些不一样的孤独1

* 4
狗店面
1. lc 22 Generate Parentheses
2. lc 249 Group Shifted Strings
3.
Board = [
  A B C D E
  F I J K L
  M N O P Q
  R S T U V
  W X Y Z
]
Board 总是按A-Z的顺序排下来的，
Input “CARS” - target word
M - 每行m个字母
Output: RR!LL!DDD!R!

* 5
两轮电话面试，都是存写code，没有什么BQ，打电话然后用Google Doc写答案。
第一题是刷题网没有的题目，题目大致内容有一点忘记了，但是大方向还是记得的。
给你一个list， 里面有各种小写的a-z的字母，比如是这样的<a,b,b,a,c,c,c>，
然后这个list Iterator 只有两个method，一个是 boolean hasNext(), return 有没有下一个字母，但是他的pointer不会往后走，
另一个是method是 char next(), 返回的是list里面下一个元素（前提是他有下一个元素的话），调用这个method会让pointer往后走。

然后让你implement 一个aggregate iterator class, 但是只能调用 Iterator的next 和 hasnext function。
然后这个aggregate iterator class 支持的operation也是 hasnext 和 next， 每个要做的功能和iterator一样。
但是next return 的是一个pair<元素，元素连续出现的次数>， 以上面那个<a,b,b,a,c,c,c>为例子，一直调用next 输出的结果就是 <a,1>,<b, >2, <a,1>, <c,3>。

第二轮是 刷题网的题目，巴肆武，找最高的山。~

* 6
Split Array Largest Sum

* 7
 Meeting Rooms II
 Given an array of meeting time intervals consisting of start and end times [[s1,e1],[s2,e2],...] (si < ei), find the minimum number of conference rooms required.

longgest consecutive stringinput(string w1, string w2)
example:
w1:aaabbbcccddd.
w2:abcd

* 8
将一个N叉树序列化成字符串，并反序列化回N叉树，类似于利口似而巴，不同之处是树的val是字符串。

要求自己创建TreeNode。考点是dfs和分隔符怎么处理，比如，如果你用逗号做分隔符，那如果一个节点的value结尾就是逗号，你怎么办。

* 9
问设计server rebalance， 给出一些server ID和 weight， 收到web request后如何挑选web server。

刷了许多离扣，没见过店面这种的。

楼主只想到算每个server probability，但是具体怎么返回哪个server有点懵，不知道在收到200 request 以后再来一个request应该怎么选server。

* 10
题干：
定义: 根据lexicographical order, 字母a最小，z最大；接下来比较每个字符串中最小字母的出现频率，如果A 的该频率小于B的该频率，则称A strictly smaller than B。
如：'a' <‘bb’, 因为左侧字符串中最小字母为‘a’其频率为1，而右侧字符串最小字母 'b' 频率为2.

输入: def solution(A,B) 字符串A和B 每个都包含多个以空格间隔的子字符串，内容都是lowercase letter。如A = 'abcd aabc bd', B='aaa aa'
输出: 对于B 中每个子字符串，找出A 中有多少个子字符串比它 strictly smaller， 并返回这个数值。如上述AB的对应输出为 [3,2]

题干：
输入：数组A （包含 dictinct integars），int K。 e.g. A = [1,4,3,2,5], K = 4;
取A中K个连续元素构建一系列子数组，如上则是[1,4,3,2], [4,3,2,5]，返回largest contiguous subarray。
largest contiguous subarray定义：逐项比较两数组元素，对于第一对non-matching的元素，值大的数组larger.
e.g. X = [1,2,4,3,5],  Y = [1,2,3,4,5], ----> X > Y, 因为X[2] > Y[2]。

* 11
1. 给一个长方形(x0, y0, x1, y1), 随机产生一个点落在里面，要求均匀分布。（给rand API产生0-1的均匀分布）简单的，直接call 两次 产生x, y 坐标即可。 （其实call一次也可以吧，做一个除法）
2. 如果有两个长方形，不重叠，要求随机点。也简单的。
3. 如果有N个长方形，不重叠， 要求随机点。也简单的。
4. 两个长方形， 是否重叠，LC有原题 （换而言之就是一个在一个的上下左右）
5. 重叠的面积怎么算？也简单的。
6.  产生一个随机点，随机均匀分布在图形中。  切割呗。。。。. From 1point 3acres bbs
7. 如果有N个相互重叠的， 要求产生一个点，均匀分布在这个图形里面，重叠部分应该是不叠加的，怎么做。。。   时间也差不多了，不会了.
切割呗。。。 怎么切割， 不会了

* 12

给一个类似输入 {a}, {b, c}, {d, e}, 输出所有的combination =>
abd, acd, abe, ace
算法部分比较简单，一个backtracking就可以解决。
唯一要注意的parse input的部分有些繁琐，当时考虑performance没有用library 想one pass 搞定，结果code 写的不是很简洁... 和面试官沟通了一下他表示理解 ；）
这里得到一个教训，在面试里代码整洁性还是比较重要的（如果对复杂度影响不大的话）

* 13 <- 有问了两遍
给一个数字N，输出小于等于N数字里会由于旋转产生Confusion的数字。比如18旋转180度后变81，就是Confusion。69旋转后还是69就不是confusion。一开始上手直接RECURSION做，但是卡了一会。面试官说他没想过这题用DFS做，但是是一个很好的approach。建议让我先写个Brute force的，大概10分钟不到写完时间也差不多到了。问了些问题就结束。

* 14
一上来就复制粘贴了一个题目 开做
一个二维坐标图，有很多points，代表cities （坐标可以任意正负）
只能往 next.x > curr.x && next.y > curr.y 的方向走
返回有最多点的path的长度，即点数

follow up: 如果点数特别大，会怎样？

* 15
第一轮感觉是一个白人小姐姐，自我介绍都没有上来直接做题。题目不难，candy crush简化版，给一个array，返回crush之后的array，比如[1,2,2,3,3,3,2,1] 输出[1,1]. 说了用stack，写代码。注意下3个以上的情况。 follow up: 1, 空间 时间 2，有没有优化，答可以in-place 用2 pointer 3问了下2 pointer的时间复杂度。

第二轮应该是国人小哥，找二叉树中最大的identical tree，返回最大的node个数。如果两个subtree结构完全一样就是identical tree。 我是从root开始，用set存每个节点的in-order traversal的list,如果set里已经有了，就更新max值。folllow up 1 时间空间复杂度 2 怎么优化，答了serialization优化空间，问时间能不能优化，答如果paretn匹配了identical tree，它的子节点就不用计算了，因为肯定比现在的小，但还是n^2。问了hint，他说如果从下往上遍历呢，没理解他的意思随便扯了扯。 时间到了就问问题了。 所以这个还能再优化吗？？

* 16

第一题，给一个string pattern，由字母（a-z) 和 ‘_' 组成，例如 “_ _ a _ _",    再给一个word， 问word 能不能 match 这个pattern。 ‘_’  代表 blank， 可以match all characters。follow up， 如果有大量固定这样的word，如何用最快的办法把所有 match 的word 找出来。
第二题。面试官给了一种游戏的玩法，问我玩过没，我说没有，面试官人很好，然后就换了一道别的题， 是 pascal 三角的变形。
总体感觉题还比较简单，但是第一题的follow up 答的不是很好。

* 17

炸点给一个起点坐标和一个终点坐标，问这两个点的可达性，炸弹半径为d。

联通问题.

* 18 Basic Calculator
Implement a basic calculator to evaluate a simple expression string.

The expression string may contain open ( and closing parentheses ), the plus + or minus sign -, non-negative integers and empty spaces .

* 19
Given a list of points in 2D space, to determine whether there is a vertical line such that all these points are symmetric to this line?

- I used two-sum type approach. creating a set and check whehter its symmetric point is in the set of not.

There are  lot of follow ups?
1. What if there are duplicate points
2. When the points are float points, how will check whether two real numbers are equal?
3. What if the we remove the vertical line constraint?

* 20

第一轮 数组 找一对数 使得和最接近target
第二轮 数组 找一个坐标 使得坐标左面的和等于右边

* 21

这周一的电面，今天hr电话通知过了准备onsite。。。解 8-puzzle/数字华容道:
start:
2 8 5  
4 1 6  
0 3 7  
target:
1 2 3
4 5 6
7 8 0
给任意初始棋盘start和目标棋盘target用二维数组表示，求移动“0”来达到最终状态的最小步数，如果不可能返回-1.

应该是算是比较标准的BFS吧。用string来记录棋盘状态。

* 22
上来先做一维数组range sum query 然后follow up就是range max query 同样是一维数组
想了半天想出用segment tree来做 但是代码写完以后一直觉得有些问题 华人大哥最后还是给放过了.

* 23
(1)找可行路径 题目是类似这种:
1 0 0 1 0
1 1 0 0 0
0 1 1 0 0
找有没有可行的路径从第一行 走到最後一行

* 24
接着是一道family tree的题目。

将父母简化成一个节点。

1.写出family tree的class(不需要写构造函数和析构函数)
2.计算两个节点之间的距离 如果两个人没有血缘关系就返回-1，距离是指从一个节点到另外一个节点最少需要走几步。  可以找最短公共父节点 然后计算距离

* 25
Input:
list of tuple, 每个tuple有个名字和List of string
[
("a", [str_a1, str_a2, str_a3, str_a4....]),
("b", [str_b1, str_b2, str_b3, str_b4, ...]),
("c", [str_c1, str_c2, str_c3,...]),
....
]

output:
输出所有pair of tuple， 用tuple名字表示
要求：其中一个tuple是另一个tuple的subarray

example:
input:
[("a", ["1", "2", "3"]), ("b", ["2","3"]), ("c", ["1", "2"])]
output:
[("a", "b"), ("a", "c")]

* 26
昨天发的题目是，给两个string, A and B。同样的长度，并且都是由小写字母组成。返回是否可以把A变成B。规定每一次只能变A中的一中字母，并且同样的字母要变就得一块变，且必须变成同一个字母。

比如A = "cbc", B = "gfg",   那么转变顺序就是 "cbc" -> "gbg" -> "gfg"，返回true。

* 27
given a n level perfectly balanced binary tree, the value of each node has a value = post order traverse value.
for example, a n = 3 tree will look like this:

         7
     /      \
   3         6  
  / \        / \ . check 1point3acres for more.
1   2        4  5

write a function that will return each node's parent in an array
public int[] findParent (int n, int[] q)

example: n = 3, q[] = {1 3, 6 , 7} will return {3, 7, 7, -1}
7, 6, 5, 4, 3, 2, 1
1, 2, 3, 4, 5, 6, 7
0, 1, 2, 3, 4, 5, 6

       6 -> [0, 2], [3, 6]
            [0, N / 2 - 1], [N / 2, N - 1],
             subarray

       6: base = 0      base + [N / 2 - 1, N - 1]
       5: base = 3      [3, 5 / 2 - 1 + 3] = [3, 4],
      build the tree but that is not optimized

      i1, N

      while N:
        mid = N / 2 + base
        if mid - 1 == i1:
          return N
        elif N - 1 = i1:
          return N
        elif i1 < mid -1 :
          N = N / 2
          base = base
        elif i1 >= mid:
          N = N -1
          base = N / 2

* 28
第一题algorithm其实很简单，给2点(x1, y1), (x2, y2), 让求出连线的所有点。主要就是些edge case好烦人，比如说垂直的情况斜率无穷大啊之类的。感觉我就是在这些edge case上浪费太多时间了，经过面试官耐心提示最后终于搞定，但半个小时已经过去了。。。

第二题题目描述一大段，说白了总结起来就是给一组数，让分成k个subarray，然后让子和尽量平均。换言之就是找max(min(subarray))。感觉应该是leetcode上的题，但我没见过，只好当时硬着头皮死想。最后也没想出来，只好说了几个思路，然后就没时间了。。。（有人知道这是哪一题么？）

* 29
一个三哥让我写bingo card。。开始不知道那是个啥。。。解释了一下 说是一个array， 5*5第一列是1-7，第二列8-14，第三列15-21， 依次类推 ， （2，2）这个位置是空的。
有个现成的api，叫generateRandom（int start, int end）来生成随机数字
1. 写个方法随机生成一个bingo card， 要求同一列不重复。
2. 写个方法随机生成很多bingo cards，要求每一个card的列跟别的card的列不能一样。
然后问了时间复杂度。
过几天hr打电话来说，不move forward了。感觉写的中规中矩啊，也不算慢。
请教各位大神，这个有什么算法来处理 随机生成bingo card一列的吗？除了最裸的生成一列去set里面查重复，有没有别的精妙算法是这个面试官想看到的？
还问了时间复杂度和空间复杂度。。。不好说啊有这个，如果有很多重复就会重新生成新的一列数组。。这个要怎么说呀 请各位大神提示

* 30
给你一个字串 s1，问你能不能透过不限数量的某操作换成 s2，只需要回答 true 或 false
某操作：
  选一个字母，把整个 s1 中的这个字母全部换成另一个字母
. 1point3acres
比如：
s1 = aabcc, s2 = ccdee
aabcc -> aabee -> aadee -> ccdee
要注意顺序是有差的，比如不能先把 a 换成 c
aabcc -> ccbcc -> ? eebee

挺难的.应该是用toplogical sort然后看到底有多少个loop留下的方法来判断.         

* 31
有两个list:A,
B,
返回：
all element in A that not in B,
all element in B that not in A.

问了一些问题去clarify。比如在A,B里面是否有duplicated的。然后list里面是什么东西，面试官说都行，我就和她说假设是integer吧，开始做的时候面试官没讲清楚，比如:
A:1,2,2,2,3,4. From 1point 3acres bbs
B:2,5. From 1point 3acres bbs
我返回了：
resultA: 1,3,4
resultB:5
写完之后面试官告诉我不对，应该返回:
resultA: 1,2,2,3,4
resultB: 5
面试官开始没讲清楚，我就按照题意去理解了。反正改起来也不难，就又重新写了。

开始的做法是最直观的。用了两个HashMap存每个list里面每个integer的frequency。每个list走两遍。这样time是O(m+n)。m是A的size，n是B的size。space是result两个list和两个HashMap。
然后写完后，面试官和我说能不能不用HashMap。我就问了两个list是sorted的吗，她说不是。我问那我可以sort一下，然后用two pointer。但是这样time就变成了Max(mlogm,nlogn, m+n)。
对方没直接回答我，就说如果在去掉HashMap的基础上，牺牲点时间是必要的，那就可以。然后我就用two pointer去做了。
逻辑就是：在sorted的A和B中，各keep一个pointer。
如果A[pA] < B[pB] 那A[pA]肯定是 element in A that not in B --> pA++
如果A[pA] > B[pB] 那B[pB]肯定是 element in B that not in A --> pB++
如果A[pA] == B[pB] 那A[pA]和B[pB]都不符合要求 --> pA++, pB++

* 32 
L465
A group of friends went on holiday and sometimes lent each other money. For example, Alice paid for Bill's lunch for $10. Then later Chris gave Alice $5 for a taxi ride. We can model each transaction as a tuple (x, y, z) which means person x gave person y $z. Assuming Alice, Bill, and Chris are person 0, 1, and 2 respectively (0, 1, 2 are the person's ID), the transactions can be represented as [[0, 1, 10], [2, 0, 5]].

Given a list of transactions between a group of people, return the minimum number of transactions required to settle the debt.
