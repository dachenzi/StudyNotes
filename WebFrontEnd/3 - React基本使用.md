

# 1 React
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;React是Facebook开发并开源的前端框架。当时他们的团队在市面上没有找到合适的MVC框架，就自己写了一个Js框架，用来架设Instagram（图片分享社交网络）。2013年React开源。
> React解决的是前端MVC框架中的View视图层的问题。

## 1.1 Virtual DOM
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DOM（文档对象模型Document Object Model）将网页内所有内容映射到一棵树型结构的层级对象模型上，浏览器提供对DOM的支持，用户可以是用脚本调用DOM API来动态的修改DOM结点，从而达到修改网页的目的，这种修改是浏览器中完成，浏览器会根据DOM的改变重绘改变的DOM结点部分。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改DOM重新渲染代价太高，前端框架为了提高效率，尽量减少DOM的重绘，提出了Virtual DOM，所有的修改都是先在Virtual DOM上完成的，通过比较算法，找出浏览器DOM之间的差异，使用这个差异操作DOM，浏览器只需要渲染这部分变化就行了。
> React实现了DOM Diff算法可以高效比对Virtual DOM和DOM间的差异。

## 1.2 支持JSX语法
JSX是一种JavaScript和XML混写的语法，是JavaScript的扩展。
```js
React.render(    /* React提供的渲染函数   */
 <div>  /* XML格式标签 */
    <div>
        <div>content</div>
    </div>
 </div>,
 document.getElementById('example') /*  javascript语法操作DOM  */
);
```
JSX规范：
1. 约定标签中`首字母小写`就是`html标记`，`首字母大写`就是`React组件`
2. 要求严格的HTML标记，要求所有标签都必须闭合。br也应该写成 <br /> ，/前留一个空格。
3. 单行省略小括号，多行请使用小括号
4. 元素有嵌套，建议多行，注意缩进
5. `JSX表达式`：表达式使用{}括起来，如果大括号内使用了引号，会当做字符串处理，例如 <div>{'2>1?true:false'}</div> 里面的表达式成了字符串了

# 2 基本使用

## 2.1 导入React
使用前需要进行导入操作：（下面的导入操作是基于ES6语法的)
```js
import React from 'react'; 导入react模块
import ReactDOM from 'react-dom'; 导入react的DOM模块，用于操作Virtual DOM
```

## 2.2 创建组件
React中我们自己写的类，被叫做组件，需要从`React.Component继承`，它会帮我们完成更多的功能。
```js

class Root extends React.Component {
    ... ... 
}

```
这个类生成`JSXElement对象`即React元素。

## 2.3 渲染函数
JSXElement对象内部可以使用rander函数来渲染一个内容，注意，只能返回唯一 一个顶级元素回去。
```js
class Root extends React.Component {
    render () {
        return <div>hello daxin</div>
    }
}
```
返回的内容，就是此Root对象，在网页上显示的标签信息  

当然也可以通过构造方法来构造一个简单的JSXElement对象
```js
class MyRoot extends React.Component {
  render() {
    return React.createElement('div',null,'hello world')
  }
}

ReactDom.render(<MyRoot />,document.getElementById('root'))
```
- 第一参数是React组件或者一个HTML的标签名称（例如div、span）。
- 第二参数是组件的props属性，如果是一个html标签，可以为null
- 第三参数是children信息，指代的是标签的内容
> 建议使用JSX语法直接生成需要的标签信息，这样看起来更简洁。

## 2.4 在网页上输出
如果需要在网页上输出我们的Root对象，那么就需要操作DOM，需要使用ReactDOM的render函数，可以帮我们在指定的DOM节点上，渲染标签
```js
ReactDom.render(<Root />, document.getElementById('root'));   // getElementById表示使用id的方式查找id为root的标签节点。
```
它接受两个参数：
- 第一个参数是JSXElement对象(`使用XML自闭合的格式编写`)
- 第二个是DOM的Element元素(`要在哪个dom节点下渲染`)
> 将React元素添加到DOM的Element元素中并渲染。

## 2.5 增加子元素(组件嵌套)
我们是可以在一个组件的render返回的顶级标签内，还可以继续嵌套自组建。
```js

class Sub extends React.Component {
  render (){
    return <div>from Sub</div>
  }
}

class Root extends React.Component {
  render (){
    return <div>From Root 
      <Sub />   // 首字母大写，否则会认为是HTML标签进行渲染
    </div>
  }
}
ReactDom.render(<Root />,document.getElementById('root'))
```
注意：
1. React组件的render函数return，只能是一个顶级元素
2. JSX语法是XML，要求所有元素必须闭合，注意 <br /> 不能写成 <br>

# 3 事件触发
在js中，我们可以针对不同的时间触发，来绑定相应的事件，来处理。下面是js中的事件：

