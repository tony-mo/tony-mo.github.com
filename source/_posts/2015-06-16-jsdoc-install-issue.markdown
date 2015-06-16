---
layout: post
title: "JSDoc安装问题"
date: 2015-06-16 17:46:59 +0800
comments: true
categories: 
---
###JSDoc安装后出现-bash: sudu: command not found

1.根据JSDoc的github安装教程
[Github安装文档](https://github.com/jsdoc3/jsdoc)
 
 使用CLI指令`npm install jsdoc`来安装JSDoc后，在命令行输入指令`jsdoc -help`，CLI提示`-bash: sudu: command not found`。
 
 百度谷歌后，发现是安装的指令的问题，使用指令`npm install -g jsdoc@"<=3.3.0""`，安装后，在CLI输入`jsdoc -help`能显示jsdoc的帮组信息。
 
 留一下任务给自己，为什么使用指令`npm install -g jsdoc@"<=3.3.0""`能够成功使用jsdoc指令?
 
