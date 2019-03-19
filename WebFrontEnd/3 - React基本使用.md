

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

## 2.5 增加子元素