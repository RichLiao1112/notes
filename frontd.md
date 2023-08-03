### https和http
HTTPS 是支持加密和验证的 HTTP。两种协议的唯一区别是HTTPS 使用 TLS (SSL) 来加密普通的 HTTP 请求和响应，并对这些请求和响应进行数字签名。因此，HTTPS 比 HTTP 安全得多。使用 HTTP 的网站的 URL 中带有 http://，而使用 HTTPS 的网站则带有 https://。

正常请求看上去这样的：
GET /hello.txt HTTP/1.1
User-Agent: curl/7.63.0 libcurl/7.63.0 OpenSSL/1.1.l zlib/1.2.11
Host: www.example.com
Accept-Language: en

https类似这样的：
t8Fw6T8UV81pQfyhDkhebbz7+oiwldr1j2gHBB3L3RFTRsQCpaSnSBZ78Vme+DpDVJPvZdZUZHpzbbcqmSW1+3xXGsERHg9YDmpYk0VVDiRvw1H5miNieJeJ/FNUjgH0BmVRWII6+T4MnDwmCMZUI/orxP3HGwYCSIvyzS3MpmmSe4iaWKCOHQ==

TLS/SSl加密http请求
TLS 使用一种称为公钥加密的技术：密钥有两个，即公钥和私钥，其中公钥通过服务器的 SSL 证书与客户端设备共享。当客户端打开与服务器的连接时，这两个设备使用公钥和私钥商定新的密钥（称为会话密钥），以加密它们之间的后续通信。

然后，所有 HTTP 请求和响应都使用这些会话密钥进行加密），使任何截获通信的人都只能看到随机字符串，而不是明文。


**整体流程**

-- 证书验证阶段 --
1. 客户端请求服务器
2. 服务器响应时携带ca证书公钥A（服务端存有ca证书包含的公钥和私钥）

-- 非对称加密阶段 --
3. 客户端解析证书，验证合法性（若验证不合法，则提示https警告，由用户选择是否继续访问）
4. 客户端取出公钥A并生成随机码 KEY，使用公钥A加密随机码KEY
5. 客户端把加密后的随机码 KEY 发送给服务器，作为接下来对称加密的密钥
6. 服务端使用私钥B解密，获取真实的随机码 KEY
7. 服务端使用随机码 KEY 对传输数据进行对称加密，响应加密后的内容

-- 对称加密阶段 --
8. 客户端使用之前生成的随机码 KEY 解密数据
9. 后续请求都通过随机码 KEY 传输内容

**拓展**
客户端如何验证公钥的合法性？
1. 浏览器和操作系统内置了信任的根证书（CA组织下人格的根证书Root签发的）
2. 客户端如当发现当前证书的签发者是A，不是根证书，那就继续找签发者A的签发机构B，直到根证书匹配。最终信任当前证书。


### [TCP](https://juejin.cn/post/7028003193502040072)

TCP是面向连接的协议，它基于运输连接来传送TCP报文段，TCP运输连接的建立和释放，是每一次面向连接的通信中必不可少的过程。

**TCP连接的3个阶段**
1. 建立TCP连接，也就是通过三报文握手来建立TCP连接
2. 数据传送，也就是基于已建立的TCP连接进行可靠的数据传输
3. 释放连接，也就是在数据传输结束后，还要通过四报文挥手来释放TCP连接

TCP的连接建立主要是解决以下三个问题
1. 使TCP双方能够确知对方的存在
2. 使TCP双方能够协商一些参数（ 最大窗口值是否使用窗口扩大选项和时间戳选项，以及服务质量等）
3. 使TCP双方能够对运输实体资源（例如缓存大小连接表中的项目等）进行分配


### 浏览器进程与线程
浏览器是多线程的，实际是浏览器提供了执行异步任务的能力。
js是单线程的，指执行js代码的线程只有一个，是浏览器提供的js引擎线程（主线程）。
除了js引擎线程（主线程）之外，浏览器中还要定时器线程，http请求线程等。

