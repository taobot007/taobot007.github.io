---
layout: post
title: Scorecard 模型实践
date: 2019-07-17
tag: Big Data and Machine Learning
---

Scorecard Model:


WOE(Weight of Evidence): 证据权重转换可以将logistic回归模型转变为标准评分卡格式。该转换过程能够简化模型的应用且增加业务解释性，同时能将特征与标签之间的非线性关系转化为线性的。

scorecard::woebin函数tree方法分箱的原理类似决策树过程。 不同的变量类型的分箱过程稍有差异.

scorecard::woebin函数chimerge分箱为自底向上的数据离散化方法，即将具有最小卡方值的相邻区间合并在一起。具体内容参见chimerge算法或原论文ChiMerge:Discretization of numeric attributs.

问题：
1. Monotonicity Constraint, 和WOE什么关系？
2. 怎样分箱， binning？ Chimerge？
3. 拒绝推断： 申请评分卡的模型开发过程中使用的数据实际上并不是从申请总体样本中随机选择的，而仅仅是从过去已经被接受的客户样本中选择的。因此，开发申请评分卡时将对被拒绝客户的状态进行推断并纳入模型开发数据集中，即拒绝推断过程。

拒绝推断的常用方法包括，

简单赋值法：人为指定被拒绝账户的标签
    忽略被拒绝申请
    所有被拒申请赋值为违约标签
    按比例赋值，使得其坏客户率是通过样本的2~5倍以上
强化法：通过外推法确定拒绝账户的标签
    简单强化法：使用通过客户开发的模型对被拒绝客户评分，将其中低分段赋予违约标签。使得拒绝客户的坏客户率为通过的2~5倍以上
    模糊强化法：通过模型计算得到正常和违约概率。
    打包强化法：先用开发的评分卡对被拒客户评分，然后指定每个分值区间的违约客户数量。
4. 模型评估指标： KS？ AUC？ Gini？ROC?
5. 稳定性指数？ PSI？
6. L1, L2 norm regulation?
7. 单变量分析： univariate analysis


## 前提知识：

* ChiMerge: 用来合并bin，而要了解chi-merge，就要了解chi-square test

* KS score: 用来查看单边量对于y值的重要程度


### chi-square test

chi-square test 卡方检验就是统计样本的实际观测值与理论推断值之间的偏离程度，实际观测值与理论推断值之间的偏离程度就决定卡方值的大小，如果卡方值越大，二者偏差程度越大；反之，二者偏差越小；若两个值完全相等时，卡方值就为0，表明理论值完全符合。

chi-square test 步骤：

1.  提出原假设：

    H0：总体X的分布函数为F(x).

    如果总体分布为离散型，则假设具体为 H0：总体X的分布律为P{X=xi}=pi， i=1，2，...

2.  将总体X的取值范围分成k个互不相交的小区间A1，A2，A3，…，Ak，如可取

    A1=（a0，a1]，A2=(a1，a2]，...，Ak=(ak-1,ak)，

    其中a0可取-∞，ak可取+∞，区间的划分视具体情况而定，但要使每个小区间所含的样本值个数不小于5，而区间个数k不要太大也不要太小。

3.  把落入第i个小区间的Ai的样本值的个数记作fi，成为组频数（真实值），所有组频数之和f1+f2+...+fk等于样本容量N。

4.  当H0为真时，根据所假设的总体理论分布，可算出总体X的值落入第i 个小区间Ai的概率pi，于是，N*pi就是落入第i个小区间Ai的样本值的理论频数（理论值）。

5.  当H0为真时，N次试验中样本值落入第i个小区间Ai的频率fi/N与概率pi应很接近，当H0不真时，则fi/N与pi相差很大。基于这种思想，皮尔逊引进如下检验统计量

     $x^2 = \sum_{i=1}^{k}  \frac{ (f_i - N * p_i)^2 }{N * p_i}$

    ，在0假设成立的情况下服从自由度为k-1的卡方分布。

chi-sqare test 的应用：

判断两个变量X与Y到底有没有关系。

假设有两个分类变量X和Y，它们的值域分别为{x1, x2}和{y1, y2}，其样本频数列联表为

| ---- | y1 | y2 | 总计 |
| x1 | a | b | a + b |
| x2 | c | d | c + d |
| 总计 | a + c | b + d | a + b + c + d |

若要推断的论述为H1：“X与Y有关系”，可以利用独立性检验来考察两个变量是否有关系，并且能较精确地给出这种判断的可靠程度。具体的做法是，由表中的数据算出检验统计量$x^2$的值。这个值越大，说明“X与Y有关系”成立的可能性越大。

