---
title: 浅谈js设计模式
date: 2018-05-11 19:17:01
tags:
categories:
author: nancy
authorLink:
---
## 策略模式
> 定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。目的是将算法的使用与算法的实现分离开来。

一个基于策略模式的程序至少由两部分组成：
1. 一组策略类，策略类封装了具体的算法，并且负责具体的计算过程。
2. 环境类Context, Context接受客户的请求，随后把请求委托给某一个策略类。

```javascript
        validator = {
            validate: function (value, type) {
                switch (type) {
                    case 'isNonEmpty ':
                        {
                            return true; // NonEmpty 验证结果
                        }
                    case 'isNumber ':
                        {
                            return true; // Number 验证结果
                            break;
                        }
                    case 'isAlphaNum ':
                        {
                            return true; // AlphaNum 验证结果
                        }
                    default:
                        {
                            return true;
                        }
                }
            }
        };
        //  测试
        alert(validator.validate("123", "isNonEmpty"));
```

```javascript
var validator = {

    // 所有可以的验证规则处理类存放的地方，后面会单独定义
    types: {},

    // 验证类型所对应的错误消息
    messages: [],

    // 当然需要使用的验证类型
    config: {},

    // 暴露的公开验证方法
    // 传入的参数是 key => value对
    validate: function (data) {

        var i, msg, type, checker, result_ok;

        // 清空所有的错误信息
        this.messages = [];

        for (i in data) {
            if (data.hasOwnProperty(i)) {

                type = this.config[i];  // 根据key查询是否有存在的验证规则
                checker = this.types[type]; // 获取验证规则的验证类

                if (!type) {
                    continue; // 如果验证规则不存在，则不处理
                }
                if (!checker) { // 如果验证规则类不存在，抛出异常
                    throw {
                        name: "ValidationError",
                        message: "No handler to validate type " + type
                    };
                }

                result_ok = checker.validate(data[i]); // 使用查到到的单个验证类进行验证
                if (!result_ok) {
                    msg = "Invalid value for *" + i + "*, " + checker.instructions;
                    this.messages.push(msg);
                }
            }
        }
        return this.hasErrors();
    },

    // helper
    hasErrors: function () {
        return this.messages.length !== 0;
    }
};





// 验证给定的值是否不为空
validator.types.isNonEmpty = {
    validate: function (value) {
        return value !== "";
    },
    instructions: "传入的值不能为空"
};

// 验证给定的值是否是数字
validator.types.isNumber = {
    validate: function (value) {
        return !isNaN(value);
    },
    instructions: "传入的值只能是合法的数字，例如：1, 3.14 or 2010"
};

// 验证给定的值是否只是字母或数字
validator.types.isAlphaNum = {
    validate: function (value) {
        return !/[^a-z0-9]/i.test(value);
    },
    instructions: "传入的值只能保护字母和数字，不能包含特殊字符"
};



var data = {
    first_name: "Tom",
    last_name: "Xu",
    age: "unknown",
    username: "TomXu"
};
//该对象的作用是检查验证类型是否存在
validator.config = {
    first_name: 'isNonEmpty',
    age: 'isNumber',
    username: 'isAlphaNum'
};


validator.validate(data);

if (validator.hasErrors()) {
    console.log(validator.messages.join("\n"));
}
```

通过策略模式，消除了大片的条件分支语句。将算法封装在独立的strategy中，使得它们易于切换，易于理解，易于扩展。同时，算法还可以复用在其他地方，从而避免许多重复的复制粘贴工作。

## 代理模式
> 代理模式为一个对象提供一种代理以控制对这个对象的访问。

虚拟代理是我们最常用的代理模式,它把一些开销很大的对象，延迟到真正需要用到这个对象的时候才去创建

