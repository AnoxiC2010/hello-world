# HTML basic

了解前端基本知识, 了解Vue语法以及Vue开发流程

HTML -> 骨架
CSS  -> 皮肤和血肉
JS  -> 思想逻辑

## 互联网三要素

1. <span style="color:red">url:  统一资源定位符</span>

   <span style="color:red">url是分为三部分(四小部分): 1, 协议  2, 域名或者ip+端口  3, 服务器内部路径 </span>

   <span style="color:red">四小部分: 3, 服务器内部路径  → 服务器内部路径 + 参数</span>

   默认处理机制: http协议默认找80端口 https: 默认找443

   服务器内部路径, 就是标记, 资源在目标计算机中的虚拟地址

2. http: 数据传输格式

3. html: 描述论文格式

html最开始是设计用来描述论文格式的



## IDEA查看html页面的过程

`<img src="https://xxx.xxx.com/xxx...x=x.jpg">`

`<img src="./x.jpg">`

- IDEA模拟的服务器端

  IDEA启动时，会监听一个端口（默认63342），用来模拟一个服务器

- 浏览器端

  1. 浏览器发起请求 /localhost:63342

  2. IDEA所模拟的服务器接收请求，分析请求，返回数据（一份html文件）

  3. 浏览器拿到了html文件，解析html相关代码-->发现img标签

     如果是相对路径，浏览器会根据相对路径拼接一个绝对路径，在发起url请求

  4. 一边解析，一边发起获得图片的请求

  5. 获得图片

  

## 浏览器发url请求的过程

- 浏览器端

  1. 浏览器拿到用户输入或者超链接的url
  2. 如果url的第二部分是域名，那么要先转化为ip地址，因为只有ip地址能唯一标记一台计算机。（dns解析）
  3. 根据这个ip地址，找到对应的在互联网上的这台物理计算机的位置
  4. 浏览器拿到返回的html页面（实际上拿到的只是html, css, js代码）
  5. 把代码解析成页面-->显示

- dns服务器

  把域名兑换成ip地址

- 百度服务器

  1. 百度的服务器内部的一个服务，监听某一个端口，如果有请求请求这个端口，接受请求
  2. 把请求的url第三部分拿过来，分析，分析请求所需要的东西：html网页，数据，图片，视频，文件
  3. 要看有没有这个资源，404代表没有
  4. 如果有的话，给这个资源
  5. 如果用户需要的是一个html页面，并且这个页面需要数据，先去数据库查数据，把数据和html文件结合到一起-->形成一个带数据的html文件，返回给浏览器

- 百度数据库





# HTML概念

HTML(hyper text markup language)超文本标记语言

- 是一个以.html为后缀的文本

- 是一个文本,也是一个网页;该文本可以用浏览器打开

  (浏览器打开.html为后缀的文本, 会解析里面的代码--会渲染出页面)

- '超文本':包含文本字体,图片,链接,甚至音乐,程序等元素的文本

- HTML 是一个使用'标签'来描述网页的文本 (标签在HTML文本中有实际意义)

```
超文本标记语言 html是一个语言: 
是一个文本语言
是一个用标记/标签 来描述的文本语言

一个偏标准的编程语言应该具有哪些特点? 
变量的定义
运算符
分支和循环结构...

学习html就是学习html的标签的含义
```



## 开发工具

- 记事本
  - EidtPlus
  - Notepad++
- 前端专业版
  - Vscode
  - WebStorm
  - Sublime
  - Hbuilder



# HTML结构

HTML标准结构模板

```html
<html>
	<head>
	</head>
	<body>
	</body>
</html>
```

结构特点

- 由标签描述的文本

- 标签成对出现,有开始和结束

  <标签名></标签名>. 不是绝对的

- 文本由`<html>`标签包裹

- 文本分为两个部分(分别由头尾标签包裹)

- 标签嵌套: 通过标签的层级嵌套, 来描述结构关系(父子关系)



