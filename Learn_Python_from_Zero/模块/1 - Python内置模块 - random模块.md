# 1 随机数random
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之所以在这里介绍random模块是因为，我们可能需要在列表中随机挑选一个或几个重复或者不重复的数据，这时，就可以使用python得知的`radnom模块`。我们常用的函数如下：
```python
random.randint(a,b): 返回[a,b]之间的整数,包含a和b本身。

random.choice(seq): 从非空序列的元素中随机挑选一个元素。

random.randrange([start,] stop, [,step]): 从指定范围内获取一个随机数。和range相同属于前包后不包区间, start(起始) 默认为0，step（步长) 默认为1 

random.shuffle(list) --> None : 直接对原列表进行洗牌(随机打乱)

random.sample(population, k): 从population(样本空间)中随机选取出k个不同的元素，返回一个新的列表
```
下面是例子：
```python
In [24]: import random                                                                                                

In [25]: random.randint(1,10)                                                                                         
Out[25]: 4

In [26]: random.choice(range(10))                                                                                     
Out[26]: 5

In [27]: random.randrange(1)                                                                                          
Out[27]: 0

In [28]: random.randrange(0,5,2)                                                                                      
Out[28]: 2
                                                                       

In [31]: lst = list(range(10))                                                                                        

In [32]: random.shuffle(lst)                                                                                          

In [33]: lst                                                                                                          
Out[33]: [8, 1, 6, 4, 3, 9, 7, 0, 5, 2]

In [34]: random.sample(lst,4)                                                                                         
Out[34]: [8, 4, 1, 0]
 
```