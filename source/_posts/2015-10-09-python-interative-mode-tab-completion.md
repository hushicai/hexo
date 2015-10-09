title: "让python在交互模式下支持tab completion"
date: 2015-10-09 17:12:29
tags:
s: python interative mode tab completion
---

本人使用的是mac os x 10.10.5版本，shell是zsh。

zsh的自定义配置在~/.oh-my-zsh/custom/my.zsh中。

首先在`~/.oh-my-zsh/custom/my.zsh`中配置如下：

```bash
export PYTHONSTARTUP=$HOME/.pythonrc.py
```

_Note：如果你的zsh没有自定义配置文件，你可以直接在`~/.zshrc`配置。_

<!-- more -->

然后在个人主目录下新建一个`~/.pythonrc.py`z文件：

```python
try:
    import readline
except ImportError:
    print("Module readline not available.")
else:
    import rlcompleter
    readline.parse_and_bind("tab: complete")
```

新开一个ternimal窗口，让以上配置生效即可。

效果如下：


{% asset_img 1.gif %}

这样练习起来就方便很多了！
