---
title: Promise及Fetch源码解析
date: 2015-11-12 20:50:25
tags: [Promise, Fetch]
---

后端的很多语言处理http请求都用上了Promise这种模型, 比如finatra. Promise代表的是一个承诺值或者是一个会抛出的异常对象, 后端可以进行异步处理. 同时, 那万恶的nodejs风格的callback也渐渐被Promise这个模型所替代了. **Promise类似状态机, 而且只有三个状态, 并只能进行2种转换**

![PromiseFetch](/images/ReactJs/PromiseFetch.png)

#### 浏览器默认的Promise模型
当前已经有很多基于Promise的js库, 比如q, bluebird等等, 都提供了比较简洁的api来使用Promise.下面的代码例子是浏览器的默认行为

<!-- more -->

```js
var p = new Promise(function(resolve, reject) {
  // Run your code here
  var k = true;
  console.log("My Name is KingDomPan");
  // Change the State
  if (k === true) {
    resolve("KingDomPan"); // 从pending到fulfilled的状态切换
  } else {
    reject("Also KingDomPan");
  }
});

p.then(function(input) {
  console.log("The Return From Resolve Callback " + input); // 触发回调
}, function(input) {
  console.log("The Return From Reject Callback " + input);
});
```
![Promise](/images/ReactJs/PromiseFetch.png)

**当然了then对象之后的返回还是一个promise, 因此可以持续then的调用**

#### 一个Promise的简短实现
**这个例子证实了Promise是可以纵向处理异步的, 而不是万恶的callback横向处理**
```js
var Promise = function() {
    this.thens = [];
};

Promise.prototype = {
    resolve: function() {
        var t = this.thens.shift(), n;
        // 通过将当前的thens挂靠在子Promise的thens下实现任务的thenable
        t && (n = t.apply(null, arguments), n instanceof Promise && (n.thens = thens));
    },
    then: function(n) {
        return this.thens.push(n)
    }
};
```

#### fetch标准源代码解析
当前IE9, Firefox, Chrome基本已经内置Fetch的全局方法, 下面的代码来源于[Fetch In Github](https://github.com/github/fetch/blob/master/fetch.js)
- function normalizeName(name) 校验http头字段, 返回小写
- function normalizeValue(value) 字符串化value
- function Headers(headers) 创建http头, 参数可以是Header或者js对象, 内部数据结构是一个map, value对应一个数组
- Headers.prototype
	- append 追加http头
	- delete 删除http头
	- get 获得某个http头的第一个值
	- getAll 获得某个http头的全部值, 返回一个数组
	- has 检测是否有某个http头
	- set 设置某个http头的值, 会覆盖map中对应http头的全部值
	- forEach(callback, context) 遍历headers, 首先遍历map中的name, 再遍历取出来的values, 传递到callback中并绑定context(call方法的使用)
- function consumed(body) 设置http体是否被使用, 如果超过一次使用的话就会触发promise的catch进行异常处理
- function fileReaderReady(reader) 返回promise, 当输入流准备就绪, 调用resolve方法进行数据的处理, 异常则进行异常处理. **reader是HTML5 FileReader之类的实现**
- function readBlobAsArrayBuffer(blob) 调用fileReaderReady对blob进行处理而已
- function readBlobAsText(blob) 调用fileReaderReady对blob进行处理而已
- support对象, 该对象检测是否具有HTML5的属性, 比如support.blob support.formData support.arrayBuffer
- function Body()
	- 很明显Body的构造函数
	- this.bodyUsed = false;
	- this._initBody = function(body)
		- this._bodyInit 保存原始body数据
		- 判断body类型, 创建this.__bodyXXXX保存body
    - this.blob()返回promise处理body数据, 数据字段对应了this.__initBody里面创建的
    - this.arrayBuffer() 同上
    - this.text() 同上
    - this.formData() 调用decode解码而已, 返回Promise
    - this.json() 调用JSON.parse()返回json对象而已, 返回Promise

- function normalizeMethod(method) 返回规定的HTTP方法字符串而已
- function Request(input, options) 创建请求对象 **这个Request会继承Body**
	- input 包含url, cookie, method, headers 等等的信息 (**如果继承Body的话**), 否则就是url(**没继承Body**)
	- options 包含body的可选信息
	- 一系列设值操作.......
- Request.prototype.clone
	- return new Request(this) 简单明了
- function decode(body) queryString的解码, 返回HTLM5的FormData()对象, 参数是字符串, 即bodyText
- function headers(xhr) 从xhr对象获得所有的响应头, 创建Headers并返回
- Body.call(Request.prototype) 果然Request是直接继承Body的
- function Response(bodyInit, options) **Response对象的创建**
	- options 返回回来的响应头信息
- Body.call(Response.prototype) 果然Response也是直接继承Body的
- Response.prototype.clone() 不废话了
- Response.error = function() 直接在构造函数上创建一error方法返回一个错误的response实例对象
- redirectStatuses 重定向的http状态码
- Response.redirect = function(url, status) {} 返回一个重定向的response
- 给window设置全局对象, 或者nodejs中的global
	- self.Headers = Headers;
	- self.Request = Request;
	- self.Response = Response;
- self.fetch(input, init) 定义伟大的fetch方法
	- 返回一个promise
		- 初始化请求对象Request
		- 创建xhr对象
			- onload 验证是不是网络异常, 返回一个Response给Promise的then进行处理
			- onerror 网络异常了才会触发这个
		- 内部函数responseURL, 主要是为了CORS的安全考虑的
		- 将Request的头信息弄到xhr对象里, 发请求
- self.fetch.polyfill = true 最后这个了这个货的一个属性, 我也不知道