具体例子：

H0: 性别与化妆与否没有关系。

然后我们看如下表：

| ---- | man | woman | total |
| makeup | 15 (55) | 95 (55) | 110 |
| no makeup | 85 (45) | 5 (45) | 90 |
| ---- | 100 | 100 | 200 |

如果性别和化妆与否没有关系，四个格子应该是括号里的数（期望值，用极大似然估计55=100*110/200，其中110/200可理解为化妆的概率，乘以男人数100，得到男人化妆概率的似然估计），这和实际值（括号外的数）有差距，理论和实际的差距说明这不是随机的组合。

应用拟合度公式:

$x^2 = \sum_{i=1}^{k}  \frac{ (f_i - N * p_i)^2 }{N * p_i} = \frac{ (95 - 55 )^2 }{55} + \frac{ (15 - 55 )^2 }{55} + \frac{ (85 - 45 )^2 }{55} + \frac{ (5 - 45 )^2 }{55} = 129.3$

而我们知道当$x^2$ > 10.828时，就以为着有99.9%的可能H0是不成立的。所以我们可以知道性别与化妆是有很大关系的。

### ChiMerge

ChiMerge 是监督的、自底向上的(即基于合并的)数据离散化方法。它依赖于卡方检验：具有最小卡方值的相邻区间合并在一起，直到满足确定的停止准则。基本思想：对于精确的离散化，相对类频率在一个区间内应当完全一致。因此，如果两个相邻的区间具有非常类似的类分布，则这两个区间可以合并；否则，它们应当保持分开。而低卡方值表明它们具有相似的类分布。


关于分箱的讨论：

    1. 最简单的离散算法是： 等宽区间。 从最小值到最大值之间,，均分为NN等份， 这样， 如果A,BA,B为最小最大值， 则每个区间的长度为W=(B−A)/NW=(B−A)/N, 则区间边界值为 A+W,A+2W,….A+(N−1)WA+W,A+2W,….A+(N−1)W.

    2. 还有一种简单算法，等频区间。区间的边界值要经过选择，使得每个区间包含大致相等的实例数量。比如说 N=10N=10，每个区间应该包含大约10%的实例。

    3. 以上两种算法有弊端：比如，等宽区间划分，划分为5区间，最高工资为50000，则所有工资低于10000的人都被划分到同一区间。等频区间可能正好相反，所有工资高于50000的人都会被划分到50000这一区间中。这两种算法都忽略了实例所属的类型，落在正确区间里的偶然性很大。

    4. C4、CART、PVMC4、CART、PVM算法在离散属性时会考虑类信息，但是是在算法实施的过程中间，而不是在预处理阶段。例如，C4C4算法（ID3决策树系列的一种），将数值属性离散为两个区间，而取这两个区间时，该属性的信息增益是最大的。

    5. 评价一个离散算法是否有效很难，因为不知道什么是最高效的分类。

    6. 离散化的主要目的是：消除数值属性以及为数值属性定义准确的类别。

    7. 高质量的离散化应该是：区间内一致，区间之间区分明显。

    8. ChiMergeChiMerge算法用卡方统计量来决定相邻区间是否一致或者是否区别明显。如果经过验证，类别属性独立于其中一个区间，则这个区间就要被合并。

    9. ChiMerge算法包括2部分：1、初始化，2、自底向上合并，当满足停止条件的时候，区间合并停止。

步骤

    1. 初始化

    根据要离散的属性对实例进行排序：每个实例属于一个区间

    2. 合并区间，又包括两步骤

        (1) 计算每一对相邻区间的卡方值

        (2) 将卡方值最小的一对区间合并

    预先设定一个卡方的阈值，在阈值之下的区间都合并，阈值之上的区间保持分区间。

卡方的计算公式：

![chi calculation](/assets/images/chi square equation.png)

### KS test and KS score

KS检验是比较一个频率分布f(x)与理论分布g(x)或者两个观测值分布的检验方法。其原假设H0:两个数据分布一致或者数据符合理论分布。D=max| f(x)- g(x)|，当实际观测值D>D(n,α)则拒绝H0，否则则接受H0假设。KS检验与t-检验之类的其他方法不同是KS检验不需要知道数据的分布情况，算是一种非参数检验方法。

PS：t-检验的假设是检验的数据满足正态分布，否则对于小样本不满足正态分布的数据用t-检验就会造成较大的偏差，虽然对于大样本不满足正态分布的数据而言t-检验还是相当精确有效的手段。

