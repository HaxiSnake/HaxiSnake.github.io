---
title: 树莓派3B——turtlebot3——ROS开发环境搭建问题记录
date: 2018-07-18 13:41:27
tags:
- raspberry3B
- ROS
- turtlebot3
- 环境搭建
categories: 
- 调试记录
---

{% asset_img turtlebot3.jpg image %}

由于一个比赛项目需要在ROS上做开发，但是按照网上教程搭建开发环境出现了一些问题，因此在此做简单整理，本笔记的开发环境如下：

* 硬件环境：turtlebot3 burger(用的其自带的树莓派为raspberry3B)
* 树莓派系统版本：ubuntu-mate-16.04.2-desktop-armhf
* PC系统版本： ubuntu-16.04.3-desktop-amd64

教程参考： https://www.ncnynl.com/archives/201702/1392.html

# 问题记录
## 问题一
问题描述：wget 安装脚本时验证出错导致脚本无法下载

解决方法：在wget命令后加上--no-check-certificate选项即可

## 问题二
问题描述：git clone 出现CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none错误

解决方式：运行 export GIT_SSL_NO_VERIFY=1

## 问题三
问题描述：安装turtlebot依赖包之后找不到catkin_make命令

解决方式：
重新执行 source /opt/ros/kinetic/setup.sh

## 问题四
问题描述：cakin_make时找不到interactive_maker模块

解决方式：修改安装脚本，使其安装desktop_full版本

## 问题五
问题描述：checksum error when launching turtlebot3_bringup

解决方式：更新OpenCR固件至最新版与ROS版本相匹配