HTML重要body标签

```html
<hr> 
<br>
<h1>-<h6> 
<div>
<a> 
<p>
<img> 
<input>
< textarea > 
< select >
<ol>--<ul>
< table >--<tr>---<td>---<th> 
< form >
```



Html就是由标签构成
Html用标签来描述文档

一个Html文档分为两部分: Head body
标签也分为两大类:  <span style="background:yellow;">body子标签</span>,  head的子标签
现在: 组件化前端时代
body控制显示

xhtml: 已经夭折

W3C曾疯狂给html扩充标签, 添加属性  →  CSS（取而代之）

```html
<!DOCTYPE html>   html5语法声明
<html lang="en">     
    <head>
        <meta charset="UTF-8"> meta标记一些网页元信息
        <title>Title</title>
    </head>
    <body>
        
    </body>
</html>
```



# HTML(body)标签分类

## 块级标签

<span style="color:red;">特性</span>

- 独占一行，每一个块级元素都会从新的一行重新开始，从上到下排布
- 可以直接控制宽度、高度以及盒子模型的相关css属性
- 在不设置宽度的情况下，块级元素的宽度是它父级元素内容的宽度
- 在不设置高度的情况下，块级元素的高度是它本身内容的高度
- 块级元素是指本身属性为`display:block;`的元素。
- 通常使用块级元素来进行大布局（大结构）的搭建

常见的块级元素：

div、p、h1、 h2、 h3、 h4、 h5、 h6，ol、ul、dl、li、form、table



## 行级标签

<span style="color:red;">特性</span>

- 和其他行级元素从左到右在一行显示
- 不能直接控制宽度、高度以及盒子模型的相关css属性，(但是直接设置内外边距的左右值是可以的)
- 行级元素的宽高是由本身内容的大小决定（文字、图片等）
- 行级元素只能容纳文本或者其他行级元素（不要在行级元素中嵌套块级元素）
- 行级元素是指本身属性为`display:inline;`的元素。
- 通常使用行级元素来进行文字、小图标（小结构）的搭建。



常见的行级元素：

- span 常用行级容器，定义文本内区块;
- a 锚点;
- b 加粗;     strong 加粗强调;    i 斜体;    strike 中划线; 
- br 强制换行;   u 下划线;
- textarea 多行文本输入框; 
- input 输入框;
- select 下拉列表
- img 引入图片



## 行内块标签

特性

- <span style="color:red;">本质属于行级元素</span>
- 可以与其他行级元素共处一行
- <span style="color:red;">可以设置宽高、内外边距</span>
- 属性为`display:inline-block;`的元素。



常见的行内块元素：

- input 输入框
- img 引入图片



可以在行内样式或css样式中改变元素的display将三种元素进行转换。

- `display: block;`(将元素变为块级元素)
- `display: inline;` (将元素变为行级元素)
- `display: inline-block;`(将元素变为行级块元素)



## HTML重要标签

### `<hr> `

标签在 HTML 页面中创建一条水平线。

`<hr>` 标签没有结束标签。

标签属性

- `noshade`:颜色是否有阴影(纯色)
- `size`:高度( 厚度,不同于height,不带单位时--默认px )
- `width`:宽度(不带单位时--默认px)
- `align`:对其方式

```html
eg:
<hr>
<hr />: 这种写法不对, xhtml语法遗留产物
<hr noshade="noshade">
<hr noshade="noshade" size="50">
<hr noshade="noshade" size="50" width="50">
<hr noshade="noshade" size="50" width="50%">
<hr noshade="noshade" size="50" width="50%" align="left">
```

注意: html和css, 长度单位
px  像素,  逻辑像素

### `<br>`

可插入一个简单的换行符。

标签是单标签（意味着它没有结束标签，因此这是错误的：`<br></br>`）

`<br>` 标签只是简单地开始新的一行