当主线程中需要发送数据请求时，就会把这个任务交给http请求线程执行。当请求的数据返回之后，再将callback里要执行的js回调交给js引擎线程执行。
浏览器才是真正执行发送请求这个任务的角色，而 JS 只是负责执行最后的回调处理。
这里的异步不是js自身实现的，而是浏览器提供的能力

chrome架构：
- Renderer进程（内核）
  * GUI渲染线程
  * JS引擎线程
  * 事件触发线程
  * 定时器触发线程
  * 异步http请求线程
  这些线程为 JS 在浏览器中完成异步任务提供了基础
- 第三方插件进程
- cpu进程
- 浏览器进程

### js事件循环

**js任务**

同步任务：在主线程上排队执行的任务，只有一个任务执行完毕，才能执行下一个任务
异步任务：不进入主线程，而是放在任务队列中，若有多个异步任务则需要在任务队列中排队等待，任务队列类似于缓冲区，任务下一步会被移到执行栈然后主线程执行调用栈的任务

**执行栈和任务队列**

执行栈：
  - 执行栈使用到的是数据结构中的栈结构， 它是一个存储函数调用的栈结构，遵循**先进后出**的原则
  - 它主要负责跟踪所有要执行的代码

  （参数数组中的左进左出：A函数调用了B函数，A先进执行栈，B后进执行栈（unshift）。B先pop出并执行，再pop出A执行）
  
任务队列：
  - 用于保存异步任务，遵循**先进先出**原则
  - 主要负责将新的任务发送到任务队列进行处理
  - 根据任务种类不同，分为微任务队列和宏任务队列

**js执行过程：**
  - 先将将代码推入执行栈依次执行
  - 当遇到异步任务时，将异步任务放入任务队列中，先挂起等待同步任务执行完。再将异步任务交由浏览器的其他线程执行（如定时器触发线程执行**定时器任务**，异步http请求线程**执行http请求任务**等）
  - 当执行栈中所有同步代码执行完成之后，会不断读取任务队列，从中取出已完成的异步任务的**回调代码**放入执行栈中继续执行

**在事件驱动的模式下，至少包含了一个执行循环来检测任务队列中是否有新任务。通过不断循环，去取出异步任务的回调来执行，这个过程就是事件循环，每一次循环就是一个事件周期。**


### [宏任务和微任务](https://juejin.cn/post/6992167223523541023)
  根据任务种类不同，分为微任务队列和宏任务队列

  - 宏任务队列：script(整体代码)、setTimeout、setInterval、I/O、UI 交互事件、setImmediate(Node.js 环境)
  - 微任务队列：Promise.then、MutaionObserver、process.nextTick(Node.js 环境)

---

1. JavaScript 引擎首先从宏任务队列中取出第一个任务，执行
2. 第一个宏任务执行完毕后，再将微任务中的所有任务取出，按照顺序分别全部执行（**执行微任务过程中产生的新的微任务并不会推迟到下一个循环中执行，而是在当前的循环中继续执行**）



### js数据类型
Undefined、Null、Boolean、Number、String、Object、Symbol、BigInt

在操作系统中，内存被分为栈区和堆区：
- 栈：存储原始数据类型（Undefined、Null、Boolean、Number、String）。引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。栈中数据的存取方式为先进后出。
- 堆：存储引用数据类型（Object、Symbol、函数）。堆是一个优先队列，是按优先级来进行排序的，优先级可以按照大小来规定


### 原型链

- 什么是原型链？
```md
原型链是一种通过对象的原型属性来**实现对象之间继承关系的机制**。每个对象都有一个原型对象，它可以从原型对象继承属性和方法。
```

- 如何创建一个继承自另一个对象的对象？
```javascript
var parent = { name: "Parent" };
var child = Object.create(parent);
console.log(child.name); // "Parent"
```

