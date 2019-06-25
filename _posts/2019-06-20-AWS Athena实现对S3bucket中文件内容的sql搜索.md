---
layout: post
title: AWS Athena实现对S3bucket中文件内容的sql搜索
date: 2019-06-20
tag: 软件设计
---

context： 我们在s3上存有数据，客户需要读取这些数据。客户来自不同对背景，有的用SAS，有的用R，有的用python，所以就造成了一个困难，如果我们只提供一种语言的interface的话就会有客户不知道该怎么用。我们需要一个统一的口径来让客户读取数据。我们最终选择的是在s3的files上建立一个sql搜索引擎，因为几乎所有的人都会sql。presto是开源的big data sql query engine。在AWS下，对应的是Athena服务。下边就是整个的structure图示：

![campaction](/assets/images/AWS Athena.png)

Flow:

1. 先设置Server-Side加密，这里是KMS加密，由我们控制private key。
2. Glue能自动提取S3中file中的数据的schema，我们通过policy和role来给Glue的crawler读取S3的权限。读出的schema存储在catalog中。
3. Athena会从catalog中读取schema并从S3中读取数据，我们可以在Athena中输入sql语句来query。