虚拟代理实现图片预加载
```javascript
var addImg = (function(){
    var img = document.createElement('img');
    document.body.appendChild(img);
    return { 
        setSrc: function(src){
            img.src = src;
        }
    }
})();
var proxyAddImg = (function(){
    var img = new Image();
    img.onload = function(){
        addImg.setSrc(this.src);
    }
    return { 
        setSrc: function(src){
            addImg.setSrc('loading.gif'); 
            img.src = src;
        }
    }
})();
proxyAddImg.setSrc('demo.png'); 
```
虚拟代理合并Http请求
我们可以通过一个代理函数来收集一段时间之内的请求，最后把请求合并到一起发送给服务器
```javascript
var proxySynData = (function(){
    var cache = [], //缓存我们需要同步的内容
        timer; //定时器
    return function(ID){
        if(!timer){ //定时器不存在就创建
            timer = setTimeout(function(){
                synData(cache.join()); //同步合并后的数据
                cache.length = 0; //清空缓存
                clearTimeout(timer); //清除定时器
                timer = null; //方便垃圾回收
            }, 2000);
        }
        cache.push(ID); //存入缓存
    }
})();
var list = document.getElementsByTagName('input');
for(var i = 0, item; item = list[i]; i++){
    item.onclick = function(){
        if(this.checked){
            proxySynData(this.id);
        }
    };
}
```

缓存代理
缓存代理就很好理解了，可以缓存一些开销很大的运算结果;如果你第二次执行函数的时候，传递了同样的参数，那么就直接使用缓存的结果,如果运算量很大，这可是不小的优化
```javascript
var add = function(){
    var sum = 0;
    for(var i = 0, l = arguments.length; i < l; i++){
        sum += arguments[i];
    }
    return sum;
};
var proxyAdd = (function(){
    var cache = {}; //缓存运算结果的缓存对象
    return function(){
        var args = Array.prototype.join.call(arguments);//把参数用逗号组成一个字符串作为“键”
        if(cache.hasOwnProperty(args)){//等价 args in cache
            console.log('使用缓存结果');
            return cache[args];//直接使用缓存对象的“值”
        }
        console.log('计算结果');
        return cache[args] = add.apply(this,arguments);//使用本体函数计算结果并加入缓存
    }
})();
console.log(proxyAdd(1,2,3,4,5)); //15
console.log(proxyAdd(1,2,3,4,5)); //15
console.log(proxyAdd(1,2,3,4,5)); //15
```

## 观察者模式
> 观察者模式又叫发布-订阅模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖它的对象都将得到通知。


定义一个事件对象，它有以下功能
1. 监听事件（订阅公众号）
2. 触发事件（公众号发布）
3. 移除事件（取订公众号）

```javascript
// subscription.js
var CLEARED = null;
var nullListeners = {
  notify: function notify() {}
};

function createListenerCollection() {
  // the current/next pattern is copied from redux's createStore code.
  // TODO: refactor+expose that code to be reusable here?
  var current = [];
  var next = [];

  return {
    clear: function clear() {
      next = CLEARED;
      current = CLEARED;
    },
    notify: function notify() {
      var listeners = current = next;
      for (var i = 0; i < listeners.length; i++) {
        listeners[i]();
      }
    },
    get: function get() {
      return next;
    },
    subscribe: function subscribe(listener) {
      var isSubscribed = true;
      if (next === current) next = current.slice();
      next.push(listener);

      return function unsubscribe() {
        if (!isSubscribed || current === CLEARED) return;
        isSubscribed = false;

        if (next === current) next = current.slice();
        next.splice(next.indexOf(listener), 1);
      };
    }
  };
}

var Subscription = function () {
  function Subscription(store, parentSub, onStateChange) {
    _classCallCheck(this, Subscription);

    this.store = store;
    this.parentSub = parentSub;
    this.onStateChange = onStateChange;
    this.unsubscribe = null;
    this.listeners = nullListeners;
  }

  Subscription.prototype.addNestedSub = function addNestedSub(listener) {
    this.trySubscribe();
    return this.listeners.subscribe(listener);
  };

  Subscription.prototype.notifyNestedSubs = function notifyNestedSubs() {
    this.listeners.notify();
  };

  Subscription.prototype.isSubscribed = function isSubscribed() {
    return Boolean(this.unsubscribe);
  };

  Subscription.prototype.trySubscribe = function trySubscribe() {
    if (!this.unsubscribe) {
      this.unsubscribe = this.parentSub ? this.parentSub.addNestedSub(this.onStateChange) : this.store.subscribe(this.onStateChange);

      this.listeners = createListenerCollection();
    }
  };

  Subscription.prototype.tryUnsubscribe = function tryUnsubscribe() {
    if (this.unsubscribe) {
      this.unsubscribe();
      this.unsubscribe = null;
      this.listeners.clear();
      this.listeners = nullListeners;
    }
  };

  return Subscription;
}();
```
Redux采用了观察者模式