<span style="color:red">注意: 在html代码上连续的空格或者换行, 在页面最终显示的时候, 仅会渲染出一个空格</span>
注意: html有一些特殊字符(不需要记忆, 只需要知道)

### `<h1>...<h6>`

`<h1>` - `<h6>` 标签可定义标题。

`<h1>` 定义最大的标题。`<h6>` 定义最小的标题。

属性`align`

- `left`:左对齐内容（默认值）
- `center`:右对齐内容
- `right`:居中对齐内容

<span style="color:red">注意1: 标题标签只有这六个</span>
<span style="color:red">注意2:  h1标签在页面上仅能出现一次(h1可以充当关键字), h2-h6可以出现多次</span>

网络爬虫, seo优化



### `<div>`

注意：

如果要实现一个页面,
<span style="color:red">第一步首先就是观察: 区和块,  划分区块, 实现内容</span>
<span style="color:red">第二步: 调</span>

大布局的调整和确定

<span style="color:red;background:yellow;">怎么写页面? 首先划分区节</span>



`<div>` 划分区或者节,  构建大布局. 最常用的一个html标签 

`<div>` 标签可以把文档分割为独立的、不同的部分。

属性`align`

- `left`:左对齐内容（默认值）
- `center`:右对齐内容
- `right`:居中对齐内容



### `<p>`

`<p>` 标签定义段落。极类似与div, 可以划分区块

`<p>` 元素会自动在其前后创建一些空白。浏览器会自动添加这些空间，您也可以在样式表中规定。

<span style="color:red;background:yellow;">p标签, 自带段前段后间距</span>



### `<a>`

`<a>` 标签定义超链接，用于从一张页面链接到另一张页面。

`<a>` 元素最重要的属性是 `href` 属性，它指示链接的目标

`<a href="http://www.baidu.com.cn">baidu</a>`

属性`href`: 指示打开的页面路径 (本质是个url)

属性`target`: 表示打开的方式

- 超链接属性`target`的`value`值时默认是`_self``

- ``_blank`:在新窗口打开

- `_parent`:在父窗口打开

- `_self`:它使目标文档显示在超链接所在框架或者窗口中

- `_top`:在顶级窗口打开

- 同组值:通过单击一个窗口中的不同链接控制另一窗口内容变化 ; 浏览器会找与`target`值相符的框架或者窗口中的文档，有则在其中显示文档。如果不存在，浏览器打开一个新窗口

  ```html
  _blank _self 同组值
  Eg:
  <a href="http://www.w3school.com.cn">默认</a>
  <a href="http://www.w3school.com.cn" target="_blank">_blank</a>
  <a href="http://www.w3school.com.cn" target="_self">_self</a>
  <hr>
  <li><a href="http://www.w3school.com.cn" target="view1">1</a></li>
  <li><a href="http://www.w3school.com.cn" target="view1">2</a></li>
  ```

  ```html
  _parent _top
  Eg:
  
  A.html
  <body bgcolor="red">
  --------a-------<br>
  <iframe src="b.html" width="800px" height="800px"></iframe>
  </body>
  
  b.html
  <body bgcolor="darkgrey">
      --------b-------<br>
      <iframe src="c.html" width="600px" height="600px"></iframe>
  </body>
  
  c.html同上
  
  d.html
  <body  bgcolor="#b8860b">
  --------d-------<br>
  <!--<a href="http://www.baidu.com" width="400px" height="400px">默认</a>-->
  <!--<a href="http://www.baidu.com" target="_self" width="400px" height="400px">_self</a>-->
  <!--<a href="http://www.baidu.com" target="_parent" width="400px" height="400px">_parent</a>-->
  <a href="http://www.baidu.com" target="_top" width="400px" height="400px">_top</a>
  </body>
  ```

  

### `<img>`

图片标签

属性

- `src` 图片地址

- `width`
- `height`
- 单位 px, %

eg:
`<img src=""  alt="加载错误" width="200" height="200" />`



