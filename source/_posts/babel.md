---
title: "babel拾遗"
date: 2017-06-3 19:08:33
tags: 
- babel
categories: 
- workflow
author: 布谷
---

我们在使用 `babel`的时候，常常搞不清楚需要安装哪些依赖，babel本身繁杂的体系也容易让人混淆，这里就自己看到的部分稍作解释，欢迎补充

### 1、 `.babelrc`

这是 babel 的配置文件，rc 表示自动执行，主要配置项有：`presets`, `plugins`, `ignore`等; 示例：
```
{
  "presets": ["es2015", "react"],
  "plugins": ["transform-decorators-legacy"]
  "ignore": ["lib/*.js"]
}
```
其中`presets`是一系列`plugins`的集合。
例如，编译 react 项目中的 jsx 语法，既可以使用 plugin: `babel-plugin-transform-react-jsx`，也可以使用包含该 plugin 的 preset: `react`

### 2、 `babel-core`
看名字，`babel-core`似乎是必备的依赖，其实`babel-core`提供了一些编译方法供**手工调用**；也就是说，我们常规的使用 webpack 或者 babelrc 的业务项目，是不需要安装`babel-core`的依赖的

### 3、 `babel-polyfill`
OK，说道正题了, 首先我们知道 babel 的作用，是用来编译es6，es7 等浏览器尚未全面支持的新语法和 api。
实际上呢，babel 为它的编译功能做了分类：
- presets 和 plugins 负责编译箭头函数等新语法
- babel-polyfill负责编译新的 api 如：Promise，Set等；以及定义在全局对象上的方法如 Object.assign

babel-polyfill 实现编译新 api 以及全局方法的方式是污染全局变量，以及修改原生对象的 prototype，这在一般的业务项目中，并不会引起太大的问题。
但是如果我开发的是一个库，可能被引入到各种第三方应用中，这就大大的不好了；所以不嫌麻烦的 babel 又给大家准备了新套餐 `babel-runtime`

### 4、 `babel-runtime` 与 `babel-plugin-transform-runtime`
`babel-runtime` 的业务范围和 `babel-polyfill` 一致（稍有不同，后面会说到），不过使用的方式是提供 helper 函数，嘛意思呢，也就是说，
如果你使用Promise，那么我定义一个叫做Promise的对象，供你在需要的地方引入；
如果你使用Object.assign, 那么我写一个同样功能的方法，供你引入使用

`babel-runtime`通过这种方式，避免污染全局变量和修改原生对象的prototype，例如，下面是一个 babel-runtime 对 `{ [name]: 'JavaScript' }` 的编译后代码：
```
'use strict';
function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true
    });
  } else {
    obj[key] = value;
  }
  return obj;
}
var obj = _defineProperty({}, 'name', 'JavaScript');
```

OK, 现在我们知道`babel-runtime`的作用了，但是我们总不能在业务代码中一个个手工引入这些 helper 函数吧，所以 babel 提供了 `babel-plugin-transform-runtime`的插件，用于编译时自动引入 helper 函数。


并且 `babel-plugin-transform-runtime` 本身依赖了 `babel-runtime`，所以在项目中使用的时候不需要再额外指定 `babel-runtime` 的依赖


思考题：
对于实例方法，如 `"foobar".includes("foo")`， `babel-runtime`可以实现编译吗？

答案是不可以，因为实例方法显然必须通过修改原生对象的prototype才能实现

### 5、合理的配置
我们现在来看看最佳的babel配置是怎样的

- presets 和 plugins 可以冗余一点，babel会按需使用
- 如果用到 Promise, Set，Map 等新api，增加 `babel-plugin-transform-runtime` 的依赖
- 如果使用到类似 `"foobar".includes("foo")` 等实例方法，额外引入对应的 polyfill（参考：https://github.com/zloirock/core-js）

最后，babel的文档真特么烂！


