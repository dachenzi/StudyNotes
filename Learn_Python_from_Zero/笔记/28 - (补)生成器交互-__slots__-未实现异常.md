# 1 生成器交互
生成器提供了一个send方法用于动态的和生成器对象进行交互。怎么理解的呢？看下面的例子：
```python
def generator():
    a = 0
    while True:
        position = yield a   # 格式
        if position:
            a = position
        a += 1

g = generator()
print(next(g))
g.send(10)
print(next(g))
print(next(g))
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面的 变量 = yield 返回值，是生成器提供的交互格式，当我们使用生成器对象的send方法时，实参就会被传递给这里的position变量，从而在函数外部来控制函数内部的运行，同时send和next一样可以推动生成器的运行。
```python
import time
import random
class Person:

    def __init__(self, name):
        self.name = name

    def eat(self):
        while True:
            something = yield
            print('{} is eating {}'.format(self.name, something))


daxin = Person('daxin')
g = daxin.eat()
next(g)

count = 0
while True:
    time.sleep(random.randrange(3))
    g.send('包子-{}'.format(count))
    count += 1
```
这个例子看起来很鸡肋，但是想一下，如果count以上的代码在另一个线程中，是不是就实现了不同线程之间的切换？
# 2 __slots__
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字典为了提升查询效率，必须用空间换时间。一般来说一个实例，属性多一点，都存储在字典中便于查询，问题不大，但是如果数百万个实例，那么字典占用的空间就很大了，那么是否可以把类的__dict__属性省了？__slots__就是干这个事情的。
