<font size=5 face='微软雅黑'>__文章目录__</font>

<!-- TOC -->

- [1 CSRF跨站请求伪造](#1-csrf跨站请求伪造)
    - [1.1 CSRF攻击介绍及防御](#11-csrf攻击介绍及防御)
    - [1.2 防御CSRF攻击](#12-防御csrf攻击)
        - [1.2.1 验证 HTTP Referer 字段](#121-验证-http-referer-字段)
        - [1.2.2 在请求地址中添加 token 并验证](#122-在请求地址中添加-token-并验证)
        - [1.2.3 在 HTTP 头中自定义属性并验证](#123-在-http-头中自定义属性并验证)
        - [1.2.4 django csrf token](#124-django-csrf-token)
    - [1.3 form表单提交](#13-form表单提交)
    - [1.4 ajax提交](#14-ajax提交)
    - [1.5 CSRF装饰器](#15-csrf装饰器)
- [2 Cookie&Session](#2-cookiesession)
    - [2.1 cookie/cookies](#21-cookiecookies)
        - [2.1.1 django中cookies用法介绍](#211-django中cookies用法介绍)
        - [2.1.2 加密的cookies](#212-加密的cookies)
        - [2.1.3 基于自定义分页的实例](#213-基于自定义分页的实例)
        - [2.1.4 利用cookie进行用户登陆检测](#214-利用cookie进行用户登陆检测)
    - [2.2 Session](#22-session)
        - [2.2.1 Django 中的 session](#221-django-中的-session)
        - [2.2.2 django 中session的操作](#222-django-中session的操作)
            - [2.2.2.1 设置session](#2221-设置session)
            - [2.2.2.2 获取session](#2222-获取session)
            - [2.2.2.3 注销时清除用户对应session信息](#2223-注销时清除用户对应session信息)
            - [2.2.2.4 所有 键、值、键值对](#2224-所有-键值键值对)
            - [2.2.2.5 其他操作](#2225-其他操作)
        - [2.2.3 使用session进行登陆验证](#223-使用session进行登陆验证)
        - [2.2.4 全局配置session](#224-全局配置session)

<!-- /TOC -->

# 1 CSRF跨站请求伪造
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CSRF跨站点请求伪造(Cross—Site Request Forgery)，跟XSS攻击一样，存在巨大的危害性，你可以这样来理解：攻击者盗用了你的身份，以你的名义发送恶意请求，对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作，比如以你的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。 

## 1.1 CSRF攻击介绍及防御
人设：Web A为存在CSRF漏洞的网站，Web B为攻击者构建的恶意网站，User C为Web A网站的合法用户。  
CSRF攻击攻击原理及过程如下：
1. 用户C打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
2. 在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
3. 用户未退出网站A之前，在同一浏览器中，打开一个TAB页访问网站B；
4. 网站B接收到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点A；
5. 浏览器在接收到这些攻击性代码后，根据网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自网站B的恶意代码被执行。

PS：还有一种情况是：多窗口浏览器，它便捷的同时也带来了一些问题，因为多窗口浏览器新开的窗口是具有当前所有会话的。即我用IE登陆了我的Blog，然后我想看新闻了，又运行一个IE进程，这个时候两个IE窗口的会话是彼此独立的，从看新闻的IE发送请求到Blog不会有我登录的cookie；但是多窗口浏览器永远都只有一个进程，各窗口的会话是通用的，即看新闻的窗口发请求到Blog是会带上我在blog登录的cookie，也有可能产生CSRF攻击。

下面是一个实例：
```bash
1. 受害者 Bob 在银行有一笔存款，通过对银行的网站发送请求 http://bank.example/withdraw?account=bob&amount=1000000&for=bob2 可以使 Bob 把 1000000 的存款转到 bob2 的账号下。通常情况下，该请求发送到网站后，服务器会先验证该请求是否来自一个合法的 session，并且该 session 的用户 Bob 已经成功登陆。

2. 黑客 Mallory 自己在该银行也有账户，他知道上文中的 URL 可以把钱进行转帐操作。Mallory 可以自己发送一个请求给银行：http://bank.example/withdraw?account=bob&amount=1000000&for=Mallory。但是这个请求来自 Mallory 而非 Bob，他不能通过安全认证，因此该请求不会起作用。

3. 这时，Mallory 想到使用 CSRF 的攻击方式，他先自己做一个网站，在网站中放入如下代码： src=”http://bank.example/withdraw?account=bob&amount=1000000&for=Mallory ”，并且通过广告等诱使 Bob 来访问他的网站。当 Bob 访问该网站时，上述 url 就会从 Bob 的浏览器发向银行，而这个请求会附带 Bob 浏览器中的 cookie 一起发向银行服务器。大多数情况下，该请求会失败，因为他要求 Bob 的认证信息。但是，如果 Bob 当时恰巧刚访问他的银行后不久，他的浏览器与银行网站之间的 session 尚未过期，浏览器的 cookie 之中含有 Bob 的认证信息。这时，悲剧发生了，这个 url 请求就会得到响应，钱将从 Bob 的账号转移到 Mallory 的账号，而 Bob 当时毫不知情。等以后 Bob 发现账户钱少了，即使他去银行查询日志，他也只能发现确实有一个来自于他本人的合法请求转移了资金，没有任何被攻击的痕迹。而 Mallory 则可以拿到钱后逍遥法外。 
```

## 1.2 防御CSRF攻击
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;目前防御 CSRF 攻击主要有三种策略：验证 HTTP Referer 字段；在请求地址中添加 token 并验证；在 HTTP 头中自定义属性并验证。

### 1.2.1 验证 HTTP Referer 字段
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，它记录了该 HTTP 请求的来源地址。在通常情况下，访问一个安全受限页面的请求来自于同一个网站，比如需要访问 http://bank.example/withdraw?account=bob&amount=1000000&for=Mallory，用户必须先登陆 bank.example，然后通过点击页面上的按钮来触发转账事件。这时，该转帐请求的 Referer 值就会是转账按钮所在的页面的 URL，通常是以 bank.example 域名开头的地址。而如果黑客要对银行网站实施 CSRF 攻击，他只能在他自己的网站构造请求，当用户通过黑客的网站发送请求到银行时，该请求的 Referer 是指向黑客自己的网站。因此，要防御 CSRF 攻击，银行网站只需要对于每一个转账请求验证其 Referer 值，如果是以 bank.example 开头的域名，则说明该请求是来自银行网站自己的请求，是合法的。如果 Referer 是其他网站的话，则有可能是黑客的 CSRF 攻击，拒绝该请求。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种方法的显而易见的好处就是简单易行，网站的普通开发人员不需要操心 CSRF 的漏洞，只需要在最后给所有安全敏感的请求统一增加一个拦截器来检查 Referer 的值就可以。特别是对于当前现有的系统，不需要改变当前系统的任何已有代码和逻辑，没有风险，非常便捷。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然而，这种方法并非万无一失。Referer 的值是由浏览器提供的，虽然 HTTP 协议上有明确的要求，但是每个浏览器对于 Referer 的具体实现可能有差别，并不能保证浏览器自身没有安全漏洞。使用验证 Referer 值的方法，就是把安全性都依赖于第三方（即浏览器）来保障，从理论上来讲，这样并不安全。事实上，对于某些浏览器，比如 IE6 或 FF2，目前已经有一些方法可以篡改 Referer 值。如果 bank.example 网站支持 IE6 浏览器，黑客完全可以把用户浏览器的 Referer 值设为以 bank.example 域名开头的地址，这样就可以通过验证，从而进行 CSRF 攻击。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;即便是使用最新的浏览器，黑客无法篡改 Referer 值，这种方法仍然有问题。因为 Referer 值会记录下用户的访问来源，有些用户认为这样会侵犯到他们自己的隐私权，特别是有些组织担心 Referer 值会把组织内网中的某些信息泄露到外网中。因此，用户自己可以设置浏览器使其在发送请求时不再提供 Referer。当他们正常访问银行网站时，网站会因为请求没有 Referer 值而认为是 CSRF 攻击，拒绝合法用户的访问。

### 1.2.2 在请求地址中添加 token 并验证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CSRF 攻击之所以能够成功，是因为黑客可以完全伪造用户的请求，该请求中所有的用户验证信息都是存在于 cookie 中，因此黑客可以在不知道这些验证信息的情况下直接利用用户自己的 cookie 来通过安全验证。要抵御 CSRF，关键在于在请求中放入黑客所不能伪造的信息，并且该信息不存在于 cookie 之中。可以在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种方法要比检查 Referer 要安全一些，token 可以在用户登陆后产生并放于 session 之中，然后在每次请求时把 token 从 session 中拿出，与请求中的 token 进行比对，但这种方法的难点在于如何把 token 以参数的形式加入请求。对于 GET 请求，token 将附在请求地址之后，这样 URL 就变成 http://url?csrftoken=tokenvalue。 而对于 POST 请求来说，要在 form 的最后加上 <input type=”hidden” name=”csrftoken” value=”tokenvalue”/>，这样就把 token 以参数的形式加入请求了。但是，在一个网站中，可以接受请求的地方非常多，要对于每一个请求都加上 token 是很麻烦的，并且很容易漏掉，通常使用的方法就是在每次页面加载时，使用 javascript 遍历整个 dom 树，对于 dom 中所有的 a 和 form 标签后加入 token。这样可以解决大部分的请求，但是对于在页面加载之后动态生成的 html 代码，这种方法就没有作用，还需要程序员在编码时手动添加 token。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该方法还有一个缺点是难以保证 token 本身的安全。特别是在一些论坛之类支持用户自己发表内容的网站，黑客可以在上面发布自己个人网站的地址。由于系统也会在这个地址后面加上 token，黑客可以在自己的网站上得到这个 token，并马上就可以发动 CSRF 攻击。为了避免这一点，系统可以在添加 token 的时候增加一个判断，如果这个链接是链到自己本站的，就在后面添加 token，如果是通向外网则不加。不过，即使这个 csrftoken 不以参数的形式附加在请求之中，黑客的网站也同样可以通过 Referer 来得到这个 token 值以发动 CSRF 攻击。这也是一些用户喜欢手动关闭浏览器 Referer 功能的原因。

### 1.2.3 在 HTTP 头中自定义属性并验证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种方法也是使用 token 并进行验证，和上一种方法不同的是，这里并不是把 token 以参数的形式置于 HTTP 请求之中，而是把它放到 HTTP 头中自定义的属性里。通过 XMLHttpRequest 这个类，可以一次性给所有该类请求加上 csrftoken 这个 HTTP 头属性，并把 token 值放入其中。这样解决了上种方法在请求中加入 token 的不便，同时，通过 XMLHttpRequest 请求的地址不会被记录到浏览器的地址栏，也不用担心 token 会透过 Referer 泄露到其他网站中去。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然而这种方法的局限性非常大。XMLHttpRequest 请求通常用于 Ajax 方法中对于页面局部的异步刷新，并非所有的请求都适合用这个类来发起，而且通过该类请求得到的页面不能被浏览器所记录下，从而进行前进，后退，刷新，收藏等操作，给用户带来不便。另外，对于没有进行 CSRF 防护的遗留系统来说，要采用这种方法来进行防护，要把所有请求都改为 XMLHttpRequest 请求，这样几乎是要重写整个网站，这代价无疑是不能接受的。

### 1.2.4 django csrf token
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;django中，是通过中间件来对CSRF进行验证的，有关中间件的信息，将在后面的小节中说明。PS：还记得之前在settings.py中注释掉的 django.middleware.csrf.CsrfViewMiddleware 吗？，它就是用于检测CSRF的中间件。如果我们render返回页面时，那么用户使用get请求访问网站时，站点会返回一段随机的csrf_token，同时也会在cookie中写一个名为csrftoken的key，这两个key对应的值是不同的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据这两种token，我们有两种方式，传递、验证CSRF token（首先打开csrf中间件，如果之前注释掉的话）即：1、form表单提交  2、ajax提交

## 1.3 form表单提交 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在表单中添加{% csrf_token %} 来渲染服务端返回的csrf_token，它会以hidden的形式在form中产生一个input标签，在我们利用form提交时会一并携带发送给服务端进行验证,当我们使用ajax代替form的submit进行提交时，可以在data字段，来获取form表单中渲染的csrf_token，进行提交，当然，这里的key的名称必须为csrfmiddlewaretoken，否则无法识别，从生成的input的name属性就能看到。
```html
<form action="/login" method="post" >
{% csrf_token %}        #渲染csrf_token
    <div>
        <label for="user">用户名：</label>
        <input type="text" id="user" name="user" placeholder='Username'>
    </div>
    <div>
        <label for="passwd">密码：</label>
        <input type="password" id="passwd" name="passwd" placeholder='Password'>
    </div>
        <input type="submit" value="登陆">
</form>
```
在前端我们看到的是：

```html
<form action="/file" method="post" enctype="multipart/form-data">
<input type="hidden" name="csrfmiddlewaretoken" value="8HElv24C9ajJZUE8gX3Cs9e0iIaXgX9iMvIfXZsKthjazqwnAvtrEvS84QC2pEik">   # 渲染的input标签
    <div>
        <label for="user">用户名：</label>
        <input type="text" id="user" name="user" placeholder='Username'>
    </div>
    <div>
        <label for="passwd">密码：</label>
        <input type="password" id="passwd" name="passwd" placeholder='Password'>
    </div>
        <input type="submit" value="登陆">
</form>
```
通过AJAX来获取Form的csrftoken进行提交　
```js
$(function(){
    $('#btn').click(function () {
        $.ajax({
            url:'/login',
            type:'POST',
            data:{'csrfmiddlewaretoken':$("input[name='csrfmiddlewaretoken']").val(),......},    // name属性选择器获取csrftoken
            success:function () {
                alert(123)
        }
        })
    })
})
```
为什么是'csrfmiddlewaretoken'这个名字？
```python
# 1、倒入 csrf中间件查看源码，获取的到底是什么信息
from django.middleware.csrf import CsrfViewMiddleware
 
# 2、源码中关于from的csrf定义 295行
request_csrf_token = ""
if request.method == "POST":
    try:
        request_csrf_token = request.POST.get('csrfmiddlewaretoken', '')   # 这里获取了csrfmiddlewaretoken，所以我们form必须要传递这个key才行。
```

## 1.4 ajax提交　　
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当页面中没有form表单(或者说不想使用form手动渲染)，那么还可以使用cookie中的csrftoken进行csrf验证，由于cookie在请求时是放在请求头中的，所以这里使用ajax的headers来定制请求头，注意请求头的key必须为(X-CSRFtoken），利用jquery cookie插件，使用$.cookie('csrftoken')来获取对应的值即可。 
```js
$(function(){
    $('#btn').click(function () {
        $.ajax({
            url:'/index',
            type:'post',
            headers:{'X-CSRFtoken':$.cookie('csrftoken')},     // 构建请求头
            data:{'user':$('#user').val(),'passwd':$('#passwd').val()},
            success:function () {
                alert(123)
        }
        })
    })
})
```
为什么是'X-CSRFtoken'这个名字？
```python
# 1、倒入 csrf中间件查看源码，获取的到底是什么信息
from django.middleware.csrf import CsrfViewMiddleware
 
 
# 2、源码中关于header中的csrf说明，295行(django 1.11)
request_csrf_token = ""
if request.method == "POST":
    try:
        request_csrf_token = request.POST.get('csrfmiddlewaretoken', '')
    except IOError:
        pass
if request_csrf_token == "":
    request_csrf_token = request.META.get(settings.CSRF_HEADER_NAME, '')
# 如果用户提交的数据中没有request_csrf_token，那么尝试从meta中获取数据(包含用户提交的所有数据也包括请求头，从中获取settings.CSRF_HEADER_NAME的值用作request_csrf_token)
 
     
# 3、倒入settings查找csrftoken的真正名称
from django.conf import settings
print(settings.CSRF_HEADER_NAME)     # 结果为：HTTP_X_CSRFTOKEN
 
 
# 4、当我们利用ajax定制 request headers时，django会为我们在我们定制的key前面添加HTTP_来进行标识，
# 所以我们实际上传递的key应该为X_CSRFTOKEN,
# 又因为提交数据时不能提交包含下划线的key，会被认为是非法数据，所以，这里的名称为X-CSRFTOKEN
# 官方建议为：X-CSRFtoken
```
PS：这里的settings模块和settings.py文件不是一回事儿，settings模块中包含了django很多的默认配置信息，如果我们要对某个配置信息进行修改，可以在settings.py中定义，如果没有定义，那么django就会按照settings中的默认配置进行处理。

扩展信息：
>在商城 APP 开发时，在与客户端联调 API 接口的过程中，我们发现，在 PHP 的 $_SERVER 超全局变量中某些自定义的 HEADER 字段居然获取不到。通过抓包工具查看数据包，该自定义头的确是存在的。
后来通过调试我们发现，根本原因是客户端错误地将字段名中的中划线写成了下划线。例如，应该是 X-ACCESS-TOKEN，却被写成了 X_ACCESS_TOKEN。
问题本身很好解决。然而我们想要知道，服务器为何要对字段名中使用了下划线的头视而不见呢？并且，不管是 Apache 还是 Nginx，对于这样的情况，都不约而同地采取了一样的策略。
在 RFC 2616 4.2 节中，有如下一段话：
Request (section 5) and Response (section 6) messages use the generic message format of RFC 822 [9] for transferring entities (the payload of the message).
这段话的意思，就是说 HTTP/1.1 的请求和响应消息使用 RFC 822 中的通用消息格式来传输实体（消息载荷）。
在 RFC 822 3.1.2 节中，对于消息格式的说明，有这样一句话：
The  field-name must be composed of printable ASCII characters (i.e., characters that  have  values  between  33.  and  126., decimal, except colon).
也就是说，HEADER 字段名可以可由可打印的 ASCII 字符组成（也就是十进制值在 33 和 126 之间的字符，不含冒号）。
不含冒号很容易理解，因为 Field-Name 和 Value 之间需要用冒号分割。然而，我们通过查询 ASCII 码表可知，下划线的十进制 ASCII 值为 95，也在此范围之内！
其实，在 HEADER 字段名中使用下划线其实是合法的、符合 HTTP 标准的。服务器之所以要默认禁止使用是因为 CGI 历史遗留问题。下划线和中划线都为会被映射为 CGI 系统变量中名中的下划线，这样容易引起混淆。
在 nginx 服务器中，通过显式地设置 underscores_in_headers on 可以开启在字段名中使用下划线。默认该选项是关闭的，所以在默认情况下，所有包含下划线的字段名都会被丢弃。
在我们的开发机中的确也开启了这个选项，为啥还是不能拿到字段名中包含下划线的 HEADER 呢？这是因为我们访问这台开发机的时候，前面还有一层代理服务器，而这台代理服务器没有开启相关选项，导致这种 HEADER 被代理服务器直接丢弃。因此也强烈建议不要在 HEADER 的 Field-Name 中使用下划线。

　　当我们对之前的ajax提交进行改造时，由于之前没有csrftoken的验证，那么如果我们开启验证，那么就需要对所有的ajax提交进行修改，那么这里可以使用ajax的setup进行全局的配置(可以查看上一篇ajax部分内容)。

```js
$(function(){
 
    // 利用ajaxSetup进行全局配置
    $.ajaxSetup({
        beforeSend:function (xhr,settings) {                            // 在发送ajax请求之前，要执行的函数
            xhr.setRequestHeader('X-CSRFtoken',$.cookie('csrftoken'))   // 定制请求头，添加csrftoken
        }
    })
 
    // 其他ajax请求，就无需再定制header发送csrf了。
    $('#btn').click(function () {
        $.ajax({
            url:'/index',
            type:'post',
            data:{'user':$('#user').val(),'passwd':$('#passwd').val()},
            success:function () {
                alert(123)
        }
        })
    })
})
```
PS：比如get，delete等请求是不需要携带csrftoken的，那么我们可以使用在进行全局配置时，对HTTP请求方式进行判断。

```html
# 判断HTTP请求方式。针对POST请求提交CSRFtoken
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    {% csrf_token %}
  
    <input type="button" onclick="Do();"  value="Do it"/>
  
    <script src="/static/plugin/jquery/jquery-1.8.0.js"></script>
    <script src="/static/plugin/jquery/jquery.cookie.js"></script>
    <script type="text/javascript">
        var csrftoken = $.cookie('csrftoken');
  
        function csrfSafeMethod(method) {
            // these HTTP methods do not require CSRF protection
            return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));      # 如过是GET|HEAD|OPTION|TRACE 这四种请求，那么返回False
        }
        $.ajaxSetup({
            beforeSend: function(xhr, settings) {
                if (!csrfSafeMethod(settings.type) && !this.crossDomain) {    # 针对非GET|HEAD|OPTION|TRACE的请求添加CSRFtoken
                    xhr.setRequestHeader("X-CSRFToken", csrftoken);
                }
            }
        });
        function Do(){
  
            $.ajax({
                url:"/app01/test/",
                data:{id:1},
                type:'POST',
                success:function(data){
                    console.log(data);
                }
            });
  
        }
    </script>
</body>
</html>
```

## 1.5 CSRF装饰器
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在我们的网站中，其实并不是所有的站点都需要进行csrftoken验证，那么如何对指定的站点函数进行csrf验证或者排除呢？
- `@csrf_protect`，为当前函数强制设置防跨站请求伪造功能，即settings中没有设置全局中间件。
- `@csrf_exempt`，取消当前函数防跨站请求伪造功能，即settings中设置了全局中间件。

使用csrf装饰器需要先进行引用：from django.views.decorators.csrf import csrf_exempt,csrf_protect
```python
from django.shortcuts import render,HttpResponse,redirect
from django.views.decorators.csrf import csrf_exempt,csrf_protect    # 导入csrf装饰器
 
@csrf_exempt                    # 设置该函数取消csrf认证
def index(request):
    error_msg = ''
    if request.method == 'POST':
        user = request.POST.get('user')
        passwd = request.POST.get('passwd')
        if user == 'daxin' and passwd == '123456' or user == 'dachenzi' and passwd == '123456':
            request.session['username'] = 'daxin'
            request.session['is_login'] = True
            return redirect('/page')
        else:
            error_msg = '用户名或密码错误'
    return render(request,'index.html',{'error_msg':error_msg,'test':{'name':'daxin','age':20}})
```

# 2 Cookie&Session
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session.典型的场景比如购物车，当你点击下单按钮时，由于HTTP协议无状态，所以并不知道是哪个用户操作的，所以服务端要为特定的用户创建了特定的Session，用用于标识这个用户，并且跟踪用户，这样才知道购物车里面有几本书。这个Session是保存在服务端的，有一个唯一标识。在服务端保存Session的方法很多，内存、数据库、文件都有。集群的时候也要考虑Session的转移，在大型的网站，一般会有专门的Session服务器集群，用来保存用户会话，这个时候 Session 信息都是放在内存的，使用一些缓存服务比如Memcached之类的来放 Session。服务端如何识别特定的客户？这个时候Cookie就登场了。每次HTTP请求的时候，客户端都会发送相应的Cookie信息到服务端。实际上大多数的应用都是用 Cookie 来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在 Cookie 里面记录一个Session ID，以后每次请求把这个会话ID发送到服务器，我就知道你是谁了。

用一句话总结的话：
- Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在缓存集群、数据库、文件中。   （服务端存储）
- Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。    （客户端存储）

## 2.1 cookie/cookies
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cookie 可以翻译为“小甜品，小饼干” ，Cookie 在网络系统中几乎无处不在，当我们浏览以前访问过的网站时，网页中可能会出现 ：你好 XXX，这会让我们感觉很亲切，就好像吃了一个小甜品一样。这其实是通过访问主机中的一个文件来实现的，这个文件就是 Cookie。在 Internet 中，Cookie 实际上是指小量信息，是由 Web 服务器创建的，将信息存储在用户计算机上的文件，不同的网站有不同的cookie，所以又成为cookies。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当 Web 服务器创建了Cookies 后，只要在其有效期内，当用户访问同一个 Web 服务器时，浏览器首先要检查本地的Cookies，并将其原样发送给 Web 服务器。而当用户结束浏览器会话时，系统将终止所有的 Cookie。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;目前有些 Cookie 是临时的，有些则是持续的。临时的 Cookie 只在浏览器上保存一段规定的时间，一旦超过规定的时间，该 Cookie 就会被系统清除，而持续性的可以永久进行存储。

PS：一个浏览器能创建的 Cookie 数量最多为 300 个，并且每个不能超过 4KB，每个 Web 站点能设置的 Cookie 总数不能超过 20 个。

在django中使用Cookie
　　现在有这样一个场景，index页面需要用户首先访问login页面，登陆成功后才能访问index页面，针对这个需求，如果只使用我们前面介绍的知识是无法进行完成的，而利用cookie则非常简单。
```python
# ------------------- views.py -------------------
 
def index(request):
    if request.method == 'GET':
        username = request.COOKIES.get('user')    # 获取用户请求中携带的cookies
        return render(request,'index.html',{'username':username})
 
def login(request):
    if request.method == 'GET':
        return render(request,'login.html')

    if request.method == 'POST':
        u = request.POST.get('username')
        p = request.POST.get('password')
        if u == 'daxin' and p == '123123':
            req = redirect('/index')
            req.set_cookie('user',u)            # 设置cookie
            return req
        else:
 
            return render(request,'login.html')
```
### 2.1.1 django中cookies用法介绍
针对cookies，django 提供了设置以及获取的方法，并且还有可选的参数。
```python
# 用变量接受返回用户的信息
response = redirect('/index')
response = render(request,'index.html')
response = Httpresponse('index')
 
# 设置cookie
response.set_cookie('key','value')
 
# 获取cookie
request.COOKIE.get('key')
```
set_cookie方法还提供了其他的参数用于配置cookie：
- key=''：键
- value=''：值
- max_age=None：过多少时间超时，单位是秒(默认是临时生效，重新启动浏览器失效)
- expires=None：在某个时间后失效，是一个具体的时间。
- path='/'：Cookie生效的路径，/ 表示根路径(默认)，根路径的cookie可以被当前站点的任何url的页面访问
- domain=None：Cookie生效的域名
- secure=False：https传输
- httponly=False：只能http协议传输，无法被JavaScript获取（不是绝对，底层抓包可以获取到也可以被覆盖）

```python
req.set_cookie('username','abc',max_age=10,path='/',......)
```

### 2.1.2 加密的cookies
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们在浏览器上可以看到我们指定的cookie中的key值，这样不是特别安全，那么我们可以对cookie进行加密。django提供cookie的加密方法。
```python
req.set_signed_cookie('name','abc.123',salt='abc.123')                 # 设置签名的cookie
request.get_signed_cookie('name',salt='abc.123',default='null')        # 获取签名的cookie
```
PS：salt表示要加的签名，获取时，必须要传递对应的签名，才可以正确获取。

### 2.1.3 基于自定义分页的实例
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通常我们在数据列表页面下面会看到供用户选择的页面数据显示条目数，一旦确定以后，那么以后选择下一页上一页，将会针对用户选择的条目数生成新的页码和数据，那么这个功能是怎么实现的呢？如何才能让浏览器记住用户选择的数据呢？这里通过cookie就能实现。

```python
# --------------------  html --------------------
 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .page {
            text-decoration: none;
            margin-right: 3px;
        }
 
        .active {
            background-color: saddlebrown;
            color: white;
        }
    </style>
</head>
<body>
<!-- 模拟数据显示 -->
<ul>
    {% for item in data %}
        <li>{{ item }}</li>
    {% endfor %}
</ul>
 
<div>
    <select name="count" id='count' onchange="pageChange(this)">
        <option value="10">10</option>
        <option value="30">30</option>
        <option value="50">50</option>
        <option value="100">50</option>
    </select>
</div>
{{ temp_list | safe }}
 
 
<script src="/static/jquery-3.3.1.min.js"></script>
<script src="/static/jquery.cookie.js"></script>
<script>
    $(function () {
        var val = $.cookie('page_size');
        if (val) {
            $('#count').val(val);        // 页面加载完毕，赋值页面数量
        }
 
    });
    function pageChange(ths) {
        var val = $(ths).val();          // 获取用户设置的页面条目数
        $.cookie('page_size',val);       // 写到cookie中(jquery.cookie)，额外的属性可以直接根{'path':'/',......}
        location.reload()
    }
</script>
</body>
</html>
 
# -------------------- views --------------------
from utils import pagination
 
def page(request):
 
    if request.method == 'GET':
        page_size = request.COOKIES.get('page_size')    # 获取page_size
        current_page = int(request.GET.get('page', 1))
        counts = len(all_data)
        if page_size:                   # 如果用户设置了page_size，修改生成的html
            page_obj = pagination.Page(current_page,counts,per_page_count=int(page_size))
        else:
            page_obj = pagination.Page(current_page, counts)
        data = all_data[page_obj.start:page_obj.end]
        temp_list =page_obj.page_str('/page/')
 
    return render(request,'page.html',{'data':data,'current_page':current_page,'temp_list':temp_list})
```

### 2.1.4 利用cookie进行用户登陆检测
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很多时候，比如我们要访问index页面，需要先进行登陆，不登陆那么拒绝访问index页面，这种场景下使用cookie可以很方便的完成。
```python
def auth(func):
    def index (request,*args,**kwargs):
        if not request.COOKIES.get('username'):     # 判断cookie中是否存在我们指定的key
            return redirect('/login')               # 不存在则跳转回login页面
        return func(request,*args,**kwargs)
    return index
```
PS：这里利用装饰器的方式对传入的function进行cookie验证，如果验证失败，那么跳转回login页面，在需要cookie验证的地方装饰即可
```python
@auth
def index(request):
    ... ...
```
扩展
- cookie会存放在相应头中，应答给用户浏览器的，用户浏览器收到后，会储存cookie。
- 上述设置cookie的方法其实就是对相应头的设置，如果我们要在相应头中添加额外的数据，那么可以使用如下方式

```python
req = redirect('/index')
req.set_signed_cookie('name','abc.123',salt='abc.123',max_age=10,path='/')
req['user'] = 'abc'      # 添加响应头数据
return req
```
## 2.2 Session
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之所以要把 cookie和session放在一起说，是因为它俩是有关系的，由于cookie 是存在用户端，而且它本身存储的尺寸大小也有限，最关键是用户可以是可见的，并可以随意的修改，很不安全。那如何又要安全，又可以方便的全局读取信息呢？于是，这个时候，一种新的存储会话机制：session 诞生了。session的机制是基于session_id的，它用来区分哪几次请求是一个人发出的。为什么要这样呢？因为HTTP是无状态无关联的，一个页面可能会被成百上千人访问，而且每个人的用户名是不一样的，那么服务器如何区分相同用户访问的呢？所以就有了找个唯一的session_id来绑定一个用户。一个用户在一次会话上就是一个session_id，这样成千上万的人访问，服务器也能区分到底是谁在访问了。而session_id，就是通过cookie答复给客户端的，所以session依赖cookie，即解决了安全问题，又能在服务端存储用户相关个性化数据。

### 2.2.1 Django 中的 session
首先根据session的工作原理，我们可以大致的分析出服务端对session的处理步骤。

服务端设置session：
1. 随机生成字符串作为session_id。       
2. 把session_id作为cookie返回给用户。
3. 在某个地方开辟空间存储该session_id。
4. 将用户相关信息存入session_id对应的字典或者其他数据类型中。
 
服务端获取session：
1. 获取cookie中的session_id
2. 根据session_id获取用户相关的数据

### 2.2.2 django 中session的操作
request封装的session，可以使用类似字典的方式进行操作。

#### 2.2.2.1 设置session
- request.session['username'] = u :仅仅这一步就完成了上面4步的功能
- request.session['is_login'] = True
  
#### 2.2.2.2 获取session
- request.session['is_login']
- request.session.get('is_login',None)
- request.session.setdefult('k1',123)
  
#### 2.2.2.3 注销时清除用户对应session信息
- request.session.clear():不会删除数据库中的sessionid,会修改对应id的数据(可能是做了标记，下次登陆会复用这个sessionid）
- request.session.delete(request.session.session_key):会删除数据库中的sessionid
- del request.session.get('username'):删除session中的某个key对应的值
  
#### 2.2.2.4 所有 键、值、键值对
- request.session.keys()
- request.session.values()
- request.session.items()
- request.session.iterkeys()
- request.session.itervalues()
- request.session.iteritems()

#### 2.2.2.5 其他操作

```python
# 用户session的随机字符串
request.session.session_key
  
# 将所有session失效日期小于当前日期的数据删除
request.session.clear_expired()
  
# 检查 用户的随机字符串 在数据库中是否存在
request.session.exists('session_key')
  
# 删除当前用户的所有Session数据
request.session.delete('session_key')
  
# 清空所有session
request.session.clear()
 
# django中的session默认有效期为两周，单独设置某个session的超时时间需要进行如下修改
request.session.set_expiry(value)
　　# 如果value是个整数(或者是个乘法表达式 60 * 60)，session会在些秒数后失效。
　　# 如果value是个datatime或timedelta，session就会在这个时间后失效。
　　# 如果value是0,用户关闭浏览器session就会失效。
　　# 如果value是None,session会依赖全局session失效策略。　　
```
PS：如果我们在session中写入了用户名称，需要在页面上展示，并不需要在后端获取(request.session.get('username'))，在后在前端渲染({{ username }})， 因为我们渲染页面的同时，传递了request对象，request对象又包涵了我们应答的所有信息，所以可以在页面上直接利用{{ request.session.username }} 进行渲染。

### 2.2.3 使用session进行登陆验证
上面使用cookie 对主页进行了登陆验证，这里使用session配合装饰器，完成页面登陆验证。
```python
def auth(func):
    def inner(request,*args,**kwargs):
        if request.session.get('is_login',None):
            return func(request,*args,**kwargs)
        else:
            return redirect('/login/')
 
    return inner
 
@auth
def index(request):
 
    if request.method == 'GET':
        return render(request,'index.html',{'username':request.session['username']})
 
def login(request):
 
    if request.method == 'GET':
        return render(request,'login.html')
 
    if request.method == 'POST':
        u = request.POST.get('username')
        p = request.POST.get('password')
 
        if u == 'daxin' and p == '123123':
            request.session['username'] = u
            request.session['is_login'] = True
            return redirect('/index/')
        else:
            return render(request,'login.html')
```
PS：django默认会把session_id和数据的对应关系放在数据库中，一般存放在django_session表中，如果是新建的项目，需要先执行如下命令生成表才可以使用。
```python
python3 manage.py makemigrations
python3 manage.py migrate
```

### 2.2.4 全局配置session
默认情况下session是有默认配置的，比如默认超时时间为两周，如果我们要修改全局配置，那么需要在settings.py中进行配置
```python
SESSION_COOKIE_NAME ＝ "sessionid"                        # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
SESSION_COOKIE_PATH ＝ "/"                                # Session的cookie保存的路径
SESSION_COOKIE_DOMAIN = None                              # Session的cookie保存的域名
SESSION_COOKIE_SECURE = False                             # 是否Https传输cookie
SESSION_COOKIE_HTTPONLY = True                            # 是否Session的cookie只支持http传输
SESSION_COOKIE_AGE = 1209600                              # Session的cookie失效日期（2周）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False                   # 是否关闭浏览器使得Session过期
SESSION_SAVE_EVERY_REQUEST = False                        # 是否每次请求都保存Session，默认修改之后才保存,建议修改为True
```
PS：上面的参数都是默认配置，如果需要修改单个配置，只需要添加对应的key和value就可以了。　

很多时候基于业务的需求，我们需要把session放在别的地方，比如缓存或者文件中，那么就需要使用如下配置了
```python
SESSION_ENGINE = 'django.contrib.sessions.backends.db'     # 默认配置，表示存放在数据库中
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 使用cache进行存储
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名(默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
```
上面cache_alias 其实是我们在配置文件中指定的chache的别名，因为我们如果要写到memcahed中，那么就需要写连接地址以及端口，那么就需要在cache中进行定义
```python
CACHES = {
    'default': {      # 这里就表示cache的别名
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
        ]
    }
}
```
PS：django默认支持memcached，不支持redis，需要安装其他插件。下面是缓存到文件中去
```python
'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()    
# 如：/var/folders/d3/j9tj0gz93dg06bmwxmhh6_xm0000gn/T
```