#### createStore(rootReducer,initialState,applyMiddleware(thunkMiddleware))

返回值：
(1) dispatch(action): 用于action的分发，改变store里面的state
(2) subscribe(listener): 注册listener，store里面state发生改变后，执行该listener。返回unsubscrib()方法，用于注销当前listener。
(3) getState(): 读取store里面的state
(4) replaceReducer(): 替换reducer，改变state修改的逻辑

所以store内部维护listener数组，用于存储所有通过store.subscribe注册的listener；当store tree更新后，依次执行数组中的listener
```javascript
  function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Expected listener to be a function.');
    }

    var isSubscribed = true;

    ensureCanMutateNextListeners();
    nextListeners.push(listener);
    了
    return function unsubscribe() {
      if (!isSubscribed) {
        return;
      }

      isSubscribed = false;

      ensureCanMutateNextListeners();
      var index = nextListeners.indexOf(listener);
      nextListeners.splice(index, 1);
    };
  }
```

dispatch方法主要完成两件事：
1、根据action查询reducer中变更state的方法，更新store tree
2、变更store tree后，依次执行listener中所有响应函数
```javascript
  function dispatch(action) {
    if (!(0, _isPlainObject2['default'])(action)) {
      throw new Error('Actions must be plain objects. ' + 'Use custom middleware for async actions.');
    }

    if (typeof action.type === 'undefined') {
      throw new Error('Actions may not have an undefined "type" property. ' + 'Have you misspelled a constant?');
    }

    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.');
    }

    try {
      isDispatching = true;
      currentState = currentReducer(currentState, action);
    } finally {
      isDispatching = false;
    }
    //循环遍历，执行listener，通知数据改变
    var listeners = currentListeners = nextListeners;
    for (var i = 0; i < listeners.length; i++) {
      var listener = listeners[i];
      listener();
    }

    return action;
  }
```

#### 在redux中，我们用connect()方法将react中的UI组件与redux的状态、事件关联起来。

```javascript
    var Connect = function (_Component) {
      _inherits(Connect, _Component);
      /*
      * 构造函数中，构造一个订阅对象，属性有this.store,方法this.onStateChange.bind(this)
      */
      function Connect(props, context) {
        _classCallCheck(this, Connect);

        var _this = _possibleConstructorReturn(this, _Component.call(this, props, context));

        _this.version = version;
        _this.state = {};
        _this.renderCount = 0;
        _this.store = props[storeKey] || context[storeKey];
        _this.propsMode = Boolean(props[storeKey]);
        _this.setWrappedInstance = _this.setWrappedInstance.bind(_this);

        (0, _invariant2.default)(_this.store, 'Could not find "' + storeKey + '" in either the context or props of ' + ('"' + displayName + '". Either wrap the root component in a <Provider>, ') + ('or explicitly pass "' + storeKey + '" as a prop to "' + displayName + '".'));

        _this.initSelector();
        _this.initSubscription();
        return _this;
      }

      ···
      // 调用store.subscribe(listener)注册监听方法，对store的变化进行订阅，当store变化的时候，更新渲染view
      Connect.prototype.componentDidMount = function componentDidMount() {
        if (!shouldHandleStateChanges) return;
        this.subscription.trySubscribe(); // //实际调用this.store.subscribe(this.onStateChange);
        this.selector.run(this.props);
        if (this.selector.shouldComponentUpdate) this.forceUpdate();
      };
       ···

      Connect.prototype.componentWillUnmount = function componentWillUnmount() {
        if (this.subscription) this.subscription.tryUnsubscribe(); // 取消订阅
        this.subscription = null;
        this.notifyNestedSubs = noop;
        this.store = null;
        this.selector.run = noop;
        this.selector.shouldComponentUpdate = false;
      };
        ···
      //初始化订阅逻辑
      Connect.prototype.initSubscription = function initSubscription() {
        if (!shouldHandleStateChanges) return;

        var parentSub = (this.propsMode ? this.props : this.context)[subscriptionKey];
        this.subscription = new _Subscription2.default(this.store, parentSub, this.onStateChange.bind(this));  //调用的是Subscription.js中方法,向store内部注册一个listener---this.onStateChange.bind(this)

        this.notifyNestedSubs = this.subscription.notifyNestedSubs.bind(this.subscription);
      };

      Connect.prototype.onStateChange = function onStateChange() {
        this.selector.run(this.props);

        if (!this.selector.shouldComponentUpdate) {
          this.notifyNestedSubs();
        } else {
          this.componentDidUpdate = this.notifyNestedSubsOnComponentDidUpdate;
          this.setState(dummyState);
        }
      };

    ···

      return Connect;
    }(_react.Component);

```

