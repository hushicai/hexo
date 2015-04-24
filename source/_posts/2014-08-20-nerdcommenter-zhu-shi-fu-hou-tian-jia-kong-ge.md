---
layout: post
title: "nerdcommenter注释符后添加空格"
tags:
    - nerdcommenter
    - space
    - 空格
description: ""
---

在vim中使用nerdcommenter注释时，我们一般用`<leader>c<space>`来自动添加或者去掉注释。

假设代码如下：

```javascript
var name = "hushicai";
```

将鼠标移到该行代码上，用`<leader>c<space>`进行注释，注释后应该是这样的：

```javascript
//var name = "hushicai";
```

然而，这可能会不符合某些团队的代码规范，比如我们团队，进行jshint之后，会提示这样的warning：

```bash
edp WARN → line 1, col 0: Missing space after line comment
```

意思就是要在单行注释的注释分隔符后面加个空格：

```javascript
// var name = "hushicai";
```

好吧，既然是规范，那我就不得不遵守了。

看了一眼nerdcommenter的文档，发现有这么一个配置可以解决问题：

```vim
let NERDSpaceDelims=1
```

ok，现在我们再使用`<leader>c<space>`进行自动注释，就可以按照规范在注释符后加一个空格了。
