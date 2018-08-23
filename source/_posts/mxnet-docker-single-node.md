---
title: mxnet学习(0) -- 使用Docker部署单节点mxnet
date: 2018-08-22 16:56:55
tags: ML, Performance, 源码编译
categories: mxnet
---

- 环境: 单server, Ubuntu 16.04

### 安装并启动Docker实例
1. 安装Docker, 按照官方教程安装: `http://docker.org/` 

2. 获取Ubuntu镜像，这里使用的是mxnet提供的image: `sudo docker pull mxnet/python`

3. 启动一个Docker instance, 可使用
```
#!/usr/bin/env bash

instance_name=$1  # Name of docker instance
cpus=$2  # Assignment of dedicated CPUs, e.g., 2,3,4,5

docker run -it --cpuset-cpus="$2" -v /home/ubuntu/projects/mxnet-learning/incubator-mxnet:/mxnet-src --name $1 -d mxnet/python bash
```
这里使用了`-v`来指定共享目录，是为了直接在物理Host主机上编译mxnet的源码，方便在同一server里的所有Docker instances使用

### 编译源码

1. 按照官方教程安装依赖库: `https://mxnet.apache.org/install/index.html?platform=Linux&language=Python&processor=CPU`
```
sudo apt update
sudo apt install -y build-essential git
sudo apt install -y libopenblas-dev liblapack-dev
sudo apt install -y libopencv-dev
```
2. 下载源码: `git clone --recursive https://github.com/apache/incubator-mxnet`
3. 开始编译
```
cd incubator-mxnet
make -j $(nproc) USE_OPENCV=1 USE_BLAS=openblas USE_DIST_KVSTORE=1
```
添加了USE_DIST_KVSTORE=1，因为后续会使用分布式的训练。编译需要等待一段时间

### Python绑定
1. 进入Docker instance。以name为`mxnet-main`实例为例
```
sudo docker exec -it mxnet-main /bin/bash
```

2. 安装python依赖
```
sudo apt-get install -y python-dev python-setuptools python-pip libgfortran3
```

3. 进入mxnet-src (docker run时设置的共享目录名称)，并安装
```
cd cd /usr/local/lib/python2.7/dist-packages # 建议进入python lib目录里把原有的mxnet给删除
rm -rf mxnet
cd /mxnet-src/python
pip install -e .
```

### 验证
1. 验证版本，从docker hub上pull的镜像自带mxnet环境，其版本号为1.2.1 (截止到2018-8-22 21:42), 通过安装github上clone的源码，其版本号为1.3.0
<center>{% asset_img version1.png Version before installation %}</center>

<center>{% asset_img version2.png Version after installation %}</center>

2. 安装完整性验证，以`image-classification`为例
```
cd /mxnet-src/example/image-classification
python train_mnist.py
```
<center>{% asset_img train.png%}</center>



