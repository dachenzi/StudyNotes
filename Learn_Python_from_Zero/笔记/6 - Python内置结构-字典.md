# 1 字典介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Python中字典属于一种映射类型，它和set相同，同样属于非线性结构存储，Python官方对dict有如下解释
- 一个映射对象映射一个可hash的值到任意一个对象上去。
- 映射是可变的对象。
- dict是当前唯一一个标准的映射类型。
- 字典的键几乎可以任意的值。
- 字典的值可以不必可hash，也就是说值可以是列表，字典，或者其他任意可变的对象
- 如果key相同，那么后面的value将会覆盖先前的value
- 不建议使用float类型作为字典的key  

简单来说：字典是由key:value键值对组成的数据的集合，它的主要特点是 __`可变的`__、__`无序的`__、__`不重复的`__。
> 字典的key必须是可hash对象。
# 2 字典操作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字典是除set集合以外另一种可变的非线性容器模型，在Python中非常强大，适合各种结构数据的存储、嵌套等。
## 2.1 字典的定义
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字典的每个key到value的键值对用冒号(:)分割，每对之间用逗号(,)分割，整个字典包括在花括号({})中。例：`{'a':1, 'b':2}`，Python提供了多种创建字典的方式，如下：
- 基本格式：`d = dict()` 或者 `d = {}` 
- `dict(**kwargs)`: 使用name=value对，来初始化一个字典
- `dict(iterable, **kwargs)`：使用可迭代对象和name=value构造字典，__`注意可迭代对象必须是一个二元结构`__
- `dict(mapping, **kwargs)`: 使用一个字典构造另一个字典
- `dic = {'a':1, 'b':2, 'c':3, 'd':[1,2,3]}`
- `dic = dict.fromkeys(iterable, value)`: 使用可迭代对象的值作为key生成字典,value默认为0，否则用指定的value对字典进行初始化。
```python
In [114]: d1=dict()                                                                                        
In [115]: d2={}                                                                                            
In [118]: d3 = dict(a=1,b=2)                                                                               
In [120]: d3                                                                                               
Out[120]: {'a': 1, 'b': 2}
In [124]: d4 = dict([('a',1),('b',2)], c=3, d=4)                                                           
In [125]: d5 = dict(d4,e=5,f=6)                                                                            
In [126]: d7 = dict.fromkeys(range(5))                                                                     
In [127]: d8 = dict.fromkeys(range(5),100)                                                                
In [128]: d7                                                                                               
Out[128]: {0: None, 1: None, 2: None, 3: None, 4: None}                                                         
In [131]: d8                                                                                               
Out[131]: {0: 100, 1: 100, 2: 100, 3: 100, 4: 100} 
``` 
## 2.2 字段元素的访问