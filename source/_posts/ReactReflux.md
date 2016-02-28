---
title: ReactReflux
date: 2016-02-04 13:28:51
tags:
---

#### Reflux
Reflux是根据React的flux创建的单向数据流类库, Reflux的单向数据流模式主要由actions和stores组成. 例如, 当组件list新增item时, 会调用actions的某个方法(如addItem(data)), 并将新的数据当参数传递进去, 通过事件机制, 数据会传递到stores中, stores可以向服务器发起请求, 并更新数据数据库. 数据更新成功后, 还是通过事件机制传递的组件list当中, 并更新ui.
![Flux Dispatcher](/images/ReactJs/reflux.png)

<!-- more -->

#### 示例代码
```js
/** @jsx React.DOM */

var Actions = Reflux.createActions([
    'addItem',
    'testItem'
]);

var Store = Reflux.createStore({
    items: [1, 2],
    // 监听Actions的各个函数
    // 只要Actions的函数被调用, 此函数的参数将会被传递到Store的OnXX处理方法
    listenables: [Actions],
    onAddItem: function (model) {
        this.items.unshift(model);
        this.trigger(this.items);
    },
    onTestItem: function() {
        console.log("OnTestItem");
    }
});

var Component = React.createClass({
    // 使用mixins监听Store, 只要Store发生调用, 就调用Component的OnStatusChange方法
    mixins: [Reflux.listenTo(Store, 'onStatusChange')],
    getInitialState: function () {
        return {list: []};
    },
    onStatusChange: function () {
        this.setState({list: Store.items});
    },
    render: function () {
        return (
            <div>
                {
                    this.state.list.map(function (item) {
                        return <p>{item}</p>
                    })
                }
            </div>
        )
    }
});

React.render(<Component />, document.body);

// 现在调用Actions的函数
/**
var x = 3;
window.setInterval(function() {
    Actions.addItem(x, 1000);
    x += 1;
}, 2000);
*/
Actions.testItem();
```

#### 与flux的相同点
- 有actions
- 有stores
- 单向数据流

#### 于flux的不同点
- 通过内部扩展actions的行为, 移除了单例的dispacther
- stores可以监听actions的行为, 无需进行冗杂的switch判断
- stores可以互相监听, 进一步的数据聚合操作, 类似于map/reduce
- waitFor被连续和平行的数据流所替代

#### 创建Action
```js
// 调用这个事件就会触发相应的事件, 在store中监听这个函数, 并作相应的处理
var ToDoActions = Reflux.createAction(options); // 返回一个定义个函数对象, 类型声明接口
```

#### 创建store并监听action
```js
var store = Reflux.createStore({
  init: function() {
    this.listenTo(ToDoActions.addItem, 'onAddItem');
  },
  onAddItem: function(model) {}
});
```

#### 异步action
为操作提供了相应的promise接口
```js
var getAll = Reflux.createAction({
    asyncResult: true
});

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(getAll, 'getAll');
    },
    getAll: function(model) {
        $.get('/all', function(data) {
            if (data) {
                getAll.completed(data);
            } else {
                getAll.failed(data);
            }
        });
    }
});

var getAll = Reflux.createAction({
    asyncResult: true
});

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(getAll, 'getAll');
    },
    getAll: function(model) {
        $.get('/all', function(data) {
            if (data) {
                getAll.completed(data);
            } else {
                getAll.failed(data);
            }

        });
    }
});

getAll({
    name: 'xxx'
})
.then(function(data) {
    console.log(data);
})
.catch(function(err) {
    throw err;
});
```

#### Action hooks
Reflux为每个action都提供了两个hook方法
- preEmit(params), action emit之前调用, 参数是action传递过来的, 返回值会传递给shouldEmit
- shouldEmit(params) action emit之前调用, 参数默认是action传递, 如果preEmit有返回值, 则是preEmit返回值, 返回值决定是否emit
```js
var addItem = Reflux.createAction({
    preEmit: function(params) {
        console.log('preEmit:' + params);
    },
    shouldEmit: function(params) {
        console.log('shouldEmit:' + params);
    }
});

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(addItem, 'addItem');
    },
    addItem: function(params) {
        console.log('addItem:' + params);
    }
});

addItem('xxx');
// 控制台打印
// $ preEmit:xxx
// $ shouldEmit:xxx
```
```js
var addItem = Reflux.createAction({
    preEmit: function(params) {
        console.log('preEmit:' + params);
        return 324;
    },
    shouldEmit: function(params) {
        console.log('shouldEmit:' + params);
        return true;
    }
});

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(addItem, 'addItem');
    },
    addItem: function(params) {
        console.log('addItem:' + params);
    }
});

addItem('xxx');

// 控制台打印
// $ preEmit: xxx
// $ shouldEmit: 324
// $ addItem: 324
```

#### Action methods
当需要给所有的actiont添加公用方法时, 可以这么干
```js
Reflux.ActionMethods.print = function(str) {
    console.log(str);
};

var addItem = Reflux.createAction();

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(addItem, 'addItem');
    },
    addItem: function(params) {
        console.log('addItem:' + params);
    }
});

addItem.print('xxx');
```

#### trigger, triggerAsync, triggerPromise
**直接调用addItem()实际上是调用trigger或者triggerAsync或者triggerPromise, 它们区别在于**
```js
var addItem = Reflux.createAction(); addItem(); // #默认调用triggerAsync，相当于addItem.triggerAsync()
var addItem = Reflux.createAction({sync:true});addItem(); // #默认调用trigger，相当于addItem.trigger()
var addItem = Reflux.createAction({asyncResult:true});addItem(); // #默认调用triggerPromise，相当于addItem.triggerPromise()
```