属性|此事件发生在何时
-----|------|
onabort|图像的加载被中断
onblur|元素失去焦点
onchange|域的内容被改变
onclick|当用户点击某个对象时调用的事件句柄
ondblclick|当用户双击某个对象时调用的事件句柄
onerror|在加载文档或图像时发生错误
onfocus|元素获得焦点
onkeydown|某个键盘按键被按下
onkeypress|某个键盘按键被按下并松开
onkeyup|某个键盘按键被松开
onload|一张页面或一幅图像完成加载
onmousedown|鼠标按钮被按下
onmousemove|鼠标被移动
onmouseout|鼠标从某元素移开
onmouseover|鼠标移到某元素之上
onmouseup|鼠标按键被松开
onreset|重置按钮被点击
onresize|窗口或框架被重新调整大小
onselect|文本被选中
onsubmit|确认按钮被点击
onunload|用户退出页面

## 3.1 javascript中的事件触发
下面是一个onlick时间，当鼠标点击字体时，会触发alert警告，填写的函数需要加括号来触发执行
```html
<html>
    <head>
        <title>test evevt</title>
    </head>
    <script>
        function HandlerClick(event) {
            alert('test')
        }
    </script>
    <body>
        <h1>welcome to daxin </h1>
        <div onclick="HandlerClick(event);" style="color:red;background-color: aqua">
            点击我
        </div>
    </body>
</html>
```
event为触发的事件,由DOM封装成对象后传入。

## 3.2 React组件中的事件触发
在React中，事件均使用`小驼峰`语法。并且不需要调用，填写名称即可，调用时会自动加括号调用
```js
class Root extends React.Component {

  handlerClick (event){
    console.log(event.target)
    alert('test')
  }

  render (){
    return <div onClick={this.handlerClick}>From Root 
    </div>
  }
}

ReactDom.render(<Root />,document.getElementById('root'))
```
通常情况下，事件触发函数都会接受一个参数event，event.target是触发事件的标签本身。

# 4 组件状态(state)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个React组件都有一个状态属性state，它是一个JavaScript对象，可以为它定义属性来保存值。如果状态变化了，会触发UI的重新渲染(自动触发render渲染函数)。
> state是每个组件自己内部使用的，是组件自己的属性。
```js
class Root extends React.Component {
    state = {
        domain:'daxin',   // 不建议在这里设置，后面会放在constructor来指定
        suffix:'.com'
    }
    render(){
        return <div>{this.state.domain}{this.state.suffix}<br /><Sub /></div>
    }
}
```
为组件设置state属性，需要使用类Python字典的格式添加。在render函数中就可以使用this进行调用。

## 4.1 修改state
修改state其实是有两种方法：(这里this用在类内部，所以可以安全使用)
- `this.setState({name:'newname'})`：方法
- `this.state.name = 'newname'`：直接修改(`不建议使用这种方式`)

那么应该在哪里修改state呢，下面来看一个场景
```js
class Root extends React.Component {
    state = {
        domain:'daxin',
        suffix:'.com'
    }
    render(){   
        // this.state.suffix = '.org'  
        this.setState({suffix:})   
        return <div>{this.state.domain}{this.state.suffix}<br /><Sub /></div>
    }
}

ReactDom.render(<Root />,document.getElementById('root'))
```
上面代码在render函数中设置了state属性，但是页面无法运行，为什么呢？
- 渲染root时，会执行render函数。
- 执行render函数时，发现state参数改变，重新触发render函数改变，发生递归。

所以，最好不要在render函数中修改state属性(使用this.state.suffix = '.org'，不会出错，但是也不建议使用)

## 4.2 在事件中修改state
在事件中修改state属性：
```js
class Root extends React.Component {
  state = {
    name:'daxin',
    age:20
  }
  handlerClick (event) {
    console.log(event.target.id)  // 触发事件的id属性
    this.setState({age:this.state.age+1})
  }

  render (){
    return <div onClick={this.handlerClick}>
    My name is {this.state.name}
    I am {this.state.age} years old 
    </div>
  }
}

ReactDom.render(<Root />,document.getElementById('root'))
```
当我们点击标签是，发现在控制台(F12,console)中提示异常:
```js
Uncaught TypeError: Cannot read property 'setState' of undefined
```
这是因为，this是函数执行的上下文决定的，this已经不是触发事件的对象了。所以我们想要在内部使用this，那么就需要为handlerClick绑定this。
```js
class Root extends React.Component {
  state = {
    name:'daxin',
    age:20
  }
  handlerClick (event) {
    this.setState({age:this.state.age+1})
  }

  render (){
    return <div onClick={this.handlerClick.bind(this)}>  // 这里调用bind方法，绑定this
    My name is {this.state.name}
    I am {this.state.age} years old 
    </div>
  }
}

ReactDom.render(<Root />,document.getElementById('root'))
```
> 在事件触发函数中一定要记得将this进行绑定。

