---
title: react事件机制源码分析
date: 2017-06-03 11:10:19
tags: react
categories: react
author: peanut
authorLink:
---

### React事件机制
### 本文主要从源码角度讨论以下问题
- react事件和原生 dom 事件的区别，混用的时候的问题。
- react 是如何在虚拟 dom 上完成事件的绑定，绑定在哪？
- 既然 react 采用的事件代理，那它又是如何做到事件冒泡的效果，如何阻止冒泡的，提供的阻止冒泡的 api 和原生阻止冒泡的 api 有何关联？
- react 存在事件捕获吗？他是真正的事件捕获吗？又是如何做到捕获的效果？
- react 事件的插件机制及合成的事件对象（reatc 支持的7个事件插件）

### react事件绑定阶段（绑定阶段和事件处理器存储阶段）
#### 绑定阶段
1：当我们在 jsx 上绑定了 react 提供的事件绑定方式，那么绑定的该事件是如何被注册到事件系统的呢？组件的 mounting 和 updata都会去解析 jsx 上绑定的事件，这里我们只看 mounting阶段;
react事务： 其源码在src/shared/utils/Transaction.js. (以下用到的)transaction: react的方法执行是基于事务的，Transaction就是给需要执行的方法fn用wrapper封装了 initializeAll 和 closeAll 方法。且支持多次封装。再通过 Transaction 提供的 perform 方法执行。 perform执行前，调用所有initializeAll 方法。perform 方法执行后，调用所有closeAll方法。
```javascript
    mountComponent: function(rootID, transaction, context) {
    // 这里只取关键部分分析
      this._rootNodeID = rootID;

      var props = this._currentElement.props;
      var mountImage;
      if (transaction.useCreateElement) {
      } else {
        // 创建每个元素的开始标签和解析元素上的props，如 style、事件、等
        var tagOpen = this._createOpenTagMarkupAndPutListeners(transaction, props);
        // 子标签作为 content 解析，会进入到mountChildren，mountChildren作用如mountComponent 同样会解析标签上的一些属性
        var tagContent = this._createContentMarkup(transaction, props, context);
        if (!tagContent && omittedCloseTags[this._tag]) {
          mountImage = tagOpen + '/>';
        } else {
          mountImage =
            tagOpen + '>' + tagContent + '</' + this._currentElement.type + '>';
        }
      }
      return mountImage;
  },
  ```
  ```javascript
  _createOpenTagMarkupAndPutListeners: function(transaction, props) {
    var ret = '<' + this._currentElement.type;
    // 这里的 props 包含标签上所有可以定义的属性，只分析事件
    for (var propKey in props) {
      // registrationNameModules 这是加载 react 应用时，注册到 react EventPluginHub这个事件插件中心所有 react 支持的事件类型
      if (registrationNameModules.hasOwnProperty(propKey)) {
        if (propValue) {
          enqueuePutListener(this._rootNodeID, propKey, propValue, transaction);
        }
      }
    }
```
2：主要来看这个enqueuePutListener,这个函数主要完成事件绑定在 document 对象上，和把相关的事件信息进行存储。
这里我们可以看出 jsx 中申明的事件确实绑定在 document 对象上了，在每一个标签解析完毕后，会把 domId，事件名称、事件处理函数存储在后面提到的listenBank这个对象中;

