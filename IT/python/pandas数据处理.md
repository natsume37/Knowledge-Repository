# pabdas 基础
## series -一维数组

> 它是由由一组任意类型的数据以及一组与之相关的数据标签（即索引）组成

```python
import pandas as pd  
  
print(pd.Series([2, 3, 4, 5, 6]))
```

运行结果
```python
0    2
1    3
2    4
3    5
4    6
dtype: int64
```

> 左边的是数据的标签，默认从 0 开始依次递增。右边是对应的数据，最后一行表明了数据类型。

### 自定义数据标签 index

```python
print(pd.Series([2, 3, 4, 5, 6], index=[3,4,7,3,6]))
```

```python
3    2
4    3
7    4
3    5
6    6
dtype: int64
```

字典形式

```python
print(pd.Series({'a': 2, 'b': 4, 'c': 6, 'd': 8}))
```

### 获取 Series 数据

```python
s1 = pd.Series([2, 4, 6, 8])
s2 = pd.Series({'a': 2, 'b': 4, 'c': 6, 'd': 8})

print(s1[0])
# 输出：2

print(s2['b'])
# 输出：4
```

### 数据对齐

```
s1 = pd.Series({'辣条': 14, '面包': 7, '可乐': 8, '烤肠': 10})  
s2 = pd.Series({'辣条': 20, '烤肠': 6, '可乐': 13, '面包': 3})  
print(s1 + s2)
```

```python
可乐    21
烤肠    16
辣条    34
面包    10
dtype: int64
```

> pandas 自动帮我们将相同数据标签的数据进行了计算，这就是数据对齐

对于对于没有对应的单条数据
```python
s1 = pd.Series({'辣条': 14, '面包': 7, '可乐': 8, '烤肠': 10})
s2 = pd.Series({'辣条': 20, '面包': 3, '雪碧': 13, '泡面': 6})
print(s1 + s2)
```
```python
可乐     NaN
泡面     NaN
烤肠     NaN
辣条    34.0
雪碧     NaN
面包    10.0
dtype: float64
```
> 对于数据标签不相同的数据，运算后结果是 **NaN**。NaN 是 Not a Number（不是一个数字）的缩写，因为其中一个 Series 中没有对应数据标签的数据，无法进行计算，因此返回了 NaN。

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/0923132023092313470230691787019.png)

#### Series.add()

> 问题解决：让没有对应数据的设置默认值为 0 进行数据对齐计算。

```pyhton
s1 = pd.Series({'辣条': 14, '面包': 7, '可乐': 8, '烤肠': 10})
s2 = pd.Series({'辣条': 20, '面包': 3, '雪碧': 13, '泡面': 6})
print(s1.add(s2, fill_value=0))  # fill_value 为数据缺失时的默认值
```

```python
可乐     8.0
泡面     6.0
烤肠    10.0
辣条    34.0
雪碧    13.0
面包    10.0
dtype: float64
```

> `add()` 方法对应的是加法，数学中的四则运算在 pandas 中都有一一对应的方法，它们的用法都是类似的。具体对应关系如下图所示：

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/092313202309231352519e1ea70a1a0.png)

## DataFrame

Series 是一维数据，而 DataFrame 是二维数据。什么意思呢？你可以把 DataFrame 想象成一个表格，表格有行和列这两个维度，所以是二维数据。

> 构建 DataFrame 的办法有很多，最常用的一种是传入一个由等长列表组成的字典。即字典里每个值都是列表，且它们的`长度必需相等`。

```python
pd.DataFrame({'辣条': [14, 20], '面包': [7, 3], '可乐': [8, 13], '烤肠': [10, 6]})
```

```python
data = {  
  '辣条': [14, 20],  
  '面包': [7, 3],  
  '可乐': [8, 13],  
  '烤肠': [10, 6]  
}  
df = pd.DataFrame(data, index=['2020-01-01', '2020-01-02'])  
print(df)
```
```python
             辣条   面包  可乐   烤肠
2020-01-01    14     7     8    10
2020-01-02    20     3    13     6
```

