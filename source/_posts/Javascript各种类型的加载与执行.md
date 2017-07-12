---
title: Javascript各种类型的加载与执行
date: 2017-07-12 14:10:27
tags: browser
categories: browser
author: yoution
---

# Javascript 各种类型的加载与执行顺序
1. script src + defer
```html
<script src="xx" defer></script>
```
  是否阻塞parse: 不阻塞   
  执行时机：parse结束后待js文件全部加载完成后，按照script出现的顺序执行，全部执行完触发DOMContentLoaded事件

2. script type=module + src
```html
<script type=module src='xx'></script>
```
  同1

3. script type=module + 代码块
```html
<script type='module'>
  code
</script>
```
  同1,虽然代码已经在本地，还是会在下个时间点的下个时间点执行

4. script src
```html
<script src="xx"></script>
```
  是否阻塞parse: 阻塞, 停止parse，但是会触发preLoad   
  执行时机：停止parse，等待js文件加载完成后，如果前面没有css文件加载，立即执行js，恢复parse;如果前面有css文件加载,则等待css加载完毕，解析css完成后，立即执行js，恢复parse。preLoad 的css 的加载不会影响到js

5. script src + async
```html
<script src="xx" async></script>
```
  是否阻塞parse: 不阻塞, 继续parse   
  执行时机：js文件加载完成后，下一个时间点执行js(相当于setTimout(fn, 0))

6. script 代码块, 前面没有css文件在加载
```html
<script>
  code
</script>
```
  是否阻塞parse: 不阻塞, 执行完代码继续parse   
  执行时机：立即执行js，执行完继续parse

7. script 代码块, 前面有css文件在加载
```html
<script>
  code
</script>
```
  是否阻塞parse: 阻塞, 同4，停止parse，但是会触发preLoad   
  执行时机：待css文件加载完毕，解析完css后，立即执行js，执行完继续parse

8. 动态插入的script标签
```html
  document.append(script)
```
  同5,动态标签不受defer，async，type='module'影响，全部async

9. script type=module + defer
```html
<script type=module defer></script>
```
  同5，async优先级高于defer

10. script async + defer
```html
<script src='xxx' async defer></script>
```
  同5

11. script 代码块 + defer
```html
<script defer>
  code
</script>
```
  script代码块不支持 defer,同6或7

12. script 代码块 + async
```html
<script async>
  code
</script>
```
  script代码块不支持 async,同6或7

13. script 代码块 + src
```html
<script src='xxx'>
  code
</script>
```
  忽略代码块中的code，优先src

