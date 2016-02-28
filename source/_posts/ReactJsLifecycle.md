---
title: ReactJs生命周期
date: 2015-12-02 20:44:39
categories: ReactJs
tags: [ReactJs]
---

##### 生命周期方法

- 最佳实践
	- `componentWillReceiveProps` 接收p触发, 可以更新ps, 不会触发render
	- `componentWillUpdate` 接收到新的ps触发, 不可以更新ps
	- `componentDidUpdate` 组件更新到render后触发, 完成之后可以更新dom
<!-- more -->

- 实例化
	- `getDefaultProps` 对于组件类来说, 这个方法只会调用一次
	- `getInitualState` 每个实例初始化都会调用一次, 可以使用`this.props`
	- `componentWillMount` 首次渲染前调用, 修改state的最后机会
	- `render` 创建虚拟DOM, 标识组件的输出
		- 只能通过`this.props`和`this.state`访问数据
		- 可以返回null, false或者任何React组件
		- 只能出现一个顶级组件(不能返回一组元素)
		- 必须纯净, 意味着不能改变组件的状态或者修改DOM的输出
	- `componentDidMount` render之后并且`真实的DOM`被渲染之后
- 存在期
	- `componentWillReceiveProps(nextProps)` 在任意时刻, 组件的props可以通过父辈的组件来更改, 获得更新props及更新state对象的机会, 传入`nextProps`对象参数, 初始化渲染的时候不会进行调用此方法, 在方法里面调用`this.setState()`不会触发`render`方法, 旧的属性使用`this.props.XX`来获取
	- `shouldComponentUpdate(nextProps, nextState)` 某个组件或者它的任何子组件不需要渲染新的p&s, 返回false, 返回false是告诉react跳过render, componentWillUpdate, componentDidUpdate
	- `componentWillUpdate(nextProps, nextState)` 组件在接收到新的props和state进行渲染之前,  不可以该方法中更新p&s
	- `componentDidUpdate(prevProps, prevState)` 更新渲染好的dom的机会
- 销毁期
	- `componentWillUnmount` 组件从层级中移除的时候

- 生命周期图
![ReactJs生命周期](/images/ReactJs/Lifecycle.png)
