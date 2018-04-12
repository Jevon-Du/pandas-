---
title: DataFrame 数据合并
keywords: 添加，合并，连接
time: 2018-2-6
---

>最近需要合并两张数据表，由于没仔细研究过pandas中合并数据表的函数，浪费了一些时间还没有什么收获，问题也没很好的解决，所以静下来好好认识一下pandas对于数据合并的操作。
>padnas文档链接：http://pandas.pydata.org/pandas-docs/stable/merging.html

#一、合并，连接 (Merge, join, and concatenate)
在需要连接/合并数据的情况下，pandas提供了丰富的功能，可以方便地将Series, DataFrame和Panel对象与索引和相关的代数功能的各种设置逻辑组合在一起。

##1. 拼接对象 (Concatenating objects)
`concat`函数（在pandas命名空间中）完成了沿轴执行拼接操作的所有繁重工作，同时对其他轴上的索引（如果有）执行可选集逻辑（并集或交集）。 请注意，我说“如果有的话”，因为对Series只有一个可能的轴将其拼接。

**`concat`函数沿着一条轴将多个对象堆叠到一起，不会去重，只是单纯的拼接。并且每次返回完整的数据拷贝，所以不适合操作大数据量以及反复调用**

![concat_basic](_image/merging_concat_basic.png)

就像它在ndarrays上的兄弟`numpy.concatenate`一样，`pandas.concat`获取同类型对象的列表或字典，并将它们与一些可配置的处理“与其他轴做什么”进行拼接：

```python
pd.concat(objs, axis=0, join='outer', join_axes=None, ignore_index=False,
          keys=None, levels=None, names=None, verify_integrity=False,
          copy=True)
```

- objs : 一段序列或者Series，DataFrame或Panel对象的映射。如果传递了一个字典，那么排序后的键将被用作键参数，除非它被传递，在这种情况下，这些值将被选择（见下文）。 任何None对象默认都被丢弃，除非它们全部为None，否则会引发ValueError。
- axis : {0, 1, ...}, 默认0，按行拼接。**拼接后的列数由join参数决定**
- join : {‘inner’, ‘outer’},默认‘outer’。如何处理其他轴上的索引。 **Outer：列数是两表并集，inner：列数是两表交集**
- ignore_index : boolean, 默认False. 如果为True，则不要使用连接轴上的索引值。 生成的轴将被标记为0，...，n - 1.如果连接对象的连接轴没有有意义的索引信息，则这非常有用。 注意其他轴上的索引值在连接中仍然受到尊重。
- join_axes : list of Index objects. Specific indexes to use for the other n - 1 axes instead of performing inner/outer set logic.
- keys : sequence, 默认None.创建层次化索引。使用传递的键作为最外层构建分层索引。 如果传递了多个层级，应该包含元组。
- levels : list of sequences, default None. Specific levels (unique values) to use for constructing a MultiIndex. Otherwise they will be inferred from the keys.
- names : list, default None. Names for the levels in the resulting hierarchical index.
- verify_integrity : boolean, default False. Check whether the new concatenated axis contains duplicates. This can be very expensive relative to the actual data concatenation.
- copy : boolean, 默认True. 设置为False, 不拷贝不必要的数据

1. 在默认情况下，axis=0为纵向拼接，此时有
```python
pd.concat([df1,df2],axis=0) 等价于 df1.append(df2)
```
2. 在axis=1 时为横向拼接 ，此时有
```python
pd.concat([df1,df2],axis=1) 等价于 pd.merge(df1,df2,left_index=True,right_index=True,how='outer')
```

##2. 在其他轴上设置逻辑
例如，将多个数据框（或面板或...）粘贴在一起时，您可以选择如何处理其他轴（除了正在连接的轴以外）。 这可以通过三种方式来完成：
- 取（排序后的）并集，join ='outer'。 这是默认选项，因为它会保留两张数据表的所有信息。
- 取交集，join ='inner'。
- 使用特定的index（在DataFrame的情况下）或indexes（在Panel或未来更高维的对象的情况下），即`join_axes`参数
```python
result = pd.concat([df1, df4], axis=1)
#请注意，行索引已经合并和排序
```
![concat_basic](_image/merging_concat_axis1.png)

