<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 面向对象介绍](#1-面向对象介绍)
- [2 面向对象](#2-面向对象)
    - [2.1 类class](#21-类class)
    - [2.2 对象instance/object](#22-对象instanceobject)
    - [2.3 Python的哲学思想](#23-python的哲学思想)
- [3 面向对象的要素](#3-面向对象的要素)
- [4 Python的类](#4-python的类)
    - [4.1 类对象及属性](#41-类对象及属性)
    - [4.2 实例化](#42-实例化)
        - [4.2.1 __init__函数](#421-__init__函数)
        - [4.2.2 实例对象(instance)](#422-实例对象instance)
        - [4.2.3 实例变量和类变量](#423-实例变量和类变量)
        - [4.2.4 __dict__和变量查找顺序](#424-__dict__和变量查找顺序)
        - [4.2.5 总结](#425-总结)
    - [4.3 装饰一个类](#43-装饰一个类)
    - [4.4 类方法和静态方法](#44-类方法和静态方法)
        - [4.4.1 普通函数(不用)](#441-普通函数不用)
        - [4.4.2 类方法](#442-类方法)
        - [4.4.3 静态方法(用的很少)](#443-静态方法用的很少)
        - [4.4.4 方法的调用](#444-方法的调用)

<!-- /TOC -->

# 1 面向对象介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;面向对象是一种程序设计思想，它把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。但并不是所有语言都支持面向对象编程的。简单的从语言本身来分的话，主要分为以下三种：(并不是说其他类型的语言不好，只是场景不适合而已，就好比操作系统，基本都是使用C语言编写的，语言没有优劣，只有适不适合)
1. `面向机器`(汇编语言等):抽象成机器指令，机器容易理解。
2. `面向过程`(C语言等):做一件事，排除个步骤，第一步干什么，第二部干什么，如果出现情况A，做什么处理，如果出现过程B，做什么处理。问题规模小，可以步骤话，按部就班的处理。
3. `面向对象OOP`(Java语言，C++语言，Python等):针对情况复杂的情况，比如表示一个大楼的作用，使用函数就比较麻烦了。
> 现在大部分语言都是面向对象的，它适合软件规模比较大的场景使用
        
# 2 面向对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那什么是面向对象？我们可以简单来说它就是一套方法论，是 __`一种认识世界、分析世界的方法论`__ 。将万事万物都抽象为各种对象，在Python中一切皆对象。下面来说一下，主要的名词含义：
## 2.1 类class
抽象的概念，是一类事物的共同特征的集合。
- 数据：有没有耳朵，有没有鼻子
- 操作事物的能力：能不能吃，能不能说等  

用计算机语言来描述，就是属性(数据)和方法(操作事物的能力)的集合。但凡说分类：都是抽象的概念
    
## 2.2 对象instance/object
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对象是类的具象、是一个实体。对于我们每个个体，都是抽象概念人类的不同实体（对象，实例），比如你吃鱼，你是对象，鱼也是对象，吃是动作.  
PS：在python中我们说类的实例通常会用instance或者object来指代。
- 属性：对象状态的抽象，用数据结构来描述
- 操作：对象行为的抽象，用操作名和实现该操作的方法来描述（函数）
> 每个人都是人类的一个单独实例，都有自己的名字、身高、体重等信息，这些信息是个人的属性，但是这些信息不能保存在人类中，因为它是抽象的概念，不能保存具体的信息，而我们每个人，每个个体，都是具体的人，就可以存储这些具体的属性，而且不同的人具有不同的属性。   
## 2.3 Python的哲学思想
在Python中一切皆对象,对象是数据和操作的封装,对象是独立的，但是对象之间可以相互作用(你推我，我打你，相互作用)。
> 目前OOP是最接近人类认知的编程范式

# 3 面向对象的要素
面向对象有三大特点：
1. `封装(Encapsulation)`，将数据和操作组装到一起，对外只暴露一些借口，通过接口访问对象。（比如驾驶员驾驶汽车，不需要了解汽车的构造，只需要知道使用什么部件怎么驾驶皆可以了，踩了油门就能跑，可以不了解其中的机动原理。）
2. `继承(Inheritance)`，多复用（继承来的就不用写了），遵循多继承少修改原则，OCP（Open-closed Principle)，使用继承来改变，来体现个性（修改实例自己，不要去修改父类）
3. `多态(Polymorphism)`，面向对象编程最灵活的地方，动态绑定，比如人吃和猫吃，都是吃，但是不同吃又不太一样。简单来说就是：同一种方法，在不同的子类上，表现方式不同。
# 4 Python的类
对象是特征(变量)与技能(函数)的结合，类是一些列对象共有的特征与技能的结合体.
- 在现实生活中：先有对象，再总结归纳出的类
- 在程序中：一定先定义类，再实例化出对象  

定义一个类：
```python
class ClassName:
    语句块
```
规范：
1. 必须使用`class`关键字
2. 类名必须是用`大驼峰`命名
3. 类定义完成后(被解释器一行一行解释完毕后），就产生一个类对象，绑定到了标识符ClassName上(一切皆对象思想，类对象也是某个对象(type)的实例)
```python
class MyClass:              # class 类名称
    '''MyClass Doc'''       # 类介绍，通过类对象的__doc__属性获取
    x = 100                 # 类属性
    def showme(self):       # 类方法名(函数)
        print('My Class')   # 函数体
    
    showme = lambda self: print('My Class')     # 等同于上面定义的函数
p = MyClass()
print(p.__doc__)
``` 
## 4.1 类对象及属性
- `类对象`：类的定义执行后会产生一个类对象
- `类的属性`：类定义中的变量和类中定义的方法都是类的属性
- `类变量`：x是类MyClass的变量  

所以根据上面例子来说： 
1. MyClass中，x，showme都是类的属性，__doc__也是类的特殊属性。
2. showme被叫做method(方法)，本质上就是普通的函数对象function，它一般要求至少有一个参数。第一个形式参数可以是self(self只是一个惯用标识符，可以换名字，但是不建议，指代当前实例本身)。当使用某个实例调用方法时，这个实例对象会被注入给第一个参数self。所以，self指代的就是实例本身。  
## 4.2 实例化
通过使用类名加括号来构建实例化一个对象。
```python
a = Person() # 实例化
```      
1. 类名后面加括号，就调用类的实例化方法，完成实例化。
2. 实例化就真正创建一个该类的对象（实例），每一次实例化都会返回不同的实例对象
```python
daxin = Person()
xiaoming = Person()
```
注意：上面的两次实例化产生的都是不同的实例，即便是参数相同。Python类实例化后，会自动调用`__init__`方法。这个方法第一个形式参数必须留给self，其他参数随意。
### 4.2.1 __init__函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Python中对象的实例化，其实分为两个部分，即实例化和初始化。主要对应__new__和__init__方法。
1. 实例化(__new__)：得到一个真正的具体的实例
2. 初始化(__init__)：对这个真正的实例进行定制化(不能返回，或return None)
> `__变量名__` 的方法叫做魔术方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__new__方法比较复杂，一般用在元类的定义和修改上，这里先记住：__new__方法实例化一个对象的过程中会调用__init__方法进行初始化，初始化完毕后再由__new__方法将产生的实例对象进行返回，当__new__方法和__init__方法没有被定义时，会`隐式`的向上(父类)查找并调用。
```python
class Person():
    def __init__(self, name, age):  # 形参
        self.name = name   # 这里daxin 赋给 self.name
        self.age = age   # 这里20 赋给 self.age
    def sing(self):
        print('{} is sing'.format(self.name))
daxin = Person('daxin',20)    # 实参
```
注意：
1. `__init__`: 的self是由__new__方法注入进来的，不用手动传递，这个self就是实例化后的对象。
2. self.name和self.age:表示对实例化添加了两个属性name和age，并进行初始化赋值。
3. `__init__`: 只是添加了一些定制化的属性，并不会返回对象。
4. daxin 是由__new__方法返回的具体的实例化对象。
> 注意：`__init__`方法只有在实例化的时候才会被调用。
### 4.2.2 实例对象(instance)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类实例化后一定会获得一个类的实例，就是实例对象。__init__方法的第一个变量self就是实例本身。通过实例化我们可以获得一个实例对象，比如上面的daxin，我们可以通过daxin.sing()来调用sing方法，但是我们并没有传递self参数啊，这是因为类实例化后，得到一个实例对象，实例对象会`绑定`到`方法`上，在使用daxin.sing()进行调用时，会把方法的调用者daxin实例，作为第一个参数self传入。而self.name就是daxin实例的name属性，name保存在daxin实例中，而不是Person类中，所以这里的name被叫做实例变量。
### 4.2.3 实例变量和类变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实例变量是每一个实例自己的变量，是自己独有的。类变量是类的变量，是类的所有实例共享的属性和方法。下面我们从一个例子来看实例变量和类变量
```python
In [2]: class Person: 
   ...:     country = 'China'         # 类变量
   ...:     def __init__(self, name, gender): 
   ...:         self.name = name      # 实例变量
   ...:         self.gender = gender 
   ...:                                                                                                 

In [3]: daxin = Person('daxin','male')                                                                  

In [4]: daxin.country                                                                          
Out[4]: 'China'

In [5]: daxin.name                                                                                      
Out[5]: 'daxin'

In [6]: Person.country                                                                                  
Out[6]: 'China'

In [7]: Person.name                                                                                     
---------------------------------------------------------------------------
AttributeError       
```
观察例子，我们可以发现：
1. 类变量我们可以通过类访问，也可以通过实例访问
2. 实例变量只能通过实例访问(因为类本身是不会知道谁是他的实力)  
> 类变量可以通过实例调用，但实例变量是实例私有的，无法通过类调用。  

PS: 获取类属性的不同的方式
```python
daxin.age
daxin.__class__.age
Person.age
type(daxin).age 
```
有几个特殊的属性先来看一下，便于后续理解
|特殊属性|含义|
|------|-----|
`__name__`|获取类对象的对象名(返回字符串)
`__class__`|获取实例对象的类型(相当于type),type和__class__返回的是同样的东西
`__dict__`|对象的属性的字典
`__qualname__`|类的限定名
```python
In [8]: daxin.__class__                                                                                 
Out[8]: __main__.Person

In [9]: Person.__class__                                                                                
Out[9]: type

In [10]: type(daxin)                                                                                    
Out[10]: __main__.Person

In [11]: type(daxin) is Person                                                                          
Out[11]: True

In [13]: Person.__name__                                                                                 
Out[13]: 'Person'   # 返回字符串
```
疑问：当类变量和实例变量重名时，到底访问的是哪个？
```python
class Person:
    age = 3
    height = 170

    def __init__(self, name, age=18):
        self.name = name
        self.age = age


tom = Person('Tom')
jerry = Person('jetty', 20)

Person.age = 30
print(1, Person.age, tom.age, jerry.age) 
print(2, Person.height, tom.height, jerry.height)
jerry.height = 175
print(3, Person.height, tom.height, jerry.height)
tom.height += 10
print(4, Person.height, tom.height, jerry.height)
Person.height += 15
print(5, Person.height, tom.height, jerry.height)
Person.weight = 70
print(6, Person.weight, tom.weight, jerry.weight)
```
分析：
1. Person返回的是自己的类属性，所以Person.age是30，tom优先访问自己的实例变量，由于指定了默认值，所以tom.age是18，jerry指定了age,所以jerry.age是20.
2. Person返回的是自己的类属性，所以Person.height是170，tom和jerry没有height实例变量，所以访问的都是类变量，值得都是170
3. Person.height是170，jerry.height=175 赋值即定义了一个实例变量height，所以tom.height = 170, jerry.height = 175
4. Person.height是170, tom.height += 10 赋值即定义了一个实例变量height, 所以tom.height = tom.height + 10 = 180, jerry.height = 175
5. Person.height += 15，Person.height = 185,tom和jerry有自己的实例变量不会收Person影响，所以tom.height = 170, jerry.height=175
6. Person.weight = 70，为Person赋值即定义了一个新的weight属性，所以Person.weight = 70,tom和jerry没有weight属性，访问Person的所以值都是70  

通过观察发现，当类变量和实例变量重名时，似乎有一定的查找规则，那么就要说说__dict__了。
### 4.2.4 __dict__和变量查找顺序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__dict__是一个非常重要的特殊属性，它是一个存放着对象的属性信息的字典(对实例来说是字典，对类来说是映射)。
```python
In [14]: Person.__dict__                                                                                
Out[14]: 
mappingproxy({'__module__': '__main__',
              'country': 'China',
              '__init__': <function __main__.Person.__init__(self, name, gender)>,
              '__dict__': <attribute '__dict__' of 'Person' objects>,
              '__weakref__': <attribute '__weakref__' of 'Person' objects>,
              '__doc__': None})

In [15]: daxin.__dict__                                                                                 
Out[15]: {'name': 'daxin', 'gender': 'male'}

In [19]: daxin.__dict__['name']         # 因为__dict__是一个字典，我们可以直接通过key来访问                                                              
Out[19]: 'daxin'

```
通过观察发现：
1. Person.__dict__：包含所有类内定义的属性及方法，但不包含实例对象的属性信息
2. daxin.__dict__ : 只记录实例自己的属性信息。
> 方法是记录在类的`__dict__`中的  

实例属性查找顺序: __`指的是实例使用.点号来访问属性，会先找自己的__dict__，如果没有，然后通过属性__class__找到自己的类，然后再在类的__dict__中查找.直接使用实例.__dict__[变量名]，不会在类中查找。`__
        
### 4.2.5 总结
1. 类变量是属于类的变量，这个类的所以实例可以共享这个变量
2. 对象(实例或者类)，可以动态的给自己增加一个属性(赋值即定义)
3. 实例.__dict__[变量名] 和 实例.变量名 都可以访问到实例自己的属性
4. 实例的同名变量会隐藏掉类变量，或者说是覆盖了这个类变量。但是注意类变量还在那里，并没有真正被覆盖。
5. 实例属性查找顺序：指的是实例使用.点号来访问属性，会先找自己的__dict__，如果没有，然后通过属性__class__找到自己的类，然后再在类的__dict__中查找。直接使用实例.__dict__[变量名]，不会在类中查找。
6. 一般来说类变量可以全部用大写来表示(当常量用)
    
## 4.3 装饰一个类
为一个类通过装饰，增加一些类属性。例如能否给一个类增加一个NAME类属性并提供属性值
```python
def dec(name):
    def warpper(cls):
        cls.NAME = name
        return cls
    return warpper


@dec('tom')  # Person = dec('tom')(Person)
class Person:
    pass

a = Person()
print(a.NAME)
```
> 类也可以作为一个装饰器，后面会说

可以给类加装饰器，那么可以给类中的方法加装饰器吗？答案当然是可以的。
## 4.4 类方法和静态方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在类中中定义的方法大多都需要传入一个self参数，用于指定一个实例化对象，用于通过对象来调用这个方法，但是也有例外的情况
### 4.4.1 普通函数(不用)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;定义在类内部的普通函数,只能通过类来进行调用
```python
In [23]: class Person: 
    ...:     def normal_function(): 
    ...:         print('normal_function') 
    ...:                                                                                                

In [24]: Person.normal_function()                                                                       
normal_function

In [25]: test = Person()                                                                                

In [27]: test.normal_function()                                                                         
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-27-9011f04ffcaa> in <module>
----> 1 test.normal_function()

TypeError: normal_function() takes 0 positional arguments but 1 was given

In [28]:   
```
注意：
1. normal_function 定义在类里，只是受Person类管辖的一个普通函数而已，在Person中，认为normal_function仅仅是Person的一个属性而已.
2. 当通过实例访问时，由于实例会默认把当前实例当作self传入函数，而函数并没有形参接受所以会报错。
> 虽然可以通过类调用，但是没人这样用，也就是说是`禁止`这么写的
### 4.4.2 类方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类方法是给类用的，在使用时会将类本身当做参数传给类方法的第一个参数，使用`@classmethod`装饰器来把类中的某个函数定义成类方法。
```python
In [31]: class Person: 
    ...:     @classmethod 
    ...:     def class_method(cls): 
    ...:         print('class={0.__name__}({0})'.format(cls)) 
    ...:         cls.HEIGHT=170 
    ...:                                                                                                

In [32]: Person.class_method()                                                                          
class=Person(<class '__main__.Person'>)

In [35]: daxin = Person()                                                                               

In [36]: daxin.class_method()                                                                           
class=Person(<class '__main__.Person'>)

In [37]:
```
注意：
1. 在类定义中，使用@classmethod装饰器的方法
2. 必须至少有一个参数，且第一个参数留给了cls，cls指代调用者即类对象自身
3. cls这个标识符可以是任意合法名称，但是为了易读，不建议修改
4. 通过cls可以直接操作类的属性
5. 通过类的实例依旧可以调用类方法，那是因为当通过实例调用时，@classmethod发现调用者是实例时，会把当前实例的__class__对象，传入给类方法的cls。
>类方法类似于C++、Java中的静态方法。
### 4.4.3 静态方法(用的很少) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;静态方法是一种普通函数，位于类定义的命名空间中，不会对任何实例类型进行操作,它有如下特点
1. 在类定义中，使用`@classmethod`装饰器修饰的方法，在调用时不会隐式的传入参数
2. 静态方法，只表明这个方法属于这个名称空间，函数归在一起，方便组织管理。
3. 可以理解为一个函数封装，可以通过类调用、也可以通过实例调用，只是强制的放在类中而已。   
```python
class Person:
    @staticmethod
    def static_method():
        print('static_method')


daxin = Person()
Person.static_method()
daxin.static_method()
```
> 和普通函数的区别是，静态方法可以使用实例调用，普通方法不可以。  
### 4.4.4 方法的调用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类几乎可以调用所有方法，普通函数的调用一般不可能出现，因为不允许这么定义。实例也几乎可以调用所有内部定义的方法，但是调用普通方法时会报错，原因是第一参数必须是类的实例self。
```python
class Test:
    age = 10
    def normal_funtion():
        print('normal_function')

    @classmethod
    def class_method(cls):
        print(cls.__name__)

    @staticmethod
    def static_method():
        # Test.age
        print('static_method')

daxin = Test()
daxin.normal_funtion()   # 无法调用，因为通过实例调用时，实例会被当作第一个参数传给normal_funcion。
daxin.class_method()     # 可以调用，当@classmethod检测到是通过实例调用时，会把当前实例的__class__，当作cls传递，可以访问类属性，但无法访问实例属性
daxin.static_method()    # 可以调用，当@staticmethod检测到是通过实例调用时，会在当前实例的类中调用静态方法，无法访问类或实例的属性，但是可以访问全局变量(比如全局变量Test类)

Test.normal_funtion()    # 可以调用，类调用时不传递参数，刚好normal_function也不需要参数，无法访问类或实例的属性
Test.class_method()      # 可以调用，类方法，通过类调用时，类会被当作cls传递给函数，可以访问类属性,无法访问实例属性
Test.static_method()     # 可以调用，静态方法，等于定义在类内的函数，无法访问类或实例的属性，但是可以访问全局变量(比如全局变量Test类)
```
        
        
        
        实例调用方法和类调用方法，在使用self时的不同之处。

            
            class Person:
                def normal_fuction():
                    print('normal_function') 
                def normal_method(self):
                    print('normal_method')
            
            调用：Person.normal_fuction()
            体会：Person.normal_method(Person()) 和 Person.normal_method(123) 的区别
                p1.normal_function() 绑定方法（会自动将对象注入到self中)
                Person.normal_function() 普通的函数
    
            > 函数默认参数是引用类型时，遵循函数那套关系

            
            
            
            
2019/02/16 
    
    
    
    
    
    
    
    
    
        
        
        
        
        
        
        
        
        