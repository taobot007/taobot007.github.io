---
layout: post
title: pandas Dataframe深入理解
date: 2019-10-28
tag: Big Data and Machine Learning
---

一直在使用pandas的Series和Dataframe数据类型，但自感有些东西还没有深入了解。特记录下来，加深印象。

## Series对象：

Series对象是由索引index和值values组成的，一个index对应一个value。其中index是pandas中的Index对象。values是numpy中的数组对象。

### 创建
```
import pandas as pd
s1 = pd.Series([2,3,4,5], index=['a', 'b', 'c', 'd'])
print(s1)
结果：
a    2
b    3
c    4
d    5
dtype: int64

print(s1.index)
结果：
Index(['a', 'b', 'c', 'd'], dtype='object')

print(s1.values)
结果：
[2 3 4 5]
```

### 对Series对象进行索引

1: 利用index

```
print(s1['a'])
结果：
2

print(s1[['a','d']])
结果：
a    2
d    5
dtype: int64


print(s1['b':'d'])
结果（注意，切片索引保存最后一个值）：
b    3
c    4
d    5
dtype: int64
```

2: 利用下标索引

```
print(s1[0])
结果：
2

print(s1[[0,3]])
结果：
a    2
d    5
dtype: int64

print(s1[1:3])
结果（注意：这里和上面不同的是不保存最后一个值，与正常索引相同）：
b    3
c    4
dtype: int64
```

3: 利用布尔Series索引

```
s1 = pd.Series([2,3,4,5], index=['a', 'b', 'c', 'd']
print(s1 > 3)
结果（这是一个bool Series）：
a    False
b    False
c     True
d     True
dtype: bool

print(s1[s1 > 3])
结果（只需要把bool Series 传入Series就可以实现索引）：
c    4
d    5
dtype: int64
```

## DataFrame对象：

DataFrame对象是一个由行列组成的表。DataFrame中行由columns组成，列由index组成，它们都是Index对象。它的值还是numpy数组。

### 创建

```
data = {'name':['ming', 'hong', 'gang', 'tian'], 'age':[12, 13, 14, 20], 'score':[80.3, 88.2, 90, 99.9]}
df1 = pd.DataFrame(data)
```

### 对DataFrame进行索引

1: 使用columns的值对列进行索引

```
print(df1['name'])
结果：
0    ming
1    hong
2    gang
3    tian
Name: name, dtype: object

print(df1[['name','age']])
结果：
name  age
0  ming   12
1  hong   13
2  gang   14
3  tian   20
```

2: 切片或者布尔Series对行进行索引

```
print(df1[0:3])
使用切片进行选择，结果：
age  name  score
0   12  ming   80.3
1   13  hong   88.2
2   14  gang   90.0

print(df1[ df1['age'] > 13 ])
使用布尔类型Series进行索引，其实还是要求布尔Series和DataFrame有相同的index，结果：
age  name  score
2   14  gang   90.0
3   20  tian   99.9
```

3: 使用loc和iloc进行索引, 这比较有意思了。

本质上loc是用index和columns当中的值进行索引，而iloc是不理会index和columns当中的值的，永远都是用从0开始的下标进行索引。所以当你搞懂这句话的时候，下面的索引就会变得非常简单：

```
print(df1.loc[3])
结果：
name     hong
score    88.2
Name: 3, dtype: object

print(df1.loc[:,'age'])
结果：
1    12
3    13
4    14
5    20
Name: age, dtype: int64

print(df1.iloc[3])
结果：
age        20
name     tian
score    99.9
Name: 5, dtype: object

print(df1.iloc[:,1])
结果：
1    ming
3    hong
4    gang
5    tian
Name: name, dtype: object
```

## pandas中的索引

### 索引的有趣性质

1. 索引可以是重复的

```
df = pd.DataFrame([[1, 2], [3, 4]], columns = ['a','b'])
df2 = pd.DataFrame([[5, 6], [7, 8]], columns = ['a','b'])
df = df.append(df2)

   a  b
0  1  2
1  3  4
0  5  6
1  7  8

这时候，如果drop掉一个索引，就将删去与这个索引相关的所有行

df = df.drop(0)

   a  b
1  3  4
1  7  8
```

### 多级索引

