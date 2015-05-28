title: "删除git submodule"
date: 2015-05-28 22:11:06
tags:
s: delete git submodule
---

git添加submodule时，很酸爽，但是删除时，很蛋疼。

git貌似没有直接提供删除submodule的方法，如果要删除submodule，需要以下步骤：

* 删除`.gitmodules`中对应的节点
* 添加`.gitmodules`到暂存区，`git add .gitmodules`
* 删除`.git/config`中对应的节点
* 删除暂存区缓存，`git rm --cached path_to_submodule`
* 删除`.git/modules/path_to_submodule`，`rm -rf .git/modules/path_to_submodule`
* 删除工作区的submodule，`rm -rf path_to_submodule`

挺繁杂的......