- 如何判断一个对象是否是另一个对象的实例？
```javascript
function Person() {}
var p = new Person();
console.log(p instanceof Person); // true
```

- 如何判断一个属性是自身属性还是继承自原型？
可以使用hasOwnProperty()方法来判断一个属性是否是自身属性。例如：
```javascript
function Person() {}
Person.prototype.name = "Person";
var p = new Person();
console.log(p.hasOwnProperty("name")); // false
console.log(Person.prototype.hasOwnProperty("name")); // true
```

- 如何遍历一个对象的所有属性，包括继承自原型的属性？
可以使用for...in循环来遍历一个对象的所有可枚举属性，包括继承自原型的属性。需要注意的是，需要使用hasOwnProperty()方法来判断一个属性是否是自身属性。例如：
```javascript
function Person() {}
Person.prototype.name = "Person";
var p = new Person();
for (var prop in p) {
  if (p.hasOwnProperty(prop)) {
    console.log(prop + ": " + p[prop]);
  }
}
// 输出：name: undefined
```
(for...of不能用于遍历对象)


### 模块化

**传统模块化**

通过```<script src="./index.js" />```标签引用模块，产生问题：
- 污染全局作用域
- 命名冲突
- 无法管理模块之前的依赖


**AMD\CMD**

CMD和AMD都是JavaScript模块化规范，它们的主要区别在于模块定义和加载方式不同。具体来说：

CMD（Common Module Definition）采用**同步方式**加载模块，即只有当需要使用某个模块时才会加载该模块，因此CMD模块可以更精细地控制模块的加载顺序。常见的CMD库有SeaJS。常用于Node服务端。
AMD（Asynchronous Module Definition）采用异步方式加载模块，即定义模块时可以指定其依赖的模块，当所有依赖的模块加载完成后再执行当前模块。常见的AMD库有RequireJS。用于浏览器端。

CMD: 
- 一个文件就是一个模块
- 每个模块都有单独的作用域
- 通过module.exports、exports.xxx='mmm'导出成员
- 通过require函数载入模块


AMD:
Require.js实现了AMD规范

定义模块
```define('模块名', ['依赖项1', '依赖项2'], funsion(){ return {} // 导出当前声明的模块对象 })```

载入模块
```require(['模块1'], function(模块1) { 模块1.start() })``` 会创建script标签以导模块

定义、载入模块较为繁琐
大量的amd模块会导致引用大量的文件，性能受影响


**ES Module**
```<script type="module" />```
```<script nomodule />```
特性：
- 自动采用严格模式
- 每个esm都运行在单独的私有作用域中，防止了全局污染问题
- esm是通过cors的方式请求外部js模块，需要开发者处理跨域
- esm的script标签会自动延迟执行脚本

import\export:

导出的是对象的引用，运行时修改模块内的值，导入处会同步改动
导入的对象是只读的值，无法修改

```javascript
export const m = {
  xxx
};
export default xxx;
import { m } from './xxx.js'
import m from './xxx.js'
import * as m from './xxx.js'

if (true) {
  import('./module.js').then(res => console.log(res));
}
```
polyfill
babel-browser-build
browser-es-module-loader
将es6编译成es5适配ie

node中
es module 中可以导入commonjs模块
commonjs中不可以导入es module模块
commonjs始终只会导出一个默认成员
import不是解构

### babel
通过插件编译转化代码


### 打包工具

**webpack**
webpack是一款现代化的打包工具，它可以将多个模块打包成一个或多个bundle文件。webpack的打包过程可以简单概括为以下几个步骤：

读取入口文件(entry)：webpack会从指定的入口文件开始分析整个项目的依赖关系。

分析依赖：webpack会递归地分析入口文件引用的所有模块，并将它们组织成一个依赖关系图。

编译模块：webpack会将每个模块转换成浏览器可执行的代码，这个过程中可能会使用各种loader来处理不同类型的文件。