多级索引（也称层次化索引）是pandas的重要功能，可以在Series、DataFrame对象上拥有2个以及2个以上的索引。实质上，单级索引对应Index对象,多级索引对应MultiIndex对象。

```
df=pd.DataFrame(np.random.randn(4),index=[list("aabb"),list('cdcd')])

df

a  c    0.945676
   d    1.240454
b  c    1.021960
   c    0.363063
dtype: float64

df['a']

c    0.945676
d    1.240454
dtype: float64


df.loc[('a', 'c'), :]      # 注意不能用df.loc[['a', 'c'], :],会把取index为a的所有都取出来

0   0.945676
dtype: float64
```


## 有关DataFrame的操作

### 基本操作

| 属性或者方法 | 描述 |
| ---- | ---- |
| T | 转置行和列 |
| axes | 返回一个列，行轴标签和列轴标签作为唯一的成员 |
| dtypes | 返回此对象中的数据类型(dtypes) |
| empty | 如果NDFrame完全为空[无项目]，则返回为True |
| ndim | 轴/数组维度大小 |
| shape | 返回表示DataFrame的维度的元组 |
| size | NDFrame中的元素数 |
| values | NDFrame的Numpy表示 |
| head() | 返回开头前n行 |
| tail() | 返回最后n行 |

### 描述性统计

| 函数 | 描述 |
| --- | --- |
| count() | 非空观测数量 |
| sum() | 所有值之和 |
| mean() | 所有值的平均值 |
| median() | 所有值的中位数 |
| mode() | 值的模值 |
| std()  | 值的标准偏差 |
| min() | 所有值中的最小值 |
| max() | 所有值中的最大值 |
| abs() | 绝对值 |
| prod() | 数组元素的乘积 |
| cumsum() | 累计总和 |
| cumprod() | 累计乘积 |
| describe() | 统计信息的汇总 |


### 使用自定义函数

要将自定义或其他库的函数应用于Pandas对象，有三个重要的方法，下面来讨论如何使用这些方法。使用适当的方法取决于函数是否期望在整个DataFrame，行或列或元素上进行操作。

* 表格函数应用：pipe()
* 行或列函数应用，作用在一维向量上：apply()
* 元素函数应用，作用于Dataframe中的每一个元素：applymap()
* apply的简化版本，作用于series上： map()

### pipe()

可以通过将函数和适当数量的参数作为管道参数来执行自定义操作。 因此，对整个DataFrame执行操作。

```
def adder(ele1,ele2):
    return ele1+ele2

df = pd.DataFrame([[5] * 3 for i in range(5)],columns=['col1','col2','col3'])
df.pipe(adder,2)

    col1 col2 col3
0    7    7     7
1    7    7     7
2    7    7     7
3    7    7     7
4    7    7     7

# 每个元素都被加上了2
```

### 行或列函数应用：apply()

可以使用apply()方法沿DataFrame或Panel的轴应用任意函数，它与描述性统计方法一样，采用可选的axis参数。 默认情况下，操作按列执行，将每列列为数组。

```
df = pd.DataFrame([[1, 2, 3], [4, 5, 6]],columns=['col1','col2','col3'])
df.apply(np.mean)

col1    2.5
col2    3.5
col3    4.5
dtype: float64

df.apply(np.mean, 1)

0    2.0
1    5.0
dtype: float64
```

### 元素函数应用： apply (for Series) applymap() (for DataFrame)

这两个是非常重要的。

```
df = pd.DataFrame([[1, 2, 3], [4, 5, 6]],columns=['col1','col2','col3'])

df['m'] = df['col1'].apply(lambda x: x*100)

df

    col1  col2  col3   m
0    1     2    3    100
1    4     5    6    400

df1 = df.applymap(lambda x: x*100)

df1

    col1    col2    col3     m
0    100    200     300    10000
1    400    500     600    40000
```

### map: 可以作用于另一个series

```
df = pd.DataFrame([[1, 2, 3], [4, 5, 6]],columns=['col1','col2','col3'])

df['m'] = df['col1'].apply(lambda x: x*100)

df

    col1  col2  col3   m
0    1     2    3    100
1    4     5    6    400

df['k'] = df[['col1', 'col2']].apply(tupel, axis=1).map(df.set_index(['col1', 'col2'])['m'])

df
    col1  col2  col3   m    k
0    1     2    3    100   100
1    4     5    6    400   400

```