## 4.3 阻止默认事件触发
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在js中，由于触发事件先于默认事件触发，所以在触发函数内return false后，将会阻止默认时间产生，但React中不是这样的，如果要阻止事件默认行为，需要使用`event.preventDefault()`
```js
class Root extends React.Component {
  state = {
    name:'daxin',
    age:20
  }

  handlerClick (event) {
    event.preventDefault()   // 阻止a标签的跳转
    setTimeout( () =>  this.setState({age:this.state.age+1}),5000)  //延迟函数，延迟5000ms,执行前面的函数
  }

  render (){
    return <a onClick={this.handlerClick.bind(this)} href='https://www.baidu.com'>
    My name is {this.state.name}
    I am {this.state.age} years old 
    </a>
  }
}

ReactDom.render(<Root />,document.getElementById('root'))
```

# 5 属性(props)
在组件应用的地方，以类似html标签的设置方式，为组件添加属性，在组件内部可以进行使用props对象进行访问，设置方式：
```js
<Sub name='daxin'/>
```
> 类似于HTML标签的属性。

```js
class Root extends React.Component {
  state = {
    name:'daxin',
    age:20
  }

  handlerClick (event) {
    event.preventDefault()
    setTimeout( () =>  this.setState({age:this.state.age+1}),
    5000)
  }

  render (){
    return <a onClick={this.handlerClick.bind(this)} href='https://www.baidu.com'>
    My name is {this.state.name}
    I am {this.state.age} years old 
    My work is {this.props.work}  // 通过props对象获取属性信息
    </a>
  }
}

ReactDom.render(<Root work='Pythoner'/>,document.getElementById('root'))
```
## 5.1 嵌套标签的属性注入
在嵌套标签的场景下，有时我们需要在嵌套的子组件内调用父类的state信息。该怎么办呢？主要的解决办法有两个
- 单个导入
- 注解注入父标签本身

```js
class Sub extends React.Component {
  render () {
    // return <div>My Parent's Name is : {this.props.parent.state.name} </div> /* 通过parent对象访问父组件state状态的name
    // return <div>My Parent's Name is : {this.props.name} </div>  /* 通过注入的指定属性，访问 */

  }
}

class Root extends React.Component {
  state = {
    name:'daxin',
    age:20
  }
  
  render (){
    return <a onClick={this.handlerClick.bind(this)} >
    My name is {this.state.name}
    I am {this.state.age} years old 
    My work is {this.props.work}
    { <Sub parent={this}/> }               /* 把父表掐你当作parent属性注入 */
    <Sub name={this.state.name}/>         /* 仅仅传递父组件某个固定的属性 */
    </a>
  }
}

ReactDom.render(<Root name='daxin' work='Pythoner' />,document.getElementById('root'))
```

## 5.2 子标签的children
有的时候我们需要像使用XML、HTML那样，在标签内编写内容或者嵌套标签，就像下面这样
```js
<Sub> Hello world  </Sub>
```
如上面的例子：那么这就需要用到子组件的children属性了，可以理解为父标签为子标签Sub，注入了一些children。即把 ` Hello world ` 注入到了Sub的`props.children`中
```js
class Sub extends React.Component {
  render () {   
    return <div>My Parent's Name is : {this.props.name}
    {this.props.children}   // 必须在这里调用children渲染，才可以显示
    </div>
  }
}

class Root extends React.Component {
  state = {
    name:'daxin',
    age:20
  }

  render (){
    return <a onClick={this.handlerClick.bind(this)} >
    My name is {this.state.name}
    I am {this.state.age} years old 
    My work is {this.props.work}
    <Sub name={this.state.name}>
      <div> I am New Info</div>  // 直接这样写，内容是无法显示的
    </Sub>
    </a>
  }
}

ReactDom.render(<Root name='daxin' work='Pythoner' />,document.getElementById('root'))
```

## 5.3 props属性的修改
props是通过外部注入到组件内的，所以是只读属性，在组件内部只能读取无法修改，对比state我们知道：
- state属于组件的私有属性，组件外无法直接访问，可以修改，但建议使用setState方法
- props属于组件的公有属性，组件外也可以访问，但是只读

# 6 构造器(constructor)
非常类似于Python类中的__init__函数，当为组件定义state属性时，建议放在构造器中。它的一般格式为：
```js
constructor (props) {
    super(props)  //调用父类并传入props
    ... ...   // 私有属性等
}
```
props为固定参数，用于接收外部注入的props属性信息。`如果定义了constructor，必须在内部第一条语句中执行super(props)，将props传递给父类才行`。
```js
class Root extends React.Component {
  constructor (props){
    super(props)
    this.state = {
      name:'daxin',
      age:20
    }
  }

  render (){
    return <a>
    My name is {this.state.name}
    I am {this.state.age} years old 
    My work is {this.props.work}
    </a>
  }
}

ReactDom.render(<Root name='daxin' work='Pythoner' />,document.getElementById('root'))
```

# 7 组件的生命周期