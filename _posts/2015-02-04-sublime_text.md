---
layout: post
title: SublimeText
category: 工具
tag: [SublimeText]
---

### 我的一些配置（持续更新）

我的sublime_text配置都放在github上了，[github地址](https://github.com/lbrb/User)
如果想直接使用我的配置可以执行下面的两行命令

	cd .config/sublime-text-3/Package
	git clone https://github.com/lbrb/User.git
	
### 我安装的一些插件

- Scope​Always #可以查看当前文件的selector scope
- git #集成git的命令

### ubuntu下支持中文输入(sublime 使用deb默认安装)

	git clone https://github.com/lbrb/sublime_text_chinese_input.git
	cd sublime_text_chinese_input
	sudo cp libsublime-imfix.so /opt/sublime_text/
	sudo cp subl /usr/bin/
	sudo cp sublime_text.desktop /usr/share/applications/

### ubuntu下安装fcitx-sunpinyin[参考](http://freetstar.com/ubuntu-most-use-friendly-fcitx-sunpinyin/)
	
	sudo find ~ -name fcitx -ok rm -rf {} \;
	sudo apt-get install  fcitx
	sudo apt-get install fcitx-sunpinyin