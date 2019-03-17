<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 JavaScript概述](#1-javascript概述)
    - [1.1 Java和JavaScript的区别](#11-java和javascript的区别)
    - [1.2 什么是JavaScript](#12-什么是javascript)
    - [1.3 JavaScript的组成](#13-javascript的组成)
- [2 JavaScript基础](#2-javascript基础)
    - [2.1 JavaScript调用方式](#21-javascript调用方式)
    - [2.2 JavaScript的变量标识符](#22-javascript的变量标识符)
        - [2.2.1 变量的命名](#221-变量的命名)
        - [2.2.2 标识符](#222-标识符)
    - [2.3 JS的数据类型](#23-js的数据类型)
        - [2.3.1 数字(Number)类型](#231-数字number类型)
        - [2.3.2 字符串(String)类型](#232-字符串string类型)
        - [2.3.3 布尔(Boolean)类型](#233-布尔boolean类型)
        - [2.3.4 数组(Array)类型](#234-数组array类型)
        - [2.3.5 对象(Object)](#235-对象object)
        - [2.3.6 空(Null)类型和未定义(Undefined)类型](#236-空null类型和未定义undefined类型)
        - [2.3.7 声明变量类型](#237-声明变量类型)
    - [2.4 运算符](#24-运算符)
        - [2.4.1 == 和 === 的区别](#241--和--的区别)
        - [2.4.2 variable++和++variable的区别](#242-variable和variable的区别)
        - [2.4.3 三元运算符](#243-三元运算符)
        - [2.4.4 逗号操作符](#244-逗号操作符)
        - [2.4.5 其他](#245-其他)
    - [2.5 流程控制](#25-流程控制)
        - [2.5.1 顺序结构](#251-顺序结构)
        - [2.5.2 分支结构](#252-分支结构)
            - [2.5.2.1 if - else结构](#2521-if---else结构)
            - [2.5.2.2 if-else if - else 结构](#2522-if-else-if---else-结构)
            - [2.5.2.3 switch...case结构](#2523-switchcase结构)
        - [2.5.3 循环结构](#253-循环结构)
            - [2.5.3.1 for条件循环](#2531-for条件循环)
            - [2.5.3.2 for-in循环](#2532-for-in循环)
            - [2.5.3.3 for-of循环](#2533-for-of循环)
            - [2.5.3.3 while..do循环](#2533-whiledo循环)
            - [2.5.3.3 do..while循环](#2533-dowhile循环)
        - [2.5.4 小练习](#254-小练习)

<!-- /TOC -->
# 1 JavaScript概述
JavaScript 是互联网上最流行的脚本语言，这门语言可用于 HTML 和 web，更可广泛用于服务器、PC、笔记本电脑、平板电脑和智能手机等设备。

## 1.1 Java和JavaScript的区别
JavaScript 与 Java 是两种完全不同的语言，无论在概念上还是设计上。
- Java（由 Sun 发明）是更复杂的编程语言。
- JavaScript 由 Brendan Eich 发明。它于 1995 年出现在 Netscape 中（该浏览器已停止更新），并于 1997 年被 ECMA（一个标准协会）采纳。ECMA-262 是 JavaScript 标准的官方名称。

## 1.2 什么是JavaScript
JavaScript 是脚本语言
- JavaScript 是一种轻量级的编程语言。
- JavaScript 是可插入 HTML 页面的编程代码。
- JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行。
- JavaScript 可以控制HTML 输出流、对事件的反应、改变 HTML 内容、改变 HTML 图像、改变 HTML 样式、验证输入等等。

## 1.3 JavaScript的组成
我们说的JavaScript通常由三部分组成：
- ECMAScript：JavaScript标准化部分，主要定义了基础的数据类型，语法，关键字等等。
- DOM(文档对象模型)：整合JS，CSS，HTML。
- BOM(浏览器对象模型)：整合JS 和 浏览器。

# 2 JavaScript基础
JavaScript 是一个程序语言。遵循ECMAScript标准，语法规则定义了语言结构。

## 2.1 JavaScript调用方式
HTML中引入JS的方式主要有两种:直接嵌入式编写、通过导入JS文件
- 直接嵌入式编写：必须位于 <script> 与 </script> 标签之间。脚本可被放置在 HTML 页面的 <body> 和 <head> 部分中。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>
        alert('hello world')    // 用于弹出告警框，便于调试
    </script>
</head>
<body>
<script>
    alert('hello world')
</script>
</body>
</html>
```
注意：通常的做法是把JS函数放入 <head> 部分中，或者放在页面底部。这样就可以把它们安置到同一处位置，不会干扰页面的内容。  --> 建议放在body的最后面，这样不会阻碍网页加载

- 导入式：在<script></script>标签内导入

```html
<script src='index.js'></script>
```
注意：外部脚本不能中不能包含 <script> 标签。

## 2.2 JavaScript的变量标识符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;变量是用于存储信息的"容器"。在代数中，我们使用字母（比如 x）来保存值（比如 5）。通过上面的表达式 z=x+y，我们能够计算出 z 的值为 11。在 JavaScript 中，这些字母被称为变量。声明一个遍变量的方式主要有以下方法：
- 使用var关键字： `var x = 1`; 根据值的类型，x是number类型。（`局部变量`）
- 不使用var关键字： `y = 1`; 根据值的类型，x是number类型。（`全局变量`） 严格模式会报错，不建议用
- 只定义不赋值：`var x`; 那么此时该变量的类型为`undefine`。
- 使用let关键字： `let x = 1`; 和var关键字创建的效果大致相同，属于`ES6的新语法`，以大括号为作用域
- 使用const关键字：`const x = 1`; 声明一个常量x，不允许更改

> `var会把变量提升`到当前全局或函数的作用域的头部先行定义。

```javascript
var x = 1;
console.log(typeof x)  
y = 2;
console.log(typeof y)、
#typeof 表示查看对象的类型
#console.log：通了浏览器的console输出，便于调试
```
Tips：
- 在一行内定义多个变量需要用逗号隔开：var a = 1, b = 2;
- 重新声明并不会修改变量的值： var a = 1;  var a      a的值依旧为1

### 2.2.1 变量的命名
变量可以使用短名称（比如 x 和 y），也可以使用描述性更好的名称（比如 age, sum, totalvolume）。
- 变量必须以字母开头
- 变量也能以 $ 和 _ 符号开头（不过我们不推荐这么做）
- 变量名称对大小写敏感（y 和 Y 是不同的变量）

主要存在以下写法：
- Camel 标记法:首字母是小写的，接下来的字母都以大写字符开头。例如：
```js
var myTestValue = 0, mySecondValue = "hi";
```
- Pascal 标记法:首字母是大写的，接下来的字母都以大写字符开头。例如：
```js
Var MyTestValue = 0, MySecondValue = "hi";
```
- 匈牙利类型标记法:在以 Pascal 标记法命名的变量前附加一个小写字母（或小写字母序列），说明该变量的类型。例如，i 表示整数，s 表示字符串，如下所示“
```js
Var iMyTestValue = 0, sMySecondValue = "hi";
```

### 2.2.2 标识符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由不以数字开头的字母、数字、下划线(_)、美元符号($)组成,常用于表示函数、变量等的名称,例如：_abc,$abc,abc,abc123是标识符，而1abc不是,JavaScript语言中代表特定含义的词称为保留字，不允许程序再定义为标识符


## 2.3 JS的数据类型
数据类型主要有一下几种：字符串（String）、数字(Number)、布尔(Boolean)、数组(Array)、对象(Object)、空（Null）、未定义（Undefined）。

|名称|说明|
----|----|
number|数值型，包括整形和浮点型
boolean|布尔型, true和false
string|字符串型
null|只有一个值null
undefined|变量声明未赋值的；对象未定义的属性
symbol|ES6新引入类型
object|是以上基本类型的复合类型，是容器

javascript的隐式转换
- 遇到字符串，加号就是拼接字符串，所有非字符和字符串相加，都是是隐式的转换为字符串。
- 如果没有字符串，加号把其他所有类型都当数字处理，非数字类型隐式转换为数字。undefined特殊，因为它没有定义值，所以转换失败会得到一个特殊值NaN
- 如果运算符是逻辑运算符，短路符，返回就是短路时的类型。没有隐式转换

> 除非你十分明确，否则不要依赖隐式转换

### 2.3.1 数字(Number)类型
JavaScript 只有一种数字类型(双精度浮点型)。数字可以带小数点，也可以不带：
```js
var a = 3.14;
var b = 10;
```
在JS中所有的数字类型均为number，并且都用64位浮点格式来存储。
- NaN类型: js来做数据转换的时候，比如当字符串转换成数字失效的时候，会返回一个 NaN 的值。类型也是属于number的
- 在变量前添加 + ，则表示把该变量转换为正数

```js
var a = 'a456';
b = Number(a)  // 无法转换，结果为NaN
console.log(b)
```
数字类型还有三种符号值：+Infinity（正无穷）、-Infinity（负无穷）和 NaN (not-a-number非数字)。

常用方法|含义
----|-----|
Number.parseFloat()|把字符串参数解析成浮点数，和全局方法 parseFloat() 作用一致
Number.parseInt()|把字符串解析成特定基数对应的整型数字，和全局方法 parseInt() 作用一致
Number.isFinite()|判断传递的值是否为有限数字
Number.isInteger()|判断传递的值是否为整数
Number.isNaN()|判断传递的值是否为NaN

内置数学对象Math：
- Math提供了绝对值、对数指数运算、三角函数运算、最大值、最小值、随机数、开方等运算函数，提供了PI值。
```js
console.log(Math.PI)
console.log(Math.abs(-1))
console.log(Math.log2(16))
console.log(Math.sqrt(2))
console.log(Math.random()) // (0, 1)
```

### 2.3.2 字符串(String)类型
字符串是存储字符（比如 "Lee daxin"）的变量。可以是引号中的任意文本。
> 由于没有字符类型，所以在字符串中表达常用的特殊字符，但是在使用特殊字符时必须加上反斜线\；常用的转义字符 \n:换行 \':单引号 \":双引号 \\:右划线
```js
var name = 'dachenzi';
var job = 'Linux';
```
ES6还提供了反引号用来定义一个字符串，可以支持多行，还支持插值
```js
let name = 'daxin'
let age = 20
info = `${name},${age}`
console.log(info)  // daxin,20
```

常用方法|含义
----|-----|
obj.length|长度
obj.trim()|移除空白
obj.trimLeft()|左边移除空白（非ES6官方语法）
obj.trimRight)|右边移除空白（非ES6官方语法）
obj.charAt(n)|返回字符串中的第n个字符
obj.concat(value, ...)|拼接
obj.indexOf(substring,start)|子序列位置
obj.lastIndexOf(substring,start)|子序列位置
obj.substring(from, to)|根据索引获取子序列
obj.slice(start, end)|切片
obj.toLowerCase()|大写
obj.toUpperCase()|小写
obj.split(delimiter, limit)|分割
obj.search(regexp)|从头开始匹配，返回匹配成功的第一个位置(g无效)
obj.match(regexp)|全局搜索，如果正则中有g表示找到全部，否则只找到第一个。
obj.replace(regexp, replacement)|替换，正则中有g则替换所有，否则只替换第一个匹配项，<br>$数字：匹配的第n个组内容；<br>$&：当前匹配的内容；<br>$`：位于匹配子串左侧的文本；<br>$'：位于匹配子串右侧的文本<br>$$：直接量$符号

### 2.3.3 布尔(Boolean)类型
布尔（逻辑）只能有两个值：true 或 false。也代表1和0，实际运算中true=1,false=0 。这点和shell是不同的。主要用于流程控制
```js
if (1) {
        alert('True')
 }
 else {
         alert("False")
 }
```

### 2.3.4 数组(Array)类型
类似于其他语言的列表，但是不是一个东西。
```js
//方式1：
var arr = new Array(1,2,3,4,5); 
//方式2:
var arr = [1,2,3,4,5]; 
//方式3:
var arr = new Array();
arr[0] = 1;
arr[1] = 2;
arr[2] = 3;
```
扩展：列表的索引为数字，而数组的索引可以为任意.
```js
<script>
     var arr = Array();
     arr[0]= 'w';
     arr['index']= 6;
     arr['input'] = 'hello world';
     console.log(arr)
</script>
```

### 2.3.5 对象(Object)
对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：
```js
//方式1：
var person={firstname:"Bill", lastname:"Gates", id:5566};
 
//方式2：
var person={
    firstname : "Bill",
    lastname  : "Gates",
    id        :  5566
};
//声明中的空格和折行无关紧要
```
Persion对象有三个属性：firstname、lastname 以及 id。访问属性的两个方式：
- persion.firstname
- persion['firstname']

### 2.3.6 空(Null)类型和未定义(Undefined)类型
Undefined 这个值表示变量不含有值,当声明的变量未初始化时，该变量的默认值是 undefined。当函数无明确返回值时，返回的也是值 "undefined。另一种只有一个值的类型是 Null，它只有一个专用值 null，即它的字面量。值 undefined 实际上是从值 null 派生来的,因此 ECMAScript 把它们定义为相等的。可以通过将变量的值设置为 null 来清空变量的值。尽管这两个值相等，但它们的含义不同。undefined 是声明了变量但未对其初始化时赋予该变量的值，null 则用于表示尚未存在的对象。如果函数或方法要返回的是对象，那么找不到该对象时，返回的通常是 null。

### 2.3.7 声明变量类型
如果我们只定义变量没有进行赋值，那么默认就是undefined类型的，如果想要声明某个类型的变量：
```js
var name=   new String;
var x=      new Number;
var y=      new Boolean;
var arr=    new Array;
var person= new Object;
```

## 2.4 运算符
常用的运算符如下:

类型|包含
---|----|
算术运算符| +  -   *   /   %    ++   --|
比较运算符| >   >=   <    <=    !=    ==    ===   !==|
逻辑运算符| &&   ||   ！|
赋值运算符| =  +=   -=  *=   /=|
字符串运算符| +  连接，两边操作数有一个或两个是字符串就做连接运算

### 2.4.1 == 和 === 的区别
```js
var a = '123';
var b = 123;
console.log(a == b);
 
// JS在做比较运算的时候，会把不同的数据类型进行转换，所以这里a 是等于 b的。
// 如果不希望JS完成类型转换，那么可以使用===。
```
python强类型语言，js属于弱类型语言。
- 强类型定义语言：一种总是强制类型定义的语言。Java和Python是强制类型定义的。如果你有一个整数，如果不显示地进行转换，你不能将其视为一个字符串。
- 弱类型定义语言：一种类型可以被忽略的语言，与强类型定义相反。JavaScript是弱类型定义的。在JavaScript中，可以将字符串'12'和整数3进行连接得到字符串'123'，然后可以把它看成整数123，而不需要显示转换。

### 2.4.2 variable++和++variable的区别
假如x=2，那么x++表达式执行后的值为3，x--表达式执行后的值为1；i++相当于i=i+1，i--相当于i=i-1；递增和递减运算符可以放在变量前也可以放在变量后：--i
```js
//区别主要体现在赋值的时候
var a = 1;
var b = a++;
//这里b的值为1，因为是先赋值后a自加
 
var b = ++a;
//这里b的值为3，因为是先a自加后赋值
```

### 2.4.3 三元运算符
它的一般格式为：`条件表达式?真值:假值`，等价于简单的if...else结构
```js
if (条件表达式) {
 真值
}
else {
 假值
}
```
比如：
```js
console.log(('3' > 30)?'真':'假')
```

### 2.4.4 逗号操作符
JS允许多个表达式写在一起，最终将返回最后一个逗号表达式的值
```js
let a = 4+5, b = true, c=a > 20 ?'t':'f'
console.log(a) //9
console.log(c) //f
function test() {
 return 3, a + b, c = a++  // 没有有var/let创建的变量为全局变量，直接修改了全局变量
}
console.log(test()) // 最终返回的是 c = a++ 的值，为9
console.log(c) // c为9
```

### 2.4.5 其他
名称|说明
instanceof|判断是否属于指定类型，但是对象必须是new关键字声明创建的。
typeof|返回类型字符串
delete|delete操作符, 删除一个对象(an object)或一个对象的属性(an object's property)或者一个数组中某一个键值(an element at a specified index in an array)。
in|如果指定的属性在对象内，则返回true


## 2.5 流程控制
JS中主要有三种流程控制结构：
- 顺序结构（从上到下顺序执行）
- 分支结构（比如if，else；switch case）
- 循环结构（比如for，while）

### 2.5.1 顺序结构
即从上到下按照顺序执行：
```js
<script>
console.log('hello')
console.log('world')
</script>
```

### 2.5.2 分支结构
和python不同的是，条件表达式的and/or是用  && / || 表示的

#### 2.5.2.1 if - else结构
```
if (条件表达式) {
    语句1；
    ... ... 
} else {
    语句2;
    ... ...
}
```
注意：只要表达式成立，或者返回值是bool都可，即if(1)也是可以的，因为1的bool类型为true；
```js
<script>

if (1) {
     console.log('true')
 } else {
     console.log('False')
 }
 
</scripts>
```
#### 2.5.2.2 if-else if - else 结构
```js
if (条件表达式1) {
    语句1；
    .... ....
} else if (条件表达式2) {
    语句2；
    ... ...
} else {
    语句3;
    ... ...
}
<script>
```
下面看一个例子
```js
var a = 10;
if (a > 10) {
  console.log('bigger than 10')
 } else if (a > 5) {
    console.log('bingger than 5')
} else {
     console.log('smaller than 5')
 }
</scripts>
```
条件的False等效
- false
- undefined
- null
- 0
- NaN
- 空字符串  

其它值都将被视为True


#### 2.5.2.3 switch...case结构
```js
switch  (条件表达式) {
      case 值1:语句1;break;
      case 值2:语句2;break;
      case 值3:语句3;break;
      case 值4:语句4;break;
      default:语句5;
}
```
下面来看一个例子
```js
 <script>
  var a  =  5 ;
  switch (a) {
      case 0:y = '星期一';break;
      case 1:y = '星期二';break;
     case 2:y = '星期三';break;
      case 3:y = '星期四';break;
      case 4:y = '星期五';break;
      case 5:y = '星期六';break;
     case 6:y = '星期七';break;
 }
 console.log(y) </script>
```
switch比else if结构更加简洁清晰，使程序可读性更强,效率更高。但是一定要注意恰当的时候break语句，否则会继续顺序向下执行。

### 2.5.3 循环结构

#### 2.5.3.1 for条件循环
```js
for (初始化表达式;条件表达式;自增或自减) {
    执行语句1;
    执行语句2;
    ... ...
}
```
下面来看一个例子：
```js
<script>
 
 for (var i = 0 ; i < 10; i++) {
         console.log(i);
 }
 
 </script>
```
死循环的例子：
```js
for(;;){   // 表达式，条件，等全为空
    console.log('abc')
}
```

#### 2.5.3.2 for-in循环
```js
for (变量 in 属组或对象) {
    执行语句1;
    执行语句2;
}
```
下面来看一个例子
```js
 <script>
     var arr = [1,2,3,4,5]
          for (var i in arr) {
         console.log(i)
     }
 </script>
```
PS：for循环，循环的是对象的索引

#### 2.5.3.3 for-of循环
for ... of 直接取值，只能用于数组，不能作用于对象。
```js
let arry = ['a',1,'c',6]
for ( let x of arry) {
    console.log(x)
}
```
#### 2.5.3.3 while..do循环
先判断条件，满足时，再执行
```js
<script>
    while (条件表达式) {
        执行语句1；
        执行语句2；
    }
</script>
```
下面来看一个例子:
```js
 <script>
     var sum = 0;
     var i = 1;
          while ( i < 101) {
         sum += i
         i++
     }
     console.log(sum)
 </script>
```

#### 2.5.3.3 do..while循环
它的一般格式为：
```js
do
 statement
while (condition);
```
先执行语句，然后在进行条件判断。

### 2.5.4 小练习
打九九乘法表
```js
for (let i = 1; i < 10; i++) {
    str = ''
    for (let j = 1; j<=i; j++) {
        str += `${j}*${i}=${i*j>=10?i*j +' ':i*j+'  '} `
    }
    console.log(str)
}


for (let i = 1;i < 10;i++) {
    str = ''
    for (let j = 1;j<=i;j++){
        str += `${j} * ${i} = ${(a=i*j)>=10?a:a+' ' }  `
        }
console.log(str)
}
```