#### 创建Store
相应Action的行为, 并同服务器交互
- 监听单个Action
```js
var addItem = Reflux.createAction();

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(addItem, 'addItem'); // 在这里进行监听操作
    },
    addItem: function(model) {
        console.log(model);
    }
});

addItem({
    name: 'xxx'
});
```

- 监听多个Action
```js
var TodoActions = Reflux.createActions([
    'item1',
    'item2',
    'item3'
]);

var TodoStore = Reflux.createStore({
  // 监听多个action方法的第二种写法
  // listenables: [ToDoActions]
    init: function() {
        this.listenToMany(TodoActions);
    },
    onItem1: function(model) { // 方法名规范
        console.log(model);
    },
    onItem2: function(model) {
        console.log(model);
    }
});

TodoActions.item1({
    name: 'xxx'
});
TodoActions.item2({
    name: 'yyy'
});
```

#### Store Methods
扩展Sotre的公用方法有两种方式
```js
Reflux.StoreMethods.print = function(str) {
    console.log(str);
};

var addItem = Reflux.createAction();

var TodoStore = Reflux.createStore({
    init: function() {
        this.listenTo(addItem, 'addItem');
    },
    addItem: function(model) {
        console.log(model);
    }
});

TodoStore.print('rrr');
```
```js
var Mixins = {
    print: function(str) {
        console.log(str);
    }
}

var addItem = Reflux.createAction();

var TodoStore = Reflux.createStore({
    mixins: [Mixins],
    init: function() {
        this.listenTo(addItem, 'addItem');
    },
    addItem: function(model) {
        console.log(model);
    }
});

TodoStore.print('rrr');
```

#### 同组件结合
Action, Store和组件这三者是通过事件机制响应变化的, 构建组件的时候首先需要监听Store的状态. 先定义Action和Store
- 当组件的生命周期结束时需要解除对Store的监听
- 当Store调用trigger时, 才会执行onStatusChange函数, 所以每次Store更新时, 需要手动调用trigger函数
```js
var TodoActions = Reflux.createActions([
    'getAll'
]);

var TodoStore = Reflux.createStore({
    items: [1, 2, 3],
    listenables: [TodoActions],
    onGetAll: function() {
        this.trigger(this.items); // 手动触发, 执行ViewComponent更新
    }
});

var TodoComponent = React.createClass({
    getInitialState: function() {
        return {
            list: []
        };
    },
    onStatusChange: function(list) {
        this.setState({
            list: list
        });
    },
    componentDidMount: function() {
        this.unsubscribe = TodoStore.listen(this.onStatusChange); // 监听
        TodoActions.getAll(); // 触发Store
    },
    componentWillUnmount: function() {
        this.unsubscribe();
    },
    render: function() {
        return ( <div> {
            this.state.list.map(function(item) {
                return <p> {
                    item
                } </p>
            })
        } </div>)
    }
});
React.render(<TodoComponent /> , document.getElementById('container'));
```

#### Mixins
```js
var TodoComponent = React.createClass({
    mixins: [Reflux.ListenerMixin], // 混入监听器
    getInitialState: function() {
        return {
            list: []
        };
    },
    onStatusChange: function(list) {
        this.setState({
            list: list
        });
    },
    componentDidMount: function() {
        this.unsubscribe = TodoStore.listen(this.onStatusChange);
        TodoActions.getAll();
    },
    render: function() {
        return ( <div> {
            this.state.list.map(function(item) {
                return <p> {
                    item
                } </p>
            })
        } </div>)
    }
});
React.render(<TodoComponent /> , document.getElementById('container'));
```

#### Reflux.listenTo
```js
var TodoComponent = React.createClass({
    mixins: [Reflux.listenTo(TodoStore, 'onStatusChange')],
    getInitialState: function() {
        return {
            list: []
        };
    },
    onStatusChange: function(list) {
        this.setState({
            list: list
        });
    },
    componentDidMount: function() {
        TodoActions.getAll();
    },
    render: function() {
        return ( <div> {
            this.state.list.map(function(item) {
                return <p> {
                    item
                } </p>
            })
        } </div>)
    }
});
React.render(<TodoComponent /> , document.getElementById('container'));
```

#### Reflux.connect
**数据会自动更新到state的list当中**
```js
var TodoComponent = React.createClass({
    mixins: [Reflux.connect(TodoStore, 'list')],
    getInitialState: function() {
        return {
            list: []
        };
    },
    componentDidMount: function() {
        TodoActions.getAll();
    },
    render: function() {
        return ( <div> {
            this.state.list.map(function(item) {
                return <p> {
                    item
                } </p>
            })
        } </div>)
    }
});
React.render(<TodoComponent /> , document.getElementById('container'));
```

#### Reflux.connectFilter
**对数据加了一层过滤器**
```js
var TodoComponent = React.createClass({
    mixins: [Reflux.connectFilter(TodoStore, 'list', function(list) {
        return list.filter(function(item) {
            return item > 1;
        });
    })],
    getInitialState: function() {
        return {
            list: []
        };
    },
    componentDidMount: function() {
        TodoActions.getAll();
    },
    render: function() {
        return ( <div> {
            this.state.list.map(function(item) {
                return <p> {
                    item
                } </p>
            })
        } </div>)
    }
});
React.render(<TodoComponent /> , document.getElementById('container'));
```