```javascript
function enqueuePutListener(id, registrationName, listener, transaction) {
  var container = ReactMount.findReactContainerForID(id);
  if (container) {
    // 确保元素类型是节点类型,并且取到 document 节点
    var doc = container.nodeType === ELEMENT_NODE_TYPE ?
      container.ownerDocument :
      container;
    // 在 document 对象上完成事件的绑定
    listenTo(registrationName, doc);
  }
  // 该方法会在jsx中每一个嵌套的dom标签渲染完毕（依次往上递归）会去调用
  transaction.getReactMountReady().enqueue(putListener, {
    id: id,
    registrationName: registrationName,
    listener: listener,
  });
}
```
3：listenTo方法，主要是处理一些浏览器冒泡和捕获兼容性的问题，同时可以看出，同种类型的事件只绑定一次，保证了效率
``` javascript
  listenTo: function(registrationName, contentDocumentHandle) {
    var mountAt = contentDocumentHandle;
    var isListening = getListeningForDocument(mountAt);
    var dependencies =
      EventPluginRegistry.registrationNameDependencies[registrationName];

    var topLevelTypes = EventConstants.topLevelTypes;
    for (var i = 0; i < dependencies.length; i++) {
      var dependency = dependencies[i];
      // 以下分支只看最后一个，上面的几个都是特殊处理过的，如兼容性什么的
      if (!(
            isListening.hasOwnProperty(dependency) &&
            isListening[dependency]
          )) {
        if (dependency === topLevelTypes.topWheel) {
        } else if (dependency === topLevelTypes.topScroll) {
        } else if (dependency === topLevelTypes.topFocus ||
            dependency === topLevelTypes.topBlur) {
        } else if (topEventMapping.hasOwnProperty(dependency)) {
          ReactBrowserEventEmitter.ReactEventListener.trapBubbledEvent(
            dependency,
            topEventMapping[dependency],
            mountAt
          );
        }
        isListening[dependency] = true;
      }
    }
  },
  ```
 4：react 中会提供捕获的api，如onClickCapture, 在事件收集阶段，react都统一处理了，如onClick、 onClickCapture转化为统一的命名如topClcik，从这我们也能察觉到，这个捕获的 api 是不是真正意义上的捕获？trapBubbledEvent 完成了事件绑定到 document 上的任务，后续具体的事件
  执行，要注意 ReactEventListener.dispatchEvent.bind(null, topLevelType)，是如何在一个节点上模拟出冒泡和捕获的效果。
  ```javascript
  trapBubbledEvent: function (topLevelType, handlerBaseName, element) {
    if (!element) {
      return null;
    }
    return EventListener.listen(
      element,   // 绑定到的DOM目标,也就是document
      handlerBaseName,   // eventType
      ReactEventListener.dispatchEvent.bind(null, topLevelType));  // document上的原生事件触发后回调
  },

  listen: function listen(target, eventType, callback) {
    if (target.addEventListener) {
      // 将原生事件添加到target这个dom上,也就是document上。
      target.addEventListener(eventType, callback, false);
      return {
        // 删除事件,这个由React自己回调,不需要调用者来销毁。但仅仅对于React合成事件才行
        remove: function remove() {
          target.removeEventListener(eventType, callback, false);
        }
      };
    } else if (target.attachEvent) {
      // attach和detach的方式
      target.attachEvent('on' + eventType, callback);
      return {
        remove: function remove() {
          target.detachEvent('on' + eventType, callback);
        }
      };
    }
  },
  ```
 总结一下: 通过以上阶段react初步完成同种类型事件的单次绑定且绑定在document对象，接下来我们再看相关的事件信息如事件名，domid，事件处理器是如何存储在listenBank 这个事件对象中的。接着上面的 enqueuePutListener()中的transaction.getReactMountReady()方法
