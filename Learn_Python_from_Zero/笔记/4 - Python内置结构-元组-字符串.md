<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 元组概念](#1-元组概念)
    - [1.1 元祖的特点](#11-元祖的特点)
    - [1.2 元组的定义](#12-元组的定义)
    - [1.3 元组的访问](#13-元组的访问)
    - [1.4 元组的查询](#14-元组的查询)
- [2 命名元组](#2-命名元组)
- [3 字符串](#3-字符串)
    - [3.1 字符串的基本操作](#31-字符串的基本操作)
        - [3.1.1 字符串的访问](#311-字符串的访问)
        - [3.1.2 字符串的拼接](#312-字符串的拼接)
    - [3.2 字符串分割](#32-字符串分割)
    - [3.3 字符串大小写](#33-字符串大小写)
    - [3.4 字符串排版](#34-字符串排版)
    - [3.5 字符串修改](#35-字符串修改)
    - [3.6 字符串查找](#36-字符串查找)
    - [3.7 字符串判断](#37-字符串判断)
    - [3.8 字符串格式化](#38-字符串格式化)
        - [3.8.1 C语言格式化](#381-c语言格式化)
        - [3.8.2 format格式化](#382-format格式化)
        - [3.8.3 对齐](#383-对齐)
        - [3.8.9 小数点与进制](#389-小数点与进制)
- [4 切片](#4-切片)
    - [`4.1 切片赋值`](#41-切片赋值)

<!-- /TOC -->
---

# 1 元组概念

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;元组（类型为 `tuple`）和列表十分相似,但是元组和字符串一样是不可变的。
## 1.1 元祖的特点
- 元组可以存储一系列的值，使用`小括号`来定义，是一个`有序`的元素的集合。
- 元组内的元素是`不可变`的
- 当元组内嵌套列表这种引用类型时，元组的不可变表示的是执行这个列表的内存地址不会变，当直接操作元组嵌套的列表时，是可以进行修改的
## 1.2 元组的定义
__格式__
```python
tuple() -> 工厂函数，用于创建并返回一个空元组
tuple(iterable) -> 使用可迭代对象的元素，来初始化一个元组
```
例子
```python
In [18]: t=(1)    # 会认为()只是优先级
In [19]: type(t) 
Out[19]: int
In [20]: t=(1,)
In [21]: type(t)
Out[21]: tuple     # tuple表示元组类型
 
#引用其他元组
In [22]: a=(1,2,3)
In [23]: t=('123',a)
In [24]: t
Out[24]: ('123', (1, 2, 3))
 
#通过索引只引用某一个值
In [27]: t=('123',a[1])
In [28]: t
Out[28]: ('123', 2)

# tuple接受一个可迭代对象转换为元组
In [35]: tuple(range(1,7,2))                                                                                    
Out[35]: (1, 3, 5)
```
## 1.3 元组的访问
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;元组和列表在内存中的格式是相同的，都是线性顺序结构，所以我们可以像列表一样，使用`索引访问`元组的元素，其中元组支持`正索引`和`负索引`，同样不支持索引超界，会提示`IndexError`。
```python
In [49]: b = (1,2,3)                                                                                                
In [50]: b[1]                                                                                                       
Out[50]: 2
In [51]: b[-1]                                                                                                        
Out[51]: 3
```
>__&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是在对元组进行修改的时候，元组本身是无法进行修改的，但当元组内嵌套的是列表这种引用类型时，元组的不可修改指的是元组中存储的嵌套列表的内存地址不可修改，但你可以对这快内存地址里的数据进行修改，因为这是列表的特性。__
```python
In [44]: a                                                                                                           
Out[44]: (1, 2, [1, 2], 1, 2, [1, 2], 1, 2, [1, 2])
In [45]: a[2][0] = 100         # 可以对嵌套的列表进行赋值操作                                                                                       
In [46]: a                                                                                                           
Out[46]: (1, 2, [100, 2], 1, 2, [100, 2], 1, 2, [100, 2])
In [47]: a[3] = 100      # 修改指向是不被允许的                                                                                             
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-47-2b62bbdeb061> in <module>
----> 1 a[3] = 100

TypeError: 'tuple' object does not support item assignment

```
## 1.4 元组的查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们通过使用元组的`index`方法和`count`来获取和统计元组中的元素。
```python
T.index(value, [start, [stop]]) --> integer -- 返回元组内匹配value的第一个元素的index,

T.count(value) --> integer -- 统计value在元组中出现的次数，不存在时，则返回0
```
__注意：t.index和t.count因为要遍历列表所有，时间复杂度都是O(n),随着列表的元素增加，而效率下降__ 
```python
In [4]: a=('1','2','3')
In [7]: a.count("4")     # 不存在，返回0
Out[7]: 0
In [8]: a.count("1")
Out[8]: 1
 
# a.index(value) 用来返回value在元组中的索引，如果value不在元组中，则会报错。如果有多个，默认返回第一个（可以指定从哪个索引开始查找到某个索引结束，指定范围区间）
In [4]: a=('1','2','3')
In [9]: a.index('1')
Out[9]: 0
In [10]: a.index('3')
Out[10]: 2
In [57]: a = ('1','2','3')                                                                                            
In [58]: a.index('4')     # 不存在，就会报错                                                                                             
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-58-dca64b8e9162> in <module>
----> 1 a.index('4')

ValueError: tuple.index(x): x not in tuple
>>> t1
('a', 'b', 'a', 'b', 'a', 'b', 'a', 'b')
>>> t1.index('a',5,7)    # 在指定的区间内查找
6
```
# 2 命名元组
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;命名元组是元组的子类，所以它也是无法进行修改的，它的特点是可以针元组的对字段进行命名。  
__格式__
```python
namedtuple(typename, field_names, *, verbose=False, rename=False, module=None) --> 返回一个拥有命名字段的 新的元组的子类
```
__常用参数含义__
- `typename`: 一般和命名元组的名称相同。
- `field_names`: 可以是空白字符或逗号分隔的字段的字符串，可以是字段的列表
> namedtuple 存放在 collections 包中，所以需要先进行导入
```python
>>> from collections import namedtuple    
>>> Point = namedtuple('Point', ['x', 'y'])  # 创建一个名为Point的命名元组类，其中含有两个字段
>>> p = Point(11, 22)             # 创建一个实例，11会传递给x，22会传递给y。
>>> p[0] + p[1]                     # 可以通过索引访问
33
>>> p.x + p.y                       # 也可以通过字段名访问
33
>>> p.x = 33            # 没有办法进行修改的                                                                                         
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-63-dac7085722b7> in <module>
----> 1 p.x = 33

AttributeError: can't set attribute
```
# 3 字符串
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串是Python中比较重要的数据类型，是以单引号'或双引号"括起来的任意文本，比如'abc'，"xyz"等等。请注意，''或""本身只是一种表示方式，不是字符串的一部分，因此，字符串'abc'只有a，b，c这3个字符。如果'本身也是一个字符，那就可以用""括起来，比如"I'm OK"包含的字符是I，'，m，空格，O，K这6个字符。有三种方法定义字符串：`单引号`，`双引号`，`三引号`，需要注意的是字符串是不可变对象，并且从Python3起，字符串就是`Unicode`类型。   
定义方式
```python
str1='this is string'
str2="this is string"
str3='''this is string'''   # 也可以是三个双引号，三个引号可以多行注释但是不能单双混合,三重引号除了能定义字符串以外，还可以表示注释。
str4='hello\n world' # 在print打印字符串的时候\n会被当作换行符进行打印
str5=r'hello\n world' # 前面使用了r对字符串进行整体转义,所见即所得
str6='hellow\\nworld' # 当然使用\也可以对特殊符号进行脱义
str7=R'hello\nworld' # 和r相同
```
## 3.1 字符串的基本操作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python的字符串是一个有序序列，所以他可以和列表一样使用下标来访问元素，但是由于它是不可变类型，所以无法对字符串中的某个字符进行修改，下面介绍下字符串的基本操作。
>Python中没有字符的概念，严格来讲，说字符是不准确的，字符串是由一个个字符串(字符)组成的，虽然听起来很别扭，但真的就是这样 - -!。
### 3.1.1 字符串的访问
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串和列表是相似，都是顺序的线性结构，所以它可以被索引，也可以被遍历。字符串的索引类似数组的下标：
```python
In [3]: a='1234567'
In [4]: a[0] --> # 下标从0开始，0表示第一个数
Out[4]: '1'
In [5]: a[3] --> # 表示第四个数
Out[5]: '4'
In [3]: a[1] = 100   # 字符串没有办法被修改                                                                                                  
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-3-8554a2b011c3> in <module>
----> 1 a[1] = 100

TypeError: 'str' object does not support item assignment

In [4]:
In [6]: for i in a:       # 可以被for循环进行迭代
   ...:     print(i) 
   ...:                                                                                                             
1
2
3
4
5
6
7
In [7]: list(a)        # 可以被当作一个可迭代对象传给list，转换为一个列表                                                                                               
Out[7]: ['1', '2', '3', '4', '5', '6', '7']

```
### 3.1.2 字符串的拼接
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当我们需要把多个字符串连接在一起，那么就需要对字符串进行拼接，python提供了`join`方法，`+号`，以及`*号`，使我们方便的完成需求。
```python
S.join(iterable) -> str  --> 使用s对可迭代对象进行拼接，返回拼接后的字符串。
```
- join: S可以为任意字符，包括空。可迭代对象中的元素`必须是字符串`类型
- +: 把两个字符串直接进行连接，返回一个新的字符串
- *: 把字符串重复复制N次，返回一个新的字符串
```python
In [11]: str1                                                                                                         
Out[11]: ['h', 'e', 'l', 'l', 'o']
In [12]: ''.join(str1)                                                                                                
Out[12]: 'hello'
In [13]: str2 = ''.join(str1)                                                                                          
In [14]: str2                                                                                                         
Out[14]: 'hello'
In [17]: '-'.join(str1)      # 使用-进行拼接                                                                                           
Out[17]: 'h-e-l-l-o'
In [15]: str2 * 2                                                                                                     
Out[15]: 'hellohello'
In [16]: str2 + str2                                                                                                  
Out[16]: 'hellohello'

In [18]: lst = ['1',['1','2'],'3']                                                                                    
In [19]: ''.join(lst)                   # lst的第1个元素是列表，不是字符串，没办法进行拼接，会报错                                                                              
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-19-58ac5d2512ec> in <module>
----> 1 ''.join(lst)

TypeError: sequence item 1: expected str instance, list found

In [20]

```
## 3.2 字符串分割
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串中有关于字符分割功能的主要有两类，`split`类和`partition`类，他们分别适用于不用的场景。但用的比较多的是`split`。
- split类：将字符串按照分割符分隔成若干字符串，并返回列表
- partition类：将字符串按照分割符分割成2段，返回这2段和分隔符组成的三元组
```python
S.split(sep=None, maxsplit=-1) -> list of strings -->  从左至右对字符串s进行切割，分割符为sep，默认为尽可能多的空字符，maxsplit表示分割几次，默认为-1，全部进行分割，返回一个切割后的列表。

S.partition(sep) -> (head, sep, tail)  --> 从左至右对字符串s进行切割，必须指定一个切割符sep，返回一个三元组，其中中间的元素为分割符，第一个和最后一个元素为按照分隔符分开后的前后两个元素。当分隔符无法对字符串进行分割时，返回的是 字符串，空，空，组成的三元组。
```
例子
```python
In [20]: s = "hello world I am Daxin"                                                                                 
In [21]: s.split()       # 默认使用空格进行分割                                                                                              
Out[21]: ['hello', 'world', 'I', 'am', 'Daxin']
In [23]: s.split('o')         # 使用字母o进行分割                                                                                        
Out[23]: ['hell', ' w', 'rld I am Daxin']   
In [24]: s.split('o',1)                   # 使用字母o进行分割，并且只分割1次                                                                          
Out[24]: ['hell', ' world I am Daxin']
In [25]: s.split(sep='o',maxsplit=1)         # 也可以指定关键字进行传参                                                                         
Out[25]: ['hell', ' world I am Daxin']
In [26]: s.partition(' ')                  # 使用' '进行分割，返回三元组                                                                           
Out[26]: ('hello', ' ', 'world I am Daxin')
In [27]: s.partition('o')                                                                                             
Out[27]: ('hell', 'o', ' world I am Daxin')

# 当分割符不存在时
In [29]: s = "helloworldIamDaxin"                                                                                     
In [30]: s.split()     # 一定会返回一个列表，如果没有被切分，那么会发那会一个元素的列表                                                                                                   
Out[30]: ['helloworldIamDaxin']
In [31]: s.partition(' ')         # 一定会返回一个三元组，如果没有被切分，那么会从字符串的最右边切开，形成一个三元组，和 一个空字符组成的列表                               
Out[31]: ('helloworldIamDaxin', '', '')
In [32]: s.partition('12')                                                                                           
Out[32]: ('helloworldIamDaxin', '', '')

```
当然split类还包含了其他两个方法：
```python
S.rsplit(sep=None, maxsplit=-1) -> list of strings --> 功能与split相同，只不过从右开始

S.splitlines([keepends]) -> list of strings  --> 按照行来切分，keepends表示是否保留换行符，True表示保留，False表示不保留，默认为False

In [33]: s = 'I am Super Man'                                                                                         
In [34]: s.rsplit('u')      # 不指定分割次数，一般和split是一样的效果                                                                                             
Out[34]: ['I am S', 'per Man']
In [35]: s.rsplit('a')                                                                                                
Out[35]: ['I ', 'm Super M', 'n']
In [37]: s.rsplit(sep='a',maxsplit=1)     #  当指分割1次时，会从右边开始切开                                                                             
Out[37]: ['I am Super M', 'n']
In [40]: s = 'hello\nworld\rI\nam\r\ndaxin'                                                                           
In [41]: s.splitlines()                                                                                               
Out[41]: ['hello', 'world', 'I', 'am', 'daxin']
In [42]: s.splitlines(True)             # 默认不保留分隔符，True表示保留分隔符                                                                             
Out[42]: ['hello\n', 'world\r', 'I\n', 'am\r\n', 'daxin']   
```
>partition类和split相似，还有个`rpartition`函数，也是从右开始截取，需要注意的是，当分隔符无法对字符切分时，返回的是`空`，`空`，`字符串`，组成的三元组。
## 3.3 字符串大小写
- `upper`：将字符串转换为大写字母
- `lower`：将字符串转换为
- `swapcase`: 大小写对调
- `capitalize`：转换成首字母大写的单词格式
- `title`: 转换成每个单词首字母大写的标题模式
```python
In [51]: s = 'hElLo wORld i aM dAxin'                                                                                 
In [52]: s.upper()                                                                                                    
Out[52]: 'HELLO WORLD I AM DAXIN'
In [53]: s.lower()                                                                                                    
Out[53]: 'hello world i am daxin'
In [54]: s.swapcase()                                                                                                 
Out[54]: 'HeLlO WorLD I Am DaXIN'
In [55]: s.capitalize()                                                                                               
Out[55]: 'Hello world i am daxin'
In [56]: s.title()                                                                                                    
Out[56]: 'Hello World I Am Daxin'    
```
## 3.4 字符串排版
- `center(width [,fillchar])`: 居中显示，参数width表示整体宽度，fillchar表示填充字符，默认填充字符为空
- `zfill(width)`：居右显示，参数width表示整体宽度，使用0进行填充
- `ljust(width [, fillchar])`：左对齐，width表示整体宽度，fillchar表示填充字符
- `rjust(width [, fillchar])`：右对齐，width表示整体宽度，fillchar表示填充字符
```python
In [61]: a                                                                                                            
Out[61]: 'abc'
In [62]: a.ljust(20,'-')                                                                                              
Out[62]: 'abc-----------------'
In [63]: a.rjust(20,'-')                                                                                              
Out[63]: '-----------------abc'
In [64]: a.center(10,'-')                                                                                             
Out[64]: '---abc----' 
```
## 3.5 字符串修改
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;what？你前面说字符串是不可变的吧，为什么这里又说字符串的修改？你在逗我吗。呵呵哒。
```python
S.replace(old, new[, count]) -> str  --> 对字符串S进行查找，将指定的old字符串转换为new字符串，count表示替换的次数，默认表示重复替换所有

S.strip([chars]) -> str --> 将字符串s进行处理，从字符串的两边删除掉匹配chars的字符串，chars可以是多个单字符，默认是所有空白字符(\n,\r\n,\r,\t等等都包含)

```
__注意：replace的替换是`生成一个新的字符串`,而`不是修改原字符串`，这也是字符串修改的原理__
```python
In [3]: s                                                                                                             
Out[3]: ' \n\t Hello World \n\r'
In [4]: s.strip()     # '不指定chars，默认是任意多个空白字符                                                                                                
Out[4]: 'Hello World'
In [5]: s.strip(' \n\tHd')       # 如果指定了chars，那么就挨个使用char进行匹配去除                                                                                     
Out[5]: 'ello World \n\r'
In [6]: s.strip(' \n\rHd')                                                                                            
Out[6]: '\t Hello Worl'
In [7]: s.replace('World','Daxin')                                                                                    
Out[7]: ' \n\t Hello Daxin \n\r' 
In [8]: s.replace('o','O')         # 默认从头到尾进行替换                                                                                     
Out[8]: ' \n\t HellO WOrld \n\r'
In [9]: s.replace('o','O',1)         # 指定替换1次                                                             
Out[9]: ' \n\t HellO World \n\r'  
```
## 3.6 字符串查找
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们有很多的时候要判断关键字是否存在一个字符串中，那么我们就需要在字符串中`遍历`查找，是否有匹配的字符串。python提供了`find`,`rfind`,`index`,`count`等函数用于完成需求。
```python
S.find(sub[, start[, end]]) -> int  --> 从左开始在字符串S中查找sub字符串，其中起始位start，结束位end，默认为整个字符串,返回找到的字符串的开头索引位，如果没有找到，那么会返回-1

S.rfind(sub[, start[, end]]) -> int  --> 从右开始在字符串S中查找sub字符串，其中起始位start，结束位end，默认为整个字符串,返回找到的字符串的开头索引位，如果没有找到，那么会返回-1

S.index(sub[, start[, end]]) -> int  --> 从左开始在字符串S中查找sub字符串，其中起始位start，结束位end，默认为整个字符串,返回找到的字符串的开头索引位，如果没有找到，那么会报异常

S.count(sub[, start[, end]]) -> int  --> 从左开始在字符串S中统计sub字符串出现的次数，其实起始位置为Start,结束位为end，默认为整个字符串。没有找到返回0
```
__index和count方法由于是遍历查找，所以时间复杂度都是O(n),会随着字符串序列的数据规模的增大，而效率下降，如果没要在字符串中进行查找，还是建议使用`find`函数。__
```python
In [15]: s = 'abc abc abc'                                                                                            
In [16]: s.find('a')                                                                                                  
Out[16]: 0
In [17]: s.find('a',1,-1)      # 指定区间， 注意这里-1表示最后1位，但是不包含-1，类似于[1,-1)                                                                                      
Out[17]: 4
In [18]: s.find('a',-1,-15)                                                                                           
Out[18]: -1
In [19]: s.rfind('a')                                                                                                 
Out[19]: 8
In [20]: s.rfind('a',2,-1)                                                                                            
Out[20]: 8
In [21]: s.rfind('c',2,-1)                                                                                            
Out[21]: 6
In [22]: s.rfind('c',2,-100)    # end点超出范围，会无法找到，start，end表示起始和终止，最好不要使用负数表示区间                                                                                     
Out[22]: -1
In [23]: s.index('a')                                                                                                 
Out[23]: 0
In [24]: s.index('a',2)     # 从索引为2，开始向右查找                                                                                          
Out[24]: 4
In [25]: s.index('e')       # 没找到，直接报异常                                                                                     
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-25-90b1c28da6f0> in <module>
----> 1 s.index('e')

ValueError: substring not found
In [26]:    
```
## 3.7 字符串判断
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python的字符串对象提供了两个函数，用于对字符串的起始位和结尾位来进行匹配，它们是`startswith`和`endswith`。
```python
S.startswith(prefix[, start[, end]]) -> bool  --> 判断字符串prefix是否是字符串S的起始字符串，start表示起始位置，end表示末尾，默认为0，即整个字符串S,返回bool类型。

S.endswith(suffix[, start[, end]]) -> bool  --> 判断字符串prefix是否是字符串S的结束字符串，start表示起始位置，end表示末尾，默认为0，即整个字符串S,返回bool类型。
```

```python
In [32]: s                                                                                                            
Out[32]: 'abc abc abc'
In [33]: s.startswith('bc',1,-1)    # 从s的[1,-1)开始判断'bc'是否是开头                                                                                     
Out[33]: True
In [34]: s.endswith('bc',2,-1)      # 从s的[2,-1)开始匹配'bc'是否是结尾                                                                                 
Out[34]: False                     # 这里-1不包含，所以返回False
In [35]: s.endswith('bc',3,7)                                                                                         
Out[35]: True
In [36]: s.startswith('abc')                                                                                          
Out[36]: True
In [37]: s.endswith('bc')                                                                                             
Out[37]: True 
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了判断开始和结尾，Python的字符串还提供了部分函数，用来判断字符串内的元素类型,比如判断字符串是否是纯数字组成？是否是纯字母组成等，这些函数的返回值统一都为`bool`型，可以作为`if语句的条件表达式`。
```python
str.isalpha() # 是否是字母吗
str.isalnum() # 是数字和字母的组合吗
str.isdigit() # 是否全是十进制数字，int
str.isdecimal() # 判断是否是数字类型，包含float，但不包含负数
str.islower() # 判断字符串是否全是小写字母
str.isupper() # 判断字符串是否全是大写字母
str.isspace() # 是否是空白字符
str.isnumberic() # 判断是否是正整数
str.isidentifier() # 是否是一个合规的变量标识符
```
## 3.8 字符串格式化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串格式化是我们需要重点掌握的东西，在早期的Python中使用的是C语言风格的字符串替换，使用起来比较难看，不符合python的风格（当然是我的猜测，哈哈）。后来Python推荐使用内置的format函数来对字符串进行格式化。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串格式化是一种拼接字符串输出样式的手段，更灵活方便，之前我们使用join和+来对字符串进行拼接。
- join：只能使用分隔符，且要求被拼接的是可迭代对象且元素必须是来字符串类型
- +: 使用起来比较方便，但是非字符串需要先转换为字符串才可以进行拼接。  
### 3.8.1 C语言格式化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Python 2.5版本以前，只能使用printf style风格的print输出，这种风格来自于C语言的printf函数，它有如下格式要求。(__不建议使用，了解即可，Pythoner还是理解format就好。__)
1. 占位符：使用%和格式字符串组成，例如%s，%d等。使用`s`时，内部其实会调用`str()`函数进行转换
2. 占位符中还可以插入修饰字符，例如`%03d`表示打印3个位置，不够的话，前面补0
3. format % value 格式字符串和被格式字符串之间使用%分割
4. values只能是一个对象，或是一个与格式字符串占位符数量相等的元组，或一个字典
```python
In [38]: 'I am %03d' % 20        # 表示3为数字，不够的话高位补0                                                                                     
Out[38]: 'I am 020'
In [39]: 'I like %s' % 'Python'      # 字符串格式化
In [50]: 'I am %s' % 20      # 20会被str作用后，传递给字符串                                                                                
Out[50]: 'I am 20'
Out[39]: 'I like Python'
In [41]: '%3.2f%%,0x%x,0X%02X' % (89.7654,10,15)     # 3.2f表示最长3为，小数点后精度为2位，当数字大时整体长度会被撑开，x表示16进制，02X表示两位显示，高位补0                                                                 
Out[41]: '89.77%,0xa,0X0F'
In [45]: "I am %-5d" % 20                                                                                             
Out[45]: 'I am 20   '
In [46]: "I am %5d" % 20                                                                                              
Out[46]: 'I am    20'  
```
### 3.8.2 format格式化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python中推崇使用format函数来对字符串进行格式化。
```python
'{}{XXX}'.format(*args, **kwargs)  -> str  --> 函数的一般格式，{}表示占位符，使用format中的参数进行传递
```
format非常灵活，下面是基本使用方法说明：
1. `args`是可变的位置参数，是一个元组
2. `kwargs`是可变关键字参会苏，是一个字典
3. `花括号`表示`占位符`
4. `{}`表示按照顺序匹配`位置参数`，`{n}`表示取位置参数中`索引为n的值`
5. `{xxx}` 表示在关键字参数中搜索名称一致的值，`kwargs`必须放在`位置参数的后面`
6. `{{}}`表示打印花括号
```python
In [52]: '{}:{}'.format('10.0.0.13','8888')       # 按照位置格式化，第一个元素给第一个括号，第二个元素给第二个括号                                                                    
Out[52]: '10.0.0.13:8888'            # 
In [53]: '{host}:{}:{}'.format('10.0.0.13','8888',host='daxin')   # 命名格式化，host表示只获取关键字为host的值来填充,其他没有指定关键字的占位符，则按照位置参数进行传递，并格式化显示                                                    
Out[53]: 'daxin:10.0.0.13:8888'
In [54]: '{hostname} {}:{}'.format('10.0.0.13','8888',hostname='daxin')                                               
Out[54]: 'daxin 10.0.0.13:8888'        # 访问元素的方式进行字符串格式化(非常不常用)
In [55]: '{0[0]}:{0[1]}'.format(['10.0.0.13','8888'])                                                                 
Out[55]: '10.0.0.13:8888'

In [57]: from collections import namedtuple                                                                           
In [58]: Point = namedtuple('_Point',['x','y'])                                                                       
In [59]: p = Point(4,5)                                                                                               
In [60]: "{{{0.x},{0.y}}}".format(p)   # 两个花括号重叠表示打印一个花括号，由于p对象含有x和y属性，所以可以在字符串格式化时直接引用                                                                            
Out[60]: '{4,5}'
```
### 3.8.3 对齐
当然字符串还提供了多种的对齐方式，便于我们对输出内容做一个简单的优化。
- `<`：左对齐（默认）
- `>`：右对齐
- `^`: 居中显示  
>__对齐方式需要在占位符内使用`：号`进行分割__
```python
In [61]: '{:5}'.format('2')     #   打印以讹字符串，这个字符串占5位，默认靠左对齐                                                                                                                
Out[61]: '2    '
In [62]: '{:>5}'.format('2')     # > 表示向右对齐                                                                                                                  
Out[62]: '    2'
In [65]: '{:0<5}'.format('2')     # 字符串站5位，向左对齐，其他为使用0填充(可以简写为'{:<05}')                                                                                                          
Out[65]: '20000'
In [66]: '{:>05}'.format('2')                                                                                                                       
Out[66]: '00002'
In [71]: '{:*>5}'.format('2')     # > 表示向右对齐，其他位用*填充                                                                                                           
Out[71]: '****2'

In [67]: '{:0^5}'.format('2')     # 居中显示，使用0进行填充                                                                                                                 
Out[67]: '00200'
In [69]: '{:*^5}'.format('2')                                                                                                                       
Out[69]: '**2**'
```
>__当填充符为数字的时候，可以与宽度写在一起，比如 `{:0<5} --> {:<05} , {:0^5} --> {:^05}`__
### 3.8.9 小数点与进制
虽然用的不多，还是这里还是举例说明一下进制和小数的使用方法
```python
In [74]: "int: {0:d}; hex: {0:x}; oct: {0:o}; bin: {0:b}".format(42)                                                                                
Out[74]: 'int: 42; hex: 2a; oct: 52; bin: 101010'
In [75]: "int: {0:d}; hex: {0:#x}; oct: {0:#o}; bin: {0:#b}".format(42)                                                                             
Out[75]: 'int: 42; hex: 0x2a; oct: 0o52; bin: 0b101010'
In [76]: octets = [10,0,0,13]                                                                                                                      
In [78]: '{:02X}{:02X}{:02X}{:02X}'.format(*octets)                                                                                                 
Out[78]: '0A00000D'
In [79]: '{:02X}-{:02X}-{:02X}-{:02X}'.format(*octets)                                                                                              
Out[79]: '0A-00-00-0D'  
```
- `d`: 表示十进制
- `x`: 表示十六进制
- `o`: 表示八进制
- `b`: 表示二进制
- `F`: 表示浮点型
- `#`: 表示添加进制前缀
- `*[1,2,3]`: 表示把列表中的元素结构出来：`*[1,2,3]` --> `1,2,3`
```python
In [82]: "{}".format(3**0.4)    # 默认按照字符串打印                                                                                                                        
Out[82]: '1.5518455739153598'
In [83]: "{:f}".format(3**0.4)         # f表示填充位为小数，小数是有精度的                                                                                                              
Out[83]: '1.551846'
In [84]: "{:02f}".format(3**0.4)         # 表示小数的长度为2，但是如果小数的位数超过2，会直接撑开                                                                                                           
Out[84]: '1.551846'
In [85]: "{:10f}".format(3**0.4)       #  表示小数的长度为10，默认是右对齐                                                                                                            
Out[85]: '  1.551846'
In [86]: "{:<10f}".format(3**0.4)          # 左对齐                                                                                                         
Out[86]: '1.551846  '
In [89]: "{:.2f}".format(3**0.4)          # .2f表示 小数点后取两位的浮点型                                                                                                          
Out[89]: '1.55'
In [90]: "{:3.2f}".format(3**0.4)          # 总长3为，小数点后保留2位，若长度超出，则撑开                                                                                                         
Out[90]: '1.55'
In [91]: "{:2.2f}".format(3**0.4)                                                                                                                   
Out[91]: '1.55'
In [92]: "{:2.2%}".format(3**0.4)          # 使用百分比显示                                                                                                         
Out[92]: '155.18%'
```
# 4 切片
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说到切片那么不得不提线性结构，为什么？因为`线性结构`都可以进行`切片操作`，除了切片，线性结构其他的特点还有
- 可迭代(for val i ...)
- len()可以获取长度
- 可以通过下标进行访问(有序)
- 列表、元组、字符串、bytes、bytearray都属于线性结构，所以都可以被切片。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那什么是切片？我们说通过索引区间访问线性结构一段数据的方法就叫做切片，需要注意的`是切片操作会引起内存复制，当对一个过于庞大的线性结构进行切片的时候，请慎重考虑内存使用率的问题`。切片的表达方式和基本特点有：
1. 格式：sequence[start:stop:[,step=1]] 返回[start, stop, step=1)的`前闭后开`子序列。
2. 支持负索引。注意方向问题
3. 当start为0或stop为0时，可以省略。`[:]`表示复制原线性结构数据(注意当对象为list时，属于`浅copy`)
4. 超过上届(右)，则取到末尾;超过下届(左)，则取到开头。
5. start一定要在stop的左边
```python
In [72]: a = 'hello world , My name is daxin'                                                                                                       
In [73]: a[2:-1]                                                                                                                                  
Out[73]: 'llo world , My name is daxi'
In [74]: a[2:]                                                                                                                                      
Out[74]: 'llo world , My name is daxin'
In [75]: a[-100:]                                                                                                                                   
Out[75]: 'hello world , My name is daxin'
In [76]: a[10:-100]     # stop位置在start左边，所以没办法取出，如果实在想要倒着取，那么需要使用步长                                                                                                                             
Out[76]: '' 
In [77]: a[10:-100:-1]      # 负步长就可以形成开闭区间，注意是从起始位开始按照step取的(所以会倒序排列返回)                                                                                                                        
Out[77]: 'dlrow olleh'
```
注意：
- 切片并不会对原数据进行修改，会返回新的数据
- 如果不是用变量接受，那么就会被标记为待回收
- 由于是新生成的数据，所以`内存地址`和原数据内存地址`一定不相同`。
## `4.1 切片赋值`
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;既然可以进行切片，那么就会引申出来，是否可以进行切片赋值,什么是切片赋值？它该如何表示？下面以列表例进行说明。  
- 切片操作写在等号的左边
- 被插入的可迭代对象在等号右边
```python
In [79]: lst = list(range(10))                                                                                                                      
In [80]: lst                                                                                                                                        
Out[80]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
In [81]: lst[1:3]                                                                                                                                  
Out[81]: [1, 2]

In [82]: lst[1:3] = 1                                                                                                                               
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-82-7fef59136c7e> in <module>
----> 1 lst[1:3] = 1

TypeError: can only assign an iterable

In [83]: lst[1:3] = [100,200]                                                                                                                       
In [84]: lst                                                                                                                                        
Out[84]: [0, 100, 200, 3, 4, 5, 6, 7, 8, 9]  
```
仔细看上面示例代码会发现几个问题：
1. lst[1:3] = 1 表示切片赋值
2. `切片赋值`，赋的值必须是`一个可迭代对象`
3. __`切片赋值改变了原数据`__
4. 字符串、元组这类不可变的元素，无法使用切片赋值(理由请看3)
> 当我们使用切片时，它会产生新的内存地址来存放生成的新列表，但是如果把切片操作放在赋值操作的左边时，那么就相当于引用了原列表的[start:stop]的索引，这种操作是不会生成新的内存空间的，换句话来讲就是直接对原列表进行了`insert操作`.
```python
In [86]: lst                                                                                                                                        
Out[86]: [0, 100, 200, 3, 4, 5, 6, 7, 8, 9]
In [87]: lst[1:3] = []               # 这种操作相当于list.remove                                                                                                               
In [88]: lst                                                                                                                                        
Out[88]: [0, 3, 4, 5, 6, 7, 8, 9]
In [89]: lst[1:3] = [100,200]        # 这种操作相当于在1:3的位置上进行了list.insert                                                                                                              
In [90]: lst                                                                                                                                        
Out[90]: [0, 100, 200, 5, 6, 7, 8, 9]
   
```
__我们知道list在进行insert和remove时的时间复杂度都是O(1)，在进行切片赋值时也容易让人难以理解，所以建议不要使用这种方法。__