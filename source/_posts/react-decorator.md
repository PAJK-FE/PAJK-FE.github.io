---
title: 浅析修饰器(Decorator)在React中的应用
date: 2017-06-02 14:27:22
tags:
- decorator
- react
categories: react
author: iloveplus
---

## 修饰器原理理解
修饰器（Decorator）是一个函数，用来修改类的行为。这是ES7的一个提案，目前Babel转码器已经支持（需安装babel-core和babel-plugin-transform-decorators）。修饰器对类的行为的改变，<font color='red'>是代码编译时发生的，而不是在运行时</font>。这意味着，修饰器能在编译阶段运行代码。
下面我们从一个例子来了解修饰器它的内部是怎么运行的：
### 未使用修饰器：
#### 转码前：
```javascript
import React, { Component } from 'react';

const testable = (target) => {
    target.title = '使用了修饰器';
}

class ListDemo extends Component {
    render() {
        let title = ListDemo.title || '未使用修饰器';
        return (
            <div>{title}</div>
        )
    }
}

export default ListDemo;
```
#### 转码后：
```javascript
var testable = function testable(target) {
    target.title = '使用了修饰器';
};

var ListDemo = function (_Component) {
    (0, _inherits3.default)(ListDemo, _Component);

    function ListDemo() {
        (0, _classCallCheck3.default)(this, ListDemo);
        return (0, _possibleConstructorReturn3.default)(this, (ListDemo.__proto__ || (0, _getPrototypeOf2.default)(ListDemo)).apply(this, arguments));
    }

    (0, _createClass3.default)(ListDemo, [{
        key: 'render',
        value: function render() {
            var title = ListDemo.title || '未使用修饰器';
            return _react2.default.createElement(
                'div',
                null,
                title
            );
        }
    }]);
    return ListDemo;
}(_react.Component);

exports.default = ListDemo;
```
### 使用修饰器：
#### 转码前：
```javascript
import React, { Component } from 'react';

const testable = (target) => {
    target.title = '使用了修饰器';
}

@testable
class ListDemo extends Component {
    render() {
        let title = ListDemo.title || '未使用修饰器';
        return (
            <div>{title}</div>
        )
    }
}

export default ListDemo;
```

#### 转码后：
```javascript
var testable = function testable(target) {
    target.title = '使用了修饰器';
};

var ListDemo = testable(_class = function (_Component) {
    (0, _inherits3.default)(ListDemo, _Component);

    function ListDemo() {
        (0, _classCallCheck3.default)(this, ListDemo);
        return (0, _possibleConstructorReturn3.default)(this, (ListDemo.__proto__ || (0, _getPrototypeOf2.default)(ListDemo)).apply(this, arguments));
    }

    (0, _createClass3.default)(ListDemo, [{
        key: 'render',
        value: function render() {
            var title = ListDemo.title || '未使用修饰器';
            return _react2.default.createElement(
                'div',
                null,
                title
            );
        }
    }]);
    return ListDemo;
}(_react.Component)) || _class;

exports.default = ListDemo;
```
对比转换后的结果，我们发现，使用修饰器等效于：
```javascript
import React, { Component } from 'react';

const testable = (target) => {
    target.title = '使用了修饰器';
}

class ListDemo extends Component {
    render() {
        let title = ListDemo.title || '未使用修饰器';
        return (
            <div>{title}</div>
        )
    }
}

export default testable(ListDemo) || ListDemo;
```

其中@testable就是一个修饰器，它修改了ListDemo这个类的行为，为它加上了静态属性title。也就是说，<font color='red'>修饰器本质就是编译时执行的函数，其中第一个参数，就是所要修饰的目标类</font>。基本上，修饰器的行为就是下面这样：
```javascript
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```
可是在应用过程中往往只传了一个Class作为参数不够灵活怎么办？如上例若title是可变的，这时我们可以<font color='red'>在外层套一个函数，只要最后返回的是一个Decorator即可，而无需管套了多少个函数传多少个参数</font>，修改后的代码如下：
```javascript
import React, { Component } from 'react';

const testable = (title) => (target) => {
    target.title = title;
}

@testable('使用了修饰器')
class ListDemo extends Component {
    render() {
        let title = ListDemo.title || '未使用修饰器';
        return (
            <div>{title}</div>
        )
    }
}

export default ListDemo;
```

理解了上面的例子后，接下来我们来看看修饰器在react中的高阶用法。

## 修饰器高阶应用
我们知道进入列表页往往需要上拉刷新，下拉加载等通用事件和效果，这时我们可以利用修饰器。如下：
```javascript
import React, { Component } from 'react';
import Refresh from './Refresh';
import More from './More';

/*
*   hasLoader: 加载更多功能
*   hasRefresh: 下拉刷新功能
*   pageSize: 每页数据条数
*/
const loadMore = (hasLoader, hasRefresh, pageSize = 10) => (ComposedComponent) => class LoadMore extends Component {
    componentDidMount() {

    }

    //todo:相关处理逻辑

    render() {
        if (!hasLoader && !hasRefresh) {
            return <ComposedComponent {...this.props} />;
        }
        
        return (
            <div>
                { hasRefresh && <Refresh /*...*/ />}
                <ComposedComponent {...this.props} />
                { hasLoader && <More /*...*/ />}
            </div>
        );
    }
};

@loadMore(true, true, 10)
class ListDemo extends Component {
    //...
}

export default ListDemo;
```

## 修饰器在react-redux中的应用
```javascript
//定义一个组件
class Home extends React.Component {
    //....
}

//以往从状态树取出对应的数据，让后通过props传给组件使用通过react-redux自带的connect()方法
export default connect(state => ({todos: state.todos}))(Home);

//使用装饰器的话就变成这样，似乎简单了很多
@connect(state => ({ todos: state.todos }))
class Home extends React.Component {
    //....
}
```

## 参考资料
[http://es6.ruanyifeng.com/#docs/decorator](http://es6.ruanyifeng.com/#docs/decorator)
[http://zhw-island.com/decorator-in-react/](http://zhw-island.com/decorator-in-react/)
[https://www.w3cschool.cn/ecmascript/43uq1q5z.html](https://www.w3cschool.cn/ecmascript/43uq1q5z.html)