<span style="color:red;background:yellow">注意: </span>
<span style="color:red;background:yellow">在前端中, 绝对路径, 不再是以盘符开头的路径.</span>
<span style="color:red;background:yellow">前端中本质上只有url,</span>
<span style="color:red;background:yellow">所谓的url就是前端的绝对路劲</span>

### `<input>`

<span style="color:red;background:yellow">\<input\>是一个表单元素</span>

`<input>`标签用于搜集用户信息。

根据不同的 type 属性值，输入字段拥有很多种形式。输入字段可以是文本字段、复选框、掩码后的文本控件、单选按钮、按钮等等。

属性`type`

- `text`:默认文本

- `password`:密码

- `button`:按钮

- `hidden`:隐藏输入框

- `radio`:单选框

- `reset`:定义重置按钮

- `submit`:提交

  ```html
  <input type="text" name="email" />
  <input type="password" name="pwd" />
  <input type="hidden" name="country" value="Norway" />
  <input type="radio" name="sex" value="male" /> Male
  <input type="radio" name="sex" value="female" /> Female
  <form action="form_action.asp" method="get">
  	<input type="text" name="email" />
  	<input type="reset" />
  	<input type="submit" />
  </form>
  
  ```

  

### `<textarea>`

<span style="color:red;background:yellow">\<textarea\>是一个表单元素</span>

`<textarea>` 标签定义多行的文本输入控件。

可以通过 `cols` 和 `rows` 属性来规定 `textarea` 的尺寸

属性

- `cols`:列

- `rows`:行

- `maxlength`:最大

- `placeholder`:提示

  ```html
  <textarea placeholder="请输入..." rows="5" cols="100" maxlength="50" >
  </textarea>`
  ```

- `readonly` :只读

  ```html
  <textarea placeholder="请输入..." rows="5" cols="100" maxlength="50" readonly>
  </textarea>
  ```

  

### `<select>`

<span style="color:red;background:yellow">\<select\>是一个表单元素</span>

选择下拉框

属性

- option:`selected`(`selected="selected"`选中状态)

- select :`multiple`(`multiple =“multiple“` 允许多选)

  按住`shift`+鼠标单击多选

- select :`size`(`size =“4”`下拉可见数4)

```html
eg:
<select multiple="multiple" size="1" >
    <option value=“aaa">aaa</option>
    <option value=“bbb">bbb</option>
    <option value=“ccc" selected="selected">ccc</option>
    <option value=“ddd">ddd</option>
</select>
```



### `<ol>`

`<ol>` 标签定义有序列表(有序:数字,字母)
属性:<span style="color:red;">\(CSS无法替代，但很少使用)</span>

- `type`:规定在列表中使用的标记类型(1,A,a,I,i)。

- `start`:规定有序列表的起始值

- `reversed`:规定列表顺序为降序。(9,8,7...)

  ```html
  Eg:
  <ol type="A" start="10" reversed>
      <li>1</li>
      <li>2</li>
      <li>3</li>
  </ol>
  ```

  

### `<ul>`

标签定义无序列表(圆点,空心点)

属性`type`:<span style="color:red;">\(CSS无法替代，但很少使用)</span>

- `disc`:默认值。实心圆。
- `circle` :空心圆。
- `square` :实心方块。

```html
Eg:
<ul type="circle">
    <li>Coffee</li>
    <li>Tea</li>
    <li>Milk</li>
</ul>
```



### `<li> `

`<li>` 标签定义列表项目。

`<li>` 标签可用在有序列表 (`<ol>`) 和无序列表 (`<ul>`) 中。



### `<table>`

`<table>` 标签定义 HTML 表格。

成套标签，结合标签`<tr><td><th>`标签使用

一个table标签代表一个表格, 一个tr标签代表一行, 一个td标签代表一行中一个单元格

主要属性

- `bgcolor`
- `border`
- `cellpadding`:规定单元边沿与其内容之间的空白。
- `cellspacing`:规定单元格之间的空白。

```html
Eg:
<table border="1px" bgcolor="red" cellpadding="30" cellspacing="100">
    <tr>
        <td>11</td>
        <td>12</td>
    </tr>
    <tr>
        <td>21</td>
        <td>22</td>
    </tr>
