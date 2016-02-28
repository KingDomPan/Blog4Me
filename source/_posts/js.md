title: 悟透Javascript
date: 2014-12-28 17:21:14
categories: Javascript
tags: [JS]
---
前几日, 闲来无事, 阅读了下<悟透Javascript>这本书, 顺手记下了一些简单的在理解范围内的知识点.

<!-- more -->

##### 悟透Javascript
1. undefined, null, number, string, boolean, object, function
2. undefined, null, "", 0, false 等价于 false; 其中 undefined == null
3. NaN != NaN; Infinity / Infinity = NaN;
4. object, function才有对象化的能力(组织数据和代码成为复杂结构)
5. js的代码只有function一种形式
6. js引擎的执行是一段一段的(预编译), 会先执行function定义的代码块, 再顺序执行其他
7. 代码执行的时刻与数据标识的结构形成了作用域的概念, 作用域是分割的
8. 原始的全局环境, 包含了一些预定义的对象, 对于浏览器环境来说, 就是window对象
9. 代码运行进行函数function会创造一个新的作用域, 作为当前作用域的子作用域
    - 预分析, 将所有定义式函数直接创建为作用域上的函数变量
    - 解释执行代码, 在当前作用域查找函数或者变量, 没找到就到上层作用域查找
    - 因此, 如果前面的语句引用了后面定义的var变量, 该变量已经存在, 只是初值为undefined
    - 因此, var定义的变量只对当前的作用域有效, 尽管上层作用域有同名的东西, 都与本作用域无关
10. 每次函数调用都会产生新的作用域, 随着函数栈形成作用域链
11. 调用上下文的信息
    - 函数本身 (函数内部引用函数本身的标识符)
    - 函数的caller属性 (调用当前函数的上层函数, null表示没调用或者被全局代码调用)
    - this (任意对象和function结合时的一个概念, 即function要服务的那个对象)
    - arguments (使用数组的方式来访问参数, 尽管并非真正的数组)
12. eval函数执行的代码并不创建新的作用域, 只在当前的作用域执行, 可以访问到当前作用域的this和arguments等对象
13. 使用function定义对象(下面2段代码等价, js内部机制就是那么做的)
    ```javascript
        function newFunction(){};
        var obj1 = new newFunction();
        var obj2 = new newFunction;
    ```
    ```javascript
        function newFunction(){};
        var obj1 = {};
        newFunction.call(obj1);//将newFunction内部的this用obj1传递
    ```
14. 每个对象都有自己的属性和方法空间, `obj1.say == obj2.say` 返回 false, 可以通过this定义一份唯一的逻辑体, 在构造对象的时候就能共享逻辑体, 于是就有了`prototype`
15. 所有function类型的对象都有一个`prototype`的属性, 这个属性又是一个对象, 可以任意的添加或者删除其他属性和方法, 通过该function构造出来的对象可以直接访问和调用. 也就是说`prototype`提供了一种同类对象之间共享属性和方法的机制
16. js构造对象的过程 `var obj = new function();`
    - 建立一个对象
    - 将该对象内置的原型对象设置为构造函数prototype引用的那个对象(对象的原型对象对外是不可见的, 只有构造函数的原型才对象可见)
    - 将该对象作为this的参数调用构造函数, 完成成员的设置等初始化的功能
