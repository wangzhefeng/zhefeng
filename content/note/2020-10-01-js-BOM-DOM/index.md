---
title: BOM、DOM
author: 王哲峰
date: '2020-10-01'
slug: js-BOM-DOM
categories:
  - 前端
tags:
  - tool
---

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
h2 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}

details {
    border: 1px solid #aaa;
    border-radius: 4px;
    padding: .5em .5em 0;
}

summary {
    font-weight: bold;
    margin: -.5em -.5em 0;
    padding: .5em;
}

details[open] {
    padding: .5em;
}

details[open] summary {
    border-bottom: 1px solid #aaa;
    margin-bottom: .5em;
}
</style>

<details><summary>目录</summary><p>

- [BOM](#bom)
  - [JavaScript 浏览器简介](#javascript-浏览器简介)
  - [window](#window)
  - [location](#location)
  - [navigator](#navigator)
  - [screen](#screen)
  - [history](#history)
  - [document](#document)
- [客户端检测](#客户端检测)
  - [能力检测](#能力检测)
  - [用户代理检测](#用户代理检测)
  - [软件与硬件检测](#软件与硬件检测)
- [DOM](#dom)
  - [元素节点(element node)](#元素节点element-node)
  - [文本节点(text node)](#文本节点text-node)
  - [属性节点(attribute node)](#属性节点attribute-node)
  - [CSS](#css)
  - [获取元素](#获取元素)
  - [获取和设置属性](#获取和设置属性)
  - [DOM 其他属性和方法](#dom-其他属性和方法)
  - [DOM 扩展](#dom-扩展)
  - [DOM2 和 DOM3](#dom2-和-dom3)
- [Canvas](#canvas)
  - [Canvas 简介](#canvas-简介)
  - [绘制形状](#绘制形状)
  - [绘制文本](#绘制文本)
  - [其他](#其他)
</p></details><p></p>


# BOM

## JavaScript 浏览器简介

目前主流的浏览器分这么几种: 

- IE 6~11: 国内用得最多的 IE 浏览器, 历来对 W3C 标准支持差。从 IE10 开始支持 ES6 标准
- Chrome: Google 出品的基于 Webkit 内核浏览器, 内置了非常强悍的 JavaScript 引擎——V8。
  由于 Chrome 一经安装就时刻保持自升级, 所以不用管它的版本, 最新版早就支持 ES6 了
- Safari: Apple 的 Mac 系统自带的基于 Webkit 内核的浏览器, 从 OS X 10.7 Lion 自带的 6.1 版本开始支持 ES6, 
  目前最新的 OS X 10.11 El Capitan 自带的 Safari 版本是 9.x, 早已支持 ES6
- Firefox: Mozilla 自己研制的 Gecko 内核和 JavaScript 引擎 OdinMonkey。早期的 Firefox 按版本发布, 
  后来终于聪明地学习 Chrome 的做法进行自升级, 时刻保持最新
- 移动设备上目前 iOS 和 Android 两大阵营分别主要使用 Apple 的 Safari 和 Google 的 Chrome, 由于两者都是 Webkit 核心, 
  结果 HTML5 首先在手机上全面普及(桌面绝对是 Microsoft 拖了后腿), 对 JavaScript 的标准支持也很好, 最新版本均支持 ES6
- 其他浏览器如 Opera 等由于市场份额太小就被自动忽略了
- 另外还要注意识别各种国产浏览器, 如某某安全浏览器, 某某旋风浏览器, 它们只是做了一个壳, 其核心调用的是 IE, 
  也有号称同时支持 IE 和 Webkit 的“双核”浏览器

<div class="warning" style='background-color:#E9D8FD; color: #69337A; border-left: solid #805AD5 4px; border-radius: 4px; padding:0.7em;'>
    <span>
        <p style='margin-top:1em; text-align:left'>
            <b>Note</b>
        </p>
        <p style='margin-left:1em;'>
            不同的浏览器对 JavaScript 支持的差异主要是, 有些 API 的接口不一样, 比如 AJAX, File 接口。
            对于 ES6 标准, 不同的浏览器对各个特性支持也不一样<br><br>
            在编写 JavaScript 的时候, 就要充分考虑到浏览器的差异, 
            尽量让同一份 JavaScript 代码能运行在不同的 浏览器中
        </p>
        <p style='margin-bottom:1em; margin-right:1em; text-align:right; font-family:Georgia'> 
            <b></b> 
            <i></i>
        </p>
    </span>
</div>

JavaScript 可以获取浏览器提供的很多对象, 并进行操作:

* window
* navigator
* screen
* location
* document
* history

## window

- `window` 对象不但充当全局作用域, 而且表示浏览器窗口
- `window` 对象有 `innerWidth` 和 `innerHeight` 属性, 可以获取浏览器窗口的内部宽度和高度
  - 内部宽高是指除去菜单栏、工具栏、边框等占位元素后, 用于显示网页的净宽高

```js
// 调整浏览器窗口大小
'use strict';
console.log('window inner size: ' + window.innerWidth + ' x ' + window.innerHeight);
```

- `window` 还有一个 `outerWidth` 和 `outerHeight` 属性, 可以获取浏览器窗口的整个宽高
- 兼容性: IE <= 8 不支持
- window 对象的 open 方法来创建新的浏览器窗口

```js
window.open(url, name, features)
```

```js
// 调用 popUp 函数的一个办法是使用 伪协议(pseudo-protocol)
function popUp(winURL) {
    window.open(winURL, "popup", "width=320,height=480");
}
```

## location

- `location` 对象表示当前页面的 URL 信息
- 页面的 URL 可以通过 `location.href` 获取, 要获取 URL 各个部分的值: 

```js
location.protocol;   // 'http'
location.host;       // 'www.example.com'
location.port;       // '8080'
location.pathname;   // '/path/index.html'
location.search;     // '?a=1&b=2'
location.hash;       // 'TOP'
```

- 要加载一个新页面, 可以调用 `location.assign()` 方法
- 要加载当前页面, 调用 `location.reload()` 方法

```js
'use strict';
if (confirm('重新加载当前项' + location.href + '?')) {
   location.reload();
} else {
   location.assign('/'); // 设置一个新的 URL 地址
}
```

## navigator

   - `navigator` 对象表示浏览器的信息, 最常用的属性包括:

      - `navigator.appName`: 浏览器名称
      - `navigator.appVersion`: 浏览器版本
      - `navigator.language`: 浏览器设置的语言
      - `navigator.platform`: 操作系统类型
      - `navigator.userAgent`: 浏览器设定的 User-Agent 字符串

   - navigator 的信息可以很容易地被用户修改, 所以 JavaScript 读取的值不一定是正确的, 
     所以可以充分利用 JavaScript 对不存在属性返回 `undefined` 的特性, 直接用短路运算符 `||` 计算

```js
var width = window.innerWidth || document.body.clientWidth;
```

示例:

```js
console.log('appName = ' + navigator.appName);
console.log('appVersion = ' + navigator.appVersion);
console.log('language = ' + navigator.language);
console.log('platform = ' + navigator.platform);
console.log('userAgent = ' + navigator.userAgent);
```

## screen

- `screen` 对象表示屏幕的信息, 常用的属性有:

   - screen.width: 屏幕宽度, 以像素为单位
   - screen.height: 屏幕高度, 以像素为单位
   - screen.colorDepth: 返回颜色位数, 如: 8、16、24

示例:

```js
'use strict';
console.log('Screen size = ' + screen.width + ' x ' + screen.height);
```

## history

- `history` 对象保存了浏览器的历史记录, JavaScript 可以调用 `history` 对象的 `back()` 或 `forward()`, 
   相当于用户点击了浏览器的后腿或前进按钮
- `history` 对象属于历史遗留对象, 对于现代 Web 页面来说, 由于大量使用 AJAX 和页面交互, 
   简单粗暴地调用 `history.back()` 可能会让用户感到非常愤怒
- 任何情况下, 都不应该使用 `history` 对象


## document


- `document` 对象表示当前页面, 由于 HTML 在浏览器中以 DOM 形式表示树形结构的, 
  `document` 对象就是整个 DOM 树的根节点
- `document` 的 `title` 属性是从 HTML 文档中的 `<title>xxx</title>` 读取的, 但是可以动态改变

```js
'use strict';
document.title = "努力学习 JavaScript!"
```

- 要查找 DOM 树的某个节点, 需要从 `document` 对象开始查找, 最常见的查找是 ID 和 Tag Name

```html
<dl id="drink-menu" style="border:solid 1px #ccc;padding:6px;">
   <dt>摩卡</dt>
   <dd>热摩卡咖啡</dd>
   <dt>酸奶</dt>
   <dd>北京酸奶</dd>
   <dt>果汁</dt>
   <dd>鲜榨苹果汁</dd>
</dl>
```

```js
'use strict';
var menu = document.getElementById('drink-menu');
var drinks = document.getElementsByTagName('dt');
var i, s;
s = '提供的饮料有:';
for (i=0; i<drinks.length; i++) {
   s = s + drinks[i].innerHTML + ',';
}
```

- `document` 对象还有一个 `cookie` 属性, 可以获取当前页面的 Cookie
   - Cookie 是由服务器发送的 key-value 标示符。因为 HTTP 协议是无状态的, 
      但是服务器要区分到底是哪个用户发过来的请求, 就可以用 Cookie 来区分。
      当一个用户成功登录后, 服务器发送一个 Cookie 给浏览器, 例如 `user=ABC123XYZ(加密的字符串)...`, 
      此后, 浏览器访问该网站时, 会在请求头附上这个 Cookie, 服务器根据 Cookie 即可区分出用户
   - Cookie 还可以存储网站的一些设置, 例如, 页面显示的语言等等
   - JavaScript 可以通过 `doucment.cookie` 读取到当前页面的 Cookie

```js
document.cookie;
```

<div class="warning" style='background-color:#E9D8FD; color: #69337A; border-left: solid #805AD5 4px; border-radius: 4px; padding:0.7em;'>
    <span>
        <p style='margin-top:1em; text-align:left'>
            <b>Note</b>
        </p>
        <p style='margin-left:1em;'>
            由于 JavaScript 能读取到页面的 Cookie, 而用户的登录信息通常也存在 Cookie 中, 
            这就造成了巨大的安全隐患, 这是因为在 HTML 页面中引入第三方的 JavaScript 代码是允许的: 
        </p>
        <p style='margin-bottom:1em; margin-right:1em; text-align:right; font-family:Georgia'> 
            <b></b> 
            <i></i>
        </p>
    </span>
</div>


```html
<!-- 当前页面在wwwexample.com -->
<html>
   <head>
      <script src="http://www.foo.com/jquery.js"></script>
   </head>
   ...
</html>
```

- 如果引入的第三方的 JavaScript 中存在恶意代码, 则 www.foo.com 网站将直接获取到 www.example.com 网站的用户登录信息
- 为了解决这个问题, 服务器在设置 Cookie 时可以使用 `httpOnly`, 设定了 `httpOnly` 的 Cookie 将不能被 JavaScript 读取。
   这个行为由浏览器实现, 主流浏览器均支持 `httpOnly` 选项, IE 从 IE6 SP1 开始支持
- 为了确保安全, 服务器端在设置 Cookie 时, 应该始终坚持使用 `httpOnly`

# 客户端检测

## 能力检测


## 用户代理检测

## 软件与硬件检测


# DOM

如果没有文档(document), DOM 也就无从谈起。
当创建一个网页并把它加载到 Web 浏览器中时, 
DOM 就在幕后悄然而生。它把编写的网页文档转换为一个文档对象。

对象是一种自足的数据集合, 与某个特定对象相关联的变量被称为这个对象的属性；
只能通过某个特定对象去调用的函数被称为这个对象的方法。

   - D: document
   - O: object(host object)
      
      - window object、BOM、Window Object Model
      - document object 

   - M: Model

      - DOM 代表着加载到浏览器窗口的当前网页
      - DOM 把一份文档(document)表示为一棵树, 更具体地说, DOM 把文档表示为一棵节点树

.. note:: JavaScript 对象类型: 

   - 用户自定义对象(user-defined object) 由程序员自行创建的对象
   - 内建对象(native object) 内建在 JavaScript 语言里的对象, 如 Array、Math 和 Date
   - 宿主对象(host object) 由浏览器提供的对象


**node--节点**

文档是由节点构成的集合, 在 DOM 里有许多不同类型的节点: 

   - 元素节点(element node)
   - 文本节点(text node)
   - 属性节点(attribute node)

## 元素节点(element node)

   - DOM 的原子是元素节点, 元素节点在文档中的布局形成了文档的结构, 标签的名字就是元素的名字
   - 元素可以包含其他元素

## 文本节点(text node)

   - 在 XHTML 文档里, 文本节点总是被包含在元素节点的内部, 但并非所有的元素节点都包含有文本节点。

## 属性节点(attribute node)

   - 属性节点用来对元素做出更加具体的描述
   - 属性总是被放在起始标签里, 所以属性节点总是被包含在元素节点中
   - 并非所有的元素都包含着属性, 但所有的属性都被元素包含

## CSS

   - 除了DOM与网页结构打交道外, 还可以通过 CSS 告诉浏览器应该如何显示一份文档的内容
   - CSS 语法(类似于 JavaScript 函数的定义)

```css
selector {
   property: value;
}
```

   -  继承(inheritance) 是 CSS 中的一项强大的功能, 类似于 DOM, 
      CSS 也把文档的内容视为一颗节点树。节点树上的各个元素将继承其父元素的样式属性。
      在某些场合, 档把样式应用于一份文档时, 其实只是想让那些样式作用于特定的元素。
      为了获得如此精细的控制, 需要在文档里插入一些能够把这段文本与其他段落区别开来的特殊标志。

      1. class 属性

         - 可以在所有元素上任意应用 class 属性
         - 在 CSS 里, 可以为 class 属性值相同的所有元素定义同一种样式, 也可以为 class 属性为一种特定类型的元素定义一种特定的样式

```html
<p class="sepcial">This paragraph has the special class</p>
<h2 class="special">So dose this headline</h2>
```

```css
.special {
   font-style: italic;
}

h2.special {
   text-transform: uppercase;
}
```

      2. id 属性

         - id 属性的用途是给网页里的某个元素加上一个独一无二的标识符
         - 在 CSS 里, 可以为特定 id 属性的元素定义一种独享的样式

```html
<ul id="pruchase">
   <li>A tin of beans</li>
   <li class="sale">Cheese</li>
   <li class="sale important">Milk</li>
</ul>
```

```css
#pruchase {
   border: 1px solid white;
   background-color: #333;
   color: #ccc;
   padding: 1em;
}
```

- 尽管 id 本身只能使用一次, CSS 还是可以利用 id 属性为包含该特定元素里的其他元素定义样式

```css
#pruchase li {
   font-weight: blod;
}
```

## 获取元素

有三种 DOM 方法可以获取元素节点, 分别是通过元素 id、标签名字、class名字

1. `document.getElementById(idName)`
   - 这个调用将返回一个对象, 这个对象对应着 document 对象里的一个独一无二的元素, 可以使用 typeof 操作符来验证这一点
   - 事实上, document 中的每一个元素都是一个对象, 利用 DOM 提供的方法能得到任何一个对象
2. `document.getElementByTagName(tagName)`
   - 一般来说, 用不着为文档里的每一个元素都定义一个独一无二的 id 值, DOM 提供了一个方法来获取那些没有 id 属性的对象
   - getElementByTagName 方法返回一个对象数组, 每个对象分别对应着文档里有着给定标签的一个元素

```js
document.getElementByTagName("li");
```

3. `document.getElementByClass(className)`
   - 这个方法的返回值与 getElementByTagName 类似, 都是一个具有相同类名的元素的数组
   - 使用这个方法还可以查找那些带有多个 className 的元素, 要指定多个类名, 只要在字符串参数中用空格分隔列名即可

## 获取和设置属性

得到需要的元素后, 可以设法获取它的各个属性, `getArrtibute` 和 `setAttribute` 方法可以分别获取属性和更改属性节点的值

1. `getAttribute`
   - 语法: getAttribute 是一个函数, 它只有一个参数, 就是查询的属性的名字: 

```js
object.getAttribute(attribute)
```

   - 与此前介绍的方法不同, getAttribute 方法不属于 document 对象, 所以不能通过 document 对象调用, 它只能通过元素节点对象调用

2. `setAttribute`
   - 语法: setAttribute 允许对属性节点的值做出修改, 与 getAttribute 一样, setAttribute 也只能用于元素节点

```js
object.setAttribute(attribute, value)
```

-  通过 setAttribute 对文档做出修改之后, 在通过浏览器的查看源代码选项去查看文档的源代码时看到的仍将是改变前的属性值, 
  也就是说, setAttribute 做出的修改不会反映在文档本身的源代码里。这种表里不一的现象源自于 DOM 的工作模式, 
  先加载文档的静态内容, 再动态刷新, 动态刷新不影响文档的静态内容。这正是 DOM 的真正威力: 对页面内容进行刷新却不需要在浏览器里刷新页面

## DOM 其他属性和方法

- nodeName
- nodeValue
- childNodes
- nextSibling
- parentNode


## DOM 扩展



## DOM2 和 DOM3




# Canvas

   Canvas 是 HTML5 新增的组件, 它就像一块幕布, 可以用 JavaScript 在上面绘制各种图表、动画等。
   在没有 Canvas 的年代, 绘图只能借助 Flash 插件实现, 页面不得不用 JavaScript 和 Flash 进行交互。
   有了 Canvas, 我们就再也不需要 Flash 了, 直接使用 JavaScript 完成绘制。

## Canvas 简介

-  一个 Canvas 定义了一个指定尺寸的矩形框, 在这个范围内我们可以随意绘制: 

```html
<canvas id="test-canvas" width="300" height="200"></canvas>
```

   -  由于浏览器对 HTML5 标准支持不一致, 所以, 通常在 `<canvas>` 内部添加一些说明性 HTML 代码, 
      如果浏览器支持 Canvas, 它将忽略 `<canvas>` 内部的 HTML, 如果浏览器不支持 Canvas, 
      它将显示 `<canvas>` 内部的 HTML: 

```html
<canvas id="test-stock" width="300" height="200">
   <p>Current Price: 25.51</p>
</canvas>
```

   - 在使用 Canvas 前, 用 `canvas.getContext` 来测试浏览器是否支持 Canvas:

```html
<canvas id="test-canvas" width="200" height="100">
<p>你的浏览器不支持 Canvas</p>
</canvas>
```

```js

'use strict';
var canvas = document.getElementById("test-canvas");
if (canvas.getContext) {
console.log("你的浏览器支持 Canvas");
} else {
console.log("你的浏览器不支持 Canvas!");
}
```

   - `getContext("2d")` 方法让我们拿到一个 `CanvasRenderingContext2D` 对象, 所有的绘图操作都需要通过这个对象完成: 

```html
<canvas id="test-canvas" width="300" height="200"></canvas>
```
```js

var canvas = document.getElementById("test-canvas");
var ctx = canvas.getContext("2d");
```

   - 如果需要绘制 3D, HTML5 还有一个  WebGL 规范, 允许在 Canvas 中绘制 3D 图形: 

```html
<canvas id="test-canvas" width="300" height="200"></canvas>
```

```js
var canvas = document.getElementById("test-canvas");
var gl = canvas.getContext("webgl");
```

## 绘制形状

- 可以在 Canvas 上绘制各种形状, 在绘制前, 需要先了解一下 Canvas 的坐标系统
   - Canvas 的坐标以左上角为原点, 水平向右为 X 轴, 垂直向下为 Y 轴, 以像素为单位, 所以每个点都是非负整数
   ![img](images/canvas_shape.png)
- `CanvasRenderingContext2D` 对象有若干方法来绘制图形: 

```html
<canvas id="test-shape-canvas" width="300" height="200"></canvas>
```

```js
'use strict';
var canvas = document.getElementById("test-shape-canvas");
var ctx = canvas.getContext("2d");

// 擦除 (0, 0) 位置大小为 200x200 的矩形, 擦除的意思是把该区域变为透明
ctx.clearRect(0, 0, 200, 200);

// 设置颜色
ctx.fillStyle = "#dddddd";

// 把 (10, 10) 位置大小为 130x130 的矩形涂色
ctx.fillRect(10, 10, 130, 130);

// 利用 Path 绘制复杂路径
var path = new Path2D();
path.arc(75, 75, 50, 0, Math.PI * 2, true);
path.moveTo(110, 75);

path.arc(75, 75, 35, 0, Math.PI, false);
path.moveTo(65, 65);

path.arc(60, 65, 5, 0, Math.PI * 2, true);
path.moveTo(95, 65);

path.arc(90, 65, 5, 0, Math.PI * 2, true);
ctx.strokeStyle = "#0000ff";
ctx.stroke(path);
```

## 绘制文本

绘制文本就是在指定的位置输出文本, 可以设置文本字体、样式、阴影等, 与 CSS 完全一致: 

```html
<canvas id="test-text-canvas" width="300" height="200"></canvas>
```

```js
'use strict';
var canvas = document.getElementById("test-text-canvas");
ctx.canvas.getContext("2d");

ctx.clearRect(0, 0, canvas.width, canvas.height);
ctx.shadowOffsetX = 2;
ctx.shadowOffsetY = 2;
ctx.shadowBlur = 2;
ctx.shadowColor = "#666666";
ctx.font = "24px Arial";
ctx.fillStyle = "#333333";
ctx.fillText("带阴影的文字", 20, 40);
```

## 其他

Canvas 除了绘制基本的形状和文本, 还可以实现动画、缩放、各种滤镜和像素转换等高级操作。
如果要实现非常复杂的操作, 考虑以下优化方案: 

- 通过创建一个不可见的 Canvas 来绘图, 然后将最终绘制结果复制到页面的可见 Canvas 中
- 尽量使用整数坐标而不是浮点数
- 可以创建多个重叠的 Canvas 绘制不同的层, 而不是在一个 Canvas 中绘制非常复杂的图
- 背景图片如果不变可以直接用 `<img>` 标签并放到最底层