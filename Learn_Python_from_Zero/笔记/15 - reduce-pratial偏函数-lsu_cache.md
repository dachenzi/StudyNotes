<font size=5 face='微软雅黑'>__文章目录__</font>

# 介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;functools模块存放着很多工具函数，大部分都是高阶函数，其作用于或返回其他函数的函数。一般来说，对于这个模块，任何可调用的对象都可以被视为函数。
# 1 reduce方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其含义是减少，它接受一个两个参数的函数，初始时从可迭代对象中取两个元素交给函数，下一次会将本次函数返回值和下一个元素传入函数进行计算，直到将可迭代对象减少为一个值，然后返回：`reduce(lambda x, y: x+y, [1, 2, 3, 4, 5]) calculates ((((1+2)+3)+4)+5)`,
```python
reduce(function, sequence[, initial]) -> value
```
- function: 两个参数的函数
- sequence：可迭代对象(不能为空)
- initital：初始值(可以理解为给函数的第一个参数指定默认值)，否则第一次会在可迭代对象中再取一个元素    

下面是一个求1到100累加的栗子
```python
# 普通版
In [24]: sum = 0
In [25]: for i in range(1,101):
    ...:     sum += i
    ...:
In [26]: print(sum)
5050

# 利用reduce版
In [22]: import functools
In [23]: functools.reduce(lambda x,y:x+y,range(101))
Out[23]: 5050
```
# 2 partial方法(偏函数)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在前面学习函数参数的时候，通过设定参数的默认值，可以降低函数调用的难度。而偏函数也可以做到这一点，funtools模块中的partial方法就是将函数的`部分参数固定下来`，相当于为部分的参数添加了一个固定的默认值，形成一个`新的函数并返回`。从partial方法返回的函数，是对原函数的封装，是一个全新的函数。
>注意：这里的偏函数和数学意义上的偏函数不一样。
```python
partial(func, *args, **keywords) - 返回一个新的被partial函数包装过的func，并带有默认值的新函数
```
## 2.1 partial方法基本使用
```python
In [27]: import functools
    ...: import inspect
    ...:
    ...:
    ...: def add(x, y):
    ...:     return x + y
    ...:
    ...:
    ...: new_add = functools.partial(add,1)
    ...: print(new_add)
    ...:
functools.partial(<function add at 0x000002798C757840>, 1)

In [28]:
In [28]: new_add(1,2)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-28-2d6520b7602a> in <module>
----> 1 new_add(1,2)

TypeError: add() takes 2 positional arguments but 3 were given

In [29]: new_add(1)
Out[29]: 2
```
- 由于我们包装了函数add，并指定了一个默认参数1，这个参数会按照位置参数，当作默认值赋给x了
- 所以当我们再次调用new_add,只需要传入y的值就行了。
- 如果再传递两个，那么连同包装前传入的1，一起传给add函数，而add函数只接受两个参数，所以会报异常。
> 获取一个函数的参数列表，可以使用前面学习的inspect模块
```python
In [30]: inspect.signature(new_add)
Out[30]: <Signature (y)>
```
- 查看new_add的签名信息，发现，它的确只需要传入一个y就可以了。  

