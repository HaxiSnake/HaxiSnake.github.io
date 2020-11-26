---
title: 树莓派3B+编译安装opencv3
date: 2018-07-17 19:17:56
tags:
- raspberry3B+
- opencv3
- 环境搭建
categories: 
- 调试记录
---
{% asset_img opencv.jpg image %}
## 一、更新源

```bash
mv sources.list /etc/apt/sources.list 
mv raspi.list /etc/apt/sources.list.d/raspi.list 
```
更新源的配置，注意文件存放的位置
文件sources.list和raspi.list具体内容如下

sources.list文件: 

        deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi
        deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi
        deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi
        deb http://mirrors.neusoft.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi



raspi.list文件:
```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main ui
deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main ui
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ stretch main ui
deb http://mirrors.neusoft.edu.cn/raspbian/raspbian/ stretch main ui
```

在终端执行更新命令:
```bash
sudo apt-get update
sudo apt-get upgrade
```
## 二、安装依赖包
```bash
    sudo apt-get install build-essential cmake git pkg-config 
    sudo apt-get install libjpeg8-dev 
    sudo apt-get install libtiff5-dev 
    sudo apt-get install  libjasper-dev 
    sudo apt-get install  libpng12-dev

    sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
    sudo apt-get install libgtk2.0-dev
    sudo apt-get install libatlas-base-dev gfortran
```
*注意：降级安装*
*有些安装包依赖的版本低需要降级安装，如下，对depends后面的进行降级安装*
```bash
    sudo aptitude install xxxx
```
##  三、下载源码
```bash
    git clone https://github.com/opencv/opencv.git
```
## 四、编译
```bash
    cmake dir/of/opencv/source
    sudo make -j4 
    sudo make install
    sudo ldconfig
```
