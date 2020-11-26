---
title: opencv3-ubuntu16.04-install
date: 2018-07-17 19:45:44
tags:
- 环境搭建
- opencv3
categories: 
- 调试记录
---

{% asset_img opencv.jpg This is an example image %}

```bash
    sudo apt-get install cmake
    sudo apt-get install python3-dev python3-numpy
    sudo apt-get install gcc g++

    sudo apt-get install libgtk2.0-dev
    sudo apt-get install libv4l-dev
    sudo apt-get install libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
    sudo apt-get install sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev liblapacke-dev
    sudo apt-get install libxvidcore-dev libx264-dev 
    sudo apt-get install libatlas-base-dev gfortran
    sudo apt-get install ffmpeg 

    sudo apt-get install git
    git clone https://github.com/opencv/opencv.git

    cmake dir/of/opencv/source
    sudo make -j4 
    sudo make install
```