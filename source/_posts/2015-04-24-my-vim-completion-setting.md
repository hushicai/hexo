---
layout: post
title: "my vim completion setting"
categories:
    - vim
tags:
    - vim
    - completion
    - youcompleteme
    - supertab
    - ultisnips
    - tern
description: ""
---

本人从事web前端开发，一直用vim作为开发工具。

最近一直在折腾completion这事，先后尝试了`youcompleteme`、`neocomplete`等工具，甚至直接使用vim内置的`ins-completion`，不过都不尽人意。

一顿折腾之后，总算弄了一套自己用着还算顺手的配置。

我期望completion可以干以下这些事

- 标识符补全
- 智能补全（omni completion）
- 路径补全
- snippets

这些补全其实`ins-completion`已经可以搞定了，但是比较费劲，`<c-x><c-o>`、`<c-x><c-f>`...好多按键...

所以有了`youcompleteme`、`neocomplete`等工具。

试用了一番，`neocomplete`比较卡，放弃了，最后选择了`youcompleteme`。

## youcompleteme

基本配置如下：

```vim
let g:ycm_min_num_of_chars_for_completion = 3 
let g:ycm_autoclose_preview_window_after_completion=1
let g:ycm_complete_in_comments = 1
let g:ycm_key_list_select_completion = ['<tab>', '<c-n>', '<Down>']
let g:ycm_key_list_previous_completion = ['<c-p>', '<Up>']
let g:ycm_confirm_extra_conf = 0
```

<!-- more -->

标识符补全：

{% asset_img 1.png %}

ycm提供了一个`identifier-based
completer`，它会收集所有当前文件或者访问过的文件中的identifiers，当你输入时，ycm会查找它们，并提供completion。

路径补全：

{% asset_img 2.png %}

ycm提供了一个`filepath completer`来补全路径。

智能补全：

{% asset_img 3.png %}

ycm会为c-family语言、python等提供内置的completer，但对于javascript、php等语言，则使用vim的omnicompletion系统。

如上图所示，ycm能够正常的提供completion，不过，它只会在点号(.)之后才能开启补全，也就是说它不会补全全局预定义的变量/函数。

可能有人会问：第一张图中不是已经补全了吗？

没错，是补全了，但那不是`omni completion`提供的，而是ycm的identifier completer提供的，对于在buffer中已经出现的标识符，ycm会把它们当作候选项。

看一下这种情况：

{% asset_img 4.png %}

`document`是dom提供给javascript的全局变量，但是这里没补全，显然ycm此时并没有启动omni completion。如果你用`<c-x><c-o>`手动触发omni completion，则可以得到补全。

{% asset_img 5.png %}

查了一下ycm的文档，发现有一个选项可以来控制是否触发omni completion:

```vim
let g:ycm_semantic_triggers = {'javascript': ['.']}
```

但是设置之后不管用，ycm其实已经移除了这个配置，但是文档没更新，太坑了！！！

为了能更快捷地触发全局变量的智能补全，我定义了两个快捷键：

```vim
function! MyTabFunction ()
    let line = getline('.')
    let substr = strpart(line, -1, col('.')+1)
    let substr = matchstr(substr, "[^ \t]*$")
    if strlen(substr) == 0
        return "\<tab>"
    endif
    return pumvisible() ? "\<c-n>" : "\<c-x>\<c-o>"
endfunction
inoremap <tab> <c-r>=MyTabFunction()<cr>
inoremap <c-o> <c-x><c-o>
```

为啥要定义两个？主要是为了处理以下两种情况：

1. 没有候选项（completion-menu不显示）
2. 有候选项，但不是智能补全的候选项，即completion-menu上的候选项是来自其他completer的，比如identifier、snippets

第一种情况，可以直接使用`<tab>`来触发智能补全并选择候选项，但是第二种情况下，候选项不是我们想要的智能补全选项，我们需要重新触发智能补全，而此时`<tab>`是充当选择候选项的功能，并不适用，所以就再定义一个`<c-o>`快捷键（当然你也可以直接用`<c-x><c-o>`）。

ok, 现在我们只需要用`<tab>`或者`<c-o>`就可以强制触发智能补全了。

## ultisnips

`ultisnips`可以很好地配合ycm，提供snippets补全。

安装了`ultisnips`之后，ycm有个配置`g:ycm_use_ultisnips_completer`可以来控制是否接收`ultisnips`的补全，默认为1。

{% asset_img 6.png %}

到目前为止，completion看起来好像已经很友好了。

其实不然，有些全局预定义函数/变量没有补全，比如`setTimeout`、`setInterval`等：

{% asset_img 7.png %}

一些html5提供的新方法也没有得到补全。

这个时候，我们可以定义`dictionary`来补全：

```vim
set complete+=k
set dictionary+=~/data/github/dotfiles/vim/dictionary/javascript.dict
```

设置完`dictionary`之后，用`<c-x><c-k>`就可以得到补全。

不过，很遗憾，ycm并不会把dictionary加到候选项中。

## tern

javascript作为一门动态类型语言，其本身的工程化能力差，omni
completion能提供的补全功能有限，比如无法补全类成员、其他文件的函数定义等。

tern应运而生，它是一个javascript代码分析引擎，和一个代码编辑器插件一起使用，可以增强该编辑器对智能化javascript编程的支持，提供了以下特性：

- 变量和属性自动补全
- 函数参数提示
- 查询一个表达式的类型
- 查找某些函数/变量的定义
- 自动化反射

tern有一个`tern_for_vim`插件，用来增强vim对智能化javascript编程的支持。

安装`tern_for_vim`之后，配置：

```vim
let g:tern_show_signature_in_pum = 1
```

_ps: 具体怎么安装详见[tern doc](http://ternjs.net/doc/manual.html)。_

再配置以下`.tern-project`:

```json
{
    "libs": [
        "browser",
        "jquery",
        "ecma5"
    ],
    "plugins": {}
}
```

completion现在变成这样：

{% asset_img 8.png %}

还可以补全jquery：

{% asset_img 9.png %}

有了tern，我们现在也可以补全setInterval/setTimeout等全局函数了：

```text
set[tab]
```

效果如下：

{% asset_img 10.png %}

很高大上，有木有。

有了tern，敲javascript代码，腰不酸了，腿不疼了，实乃居家码字之首选！
