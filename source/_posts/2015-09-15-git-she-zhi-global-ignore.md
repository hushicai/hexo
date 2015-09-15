title: "git设置global ignore"
date: 2015-09-15 17:28:26
tags:
---

在个人主目录下新建一个`~/.gitignore`文件，填上想全局忽略的文件：

```bash
cscope.files
cscope.out
```

然后在`~/.gitconfig`中配置一下就行了：

```bash
git config --global core.excludesfile ~/.gitignore
```

执行完以上命令后，在`~/.gitconfig`中会添加一个新项目：

```gitconfig
[core]
	excludesfile = /Users/hushicai/.gitignore
```

搞定！
