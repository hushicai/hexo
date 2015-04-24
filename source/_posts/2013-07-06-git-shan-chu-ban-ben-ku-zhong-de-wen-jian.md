---
layout: post
title: "git删除版本库中的文件"
description: ""
---

已经添加到版本库的某个文件，应该怎么删除呢？

* 使用git删除命令

```bash
git rm <file>
```

运行该命令后，git会提示：

```text
rm '<file>'
```

再执行 `svn status`，可以看到文件已从暂存区删除，并等待 `commit`。

* 手动删除文件

```bash
rm <file>
```

然后再从git暂存区中把该文件删除即可：

```bash
git reset HEAD <file>
```

或者可以使用以下方式从暂存区删除文件：

```bash
git add -u
```