```python
result = pd.concat([df1, df4], axis=1, join='inner')
```
![concat_basic](_image/merging_concat_axis1_inner.png)
最后，假设我们只是想重用原始DataFrame中的*exact index*：
```python
result = pd.concat([df1, df4], axis=1, join_axes=[df1.index])
```
![concat_basic](_image/merging_concat_axis1_join_axes.png)

##3. 使用append追加数据
**append用来给拼接两个或多个数据表（Series也算数据表），基本效果和concat无异。并且每次返回拼接后的完整数据拷贝，不修改原对象**
```python
Series.append(to_append, ignore_index=False, verify_integrity=False)
#拼接两个或者更多的Series
DataFrame.append(other, ignore_index=False, verify_integrity=False)
#追加其他行到这个frame的尾部，返回追加后的新对象。不在此frame中的列将作为新列添加，并填充Nan。
```
```python
result = df1.append(df2)
```
![concat_basic](_image/merging_append1.png)
在DataFrame的情况下，行索引必不相交，但是列不需要，即保留所有信息：
```python
result = df1.append(df4)
```
![concat_basic](_image/merging_append2.png)
append可以追加多个对象：
```python
result = df1.append([df2, df3])
```

##4. 忽略连接轴上的索引
`ignore_index`参数来忽略连接轴上的索引
```python
result = pd.concat([df1, df4], ignore_index=True)
result = df1.append(df4, ignore_index=True)
#两者效果相同
```
![concat_basic](_image/merging_concat_ignore_index.png)

##5. 混合ndim的数据拼接

##6. 分组关键字来拼接数据

##7. 向DataFrame追加行
虽然`DataFrame.append()`方法不是特别有效（因为必须创建一个新的对象），但是传递一个Series或者字典来附加一行到DataFrame也可行，这样就**返回一个新的DataFrame**。
```
In [34]: s2 = pd.Series(['X0', 'X1', 'X2', 'X3'], index=['A', 'B', 'C', 'D'])
In [35]: result = df1.append(s2, ignore_index=True)
```
![concat_basic](_image/merging_append_series_as_row.png)
使用ignore_index来指示DataFrame放弃Series的索引，接着DataFrame的索引继续编号。 如果你想保留Series的索引，应该构造一个适当索引的DataFrame来连接这些对象。
也可以使用字典来添加数据
```
In [36]: dicts = [{'A': 1, 'B': 2, 'C': 3, 'X': 4},
   ....:          {'A': 5, 'B': 6, 'C': 7, 'Y': 8}]
In [37]: result = df1.append(dicts, ignore_index=True)
```
![concat_basic](_image/merging_append_dits.png)

------------------------------------------------------

#二、数据库风格的DataFrame连接/合并（Database-style DataFrame joining/merging）
pandas具有功能全面，**高性能**的内存数据连接操作，与SQL等关系数据库非常相似。与其他开源实现（比如R中的base :: merge.data.frame）相比，pandas中的方法在性能上要好得多（在某些情况下，性能要好得多）。是因为对DataFrame中数据进行了精细的算法设计和内部布局。
pandas提供了一个单独的函数`merge`，作为对DataFrame对象进行所有标准数据库连接操作的入口点：
```python
pd.merge(left, right, how='inner', on=None, left_on=None, right_on=None,
         left_index=False, right_index=False, sort=True,
         suffixes=('_x', '_y'), copy=True, indicator=False,
         validate=None)
```
- `left`: 一个DataFrame对象
- `right`: 另一个DataFrame对象
- `on`: 用作连接键的Columns(names)。必须出现在左右两个DataFrame中。如果没有传递，并且`left_index`和`right_index`都设置为False，DataFrame中的列名的交集将被推断为连接键。**如果两个对象上的连接键列名不同，则可以通过 `left_on=XXX`, `right_on=XXX`来分别指定**
- `left_on`: 左DataFrame中用作连接键的Columns。可以是列名或长度等于DataFrame长度的数组。**这个参数中左右列名不相同，但列的含义相同。**
- `right_on`: 右DataFrame中用作连接键的Columns。可以是列名或长度等于DataFrame长度的数组
- `left_index`: 如果为`True`, 使用左DataFrame的索引(行labels)作为连接键。在一个有多级行索引的DataFrame中，多级索引的层次数必须匹配右DataFrame连接键的数量。**有时候DataFrame中的索引会作为连接键**
- `right_index`: 用法同left_index
- `how`: 数据表连接方式，'left', 'right', 'outer', 'inner'中之一. 默认`inner`。
- `sort`: 按照字典顺序通过连接键对合并后的数据进行排序。默认为True.在大多数情况下设置为False可以提高性能。
- `suffixes`: 用于叠合列的字符串后缀元祖，**用于指定当左右DataFrame存在相同列名时在列名后面附加的后缀名称**，默认为(`'_x'`, `'_y'`).**指的是当左右对象中存在除连接键外的同名列时，结果集中的区分方式，可以各加一个小尾巴。对于多对多连接，结果采用的是行的笛卡尔积。**
- `copy`: Always copy data (default True) from the passed DataFrame objects, even when reindexing is not necessary. Cannot be avoided in many cases but may improve performance / memory usage. The cases where copying can be avoided are somewhat pathological but this option is provided nonetheless.
- `indicator`: Add a column to the output DataFrame called _merge with information on the source of each row. _merge is Categorical-type and takes on a value of left_only for observations whose merge key only appears in 'left' DataFrame, right_only for observations whose merge key only appears in 'right' DataFrame, and both if the observation’s merge key is found in both.

    *New in version 0.17.0.*