#### 列的增删查改

基础数据
```python
df = pd.DataFrame({'辣条': [14, 20], '面包': [7, 3], '可乐': [8, 13], '烤肠': [10, 6]})
```

#### 查
只查看可乐的销售量：
```python
print(df['可乐'])
```

```python
0     8
1    13
Name: 可乐, dtype: int64
```

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/09231420230923140887472e7091236.png)

查看多列数据

> 只需要把多列数据放入一个列表中即可

```python
print(df[['可乐', '辣条']])
```

### 修改列

> 直接对列进行赋值操作

```python
df['可乐'] = [18, 23]
print(df)
```

修改后的数据：

```python
    辣条   面包   可乐  烤肠
0    14     7    18    10
1    20     3    23     6
```

### 新增列

> 如果想要新增一列同样也非常的简单，对表格中不存在的列直接赋值就能添加新的列了。

```python
df['糖果'] = [3, 5]
print(df)
```

```python
    辣条   面包  可乐   烤肠   糖果
0    14     7     8    10     3
1    20     3    13     6     5
```

### 删除列

> 删除列和字典删除元素不一样，需要用到 `drop()` 方法。我们先来看一下用法：

```python
df.drop('面包', axis=1, inplace=True)
print(df)
# 或者 print(df.drop('面包', axis=1))
```

> `drop()` 方法的第一个参数是要删除的列名或索引。axis 表示针对行或列进行删除，axis = 0 表示删除对应的行，axis = 1 表示删除对应的列，axis 默认为 0。

> 最后的 inplace = True 表示直接修改原数据，否则 `drop()` 方法只是返回删除后的表格，对原表格没有影响。因此上面两种写法的结果是一样的。

### 总结

```python
data = {  
    '辣条': [14, 20],  
    '面包': [7, 3],  
    '可乐': [8, 13],  
    '烤肠': [10, 6]  
}  
df = pd.DataFrame(data, index=['2020-01-01', '2020-01-02'])  
  
# 新增  
df["新增列"] = [12, 18]  
  
# 删除  
df.drop("面包",axis=1, inplace=True)  
  
# 改  
df['烤肠'] = [99,33]  
  
# 查  
print(df[["烤肠", "辣条"]])  
  
print(df)
```

```python
            烤肠  辣条
2020-01-01    99    14
2020-01-02    33    20
            辣条  可乐  烤肠  新增列
2020-01-01    14     8    99      12
2020-01-02    20    13    33      18
```


## pandas 文件操作

### 导入文件
```python
df = pd.read_csv('2019年销售数据.csv')
print(type(df))
# 输出：<class 'pandas.core.frame.DataFrame'>
```

### 查看数据前 5 行数据

```python
print(df.head())

# df.head(2) 导入前两行数据
```

```python
   销售员 团队    第一季度   第二季度   第三季度    第四季度
0   刘一    A      6324      5621      6069      6005
1   陈二    B      4508      3391      5933      5002
2   张三    C      3426      3549      5872      5759
3   李四    D      2104      3939      3285      3461
4   王五    A      4830      5763      2923      4033
```

其他操作
- df.tail() 查看末尾数据
- info() 查看表格的大致信息
	![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/0923142023092314265539fbf69c614.png)
- df.describe()  数据的统计摘要
	```python
	           第一季度       第二季度      第三季度      第四季度
count    20.000000    20.000000    20.000000    20.000000
mean   3969.150000  4252.250000  4309.300000  4284.700000
std    1518.736084  1352.492275  1646.188871  1286.032377
min    1501.000000  1553.000000  1272.000000  2091.000000
25%    2760.000000  3412.750000  3194.500000  3469.250000
50%    4162.000000  4392.500000  4536.000000  4120.500000
75%    4888.500000  5412.250000  5818.000000  5202.250000
max    6924.000000  6176.000000  6499.000000  6683.000000
```
生成的摘要从上往下分别表示数量、平均数、标准差、最小值、25% 50% 75% 位置的值和最大值。

