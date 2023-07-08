# 一、层次化索引



```python
data = pd.Series(np.random.randn(9),
                 index=[['a', 'a', 'a', 'b', 'b', 'c', 'c', 'd', 'd'],
                        [1, 2, 3, 1, 3, 1, 2, 2, 3]])
print(data['a'])     #直接对外层索引
print(data['a':'b']) #两个外层索引，使用切片
print(data.loc[['b', 'd']])  #间隔选取索引
print(data.loc[:, 2])#：是跳过第一层索引，找二层索引
frame2 = frame.set_index(['c', 'd'])  #建立一个新的层次化索引
frame2 = frame.set_index(['c', 'd'], drop=False)  #过程删除的列不删除
print(frame2.reset_index())   #重新设置索引
```

# 二、数据连接

```python
left = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                     'A': ['A0', 'A1', 'A2', 'A3'],
                     'B': ['B0', 'B1', 'B2', 'B3']})
right = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                      'C': ['C0', 'C1', 'C2', 'C3'],
                      'D': ['D0', 'D1', 'D2', 'D3']})
print(pd.merge(left, right))    
#将二者合并，重叠部分连接，行连接起来，默认将重叠的列名进行连接
print(pd.merge(left, right, on='key')) #on是指定什么进行连接
```

## 2.1多个键值连接如下，都进行匹配

```python
left = pd.DataFrame({'key1': ['K0', 'K0', 'K1', 'K2'],
                     'key2': ['K0', 'K1', 'K0', 'K1'],
                     'A': ['A0', 'A1', 'A2', 'A3'],
                     'B': ['B0', 'B1', 'B2', 'B3']})
right = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                      'key2': ['K0', 'K0', 'K0', 'K0'],
                      'C': ['C0', 'C1', 'C2', 'C3'],
                      'D': ['D0', 'D1', 'D2', 'D3']})
print(pd.merge(left, right, on=['key1', 'key2']))   #多个匹配连接
print(pd.merge(left, right, how='left', on=['key1', 'key2'])) 
#以左边为基准进行结果输出，左边为准去匹配右边是否有，有就填上，没有就是空值。
print(pd.merge(left, right, how='right', on=['key1', 'key2'])) 
#以右边为基准进行结果输出
print(pd.merge(left, right, how='outer', on=['key1', 'key2']))
#所有键的组合，相当于一个并集
```

## 2.2格式不一样

```python
df_obj1 = pd.DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'a', 'b'],
                        'data': np.random.randint(0, 10, 7)})
df_obj2 = pd.DataFrame({'key': ['a', 'b', 'd'],
                        'data': np.random.randint(0, 10, 3)})
print(pd.merge(df_obj1, df_obj2, on='key', suffixes=('_left', '_right')))
#添加了一个后缀，避免名字重复，将行数不同的表格进行匹配。
print(pd.merge(df_obj1, df_obj2, left_on='key', right_index=True))
#左边表按照key连接，右边使用行索引连接
```

# 三、数据合并

```python
left2 = pd.DataFrame([[1., 2.], [3., 4.], [5., 6.]],
                     index=['a', 'c', 'e'],
                     columns=['语文', '数学'])
right2 = pd.DataFrame([[7., 8.], [9., 10.], [11., 12.], [13., 14.]],
                      index=['b', 'c', 'd', 'e'],
                      columns=['英语', '综合'])
print(pd.merge(left2, right2, how='outer', left_index=True, right_index=True))  #外连接，取并集，按照索引取并集
print(left2.join(right2, how='outer'))  #输出与上述一样，但不能有重叠的列
#join是按照索引进行合并，但是要求没有重叠的列名
```

## 3.1 Numpy的连接

```python
arr1 = np.random.randint(0, 10, (3, 4))
arr2 = np.random.randint(0, 10, (3, 4))
print(np.concatenate([arr1, arr2]))         #按照行索引拼接
print(np.concatenate([arr1, arr2], axis=1)) #按照列索引拼接
```

## 3.2 pandas的连接

```python
df1 = pd.DataFrame(np.arange(6).reshape(3, 2),
                   index=list('abc'), columns=['one', 'two'])
df2 = pd.DataFrame(np.arange(4).reshape(2, 2)+5,
                   index=list('ac'), columns=['three', 'four'])
print(pd.concat([df1, df2]))   #使用pandas连接，默认情况下是外连接
print(pd.concat([df1, df2], axis=1))  #使用列连接起来，默认外连接
#也可以合并相同名的

```

# 四、重塑和轴向旋转

## 4.1重塑

```python
data = pd.DataFrame(np.arange(6).reshape(2, 3),
                    index=pd.Index(['老王', '小刘'], name='姓名'),
                    columns=pd.Index(['语文', '数学', '英语'], name='科目'))
print(data.stack())   #将列索引旋转为行索引，完成层级索引，转化为Series对象
r = data.stack()
print(r.unstack())    #将Series转化回去
print(r.unstack(0))   #另外一种DataFrame对象，可以指定层级编号
a1 = pd.Series(np.arange(4), index=list('abcd'))
a2 = pd.Series([4, 5, 6], index=list('cde'))
s1 = pd.concat([a1, a2], keys=['data1', 'data2'])  #加上外层索引
print(type(s1))   #生成的是Series
print(s1.unstack())  #转化为DataFrame   
print(s1.unstack().stack())   #默认过滤缺失数据，进行二次转换
print(s1.unstack().stack(dropna=False))  #不过滤NaN值
#unstack默认索引是内层
```

## 4.2轴向旋转

```python
df3 = pd.DataFrame({'date': ['2018-11-22', '2018-11-22', '2018-11-23', '2018-11-23', '2018-11-24'],
                    'class': ['a', 'b', 'c', 'd', 'e'],
                    'values': [5, 3, 2, 6, 1]}, columns=['date', 'class', 'values'])
print(df3.pivot('date', 'class', 'values')) #先列再行 
print(df3.set_index(['date', 'class']).unstack('class')) #与上述相同
#pivot方法本质上就是set_index和unstack的结合
```

# 五、数据分组与组合

## 5.1分组

```python
df1 = pd.DataFrame({'fruit': ['apple', 'banana', 'orange', 'apple', 'banana'],
                    'color': ['red', 'yellow', 'yellow', 'cyan', 'cyan'],
                    'price': [8.5, 6.8, 5.6, 7.8, 6.4]})
g = df1.groupby(by='fruit')  #可形成可迭代对象
for i in g:   #对迭代对象进行输出
    print(i)  
for name, group in g:   
    print(name)          #输出组名
    print('-'*100)
    print(group)         #输出数据块
print(type(group))   #数据块是DataFrame类型
print(dict(list(df1.groupby(by='fruit')))['apple']) #选取任意数据块

```

## 5.2聚合

```python
print(df1.groupby('fruit')['price'].mean())  # 根据水果完成分组,价格，平均值
print(df1['price'].groupby(df1['fruit']).mean())  #语法糖
print(df1.groupby('fruit', as_index=False)['price'].mean())  #返回有行索引
def diff(arr):
    return arr.max() - arr.min()
print(df1.groupby('fruit')['price'].agg(diff))  #定义函数求差值
```