在金融领域中，我们的y值和预测得到的违约概率刚好是两个分布未知的两个分布。好的信用风控模型一般从准确性、稳定性和可解释性来评估模型。一般来说。好人样本的分布同坏人样本的分布应该是有很大不同的，KS正好是有效性指标中的区分能力指标：KS用于模型风险区分能力进行评估，KS指标衡量的是好坏样本累计分布之间的差值。好坏样本累计差异越大，KS指标越大，那么模型的风险区分能力越强。

KS的score计算步骤如下：

    1. 计算每个评分区间的好坏账户数（计算的是特征的KS的话，是每个特征对应的好坏账户数）。
    2. 计算每个评分区间的累计好账户数占总好账户数比率(good%)和累计坏账户数占总坏账户数比率(bad%)。
    3. 计算每个评分区间累计坏账户占比与累计好账户占比差的绝对值（累计good%-累计bad%），然后对这些绝对值取最大值即得此评分卡的KS值。

或者一句话： KS score 是 真阳率 - 假阳率 （ TPR - FPR ）

### ROC 曲线

接收者操作特征曲线（receiver operating characteristic curve，或者叫ROC曲线）是一种坐标图式的分析工具，用于 (1) 选择最佳的信号侦测模型、舍弃次佳的模型。 (2) 在同一模型中设定最佳阈值。

ROC曲线首先是由二战中的电子工程师和雷达工程师发明的，用来侦测战场上的敌军载具（飞机、船舰），也就是信号检测理论。

ROC分析的是二元分类模型，也就是输出结果只有两种类别的模型，例如：（阳性／阴性）。

ROC空间将伪阳性率（FPR）定义为 X 轴，真阳性率（TPR）定义为 Y 轴。

TPR：在所有实际为阳性的样本中，被正确地判断为阳性之比率。

TPR=TP/(TP+FN)

FPR：在所有实际为阴性的样本中，被错误地判断为阳性之比率。

FPR=FP/(FP+TN)

给定一个二元分类模型和它的阈值，就能从所有样本的（阳性／阴性）真实值和预测值计算出一个 (X=FPR, Y=TPR) 座标点。

用图形的方式来理解：

![ROC](/assets/images/ROC_curves.png)

从图中我们可以看出来，一个feature能不能很好地把P和N的distribution分开。如果P和N的distribution在feature轴上分得很开，那么ROC的曲线肯定离中线很远，如果不能，即P和N的distribution基本是重合的，那么ROC的曲线就和中线重合了。

而且 ROC 曲线不一定在中线上方，如果对于一个feature轴，P在轴的左边，N在轴的右边，那么ROC曲线就在中线下方，意味着feature和tag负相关。

### ROC 与 KS score的关系

KS = max(abs(TPR - FPR))

而 ROC = TPR = f(FPR)

中线上： g(FPR) = FPR

那么 KS = max(abs（TPR - FPR) = max(abs(ROC - g(FPR))) 即 KS值就是ROC曲线与中线的最大差值。


### Feature Selection

I will share 3 Feature selection techniques that are easy to use and also gives good results.

1. Univariate Selection   ----> 依据 $x^2$ statistical test

2. Feature Importance     ----> 依据Decision Tree中的Gini系数或者Entropy

3. Correlation Matrix with Heatmap ----> 计算 correlation 的大小

### Domain Knowledge

1. Observation Date: 指的是开始观察对时间。所有对attribute都是在这个时间之前对信息，而tag是这个时间之后的预测，比如说这个日期之后的十二个月。

2. We divide the data into 4 data sets: Training, In-Time Testing, Out-of-Time Testing, and Holdout. Training is easy to understand. Holdout is the final performance checking process. (Outsiders usually call this one Testing). Why do we want In-Time Testing and Out-Time Testing? Because it is possible that training gives 50 KS, in-time testing gives 51 KS and out of time testing gives 23 KS. That means out model is not overfitting, however there is some time variable hidden in the dataset. We need to look deeper into the dataset.

## 我的实践

### 从XX team拿到数据之后，我接到的任务是“study the data”.这是个很模糊的任务。我做了以下的步骤：

1. 查看了datafram的shape，有多少列，多少行

2. 找到tag是哪个，id是哪个，time在哪里

3. 看看针对time的bad rate的distribution是什么.

4. 试着解释每个attribute是什么意思

5. 在这个过程中，我发现这个dataset有5000多个attribute，一个一个解读它们将是非常繁重的工作。所以我把它们归成几十个大类，聚焦打击范围