### 常用函数

- max()
- min()
- mean()
- sum()

### 排序操作 
```python
sort_values("排序列明",ascending=False, inplace=True)
第二个参数：默认升序，inplace=  是否修改原数据
```

## 文件的保存

```python
df.to_csv('2019年销售数据.csv', index=False)
```

`index=False` 是因为不希望将最左侧的索引保存到文件中

```python
import pandas as pd

df = pd.read_csv('2019年销售数据.csv')  # 导入 csv 文件
df['总和'] = df['第一季度'] + df['第二季度'] + df['第三季度'] + df['第四季度']  # 汇总
df.sort_values('总和', ascending=False, inplace=True)  # 排序
df.to_csv('2019年销售数据.csv', index=False)  # 导出 csv 文件
```

- read_excel
- to_excel
- read_csv
- to_csv










## 字符设置

因为我们的表格中有中文，中文占用的字符和英文、数字占用的字符不一样，因此需要调用 `pd.set_option()` 使表格对齐显示。如果你是使用 Jupyter 来运行代码的，Jupyter 会自动渲染出一个表格，则无需这个设置。

```python
pd.set_option('display.unicode.ambiguous_as_wide', True)
pd.set_option('display.unicode.east_asian_width', True)
```
示例代码：
```python
pd.set_option('display.unicode.ambiguous_as_wide', True)  
pd.set_option('display.unicode.east_asian_width', True)  
  
df = pd.DataFrame({'辣条': [14, 20], '面包': [7, 3], '可乐': [8, 13], '烤肠': [10, 6]})  
print(df)
```

```python
   辣条  面包  可乐  烤肠
0    14     7     8    10
1    20     3    13     6
```



# pandas 进阶

## 数据筛选

案例：筛选出那些`总销售额低于平均值`的销售员

```python
df[df['总和'] < df['总和'].mean()]
```

> [中括号里填写筛选条件]  符合条件为 True、不符合条件的返回 Fale、最好它会删除所有的 false 的行

### & |  与或运算

```python
# 筛选出总和大于 10000 且小于 12000 的
df[(df['总和'] > 10000) & (df['总和'] < 12000)]

# 筛选出总和小于 5000 或大于 12000 的
df[(df['总和'] < 5000) | (df['总和'] > 12000)]
```

> 多个运算条件、条件外面要加小括号

## 打标签

```python
cut()
pd.cut(df["lie"], bins=[区间]， label=["给区间打的标签"])

pd.cut(df['总和'], bins=[0, 1000, 2000, 3000], labels=['不合格', '良好', '优秀'])
```

> 上面这段代码的意思是，按照 **(0, 1000]**、**(1000, 2000]** 和 **(2000, 3000]** 的数值区间，把数据分为不合格、良好和优秀三组

第一个参数是要分类的列；第二个参数 `bins` 是分类的方式，即分类区间，默认是左开右闭，设置 `bins=[0, 1000, 2000, 3000]`，那对应的分类区间就是 **(0, 1000]**、**(1000, 2000]** 和 **(2000, 3000]**。如果你想要左闭右开的方式，可以再添加一个参数 `right=False`。

最后的参数 `labels` 分别对应了这三组的标签名。即 **(0, 1000]** 表示不合格，**(1000, 2000]** 表示良好，**(2000, 3000]** 表示优秀。

```python
df['绩效'] = pd.cut(df['总和'], bins=[0, df['总和'].mean() * 0.9, df['总和'].mean() * 1.2, df['总和'].max()], labels=['不合格', '良好', '优秀'])
```

## 行的增删改查

- `loc[]`  基于索引的查询行
> 注意 ： `loc[]` 并不是一个方法、而是一个字典

```python
import pandas as pd

df = pd.DataFrame({'辣条': [14, 20], '面包': [7, 3], '可乐': [8, 13], '烤肠': [10, 6]})
```

