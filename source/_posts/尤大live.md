---
title: 尤大live笔记
date: 2017-06-04 15:30:36
tags: browser
categories: browser
author: peanut
authorLink:
---
### vue2.0 源码
[vue2.0 源码学习](http://hcysun.me/2017/03/03/Vue源码学习/)。
### 框架选型
框架选型不谈场景都是刷流氓，不同的场景下、开发者群体、偏好不一致，多种方案的并存是必然的有益的。
如何选型：经验和对框架工具背后想要解决的问题深入理解
### 组件
主流框架都以组件作为基本的抽象单元，组件的发展史 page->application，模块封装.
react 揭示了组件可以是函数，
组件的分类
- 接入型 container
- 展示型 component
- 交互型 比如各类加强版的表单组件，通常强调复用
- 功能型 比如 `<router-view>`，`<transition>`，作为一种扩展、抽象机制存在。 

模板和 jsx 对比，jsx 灵活性高，模板在以展示型为主的场景下不差，jsx 在功能性组件的书写远超模板
colocation: 把该放的东西放一起和以前的 html、js、css 分离相对应
### 变化侦测机制和渲染机制
渲染中的申明式和命令式
命令式的维护问题，申明式申明数据和 dom 的映射关系，不需要手动去维护操作
view = render(state)
vue的模板也是编译成渲染函数，模板和 jsx 的本质是一样的，具体实现可以是 vdom 也可以是细粒度的绑定如 vue1；
变化侦测：[变化侦测 ppt](https://docs.google.com/presentation/d/1_BlJxudppfKmAtfbNIcqNwzrC5vLrR_h1e09apcpdNY/edit#slide=id.g19eebb1966_0_346)。
<div onclick="clickHandler"></div> 被人诟病，为啥 vue  的声明式写法就是推崇的？ 前者的作用域的全局的，vue 模板是在特定的作用域中。另外的原因是：colocation，jsx 或模板中处理器是在相关联的文件中，还是维护性问题。
变化侦测分为push 和 pull，react 中的setState和 angular1中的脏检查都是pull，对于数据的变化，要有钩子来触发更新，暴力的比对如 react 中的 vdom 的 diff，暴力的遍历比对，找出了变化的部分；push如 vue 的响应式和 rxjs，在数据变动后，能确切的知道具体什么变了，相对细粒度的更新，pull 是粗粒度的，所以要帮助 pull 来提高性能，如 purecomponent 、shouldComponenetUpdata等，push 是细粒度的，我们知道的信息多一点，代价就是每一个绑定都要有 watch，带来内存的开销和依赖追踪的开销。
vue2选择中等粒度的更新，组件级别是 push，每一个组件是一个响应式的 watcher，当数据变化的直接知道具体哪些组件变了，组件内部是 vdom 的比对
本质的区别是：用侦测成本换一定程度的自动优化，各有利弊
### 状态管理
flux reflux redux mobx vuex rxjs
本质：从原事件映射到状态的改变再到 ui 的变化，申明式的渲染已经解决了状态到 ui 的变化，状态管理就是如何管理将事件源映射到状态变化的这个过程，如何将映射的过程从视图组件剥离出来，如何组织这部分代码，提高维护性，这是状态管理要解决的问题。
redux 和 mobx，两种范式： redux 函数式、immutable；mobx 和 vuex数据可以变，响应式，已经做好了申明式的副作用
[把 Vue 当 redux 用](https://jsfiddle.net/yyx990803/0a22ojps/)
[把 Vue 当 MobX 用](https://jsfiddle.net/yyx990803/f5a24dk3/)
都没有回答如何处理异步，redux 中间件生态，vuex 和 mobx action 中搞
简单的 curd 应用没必要，复杂的如实时服务端推送，竞态，rxjs
组件的全局状态和局部状态如何区分，没有很明显的界限，Elm 中的解决方案
### 路由
url 映射到组件树的过程
react-router 4 用组件自身来做路由，功能性组件，路由表分散，耦合在组件中
web 路由和 app 路由
### css 方案
主流的 CSS 方案
- 跟 JS 完全解耦，靠预处理器和比如 BEM 这样的规范来保持可维护性，偏传统
- CSS Modules，依然是 CSS，但是通过编译来避免 CSS 类名的全局冲突
- 各类 CSS-in-JS 方案，React 社区为代表，比较激进
- Vue 的单文件组件 CSS，或是 Angular 的组件 CSS（写在装饰器里面），一种比较折中的方案

传统 css 的一些问题：
1. 作用域
2. Critical CSS
3. Atomic CSS
4. 分发复用
5. 跨平台复用

[css-in-js](https://speakerdeck.com/vjeux/react-css-in-js)

### 构建工具
grunt gulp webpack
构建工具解决的其实是几方面的问题：
- 任务的自动化
- 开发体验和效率（新的语言功能，语法糖，hot reload 等等）
- 部署相关的需求
- 编译时优化

[关于部署](https://www.zhihu.com/question/20790576)

### 服务端数据通信
传统restful接口、数据直连、实时推送
实时推送方案：Meteor
数据关联： GraphQL,复杂数据的关联比 restful 表达力更强,

### 服务端渲染
[vue 服务端渲染](ssr.vuejs.org)

### 跨平台渲染
weex rn, 框架的渲染机制与 dom 解耦，不一定要 vdom，把框架节点操作与原生的渲染引擎对接

### 新规范
[web component新规范](https://www.zhihu.com/question/58731753)