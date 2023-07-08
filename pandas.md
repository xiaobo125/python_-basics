# 一、处理缺失数据

## 1.1删除数据

### 1.Series

#### ①检测缺失值函数

```python
data1 = pd.Series(['a', 'b', np.nan, 'd'])  #创建Series
print(data1.isnull())   #是否有缺失值
print(data1.notnull())  #是否没有缺失值
```

#### ②滤除缺失数据（下述两个函数等价）

```python
print(data1.dropna())
print(data1[data1.notnull()])
```

### 2.DataFrame

#### ①正常数表操作示例

```python
data2 = pd.DataFrame([[1., 6.5, 3.], [1., np.nan, np.nan], [np.nan, np.nan, np.nan], [np.nan, 6.7, 7.0]])  #使用双列表创建DataFrame
print(data2.dropna())     #只要含有NaN缺失值的都会被删除
print(data2.dropna(how='all'))  #丢弃所有值都为空值的行
print(data2.dropna(axis=1))     #删除含有NaN值得列
```

#### ②随机数表操作示例

```python
df = pd.DataFrame(np.random.randn(7, 3)) #建立7行3列的随机数表格
df.iloc[:4, 1] = np.nan   #前4行第1列（索引）设置为空值
df.iloc[:2, 2] = np.nan   #继续赋予缺失值
print(df.dropna())        #将含有缺失值的都删除掉
print(df.dropna(thresh=2))#删除有两个缺失值数据，参数代表去除的空值个数。
```

## 1.2填充数据

### 1.数据上接上述随机数

```python
print(df.fillna(0))  #将所有NaN值替换成0，参数是替换值。
print(df.fillna({1: 0.9, 2: 0}))  #使用字典，将列索引变成对应的值，对本身df没有修改
print(df.fillna({1: 0.9, 2: 0}, inplace=True))  #当有参数时，此处为永久修改。
```

### 2.新建数据集

```python
df2 = pd.DataFrame(np.random.randn(6, 3))  #新建数据集
df2.iloc[2:, 1] = np.nan    #设置空值
df2.iloc[4:, 2] = np.nan    #设置空值
print(df2.fillna(method='ffill'))  #将空值变成空值列上一个数
print(df2.fillna(method='ffill', limit=2))  #指定填充个数
```

# 二、数据转换

## 2.1移除重复数据

### 1.检查是否出现重复值

```python
data = pd.DataFrame({'K1': ['one', 'two'] * 3 + ['two'], 'k2': [1, 1, 2, 3, 3, 4, 4]})  #建立数据集，数据量较小
print(data.duplicated())  #返回一个布尔Series，看是否数据重复
```

### 2.删除出现的重复值

```python
print(data.drop_duplicates())   #删除重复值行
#上述两种方法判定的是全部的列
print(data.drop_duplicates(['k1']))  #可以按照指定列去除重复列
#上述默认删除都是后面重复值，保留第一次出现的数据。
print(data.drop_duplicates(['k2'], keep='last') #保留最后一列的数据。
```

## 2.2利用函数或映射进行数据转换

### 1.映射

```python
data = pd.DataFrame({'food': ['Apple', 'banana', 'orange', 'apple', 'Mango', 'tomato'], 'price': [4, 3, 3.5, 6, 12, 3]})
#建立数据集
meat = {'banana': 'fruit',
        'orange': 'fruit',
        'apple': 'fruit',
        'mango': 'fruit',
        'tomato': 'vegetables'}   #通过字典建立对应映射
data['class'] = data['food'].map(meat)   #增加一列，使用匹配。
print(data)
low = data['food'].str.lower()   #将food列变为小写，调整来匹配。
data['class'] = low.map(meat)    #添加类型列，进行匹配输出
print(data)
data['class1'] = data['food'].map(lambda x:meat[x.lower()])  
#与前述字母变小操作相同
print(data)
```

### 2.替换值（处理异常值问题）

```python
data = pd.Series([1, -999, 2, -1000, 3])  #建一个含有异常值的Series
print(data.replace(-999, np.nan))         #对值进行替换
print(data.replace([-999, -1000], [np.nan, 0]))  #替换多个值
print(data.replace({-999: np.nan, -1000: 0}))    #使用字典写法
#上述均返回新的数据，不是在原有数据上操作
```

### 3.重命名轴索引

#### ①与重新索引不同

```python
data = pd.DataFrame(np.arange(12).reshape(3, 4),
                    index=['Beijing', 'Tokyo', 'New York'],
                    columns=['one', 'two', 'three', 'four'])
print(data)
print(data.reindex(['Beijing', 'New York', 'Tokyo'])) 
#重新索引，此时标签调换顺序，reindex只能修改已有的标签名，不能修改原来未有的。如果使用新的，则会生成新的表格，此方法不会对原来表格产生影响。
```

#### ②重命名轴索引的函数rename

```python
tran = lambda x: x[:4].upper()   #定义一个大写函数
print(data.index.map(tran))      #前四位大写
data.index = data.index.map(tran)#赋值
print(data)                      #更改原来表格
print(data.rename(index=str.title, columns=str.upper)) 
#rename重命名，但不影响原来的值，创建新的表格。
print(data.rename(index={'TOKY': '东京'}, columns={'three': '第三年'}))   #结合字典对标签更新   永久保存使用
```