合并代码：webpack会将所有编译后的模块合并成一个或多个bundle文件，这个过程中可能会使用各种插件来优化代码。

输出文件(output)：webpack会将打包后的文件输出到指定的目录中，这个过程中可以配置各种输出选项。

总之，webpack通过分析依赖关系、编译模块、合并代码和输出文件等步骤，将多个模块打包成一个或多个bundle文件，从而实现了高效的模块化开发和部署。

**webpack模块加载原理**
webpack遵循es module标准的import声明
遵循commonjs标准的require函数
遵循amd标准的define函数和require函数
样式代码中的@import指令和url函数
html代码中的图片标签src属性



**loader 模块化核心** 专注实现模块化资源加载，通过不同的module处理不同的前端资源(css/js/html)，
- 编译转换类
- 文件操作类
- 代码检查类

源码 => loader1 => loader1结果 => loader2 => => loader2结果 => loader3 => loader3结果 => ... => 最终结果（必须为js）

js驱动整个前端应用
```javascript
// webpack module loader
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: './dist',
    publicPath: 'dist/'
  },
  module: {
    rules: [
      {
        test: ./js$/,
        use: {
          loader: 'babel-loader', // 加载器
          options: {
            presets: ['@bable/preset-env'], // es新特性编译插件集合
          }
        }
      },
      {
        test: /.css$/,
        use: [
          // use 数组内的组件从末位优先执行
          'style-loader',
          'css-loader', // 优先于style-loader执行。
        ]
      },
      {
        test: /.png$/,
        // use: [
        //   'file-loader'
        // ],
        use: {
          // file-loader使publicPath和hash文件名路径访问
          // url-loader 使png转为 data:image/png;xxxxxx 适用为体积小的资源
          loader: 'url-loader',
          options: {
            limit: 10 * 1024 // 小于10kb的转为 data url
          }
        }
      },
      {
        test: /.html$/,
        use: {
          loader: 'html-loader',
          options: {
            attrs: ['img:src', 'a:href'],
          }
        }
      }
    ]
  }
}
```

**plugin** 解决资源加载资源以外的自动化工作

插件机制
通过勾子机制实现
https://www.webpackjs.com/api/compiler-hooks/
```javascript
class MyPlugin {
  apply(complier) {
    // webpack启动时自动启动

    complier.hooks.emit.tap() // emit勾子
  }
}
```

**webpack watch**
webpack --watch 修改代码后自动编译

配合brower-sync监听dist目录自动刷新
磁盘写、读 效率较低

**webpack dev server**
安装webpack-dev-server依赖
执行webpack-dev-server启动应用即可自动编译 刷新页面。打包结果在内存中，没有保存在磁盘里
无效写读磁盘，效率较高

**hot module replace 热替换**
实时替换模块，页面不刷新，保留页面状态
已集成至webpack-dev-server --hot

**source map**
记录js压缩打包前的变量和打包后的变量关联关系，易于调试错误。

webpack.config 设置变量开启 devtool: 'source-map' // devtool有多种source map模式，分别对应打包速度、生产环境是否能用等等

**tree shaking**
生产模式打包会自动删除冗余代码

### 箭头函数
箭头函数是ES6中新增的一种函数定义方式，相对于传统的函数定义方式，它具有以下优缺点：

优点：

更简洁的语法：箭头函数可以用更简洁的语法来定义函数，省略了function关键字和大括号，使代码更加简洁易读。
简化this指向：箭头函数的this指向是在定义时确定的，而不是在运行时确定的，这样可以避免this指向错误的问题。

缺点：

不能作为构造函数使用：箭头函数没有自己的this，也没有prototype属性，因此不能作为构造函数使用。
不能使用arguments对象：箭头函数没有自己的arguments对象，只能使用外部函数的arguments对象。


### this指向

