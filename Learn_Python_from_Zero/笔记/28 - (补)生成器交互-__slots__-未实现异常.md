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
```python
class A:
    def __init__(self):
        self.name = 'daxin'
        self.age  = 20
        self.country = 'China'
        self.language = 'Chinese'
        ... ...
a = A()
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实例化对象时，它的属性信息都会存放在实例自己的__dict__字典中去，由于没办法固定实例的属性个数，所以这个字典就会很大。比如__dict__申请了300间客房，而只有4个客人住，并且每个实例都是这样。当使用了__slots__时
```python
class A:

    __slots__ = ['name','age']

    def __init__(self):
        self.name = 'daxin'
        self.age  = 20

    def say(self):
        pass

    def hello(self):
        pass


a = A()
a.name = 'tom'
a.sex = 'Man'  # 无法设置，因为__slots__没有允许
print(a.__class__.__dict__)  # {'__module__': '__main__', '__slots__': ['name', 'age'], '__init__': <function A.__init__ at 0x0000022422379950>, 'say': <function A.say at 0x00000224223799D8>, 'hello': <function A.hello at 0x0000022422379A60>, 'age': <member 'age' of 'A' objects>, 'name': <member 'name' of 'A' objects>, '__doc__': None}
```
当类使用了__slots__属性时：
1. 实例的属性被__slots__约束，包括在__init__函数中self设置的情况
2. 实例不会生成__dict__属性。
3. 父类的__slots__对应着子类属性名称
4. 另外还存在属性名称对应的<member object>对象，可以理解为类描述器东西
5. __slots__只约束实例的属性，并不约束实例的方法（实例没有方法，实例是被绑定在类中定义的方法上的）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;换句话说：__slots__告诉解释器，实例的属性都叫什么，一般来说，既然要节约内存，最好还是使用元组比较好，一旦类提供了__slots__，就阻止实例产生__dict__来保存实例的属性。__slots__不影响子类，不会被继承
https://www.cnblogs.com/rainfd/p/slots.html
```python

```