</table>
```

补充: th: 表头取代td  thead, tbody, tfoot 

注释：如果您使用 thead、tfoot 以及 tbody 元素，您就必须使用全部的元素。它们的出现次序是：thead、tfoot、tbody，这样浏览器就可以在收到所有数据前呈现页脚了。您必须在 table 元素内部使用这些标签。

```html
<table border="1">
  <thead>
    <tr>
      <th>Month</th>
      <th>Savings</th>
    </tr>
  </thead>

  <tfoot>
    <tr>
      <td>Sum</td>
      <td>$180</td>
    </tr>
  </tfoot>

  <tbody>
    <tr>
      <td>January</td>
      <td>$100</td>
    </tr>
    <tr>
      <td>February</td>
      <td>$80</td>
    </tr>
  </tbody>
</table>
```



### `<tr>`

`<tr>` 标签定义 HTML 表格中的行。

`<tr>` 元素包含一个或多个 <th> 或 <td> 元素。

属性

- bgcolor
- valign:规定表格行中内容的垂直对齐方式。
  - top
  - middle
  - bottom



### `<td>`

`<td>` 标签定义 HTML 表格中的标准单元格。

属性

- `colspan`:横跨列数
- `rowspan`:横跨行数

```HTML
Eg:
<table border="1px" bgcolor="red" cellpadding="30" cellspacing="100">
    <tr>
        <td colspan="2">11</td>
    </tr>
    <tr>
        <td rowspan="2">21</td>
        <td>22</td>
    </tr>
    <tr>
        <td>12</td>
    </tr>
</table>
```



### `<form>`

<span style="color:red;background:yellow;">表单元素常用于表单提交数据, 表单元素有一个默认的属性value→value等价于用户输入</span>

表单

`<form>`用于与服务器端的交互。

```html
Eg:
<form action=“url" method="get">
    <p>First name: <input type="text" name="fname" /></p>
    <p>Last name: <input type="text" name="lname" /></p>
    <input type="submit" value="Submit" />
</form>
```

form表单, 一般用来向后台(后端服务器) 提交数据 → 登录注册

用于搜索只是附带功能



#### 提交表单的过程演示

```html
<form action="https://www.baidu.com/s" method="get">
	<input name="wd">
    <input type="submit">
</form>
```

浏览器端

1. 在浏览器里面触发请求，url:https://www.baidu.com/ 

2. 百度服务器返回页面，浏览器拿到这个html页面解析执行，如果有外部资源(html代码中引用了图片等)，异步加载外部资源

3. 完全显示和得到一个页面

4. 在页面中表单的输入框中输入一个内容，点击搜索(从结果来看获得一个新页面，意味着点击搜索之后，又发起了一次网络请求)

url哪来的→点击搜索这个按钮所产生的

form是怎么产生一个url的

`https://www.baidu.com/s?wd=1`

key-value数据交互，数据具有自我描述性

value就是表单元素的value值

key就是表单元素的name属性设置的

除了参数的那一部分是哪来的→action中写的

参数哪来的→表单元素在表单中提交的

多个参数之间用&符号链接`https://s.taobao.com/search?x=1&q=2 `



#### 关于请求方式

method: 请求方式 http
get, post: 99.9999%  get:70-80%
为什么http要有多种请求方式,  发起一个http的请求目的是多种多样的, 进一步产生的很多不同的请求方式

get和post 的区别:

1. get请求,一般把参数拼接到url之后
   post请求(也可以把参数拼接到url之后), 但是一般放到请求正文里
2. get请求相对而言不安全
   post相对更安全一些
   第一个安全的方面:  参数不会放到url之后
   第二个安全: get请求浏览器历史记录中保存 post不会记录参数(正文)
