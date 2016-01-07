title: 让vim支持`+clipboard`
s: rang vim zhi chi `+clipboard`
date: 2016-01-08 01:20:40
tags:
- vim
- clipboard
---

ubuntu中系统默认的vim真是折腾,竟然不支持`+clipboard`.

如果没有开启clipboard特性,那vim就没法和系统的粘贴板交互,也就是说我们用`"+y`命令复制之后,没办法粘贴到其他地方,比如浏览器等.

`vim-gnome`这个包给vim提供了`+clipboard`特性.我们只需要安装一下就可以了.

```bash
sudo apt-get install vim-gnome
```

然后再查看一下,应该已经成功开启了:

```bash
vim --version | grep clipboard
```

## 参考文章

- http://vimcasts.org/blog/2013/11/getting-vim-with-clipboard-support/
