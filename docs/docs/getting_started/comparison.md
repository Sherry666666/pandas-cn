# 与其他工具比较

## 与R/R库的比较

由于``pandas``旨在为人们提供可以替代[R](http://www.r-project.org/)的大量数据操作和分析的功能，因此本章节会提供较为详细的[R语言](http://en.wikipedia.org/wiki/R_(programming_language))的介绍以及与相关的许多第三方库的对比说明，比如我们的``pandas``库。在与R和CRAN库的比较中，我们关注以下事项：

- **功能/灵活性**：每个工具可以/不​​可以做什么
- **性能**：操作速度有多快。硬性数字/基准是优选的
- **易于使用**：一种工具更容易/更难使用（您可能需要对此进行判断，并进行并排代码比较）

此页面还为这些R包的用户提供了一些翻译指南。

要将``DataFrame``对象从 ``pandas`` 转化为到 R 的数据类型，有一个选择是采用HDF5文件，请参阅[外部兼容性](https://pandas.pydata.org/pandas-docs/stable/../user_guide/io.html#io-external-compatibility)示例。

### 快速参考

我们将从快速参考指南开始，将[dplyr](https://cran.r-project.org/package=dplyr)与pandas等效的一些常见R操作配对。

#### 查询、过滤、采样

R | Pandas
---|---
dim(df) | df.shape
head(df) | df.head()
slice(df, 1:10) | df.iloc[:9]
filter(df, col1 == 1, col2 == 1) | df.query('col1 == 1 & col2 == 1')
df[df$col1 == 1 & df$col2 == 1,] | df[(df.col1 == 1) & (df.col2 == 1)]
select(df, col1, col2) | df[['col1', 'col2']]
select(df, col1:col3) | df.loc[:, 'col1':'col3']
select(df, -(col1:col3)) | df.drop(cols_to_drop, axis=1)但是看[[1]](#select-range)
distinct(select(df, col1)) | df[['col1']].drop_duplicates()
distinct(select(df, col1, col2)) | df[['col1', 'col2']].drop_duplicates()
sample_n(df, 10) | df.sample(n=10)
sample_frac(df, 0.01) | df.sample(frac=0.01)

::: tip Note

R表示列的子集 ``(select(df，col1：col3)`` 的缩写更接近 Pandas 的写法，如果您有列的列表，例如 ``df[cols[1:3]`` 或 ``df.drop(cols[1:3])``，按列名执行此操作可能会引起混乱。

::: 

#### 排序

R | Pandas
---|---
arrange(df, col1, col2) | df.sort_values(['col1', 'col2'])
arrange(df, desc(col1)) | df.sort_values('col1', ascending=False)

#### 变换

R | Pandas
---|---
select(df, col_one = col1) | df.rename(columns={'col1': 'col_one'})['col_one']
rename(df, col_one = col1) | df.rename(columns={'col1': 'col_one'})
mutate(df, c=a-b) | df.assign(c=df.a-df.b)

#### 分组和组合

R | Pandas
---|---
summary(df) | df.describe()
gdf <- group_by(df, col1) | gdf = df.groupby('col1')
summarise(gdf, avg=mean(col1, na.rm=TRUE)) | df.groupby('col1').agg({'col1': 'mean'})
summarise(gdf, total=sum(col1)) | df.groupby('col1').sum()

### 基本的R用法

#### 用R``c``方法来进行切片操作

R使您可以轻松地按名称访问列（``data.frame``）

``` r
df <- data.frame(a=rnorm(5), b=rnorm(5), c=rnorm(5), d=rnorm(5), e=rnorm(5))
df[, c("a", "c", "e")]
```

或整数位置

``` r
df <- data.frame(matrix(rnorm(1000), ncol=100))
df[, c(1:10, 25:30, 40, 50:100)]
```

按名称选择多个``pandas``的列非常简单

``` python
In [1]: df = pd.DataFrame(np.random.randn(10, 3), columns=list('abc'))

In [2]: df[['a', 'c']]
Out[2]: 
          a         c
0  0.469112 -1.509059
1 -1.135632 -0.173215
2  0.119209 -0.861849
3 -2.104569  1.071804
4  0.721555 -1.039575
5  0.271860  0.567020
6  0.276232 -0.673690
7  0.113648  0.524988
8  0.404705 -1.715002
9 -1.039268 -1.157892

In [3]: df.loc[:, ['a', 'c']]
Out[3]: 
          a         c
0  0.469112 -1.509059
1 -1.135632 -0.173215
2  0.119209 -0.861849
3 -2.104569  1.071804
4  0.721555 -1.039575
5  0.271860  0.567020
6  0.276232 -0.673690
7  0.113648  0.524988
8  0.404705 -1.715002
9 -1.039268 -1.157892
```

通过整数位置选择多个不连续的列可以通过``iloc``索引器属性和 ``numpy.r_`` 的组合来实现。

``` python
In [4]: named = list('abcdefg')

In [5]: n = 30

In [6]: columns = named + np.arange(len(named), n).tolist()

In [7]: df = pd.DataFrame(np.random.randn(n, n), columns=columns)

In [8]: df.iloc[:, np.r_[:10, 24:30]]
Out[8]: 
           a         b         c         d         e         f         g         7         8         9        24        25        26        27        28        29
0  -1.344312  0.844885  1.075770 -0.109050  1.643563 -1.469388  0.357021 -0.674600 -1.776904 -0.968914 -1.170299 -0.226169  0.410835  0.813850  0.132003 -0.827317
1  -0.076467 -1.187678  1.130127 -1.436737 -1.413681  1.607920  1.024180  0.569605  0.875906 -2.211372  0.959726 -1.110336 -0.619976  0.149748 -0.732339  0.687738
2   0.176444  0.403310 -0.154951  0.301624 -2.179861 -1.369849 -0.954208  1.462696 -1.743161 -0.826591  0.084844  0.432390  1.519970 -0.493662  0.600178  0.274230
3   0.132885 -0.023688  2.410179  1.450520  0.206053 -0.251905 -2.213588  1.063327  1.266143  0.299368 -2.484478 -0.281461  0.030711  0.109121  1.126203 -0.977349
4   1.474071 -0.064034 -1.282782  0.781836 -1.071357  0.441153  2.353925  0.583787  0.221471 -0.744471 -1.197071 -1.066969 -0.303421 -0.858447  0.306996 -0.028665
..       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...
25  1.492125 -0.068190  0.681456  1.221829 -0.434352  1.204815 -0.195612  1.251683 -1.040389 -0.796211  1.944517  0.042344 -0.307904  0.428572  0.880609  0.487645
26  0.725238  0.624607 -0.141185 -0.143948 -0.328162  2.095086 -0.608888 -0.926422  1.872601 -2.513465 -0.846188  1.190624  0.778507  1.008500  1.424017  0.717110
27  1.262419  1.950057  0.301038 -0.933858  0.814946  0.181439 -0.110015 -2.364638 -1.584814  0.307941 -1.341814  0.334281 -0.162227  1.007824  2.826008  1.458383
28 -1.585746 -0.899734  0.921494 -0.211762 -0.059182  0.058308  0.915377 -0.696321  0.150664 -3.060395  0.403620 -0.026602 -0.240481  0.577223 -1.088417  0.326687
29 -0.986248  0.169729 -1.158091  1.019673  0.646039  0.917399 -0.010435  0.366366  0.922729  0.869610 -1.209247 -0.671466  0.332872 -2.013086 -1.602549  0.333109

[30 rows x 16 columns]
```

#### ``aggregate``

在R中，您可能希望将数据分成几个子集，并计算每个子集的平均值。使用名为``df``的data.frame并将其分成组``by1``和``by2``：

``` r
df <- data.frame(
  v1 = c(1,3,5,7,8,3,5,NA,4,5,7,9),
  v2 = c(11,33,55,77,88,33,55,NA,44,55,77,99),
  by1 = c("red", "blue", 1, 2, NA, "big", 1, 2, "red", 1, NA, 12),
  by2 = c("wet", "dry", 99, 95, NA, "damp", 95, 99, "red", 99, NA, NA))
aggregate(x=df[, c("v1", "v2")], by=list(mydf2$by1, mydf2$by2), FUN = mean)
```

该[``groupby()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby)方法类似于基本R的 ``aggregate``
函数。

``` python
In [9]: df = pd.DataFrame(
   ...:     {'v1': [1, 3, 5, 7, 8, 3, 5, np.nan, 4, 5, 7, 9],
   ...:      'v2': [11, 33, 55, 77, 88, 33, 55, np.nan, 44, 55, 77, 99],
   ...:      'by1': ["red", "blue", 1, 2, np.nan, "big", 1, 2, "red", 1, np.nan, 12],
   ...:      'by2': ["wet", "dry", 99, 95, np.nan, "damp", 95, 99, "red", 99, np.nan,
   ...:              np.nan]})
   ...: 

In [10]: g = df.groupby(['by1', 'by2'])

In [11]: g[['v1', 'v2']].mean()
Out[11]: 
            v1    v2
by1  by2            
1    95    5.0  55.0
     99    5.0  55.0
2    95    7.0  77.0
     99    NaN   NaN
big  damp  3.0  33.0
blue dry   3.0  33.0
red  red   4.0  44.0
     wet   1.0  11.0
```

有关更多详细信息和示例，请参阅[groupby文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/groupby.html#groupby-split)。

#### ``match``/ ``%in%``

在R中选择数据的常用方法是使用``%in%``使用该函数定义的数据``match``。运算符``%in%``用于返回指示是否存在匹配的逻辑向量：

``` r
s <- 0:4
s %in% c(2,4)
```

该[``isin()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.isin.html#pandas.DataFrame.isin)方法类似于R ``%in%``运算符：

``` python
In [12]: s = pd.Series(np.arange(5), dtype=np.float32)

In [13]: s.isin([2, 4])
Out[13]: 
0    False
1    False
2     True
3    False
4     True
dtype: bool
```

该``match``函数返回其第二个参数匹配位置的向量：

``` r
s <- 0:4
match(s, c(2,4))
```

有关更多详细信息和示例，请参阅[重塑文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/indexing.html#indexing-basics-indexing-isin)。

#### ``tapply``

``tapply``类似于``aggregate``，但数据可以是一个参差不齐的数组，因为子类大小可能是不规则的。使用调用的data.frame
 ``baseball``，并根据数组检索信息``team``：

``` r
baseball <-
  data.frame(team = gl(5, 5,
             labels = paste("Team", LETTERS[1:5])),
             player = sample(letters, 25),
             batting.average = runif(25, .200, .400))

tapply(baseball$batting.average, baseball.example$team,
       max)
```

在``pandas``我们可以使用[``pivot_table()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.pivot_table.html#pandas.pivot_table)方法来处理这个：

``` python
In [14]: import random

In [15]: import string

In [16]: baseball = pd.DataFrame(
   ....:     {'team': ["team %d" % (x + 1) for x in range(5)] * 5,
   ....:      'player': random.sample(list(string.ascii_lowercase), 25),
   ....:      'batting avg': np.random.uniform(.200, .400, 25)})
   ....: 

In [17]: baseball.pivot_table(values='batting avg', columns='team', aggfunc=np.max)
Out[17]: 
team           team 1    team 2    team 3    team 4    team 5
batting avg  0.352134  0.295327  0.397191  0.394457  0.396194
```

有关更多详细信息和示例，请参阅[重塑文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/reshaping.html#reshaping-pivot)。

#### ``subset``

该[``query()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.query.html#pandas.DataFrame.query)方法类似于基本R ``subset``
函数。在R中，您可能希望获取``data.frame``一列的值小于另一列的值的行：

``` r
df <- data.frame(a=rnorm(10), b=rnorm(10))
subset(df, a <= b)
df[df$a <= df$b,]  ## note the comma
```

在``pandas``，有几种方法可以执行子集化。您可以使用
 [``query()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.query.html#pandas.DataFrame.query)或传递表达式，就像它是索引/切片以及标准布尔索引一样：

``` python
In [18]: df = pd.DataFrame({'a': np.random.randn(10), 'b': np.random.randn(10)})

In [19]: df.query('a <= b')
Out[19]: 
          a         b
1  0.174950  0.552887
2 -0.023167  0.148084
3 -0.495291 -0.300218
4 -0.860736  0.197378
5 -1.134146  1.720780
7 -0.290098  0.083515
8  0.238636  0.946550

In [20]: df[df.a <= df.b]
Out[20]: 
          a         b
1  0.174950  0.552887
2 -0.023167  0.148084
3 -0.495291 -0.300218
4 -0.860736  0.197378
5 -1.134146  1.720780
7 -0.290098  0.083515
8  0.238636  0.946550

In [21]: df.loc[df.a <= df.b]
Out[21]: 
          a         b
1  0.174950  0.552887
2 -0.023167  0.148084
3 -0.495291 -0.300218
4 -0.860736  0.197378
5 -1.134146  1.720780
7 -0.290098  0.083515
8  0.238636  0.946550
```

有关更多详细信息和示例，请参阅[查询文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/indexing.html#indexing-query)。

#### ``with``

使用``df``带有列的R中调用的data.frame的表达式``a``，
 ``b``将使用``with``如下方式进行求值：

``` r
df <- data.frame(a=rnorm(10), b=rnorm(10))
with(df, a + b)
df$a + df$b  ## same as the previous expression
```

在``pandas``等效表达式中，使用该
 [``eval()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.eval.html#pandas.DataFrame.eval)方法将是：

``` python
In [22]: df = pd.DataFrame({'a': np.random.randn(10), 'b': np.random.randn(10)})

In [23]: df.eval('a + b')
Out[23]: 
0   -0.091430
1   -2.483890
2   -0.252728
3   -0.626444
4   -0.261740
5    2.149503
6   -0.332214
7    0.799331
8   -2.377245
9    2.104677
dtype: float64

In [24]: df.a + df.b  ## same as the previous expression
Out[24]: 
0   -0.091430
1   -2.483890
2   -0.252728
3   -0.626444
4   -0.261740
5    2.149503
6   -0.332214
7    0.799331
8   -2.377245
9    2.104677
dtype: float64
```

在某些情况下，[``eval()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.eval.html#pandas.DataFrame.eval)将比纯Python中的评估快得多。有关更多详细信息和示例，请参阅[eval文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/enhancingperf.html#enhancingperf-eval)。

### plyr 

``plyr``是用于数据分析的拆分应用组合策略的R库。这些函数围绕R，``a``
for ``arrays``，``l``for ``lists``和``d``for中的三个数据结构``data.frame``。下表显示了如何在Python中映射这些数据结构。

R | Python
---|---
array | list
lists | 字典（dist）或对象列表（list of objects）
data.frame | dataframe

#### ``ddply``

在R中使用名为``df``的data.frame的表达式，比如您有一个希望按``月``汇总``x``的需求：

``` r
require(plyr)
df <- data.frame(
  x = runif(120, 1, 168),
  y = runif(120, 7, 334),
  z = runif(120, 1.7, 20.7),
  month = rep(c(5,6,7,8),30),
  week = sample(1:4, 120, TRUE)
)

ddply(df, .(month, week), summarize,
      mean = round(mean(x), 2),
      sd = round(sd(x), 2))
```

在``pandas``等效表达式中，使用该
 [``groupby()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby)方法将是：

``` python
In [25]: df = pd.DataFrame({'x': np.random.uniform(1., 168., 120),
   ....:                    'y': np.random.uniform(7., 334., 120),
   ....:                    'z': np.random.uniform(1.7, 20.7, 120),
   ....:                    'month': [5, 6, 7, 8] * 30,
   ....:                    'week': np.random.randint(1, 4, 120)})
   ....: 

In [26]: grouped = df.groupby(['month', 'week'])

In [27]: grouped['x'].agg([np.mean, np.std])
Out[27]: 
                  mean        std
month week                       
5     1      63.653367  40.601965
      2      78.126605  53.342400
      3      92.091886  57.630110
6     1      81.747070  54.339218
      2      70.971205  54.687287
      3     100.968344  54.010081
7     1      61.576332  38.844274
      2      61.733510  48.209013
      3      71.688795  37.595638
8     1      62.741922  34.618153
      2      91.774627  49.790202
      3      73.936856  60.773900
```

有关更多详细信息和示例，请参阅[groupby文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/groupby.html#groupby-aggregate)。

### 重塑/ reshape2 

#### ``melt.array``

使用``a``在R中调用的3维数组的表达式，您希望将其融合到data.frame中：

``` r
a <- array(c(1:23, NA), c(2,3,4))
data.frame(melt(a))
```

在Python中，既然``a``是一个列表，你可以简单地使用列表理解。

``` python
In [28]: a = np.array(list(range(1, 24)) + [np.NAN]).reshape(2, 3, 4)

In [29]: pd.DataFrame([tuple(list(x) + [val]) for x, val in np.ndenumerate(a)])
Out[29]: 
    0  1  2     3
0   0  0  0   1.0
1   0  0  1   2.0
2   0  0  2   3.0
3   0  0  3   4.0
4   0  1  0   5.0
.. .. .. ..   ...
19  1  1  3  20.0
20  1  2  0  21.0
21  1  2  1  22.0
22  1  2  2  23.0
23  1  2  3   NaN

[24 rows x 4 columns]
```

#### ``melt.list``

使用``a``R中调用的列表的表达式，您希望将其融合到data.frame中：

``` r
a <- as.list(c(1:4, NA))
data.frame(melt(a))
```

在Python中，此列表将是元组列表，因此
 [``DataFrame()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.html#pandas.DataFrame)方法会根据需要将其转换为数据帧。

``` python
In [30]: a = list(enumerate(list(range(1, 5)) + [np.NAN]))

In [31]: pd.DataFrame(a)
Out[31]: 
   0    1
0  0  1.0
1  1  2.0
2  2  3.0
3  3  4.0
4  4  NaN
```

有关更多详细信息和示例，请参阅[“进入数据结构”文档](https://pandas.pydata.org/pandas-docs/stable/dsintro.html#dsintro)。

#### ``melt.data.frame``

使用``cheese``在R中调用的data.frame的表达式，您要在其中重新整形data.frame：

``` r
cheese <- data.frame(
  first = c('John', 'Mary'),
  last = c('Doe', 'Bo'),
  height = c(5.5, 6.0),
  weight = c(130, 150)
)
melt(cheese, id=c("first", "last"))
```

在Python中，该[``melt()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.melt.html#pandas.melt)方法是R等价物：

``` python
In [32]: cheese = pd.DataFrame({'first': ['John', 'Mary'],
   ....:                        'last': ['Doe', 'Bo'],
   ....:                        'height': [5.5, 6.0],
   ....:                        'weight': [130, 150]})
   ....: 

In [33]: pd.melt(cheese, id_vars=['first', 'last'])
Out[33]: 
  first last variable  value
0  John  Doe   height    5.5
1  Mary   Bo   height    6.0
2  John  Doe   weight  130.0
3  Mary   Bo   weight  150.0

In [34]: cheese.set_index(['first', 'last']).stack()  ## alternative way
Out[34]: 
first  last        
John   Doe   height      5.5
             weight    130.0
Mary   Bo    height      6.0
             weight    150.0
dtype: float64
```

有关更多详细信息和示例，请参阅[重塑文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/reshaping.html#reshaping-melt)。

#### ``cast``

在R中``acast``是一个表达式，它使用``df``在R中调用的data.frame 来转换为更高维的数组：

``` r
df <- data.frame(
  x = runif(12, 1, 168),
  y = runif(12, 7, 334),
  z = runif(12, 1.7, 20.7),
  month = rep(c(5,6,7),4),
  week = rep(c(1,2), 6)
)

mdf <- melt(df, id=c("month", "week"))
acast(mdf, week ~ month ~ variable, mean)
```

在Python中，最好的方法是使用[``pivot_table()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.pivot_table.html#pandas.pivot_table)：

``` python
In [35]: df = pd.DataFrame({'x': np.random.uniform(1., 168., 12),
   ....:                    'y': np.random.uniform(7., 334., 12),
   ....:                    'z': np.random.uniform(1.7, 20.7, 12),
   ....:                    'month': [5, 6, 7] * 4,
   ....:                    'week': [1, 2] * 6})
   ....: 

In [36]: mdf = pd.melt(df, id_vars=['month', 'week'])

In [37]: pd.pivot_table(mdf, values='value', index=['variable', 'week'],
   ....:                columns=['month'], aggfunc=np.mean)
   ....: 
Out[37]: 
month                  5           6           7
variable week                                   
x        1     93.888747   98.762034   55.219673
         2     94.391427   38.112932   83.942781
y        1     94.306912  279.454811  227.840449
         2     87.392662  193.028166  173.899260
z        1     11.016009   10.079307   16.170549
         2      8.476111   17.638509   19.003494
```

类似地``dcast``，使用``df``R中调用的data.frame 来基于``Animal``和聚合信息``FeedType``：

``` r
df <- data.frame(
  Animal = c('Animal1', 'Animal2', 'Animal3', 'Animal2', 'Animal1',
             'Animal2', 'Animal3'),
  FeedType = c('A', 'B', 'A', 'A', 'B', 'B', 'A'),
  Amount = c(10, 7, 4, 2, 5, 6, 2)
)

dcast(df, Animal ~ FeedType, sum, fill=NaN)
## Alternative method using base R
with(df, tapply(Amount, list(Animal, FeedType), sum))
```

Python可以通过两种不同的方式处理它。首先，类似于上面使用[``pivot_table()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.pivot_table.html#pandas.pivot_table)：

``` python
In [38]: df = pd.DataFrame({
   ....:     'Animal': ['Animal1', 'Animal2', 'Animal3', 'Animal2', 'Animal1',
   ....:                'Animal2', 'Animal3'],
   ....:     'FeedType': ['A', 'B', 'A', 'A', 'B', 'B', 'A'],
   ....:     'Amount': [10, 7, 4, 2, 5, 6, 2],
   ....: })
   ....: 

In [39]: df.pivot_table(values='Amount', index='Animal', columns='FeedType',
   ....:                aggfunc='sum')
   ....: 
Out[39]: 
FeedType     A     B
Animal              
Animal1   10.0   5.0
Animal2    2.0  13.0
Animal3    6.0   NaN
```

第二种方法是使用该[``groupby()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby)方法：

``` python
In [40]: df.groupby(['Animal', 'FeedType'])['Amount'].sum()
Out[40]: 
Animal   FeedType
Animal1  A           10
         B            5
Animal2  A            2
         B           13
Animal3  A            6
Name: Amount, dtype: int64
```

有关更多详细信息和示例，请参阅[重新整形文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/reshaping.html#reshaping-pivot)或[groupby文档](https://pandas.pydata.org/pandas-docs/stable/../user_guide/groupby.html#groupby-split)。

#### ``factor``

pandas具有分类数据的数据类型。

``` r
cut(c(1,2,3,4,5,6), 3)
factor(c(1,2,3,2,2,3))
```

在Pandas，这是完成与``pd.cut``和``astype("category")``：

``` python
In [41]: pd.cut(pd.Series([1, 2, 3, 4, 5, 6]), 3)
Out[41]: 
0    (0.995, 2.667]
1    (0.995, 2.667]
2    (2.667, 4.333]
3    (2.667, 4.333]
4      (4.333, 6.0]
5      (4.333, 6.0]
dtype: category
Categories (3, interval[float64]): [(0.995, 2.667] < (2.667, 4.333] < (4.333, 6.0]]

In [42]: pd.Series([1, 2, 3, 2, 2, 3]).astype("category")
Out[42]: 
0    1
1    2
2    3
3    2
4    2
5    3
dtype: category
Categories (3, int64): [1, 2, 3]
```

有关更多详细信息和示例，请参阅[分类介绍](https://pandas.pydata.org/pandas-docs/stable/../user_guide/categorical.html#categorical)和
 [API文档](https://pandas.pydata.org/pandas-docs/stable/../reference/arrays.html#api-arrays-categorical)。还有一个关于[R因子差异](https://pandas.pydata.org/pandas-docs/stable/../user_guide/categorical.html#categorical-rfactor)的文档
 。

## 与SQL比较

由于许多潜在的pandas用户对[SQL](https://en.wikipedia.org/wiki/SQL)有一定的了解
 ，因此本页面旨在提供一些使用pandas如何执行各种SQL操作的示例。

如果您是 pandas 的新手，您可能需要先阅读[10分钟到 pandas ，](https://pandas.pydata.org/pandas-docs/stable/10min.html#min) 
以熟悉图书馆。

按照惯例，我们按如下方式导入pandas和NumPy：

``` python
In [1]: import pandas as pd

In [2]: import numpy as np
```

大多数示例将使用``tips``pandas测试中找到的数据集。我们将数据读入名为*tips*的DataFrame中，并假设我们有一个具有相同名称和结构的数据库表。

``` python
In [3]: url = ('https://raw.github.com/pandas-dev'
   ...:        '/pandas/master/pandas/tests/data/tips.csv')
   ...: 

In [4]: tips = pd.read_csv(url)

In [5]: tips.head()
Out[5]: 
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

### SELECT 

在SQL中，使用您要选择的以逗号分隔的列列表（或``*``
选择所有列）来完成选择：

``` sql
SELECT total_bill, tip, smoker, time
FROM tips
LIMIT 5;
```

使用pandas，通过将列名列表传递给DataFrame来完成列选择：

``` python
In [6]: tips[['total_bill', 'tip', 'smoker', 'time']].head(5)
Out[6]: 
   total_bill   tip smoker    time
0       16.99  1.01     No  Dinner
1       10.34  1.66     No  Dinner
2       21.01  3.50     No  Dinner
3       23.68  3.31     No  Dinner
4       24.59  3.61     No  Dinner
```

在没有列名列表的情况下调用DataFrame将显示所有列（类似于SQL
 ``*``）。

### 哪里

SQL中的过滤是通过WHERE子句完成的。

``` sql
SELECT *
FROM tips
WHERE time = 'Dinner'
LIMIT 5;
```

DataFrame可以通过多种方式进行过滤; 最直观的是使用
 [布尔索引](https://pandas.pydata.org/pandas-docs/stable/indexing.html#boolean-indexing)。

``` python
In [7]: tips[tips['time'] == 'Dinner'].head(5)
Out[7]: 
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

上面的语句只是将一个``Series``True / False对象传递给DataFrame，返回所有行为True。

``` python
In [8]: is_dinner = tips['time'] == 'Dinner'

In [9]: is_dinner.value_counts()
Out[9]: 
True     176
False     68
Name: time, dtype: int64

In [10]: tips[is_dinner].head(5)
Out[10]: 
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

就像SQL的OR和AND一样，可以使用|将多个条件传递给DataFrame （OR）和＆（AND）。

``` sql
-- tips of more than $5.00 at Dinner meals
SELECT *
FROM tips
WHERE time = 'Dinner' AND tip > 5.00;
```

``` python
## tips of more than $5.00 at Dinner meals
In [11]: tips[(tips['time'] == 'Dinner') & (tips['tip'] > 5.00)]
Out[11]: 
     total_bill    tip     sex smoker  day    time  size
23        39.42   7.58    Male     No  Sat  Dinner     4
44        30.40   5.60    Male     No  Sun  Dinner     4
47        32.40   6.00    Male     No  Sun  Dinner     4
52        34.81   5.20  Female     No  Sun  Dinner     4
59        48.27   6.73    Male     No  Sat  Dinner     4
116       29.93   5.07    Male     No  Sun  Dinner     4
155       29.85   5.14  Female     No  Sun  Dinner     5
170       50.81  10.00    Male    Yes  Sat  Dinner     3
172        7.25   5.15    Male    Yes  Sun  Dinner     2
181       23.33   5.65    Male    Yes  Sun  Dinner     2
183       23.17   6.50    Male    Yes  Sun  Dinner     4
211       25.89   5.16    Male    Yes  Sat  Dinner     4
212       48.33   9.00    Male     No  Sat  Dinner     4
214       28.17   6.50  Female    Yes  Sat  Dinner     3
239       29.03   5.92    Male     No  Sat  Dinner     3
```

``` sql
-- tips by parties of at least 5 diners OR bill total was more than $45
SELECT *
FROM tips
WHERE size >= 5 OR total_bill > 45;
```

``` python
## tips by parties of at least 5 diners OR bill total was more than $45
In [12]: tips[(tips['size'] >= 5) | (tips['total_bill'] > 45)]
Out[12]: 
     total_bill    tip     sex smoker   day    time  size
59        48.27   6.73    Male     No   Sat  Dinner     4
125       29.80   4.20  Female     No  Thur   Lunch     6
141       34.30   6.70    Male     No  Thur   Lunch     6
142       41.19   5.00    Male     No  Thur   Lunch     5
143       27.05   5.00  Female     No  Thur   Lunch     6
155       29.85   5.14  Female     No   Sun  Dinner     5
156       48.17   5.00    Male     No   Sun  Dinner     6
170       50.81  10.00    Male    Yes   Sat  Dinner     3
182       45.35   3.50    Male    Yes   Sun  Dinner     3
185       20.69   5.00    Male     No   Sun  Dinner     5
187       30.46   2.00    Male    Yes   Sun  Dinner     5
212       48.33   9.00    Male     No   Sat  Dinner     4
216       28.15   3.00    Male    Yes   Sat  Dinner     5
```

使用[``notna()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.Series.notna.html#pandas.Series.notna)和[``isna()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.Series.isna.html#pandas.Series.isna)
方法完成NULL检查。

``` python
In [13]: frame = pd.DataFrame({'col1': ['A', 'B', np.NaN, 'C', 'D'],
   ....:                       'col2': ['F', np.NaN, 'G', 'H', 'I']})
   ....: 

In [14]: frame
Out[14]: 
  col1 col2
0    A    F
1    B  NaN
2  NaN    G
3    C    H
4    D    I
```

假设我们有一个与上面的DataFrame结构相同的表。我们只能``col2``通过以下查询看到IS NULL 的记录：

``` sql
SELECT *
FROM frame
WHERE col2 IS NULL;
```

``` python
In [15]: frame[frame['col2'].isna()]
Out[15]: 
  col1 col2
1    B  NaN
```

获取``col1``IS NOT NULL的项目可以完成[``notna()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.Series.notna.html#pandas.Series.notna)。

``` sql
SELECT *
FROM frame
WHERE col1 IS NOT NULL;
```

``` python
In [16]: frame[frame['col1'].notna()]
Out[16]: 
  col1 col2
0    A    F
1    B  NaN
3    C    H
4    D    I
```

### GROUP BY 

在pandas中，SQL的GROUP BY操作使用类似命名的
 [``groupby()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby)方法执行。[``groupby()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby)通常是指我们想要将数据集拆分成组，应用某些功能（通常是聚合），然后将这些组合在一起的过程。

常见的SQL操作是获取整个数据集中每个组中的记录数。例如，一个查询向我们提供性别留下的提示数量：

``` sql
SELECT sex, count(*)
FROM tips
GROUP BY sex;
/*
Female     87
Male      157
*/
```

 pandas 相当于：

``` python
In [17]: tips.groupby('sex').size()
Out[17]: 
sex
Female     87
Male      157
dtype: int64
```

请注意，在我们使用的pandas代码中[``size()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.core.groupby.DataFrameGroupBy.size.html#pandas.core.groupby.DataFrameGroupBy.size)，没有
 [``count()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.core.groupby.DataFrameGroupBy.count.html#pandas.core.groupby.DataFrameGroupBy.count)。这是因为
 [``count()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.core.groupby.DataFrameGroupBy.count.html#pandas.core.groupby.DataFrameGroupBy.count)将函数应用于每个列，返回每个列中的记录数。``not null``

``` python
In [18]: tips.groupby('sex').count()
Out[18]: 
        total_bill  tip  smoker  day  time  size
sex                                             
Female          87   87      87   87    87    87
Male           157  157     157  157   157   157
```

或者，我们可以将该[``count()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.core.groupby.DataFrameGroupBy.count.html#pandas.core.groupby.DataFrameGroupBy.count)方法应用于单个列：

``` python
In [19]: tips.groupby('sex')['total_bill'].count()
Out[19]: 
sex
Female     87
Male      157
Name: total_bill, dtype: int64
```

也可以一次应用多个功能。例如，假设我们希望查看提示量与星期几的不同之处 - ``agg()``允许您将字典传递给分组的DataFrame，指示要应用于特定列的函数。

``` sql
SELECT day, AVG(tip), COUNT(*)
FROM tips
GROUP BY day;
/*
Fri   2.734737   19
Sat   2.993103   87
Sun   3.255132   76
Thur  2.771452   62
*/
```

``` python
In [20]: tips.groupby('day').agg({'tip': np.mean, 'day': np.size})
Out[20]: 
           tip  day
day                
Fri   2.734737   19
Sat   2.993103   87
Sun   3.255132   76
Thur  2.771452   62
```

通过将列列表传递给[``groupby()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.groupby.html#pandas.DataFrame.groupby)方法来完成多个列的分组
 。

``` sql
SELECT smoker, day, COUNT(*), AVG(tip)
FROM tips
GROUP BY smoker, day;
/*
smoker day
No     Fri      4  2.812500
       Sat     45  3.102889
       Sun     57  3.167895
       Thur    45  2.673778
Yes    Fri     15  2.714000
       Sat     42  2.875476
       Sun     19  3.516842
       Thur    17  3.030000
*/
```

``` python
In [21]: tips.groupby(['smoker', 'day']).agg({'tip': [np.size, np.mean]})
Out[21]: 
              tip          
             size      mean
smoker day                 
No     Fri    4.0  2.812500
       Sat   45.0  3.102889
       Sun   57.0  3.167895
       Thur  45.0  2.673778
Yes    Fri   15.0  2.714000
       Sat   42.0  2.875476
       Sun   19.0  3.516842
       Thur  17.0  3.030000
```

### 加入

可以使用[``join()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.join.html#pandas.DataFrame.join)或执行JOIN [``merge()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.merge.html#pandas.merge)。默认情况下，
 [``join()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.join.html#pandas.DataFrame.join)将在其索引上加入DataFrame。每个方法都有参数，允许您指定要执行的连接类型（LEFT，RIGHT，INNER，FULL）或要连接的列（列名称或索引）。

``` python
In [22]: df1 = pd.DataFrame({'key': ['A', 'B', 'C', 'D'],
   ....:                     'value': np.random.randn(4)})
   ....: 

In [23]: df2 = pd.DataFrame({'key': ['B', 'D', 'D', 'E'],
   ....:                     'value': np.random.randn(4)})
   ....:
```

假设我们有两个与DataFrames名称和结构相同的数据库表。

现在让我们来看看各种类型的JOIN。

#### 内部联接

``` sql
SELECT *
FROM df1
INNER JOIN df2
  ON df1.key = df2.key;
```

``` python
## merge performs an INNER JOIN by default
In [24]: pd.merge(df1, df2, on='key')
Out[24]: 
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209
```

[``merge()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.merge.html#pandas.merge) 当您想要将一个DataFrame列与另一个DataFrame索引连接时，还会为这些情况提供参数。

``` python
In [25]: indexed_df2 = df2.set_index('key')

In [26]: pd.merge(df1, indexed_df2, left_on='key', right_index=True)
Out[26]: 
  key   value_x   value_y
1   B -0.282863  1.212112
3   D -1.135632 -0.173215
3   D -1.135632  0.119209
```

#### LEFT OUTER JOIN 

``` sql
-- show all records from df1
SELECT *
FROM df1
LEFT OUTER JOIN df2
  ON df1.key = df2.key;
```

``` python
## show all records from df1
In [27]: pd.merge(df1, df2, on='key', how='left')
Out[27]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
```

#### 正确加入

``` sql
-- show all records from df2
SELECT *
FROM df1
RIGHT OUTER JOIN df2
  ON df1.key = df2.key;
```

``` python
## show all records from df2
In [28]: pd.merge(df1, df2, on='key', how='right')
Out[28]: 
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209
3   E       NaN -1.044236
```

#### 全加入

pandas还允许显示数据集两侧的FULL JOIN，无论连接列是否找到匹配项。在编写时，所有RDBMS（MySQL）都不支持FULL JOIN。

``` sql
-- show all records from both tables
SELECT *
FROM df1
FULL OUTER JOIN df2
  ON df1.key = df2.key;
```

``` python
## show all records from both frames
In [29]: pd.merge(df1, df2, on='key', how='outer')
Out[29]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E       NaN -1.044236
```

### UNION 

UNION ALL可以使用[``concat()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.concat.html#pandas.concat)。

``` python
In [30]: df1 = pd.DataFrame({'city': ['Chicago', 'San Francisco', 'New York City'],
   ....:                     'rank': range(1, 4)})
   ....: 

In [31]: df2 = pd.DataFrame({'city': ['Chicago', 'Boston', 'Los Angeles'],
   ....:                     'rank': [1, 4, 5]})
   ....:
```

``` sql
SELECT city, rank
FROM df1
UNION ALL
SELECT city, rank
FROM df2;
/*
         city  rank
      Chicago     1
San Francisco     2
New York City     3
      Chicago     1
       Boston     4
  Los Angeles     5
*/
```

``` python
In [32]: pd.concat([df1, df2])
Out[32]: 
            city  rank
0        Chicago     1
1  San Francisco     2
2  New York City     3
0        Chicago     1
1         Boston     4
2    Los Angeles     5
```

SQL的UNION类似于UNION ALL，但是UNION将删除重复的行。

``` sql
SELECT city, rank
FROM df1
UNION
SELECT city, rank
FROM df2;
-- notice that there is only one Chicago record this time
/*
         city  rank
      Chicago     1
San Francisco     2
New York City     3
       Boston     4
  Los Angeles     5
*/
```

在 pandas 中，您可以[``concat()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.concat.html#pandas.concat)结合使用
 [``drop_duplicates()``](https://pandas.pydata.org/pandas-docs/stable/../reference/api/pandas.DataFrame.drop_duplicates.html#pandas.DataFrame.drop_duplicates)。

``` python
In [33]: pd.concat([df1, df2]).drop_duplicates()
Out[33]: 
            city  rank
0        Chicago     1
1  San Francisco     2
2  New York City     3
1         Boston     4
2    Los Angeles     5
```

### Pandas等同于某些SQL分析和聚合函数

#### 带有偏移量的前N行

``` sql
-- MySQL
SELECT * FROM tips
ORDER BY tip DESC
LIMIT 10 OFFSET 5;
```

``` python
In [34]: tips.nlargest(10 + 5, columns='tip').tail(10)
Out[34]: 
     total_bill   tip     sex smoker   day    time  size
183       23.17  6.50    Male    Yes   Sun  Dinner     4
214       28.17  6.50  Female    Yes   Sat  Dinner     3
47        32.40  6.00    Male     No   Sun  Dinner     4
239       29.03  5.92    Male     No   Sat  Dinner     3
88        24.71  5.85    Male     No  Thur   Lunch     2
181       23.33  5.65    Male    Yes   Sun  Dinner     2
44        30.40  5.60    Male     No   Sun  Dinner     4
52        34.81  5.20  Female     No   Sun  Dinner     4
85        34.83  5.17  Female     No  Thur   Lunch     4
211       25.89  5.16    Male    Yes   Sat  Dinner     4
```

#### 每组前N行

``` sql
-- Oracle's ROW_NUMBER() analytic function
SELECT * FROM (
  SELECT
    t.*,
    ROW_NUMBER() OVER(PARTITION BY day ORDER BY total_bill DESC) AS rn
  FROM tips t
)
WHERE rn < 3
ORDER BY day, rn;
```

``` python
In [35]: (tips.assign(rn=tips.sort_values(['total_bill'], ascending=False)
   ....:                     .groupby(['day'])
   ....:                     .cumcount() + 1)
   ....:      .query('rn < 3')
   ....:      .sort_values(['day', 'rn']))
   ....: 
Out[35]: 
     total_bill    tip     sex smoker   day    time  size  rn
95        40.17   4.73    Male    Yes   Fri  Dinner     4   1
90        28.97   3.00    Male    Yes   Fri  Dinner     2   2
170       50.81  10.00    Male    Yes   Sat  Dinner     3   1
212       48.33   9.00    Male     No   Sat  Dinner     4   2
156       48.17   5.00    Male     No   Sun  Dinner     6   1
182       45.35   3.50    Male    Yes   Sun  Dinner     3   2
197       43.11   5.00  Female    Yes  Thur   Lunch     4   1
142       41.19   5.00    Male     No  Thur   Lunch     5   2
```

同样使用*rank（method ='first'）*函数

``` python
In [36]: (tips.assign(rnk=tips.groupby(['day'])['total_bill']
   ....:                      .rank(method='first', ascending=False))
   ....:      .query('rnk < 3')
   ....:      .sort_values(['day', 'rnk']))
   ....: 
Out[36]: 
     total_bill    tip     sex smoker   day    time  size  rnk
95        40.17   4.73    Male    Yes   Fri  Dinner     4  1.0
90        28.97   3.00    Male    Yes   Fri  Dinner     2  2.0
170       50.81  10.00    Male    Yes   Sat  Dinner     3  1.0
212       48.33   9.00    Male     No   Sat  Dinner     4  2.0
156       48.17   5.00    Male     No   Sun  Dinner     6  1.0
182       45.35   3.50    Male    Yes   Sun  Dinner     3  2.0
197       43.11   5.00  Female    Yes  Thur   Lunch     4  1.0
142       41.19   5.00    Male     No  Thur   Lunch     5  2.0
```

``` sql
-- Oracle's RANK() analytic function
SELECT * FROM (
  SELECT
    t.*,
    RANK() OVER(PARTITION BY sex ORDER BY tip) AS rnk
  FROM tips t
  WHERE tip < 2
)
WHERE rnk < 3
ORDER BY sex, rnk;
```

让我们找到每个性别组（等级<3）的提示（提示<2）。请注意，使用``rank(method='min')``函数时
 *rnk_min*对于相同的*提示*保持不变
（如Oracle的RANK（）函数）

``` python
In [37]: (tips[tips['tip'] < 2]
   ....:     .assign(rnk_min=tips.groupby(['sex'])['tip']
   ....:                         .rank(method='min'))
   ....:     .query('rnk_min < 3')
   ....:     .sort_values(['sex', 'rnk_min']))
   ....: 
Out[37]: 
     total_bill   tip     sex smoker  day    time  size  rnk_min
67         3.07  1.00  Female    Yes  Sat  Dinner     1      1.0
92         5.75  1.00  Female    Yes  Fri  Dinner     2      1.0
111        7.25  1.00  Female     No  Sat  Dinner     1      1.0
236       12.60  1.00    Male    Yes  Sat  Dinner     2      1.0
237       32.83  1.17    Male    Yes  Sat  Dinner     2      2.0
```

### 更新

``` sql
UPDATE tips
SET tip = tip*2
WHERE tip < 2;
```

``` python
In [38]: tips.loc[tips['tip'] < 2, 'tip'] *= 2
```

### 删除

``` sql
DELETE FROM tips
WHERE tip > 9;
```

在pandas中，我们选择应保留的行，而不是删除它们

``` python
In [39]: tips = tips.loc[tips['tip'] <= 9]
```

## 与SAS比较

For potential users coming from [SAS](https://en.wikipedia.org/wiki/SAS_(software)) this page is meant to demonstrate how different SAS operations would be performed in pandas.

If you’re new to pandas, you might want to first read through [10 Minutes to pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/10min.html#min) to familiarize yourself with the library.

As is customary, we import pandas and NumPy as follows:

``` python
In [1]: import pandas as pd

In [2]: import numpy as np
```

::: tip Note

Throughout this tutorial, the pandas DataFrame will be displayed by calling df.head(), which displays the first N (default 5) rows of the DataFrame. This is often used in interactive work (e.g. Jupyter notebook or terminal) - the equivalent in SAS would be:

``` python
proc print data=df(obs=5);
run;
```

:::

### Data Structures

#### General Terminology Translation

pandas | SAS
---|---
DataFrame | data set
column | variable
row | observation
groupby | BY-group
NaN | .

#### DataFrame / Series

A ``DataFrame`` in pandas is analogous to a SAS data set - a two-dimensional data source with labeled columns that can be of different types. As will be shown in this document, almost any operation that can be applied to a data set using SAS’s ``DATA`` step, can also be accomplished in pandas.

A ``Series`` is the data structure that represents one column of a ``DataFrame``. SAS doesn’t have a separate data structure for a single column, but in general, working with a ``Series`` is analogous to referencing a column in the ``DATA`` step.

#### Index

Every ``DataFrame`` and ``Series`` has an ``Index`` - which are labels on the rows of the data. SAS does not have an exactly analogous concept. A data set’s rows are essentially unlabeled, other than an implicit integer index that can be accessed during the DATA step (``_N_``).

In pandas, if no index is specified, an integer index is also used by default (first row = 0, second row = 1, and so on). While using a labeled ``Index`` or ``MultiIndex`` can enable sophisticated analyses and is ultimately an important part of pandas to understand, for this comparison we will essentially ignore the ``Index`` and just treat the ``DataFrame`` as a collection of columns. Please see the [indexing documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#indexing) for much more on how to use an ``Index`` effectively.

### Data Input / Output

#### Constructing a DataFrame from Values

A SAS data set can be built from specified values by placing the data after a ``datalines`` statement and specifying the column names.

``` sas
data df;
    input x y;
    datalines;
    1 2
    3 4
    5 6
    ;
run;
```

A pandas ``DataFrame`` can be constructed in many different ways, but for a small number of values, it is often convenient to specify it as a Python dictionary, where the keys are the column names and the values are the data.

``` python
In [3]: df = pd.DataFrame({'x': [1, 3, 5], 'y': [2, 4, 6]})

In [4]: df
Out[4]: 
   x  y
0  1  2
1  3  4
2  5  6
```

#### Reading External Data

Like SAS, pandas provides utilities for reading in data from many formats. The ``tips`` dataset, found within the pandas tests ([csv](https://raw.github.com/pandas-dev/pandas/master/pandas/tests/data/tips.csv)) will be used in many of the following examples.

SAS provides ``PROC IMPORT`` to read csv data into a data set.

``` sas
proc import datafile='tips.csv' dbms=csv out=tips replace;
    getnames=yes;
run;
```

The pandas method is [read_csv()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html#pandas.read_csv), which works similarly.

``` python
In [5]: url = ('https://raw.github.com/pandas-dev/'
   ...:        'pandas/master/pandas/tests/data/tips.csv')
   ...: 

In [6]: tips = pd.read_csv(url)

In [7]: tips.head()
Out[7]: 
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

Like ``PROC`` ``IMPORT``, ``read_csv`` can take a number of parameters to specify how the data should be parsed. For example, if the data was instead tab delimited, and did not have column names, the pandas command would be:

``` python
tips = pd.read_csv('tips.csv', sep='\t', header=None)

# alternatively, read_table is an alias to read_csv with tab delimiter
tips = pd.read_table('tips.csv', header=None)
```

In addition to text/csv, pandas supports a variety of other data formats such as Excel, HDF5, and SQL databases. These are all read via a ``pd.read_*`` function. See the [IO documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#io) for more details.

#### Exporting Data

The inverse of ``PROC`` ``IMPORT`` in SAS is ``PROC`` ``EXPORT``

``` sas
proc export data=tips outfile='tips2.csv' dbms=csv;
run;
```

Similarly in pandas, the opposite of ``read_csv`` is [to_csv()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_csv.html#pandas.DataFrame.to_csv), and other data formats follow a similar api.

``` python
tips.to_csv('tips2.csv')
```

### Data Operations

#### Operations on Columns

In the DATA step, arbitrary math expressions can be used on new or existing columns.

``` sas
data tips;
    set tips;
    total_bill = total_bill - 2;
    new_bill = total_bill / 2;
run;
```

pandas provides similar vectorized operations by specifying the individual ``Series`` in the ``DataFrame``. New columns can be assigned in the same way.

``` python
In [8]: tips['total_bill'] = tips['total_bill'] - 2

In [9]: tips['new_bill'] = tips['total_bill'] / 2.0

In [10]: tips.head()
Out[10]: 
   total_bill   tip     sex smoker  day    time  size  new_bill
0       14.99  1.01  Female     No  Sun  Dinner     2     7.495
1        8.34  1.66    Male     No  Sun  Dinner     3     4.170
2       19.01  3.50    Male     No  Sun  Dinner     3     9.505
3       21.68  3.31    Male     No  Sun  Dinner     2    10.840
4       22.59  3.61  Female     No  Sun  Dinner     4    11.295
```

#### Filtering

Filtering in SAS is done with an ``if`` or ``where`` statement, on one or more columns.

``` python
data tips;
    set tips;
    if total_bill > 10;
run;

data tips;
    set tips;
    where total_bill > 10;
    /* equivalent in this case - where happens before the
       DATA step begins and can also be used in PROC statements */
run;
```

DataFrames can be filtered in multiple ways; the most intuitive of which is using [boolean indexing](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#indexing-boolean)

``` python
In [11]: tips[tips['total_bill'] > 10].head()
Out[11]: 
   total_bill   tip     sex smoker  day    time  size
0       14.99  1.01  Female     No  Sun  Dinner     2
2       19.01  3.50    Male     No  Sun  Dinner     3
3       21.68  3.31    Male     No  Sun  Dinner     2
4       22.59  3.61  Female     No  Sun  Dinner     4
5       23.29  4.71    Male     No  Sun  Dinner     4
```

#### If/Then Logic

In SAS, if/then logic can be used to create new columns.

``` sas
data tips;
    set tips;
    format bucket $4.;

    if total_bill < 10 then bucket = 'low';
    else bucket = 'high';
run;
```

The same operation in pandas can be accomplished using the ``where`` method from ``numpy``.

``` python
In [12]: tips['bucket'] = np.where(tips['total_bill'] < 10, 'low', 'high')

In [13]: tips.head()
Out[13]: 
   total_bill   tip     sex smoker  day    time  size bucket
0       14.99  1.01  Female     No  Sun  Dinner     2   high
1        8.34  1.66    Male     No  Sun  Dinner     3    low
2       19.01  3.50    Male     No  Sun  Dinner     3   high
3       21.68  3.31    Male     No  Sun  Dinner     2   high
4       22.59  3.61  Female     No  Sun  Dinner     4   high
```

### Date Functionality

SAS provides a variety of functions to do operations on date/datetime columns.

``` sas
data tips;
    set tips;
    format date1 date2 date1_plusmonth mmddyy10.;
    date1 = mdy(1, 15, 2013);
    date2 = mdy(2, 15, 2015);
    date1_year = year(date1);
    date2_month = month(date2);
    * shift date to beginning of next interval;
    date1_next = intnx('MONTH', date1, 1);
    * count intervals between dates;
    months_between = intck('MONTH', date1, date2);
run;
```

The equivalent pandas operations are shown below. In addition to these functions pandas supports other Time Series features not available in Base SAS (such as resampling and custom offsets) - see the [timeseries documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#timeseries) for more details.

``` python
In [14]: tips['date1'] = pd.Timestamp('2013-01-15')

In [15]: tips['date2'] = pd.Timestamp('2015-02-15')

In [16]: tips['date1_year'] = tips['date1'].dt.year

In [17]: tips['date2_month'] = tips['date2'].dt.month

In [18]: tips['date1_next'] = tips['date1'] + pd.offsets.MonthBegin()

In [19]: tips['months_between'] = (
   ....:     tips['date2'].dt.to_period('M') - tips['date1'].dt.to_period('M'))
   ....: 

In [20]: tips[['date1', 'date2', 'date1_year', 'date2_month',
   ....:       'date1_next', 'months_between']].head()
   ....: 
Out[20]: 
       date1      date2  date1_year  date2_month date1_next    months_between
0 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
1 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
2 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
3 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
4 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
```

#### Selection of Columns

SAS provides keywords in the DATA step to select, drop, and rename columns.

``` sas
data tips;
    set tips;
    keep sex total_bill tip;
run;

data tips;
    set tips;
    drop sex;
run;

data tips;
    set tips;
    rename total_bill=total_bill_2;
run;
```

The same operations are expressed in pandas below.

``` python
# keep
In [21]: tips[['sex', 'total_bill', 'tip']].head()
Out[21]: 
      sex  total_bill   tip
0  Female       14.99  1.01
1    Male        8.34  1.66
2    Male       19.01  3.50
3    Male       21.68  3.31
4  Female       22.59  3.61

# drop
In [22]: tips.drop('sex', axis=1).head()
Out[22]: 
   total_bill   tip smoker  day    time  size
0       14.99  1.01     No  Sun  Dinner     2
1        8.34  1.66     No  Sun  Dinner     3
2       19.01  3.50     No  Sun  Dinner     3
3       21.68  3.31     No  Sun  Dinner     2
4       22.59  3.61     No  Sun  Dinner     4

# rename
In [23]: tips.rename(columns={'total_bill': 'total_bill_2'}).head()
Out[23]: 
   total_bill_2   tip     sex smoker  day    time  size
0         14.99  1.01  Female     No  Sun  Dinner     2
1          8.34  1.66    Male     No  Sun  Dinner     3
2         19.01  3.50    Male     No  Sun  Dinner     3
3         21.68  3.31    Male     No  Sun  Dinner     2
4         22.59  3.61  Female     No  Sun  Dinner     4
```

#### Sorting by Values

Sorting in SAS is accomplished via ``PROC`` ``SORT``

``` sas
proc sort data=tips;
    by sex total_bill;
run;
```

pandas objects have a [sort_values()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.sort_values.html#pandas.DataFrame.sort_values) method, which takes a list of columns to sort by.

``` python
In [24]: tips = tips.sort_values(['sex', 'total_bill'])

In [25]: tips.head()
Out[25]: 
     total_bill   tip     sex smoker   day    time  size
67         1.07  1.00  Female    Yes   Sat  Dinner     1
92         3.75  1.00  Female    Yes   Fri  Dinner     2
111        5.25  1.00  Female     No   Sat  Dinner     1
145        6.35  1.50  Female     No  Thur   Lunch     2
135        6.51  1.25  Female     No  Thur   Lunch     2
```

### String Processing

#### Length

SAS determines the length of a character string with the [LENGTHN](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a002284668.htm) and [LENGTHC](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a002283942.htm) functions. LENGTHN excludes trailing blanks and LENGTHC includes trailing blanks.

``` sas
data _null_;
set tips;
put(LENGTHN(time));
put(LENGTHC(time));
run;
```

Python determines the length of a character string with the ``len`` function. ``len`` includes trailing blanks. Use ``len`` and ``rstrip`` to exclude trailing blanks.

``` python
In [26]: tips['time'].str.len().head()
Out[26]: 
67     6
92     6
111    6
145    5
135    5
Name: time, dtype: int64

In [27]: tips['time'].str.rstrip().str.len().head()
Out[27]: 
67     6
92     6
111    6
145    5
135    5
Name: time, dtype: int64
```

#### Find

SAS determines the position of a character in a string with the [FINDW](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a002978282.htm) function. ``FINDW`` takes the string defined by the first argument and searches for the first position of the substring you supply as the second argument.

``` sas
data _null_;
set tips;
put(FINDW(sex,'ale'));
run;
```

Python determines the position of a character in a string with the ``find`` function. ``find`` searches for the first position of the substring. If the substring is found, the function returns its position. Keep in mind that Python indexes are zero-based and the function will return -1 if it fails to find the substring.

``` python
In [28]: tips['sex'].str.find("ale").head()
Out[28]: 
67     3
92     3
111    3
145    3
135    3
Name: sex, dtype: int64
```

#### Substring

SAS extracts a substring from a string based on its position with the [SUBSTR](https://www2.sas.com/proceedings/sugi25/25/cc/25p088.pdf) function.

``` sas
data _null_;
set tips;
put(substr(sex,1,1));
run;
```

With pandas you can use ``[]`` notation to extract a substring from a string by position locations. Keep in mind that Python indexes are zero-based.

``` sas
In [29]: tips['sex'].str[0:1].head()
Out[29]: 
67     F
92     F
111    F
145    F
135    F
Name: sex, dtype: object
```

#### Scan

The SAS [SCAN](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a000214639.htm) function returns the nth word from a string. The first argument is the string you want to parse and the second argument specifies which word you want to extract.

``` sas
data firstlast;
input String $60.;
First_Name = scan(string, 1);
Last_Name = scan(string, -1);
datalines2;
John Smith;
Jane Cook;
;;;
run;
```

Python extracts a substring from a string based on its text by using regular expressions. There are much more powerful approaches, but this just shows a simple approach.

``` python
In [30]: firstlast = pd.DataFrame({'String': ['John Smith', 'Jane Cook']})

In [31]: firstlast['First_Name'] = firstlast['String'].str.split(" ", expand=True)[0]

In [32]: firstlast['Last_Name'] = firstlast['String'].str.rsplit(" ", expand=True)[0]

In [33]: firstlast
Out[33]: 
       String First_Name Last_Name
0  John Smith       John      John
1   Jane Cook       Jane      Jane
```

#### Upcase, Lowcase, and Propcase

The SAS [UPCASE](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a000245965.htm) [LOWCASE](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a000245912.htm) and [PROPCASE](https://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/a002598106.htm) functions change the case of the argument.

``` sas
data firstlast;
input String $60.;
string_up = UPCASE(string);
string_low = LOWCASE(string);
string_prop = PROPCASE(string);
datalines2;
John Smith;
Jane Cook;
;;;
run;
```

The equivalent Python functions are ``upper``, ``lower``, and ``title``.

``` python
In [34]: firstlast = pd.DataFrame({'String': ['John Smith', 'Jane Cook']})

In [35]: firstlast['string_up'] = firstlast['String'].str.upper()

In [36]: firstlast['string_low'] = firstlast['String'].str.lower()

In [37]: firstlast['string_prop'] = firstlast['String'].str.title()

In [38]: firstlast
Out[38]: 
       String   string_up  string_low string_prop
0  John Smith  JOHN SMITH  john smith  John Smith
1   Jane Cook   JANE COOK   jane cook   Jane Cook
```

### Merging

The following tables will be used in the merge examples

``` python
In [39]: df1 = pd.DataFrame({'key': ['A', 'B', 'C', 'D'],
   ....:                     'value': np.random.randn(4)})
   ....: 

In [40]: df1
Out[40]: 
  key     value
0   A  0.469112
1   B -0.282863
2   C -1.509059
3   D -1.135632

In [41]: df2 = pd.DataFrame({'key': ['B', 'D', 'D', 'E'],
   ....:                     'value': np.random.randn(4)})
   ....: 

In [42]: df2
Out[42]: 
  key     value
0   B  1.212112
1   D -0.173215
2   D  0.119209
3   E -1.044236
```

In SAS, data must be explicitly sorted before merging. Different types of joins are accomplished using the ``in=`` dummy variables to track whether a match was found in one or both input frames.

``` sas
proc sort data=df1;
    by key;
run;

proc sort data=df2;
    by key;
run;

data left_join inner_join right_join outer_join;
    merge df1(in=a) df2(in=b);

    if a and b then output inner_join;
    if a then output left_join;
    if b then output right_join;
    if a or b then output outer_join;
run;
```

pandas DataFrames have a [merge()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.merge.html#pandas.DataFrame.merge) method, which provides similar functionality. Note that the data does not have to be sorted ahead of time, and different join types are accomplished via the ``how`` keyword.

``` python
In [43]: inner_join = df1.merge(df2, on=['key'], how='inner')

In [44]: inner_join
Out[44]: 
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209

In [45]: left_join = df1.merge(df2, on=['key'], how='left')

In [46]: left_join
Out[46]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209

In [47]: right_join = df1.merge(df2, on=['key'], how='right')

In [48]: right_join
Out[48]: 
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209
3   E       NaN -1.044236

In [49]: outer_join = df1.merge(df2, on=['key'], how='outer')

In [50]: outer_join
Out[50]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E       NaN -1.044236
```

### Missing Data

Like SAS, pandas has a representation for missing data - which is the special float value ``NaN`` (not a number). Many of the semantics are the same, for example missing data propagates through numeric operations, and is ignored by default for aggregations.

``` python
In [51]: outer_join
Out[51]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E       NaN -1.044236

In [52]: outer_join['value_x'] + outer_join['value_y']
Out[52]: 
0         NaN
1    0.929249
2         NaN
3   -1.308847
4   -1.016424
5         NaN
dtype: float64

In [53]: outer_join['value_x'].sum()
Out[53]: -3.5940742896293765
```

One difference is that missing data cannot be compared to its sentinel value. For example, in SAS you could do this to filter missing values.

``` sas
data outer_join_nulls;
    set outer_join;
    if value_x = .;
run;

data outer_join_no_nulls;
    set outer_join;
    if value_x ^= .;
run;
```

Which doesn’t work in pandas. Instead, the ``pd.isna`` or ``pd.notna`` functions should be used for comparisons.

``` python
In [54]: outer_join[pd.isna(outer_join['value_x'])]
Out[54]: 
  key  value_x   value_y
5   E      NaN -1.044236

In [55]: outer_join[pd.notna(outer_join['value_x'])]
Out[55]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
```

pandas also provides a variety of methods to work with missing data - some of which would be challenging to express in SAS. For example, there are methods to drop all rows with any missing values, replacing mi(ssing values with a specified value, like the mean, or forward filling from previous rows. See the [missing data documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html#missing-data) for more.

``` python
In [56]: outer_join.dropna()
Out[56]: 
  key   value_x   value_y
1   B -0.282863  1.212112
3   D -1.135632 -0.173215
4   D -1.135632  0.119209

In [57]: outer_join.fillna(method='ffill')
Out[57]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059  1.212112
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E -1.135632 -1.044236

In [58]: outer_join['value_x'].fillna(outer_join['value_x'].mean())
Out[58]: 
0    0.469112
1   -0.282863
2   -1.509059
3   -1.135632
4   -1.135632
5   -0.718815
Name: value_x, dtype: float64
```

### GroupBy

#### Aggregation

SAS’s PROC SUMMARY can be used to group by one or more key variables and compute aggregations on numeric columns.

``` sas
proc summary data=tips nway;
    class sex smoker;
    var total_bill tip;
    output out=tips_summed sum=;
run;
```

pandas provides a flexible ``groupby`` mechanism that allows similar aggregations. See the [groupby documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/groupby.html#groupby) for more details and examples.

``` python
In [59]: tips_summed = tips.groupby(['sex', 'smoker'])['total_bill', 'tip'].sum()

In [60]: tips_summed.head()
Out[60]: 
               total_bill     tip
sex    smoker                    
Female No          869.68  149.77
       Yes         527.27   96.74
Male   No         1725.75  302.00
       Yes        1217.07  183.07
```

#### Transformation

In SAS, if the group aggregations need to be used with the original frame, it must be merged back together. For example, to subtract the mean for each observation by smoker group.

``` sas
proc summary data=tips missing nway;
    class smoker;
    var total_bill;
    output out=smoker_means mean(total_bill)=group_bill;
run;

proc sort data=tips;
    by smoker;
run;

data tips;
    merge tips(in=a) smoker_means(in=b);
    by smoker;
    adj_total_bill = total_bill - group_bill;
    if a and b;
run;
```

pandas ``groubpy`` provides a ``transform`` mechanism that allows these type of operations to be succinctly expressed in one operation.

``` python
In [61]: gb = tips.groupby('smoker')['total_bill']

In [62]: tips['adj_total_bill'] = tips['total_bill'] - gb.transform('mean')

In [63]: tips.head()
Out[63]: 
     total_bill   tip     sex smoker   day    time  size  adj_total_bill
67         1.07  1.00  Female    Yes   Sat  Dinner     1      -17.686344
92         3.75  1.00  Female    Yes   Fri  Dinner     2      -15.006344
111        5.25  1.00  Female     No   Sat  Dinner     1      -11.938278
145        6.35  1.50  Female     No  Thur   Lunch     2      -10.838278
135        6.51  1.25  Female     No  Thur   Lunch     2      -10.678278
```

#### By Group Processing

In addition to aggregation, pandas ``groupby`` can be used to replicate most other by group processing from SAS. For example, this ``DATA`` step reads the data by sex/smoker group and filters to the first entry for each.

``` sas
proc sort data=tips;
   by sex smoker;
run;

data tips_first;
    set tips;
    by sex smoker;
    if FIRST.sex or FIRST.smoker then output;
run;
```

In pandas this would be written as:

``` python
In [64]: tips.groupby(['sex', 'smoker']).first()
Out[64]: 
               total_bill   tip   day    time  size  adj_total_bill
sex    smoker                                                      
Female No            5.25  1.00   Sat  Dinner     1      -11.938278
       Yes           1.07  1.00   Sat  Dinner     1      -17.686344
Male   No            5.51  2.00  Thur   Lunch     2      -11.678278
       Yes           5.25  5.15   Sun  Dinner     2      -13.506344
```

### Other Considerations

#### Disk vs Memory

pandas operates exclusively in memory, where a SAS data set exists on disk. This means that the size of data able to be loaded in pandas is limited by your machine’s memory, but also that the operations on that data may be faster.

If out of core processing is needed, one possibility is the [dask.dataframe](https://dask.pydata.org/en/latest/dataframe.html) library (currently in development) which provides a subset of pandas functionality for an on-disk ``DataFrame``

#### Data Interop

pandas provides a [read_sas()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_sas.html#pandas.read_sas) method that can read SAS data saved in the XPORT or SAS7BDAT binary format.

``` python
libname xportout xport 'transport-file.xpt';
data xportout.tips;
    set tips(rename=(total_bill=tbill));
    * xport variable names limited to 6 characters;
run;
```

``` python
df = pd.read_sas('transport-file.xpt')
df = pd.read_sas('binary-file.sas7bdat')
```

You can also specify the file format directly. By default, pandas will try to infer the file format based on its extension.

``` python
df = pd.read_sas('transport-file.xpt', format='xport')
df = pd.read_sas('binary-file.sas7bdat', format='sas7bdat')
```

XPORT is a relatively limited format and the parsing of it is not as optimized as some of the other pandas readers. An alternative way to interop data between SAS and pandas is to serialize to csv.

``` python
# version 0.17, 10M rows

In [8]: %time df = pd.read_sas('big.xpt')
Wall time: 14.6 s

In [9]: %time df = pd.read_csv('big.csv')
Wall time: 4.86 s
```

## 与Stata比较

For potential users coming from [Stata](https://en.wikipedia.org/wiki/Stata) this page is meant to demonstrate how different Stata operations would be performed in pandas.

If you’re new to pandas, you might want to first read through [10 Minutes to pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/10min.html#min) to familiarize yourself with the library.

As is customary, we import pandas and NumPy as follows. This means that we can refer to the libraries as ``pd`` and ``np``, respectively, for the rest of the document.

``` python
In [1]: import pandas as pd

In [2]: import numpy as np
```

::: tip Note

Throughout this tutorial, the pandas ``DataFrame`` will be displayed by calling ``df.head()``, which displays the first N (default 5) rows of the ``DataFrame``. This is often used in interactive work (e.g. [Jupyter notebook](https://jupyter.org/) or terminal) – the equivalent in Stata would be:

``` bash
list in 1/5
```

:::

### Data Structures

#### General Terminology Translation

pandas | Stata
---|---
DataFrame | data set
column | variable
row | observation
groupby | bysort
NaN | .

#### DataFrame / Series

A ``DataFrame`` in pandas is analogous to a Stata data set – a two-dimensional data source with labeled columns that can be of different types. As will be shown in this document, almost any operation that can be applied to a data set in Stata can also be accomplished in pandas.

A ``Series`` is the data structure that represents one column of a ``DataFrame``. Stata doesn’t have a separate data structure for a single column, but in general, working with a ``Series`` is analogous to referencing a column of a data set in Stata.

#### Index

Every ``DataFrame`` and ``Series`` has an ``Index`` – labels on the rows of the data. Stata does not have an exactly analogous concept. In Stata, a data set’s rows are essentially unlabeled, other than an implicit integer index that can be accessed with ``_n``.

In pandas, if no index is specified, an integer index is also used by default (first row = 0, second row = 1, and so on). While using a labeled ``Index`` or ``MultiIndex`` can enable sophisticated analyses and is ultimately an important part of pandas to understand, for this comparison we will essentially ignore the ``Index`` and just treat the ``DataFrame`` as a collection of columns. Please see the [indexing documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#indexing) for much more on how to use an ``Index`` effectively.

### Data Input / Output

#### Constructing a DataFrame from Values

A Stata data set can be built from specified values by placing the data after an ``input`` statement and specifying the column names.

``` bash
input x y
1 2
3 4
5 6
end
```

A pandas ``DataFrame`` can be constructed in many different ways, but for a small number of values, it is often convenient to specify it as a Python dictionary, where the keys are the column names and the values are the data.

``` python
In [3]: df = pd.DataFrame({'x': [1, 3, 5], 'y': [2, 4, 6]})

In [4]: df
Out[4]: 
   x  y
0  1  2
1  3  4
2  5  6
```

#### Reading External Data

Like Stata, pandas provides utilities for reading in data from many formats. The ``tips`` data set, found within the pandas tests ([csv](https://raw.github.com/pandas-dev/pandas/master/pandas/tests/data/tips.csv)) will be used in many of the following examples.

Stata provides ``import`` ``delimited`` to read csv data into a data set in memory. If the ``tips.csv`` file is in the current working directory, we can import it as follows.

``` python
import delimited tips.csv
```

The pandas method is [read_csv()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html#pandas.read_csv), which works similarly. Additionally, it will automatically download the data set if presented with a url.

``` python
In [5]: url = ('https://raw.github.com/pandas-dev'
   ...:        '/pandas/master/pandas/tests/data/tips.csv')
   ...: 

In [6]: tips = pd.read_csv(url)

In [7]: tips.head()
Out[7]: 
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

Like ``import`` ``delimited``, [read_csv()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html#pandas.read_csv) can take a number of parameters to specify how the data should be parsed. For example, if the data were instead tab delimited, did not have column names, and existed in the current working directory, the pandas command would be:

``` python
tips = pd.read_csv('tips.csv', sep='\t', header=None)

# alternatively, read_table is an alias to read_csv with tab delimiter
tips = pd.read_table('tips.csv', header=None)
```

Pandas can also read Stata data sets in ``.dta`` format with the [read_stata()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_stata.html#pandas.read_stata) function.

``` python
df = pd.read_stata('data.dta')
```

In addition to text/csv and Stata files, pandas supports a variety of other data formats such as Excel, SAS, HDF5, Parquet, and SQL databases. These are all read via a ``pd.read_*`` function. See the [IO documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#io) for more details.

#### Exporting Data

The inverse of ``import`` ``delimited`` in Stata is ``export`` ``delimited``

``` bash
export delimited tips2.csv
```

Similarly in pandas, the opposite of ``read_csv`` is [DataFrame.to_csv()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_csv.html#pandas.DataFrame.to_csv).

``` python
tips.to_csv('tips2.csv')
```

Pandas can also export to Stata file format with the [DataFrame.to_stata()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_stata.html#pandas.DataFrame.to_stata) method.

``` python
tips.to_stata('tips2.dta')
```

### Data Operations

#### Operations on Columns

In Stata, arbitrary math expressions can be used with the ``generate`` and ``replace`` commands on new or existing columns. The drop command drops the column from the data set.

``` python
replace total_bill = total_bill - 2
generate new_bill = total_bill / 2
drop new_bill
```

pandas provides similar vectorized operations by specifying the individual ``Series`` in the ``DataFrame``. New columns can be assigned in the same way. The [DataFrame.drop()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.drop.html#pandas.DataFrame.drop) method drops a column from the ``DataFrame``.

``` python
In [8]: tips['total_bill'] = tips['total_bill'] - 2

In [9]: tips['new_bill'] = tips['total_bill'] / 2

In [10]: tips.head()
Out[10]: 
   total_bill   tip     sex smoker  day    time  size  new_bill
0       14.99  1.01  Female     No  Sun  Dinner     2     7.495
1        8.34  1.66    Male     No  Sun  Dinner     3     4.170
2       19.01  3.50    Male     No  Sun  Dinner     3     9.505
3       21.68  3.31    Male     No  Sun  Dinner     2    10.840
4       22.59  3.61  Female     No  Sun  Dinner     4    11.295

In [11]: tips = tips.drop('new_bill', axis=1)
```

#### Filtering

Filtering in Stata is done with an if clause on one or more columns.

``` bash
list if total_bill > 10
```

DataFrames can be filtered in multiple ways; the most intuitive of which is using [boolean indexing](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#indexing-boolean).

``` python
In [12]: tips[tips['total_bill'] > 10].head()
Out[12]: 
   total_bill   tip     sex smoker  day    time  size
0       14.99  1.01  Female     No  Sun  Dinner     2
2       19.01  3.50    Male     No  Sun  Dinner     3
3       21.68  3.31    Male     No  Sun  Dinner     2
4       22.59  3.61  Female     No  Sun  Dinner     4
5       23.29  4.71    Male     No  Sun  Dinner     4
```

#### If/Then Logic

In Stata, an ``if`` clause can also be used to create new columns.

``` bash
generate bucket = "low" if total_bill < 10
replace bucket = "high" if total_bill >= 10
```

The same operation in pandas can be accomplished using the ``where`` method from ``numpy``.

``` python
In [13]: tips['bucket'] = np.where(tips['total_bill'] < 10, 'low', 'high')

In [14]: tips.head()
Out[14]: 
   total_bill   tip     sex smoker  day    time  size bucket
0       14.99  1.01  Female     No  Sun  Dinner     2   high
1        8.34  1.66    Male     No  Sun  Dinner     3    low
2       19.01  3.50    Male     No  Sun  Dinner     3   high
3       21.68  3.31    Male     No  Sun  Dinner     2   high
4       22.59  3.61  Female     No  Sun  Dinner     4   high
```

#### Date Functionality

Stata provides a variety of functions to do operations on date/datetime columns.

``` bash
generate date1 = mdy(1, 15, 2013)
generate date2 = date("Feb152015", "MDY")

generate date1_year = year(date1)
generate date2_month = month(date2)

* shift date to beginning of next month
generate date1_next = mdy(month(date1) + 1, 1, year(date1)) if month(date1) != 12
replace date1_next = mdy(1, 1, year(date1) + 1) if month(date1) == 12
generate months_between = mofd(date2) - mofd(date1)

list date1 date2 date1_year date2_month date1_next months_between
```

The equivalent pandas operations are shown below. In addition to these functions, pandas supports other Time Series features not available in Stata (such as time zone handling and custom offsets) – see the [timeseries documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#timeseries) for more details.

``` python
In [15]: tips['date1'] = pd.Timestamp('2013-01-15')

In [16]: tips['date2'] = pd.Timestamp('2015-02-15')

In [17]: tips['date1_year'] = tips['date1'].dt.year

In [18]: tips['date2_month'] = tips['date2'].dt.month

In [19]: tips['date1_next'] = tips['date1'] + pd.offsets.MonthBegin()

In [20]: tips['months_between'] = (tips['date2'].dt.to_period('M')
   ....:                           - tips['date1'].dt.to_period('M'))
   ....: 

In [21]: tips[['date1', 'date2', 'date1_year', 'date2_month', 'date1_next',
   ....:       'months_between']].head()
   ....: 
Out[21]: 
       date1      date2  date1_year  date2_month date1_next    months_between
0 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
1 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
2 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
3 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
4 2013-01-15 2015-02-15        2013            2 2013-02-01  <25 * MonthEnds>
```

#### Selection of Columns

Stata provides keywords to select, drop, and rename columns.

``` bash
keep sex total_bill tip

drop sex

rename total_bill total_bill_2
```

The same operations are expressed in pandas below. Note that in contrast to Stata, these operations do not happen in place. To make these changes persist, assign the operation back to a variable.

``` python
# keep
In [22]: tips[['sex', 'total_bill', 'tip']].head()
Out[22]: 
      sex  total_bill   tip
0  Female       14.99  1.01
1    Male        8.34  1.66
2    Male       19.01  3.50
3    Male       21.68  3.31
4  Female       22.59  3.61

# drop
In [23]: tips.drop('sex', axis=1).head()
Out[23]: 
   total_bill   tip smoker  day    time  size
0       14.99  1.01     No  Sun  Dinner     2
1        8.34  1.66     No  Sun  Dinner     3
2       19.01  3.50     No  Sun  Dinner     3
3       21.68  3.31     No  Sun  Dinner     2
4       22.59  3.61     No  Sun  Dinner     4

# rename
In [24]: tips.rename(columns={'total_bill': 'total_bill_2'}).head()
Out[24]: 
   total_bill_2   tip     sex smoker  day    time  size
0         14.99  1.01  Female     No  Sun  Dinner     2
1          8.34  1.66    Male     No  Sun  Dinner     3
2         19.01  3.50    Male     No  Sun  Dinner     3
3         21.68  3.31    Male     No  Sun  Dinner     2
4         22.59  3.61  Female     No  Sun  Dinner     4
```

#### Sorting by Values

Sorting in Stata is accomplished via ``sort``

``` bash
sort sex total_bill
```

pandas objects have a [DataFrame.sort_values()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.sort_values.html#pandas.DataFrame.sort_values) method, which takes a list of columns to sort by.

``` python
In [25]: tips = tips.sort_values(['sex', 'total_bill'])

In [26]: tips.head()
Out[26]: 
     total_bill   tip     sex smoker   day    time  size
67         1.07  1.00  Female    Yes   Sat  Dinner     1
92         3.75  1.00  Female    Yes   Fri  Dinner     2
111        5.25  1.00  Female     No   Sat  Dinner     1
145        6.35  1.50  Female     No  Thur   Lunch     2
135        6.51  1.25  Female     No  Thur   Lunch     2
```

### String Processing

#### Finding Length of String

Stata determines the length of a character string with the ``strlen()`` and ``ustrlen()`` functions for ASCII and Unicode strings, respectively.

``` bash
generate strlen_time = strlen(time)
generate ustrlen_time = ustrlen(time)
```

Python determines the length of a character string with the len function. In Python 3, all strings are Unicode strings. ``len`` includes trailing blanks. Use ``len`` and ``rstrip`` to exclude trailing blanks.

``` python
In [27]: tips['time'].str.len().head()
Out[27]: 
67     6
92     6
111    6
145    5
135    5
Name: time, dtype: int64

In [28]: tips['time'].str.rstrip().str.len().head()
Out[28]: 
67     6
92     6
111    6
145    5
135    5
Name: time, dtype: int64
```

#### Finding Position of Substring

Stata determines the position of a character in a string with the ``strpos()`` function. This takes the string defined by the first argument and searches for the first position of the substring you supply as the second argument.

``` bash
generate str_position = strpos(sex, "ale")
```

Python determines the position of a character in a string with the ``find()`` function. ``find`` searches for the first position of the substring. If the substring is found, the function returns its position. Keep in mind that Python indexes are zero-based and the function will return -1 if it fails to find the substring.

``` python
In [29]: tips['sex'].str.find("ale").head()
Out[29]: 
67     3
92     3
111    3
145    3
135    3
Name: sex, dtype: int64
```

#### Extracting Substring by Position

Stata extracts a substring from a string based on its position with the ``substr()`` function.

``` bash
generate short_sex = substr(sex, 1, 1)
```

With pandas you can use ``[]`` notation to extract a substring from a string by position locations. Keep in mind that Python indexes are zero-based.

``` python
In [30]: tips['sex'].str[0:1].head()
Out[30]: 
67     F
92     F
111    F
145    F
135    F
Name: sex, dtype: object
```

#### Extracting nth Word

The Stata ``word()`` function returns the nth word from a string. The first argument is the string you want to parse and the second argument specifies which word you want to extract.

``` bash
clear
input str20 string
"John Smith"
"Jane Cook"
end

generate first_name = word(name, 1)
generate last_name = word(name, -1)
```

Python extracts a substring from a string based on its text by using regular expressions. There are much more powerful approaches, but this just shows a simple approach.

``` python
In [31]: firstlast = pd.DataFrame({'string': ['John Smith', 'Jane Cook']})

In [32]: firstlast['First_Name'] = firstlast['string'].str.split(" ", expand=True)[0]

In [33]: firstlast['Last_Name'] = firstlast['string'].str.rsplit(" ", expand=True)[0]

In [34]: firstlast
Out[34]: 
       string First_Name Last_Name
0  John Smith       John      John
1   Jane Cook       Jane      Jane
```

#### Changing Case

The Stata ``strupper()``, ``strlower()``, ``strproper()``, ``ustrupper()``, ``ustrlower()``, and ``ustrtitle()`` functions change the case of ASCII and Unicode strings, respectively.

``` bash
clear
input str20 string
"John Smith"
"Jane Cook"
end

generate upper = strupper(string)
generate lower = strlower(string)
generate title = strproper(string)
list
```

The equivalent Python functions are ``upper``, ``lower``, and ``title``.

``` python
In [35]: firstlast = pd.DataFrame({'string': ['John Smith', 'Jane Cook']})

In [36]: firstlast['upper'] = firstlast['string'].str.upper()

In [37]: firstlast['lower'] = firstlast['string'].str.lower()

In [38]: firstlast['title'] = firstlast['string'].str.title()

In [39]: firstlast
Out[39]: 
       string       upper       lower       title
0  John Smith  JOHN SMITH  john smith  John Smith
1   Jane Cook   JANE COOK   jane cook   Jane Cook
```

### Merging

The following tables will be used in the merge examples

``` python
In [40]: df1 = pd.DataFrame({'key': ['A', 'B', 'C', 'D'],
   ....:                     'value': np.random.randn(4)})
   ....: 

In [41]: df1
Out[41]: 
  key     value
0   A  0.469112
1   B -0.282863
2   C -1.509059
3   D -1.135632

In [42]: df2 = pd.DataFrame({'key': ['B', 'D', 'D', 'E'],
   ....:                     'value': np.random.randn(4)})
   ....: 

In [43]: df2
Out[43]: 
  key     value
0   B  1.212112
1   D -0.173215
2   D  0.119209
3   E -1.044236
```

In Stata, to perform a merge, one data set must be in memory and the other must be referenced as a file name on disk. In contrast, Python must have both ``DataFrames`` already in memory.

By default, Stata performs an outer join, where all observations from both data sets are left in memory after the merge. One can keep only observations from the initial data set, the merged data set, or the intersection of the two by using the values created in the ``_merge`` variable.

``` bash
* First create df2 and save to disk
clear
input str1 key
B
D
D
E
end
generate value = rnormal()
save df2.dta

* Now create df1 in memory
clear
input str1 key
A
B
C
D
end
generate value = rnormal()

preserve

* Left join
merge 1:n key using df2.dta
keep if _merge == 1

* Right join
restore, preserve
merge 1:n key using df2.dta
keep if _merge == 2

* Inner join
restore, preserve
merge 1:n key using df2.dta
keep if _merge == 3

* Outer join
restore
merge 1:n key using df2.dta
```

pandas DataFrames have a [DataFrame.merge()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.merge.html#pandas.DataFrame.merge) method, which provides similar functionality. Note that different join types are accomplished via the ``how`` keyword.

``` python
In [44]: inner_join = df1.merge(df2, on=['key'], how='inner')

In [45]: inner_join
Out[45]: 
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209

In [46]: left_join = df1.merge(df2, on=['key'], how='left')

In [47]: left_join
Out[47]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209

In [48]: right_join = df1.merge(df2, on=['key'], how='right')

In [49]: right_join
Out[49]: 
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209
3   E       NaN -1.044236

In [50]: outer_join = df1.merge(df2, on=['key'], how='outer')

In [51]: outer_join
Out[51]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E       NaN -1.044236
```

### Missing Data

Like Stata, pandas has a representation for missing data – the special float value ``NaN`` (not a number). Many of the semantics are the same; for example missing data propagates through numeric operations, and is ignored by default for aggregations.

``` python
In [52]: outer_join
Out[52]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E       NaN -1.044236

In [53]: outer_join['value_x'] + outer_join['value_y']
Out[53]: 
0         NaN
1    0.929249
2         NaN
3   -1.308847
4   -1.016424
5         NaN
dtype: float64

In [54]: outer_join['value_x'].sum()
Out[54]: -3.5940742896293765
```

One difference is that missing data cannot be compared to its sentinel value. For example, in Stata you could do this to filter missing values.

``` bash
* Keep missing values
list if value_x == .
* Keep non-missing values
list if value_x != .
```

This doesn’t work in pandas. Instead, the ``pd.isna()`` or ``pd.notna()`` functions should be used for comparisons.

``` python
In [55]: outer_join[pd.isna(outer_join['value_x'])]
Out[55]: 
  key  value_x   value_y
5   E      NaN -1.044236

In [56]: outer_join[pd.notna(outer_join['value_x'])]
Out[56]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
```

Pandas also provides a variety of methods to work with missing data – some of which would be challenging to express in Stata. For example, there are methods to drop all rows with any missing values, replacing missing values with a specified value, like the mean, or forward filling from previous rows. See the [missing data documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html#missing-data) for more.

``` python
# Drop rows with any missing value
In [57]: outer_join.dropna()
Out[57]: 
  key   value_x   value_y
1   B -0.282863  1.212112
3   D -1.135632 -0.173215
4   D -1.135632  0.119209

# Fill forwards
In [58]: outer_join.fillna(method='ffill')
Out[58]: 
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059  1.212112
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E -1.135632 -1.044236

# Impute missing values with the mean
In [59]: outer_join['value_x'].fillna(outer_join['value_x'].mean())
Out[59]: 
0    0.469112
1   -0.282863
2   -1.509059
3   -1.135632
4   -1.135632
5   -0.718815
Name: value_x, dtype: float64
```

### GroupBy

#### Aggregation

Stata’s ``collapse`` can be used to group by one or more key variables and compute aggregations on numeric columns.

``` bash
collapse (sum) total_bill tip, by(sex smoker)
```

pandas provides a flexible ``groupby`` mechanism that allows similar aggregations. See the [groupby documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/groupby.html#groupby) for more details and examples.

``` python
In [60]: tips_summed = tips.groupby(['sex', 'smoker'])['total_bill', 'tip'].sum()

In [61]: tips_summed.head()
Out[61]: 
               total_bill     tip
sex    smoker                    
Female No          869.68  149.77
       Yes         527.27   96.74
Male   No         1725.75  302.00
       Yes        1217.07  183.07
```

#### Transformation

In Stata, if the group aggregations need to be used with the original data set, one would usually use ``bysort`` with ``egen()``. For example, to subtract the mean for each observation by smoker group.

``` bash
bysort sex smoker: egen group_bill = mean(total_bill)
generate adj_total_bill = total_bill - group_bill
```

pandas ``groubpy`` provides a ``transform`` mechanism that allows these type of operations to be succinctly expressed in one operation.

``` python
In [62]: gb = tips.groupby('smoker')['total_bill']

In [63]: tips['adj_total_bill'] = tips['total_bill'] - gb.transform('mean')

In [64]: tips.head()
Out[64]: 
     total_bill   tip     sex smoker   day    time  size  adj_total_bill
67         1.07  1.00  Female    Yes   Sat  Dinner     1      -17.686344
92         3.75  1.00  Female    Yes   Fri  Dinner     2      -15.006344
111        5.25  1.00  Female     No   Sat  Dinner     1      -11.938278
145        6.35  1.50  Female     No  Thur   Lunch     2      -10.838278
135        6.51  1.25  Female     No  Thur   Lunch     2      -10.678278
```

#### By Group Processing

In addition to aggregation, pandas ``groupby`` can be used to replicate most other ``bysort`` processing from Stata. For example, the following example lists the first observation in the current sort order by sex/smoker group.

``` bash
bysort sex smoker: list if _n == 1
```

In pandas this would be written as:

``` python
In [65]: tips.groupby(['sex', 'smoker']).first()
Out[65]: 
               total_bill   tip   day    time  size  adj_total_bill
sex    smoker                                                      
Female No            5.25  1.00   Sat  Dinner     1      -11.938278
       Yes           1.07  1.00   Sat  Dinner     1      -17.686344
Male   No            5.51  2.00  Thur   Lunch     2      -11.678278
       Yes           5.25  5.15   Sun  Dinner     2      -13.506344
```

### Other Considerations

#### Disk vs Memory

Pandas and Stata both operate exclusively in memory. This means that the size of data able to be loaded in pandas is limited by your machine’s memory. If out of core processing is needed, one possibility is the [dask.dataframe](http://dask.pydata.org/en/latest/dataframe.html) library, which provides a subset of pandas functionality for an on-disk ``DataFrame``.
