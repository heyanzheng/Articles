#【原创】修改CentOS终端命令配色

本人使用的是iTerm2，所以也只介绍如何更改系统终端和iTerm2。

## 修改CentOS终端配色
使用root或用户名登陆后，默认进入~目录，修改.bash_profle：

	# vi .bash_profile
打开后，在文件后面输入以下命令：

	export CLICOLOR=1
	export LSCOLORS=gxfxcxdxbxegedabagacad
	export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]\$ '
	export TERM=xterm-color

> LSCOLORS 是定义ls命令输出结果颜色的，格式为每两个字母一组，分别设置某个文件类型的文字颜色和背景颜色；网上有个工具可自定义生成：http://geoff.greer.fm/lscolors/<br>
> PS1是终端提示符的格式定义

## 修改iTerm2配色
1. 为使上面的生效，iTerm2可能需要修改 Preferences -> 相应的Profiles -> Terminal 中为xterm-256color或xterm-new，具体哪个我自己感觉系统不一样这里的选择也不一样。大家都试试看，看哪个会生效。

2. Preferences -> 相应的Profiles -> Colors中可以"Load Presets"自定义的样式来使用。目前网上有很多关于iTerm的主题，这里推荐 [Github iTerm-2-Color-Themes](https://github.com/baskerville/iTerm-2-Color-Themes) 。下载下来后直接Load即可。

3. 记得重启下。

