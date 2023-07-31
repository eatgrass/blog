---
date: 2023-07-30
title: 终端如何工作
---

虽说天天跟终端(Terminal)打交道, 但是我发现自己并不真正了解终端是如何工作的, 而且在GUI做为主流交互方式的今天, 这算是一个普遍现象, 大家并不关心终端后面各种奇怪的设置是怎么回事, 只要能帮我正常执行命令就好. 我甚至分不太清Terminal, Shell到底是什么关系, 直到前些天在折腾nvim时碰到一些问题, 才想起或许我该好好了解一下这个老朋友, 而且最后也证明了解这些知识确实非常有益, 不仅顺利解决了问题, 之前一直模模糊糊的地方也清晰了起来.

## 从Teleprinter说起

在非常早的年代, 大家是使用一种类似打字机的东西作为计算机的输入输出设备的, 当时没有显示器, 计算机输出的结果就由它一个一个字符打印到纸带上的, 这个名字很形象, 一个Tele到Computer的Printer, 也叫TeleTypeWriter 或者另外一个你可能眼熟的名字`tty`, 当你查看你的`/dev`目录时就会发现一些tty开头的设备文件. 每个文件关联着我们的某个进程, 我们的终端通过`stdin`, `stdout`, `stderr`与进程交互. 没错, Teleprinter肉身可能在博物馆里才能找到了, 今天代替它的是各种运行于图形界面下的终端模拟器, 但是tty这个名字则被沿用至今.

另外一个有趣的地方是, 我在初学C编程`printf("Hello World\n")`的时候, 就有点纳闷, 这个`print`多少看着有点奇怪, 但在了解到当时的输出设备确实是台打字机的时候, 那就合理了.

## Terminal

再后来则是显示器代替了纸带打印, 但是GUI还未出现, 显示器仍作为对接`tty`输出的{{% link href = "https://en.wikipedia.org/wiki/Device_file#Character_devices" text = "字符设备" %}}, 此时也出现了各种类型的终端, 根据协议大致可分为两类, 一类兼容{{% link href = "https://en.wikipedia.org/wiki/ANSI_escape_code" text = "ANSI协议" %}}, 例如xterm, xterm-color256, linux, VT100等, 而另一类是非ANSI协议, 代表有IBM3270.

这类真实的终端的交互过程是, 终端会告诉计算机终端的类型, 计算机不仅会向终端输出字符内容, 还会使用预先定义特殊序列字符来控制终端如何显示, 如移动光标, 前景色, 背景色都是靠特殊的序列字符来实现的.

## 终端模拟器

我们今天称之为终端的软件,例如iTerm, Windows Terminal, Alacritty 甚至包括Tmux, 严格意义上应该叫终端模拟器, 我们通过环境变量`$TERM`来控制这些软件所要模拟的终端, 那么不同的终端所支持的能力不同, 比如我们可能会把终端类型环境变量设成xterm-color256一类的值来支持显示不同的颜色.

终端模拟器又是如何来模拟这些终端的呢? 一般来说, 模拟器不会直接实现某类终端, 而是读取定义在系统中的终端数据库, 不同的系统可能会使用
{{% link href = "https://man7.org/linux/man-pages/man5/terminfo.5.html" text = "terminfo" %}}或者`termcap`, 这些数据库里定义了不同类型的终端所具备的能力和使用的序列.

以下截取我的termcap输出

```plaintext
alacritty|alacritty terminal emulator,
        am, bce, ccc, hs, km, mc5i, mir, msgr, npc, xenl,
        colors#256, cols#80, it#8, lines#24, pairs#32767,
        acsc=``aaffggiijjkkllmmnnooppqqrrssttuuvvwwxxyyzz{{||}}~~,
        bel=^G, bold=\E[1m, cbt=\E[Z, civis=\E[?25l,
```

## SSH

那么SSH连接到远程主机的过程也就容易理解了

1. 本地终端启动shell进程, 与shell标准输入输出进行交互
2. 在shell中使用ssh客户端连接到远端主机的ssh服务器时,会设置连接{{% link href = "https://www.openssh.com/txt/release-8.7" text = "TERM环境变量" %}}, 需要使用的shell
3. 远程shell输出为特殊序列的格式, 通过ssh链接发送给本地终端

一张大图概括

````ascii-diagram

   ┌─────────────┐
   │Local        │
   │  Todo       ├─────▶
   │ ▒▒▒▒▒▒▒ ▒▒▒ │
   └─────────────┘
````


