# 1 封装与解构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;封装与解构属于Python语言的一种特性，它使用起来很像其他语言中的`"逗号表达式"`，但内部原理是不同的，在某些场景下：比如变量交换复制时使用，显得非常优雅。
## 1.1 封装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;封装故名思议就是装箱，把多个值使用逗号分隔，组合在一起，本质上来看，其返回的是一个元组，只是省略了小括号。(一定要区别与C语言的逗号表达式)
```python
In [91]: t1 = (1,2)  # 定义一个元组                                                                                                                                
In [92]: t2 = 1,2    # 省略括号，其内部还是会封装成元组                                                                                                                               
In [93]: t1                                                                                                                                         
Out[93]: (1, 2)
In [94]: t2                                                                                                                                         
Out[94]: (1, 2)
In [95]: type(t1)                                                                                                                                   
Out[95]: tuple
In [96]: type(t2)                                                                                                                                   
Out[96]: tuple

```
## 1.2 解构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解构，解构，就是把箱子解开，在Python中表示从线性结构中把元素解开，并且顺序的赋值给其他变量，需要注意的是，解构时接受元素的变量，需要放在等式的左边，并且数量要和右边解开的元素的个数一致。
```python
In [97]: t1                                                                                                                                         
Out[97]: (1, 2)
In [98]: a,b = t1    # 表示把1赋给a，把2赋给b                                                                                                                               
In [99]: a                                                                                                                                          
Out[99]: 1
In [100]: b                                                                                                                                         
Out[100]: 2
In [101]: a,b,c = t1           # 当接受元素的变量多余解构的元素时，会提示ValueError，反之相同                                                                                                                     
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-101-2e8ad53e5fc7> in <module>
----> 1 a,b,c = t1

ValueError: not enough values to unpack (expected 3, got 2)

In [102]:  
```
## 1.3 Python3的解构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;了解了封装与解构，那么回想一下当我们需要进行变量交换的时候，是否可以通过封装与解构进行优化呢？当我们在其他语言中进行a，b变量的值的交换，我们需要一个中间变量temp，即：`a = temp,a = b,b = temp`，在Python中我们可以省略。
```python
>>> a = 1
>>> b = 200
>>> a,b = b,a    # 这样就完成了变量的交换了，是不是很方便
>>> a
200
>>> b
1
>>> 
```
>为什么可以使用这种骚操作，是因为Python在进行变量赋值时，会先计算等式右边的表达式，封装起来，然后再进行解构，赋给对应位置上的变量。并且还提供了其他更便捷的方法比如*号。  