- `validate` : string, default None. If specified, checks if merge is of specified type.
  - “one_to_one” or “1:1”: checks if merge keys are unique in both left and right datasets.
  - “one_to_many” or “1:m”: checks if merge keys are unique in left dataset.
  - “many_to_one” or “m:1”: checks if merge keys are unique in right dataset. 
  - “many_to_many” or “m:m”: allowed, but does not result in checks.

    *New in version 0.21.0.*

返回类型将与`left`相同。如果`left`是一个`DataFrame`而`right`是DataFrame的一个子类，返回类型仍然是`DataFrame`。
`merge`是pandas命名空间中的一个函数，也可以作为DataFrame实例方法使用，调用的DataFrame被隐式地视为连接中的左对象。
相关的`DataFrame.join`方法在内部使用合并索引（默认情况下）和column(s) - 索引连接。 如果你只加入行索引，你可能希望使用DataFrame.join来保存一些输入。

##1.简要介绍合并方法（关系代数）
以下几种情况需要重点理解：
- **one-to-one** joins: 例如，在索引（必须包含唯一值）上连接两个DataFrame对象
- **many-to-one** joins: 例如，将索引（唯一）连接到DataFrame中的一个或多个列时
- **many-to-many** joins: 连接列和列

**注意：在列上连接列（可能是多对多连接）时，传递的DataFrame对象上的任何索引都将被丢弃。**

花费一些时间来理解多对多连接的结果是值得的。 在SQL/标准关系代数中，如果一个键组合在两个表中出现不止一次，则生成的表将具有关联数据的`笛卡尔乘积`。 这是一个非常基本的例子，有一个独特的组合键：
```python
In [38]: left = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
   ....:                      'A': ['A0', 'A1', 'A2', 'A3'],
   ....:                      'B': ['B0', 'B1', 'B2', 'B3']})
   ....: 

In [39]: right = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
   ....:                       'C': ['C0', 'C1', 'C2', 'C3'],
   ....:                       'D': ['D0', 'D1', 'D2', 'D3']})
   ....: 

In [40]: result = pd.merge(left, right, on='key')   #默认inner连接
```
![merge_on_key](_image/merging_merge_on_key.png)
下面是一个更复杂的例子，有多个连接键：
```python
In [41]: left = pd.DataFrame({'key1': ['K0', 'K0', 'K1', 'K2'],
   ....:                      'key2': ['K0', 'K1', 'K0', 'K1'],
   ....:                      'A': ['A0', 'A1', 'A2', 'A3'],
   ....:                      'B': ['B0', 'B1', 'B2', 'B3']})
   ....: 

In [42]: right = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
   ....:                       'key2': ['K0', 'K0', 'K0', 'K0'],
   ....:                       'C': ['C0', 'C1', 'C2', 'C3'],
   ....:                       'D': ['D0', 'D1', 'D2', 'D3']})
   ....: 

In [43]: result = pd.merge(left, right, on=['key1', 'key2'])
```
![merge_on_key](_image/merging_merge_on_key_multiple.png)
`merge `方法的`how`参数指定如何确定哪些键将被包含在结果表中。如果组合键**没有出现**在左侧或右侧表格中，则连接表中的值将为`NA`。以下是选项及其SQL等效名称的摘要：
连接方式 | SQL连接名称 | 描述
--- | --- | ---
inner | INNER JOIN | 内连接，取笛卡尔积中，左右表中有交集（有关联）的行数据
outer | FULL OUTER JOIN | 外连接，左右表的笛卡尔积，即取左右表中行数据的并集，缺失数据以Nan填充
left | LEFT OUTER JOIN | 左连接，笛卡尔积中，左表中的数据必须都显示，即行数据按左表对齐
right | RIGHT OUTER JOIN | 右连接，笛卡尔积中，右表中的数据必须都显示，即行数据按右表对齐
```python
result = pd.merge(left, right, how='outer', on=['key1', 'key2']) #笛卡尔积
result = pd.merge(left, right, how='inner', on=['key1', 'key2']) #笛卡尔积中的非Nan数据
result = pd.merge(left, right, how='left', on=['key1', 'key2']) #笛卡尔积中的left为非Nan数据
result = pd.merge(left, right, how='right', on=['key1', 'key2']) #笛卡尔积中的right为非Nan
```
![merge_on_key_left](_image/merging_merge_on_key_left.png)

