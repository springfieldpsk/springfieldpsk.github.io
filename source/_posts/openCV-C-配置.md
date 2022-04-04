---
title: openCV C++ 配置
date: 2022-04-04 13:57:40
tags:
---

## 下载

在 [openCV](https://opencv.org/releases/) 官网下载相应包

## 配置环境变量

安装结束后将子文件夹build、build\x64\vc15\bin 添加至path
<!--more-->

## vs

在vs中新建项目

VC++目录添加 
    
包含目录：\build\include \build\include\opencv2

库目录：\build\x64\vc15\lib

链接器添加

输入\附加依赖项： opencv_world版本号d.lib opencv_world版本号.lib 