- `*号`： 使用方式为: `*变量名`，贪婪的吸收解构的元素形成一个列表,__无论能否吸收，都会返回一个列表__。
- `_号`：表示丢弃一个变量(实际上是使用`_`接受变量，但是使用这么一个符号，就表示我们不想用它)
```python
In [1]: t = list(range(5))                                                                                                                          
In [2]: t                                                                                                                                           
Out[2]: [0, 1, 2, 3, 4]
In [3]: head,*mid,tail = t                                                                                                                          
In [4]: head                                                                                                                                        
Out[4]: 0
In [5]: mid                                                                                                                                         
Out[5]: [1, 2, 3]
In [6]: tail                                                                                                                                        
Out[6]: 4

```
需要注意的是：
1. *变量名，这种格式不能单独使用
2. 也不能多个\*变量名，连续使用(因为从原理上看，两个、*变量名连起来使用，会引起歧义，所以Python禁止了这种写法)
3. *_这种格式，可以收集足够多的元素丢弃
```python
# 1 从[1,(2,3,4),5]中取出4来
>>> _,(*_,a),_ = [1,(2,3,4),5]
>>> a
>>> 4

# 环境变量JAVA_HOME=/usr/bin/java，返回环境变量名和路径
>>> env,path = 'JAVA_HOME=/usr/bin/java'.split('=')
>>> env,path
>>> ('JAVA_HOME','/usr/bin/java')

或者

In [9]:  env, _, path = 'JAVA_HOME=/usr/bin/java'.partition('=')                                                                                    
In [10]: env,path                                                                                                                                   
Out[10]: ('JAVA_HOME', '/usr/bin/java')
```
__解构是Python提供的很好的功能，可以方便的提取复杂的数据结构的值，配置_使用时，会更加便捷。__
# 2 set类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;集合set在Python中是一个非常重要的`非线性结构`，它使用`{}`表示，用三个词总结集合的特点就是：__`可变的`__、__`无序的`__、__`不重复`__。它的官方解释如下：
- set是一个无序的，不同的`可hash对象`组成的集。
- 常用来进行成员测试，在一个序列中去掉重复的对象，和进行数学上的计算，比如`交集(intersection)`、`并集(union)`、`差集(difference)`、`对称差集(symmetric difference)`等。
- 和其他容器类型相似，在一个无序的集合中支持`x in set`,`len(set)`,`for x in set`等操作。
- set不会记录元素的位置以及元素加入集合的顺序，所以set不支持`索引`，`切片`或者其他的类序列的操作。
> 什么是可hash对象，可以简单的理解为可以被hash()计算的对象，在Python中，可hash对象是不可变类型的，比如tuple, str, int, 等等。
## 2.1 set的定义
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python提供了两种定义一个集合的方式，`set()` 和 `set(iterable)`，他们的用法如下：
```python
set() -> new empty set object     --> 返回一个空的set对象
set(iterable) -> new set object   --> 返回一个set对象，元素有iterable填充
```
例如：
```python
In [44]: s1 = set()                                                                                        
In [45]: s2 = set(range(5))             # {0, 1, 2, 3, 4}                                                                    
In [46]: s3 = set(list(range(10)))   # 外面使用list进行转换，多此一举，{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}                                                                     
In [47]: s4 = {}           # 这种方式实际上是创建了一个字典                                                                               
In [48]: s4                                                                                                
Out[48]: {}
In [50]: type(s4)                                                                                          
Out[50]: dict
In [51]: s5 = {(1,3),3,'a'}                                                                                  
In [52]: s6={[1,2],(1,2,),1,2}       # list属于不可hash对象，所以无法添加到set中去                                                                        
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-52-237c72203405> in <module>
----> 1 s6={[1,2],(1,2,),1,2}

TypeError: unhashable type: 'list'

In [53]:
```
## 2.2 set的基本操作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set是可变的类型，那就意味着，我们可以对set进行增加、删除、修改等操作。
### 2.2.1 增加元素
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set提供了两种定义一个集合的方式，`add` 和 `update`，他们的用法如下：
```python
s.add() --> None --> 在集合s中添加一个元素，如果元素存在，则什么都不做(去重特性)。（就地修改）

s.update(*other) --> None --> 把*others个可迭代可hash对象，和s进行并集，然后赋给s。（就地修改）
```
例
```python
In [62]: s                                                                                                 
Out[62]: {1, 2, 3}
In [63]: s.add('abc')     # 把'abc'当作一个元素添加进去                                                                                     
In [64]: s                                                                                                 
Out[64]: {1, 2, 3, 'abc'}  
In [65]: s.add((1,2,3))     # 把(1,2,3) 当作一个元素添加进去                                                                             
In [66]: s                                                                                                 
Out[66]: {(1, 2, 3), 1, 2, 3, 'abc'}
In [67]: s.update(range(5),'abcdef',[5,6,7,8])     # 多个可迭代可hash对象取并集                                                      
In [68]: s                                                                                                 
Out[68]: {(1, 2, 3), 0, 1, 2, 3, 4, 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}
```
### 2.2.2 删除元素
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set提供了多种定义一个集合的方式，比如 `remove` ,`pop`，他们的用法如下：
```python
s.remove()  --> None --> 在集合s中删除一个元素，这个元素必须存在集合s中，否则会报KeyError异常

s.pop()  --> item --> 在集合s中随便弹出一个元素，并返回元素的本身，如果集合本身为空，那么会提示KeyError异常

s.discard(elem) --> None --> 在集合s中删除一个元素，如果元素不存在集合中，那么什么也不做

s.clear()  --> None --> 清空集合
```
例
```python
In [71]: s                                                                                                 
Out[71]: {(1, 2, 3), 0, 1, 2, 3, 4, 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}

In [72]: s.remove(0)                                                                                       

In [73]: s                                                                                                 
Out[73]: {(1, 2, 3), 1, 2, 3, 4, 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}

In [74]: s.remove(1000)       # 不存在集合内的元素，删除会报异常                                                                               
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-74-9d9da17ab719> in <module>
----> 1 s.remove(1000)

KeyError: 1000
In [76]: s                                                                                                 
Out[76]: {(1, 2, 3), 1, 2, 3, 4, 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}

In [77]: s.pop()                                                                                           
Out[77]: 1

In [78]: s                                                                                                 
Out[78]: {(1, 2, 3), 2, 3, 4, 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}
In [82]: s1                                                                                                
Out[82]: set()
In [84]: s1.pop()      # 空集合会报异常                                                                                    
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-84-095118de218b> in <module>
----> 1 s1.pop()

KeyError: 'pop from an empty set'

In [85]: s                                                                                                 
Out[85]: {(1, 2, 3), 4, 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}
In [86]: s.discard(1000)     # 直接删除，不会返回                                                                             
In [89]: s.discard(4)                                                                                      
In [90]: s                                                                                                 
Out[90]: {(1, 2, 3), 5, 6, 7, 8, 'a', 'abc', 'b', 'c', 'd', 'e', 'f'}
In [92]: s.clear()                                                                                         
In [93]: s                                                                                                 
Out[93]: set()
```
### 2.2.3 修改元素
上来我们需要先想一个问题，为什么要修改set呢？修改的本质是什么？
- 修改的本质其实就是找到这个元素，删除，然后在加入新的元素
- 由于集合是非线性结构，所以无法被索引
- 但是set是容器，可以被迭代  
>所以set不能像list那样，通过索引修改元素，因为它无序的特性，修改其实等同于删除、再添加而已。
### 2.2.4 成员判断
我们说既然set是容器，那么我们就可以对容器内的元素进行判断，那么就需要使用成员判断符 `in` 和 `not in` 了。
- in: x in s, 判断元素x是否是在集合s中，返回bool类型
- not in : x not in s,  判断元素x不在集合s中，返回bool类型
```python
In [95]: s = set(range(20))                                                                                
In [96]: 1 in s                                                                                            
Out[96]: True
In [97]: 10000 not in s                                                                                    
Out[97]: True
```
> 我们知道在list，str这种`线性结构`中进行`成员判断`时，它的时间复杂度是`O(n)`的，因为需要遍历，但是在set中就很简单了，它只需要把要判断的元素进行hash，找到set中对应的门牌号，把里面的数据拽出来，看看是不是相同。 这种操作通常都是`O(1)`的，所以在set中，`成员判断效率很高`，当然在`dict类型`中，也是如此。
## 2.3 set小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;线性结构的查询时间复杂度是`O(n)`，即随着数据规模的这不过大而增加耗时，set、dict等结构，内部使用的是hash值作为key，时间复杂度可以做到`O(1)`查询时间和数据规模无关，在python中可hash对象有(都属于不可变类型)
- 数值类型int、float、complex
- 布尔值True、False
- 字符串String、Bytes
- 元组tuple
- None  
# 3 集合
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单来说，所谓的一个集合，就是将数个对象归类而分成为一个或数个形态各异的大小整体。 一般来讲，集合是具有某种特性的事物的整体，或是一些确认对象的汇集。构成集合的事物或对象称作元素或是成员。集合的元素可以是任何事物，可以是人，可以是物，也可以是字母或数字等。 （此解释来自于维基百科）
- 全集：所有元素的结合。例如实数集，所有实数组成的集合就是实数集
- 子集subset和超集superset: 一个集合A所有的元素都在另一个集合B内，A是B的子集，B是A的超集
- 真子集和真超集：A是B的子集，且A不等于B，A就是B的真子集，B就是A的真超集
- 并集：多个集合合并的结果
- 交集：多个集合的公共部分
- 差集：集合中除去和其他集合共有的部分
>这些是小学数学基础概念哦，兄弟们。
## 3.1 集合运算
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过集合运算，我们可以方便的对求出集合的差集、并集等，Python的集合除了提供了大量的集合运算方法还提供了不少的特殊符号用来表示集合运算。
## 3.2 并集
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将集合A和集合B所有元素合并在一起，组成的集合称为集合A和集合B的并集
```python
s.union(*other) --> new set object --> 把多个集合和集合s进行合并，返回一个新的集合对象
--> 使用 | 表示

s.update(*other) --> None --> 把*others个可迭代可hash对象，和s进行并集，然后赋给s。（就地修改）
--> 使用 != 表示
```
## 3.3 交集
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;集合A和集合B，由所有属于A且属于B的元素组成的集合。
```python
s.intersection(*other) --> new set object --> 返回多个集合的交集
--> 使用 & 表示

s.intersection_update(*other) --> None  --> 获取多个集合的交集，就地进行修改
--> 使用 &= 表示
```
## 3.3 差集
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;集合A和B，由所有属于A且不属于B的元素组成的集合。
```python
s.difference(*others) --> new set object --> 返回集合s和其他多个集合的差集
--> 使用 - 表示

s.difference_update(*other) --> None --> 返回集合s和其他多个集合的差集，就地进行修改
--> 使用 -= 表示
```
## 3.4 对称差集
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不属于集合A和集合B交集的其他元素组成的集合，数学表达式为：(A-B) U (B-A)。
```python
s.symmetric_difference(*other) --> new set object --> 返回和另一些集合的对称差集
--> 使用 ^ 表示

s.symmetric_difference_update --> None --> 返回和另一些集合的对称差集，就地修改
--> 使用 ^= 表示
```
## 3.5 集合的其他运算
```python
s.issubset(other) --> bool --> 判断当前集合是否是另一个集合的子集
--> 使用 <= 表示

set1 < set2 : 判断set1是否是set2的真子集

s.isssuperset(other) --> bool --> 判断当前集合是否是other的超集
--> 使用 >= 表示

set1 > set2 : 判断set1是否是other的真超集

s.isdisjoint(other) --> bool --> 判断当前集合和另一个集合有没有交集
```