在JavaScript中，this关键字用于指向当前执行环境的上下文对象。它的指向是在函数执行时动态确定的，具体取决于函数被调用时的方式。

下面是几种常见的this指向情况：

全局作用域：在全局作用域中，this指向全局对象（浏览器中为window对象）。

函数中的this：在函数中，this的指向取决于函数的调用方式。

作为普通函数调用时，this指向全局对象（非严格模式）或undefined（严格模式）。
作为对象的方法调用时，this指向该对象。
作为构造函数调用时，this指向新创建的实例对象。
使用apply()或call()方法调用时，this指向传入的第一个参数。
箭头函数中的this：在箭头函数中，this指向定义时所在的作用域中的this，而不是调用时的this。

```javascript
// 全局作用域中的this
console.log(this); // window

// 函数中的this
function foo() {
  console.log(this);
}
foo(); // window

const obj = {
  name: 'Alice',
  sayName() {
    console.log(this.name);
  }
};
obj.sayName(); // Alice

function Person(name) {
  this.name = name;
}
const p = new Person('Bob');
console.log(p.name); // Bob

function bar() {
  console.log(this.name);
}
const obj2 = { name: 'Charlie' };
bar.call(obj2); // Charlie

// 箭头函数中的this
const obj3 = {
  name: 'David',
  sayName: () => {
    console.log(this.name);
  }
};
obj3.sayName(); // undefined

const obj = {
  name: 'Alice',
  sayName: function() {
    const arrowFunc = () => {
      console.log(this.name);
    };
    arrowFunc();
  }
};
obj.sayName(); // Alice
```



### defer和async

都是异步加载，都不会阻塞HTML文档的解析和渲染

```html
<script defer src="script1.js"></script>
defer

脚本的加载不会阻塞HTML文档的解析和渲染。
脚本的执行顺序与它们在HTML文档中的顺序一致。
脚本的执行时机是在文档解析完成后，DOMContentLoaded事件触发前。


async
脚本的执行时机是在它被下载完成后立即执行
多个带有async属性的脚本的执行顺序是不确定的，取决于它们下载完成的顺序

```


### 严格模式

严格模式是ES5引入的，不属于ES6，ES6自动采用严格模式

- 禁止使用未声明的变量：在严格模式下，如果使用未声明的变量，会抛出一个ReferenceError错误。
- 禁止删除变量或函数：在严格模式下，如果使用delete操作符删除变量或函数，会抛出一个SyntaxError错误。
- 禁止使用八进制字面量：在严格模式下，如果使用八进制字面量，会抛出一个SyntaxError错误。
- 禁止使用with语句：在严格模式下，如果使用with语句，会抛出一个SyntaxError错误。
- 函数中的this值：在严格模式下，函数中的this值为undefined，而不是全局对象。
- 禁止在函数内部重新声明参数：在严格模式下，如果在函数内部重新声明参数，会抛出一个SyntaxError错误。
- 禁止使用eval和arguments作为变量名：在严格模式下，如果使用eval或arguments作为变量名，会抛出一个SyntaxError错误。
- 限制了eval的作用域：在严格模式下，eval函数中的代码不能访问调用它的函数的变量和函数。


### 常见兼容性问题

- 不同浏览器的标签默认的margin和padding不同； 通过设置css解决
```css
* { margin: 0; padding: 0; }
```
- 某些浏览器图片有默认间距，通过设置```font-size: 0```可解决
- 标签最低高度设置min-height不兼容问题；如果我们要设置一个标签的最小高度200px，需要进行的设置为：
```css
{ min-height:200px; height:auto !important; height:200px; overflow:visible };
```
- 图片加a标签在IE9中会有边框；```img{ border: none; }```可解决
- 在Chrome中字体不能小于10px；使用scale解决```p{font-size: 12px; transform: scale(0.8);}```
- 防抖和节流
```javascript

// 防抖
function debounce(fn: () => void, delay: number) {
    let timer: number;
    return function() {
        if (timer) clearTimeout(timer);
        let args = arguments;
        timer = setTimeout(() => {
            fn.apply(this, args);
        }, delay)
    }
}


// 节流
function throttle(fn: () => void, delay: number) {
    let timer: number;

    return function() {
        if (timer) return;
        let args = arguments;
        timer = setTimeout(() => {
            fn.apply(this, args);
        }, delay);
    }
}
```

