---
title: atom前端实用插件
date: 2017-05-24 16:04:37
tags:
- atom
categories:
- 工具
author: Rock
---


工欲善其事，必先利其器。

今天给大家推荐atom编辑器的插件安装教程和一些前端实用插件~

### 查看已安装插件

在terminal输入 
`
apm ls
`


```
Built-in Atom Packages (xxx)
.........省略n多内置插件

下面是已安装的插件
Community Packages (10) /Users/rock/.atom/packages
├── atom-beautify@0.29.24
├── atom-ctags@5.0.0
├── autocomplete-plus@2.35.4
├── colorpicker@0.3.0
├── csscomb@0.3.1
├── emmet-atom@2.4.3
├── javascript-snippets@1.2.1
├── minimap@4.28.2
├── nuclide@0.166.0
└── scss-snippets@0.6.0
```

### 安装插件

可以先去
[atom安装包搜索页](https://atom.io/packages)
找到自己喜欢的插件后

1.
`
apm install packageName
`

2.
```
Installing autoprefixer to /Users/rock/.atom/packages ✓(这个勾说明已经安装成功了)
```

3.
`
apm uninstall packageName (从入门到放弃)
`

是不是很简单呢？


下面有同学吐槽说 apm install 安装失败了怎么办?

![alt](http://imgsrc.baidu.com/forum/w%3D580/sign=39392c6bab773912c4268569c8188675/aa0229f33a87e950fa1099ba12385343faf2b4e7.jpg)

好吧 我再讲讲别的安装方法

1.
`
输入cd ~/.atom/packages/ (atom路径下packages目录)
`

2.
找到插件地址后进入git repo跳转的链接
[minimap](https://atom.io/packages/minimap) -》 [git地址](https://github.com/atom-minimap/minimap)

3.
用git clone 插件目录
`
git clone https://github.com/atom-minimap/minimap.git
`

4.
然后可以直接在`~/.atom/packages/`目录下apm install minimap

或者在packages目录进入package后直接安装 `cd minimap && apm install`

5.
`rm -rf ~/.atom/packages/minimap`(暴力删除方法)



### 实用插件推荐

*    [emmet](https://atom.io/packages/emmet)
		zend-coding
*    [autoprefixer](https://atom.io/packages/autoprefixer)
		样式兼容补全
*    [minimap](https://atom.io/packages/minimap)
		抄袭sublime的代码小地图
*    [javascript-snippets](https://atom.io/packages/javascript-snippets)
		js代码自动补全
*    [react-snippets](https://atom.io/packages/react-snippets)
		react 补全
*    [autocomplete-plus](https://atom.io/packages/autocomplete-plus)
		js代码补全建议
*    [terminal-plus](https://atom.io/packages/terminal-plus)
		terminal for atom
*    [atom-beautify](https://atom.io/packages/atom-beautify)
		代码格式化
*    [colorpicker](https://atom.io/packages/colorpicker)
		取色器(好像有bug)
*    [hyperclick](https://atom.io/packages/hyperclick)
		传伟同学推荐的文件跳转
*    [eslint](https://atom.io/packages/eslint)
		eslint 提示
		

		