数据集
```python
    辣条   面包  可乐   烤肠
0    14     7     8    10
1    20     3    13     6
```

```python
print(df.loc[0])
```

```python
辣条    14
面包     7
可乐     8
烤肠    10
Name: 0, dtype: int64
```

```python
print(df.loc[0, '辣条'])
```

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/09241420230924145251256dc9574a3.png)

## 分片操作

```python
# 行分片
print(df.loc[0:1, '辣条'])

# 列分片
print(df.loc[0, '辣条':'可乐'])

# 同时分片
print(df.loc[0:1, '辣条':'可乐'])
```

> 和 python 的数组分片不同、这是都左右都包含

你也可以通过省略冒号前后的内容来实现全选，例如：`df.loc[:, '辣条':'可乐']`

`loc` 同时也支持布尔索引来进行数据的筛选，比如获取辣条销量大于 15 的数据：

```python
df.loc[df['辣条'] > 15]
```

上面的写法等价于 `df[df['辣条'] > 15]`，不同之处在于 `loc` 还能通过第二个参数筛选出只想查看的列，比如：

```python
print(df.loc[df['辣条'] > 15, ['辣条', '面包']])
```

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/092414202309241456662d03f179765.png)

除了比较常用的 `loc` 之外，我们还能使用 `iloc`。用法和 `loc` 一样，区别在于 `loc` 使用的参数是索引，而 `iloc` 的参数是位置，即第几行。

因此，在不指定索引的情况下，`loc` 和 `iloc` 的效果是一样的。但当单独指定了索引，我们想获取前 3 行数据时可以像下面这样写：

```python
import pandas as pd

data = {
  '辣条': [14, 20, 12, 15, 17],
  '面包': [7, 3, 8, 3, 9],
  '可乐': [8, 13, 23, 12, 19],
  '烤肠': [10, 6, 21, 24, 18]
}
df = pd.DataFrame(data, index=['2020-01-01', '2020-01-02', '2020-01-03', '2020-01-04', '2020-01-05'])
print(df.iloc[:3])  # :3 表示 0、1、2 前三个
```

索引变成了日期，想要按位置获取数据的话只能使用 `iloc`，这时如果使用 `df.loc[:3]` 将会报错。上面的代码运行结果如下：

```python
             辣条   面包  可乐   烤肠
2020-01-01    14     7     8    10
2020-01-02    20     3    13     6
2020-01-03    12     8    23    21
```

> 注意，`iloc` 的分片和 Python 的列表分片一样，要和 `loc` 的分片规则区分开来。

## 修改行

假设你要修改一行的数据
- 赋值为一个数字
- 赋值为长度和列数相等的 `列表`

```python
df.loc[0] = 1  # 第一行都改成 1
print(df)


    辣条   面包  可乐   烤肠
0     1     1     1     1
1    20     3    13     6
```

```python 
# 按顺序修改成 1 2 3 4
df.loc[0] = [1, 2, 3, 4]
print(df)


    辣条   面包  可乐   烤肠
0     1     2     3     4
1    20     3    13     6
```

当然你也可以使用例如 `df.loc[0, '辣条'] = 23` 定位到行和列来修改特定的数据。

## 新增行

和新增列类似只需传入表格中不存在的索引即可

```python
# 添加第三行，全为 1
df.loc[2] = 1
# 添加第四行，分别为 1 2 3 4
df.loc[3] = [1, 2, 3, 4]
print(df)
```

```python
    辣条   面包  可乐   烤肠
0    14     7     8    10
1    20     3    13     6
2     1     1     1     1
3     1     2     3     4
```

> 注意：在新增行时，是不能使用 `iloc` 传入索引的哦

## 删除行

删除行和删除列一样，都是使用 `drop()` 方法。删除列的使用传入了 `axis=1` 表示对列进行删除，`axis` 默认为 0，因此删除行时省略 `axis` 参数即可。