### 高阶函数（HOF）

- 什么是高阶函数

可以接受函数做为参数或者函数作为返回值的函数。

### 高阶组件（HOC）

- 什么是高阶组件

可以接受组件做为参数或者函数作为返回值的函数。


高阶函数、组件常用于逻辑、组件复用

```jsx

function Auth(Comp) {
  return class extends React.Component {
    render() {
      return <Comp />
    }
  }
}

```

### React 事件

React根据w3c规范来定义自己的事件系统，其事件被称之为合成事件 (SyntheticEvent)。而其自定义事件系统的动机主要包含以下几个方面：

- 抹平不同浏览器之间的兼容性差异。最主要的动机
- 提供一个抽象的跨平台事件机制
- 所有事件的触发都代理到了 document，而不是 DOM 节点本身。document.addEventListener
- 可以干预事件的分发。React.16引入 **Fiber** 架构，可以通过干预事件的分发以优化用户的交互体验。

React 为了在触发事件时可以查找到对应的回调去执行，会把组件内的所有事件统一地存放到一个对象中（listenerBank）。

首先会根据事件类型分类存储，例如 click 事件相关的统一存储在一个对象中，回调函数的存储采用键值对（key/value）的方式存储在对象中，key 是组件的唯一标识 id，value 对应的就是事件的回调函数。

React事件分发也就是事件触发。React 的事件触发只会发生在 DOM 事件流的冒泡阶段，因为在 document 上注册时就默认是在冒泡阶段被触发执行。

1. 触发事件，开始 DOM 事件流，先后经过三个阶段：事件捕获阶段、处于目标阶段和事件冒泡阶段
2. 当事件冒泡到 document 时，触发统一的事件分发函数 ReactEventListener.dispatchEvent
3. 根据原生事件对象（nativeEvent）找到当前节点（即事件触发节点）对应的 ReactDOMComponent 对象
4. 事件的合成
    - 根据当前事件类型生成对应的合成对象
    - 封装原生事件对象和冒泡机制
    - 查找当前元素以及它所有父级
    - 在 listenerBank 中查找事件回调函数并合成到 events 中
5. 批量执行合成事件（events）内的回调函数
6. 如果没有阻止冒泡，会将继续进行 DOM 事件流的冒泡（从 document 到 window），否则结束事件触发

**React 合成事件和原生 DOM 事件的主要区别**
- React 组件上声明的事件没有绑定在 React 组件对应的原生 DOM 节点上
- React 利用事件委托机制，将几乎所有事件的触发代理（delegate）在 document 节点上，事件对象(event)是合成对象(SyntheticEvent)，不是原生事件对象，但通过 nativeEvent 属性访问原生事件对象。
- 由于 React 的事件委托机制，React 组件对应的原生 DOM 节点上的事件触发时机总是在 React 组件上的事件之前。

原生事件（阻止冒泡）会阻止合成事件的执行
合成事件（阻止冒泡）不会阻止原生事件的执行

### React 18
1. render方法废弃，使用createRoot

2. 批量setState
自动批量更新state，减少渲染次数。18之前，只在react事件中批量更新，18之后可以在Promise\setTimeout\dom原生事件等等批量setState。例子：
```javascript
setTimeout(() => {
  setCount(c => c + 1);
  setFlage(f => !f);
  // version < 18，渲染两次
  // version >= 18，渲染一次
}, 1000)

```

3. concurrent - 并发渲染

**version < 18**

