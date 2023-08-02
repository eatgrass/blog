---
date: 2023-07-30
title: 终端如何工作
tags: System
categories: Computer Science
---

终端(**Terminal**)，对于工程师来说算是常用工具， 但是我发现自己并不真正了解终端是如何工作的，而且在GUI做为主流交互方式的今天，这或许算是一个普遍现象，大家并不关心终端后面各种奇怪的配置是怎么回事，只要能帮我正常执行命令就好。 我甚至分不太清**Terminal**和**Shell**到底是什么关系，直到前些天在折腾nvim时碰到一些问题，才想起或许该好好了解一下相关的知识。最后不仅顺利解决了问题，之前一直模模糊糊的地方也清晰了。

## 从Teleprinter说起

这个外观上看上去像一台打字机的东西就是{{% link href = "https://en.wikipedia.org/wiki/Teleprinter" text = "Teleprinter" %}}，历史比较悠久，出现的年代比计算机还要再早个100年。 曾经被用于电报系统，后来被用作连接计算机的IO设备，通过键盘进行输入， 打字机再将计算结果一个字符一个字符的打印到纸带上。

它还有另外一个名字更被人所熟知  **TTY**，打开系统的`/dev`目录，你会发现一些以tty开头的设备文件，每个文件代表着某个终端在通过`stdin`，`stdout`，`stderr`在于某个进程交互。曾经这些设备文件的另一端是一台真实的终端， 现在Teleprinter的肉身可能只存在于博物馆里了，但是tty这个名字则被沿用至今。

另外一个有趣的地方是，我在初学编程时，看到`printf("Hello World\n")`，会有点纳闷，这个`print`多少看着有点奇怪，但在了解到当时的输出设备确实是台打字机的时候，那也就合理了。

## Terminal

再后来的终端则是用显示器代替了纸带打印，但是GUI还未出现，物理终端仍作为对接`tty`输入输出的{{% link href = "https://en.wikipedia.org/wiki/Device_file#Character_devices" text = "字符设备" %}}，此时也出现了各种类型的终端，根据协议大致可分为两类，一类兼容{{% link href = "https://en.wikipedia.org/wiki/ANSI_escape_code" text = "ANSI协议" %}}，例如xterm, xterm-color256, linux, VT100等，而另一类是非ANSI协议，代表有IBM3270。

交互过程中，这类真实终端会告诉计算机自己的类型，计算机不仅会向终端输出字符内容，还会使用预先定义特殊序列字符来控制终端，如移动光标，设置前景色、背景色等。

## 终端模拟器

我们今天称之为终端的软件，例如iTerm, Windows Terminal, Alacritty 甚至包括Tmux，严格意义上其实是终端模拟器，我们通过环境变量`$TERM`来控制这些软件所要模拟的终端，不同的终端所支持的能力不同，比如我们可能会把终端类型环境变量设成xterm-color256一类的值来支持色彩显示。

终端模拟器又是如何实现对终端的模拟的呢？一般来说，模拟器会尽可能多的支持各种能力，而不是直接实现某种终端。对终端能力的抽象由系统的终端数据库来完成，不同的系统可能会使用
{{% link href = "https://man7.org/linux/man-pages/man5/terminfo.5.html" text = "terminfo" %}}或者`termcap` 作为终端数据库。

我在这里截取部分{{% link href = "https://www.commandlinux.com/man-page/man1/infocmp.1.html" text = "infocmp" %}}命令的输出。

```plaintext
    alacritty|alacritty terminal emulator,

    cursor_left=^H, cursor_normal=\E[?12l\E[?25h,
    cursor_right=\E[C, cursor_up=\E[A,
    cursor_visible=\E[?12;25h, delete_character=\E[P,
    delete_line=\E[M, dis_status_line=\E]2;\007,
    enter_alt_charset_mode=\E(0, enter_am_mode=\E[?7h,
    enter_bold_mode=\E[1m,

    key_f9=\E[20~, key_home=\EOH, key_ic=\E[2~,
    key_left=\EOD, key_mouse=\E[M, key_npage=\E[6~,
```

拿`cursor_left=^H`这行来说,是指当终端收到`<Ctrl>H`序列时,需要将光标左移一个位置。

````ascii-diagram
┌──────────────────────────────┐
│                              │
│   ┌──tty───┬────────────┐    │
│   │     in ┤  attached  │    │
│   │        │  process   │░░  │
│   │err/out ┤shell,ssh.. │░░  │
│   └──────┬─┴────────────┘░░  │
│     ▲░░░░│░░░░░░░░░░▲░░░░░░  │
│     │    │          │        │
│   stdin stdout/err  │        │
│     │    │          │        │
│     │    ▼          │        │
│   ┌─┴────────┐     folk      │
│   │          │      │        │
│   │   term   │░░    │        │
│   │          ├──────┘        │
│   └──────────┘░░             │
│      ░░░░░░░░░░░             │
│                              │
└──────────────────────────────┘
````

## SSH

那么SSH远程到主机的过程也就容易理解了

1. 本地终端启动shell进程，与shell标准输入输出进行交互
2. 在shell中使用ssh客户端连接到远端主机的ssh服务器时，会设置连接{{% link href = "https://www.openssh.com/txt/release-8.7" text = "TERM环境变量" %}}，需要使用的shell
3. 远程shell将输出转为合适的终端字符序列，通过ssh发送给本地

当连接到远程主机，想将光标左移时，我们在键盘上按下 {{% kbd key = "Left"%}}，终端会查找`key_left`所对应的字符序列，并将序列`<ESC>0D`发送给远程主机，本地终端在收到`<Ctrl>H`后，将光标完成左移。

所以从触发按键到屏幕上光标完成移动，实际上经过了一个完整网络收发过程，如此也就解释了为什么在远程主机上遇到资源瓶颈造成的卡顿或网络延迟会直接反映到本地的终端上。
