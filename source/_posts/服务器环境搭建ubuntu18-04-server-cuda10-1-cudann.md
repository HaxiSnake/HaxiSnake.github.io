---
title: 服务器环境搭建
date: 2019-05-07 11:27:14
tags:
---

ubuntu18.04-server
cuda10.1
cudann7.5.0

# 安装显卡驱动
1. linux查看显卡型号

    lshw -numeric -C display

2. 下载nvidia官方驱动

    [官网链接](https://www.geforce.cn/drivers)

3. bios禁用禁用secure boot

4. 禁用nouveau

    打开文件

        sudo vim /etc/modprobe.d/blacklist.conf

    在最后一行添加:

        blacklist nouveau

    执行如下命令生效

        sudo update-initramfs -u

5. 重启

    使用命令查看nouveau有没有运行

        lsmod | grep nouveau  # 没输出代表禁用生效

6. 停止可视化界面

        sudo telinit 3

7. 安装驱动

    先添加可执行权限

        sudo chmod a+x filename.run

    执行安装：

        sudo ./filename.run -no-opengl-files

    -noopengl-files这个参数一定要加，否则会循环登录

# 安装CUDA10.1

1. 下载cuda10.1安装文件 

    [官网链接](https://developer.nvidia.com/cuda-downloads)


2. 运行安装

    执行安装文件进行安装，同nvidia驱动的执行方式。

    注意此时不需要选择安装驱动。驱动版本要注意跟cuda版本相对应。

# 安装cudann

请见官网[安装教程](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html)