#### 事件句柄存储阶段
添加事件处理器到listenerBank这个统一的事件处理器对象中，jsx中每一个嵌套的dom标签渲染完毕（依次往上递归）会去调用
```javascript
putListener: function(id, registrationName, listener) {
  // eg: listenerBank['onclick'][nodeId]
    var bankForRegistrationName =
      listenerBank[registrationName] || (listenerBank[registrationName] = {});
    bankForRegistrationName[id] = listener;
  },
  ```
  ### 事件触发执行阶段（分复合事件对象合成阶段和事件函数队列执行）
  看到这其实我们就明白了，react 的这一套事件机制其实还是在利用原生事件的冒泡机制，所以原生事件和 react 事件混合使用会有一些坑，比如说在 componentDidMount拿到真实 dom 通过原生事件的方式来绑定一些事件，阻止冒泡，那我们通过 react 绑定的事件是不会响应的等等，稍后一并总结
  #### 复合事件对象合成阶段
  1：先说下 react 所支持的事件插件，BeforeInputEventPlugin, ChangeEventPlugin, DOMEventPluginOrder, EnterLeaveEventPlugin,
  SelectEventPlugin, SimpleEventPlugin, TapEventPlugin, 总共7个事件插件。SimpleEventPlugin插件是我们常用的，支持的事件类型有焦点事件、键盘事件、鼠标事件、拖放事件、触摸事件、鼠标滚轮事件、ui事件, 这些事件类型继承自SyntheticEvent（提供事件对象中相同的部分）。每种事件类型合成的事件对象参数有相同的部分又有定制的部分，当然一个操作可能会引起多种事件类型，如鼠标点击会引起鼠标事件和 ui 事件。所以 reatc 会根据特定的事件类型去合成特定的事件对象（可能是多个事件类型复合而成）。就以我们常用的SimpleEventPlugin插件支持的鼠标事件大概说一下,鼠标事件又分很多种，比如单击事件、右键菜单事件、鼠标进出等、下图为鼠标事件对象中的一些参数，
  ```javascript
var MouseEventInterface = {
  screenX: null,
  screenY: null,
  clientX: null,
  clientY: null,
  ctrlKey: null,
  shiftKey: null,
  altKey: null,
  metaKey: null,
  getModifierState: getEventModifierState,
  button: function(event) {
    // 一些兼容性的处理
  },
  buttons: null,
  relatedTarget: function(event) {
  //  同上
  },
  // "Proprietary" Interface.
  pageX: function(event) {
  },
  pageY: function(event) {
  },
};
```
SyntheticEvent: 提供事件对象共同的部分，和在原型上挂一些复合事件对象的方法如preventDefault、stopPropagation、persist、destructor（看起来是不是很熟悉，这就是 react 提供出来的关于事件对象的 api), 注意一下this.isPropagationStopped、this.isPropagationStopped，这是后续我们会看到的一个关键标识。
```javascript
var EventInterface = {
  type: null,
  currentTarget: emptyFunction.thatReturnsNull,
  eventPhase: null,
  bubbles: null,
  cancelable: null,
  timeStamp: function(event) {
    return event.timeStamp || Date.now();
  },
  defaultPrevented: null,
  isTrusted: null,
};

assign(SyntheticEvent.prototype, {

  preventDefault: function() {
    this.defaultPrevented = true;
    var event = this.nativeEvent;
    if (!event) {
      return;
    }
    if (event.preventDefault) {
      event.preventDefault();
    } else {
      event.returnValue = false;
    }
    // 注意这个方法，这是才是 react 提供出来阻止默认事件的方法，以上为了在具体的触发元素上同步信息
    this.isDefaultPrevented = emptyFunction.thatReturnsTrue;
  },

  stopPropagation: function() {
    var event = this.nativeEvent;
    if (!event) {
      return;
    }
    if (event.stopPropagation) {
      event.stopPropagation();
    } else {
      event.cancelBubble = true;
    }
    // 注意这个方法，这是才是 react 提供出来阻止冒泡的方法，以上为了在具体的触发元素上同步信息
    this.isPropagationStopped = emptyFunction.thatReturnsTrue;
  },
  persist: function() {
    this.isPersistent = emptyFunction.thatReturnsTrue;
  },
  isPersistent: emptyFunction.thatReturnsFalse,
  destructor: function() {
    var Interface = this.constructor.Interface;
    for (var propName in Interface) {
      this[propName] = null;
    }
    this.dispatchConfig = null;
    this.dispatchMarker = null;
    this.nativeEvent = null;
  },

});
```
2：以上只是介绍了 react 的事件插件，下面我们继续看看如何合成这个复合事件对象，在第一部分我们已经完成了事件的绑定和事件句柄的存储，接下来就分析一下执行阶段中的事件对象的合成，react 的这一套事件机制是建立在原生事件的基础上的，我们在具体的元素上如 button 点击了一下，那么他会通过事件流中的冒泡阶段到达 document 上，我们在第一阶段已经完成了 document 上该类型事件的绑定，剩下的就是去执行这个队列，接上面的trapBubbledEvent，
```javascript
// topLevelType：讲相关的事件名称转为 top 开头的，如topClick。前面讲到过的
// nativeEvent: 用户触发click等事件时，浏览器传递的原生事件
dispatchEvent: function (topLevelType, nativeEvent) {
    // 静态池、批处理的为了提高性能的不说了
    var bookKeeping = TopLevelCallbackBookKeeping.getPooled(topLevelType, nativeEvent);
    try {
      // 放入批处理队列中,React事件流也是一个消息队列的方式
      ReactUpdates.batchedUpdates(handleTopLevelImpl, bookKeeping);
    } finally {
      TopLevelCallbackBookKeeping.release(bookKeeping);
    }
}
handleTopLevel: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
    // 采用对象池的方式构造出合成事件。不同的eventType的合成事件不同
    var events = EventPluginHub.extractEvents(topLevelType, targetInst, nativeEvent, nativeEventTarget);
    // 批处理队列中的events
    runEventQueueInBatch(events);
  }
```
3 根据事件类型构造事件对象，不同的事件类型合成的事件对象参数上面已经说过，主要看事件对象中参数中的 _dispatchIDs 和 _dispatchListeners的生成
```javascript
extractEvents: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
    var events;
    // EventPluginHub可以存储React合成事件的callback,也存储了一些plugin,这些plugin在EventPluginHub初始化时就注册就来了
    var plugins = EventPluginRegistry.plugins;
    for (var i = 0; i < plugins.length; i++) {
      var possiblePlugin = plugins[i];
      if (possiblePlugin) {
        // 根据eventType构造不同的合成事件SyntheticEvent，  如SimpleEventPlugin
        var extractedEvents = possiblePlugin.extractEvents(topLevelType, targetInst, nativeEvent, nativeEventTarget);
        if (extractedEvents) {
          // 将构造好的合成事件extractedEvents添加到events数组中,这样就保存了所有plugin构造的合成事件
          events = accumulateInto(events, extractedEvents);
        }
      }
    }
    return events;
  },
```
4：当我们在具体 dom 上触发了事件（click），会先从根组件到具体触发事件的 dom向下遍历，依次取出我们之前在listenBank 中存储的相关捕获类型的事件的监听器（onClickCapture）和domID,然后在从具体 dom 向上遍历至根组件，依次取出冒泡类型的监听器（onClick）和 domId, 分别挂载在事件对象中的 _dispatchListeners和 _dispatchIDs中，至此这个完整的事件对象合成完毕。accumulateDirectionalDispatches完成挂载。
```javascript
function accumulateTwoPhaseDispatchesSkipTarget(events) {
  // 对合成的 events 分别执行 accumulateTwoPhaseDispatchesSingleSkipTarget
  forEachAccumulated(events, accumulateTwoPhaseDispatchesSingleSkipTarget);
}
// traverseTwoPhaseSkipTarget 会在遍历 dom 的结构中依次调用accumulateDirectionalDispatches
function accumulateTwoPhaseDispatchesSingleSkipTarget(event) {
  if (event && event.dispatchConfig.phasedRegistrationNames) {
    EventPluginHub.injection.getInstanceHandle().traverseTwoPhaseSkipTarget(
      event.dispatchMarker,
      accumulateDirectionalDispatches,
      event
    );
  }
}
// domID,正在遍历的 dom，upwards向上还是向下，模拟捕获和冒泡的顺序，event复合事件对象
function accumulateDirectionalDispatches(domID, upwards, event) {
  var phase = upwards ? PropagationPhases.bubbled : PropagationPhases.captured;
  var listener = listenerAtPhase(domID, event, phase);
  if (listener) {
    event._dispatchListeners =
      accumulateInto(event._dispatchListeners, listener);
    event._dispatchIDs = accumulateInto(event._dispatchIDs, domID);
  }
}
// 先向下，再向上
traverseTwoPhaseSkipTarget: function(targetID, cb, arg) {
    if (targetID) {
      traverseParentPath('', targetID, cb, arg, true, true);
      traverseParentPath(targetID, '', cb, arg, true, true);
    }
  },
function traverseParentPath(start, stop, cb, arg, skipFirst, skipLast) {
  start = start || '';
  stop = stop || '';
  var traverseUp = isAncestorIDOf(stop, start);
  // Traverse from `start` to `stop` one depth at a time.
  var depth = 0;
  var traverse = traverseUp ? getParentID : getNextDescendantID;
  for (var id = start; /* until break */; id = traverse(id, stop)) {
    var ret;
    if ((!skipFirst || id !== start) && (!skipLast || id !== stop)) {
      ret = cb(id, traverseUp, arg);
    }
    if (ret === false || id === stop) {
      // Only break //after// visiting `stop`.
      break;
    }
  }
}
```
#### 执行阶段
1：剩下的就是取出复合事件对象中的_dispatchListeners这个队列来执行，在SyntheticEvent中我们说到，react 提供的几个关于事件对象的几个 api，如preventDefault、stopPropagation、persist，我们说过注意一下this.isPropagationStopped、this.isPropagationStopped，以stopPropagation 这个 api 为列，在事件处理其中我们调用 e.stopPropagation()、e.preventDefault(),这两个虽然看起来和原生的 api 长得一样（也仅仅是长得一样），其实调用他们时，该事件流已经传播到了 document上了，为了保持和原生事件对象的参数同步（毕竟复合对象中有个nativeEventz这个里面是原生的事件对象）所以拿到原生对象再去调用一下相关的方法，并且设置了event.isPropagationStopped = 返回值为 true 的函数，所以在执行这个事件处理器队列的时候， 会 break，达到了阻止冒泡的效果（并非真正的阻止冒泡，真正的冒泡是存在多级嵌套的父子关系的元素，依次传递)。同时可以看到事件处理器默认有两个参数，复合事件对象和 domid。
```javascript
function executeDispatchesInOrder(event, simulated) {
  var dispatchListeners = event._dispatchListeners;
  var dispatchIDs = event._dispatchIDs;
  if (Array.isArray(dispatchListeners)) {
    for (var i = 0; i < dispatchListeners.length; i++) {
      if (event.isPropagationStopped()) {
        break;
      }
      // Listeners and IDs are two parallel arrays that are always in sync.
      executeDispatch(event, simulated, dispatchListeners[i], dispatchIDs[i]);
    }
  } else if (dispatchListeners) {
    executeDispatch(event, simulated, dispatchListeners, dispatchIDs);
  }
  event._dispatchListeners = null;
  event._dispatchIDs = null;
}
```
### react 事件系统架构图
现在我们再来看看 fb 官方给出的 react 整个事件系统的架构，我们就能很容易的看明白整个流程。
```javascript
 Overview of React and the event system:
 *
 * +------------+    .
 * |    DOM     |    .
 * +------------+    .
 *       |           .
 *       v           .
 * +------------+    .
 * | ReactEvent |    .
 * |  Listener  |    .
 * +------------+    .                         +-----------+
 *       |           .               +--------+|SimpleEvent|
 *       |           .               |         |Plugin     |
 * +-----|------+    .               v         +-----------+
 * |     |      |    .    +--------------+                    +------------+
 * |     +-----------.--->|EventPluginHub|                    |    Event   |
 * |            |    .    |              |     +-----------+  | Propagators|
 * | ReactEvent |    .    |              |     |TapEvent   |  |------------|
 * |  Emitter   |    .    |              |<---+|Plugin     |  |other plugin|
 * |            |    .    |              |     +-----------+  |  utilities |
 * |     +-----------.--->|              |                    +------------+
 * |     |      |    .    +--------------+
 * +-----|------+    .                ^        +-----------+
 *       |           .                |        |Enter/Leave|
 *       +           .                +-------+|Plugin     |
 * +-------------+   .                         +-----------+
 * | application |   .
 * |-------------|   .
 * |             |   .
 * |             |   .
 * +-------------+   .
 *                   .
 *    React Core     .  General Purpose Event Plugin System
 */
```
ReactEventListener：负责事件注册和事件分发。React将DOM事件全都注册到document这个节点上。事件分发主要调用dispatchEvent进行。
ReactEventEmitter：负责每个组件上事件的执行。
EventPluginHub：负责事件的存储，合成事件以对象池的方式实现创建和销毁，大大提高了性能。（核心）
SimpleEventPlugin等plugin：根据不同的事件类型，构造不同的合成事件。

看到这我们应该很容易的能回答之前的问题了，以及开发中遇到的和事件相关的坑大家应该知道为什么了吧。