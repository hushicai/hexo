title: win7将CapsLock映射为Ctrl
s: win7 jiang CapsLock ying she wei Ctrl
date: 2016-01-07 23:52:13
tags:
- windows
- Caps Lock
- Ctrl
---

从狼厂离职之后,没有了mac,一直在用windows系统,以sublime作为开发工具.

最近实在受不了,装了个ubuntu虚拟机,回到vim.(暂时还不想买mac...)

作为一个vim党,快速移动手指很重要,很显然,键盘上的CapsLock键实在是太浪费了,必须把它搞成Ctrl键.

这有个windows注册文件,内容如下:

```text
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,02,00,00,00,1d,00,3a,00,00,00,00,00 
```

将以上内容保存到一个reg文件中,比如`caps_lock_to_ctrl.reg`,双击注册到windows注册表,然后重启电脑即可.

## 参考文章

- http://johnhaller.com/useful-stuff/disable-caps-lock
- http://vim.wikia.com/wiki/Map_caps_lock_to_escape_in_Windows#Use_the_Caps_Lock_key_as_Ctrl