根据前面我们所学的函数知识，我们知道函数传参的方式有很多种，利用偏函数包装后产生的新函数的传参会有所不同，下面会列举不同传参方式被偏函数包装后的签名信息。
```python
# 最复杂的函数的形参定义方式
def add(x, y, *args, m, n, **kwargs):
    return x + y
```
- add1 = `functools.partial(add,x=1)`：包装后的签名信息(*, x=1, y, m, n, **kwargs)，只接受keyword-only的方式赋值了
- add2 = `functools.partial(add,1,y=20)`：包装后的签名信息(*, y=20, m, n, **kwargs)，1已经被包装给x了其他参数只接受keyword-only的方式赋值了
- add3 = `functools.partial(add,1,2,3,m=10,n=20,a=30,b=40)`：包装后的签名信息(*args, m=10, n=20, **kwargs),1给了x，2给了y， 3给了args，可以直接调用add3，而不用传递任何参数
- add4 = `functools.partial(add,m=10,n=20,a='10')`：包装后的签名信息(x, y, *args, m=10, n=20, **kwargs),a='10'已被kwargs收集，依旧可以使用位置加关键字传递实参。
## 2.2 partial原码分析
上面我们已经了解了partial的基本使用，下面我们来学习一下partial的原码，看它到底是怎么实现的，partial的原码存在于documentation中，下面是原码：
```python
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()    # 偏函数包装时指定的位置位置参数进行拷贝
        newkeywords.update(fkeywords)    # 将包装完后，传递给偏函数的关键字参数更新到keyword字典中去（key相同的被替换）
        return func(*args, *fargs, **newkeywords)  # 把偏函数包装的位置参数优先传递给被包装函数，然后是偏函数的位置参数，然后是关键字参数
    newfunc.func = func    # 新增函数属性，将被包装的函数绑定在了偏函数上，可以直接通过偏函数的func属性来调用原函数
    newfunc.args = args    # 记录包装指定的位置参数
    newfunc.keywords = keywords # 记录包装指定的关键字参数
    return newfunc
```
上面是偏函数的原码注释，如果不是很理解，请看下图
![partial](photo/partial.png)
## 2.3 functools.warps实现分析
现在我们在来看一下functools.warps函数的原码实现，前面我们已经说明了，它是用来拷贝函数签名信息的装饰器，它在内部是使用了偏函数实现的。
```python
def wraps(wrapped,
          assigned = WRAPPER_ASSIGNMENTS,
          updated = WRAPPER_UPDATES):

    return partial(update_wrapper, wrapped=wrapped,
                   assigned=assigned, updated=updated)  
```
使用偏函数包装了update_wrapper函数，并设置了下面参数的默认值：
- wrapped=wrapped：将传入给wraps的函数，使用偏函数，当作update_wrapper的默认值。
- assigned=assigned：要拷贝的信息`'__module__', '__name__', '__qualname__', '__doc__','__annotations__'`
- updated=updated: 这里使用的是`'__dict__'`,用来拷贝函数的属性信息
>__dict__是用来存储对象属性的一个字典，其键为属性名，值为属性的值    

下面来看一下update_wrapper函数，因为真正执行的就是它:
```python
def update_wrapper(wrapper,
                   wrapped,
                   assigned = WRAPPER_ASSIGNMENTS,
                   updated = WRAPPER_UPDATES):
    for attr in assigned:
        try:
            value = getattr(wrapped, attr)
        except AttributeError:
            pass
        else:
            setattr(wrapper, attr, value)
    for attr in updated:
        getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
    wrapper.__wrapped__ = wrapped   # 将被包装的函数，绑定在__wrapped__属性上。
    return wrapper
```
- update_wrapper在外层被wraps包装，实际上只需要传入wrapper即可
- 后面的代码可以理解为是通过反射获取wrapped的属性值，然后update到wrapper中(拷贝属性的过程)
- 最后返回包装好的函数wrapper
> 这里之所以使用偏函数实现，是因为对于拷贝这个过程来说，要拷贝的属性一般是不会改变的，那么针对这些不长改变的东西进行偏函数包装，那么在使用起来会非常方便，我觉得这就是偏函数的精髓吧。  

结合前面参数检查的例子，来加深functools.wraps的实现过程理解。
```python
def check(fn):
    @functools.wraps(fn) 
    def wrapper(*args, **kwargs):
        sig = inspect.signature(fn) 
        params = sig.parameters  
        values = list(params.values()) 
        for i, k in enumerate(args): 
            if values[i].annotation != inspect._empty: 
                if not isinstance(k, values[i].annotation): 
                    raise ('Key Error')
        for k, v in kwargs.items():
            if params[k].annotation != inspect._empty:
                if not isinstance(v, params[k].annotation):
                    raise ('Key Error')
        return fn(*args, **kwargs)

    return wrapper
```
-  `@functools.wraps(fn)` 表示一个有参装饰器，在这里实际上等于：`wrapper = functools.wraps(fn)(wrapper)`
- `functools.wraps(fn)` 的返回值就是偏函数`update_wrapper` , 所以也可以理解为这里实际上：`update_wrapper(wrapper)` 
- `update_wrapper` 在这里将wrapped的属性(也就是fn)，拷贝到了wrapper上，并返回了wrapper。  

经过上述数说明 `@functools.wraps(fn)` 就等价于 `wrapper = update_wrapper(wrapper)`，那么再来看拷贝的过程，就很好理解了。
# 3 lsu_cache方法