3. url之后拼接参数, 最多拼接1kb
4. get和post的区别更多是'语义化'的区别
   get一般用来请求数据
   post一般用来提交数据  -- 登录注册, 添加信息

整体来讲, from登录注册.  



# CSS

**CSS历史**

- web的 衰落：
  在web早期（1990-1993）,html是一个很局限的语言。几乎完全由用于描述段落，超链接，列表和标题的结构化元素组成。随着万维网的出现（用户交互体验的加强），对html的要求越来越大，人们迫切需要html增加新的元素，去完成一个个特定的功能。
- 一片混乱：
  迫于压力，html开始出现`<font>`，`<i>` , `<s>` 等标签,还增加了某些标签的很多属性。<span style="color:red;">但是html是一种描述结构的语言，也开始描述外在表现了</span>。几年之后这种随便的做法暴露出严重的问题：1：由于html既写结构又写样式，导致页面缺乏结构性，降低了网页的可访问性。2：页面维护越来越困难
- 在1995年w3c开始发布CSS的计划(work-in-progress)。



**CSS概念**

CSS 指<span style="color:red;">层叠样式表</span> （级联样式表）
主要用于设置HTML页面中的文本内容（字体、大小、对齐方式等）、图片的外形（宽高、边框样式、边距等）以及版面的布局等等<span style="color:red;">外观显示</span>样式。
<span style="color:red;">CSS以HTML为基础</span>，提供了丰富样式功能。



# CSS与HTML结合方式

<span style="color:red;">CSS依赖于HTML而存在</span>

方式一:
```html
<p style="background-color:#FF0000; color:#FFFFFF">
	p标签段落内容。
</p>
```

方式二：
```html
<head>
    <style type=”text/css”>
        p { color:#FF0000;}
    </style>
</head>
```

方式三:

- 链接

  ```html
  <link rel="stylesheet" type="text/css" href="css_3.css" />
  ```

  

- 导入

  ```html
  <style type="text/css">
  	@import url(css_3.css);
  </style>
  ```



# 选择器

## 标签选择器

通过标签名, 把对应的html代码和他的css样式关联起来

e.g.

对所有`<p>`标签

```html
<style>
	p {color:red;}
</style>
```



## 类选择器

通过类名, 把css样式和html代码结合起来

类选择器的选择符是 “.“
类选择器在css样式编码中是最常用到的 

e.g.

style

```html
<!-- style -->
<style>
    .aaa{
        background: #FF0000;
    }
</style>
<!-- class -->
<div class="aaa">11
</div>
```



## id选择器

通过id名把对应的html代码和css样式结合起来

id选择器以 "#" 来定义
不同于类选择器，id 选择器不能结合使用
不提倡用id 去写样式，因为他的权重太高。 id 主要是为了 js 做准备。

注：同一个id名在一个html文档上只能出现一次（即使浏览器不报错也按规范写）

e.g.

```html
<!-- style -->
<style>
    #aaa{
        background: #FF0000;
    }
</style>
<!-- class -->
<div id="aaa">11
</div>
```



## 复合选择器

### 关联选择器（包含选择器）-->嵌套

```css
p { color:#00FF00;}
p b { color:#FF000;}
p { color:#00FF00;}
.aaa b { color:#FF000;}
```

后代选择器用来选择元素或元素组的后代，其写法就是把外层标记写在前面，内层标记写在后面，中间用空格分隔。当标记发生嵌套时，内层标记就成为外层标记的后代。

```html
<p>
	<b>刘德华</b>
</p>
<p>p标签段落</p>
```

### 组合选择器

多个不同选择器要用逗号分隔开

```css
p,div { color:#FF0000;}
```

```html
<p>P标签显示段落。</p>
<div>DIV标签显示段落</div>
```

类组合

```css
.aaa ,div { color:#FF0000;}
```

```html
<p class="aaa">
P标签显示段落。
</p>
<div>DIV标签显示段落</div>
```

