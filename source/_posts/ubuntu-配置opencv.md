---
title: ubuntu 配置opencv
date: 2022-04-05 20:27:03
tags:
---

## 过程

<!-- more -->
1. 安装依赖
   
    ```bash
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install libgtk2.0-dev 
    sudo apt-get install pkg-config
    ```

2. 安装 C/C++ 依赖

    ```bash
    sudo apt install -y g++
    sudo apt-get install cmake
    sudo apt-get install make
    ```

3. 克隆git opencv库

    ```bash
    git clone https://github.com/opencv/opencv.git
    git clone https://github.com/opencv/opencv_contrib.git
    ```

4. 创建文件夹   

    ```bash
    mkdir -p build && cd build
    ```

5. cmake 配置

    ```bash
    cmake -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules ../opencv
    ```

6. 构建

    ```bash
    make -j4
    ```

7. 安装

    ```bash
    sudo make install
    ```

8. 创建 CMAKE 文件

    ```CMAKE
    cmake_minimum_required(VERSION 2.8)
    project( DisplayImage )
    find_package( OpenCV REQUIRED )
    include_directories( ${OpenCV_INCLUDE_DIRS} )
    add_executable( DisplayImage DisplayImage.cpp )
    target_link_libraries( DisplayImage ${OpenCV_LIBS} )
    ```

    ```bash
    cd <DisplayImage_directory>
    cmake .
    make
    ```
    
9. 运行二进制文件

    ```bash
    ./DisplayImage
    ```

ref: https://stackoverflow.com/questions/65738296/how-to-run-a-simple-opencv-code-in-c-on-linux