```python
df.drop(0, inplace=True)  # 删除第一行
print(df)
# 或者 print(df.drop(0))
```

## 数据分组

`groupby()`

案例演示: 请你筛选出 `数据.csv` 中团队第一季度销售额低于平均值的团队，并只展示出第一季度的数据。

```csv
销售员,团队,第一季度,第二季度,第三季度,第四季度,总和  
刘一,A,6324,5621,6069,6005,24019  
陈二,B,4508,3391,5933,5002,18834  
张三,C,3426,3549,5872,5759,18606  
李四,D,2104,3939,3285,3461,12789  
王五,A,4830,5763,2923,4033,17549  
赵六,B,2545,3747,2232,3472,11996  
孙七,C,3964,5053,6499,5056,20572  
周八,D,5584,6176,5405,6683,23848  
吴九,A,2769,5440,5724,4338,18271  
郑十,B,4935,2617,2424,3833,13809  
冯十一,C,4360,4846,4355,5182,18743  
蒋十二,D,4509,3673,4717,3489,16388  
毕十三,A,4873,5403,3889,2567,16732  
朱十四,B,3338,4965,5800,4061,18164  
秦十五,C,1501,2238,1272,2265,7276  
许十六,D,5369,3420,5169,5790,19748  
钱十七,A,6924,5912,5907,5263,24006  
沈十八,B,2733,2649,3402,3164,11948  
韩十九,C,1519,1553,1411,2091,6574  
杨二十,D,3268,5090,3898,4180,16436
```

```python
df = pd.read_csv("data/数据.csv")  
grouped = df.groupby("团队").sum()  
data = grouped.loc[grouped["第一季度"] < grouped["总和"].mean(), "第一季度"]  
print(data)
```

# pandas 高阶

## 表格合并

将表格内容从上往下进行叠加。这在 pandas 中使用 `concat()` 方法

```python
pd.concat([df1, df2])
```

完整代码：
```python
import pandas as pd

df_a = pd.read_csv('2019年团队A销售数据.csv')
df_b = pd.read_csv('2019年团队B销售数据.csv')
df_c = pd.read_csv('2019年团队C销售数据.csv')
df_d = pd.read_csv('2019年团队D销售数据.csv')

df = pd.concat([df_a, df_b, df_c, df_d])
print(type(df))
# 输出：<class 'pandas.core.frame.DataFrame'>
```

表格的横向合并

将这两个表格进行横向合并最简单的方法是 `pd.merge(表一, 表二)`

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/092415202309241537832182c672465.png)

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/09241520230924153729956840b1f18.png)

上面的例子是 `merge()` 方法最简单的情况，只传入两个需要横向合并的表格作为参数。`merge()` 方法还有两个比较重要的参数分别是 `on` 和 `how`。

参数 `on` 表明用于合并的列名，可以使用列表指定多个列名，这些列名必须同时存在于两个表格当中。如果没有指定参数 `on`，那么 pandas 会自动将两个表格都有的列名作为参数 `on` 的值。

在上面的列子中参数 `on` 默认就是销售员，合并时 pandas 会将销售员这列值一样的行进行横向合并，并不是简单地按顺序合并，因此两张表的排列顺序不一样也能正确地进行合并。

如果、在第三季度、刘已离职了、那么合并后

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/09241520230924153998087245a6a90.png)

可以看到，合并后刘一的数据被剔除了，这其实是参数 `how` 导致的。参数 `how` 指定了合并的方式，总共有 `left`、`right`、`outer` 和 `inner` 四种方式可选，默认为 `inner`。

-  `inner`（内连接）表示只保留参数 `on` 指定的列（上面的例子中是销售员）中两个表格都出现的部分。因为刘一只在表一中出现，因此合并后就剔除了刘一这行数据。

-  `outer`（外连接）和 `inner` 相反，它会保留两个表格中所有的数据，数据缺失部分以 NaN 填充。即以 NaN 填充上面的例子中刘一在第三四季度的数据。