## 2.3离散化和面元划分

```python
ages = [20, 22, 25, 27, 21, 23, 37, 31, 61, 45, 41, 22]
bins = [18, 25, 35, 60, 100]    #面元
cats = pd.cut(ages, bins)       #划分的面元
print(cats)
print(cats.codes)               #输出一个数组，看属于哪个阶段
print(cats.categories)          #看有几个区间
print(pd.value_counts(cats))    #统计每个区间拥有值的个数
print(pd.cut(ages, [18, 26, 36, 61, 100], right=False))#区间指定左闭
names = ['青年', '年轻人', '中年', '老年']
print(pd.cut(ages, bins, labels=names))#设定标签
data = np.random.rand(20)           #定义一组值
print(pd.cut(data, 4, precision=2)) #将值分为四段，并且指定小数位数
data = np.random.randn(1000)        #定义一组值
cats = pd.qcut(data, 4)             #划分四个阶段内相等的区间
print(cats)                           
print(pd.value_counts(cats))        #验证每个阶段内值个数
data = np.random.randn(1000)
cats = pd.qcut(data, [0, 0.1, 0.5, 0.9, 1.])  #此处先对列表内均匀分份
print(pd.value_counts(cats))                  #结果排序是按照数量排序
```

## 2.4检测和过滤异常值

```python
data = pd.DataFrame(np.random.randn(1000, 4))
print(data[(np.abs(data)>3).any(1)])       #选出所有绝对值大于3的数
data[np.abs(data) > 3] = 3                 #将绝对值大于3的替换成3
print(data.describe())                     #观看数据特性
```

## 2.5排列和随机取样

```python
df = pd.DataFrame(np.arange(20).reshape((5, 4)))
sam = np.random.permutation(5)   #打乱顺序重排
print(df.take(sam))              #根据打乱顺序行进行行重排
print(df.sample(n=3))            #随机选择三行
ch = pd.Series([5, 7, 1, 6, 3])
print(ch.sample(n=10, replace=True))  #此处可以重复选择
```

# 三、字符串操作

```python
val = 'a, b,  c'
print(val.split(','))                  #按照分隔符打印输出
p = [x.strip() for x in val.split(',')]#去掉空格，包括换行符
f, s, t = p                            #字符串赋值
print(f+'::'+s+'::'+t)                 #字符串连接
print('::'.join(p))                    #字符串连接，和上述一个语句相同
print(val.index(','))                  #输出1，显示值对应下标
print(val.index(':'))                  #输出错误，无值出现错误
print(val.find(':'))                   #输出-1，如果没有找到，输出-1
print(val.find('a'))                   #输出0，找到返回下标位置
```

## 3.2正则表达式

```python 
text = "foo  bar\t  bat   \t  qq"
print(re.split('\s+', text))    #将单词分开
res = re.compile('\s+')         
print(res.split(text))          #第二种方式
reg = re.split(res, text)
print(reg)                      #第三种方式
#目的是先将正则表达式编译好，使用时直接导入变量
print(res.findall(text))        #空格，制表符都匹配出来，输出为列表
print(re.findall(res, text))    #与上式相同
print(re.sub(res, '9', text))   #将空格，制表符替换为9
t1 = re.match('f', text)        #匹配函数
print(t1.group())               #match方法只在起始位置查找
print(re.search('b', text).group())  #可以查看所有，但是只显示第一个
```

<<<<<<<< HEAD:1Data cleaning and preparation.md
## pandas的矢量化字符串函数

```python
data = {'a': 'dava@qq.com', 'b': 'steve@gmail',
        'c': 'sam@gmail', 'd': np.nan}
data = pd.Series(data)            
print(data.str.contains('gmail'))   #查找序列中是否有字符串gmail
print(data.str.split('@'))          #以@进行一个分离
print(data.str.findall('@'))        #字符串查找
print(data.str[:5])                 #通过切片对字符串截取
```

# 四、总结

1.部分数据缺失（存在大量空值）——填充或者删除缺失数据

检测缺失数据isnull()，删除dropna，替换fillna

2.数据存在重复值

检测重复值duplicated()，删除重复值drop_duplicates(inplace=True)

3.数据类型不统一——使用数据类型转换

4.部分数据包含数值和字符串——字符串处理（分隔，替换）

5.部分数据存在异常值——删除异常值drop

6.少数数据不利于分析——数据替换replace

```python
data = pd.read_csv('guazi.csv')
(data.isnull()).sum()                             #找缺失数据的值进行求和
data.dropa(subset['yuanjia']，inplace=True)           #删除具体行缺失数据
data.duplicated()             #检测重复数据
data.duplicated().sum         #对数据求和
data.drop_duplicates(inplace=True)  #在原有数据上删除重复数据
(data.shoujia.str.contain('万')).sum()  #寻找有多少行有万字
data['shoujia'] = data.shoujia.map(lambda x:folat(x.replace('万','')))
#对万字进行取代
data['shoujia'] = data.shoujia.map(lambda x:(x.replace('万','')).astype('float64')   #先进行处理在转换
data.drop([4, 9]，inplace=True)  #删除异常值                               
```
========

>>>>>>>> 7442bcdf43da5298e1168223036491577b104055:pandas.md
