<font size=5 face='微软雅黑'>__文章目录__</font>

<!-- TOC -->

- [1 Javascript函数](#1-javascript函数)
    - [1.1 基本语法](#11-基本语法)
    - [1.2 函数表达式](#12-函数表达式)
    - [1.3 函数的调用](#13-函数的调用)
    - [1.4 高阶函数](#14-高阶函数)

<!-- /TOC -->
# 1 Javascript函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;javascript的函数和其他任何一门语言的函数是相同的，都是用来组合代码块完成某个功能和任务的。不同的是：执行函数的代码可以放在函数之前，也可以放在函数之后，因为js是先编译完毕后再执行代码中的语句。(不建议这么做)

## 1.1 基本语法
函数分为带参数的和不带参数的。
```js
<script>

// 不带参数的函数
function functionname() {
    执行代码
}
 
// 带参数的函数
function myFunction(var1,var2) {   // 同样是形参
    执行代码
}
 
</script>
```

## 1.2 函数表达式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用表达式来定义函数，表达式中的函数名可以省略，如果这个函数名不省略，也只能用在此函数内部。

```js
// 匿名函数
const add = function(x, y){
 return x + y;
};
console.log(add(4, 6));

// 有名字的函数表达式
const sub = function fn(x, y){
 return x - y;
};
console.log(sub(5, 3));
//console.log(fn(3, 2)); // fn只能用在函数内部

// 有名字的函数表达式
const sum = function _sum(n) {
 if (n===1) return n;
 return n + _sum(--n) // _sum只能内部使用
}
console.log(sum(4));
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;函数和匿名函数，本质上都是一样的，都是函数对象，只不过函数有自己的标识符——函数名，匿名函数需要借助其它的标识符而已。区别在于，`函数会声明提升，函数表达式不会`。

## 1.3 函数的调用
和Python的相同，只需要写函数名加上括号即可，如果函数有参数，那么在括号内传入即可。
```js
<script>
 
function myFunction(name,job){
    alert("Welcome " + name + ", the " + job);
}
 
myFunction('daxin','Linux')    // 调用时传递参数
</script>
```
PS：函数对象也有lenght属性，它返回的是函数期望的参数个数

## 1.4 高阶函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当一个函数接受一个函数作为参数，或者这个函数返回的是一个函数，那么就称这个函数为高阶函数。  

计数器：
```js
function counter() {
    let c = 0
    return function () {
        return ++c
    }
}

c = counter()
console.log(c())  // 1
console.log(c())  // 2
console.log(c())  // 3
```
完成一个map函数：可以对某一个数组的元素进行某种处理
```js
function map(arry, func) {
    new_arry = new Array
    for (let x in arry) {
        new_arry[x] = func(arry[x])
    }
    return new_arry
}

console.log(map([1,2,3,4,5], x => ++x))
```
生成器版本：
```js
function map(arry, func) {
    new_arry = new Array 
    return (function * () {
        for (let x in arry) {
            new_arry[x] = func(arry[x]);
            yield new_arry[x];
        }
    })();
    
}

c = map([1,2,3,4,5], x => ++x)
console.log(c.next())
console.log(c.next())
console.log(c.next())
```
> function * () 表示一个匿名生成器，生成器函数需要使用 * 来分割。












内置对象arguments
在javascript的函数中内置了一个arguments对象，用来存放传入的函数内部的参数。就算没有指定函数接受参数，如果我们调用时加上了参数，那么也还是会存储到arguments中去，但对于python来说，就会报错了。


<script>
function add(){
    for (var item in arguments) {
        console.log(arguments[item])
    }
}
</script>
 
>> add(1,2,3,4)
>> 1
>> 2
>> 3
>> 4
PS：arguments的length属性，表示传入参数的个数。

函数的其他类型
javascript中的函数其实分为：普通函数，匿名函数，立即执行函数(IIFE,或者叫做自执行函数）

普通函数就如上面所说，那什么是匿名函数？

匿名函数
匿名函数就是没有名称，在调用时直接定义，比如定时器的例子。


setInterval( function () {
    var i = 0;
    console.log(i);
},5000)
PS：定时器接受的第一个参数是函数名(),这里可以直接定义函数，那么定时器每隔5秒钟就会执行以下这段函数

当然匿名函数也可以被复制给一个变量称为有名函数，这样创建函数的方式叫做 函数的表达式形式。

var funcname = function () {
    alert(123)
};
funcname()
立即执行函数
由于函数是一个封闭的作用域范围，并且可以嵌套函数，所以可以使用这种匿名自执行函数来实现封装自己的所有函数和变量，从而避免来自多个js文件的多个函数相互冲突，并且，由于它们位于同一个函数中，所以可以互相引用。 
由于外部无法引用函数内部的变量，因此在执行完后很快就会被释放，关键是这种机制不会污染全局对象。这同时也相当于定义了一个命名空间，来自不同的js文件的功能位于自己的命名空间内。

基本格式
基本格式如下：还有基于!,+,void等，这里不在赘述


( function (接收的参数) {
    代码
})(传递的参数)　
PS：那么为什么不直接在匿名函数后面加括号来直接执行呢，就像python一样？在javascript中会提示语法错误，因为当解释器解析到function时，会把这段代码当作函数的定义，所以就算在函数的定义后面直接加括号执行，也会报错。因为解释器不识别。