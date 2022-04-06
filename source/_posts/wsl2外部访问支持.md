---
title: wsl2外部访问支持
date: 2022-04-06 21:36:14
tags: wsl2
---

[wslpp](https://link.zhihu.com/?target=https%3A//github.com/HobaiRiku/wsl2-auto-portproxy)

通过该软件可实现wsl的外部访问

注意：

1. 基于GO，因此需要安装Golang
2. 安装至wsl后，通过make build构建
3. 在 windows 用户文件夹 C:/user/{用户名} 下创建 .wslpp 文件夹 并创建config.json 配置转发接口