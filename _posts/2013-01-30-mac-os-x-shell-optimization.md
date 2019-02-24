---
layout: post
title: "Mac OS X Shell 优化"
description: "Mac下的Shell默认是白底黑字并且没有linux下各种色彩区分的，这对于用习惯了linux下shell的童鞋们来讲实在是太不舒服了！"
category: mac
tags: [mac, shell]
---

Mac下的Shell默认是白底黑字并且没有linux下各种色彩区分的，这对于用习惯了linux下shell的童鞋们来讲实在是太不舒服了！

<img src="http://ww3.sinaimg.cn/mw690/713d9449gw1e3pa9xfv1fj.jpg">

本着折腾的精神，探索更完美的Shell使用体验之道，Let’s begin！

##第一步
修改终端配色方案
终端 -> 编好设置 -> 高级 -> 声明终端为：xterm-256color
终端 -> 编好设置 -> 描述文件 -> Homebrew (要设置为默认的啦)
<img src="http://ww4.sinaimg.cn/mw690/713d9449gw1e3pa9u5x95j.jpg">

这个时候打开终端已经能初见成果了，虽然文字依然只有一种颜色，但底色至少不是白色那样单调了

##第二步
给当前用户增加配置文件，修改路径，提示符颜色，以及打开ls命令的高亮显示。

	cd ~
	vim .bash_profile

保存以下内容

	# change shell color
	export CLICOLOR=1
	export LSCOLORS=gxfxaxdxcxegedabagacad
	# grep
	alias grep='grep --color=always'

之后再重新打开终端之后，就有如下效果了

<img src="http://ww3.sinaimg.cn/mw690/713d9449gw1e3pa9wmqzsj.jpg">

大功告成，享受你的shell吧！