## 属性选择器

`[attribute]`
用于选取带有指定属性的元素

```css
[target]
{ 
background-color:yellow;
}
div[id]
{ 
background-color:yellow;
}
```



`[attribute=value]`
用于选取带有指定属性和值的元素。

```css
a[href="http://www.w3school.com.cn/"][title="W3School"] {
color: red;
}
```



`[attribute~=value]`
用于选取属性值中包含指定词汇的元素。

```css
[title~=flower]
{ 
background-color:yellow;
}
```



`[attribute|=value]`
用于选取带有以指定值开头的属性值的元素，该值必须是整个单词。



`[attribute^=value]`
匹配属性值以指定值开头的每个元素。



`[attribute$=value]`
匹配属性值以指定值结尾的每个元素。



`[attribute*=value]`
匹配属性值中包含指定值的每个元素。

```css
a[href*="w3school.com.cn"] {color: red;}
```



## 伪类选择器

CSS伪类是用来添加一些选择器的特殊效果。

其实就在html中预先定义好的一些选择器。

:link 应用于未被访问过的链接；

:hover 应用于鼠标悬停到的元素；

:active 应用于被激活的元素；

:visited 应用于被访问过的链接，与:link互斥。

:focus 应用于拥有键盘输入焦点的元素。

:first-child 选择某个元素的第一个子元素；

:last-child 选择某个元素的最后一个子元素；

:nth-child() 选择某个元素的一个或多个特定的子元素；

:nth-last-child() 选择某个元素的一个或多个特定的子元素，从这个元素的最后一个子元素开始算；

:nth-of-type() 选择指定的元素；

:nth-last-of-type() 选择指定的元素，从元素的最后一个开始计算；

:first-of-type 选择一个上级元素下的第一个同类子元素；

:last-of-type 选择一个上级元素的最后一个同类子元素；

:only-child 选择的元素是它的父元素的唯一一个子元素；

:only-of-type 选择一个元素是它的上级元素的唯一一个相同类型的子元素；

:empty 选择的元素里面没有任何内容。

Eg:

```css
<a class="red" href="https://www.baidu.com/" target="_blank">CSS 语法</a>

a:link {color:#FF0000;} /* 未访问的链接 */ 不一定和你期望的一样

a:visited {color:#00FF00;} /* 已访问的链接 */

a:hover {color:#FF00FF;} /* 鼠标划过链接 */

a:active {color:#0000FF;} /* 已选中的链接 */
```

![img](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\HTML-CSS\HTML-CSS-basic-learning.assets\clip_image002.jpg)

```css
a.red:link {color:#FF0000;} /* 未访问的链接 */

a.red:visited {color:#00FF00;} /* 已访问的链接 */

a.red:hover {color:#FF00FF;} /* 鼠标划过链接 */

a.red:active {color:#0000FF;} /* 已选中的链接 */
```

 

## 伪元素选择器

伪元素是对元素中的特定内容进行操作，而不是描述状态。它的操作层次比伪类更深一层，因此动态性比伪类低很多
:first-letter 选择元素文本的第一个字（母）。
:first-line 选择元素文本的第一行。
:before 在元素内容的最前面添加新内容。
:after 在元素内容的最后面添加新内容。

```css
div::before {
content: "1111";
height: 40px;
width: 100px;
background-color: blue;
display: block;
}
```



## 区别(伪元素/伪类)

伪类：用于已有元素处于某种状态时为其添加对应的样式，这个状态是根据用户行为而动态变化的。
伪元素：用于创建一些不在DOM树中的元素，并为其添加样式。



# 样式优先级

`!important`
CSS定义了一个!important命令，该命令被赋予最大的优先级。也就是说不管权重如何以及样式位置的远近，!important都具有最大优先级。
 `div{ background: red !important; }`