以下是DataFrame中有重复连接键的另一个示例：
```python
In [48]: left = pd.DataFrame({'A' : [1,2], 'B' : [2, 2]})

In [49]: right = pd.DataFrame({'A' : [4,5,6], 'B': [2,2,2]})

In [50]: result = pd.merge(left, right, on='B', how='outer') #这时suffix默认为`_x`和`_y`
```
![merge_on_key_left](_image/merging_merge_on_key_dup.png)

**警告：在重复键上进行连接(join)/合并(merge)，会返回两个表的各个重复键在行维度相乘后的Frame(即笛卡尔积)，这可能导致内存溢出。在连接大型DataFramez之前，用户有责任管理连接键中的重复键。**

##检查重复键
*New in version 0.21.0.*

用户可以使用`validate`参数来自动检查连接键中是否有意外的重复项。 键唯一性在合并操作前被检查，所以应该防止内存溢出。检查键唯一性也是确保用户数据结构如预期的好方法。

在下面的例子中，在右边的DataFrame中`B`列有重复值。 由于这不是一个一对一的合并 - 在validate参数中指定 - 会引发异常。
```python
In [51]: left = pd.DataFrame({'A' : [1,2], 'B' : [1, 2]})

In [52]: right = pd.DataFrame({'A' : [4,5,6], 'B': [2, 2, 2]})

In [53]: result = pd.merge(left, right, on='B', how='outer', validate="one_to_one")
...
MergeError: Merge keys are not unique in right dataset; not a one-to-one merge
```
如果用户知道右侧DataFrame中的重复项，但是要确保在左侧的DataFrame中没有重复项，则可以使用`validate ='one_to_many'`参数，而不会引发异常。
```python
In [53]: pd.merge(left, right, on='B', how='outer', validate="one_to_many")
Out[53]: 
   A_x  B  A_y
0    1  1  NaN
1    2  2  4.0
2    2  2  5.0
3    2  2  6.0
```

##The merge indicator

##Merge Dtypes

##索引上的连接
```python
DataFrame.join(other, on=None, how='left', lsuffix='', rsuffix='', sort=False)
#默认左索引连接
```
`DataFrame.join`是将两个潜在索引不同的DataFrame的列组合成单个结果DataFrame的一种便捷方法。 这是一个非常基本的例子：
```python
In [78]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
   ....:                      'B': ['B0', 'B1', 'B2']},
   ....:                      index=['K0', 'K1', 'K2'])
   ....: 

In [79]: right = pd.DataFrame({'C': ['C0', 'C2', 'C3'],
   ....:                       'D': ['D0', 'D2', 'D3']},
   ....:                       index=['K0', 'K2', 'K3'])
   ....: 

In [80]: result = left.join(right)
```
![merging_join](_image/merging_join.png)

```python
result = left.join(right, how='outer')
#两者效果相同
result = pd.merge(left, right, left_index=True, right_index=True, how='outer')
```
![merging_join](_image/merging_join_outer.png)
