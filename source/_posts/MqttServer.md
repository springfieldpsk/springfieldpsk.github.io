---
title: MqttServer
date: 2022-04-10 21:46:35
tags:
---
## 思路

使用mosquitto搭建mqtt服务器

<!-- more -->

## 流程

1. 使用apt安装mosquitto 与 测试用客户端

    ```bash
    sudo apt-get update 
    sudo apt-get install mosquitto 
    sudo apt-get install mosquitto-clients
    ```

2. 添加配置文件

    ```bash
    sudo vi /etc/mosquitto/conf.d/myconfig.conf

    #添加监听端口（很重要，否则只能本机访问）
    listener 1883
    #-------------------------------------------
    # 关闭匿名访问，客户端必须使用用户名
    allow_anonymous false

    #指定 用户名-密码 文件
    password_file /etc/mosquitto/pwfile.txt
    #--------------------------------------------
    ```

4. 添加账户和密码

    ···bash
    sudo mosquitto_passwd -c /etc/mosquitto/pwfile.txt 用户名
    ```

5. 启动并监控mosquitto

    ```bash
    sudo service mosquitto start
    sudo service mosquitto status
    ···