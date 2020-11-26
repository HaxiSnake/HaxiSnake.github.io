---
title: OpenPose环境搭建笔记
date: 2019-04-09 20:29:54
tags:
---

# OpenPose环境搭建笔记

## 编译caffe

* recipe for target '.build_release/src/caffe/proto/caffe.pb.o' failed

参考教程：
https://blog.csdn.net/w5688414/article/details/79478695

*  ./include/caffe/util/hdf5.hpp:6:18: fatal error: hdf5.h: No such file or directory

解决办法：在Makefile.config找到以下行并添加蓝色部分

INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial 

LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-Linux-gnu/hdf5/serial

3.1 如果仍然提示找不到lhdf5和lhdf5_hl(这是因为ubuntu16.04的文件包含位置发生了变化，尤其是需要用到的hdf5的位置，所以需要更改这一路径)选自
caffe ---找不到lhdf5_hl和lhdf5的错误

解决办法：然后根据情况执行下面两句：

cd /usr/lib/x86_64-linux-gnu

sudo ln -s libhdf5_serial.so.10.1.0 libhdf5.so

sudo ln -s libhdf5_serial_hl.so.10.0.2 libhdf5_hl.so

* https://github.com/BVLC/caffe/issues/6359

/usr/include/c++/5/typeinfo:39:37: error: expected ‘}’ before end of line
/usr/include/c++/5/typeinfo:39:37: error: expected unqualified-id before end of line
/usr/include/c++/5/typeinfo:39:37: error: expected ‘}’ before end of line
/usr/include/c++/5/typeinfo:39:37: error: expected ‘}’ before end of line
/usr/include/c++/5/typeinfo:39:37: error: expected ‘}’ before end of line
/usr/include/c++/5/typeinfo:39:37: error: expected declaration before end of line
Makefile:588: recipe for target '.build_release/src/caffe/proto/caffe.pb.o' failed
make: *** [.build_release/src/caffe/proto/caffe.pb.o] Error 1

CXXFLAGS += -pthread -fPIC $(COMMON_FLAGS) $(WARNINGS) -std=c++11
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS) -std=c++11
LINKFLAGS += -pthread -fPIC $(COMMON_FLAGS) $(WARNINGS) -std=c++11

* Unsupported gpu architecture 'compute_20'

注释Makefile.config 中 :

    # CUDA architecture setting: going with all of them.
    # For CUDA < 6.0, comment the *_50 through *_61 lines for compatibility.
    # For CUDA < 8.0, comment the *_60 and *_61 lines for compatibility.
    # For CUDA >= 9.0, comment the *_20 and *_21 lines for compatibility.
    CUDA_ARCH :=    -gencode arch=compute_30,code=sm_30 \
                    -gencode arch=compute_35,code=sm_35 \
                    -gencode arch=compute_50,code=sm_50 \
                    -gencode arch=compute_52,code=sm_52 \
                    -gencode arch=compute_60,code=sm_60 \
                    -gencode arch=compute_61,code=sm_61 \
                    -gencode arch=compute_61,code=compute_61

* leveldb
https://blog.csdn.net/oeljeklaus/article/details/78802922

https://blog.csdn.net/fogxcg/article/details/75332146

* build opencv3.4

cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local -D PYTHON3_EXECUTABLE=/usr/bin/python3 -D PYTHON_INCLUDE_DIR=/us
r/include/python3 -D PYTHON_INCLUDE_DIR2=/usr/include/x86_64-linux-gnu/python3.5m -D PYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.5m.so -D PYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include/ 

* levelDB出错

https://blog.csdn.net/jyl1999xxxx/article/details/80583465

* glog 和 gflags 的安装
https://www.cnblogs.com/burningTheStar/p/6986048.html
-fPIC
https://github.com/BVLC/caffe/issues/2171

* make runtest .build_release/tools/caffe: error while loading shared libraries: libopencv_imgcodecs.so.3.4: cannot open shared object file: No such file or directory
Makefile:542: recipe for target 'runtest' failed

* make runtest https://blog.csdn.net/wonengguwozai/article/details/52724409

# 编译openpose 

* cuda结构不共存 Unsupported gpu architecture 'compute_xx'

在cmake-gui 中选择 CUDA_ARCH 为mannul 然后再删掉其中不符合的结构

* g++: error trying to exec 'cc1plus': execvp: 没有那个文件或目录

gcc g++ 版本不兼容 

https://blog.csdn.net/yile0000/article/details/80105625

nvidia驱动

https://blog.csdn.net/wf19930209/article/details/81877822

sudo service lightdm stop

* gflags.cc is being linked both statically and dynamically 

https://github.com/google/glog/issues/53

修改CmakeCache.txt中的
GFLAGS_LIBRARY:FILEPATH=/usr/local/lib/libgflags.a 
改为：
GFLAGS_LIBRARY:FILEPATH=/usr/local/lib/libgflags.so
然后重新编译

* [libprotobuf ERROR google/protobuf/message_lite.cc:123] Can't parse message of type "caffe.NetParameter" because it is missing required fields: layer[0].clip_param.min, layer[0].clip_param.max

要用openpose自带的caffe库

选择openpose编译caffe时选项可以在3rdparty/caffe中设置编译参数