6. 而record共有2million个，对于我拥有的资源来说，数量偏大。我采用了XXX的建议，抽取了百分之五的样本。这样计算就快捷多了。

7. 根据XXX的建议，我要对每个attribute计算出KS score。然后按照score从大到小排列attribute，并选取其中的几百至一千多个进行更深的分析。

8. 同时，我要对bad rate进行一个时间上对分析，看看有没有什么问题。这个时间是observation date,而tag是一年之后会产生的。

9. XXX 部门告诉我们他们给这些data分了segmentation，所以我对每个segmentation也做了bad rate 在时间上的分析。 对于大多数的segmentation， bad rate越靠近结束越低，这是正常的，因为还没有产生违约。对于 Seg 1 & 2 的用户，他们是已经有违约的人员，他们的bad rate 走势正常。而seg 3-10 的用户，他们是没有发现任何违约的人，所以到结束的时间点，他们不可能一下子产生3个月的违约，但是他们的bad rate并没有drop to 0 in May 2019，而我们知道这些data是 June 2019 整理的，所以这是有问题的。可能会有target leak的问题产生。这是我们看到的第一个问题。这个问题至今没有很好的答复。我们向他们要了trade data，但是还没有回应。

10. 第二问题是，seg 11的用户，所有的bad rate都是0。这点我们也问了，没有得到很好的解释。

11. 后来他们告诉我们，trainning data time range is April 2017 to March 2017 and validation date is April 2017 to June 2017. 我们看了一眼各个seg上的bad rate，是stable的。所以9，10中的问题我们没有追究下去。

12. 他们还给我们几个版本的data dictionary， 用来描述每个field是什么意思，能不能用。我们向他们要每个variable的special values和value range，因为这对于我们建立模型很重要。他们至今未给。

13. 我们looked into validation data set. 依旧是看了以下几点： 1） population in each segmentation； 2) bad rate over time in each segmentation；

14. 我们发现了validation data 的几个问题： 1）positive population只占了总 population 的 6%。这虽然与实际情况相近，但是因为我们model的卖点是能很好地handle positive data，所以我们认为validation population里边应该有更多的positive data； 2） 每个segmentation的sample都是random selected的，当数量很大时问题不大，但是当数量比较小时有可能出问题，比如说我们看到的bad rate over time 很不稳定，所以我们要求他们重新做sampling，用straitified sampling 的方式来做。

15. 对negative data 建模时，因为我先采用当是boost decision tree的方法，boost decision tree 不需要分segmentation，所以我就把所有的training data 一起放入了。我选择了over all top 300 variables，train好了后发现ks score在每个seg上的表现不一，有的超过80，有的是零，这种情况下，这个model是完全不能用的。

16. XXX建议，我们先看对于每个seg，所有variable的ks是怎样分布的。我做了这么一张表，发现对于前300个overall top variable，他们的ks在前几个seg上表现很好，但是在其它的seg上表现得很不好。这意味这如果只选前300个overall top variables，会产生不平衡的问题。XXX建议我，选300个overall top ks variable的同时，再在每个seg里选前50个top ks variables放入model，用来保证平衡性。

17. 于此同时，我check了之前他们建立的模型对negative data的表现。我发现他们的模型对于每个seg，ks值都是相对稳定的，在20-30之间。我的模型确实问题很大。

18. 从16的建议出发，我对比了几个模型： 在seg 1， 2 上选前2000个feature，在1，2上训练；在seg 3， 4 上选前200个feature， 在3， 4上训练。。。。最终我发现，如果我在全部的population上每个seg选200个feature，并在全部的population上训练，得出的结果是最好的。

19. 09/11 我们得到了他们的最新model，但是似乎问题很大，最大的问题是，他们的evalation是在client data上，但并没有分client，这样的话，没法满足不同用户的需求。

20. 09/13 XXX 建议我们了解每个用户代码到底是什么意思。他是客户导向的，我们的model最终是要卖给客户的，所以了解我们的客户，哪些是重要的，哪些不是的，对于我们评估model的performance是非常critical的。

21. 09/18 我发现对于某些client，他们的client tag和 我们公司定义的tag 出入很大。用crosstab就可以看出来。

22.

### Behavior:

1. 做model的同时开始建立ppt，用来向manager汇报用。

2. 活儿要在保证准确性的前提下快快做，尤其是跟大老板一起干活的时候。

3. 不会就问。

4. 每次提交数据要有深度分析。

--------------
Reference:

https://blog.csdn.net/qunxingvip/article/details/50449376