线性渲染，无法中止
State update 1 => State update 2

**version >= 18**

渲染可被中断、继续、终止
渲染可在后台运行
渲染可以有优先级
这是一种新的底层机制，不是新功能

State update 1 => State update 2 => State update 1

State update 3 => State update 4 => State update 3

4. Transitions 

用于区别渲染优先级

应对同时有大量渲染的情况

```javascript
import { startTranstions, useTransition } from React;

// class 组件使用 startTranstions

// hook 使用useTranstion

/**
 * isPending 是否低优先级
 * startTransition 用于包裹需要降低优先级的function
 */
const [isPending, startTransition] = useTransition();

比如纯前端filter交互：输入框输入内容后，模糊搜索列表数据。
当列表数据过大时，会导致输入框渲染很慢，列表在更新里，输入的内容还没有出现，导致卡顿

const onChange = (e) => {
  setInput(e.target.value);
  startTransition(() => {
    setSearchParams(e.target.value);
  })
}

<div>
  <Input onChange={onChange} />
  <List filterParams={searchParams}>
</div>


```

5. Suspense

优化并发请求获取数据渲染页面的体验

fetch on render
fetch then render
fetch while render (Suspense)

**Suspense原理:**
```javascript
function Component() {
  if (data) {
    return <div>{data.message}</div>
  }
  throw promise
  // React will catch this, find the closest "Suspense" component
}

<Suspense fallback={<div>loading...</div>}>
  <Component />
</Suspense>
```

**并发获取数据并渲染的常见处理：**
```javascript
import React, { useEffect, useState, Suspense } from 'react'

const BaseInfo = () => {
  const [profile, setProfile] = useState(undefined);
  useEffect(() => {
    fetchProfile().then(res => setProfile(res?.data)) // 3秒返回
  }, [])
  return (
    <div>
      {profile?.name}
    </div>
  )
}
const Frinends = () => {
  const [list, setList] = useState([]);
  useEffect(() => {
    fetchFriends().then(res => setFriends(res?.data)) // 3秒返回
  }, [])
  return (
    <div>
      {list.map(it => <div>{it.name} - {it.id}</div>)}
    </div>
  )
}

const Page1 = () => {
  // fetchProfile 3秒 => fetchFriends 3秒
  // 需要6秒后才完成渲染 
  // fetch on render 
  return (
    <div>
      <BaseInfo />
      <Frinends />
    </div>
  )
}


const Page2 = () => {
  const [profile, setProfile] = useState(undefined);
  const [list, setList] = useState([]);
  // fetch then render 3秒
  // 只能等请求全部获取后才渲染
  useEffect(() => {
    Promise.all([
      new Promise(resolve => setTimeout(() => resolve({ name: 'xxx' }), 3000)), 
      new Promise(resolve => setTimeout(() => resolve([{ name: 'friends1' }, { name: 'friends2' }]), 3000))
    ]).then(([profile, friends]) => {
      setProfile(profile);
      setList(friends);
    })
  }, [])
  return (
    <div>
      <BaseInfo {...profile} />
      <Frinends {...list} />
    </div>
  )
}

// Suspense 处理
function wrapPromise(promise) {
  let status = 'pending';
  let result;
  let suspender = promise.then(
    (resolve) => {
      status = 'success';
      result = r;
    },
    (error) => {
      status = 'error';
      result = error;
    }
  );
  return {
    read() {
      if (status === 'pending') {
        throw suspender;
      } else if (status === 'error') {
        throw result;
      } else if (status === 'success') {
        return result
      }
    }
  }
}
const page3 = () => {
  // 哪个组个请求结束就立刻限显示对应组件
  return (
    <div>
      <Suspense fallback={<div>loading...</div>}>
        <BaseInfo />
      </Suspense>
      <Suspense fallback={<div>loading...</div>}>
        <Friends />
      </Suspense>
    </div>
  )
}



```