## Sort

一共有两种： 按index排序和按value来排序

```
# 按index排序

unsorted_df = pd.DataFrame(np.random.randn(10,2),index=[1,4,6,2,3,5,9,8,0,7],columns = ['col2','col1'])
sorted_df=unsorted_df.sort_index(axis=1)

# 按value排序

unsorted_df = pd.DataFrame({'col1':[2,1,1,1],'col2':[1,3,2,4]})
sorted_df = unsorted_df.sort_values(by='col1')
```

## 缺失数据

1: 检查缺失值

为了更容易地检测缺失值(以及跨越不同的数组dtype)，Pandas提供了isnull()和notnull()函数，它们也是Series和DataFrame对象的方法.

```
df = pd.DataFrame(np.random.randn(5, 3), index=['a', 'c', 'e', 'f',
'h'],columns=['one', 'two', 'three'])

df = df.reindex(['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'])

print (df['one'].isnull())

a    False
b     True
c    False
d     True
e    False
f    False
g     True
h    False
Name: one, dtype: bool

```

2: 缺少数据的计算

    * 在求和数据时，NA将被视为0
    * 如果数据全部是NA，那么结果将是NA

3: 清理/填充缺少数据

Pandas提供了各种方法来清除缺失的值。fillna()函数可以通过几种方法用非空数据“填充”NA值.

    1) 用标量值替换NaN
    ```
    df = pd.DataFrame(np.random.randn(3, 3), index=['a', 'c', 'e'],columns=['one','two', 'three'])
    df = df.reindex(['a', 'b', 'c'])
    print (df)
    print ("NaN replaced with '0':")
    print (df.fillna(0))

            one       two     three
    a -0.479425 -1.711840 -1.453384
    b       NaN       NaN       NaN
    c -0.733606 -0.813315  0.476788
    NaN replaced with '0':
            one       two     three
    a -0.479425 -1.711840 -1.453384
    b  0.000000  0.000000  0.000000
    c -0.733606 -0.813315  0.476788

    ```
    2) 填写NA前进和后退
    使用前值或后值来填充
    | 方法 | 动作 |
    | --- | --- |
    | pad/fill | 填充方法向前 |
    | bfill/backfill | 填充方法向后 |

    ```
    import pandas as pd
    import numpy as np

    df = pd.DataFrame(np.random.randn(5, 3), index=['a', 'c', 'e', 'f',
    'h'],columns=['one', 'two', 'three'])
    df = df.reindex(['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'])

    print (df.fillna(method='pad'))

        one       two     three
    a  0.614938 -0.452498 -2.113057
    b  0.614938 -0.452498 -2.113057
    c -0.118390  1.333962 -0.037907
    d -0.118390  1.333962 -0.037907
    e  0.699733  0.502142 -0.243700
    f  0.544225 -0.923116 -1.123218
    g  0.544225 -0.923116 -1.123218
    h -0.669783  1.187865  1.112835
    ```

    3) 还可以用dropna()来删除缺失值的行或者列。 dropna()默认为drop掉行。

## 分组（groupby)

在许多情况下，我们将数据分成多个集合，并在每个子集上应用一些函数。在应用函数中，可以执行以下操作 -

* 聚合 - 计算汇总统计
* 转换 - 执行一些特定于组的操作
* 过滤 - 在某些情况下丢弃数据

### 查看分组

单列分组：

```
import pandas as pd
ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],           'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)

print (df.groupby('Team').groups)

{
'Devils': Int64Index([2, 3], dtype='int64'),
'Kings': Int64Index([4, 6, 7], dtype='int64'),
'Riders': Int64Index([0, 1, 8, 11], dtype='int64'),
'Royals': Int64Index([9, 10], dtype='int64'),
'kings': Int64Index([5], dtype='int64')
}
```

多列分组：

