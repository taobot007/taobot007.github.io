---
layout: post
title: Object Oriented Design
date: 2016-06-10
tag: Leetcode
---

#### 解决步骤

* 第一步: 澄清问题: six Ws 原则: who, what, where, when, how, why

* 第二步: 列出核心object

* 第三步: 分析object间关系

* 第四步: 定义每个object的动作

#### Design Patterns

* Singleton

* Factory Method

* ...

#### 例题与解答

* Deck of Cards: Design the data structures for a generic deck of cards. Explain how you would subclass the data structures to implement blackjack.

第一步: 向interviewer澄清问题.假设interviewer告诉你这是一个普通的二人扑克游戏.

第二步: 列出核心object: card, deck, hand

第三步: 分析关系: deck上边会放着待分配的card,打出去的不用的card,还有刚打出来准备比大小的card; hand里会有card;card上边有value和花色

第四步: 定义动作: deck会用来比较card大小,存储并shuffle card,分配card,比较card; hand会收集card并打出card
