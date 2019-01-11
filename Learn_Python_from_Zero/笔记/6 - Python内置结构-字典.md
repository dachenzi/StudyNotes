<!-- TOC -->

- [1 字典介绍](#1-字典介绍)
- [2 字典的基本操作](#2-字典的基本操作)
    - [2.1 字典的定义](#21-字典的定义)
    - [2.2 字典元素的访问](#22-字典元素的访问)
    - [2.3 字典的增删改](#23-字典的增删改)
- [3 字典遍历](#3-字典遍历)
    - [3.1 遍历字典的key](#31-遍历字典的key)
    - [3.2 遍历字典的value](#32-遍历字典的value)
    - [3.3 变量字典的键值对](#33-变量字典的键值对)
    - [3.4 字典遍历小结](#34-字典遍历小结)

<!-- /TOC -->
---

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
# 2 字典的基本操作
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
## 2.2 字典元素的访问
有如下三种方式访问字典的键值对：
- `d[key]`: 返回key对应的value，key不存在抛出KeyError异常
- `dict.get(key[, default])`: 返回key对应的value，key不存在返回缺省值，如果没有设置缺省值赶回None
- `dict.setdefault(key[, default])`: 返回key对应的值value，key不存在，添加key:value键值对(value设置为default)，并返回value，如果default没有设置缺省为None
```python
In [32]: dic = {'a':1, 'b':2, 'c':3, 'd':4}                                                                                            
In [33]: dic['a']                                                                                                                      
Out[33]: 1
In [34]: dic['e']          # 不存在'e'这个key，所以直接访问会出现异常                                                                                                               
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-34-87d22c709971> in <module>
----> 1 dic['e']

KeyError: 'e'
In [35]: dic.get('a')     # key存在，则返回对应的值                                                                                                                
Out[35]: 1
In [36]: dic.get('e')     # 不存在，默认会返回None，ipython优化了None的输出，所以这里无显示                                                                                                             
In [37]: dic.get('e','not exist')     # 不存在时，由指定的default进行返回                                                                                                 
Out[37]: 'not exist'
In [38]: dic.setdefault('a', 'ok?')   # 设置a的值为ok?,a存在，所以返回a对应的值1                                                                                                 
Out[38]: 1
In [39]: dic.setdefault('e', 'ok?')   # 设置e的值为ok?，e不存在，所以设置并放回value(key不存在时等同于设置并访问了)                                                                                                 
Out[39]: 'ok?'
In [40]: dic                                                                                                                           
Out[40]: {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 'ok?'}
```
>__当然字典也可以被迭代访问,后面介绍__
## 2.3 字典的增删改
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于字典可变、非线性、无序的特性，而且字典的key必须是可hash的对象，查找某个key也是直接hash()，然后找到对应的房间的，所以我们对它某个key的修改可以理解为是`O(1)`的操作，效率很高，主要有以下几种方法。
- `d[key] = value`: 将key对应的值修改为value，key不存在添加新的key:value对。
- `dic.update([other]) --> None`: 使用另一个字典的k,v对更新本字典，key不存在时添加，存在时则覆盖，所以不会返回新的字典，属于原地修改。
- `dic.pop(key[, default])`: key存在，移除它，并返回它的value，key不存在返回指定的default，如果default未指定，那么会返回KeyError异常。
- `dic.popitem()`: 移除并返回一个任意的键值对，字典为empty时，抛出KeyError异常。
- `dic.clear()`: 清空字典。
- `del dic['a']`: 通用的删除变量方法。
```python
In [41]: dic                                                                                                                           
Out[41]: {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 'ok?'}

In [42]: dic['a'] = 1000                                                                                                               
In [43]: dic                                                                                                                           
Out[43]: {'a': 1000, 'b': 2, 'c': 3, 'd': 4, 'e': 'ok?'}

In [44]: dic2 = {'a':200,'f':50}                                                                                                       
In [45]: dic.update(dic2)                                                                                                              
In [46]: dic                                                                                                                           
Out[46]: {'a': 200, 'b': 2, 'c': 3, 'd': 4, 'e': 'ok?', 'f': 50}

In [48]: dic.pop('a')             # 弹出一个key'a'，key存在返回key对应的value                                                                                                   
Out[48]: 200

In [49]: dic.pop('g')              # 弹出一个key'g'，key不存在，又没有指定default，则会报KeyError异常                                                                                                        
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-49-311c3ba80251> in <module>
----> 1 dic.pop('g')

KeyError: 'g'

In [50]: dic.pop('g','not exist')       # 指定了default，当key不存在时，会返回指定的default值                                                                                               
Out[50]: 'not exist'

In [51]: dic.popitem()          # 弹出一个key:valyue键值对，返回对象是元组形式                                                                                                       
Out[51]: ('d', 4)
In [52]: dic                                                                                                                           
Out[52]: {'b': 2, 'c': 3, 'e': 'ok?', 'f': 50}

In [53]: del dic['e']                                                                                                                  
In [54]: dic                                                                                                                           
Out[54]: {'b': 2, 'c': 3, 'f': 50}

In [55]: dic.clear()                                                                                                                   
In [56]: dic                                                                                                                           
Out[56]: {}
```
# 3 字典遍历
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Python中，我们所说的基本数据结构：字符串、元组、列表，集合，包括字典，都可以认为是一个容器箱子，只要是容器，我们就可以进行遍历(是否有序和是否可以遍历没有必然关系，只不过有序的话是顺序拿出，而无序则是随机拿出)，我们可以使用多种方式对字典进行迭代遍历，但是有些地方和其他类型不同。  

## 3.1 遍历字典的key
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`dic.keys()`: --> `dict_keys` --> 返回字典dic的所有key组成的一个dict_keys视图集合(__`类set结构，不会生成新的内存空间`__)。
```python
In [64]: dic = {'a':1, 'b':2, 'c':3, 'd':4}                                                                                            

In [65]: for key in dic: 
    ...:     print(key)                                                                                                                      
a
d
c
b

In [66]: for key in dic.keys(): 
    ...:     print(key)                                                                                                                            
a
d
c
b

In [67]: dic.keys()                                                                                                                    
Out[67]: dict_keys(['a', 'd', 'c', 'b'])
```
> 迭代字典就是在迭代字典的key，所以直接迭代字典和使用字典的keys()方法返回一个keys的视图然后再迭代，是一样的效果。  

## 3.2 遍历字典的value
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`dic.values()`: --> `dict_values` --> 返回字典dic的所有values组成的一个dict_values视图集合(__`类set结构，不会生成新的内存空间`__)。
```python
In [69]: for i in dic: 
    ...:     print(dic[i])                                                                                                                           
1
4
3
2

In [70]: for i in dic.values(): 
    ...:     print(i)                                                                                                                           
1
4
3
2

In [71]: dic.values()                                                                                                                  
Out[71]: dict_values([1, 4, 3, 2])
```
>可以首先遍历字典的key，然后再通过key来访问对应的value，也可以通过values()直接访问values。  

## 3.3 变量字典的键值对   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`dic.items()`: --> `dict_items` --> 返回字典dic的所有的key和value(每个key和value的键值对由元组表示)组成的一个dict_items视图集合(__`类set结构，不会生成新的内存空间`__)。
```python
In [75]: for i in dic.items(): 
    ...:     print(i) 
    ...:                                                                                                                               
('a', 1)
('d', 4)
('c', 3)
('b', 2)

In [77]: for key,value in dic.items(): 
    ...:     print('key:{} value:{}'.format(key,value)) 
    ...:                                                                                                                               
key:a value:1
key:d value:4
key:c value:3
key:b value:2

In [78]: 
```
>由于返回的每个键值对为元组格式，那么利用我们前面学的封装与结构，可以很方便的获取key和它对应的value
## 3.4 字典遍历小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Python3中，keys、values、items方法返回一个类似一个生成器的可迭代对象，不会把函数的返回结果复制到内存空间中