> 注意，如果某列中用 NaN 补全了空位，那这一列数据就会变为浮点型数据。

-  `left`（左连接）表示将表二合并到表一中，具体指保留表一的全部数据，将表二中两表中共同的销售员数据进行合并，剔除表二中独有的数据。缺失数据同样也是用 NaN 填充。`right`（右连接）正好相反，表示将表一合并到表二中。

- `right` 与 left 相反

## 数据清洗（缺失值、格式、数据错误）

### 缺失值处理
- 删除缺失行
- 为缺失行赋值

df.dropna(how='all',thresh=N,subset=[...])

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/0924152023092415574273e8b411dca.png)


假设我们认为有缺失数据的那一行数据都不可信，需要将其删除，只需一行代码即可:
```python
df.dropna()
```

`dropna()` 方法的作用是删除所有包含 `NaN` 的行，执行后上表中数据缺失的那 2 行数据就会被删除。

> 提示：NaN 是 Not a Number，在 pandas 中用于表示数据缺失。

`dropna()` 方法同样也是返回删除后的表格，不会对原表格有影响。如需直接在原表格上删除，传入 `inplace=True` 即可。

```python
df.dropna(how='all')
```

`thresh` 参数来控制当一行中非空值数量小于多少时才删除此行，比如当一行数据中，非空值数量小于 5 个时删除这一行，代码就可以这样写：

```python
df.dropna(thresh=5)
```

当我们只想分析价格时，其他数据的缺失对我们的分析没有影响。这时可以通过 `subset` 参数决定哪几列有数据缺失时才进行删除。比如，当书名和价格这两列有数据缺失时可以这样写：

```python
df.dropna(subset=['书名', '价格'])
```

### 数据填充

除了删除缺失数据行，有些情况下我们还能为缺失数据赋值。比如价格缺失了，我们可以将缺失价格设置为 0，方法如下：

```python
# 填充 0
df['价格'].fillna(0)

# 填充平均值
df['价格'].fillna(df['价格'].mean())

# 首先是选取有数据缺失的列，然后调用 `fillna()` 方法，该方法会将传入的参数填充到该列所有缺失的数据中。
```

## 数据格式统一

### pandas 字符串操作

```python
df['价格'].str.replace('元', '')
```

先访问 `str` 属性，再调用里面的字符串方法。Python 里的字符串方法，比如 `replace()`、`upper()`、`lower()`、`split()` 等等在其中。

因为字符串不可变、所以为了修改原数据需要替换修改后的数据
```python
df['价格'] = df['价格'].str.replace('元', '')
```

### 格式转换

这样处理完之后看上去是数字，但实际上类型还是字符串，我们需要使用 `astype()` 方法进行类型转换。价格是小数，所以我们将其转换成浮点数类型（float）

```python
df['价格'] = df['价格'].str.replace('元', '').astype('float')
```

> 转化为浮点型

## 小数点控制

`round()`

出来的平均价格是 `33.090161`，然而我们并不需要精确到小数点后这么多位。因此我们可以使用 `round()` 方法进行四舍五入，参数是要保留的小数位数。所以最后的代码如下：

```python
df['价格'].fillna(df['价格'].mean().round(1), inplace=True)  # 保留 1 位小数
```

案例代码：
```python
df = pd.read_csv('https://media-zip1.baydn.com/storage_media_zip/srfeae/dc3fa2c70032c4f4dfd7d878d79eb4da.41767dfc9dd1646b2a9f71527db2125f.csv')  
  
df["评分"] = df["评分"].str.replace("分","").astype('float')  
df["评分"].fillna(df['评分'].mean().round(1),inplace=True)  
print(df.head(10))
```

## 重复数据处理

-  `drop_duplicates()`
故意创建一个重复数据