```
import pandas as pd
ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)
print (df.groupby(['Team','Year']).groups)

{
('Devils', 2014): Int64Index([2], dtype='int64'),
('Devils', 2015): Int64Index([3], dtype='int64'),
('Kings', 2014): Int64Index([4], dtype='int64'),
('Kings', 2016): Int64Index([6], dtype='int64'),
('Kings', 2017): Int64Index([7], dtype='int64'),
('Riders', 2014): Int64Index([0], dtype='int64'),
('Riders', 2015): Int64Index([1], dtype='int64'),
('Riders', 2016): Int64Index([8], dtype='int64'),
('Riders', 2017): Int64Index([11], dtype='int64'),
('Royals', 2014): Int64Index([9], dtype='int64'),
('Royals', 2015): Int64Index([10], dtype='int64'),
('kings', 2015): Int64Index([5], dtype='int64')
}
```

### 迭代遍历分组

```
import pandas as pd

ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)

grouped = df.groupby('Year')

for name,group in grouped:
    print (name)
    print (group)

2014
   Points  Rank    Team  Year
0     876     1  Riders  2014
2     863     2  Devils  2014
4     741     3   Kings  2014
9     701     4  Royals  2014
2015
    Points  Rank    Team  Year
1      789     2  Riders  2015
3      673     3  Devils  2015
5      812     4   kings  2015
10     804     1  Royals  2015
2016
   Points  Rank    Team  Year
6     756     1   Kings  2016
8     694     2  Riders  2016
2017
    Points  Rank    Team  Year
7      788     1   Kings  2017
11     690     2  Riders  2017
```

### 聚合操作 （最重要）

聚合函数为每个组返回单个聚合值。当创建了分组(group by)对象，就可以对分组数据执行多个聚合操作。

一个比较常用的是通过聚合或等效的agg方法聚合。

```
import pandas as pd
import numpy as np

ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)

grouped = df.groupby('Year')
print (grouped['Points'].agg(np.mean))

Year
2014    795.25
2015    769.50
2016    725.00
2017    739.00
Name: Points, dtype: float64

```

一次应用多个聚合：

```
import pandas as pd
import numpy as np

ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)

grouped = df.groupby('Team')
agg = grouped['Points'].agg([np.sum, np.mean, np.std])
print (agg)

        sum        mean         std
Team                                
Devils  1536  768.000000  134.350288
Kings   2285  761.666667   24.006943
Riders  3049  762.250000   88.567771
Royals  1505  752.500000   72.831998
kings    812  812.000000         NaN
```

### 转换

分组或列上的转换返回索引大小与被分组的索引相同的对象。因此，转换应该返回与组块大小相同的结果。

```
import pandas as pd
import numpy as np

ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)

grouped = df.groupby('Team')
score = lambda x: (x - x.mean()) / x.std()*10     # 这个x似乎指的是group里的每一列的元素
print (grouped.transform(score))

       Points       Rank       Year
0   12.843272 -15.000000 -11.618950
1    3.020286   5.000000  -3.872983
2    7.071068  -7.071068  -7.071068
3   -7.071068   7.071068   7.071068
4   -8.608621  11.547005 -10.910895
5         NaN        NaN        NaN
6   -2.360428  -5.773503   2.182179
7   10.969049  -5.773503   8.728716
8   -7.705963   5.000000   3.872983
9   -7.071068   7.071068  -7.071068
10   7.071068  -7.071068   7.071068
11  -8.157595   5.000000  11.618950

```

### 过滤

过滤根据定义的标准过滤数据并返回数据的子集。filter()函数用于过滤数据。

```
import pandas as pd
import numpy as np
ipl_data = {'Team': ['Riders', 'Riders', 'Devils', 'Devils', 'Kings',
         'kings', 'Kings', 'Kings', 'Riders', 'Royals', 'Royals', 'Riders'],
         'Rank': [1, 2, 2, 3, 3,4 ,1 ,1,2 , 4,1,2],
         'Year': [2014,2015,2014,2015,2014,2015,2016,2017,2016,2014,2015,2017],
         'Points':[876,789,863,673,741,812,756,788,694,701,804,690]}
df = pd.DataFrame(ipl_data)
filter = df.groupby('Team').filter(lambda x: len(x) >= 3)   # 这里的x指的是Team中的一个group

print (filter)

    Points  Rank    Team  Year
0      876     1  Riders  2014
1      789     2  Riders  2015
4      741     3   Kings  2014
6      756     1   Kings  2016
7      788     1   Kings  2017
8      694     2  Riders  2016
11     690     2  Riders  2017
```







