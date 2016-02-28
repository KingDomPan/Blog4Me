---
title: ReactRouter
date: 2015-12-04 13:14:53
tags: [ReactJs, ReactRouter]
---

##### 内置对象
- `import { Router, Route, Link, IndexRoute, Redirect } from 'react-router';`
- `import { createHistory, createHashHistory, useBasename } from 'history';`
- Route
  - onEnter回调, 渲染对象的组件前进行拦截操作, 比如权限验证, **异步验证需要需要第三个参数callback**
  - onUpdate回调

<!-- more -->

##### 最佳实践点
- 路由组件对象的属性props会被注入location, history对象, **history**对象即为浏览器的HTML5History对象, 这些对象在`RoutingContext.js`执行
- `location`代表一个URL, 每个URL都会有state对象存放数据, 但是state的对象数据不会出现在URL上, 事实上数据会存在sessionStorage中
- 在路由组件的子组件中, 可以使用context的api来获取location和history对象, `this.context.history`和`this.context.location`
- `<Link to={path} state={null} />` == `history.pushState(null, path)`
- `<Redirect from={currentPath} to={nextPath} />` == `history.replaceState(null, nextPath)`
- React和React-Router的UI重绘, **主要URL保持一致, 返回的UI总是一定的**
- 路径参数`params消息`, `this.props.params.id`
- 其他参数`location对象`, `this.props.location.query`
![Component 2 Router](/images/ReactRouter/1451466274435.png)
- 点击`浏览器按钮`和`Link`组件的机制
![重绘机制](/images/ReactRouter/1451466377835.png)
- location结构
![Location](/images/ReactRouter/1451466588319.png)
- state结构
![State](/images/ReactRouter/1451466653355.png)

- 点击Link后的操作
  1. Link最终会生成`a`标签, `to` `query` `hash`属性会被渲染成href属性
  2. 内部使用脚本拦截了浏览器的默认行为
  3. 调用history.pushState方法(`history`包里面的`create*History`方法创建的对象)
  4. history包中的pushState支持传入state和path, 函数内部将2个参数传递到`createLocation`中, 返回Location对象
  5. 系统将上述Location对象作为参数传递到TransitionTo方法中, 调用window.location.hash或者`window.history.pushState`修改应用的URL, 同时触发`history.listeners`中注册的监听器
  6. `matchRoutes`方法会匹配出Route组件树中与当前`location`对象最匹配的一个, 并返回`nextState`对象
  7. 在Router组件的`componentWillMount`生命周期方法中调用`history.listen(listener)`方法, 传入nextState对象
  8. 执行`this.setState(nextState);`
```javascript
// 这是一段登录组件的代码
const location = this.props.location;
// 在登录前访问其他URL, 将被重定向到登录组件, location的对象中存在着上一个地址的信息, 在登录成功后可以从这里取出信息, 进行访问页面的定位, 重要的是location的state对象, 这个对象和history里面的state一样, 存放着字典数据
if (location.state && location.state.nextPathname) {
  this.props.history.replaceState(null, location.state.nextPathname);
} else {
  // 如果是直接访问登录界面, 登录成功后默认跳转到/about
  this.props.history.replaceState(null, '/about');
}
```

```javascript
// 重定向组件 from to
<Redirect from="/archives" to="/archives/posts"/>
// 重定向后的处理策略, 采用多重子组件渲染的方式
<Route onEnter={requireAuth} path="archives" component={Archives}>
  // 使用 components 声明路由所对应的多个组件
  <Route path="posts" components={{
    original: Original, // 成为上级组件的一个属性
    reproduce: Reproduce, // 成为上级组件的一个属性
  }}/>
</Route>

// 权限验证, 如果没验证通过, 那么跳转到登录页, 这边的参数即为上段代码里面出现的参数
function requireAuth(nextState, replaceState) {
  if (!hasLogin()) {
    // 改方法和Redirect组件类似, 不会在浏览器中留下重定向前的历史
    replaceState({
      // 实际上是/archives/posts
    nextPathname: nextState.location.pathname
  },'/signIn');
  }
}
```

##### 路由生命周期
- 假设存在以下的路由配置
```javascript
<Route path="/" component={App}>
  <IndexRoute component={Home}/>
  <Route path="invoices/:invoiceId" component={Invoice}/>
  <Route path="accounts/:accountId" component={Account}/>
</Route>
```
- `打开'/'`
| 组件       |  生命周期|
| :-------  | ------:  |
| App    |  componentDidMount  |
| Home    |  componentDidMount  |
| Invoice    |  N/A  |
| Account    |  N/A  |

- `当用户从 '/' 跳转到 '/invoice/123'`
    - `app`从`router`接收到新的props(`children`, `params`, `location`等), 所以触发了`cwrp`, `cdu`
    - `Home`不被渲染, 所以被卸载, 触发`cwu`
    - `Invoice`首次被挂载
| 组件       |  生命周期|
| :-------  | ------:  |
| App    |  componentWillReceiveProps, componentDidUpdate  |
| Home    |  componentWillUnmount  |
| Invoice    |  componentDidMount  |
| Account    |  N/A  |

- `当用户从 /invoice/123 跳转到 /invoice/789`
  - 所有的组件之前都已经被挂载, 所以只是从`router`更新了`props`
| 组件       |  生命周期|
| :-------  | ------:  |
| App    |  componentWillReceiveProps, componentDidUpdate  |
| Home    |  N/A  |
| Invoice    |  componentWillReceiveProps, componentDidUpdate  |
| Account    |  N/A  |

- `当从 /invoice/789 跳转到 /accounts/123`
| 组件       |  生命周期|
| :-------  | ------:  |
| App    |  componentWillReceiveProps, componentDidUpdate  |
| Home    |  N/A  |
| Invoice    |  componentWillUnmount  |
| Account    |  componentDidMount  |