```pyhon
df = pd.DataFrame({'用户名': ['刘一', '陈二', '刘一', '张三'], '交易金额': [25.8, 15.5, 56.3, 46.2]})
repeat = pd.concat([df, df])
print(repeat)
```

```python
   用户名   交易金额
0   刘一      25.8
1   陈二      15.5
2   刘一      56.3
3   张三      46.2
0   刘一      25.8
1   陈二      15.5
2   刘一      56.3
3   张三      46.2
```

- `drop_duplicates()`  删除完全重复的行，即每一列的数据都完全相同的行。
```python
print(repeat.drop_duplicates())
```

```python
   用户名   交易金额
0   刘一      25.8
1   陈二      15.5
2   刘一      56.3
3   张三      46.2
```

参数：
- `subset` 指定按列去重，即只要这一列的数据重复就会删除重复的内容
- `keep='last'`  ，按照用户名进行去重刘一的两条记录只剩一条了。去重默认是保留第一条不重复的数据，如果你想保留最后一条不重复的数据，可以传入 keep='last'。

## 查询不重复的数据

只要在对应的列上调用 `unique()` 方法即可。

```python
print(repeat['用户名'].unique())
# 输出：['刘一' '陈二' '张三']

print(len(repeat['用户名'].unique()))
# 输出：3

print(repeat['用户名'].nunique())
# 输出：3
```

- unique()  查看不重复的内容
- nunique()查看不重复的个数

## apply 方法

`apply()` 方法的第一个参数是一个函数，当我们传入函数后，`apply()` 方法会将该函数应用到表格列里的每一个数据中，并将表格数据作为参数传给函数。

```python
df['价格'] = df['价格'].str.replace('元', '').astype('float')

# 等价于
def format_price(x):
  return float(x.replace('元', ''))

df['价格'] = df['价格'].apply(format_price)

# 等价于
df['价格'] = df['价格'].apply(lambda x: float(x.replace('元', '')))
```

`apply()` 方法做的事其实就是将每个数据都传入函数中，再将函数处理后的结果替换掉原来的数据。

## agg()
- 字典方法

```python
df_rfm = df.groupby('用户名').agg({
  '订单日期': lambda x: (pd.to_datetime('2019-12-31') - x.max()).days,  # 计算 R
  '用户名': lambda x: len(x),  # 计算 F
  '订单金额': lambda x: x.sum()  # 计算 M
})
```

> `agg()` 方法是 pandas 中的聚合方法，可以针对多列同时进行操作，我们给 agg() 方法传入一个字典，字典的键是要进行操作的列名，值为一个函数。函数的参数是每一列中的数据，函数的返回值将替换原来的值。你可以将 agg() 方法理解为同时针对多列进行不同操作的 `apply()` 方法。

## len() 获取数据长度
## rename 更改列名
```python
df_rfm.rename(columns={'订单日期': 'R', '用户名': 'F', '订单金额': 'M'}, inplace=True)
```

```python
import pandas as pd  
  
pd.set_option('display.unicode.ambiguous_as_wide', True)  
pd.set_option('display.unicode.east_asian_width', True)  
  
# 读取订单表格数据  
df = pd.read_csv(  
    'https://media-zip1.baydn.com/storage_media_zip/srfeae/bf6dc7d814c520c60e5e632d281f14a4.ba163c25251bd44b74bde1bb4af7abdc.csv')  
  
df['订单日期'] = pd.to_datetime(df['订单日期'])  
df_rfm = df.groupby("用户名").agg({  
    "订单日期": lambda x: (pd.to_datetime('2019-12-31') - x.max()).days,  
    "用户名": lambda x: len(x),  
    "订单金额": lambda x: x.sum()  
})  
  
df_rfm.rename(columns={"订单日期": "R", "用户名": "F", "订单金额": "M"}, inplace=True)  
print(df_rfm.head(5))
```

```python
          R  F     M
用户名              
丁元     57  1  1873
丁发    180  1  4151
丁和     25  3  1109
丁妹     37  2  2271
丁学     41  5  2291
```