## 装饰者模式
> 在不改变对象自身的基础上，在程序运行期间给对象动态地添加一些额外职责

```javascript
// 给window绑定onload事件
window.onload = function() {
    alert(1)
}

var _onload = window.onload || function(){}  //维护中间变量，但是如果装饰链太长，或需要的装饰函数太多，这些中间变量的数量就会越来越多

window.onload = function() {
    _onload()
    alert(2)
}

//可能会存在this被劫持问题
var getId = document.getElementById; //全局函数，this指向window
document.getElementById = function(ID){
    console.log(1);
    return getId(ID);
}
document.getElementById('demo'); //this预期指向document

//需要手动把document当做上下文this传入getId
```
### AOP装饰函数
> AOP（Aspect Oriented Programming）面向切面编程 
把一些与核心业务逻辑无关的功能抽离出来 
再通过“动态织入”方式掺入业务逻辑模块

```javascript
// 前置装饰
Function.prototype.before = function(beforeFunc){
    var that = this; //保存原函数的引用
    return function(){ //返回了了包含原函数和新函数的代理函数
        beforeFunc.apply(this, arguments); // 执行新函数
        return that.apply(this, arguments); //执行原函数并返回原函数的执行结果
    }
}

document.getElementById = document.getElementById.before(function() {
    alet(1)
})
```

```javascript
//定义一个组件
class Home extends Component {
    //....
}

//以往从状态树取出对应的数据，通过props传给组件使用通过react-redux自带的connect()方法
export default connect(state => ({todos: state.todos}))(Home);

//使用装饰器的话就变成这样，好像没那么复杂
@connect(state => ({ todos: state.todos }))
class Home extends React.Component {
    //....
}
```
## 适配器模式
适配器模式是作为两个不兼容的接口之间的桥梁，它结合了两个独立接口的功能。

假设我们正在编写一个渲染广东省地图的页面，目前从第三方资源里获得了广东省的所有城市以及它们所对应的ID,并且成功地渲染到页面中：
```javascript
var getGuangdongCity = function () {
    var GuangdongCity = [
       {
          name:'shenzhen',
          id  : '11'
       },
       {
         name:'guangzhou',
         id:12
       }
  ];
  return GuangdongCity;
};

var render  = function (fn) {
    console.log('starting render Guangdong map');
    document.write(JSON.stringify(fn()));
};
/* var GuangdongCity = {
//                      shenzhen:11,
//                      guangzhou:12,
//                      zhuhai:13
//                     };
*/
var addressAdapter = function (oldAddressfn) {
var address = {},
    oldAddress = oldAddressfn();
    for(var i = 0 , c; c = oldAddress[i++];){
        address[c.name] = c.id;  //此处我们遍历老数据把它们添加到空对象中然后返回这个对象
    }
    return function () {
        return address;
    }
};
render(addressAdapter(getGuangdongCity));

```
使用适配器模式可以解决参数类型有些许不一致造成的问题。

redux为了和react适配，所有有 mapStateToProps()这个函数来把state转为Props外部状态，这样就可以从外部又回到组件内了