!important <span style="color:red;">> 行内 > ID选择器 > 伪类|类 | 属性选择 > 标签 > 通配符</span>
<span style="color:red;">就近原则</span> ： 在优先级同等的情况下, 满足适配的就近原则(谁在下面听谁的)



# 盒子模型

一个html标签, 在页面上真正占据的空间, 我们可以把它看成一个矩形的盒子

<span style="color:red;">盒子模型, 从内向外大值可以分为四块内容:  元素内容大小, 内边距, 边框, 外边距</span>

<span style="background:yellow;">注意1:</span>
<span style="background:yellow;">对于一个盒子模型, 我们设置的宽高 仅仅是, 最里面元素内容的大小</span>
<span style="background:yellow;">注意2: </span>
<span style="background:yellow;">子元素or子标签, 仅能占据父元素的元素内容区域</span>

<span style="background:yellow;">注意3:</span>
 <span style="background:yellow;">(设计的略有缺陷)背景色 =  元素内容区域大小 +  内边距大小</span>

- 块级元素上表现完全正常
- <span style="color:red;">行级元素上表现完全不正常</span>: (一般没有给行级元素加背景色的需求)



<span style="color:red;">起码对于div来讲 → 宽度可以继承, 高度不可以继承</span>

![image-20210413203340978](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\HTML-CSS\HTML-CSS-basic-learning.assets\image-20210413203340978.png)

**盒子模型属性**

- `margin`: 外边距(参数方式上右下左)
- `padding`:内边距
- `border`:边框
- 内容

注意

- background = padding + 内容
- 高宽属性仅是'内容'高宽

行内元素的`margin`和`padding`属性

- 水平方向的padding-left,padding-right,margin-left,margin- right都产生边距效果
- 竖直方向的padding-top,padding-bottom,margin-top,margin-bottom不 会产生边距效果.



# 外边距合并现象

如果两个盒子模型, 他们的外边距, 在垂直方向上紧相邻, 那么就会产生一种现象, 叫外边距合并

父子之间 or 兄弟之间

![image-20210413204128082](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\HTML-CSS\HTML-CSS-basic-learning.assets\image-20210413204128082.png)

![image-20210413204132831](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\HTML-CSS\HTML-CSS-basic-learning.assets\image-20210413204132831.png)



# 行/块元素转换

对应属性

- 块级元素对应于`display:block`.
- 行内元素对应于`display:inline`.
- 行内块元素对应于`display: inline-block`属性

改变

- `display: block` 显示为块级元素
- `display: inline` 显示为内联元素
- `display: inline-block` 显示为内联块级元素，表现为同行显示并可修改宽高内外边距等属性





# float浮动

<span style="color:red;">标准流</span>：

- 块级元素纵向有序排列
- 行内块（行内）元素横向有序排列
- float被设计出来的初衷是用于:文字环绕效果

设置了浮动的元素，使元素脱离标准流（块级元素横向排列等）
浮动特性

1. 浮动只影响后面的元素
2. 连续浮动一行显示
3. 浮动以元素顶部为基准对齐
4. 浮动可是实现模式转换



clear: 谁受影响谁清除

- left:在左侧不允许浮动元素。
- right:在右侧不允许浮动元素。
- both:在左右侧不允许浮动元素。

e.g.

`.text { clear:both; }`



overflow

overflow 属性规定当内容溢出元素框时发生的事情
属性

- `visible`:默认值。内容不会被修剪，会呈现在元素框之外。
- <span style="color:red;">`hidden`</span>:内容会被修剪，并且其余内容是不可见的。
- `scroll`:内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。
- `auto`:如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。

注意:

1. `display:none;` 元素隐藏不占位置
2. `overflow:hiddenx;` 将超出部分的元素隐藏
3. `visibility:hidden;` 元素隐藏占位置

注意:

- 主要长宽单位
  - px 像素
  - %  相对于<span style="color:red;">父元素</span>的大小
- line-height(设置行高)
- 浏览器默认字体大小为:16px
- 浏览器默认行高：18px