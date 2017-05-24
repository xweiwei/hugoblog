+++
css = []
date = "2017-05-25T00:58:19+08:00"
description = ""
draft = false
highlight = true
metaimage = "/path/to/img.png"
metavideo = "/path/to/video.mp4"
nodisqus = false
notoc = false
scripts = []
sharebuttons = true
tags = ["机器学习"]
title = "体验tensorflow"

+++


## 安装

tensorflow提供多种环境的安装方式，具体查看[install](https://www.tensorflow.org/install/)

我使用的是Ubunt 17.04的环境，因为笔记本是带有NVIDIA的独立显卡的，一直用Ubuntu，独显都是关闭的，这回玩机器学习就可以派上用场了，虽然不是高端显卡，但是总比笔记本的CPU跑的快吧。所以这里接下来安装带显卡的版本。

在[tensorflow](https://github.com/tensorflow/tensorflow)的下载最新(2017/5/24)的whl

```
wget "https://ci.tensorflow.org/view/Nightly/job/nightly-python35-linux-cpu/lastSuccessfulBuild/artifact/pip_test/whl/tensorflow-1.2.0rc0-cp35-cp35m-linux_x86_64.whl"
sudo pip3 install tensorflow_gpu-1.2.0rc0-cp35-cp35m-linux_x86_64.whl
```

安装nvidia-cuda-toolkit

```
sudo apt-get install nvidia-cuda-toolkit
```

安装libcudnn
这个需要去nvidia官网注册账户下载
我选的是5.1的版本[libcudnn5_5.1.10-1+cuda8.0_amd64.deb](http://developer2.download.nvidia.com/compute/machine-learning/cudnn/secure/v5.1/prod_20161129/8.0/libcudnn5_5.1.10-1%2Bcuda8.0_amd64.deb?9jGUrECT4vEIctTCghrrlFVawG6MmMwk8vZcMnlZn52_R1pZlETOEcNIdCo6xXy-U5_MlDM1JBmEislL3dG4geAgS4MRrl4Eiw3dUuFaUYMFlohz-21ETOgadcCgAjWuvKyjgi0gS3rw4gp2cjRXsech39MAzttF8sJ8MQsv7dHRpnJlmPJrN66BYEdXCwVgr_x7FaGKoI_aYGQ-22bBRA7E)

```
wget "http://developer2.download.nvidia.com/compute/machine-learning/cudnn/secure/v5.1/prod_20161129/8.0/libcudnn5_5.1.10-1%2Bcuda8.0_amd64.deb?9jGUrECT4vEIctTCghrrlFVawG6MmMwk8vZcMnlZn52_R1pZlETOEcNIdCo6xXy-U5_MlDM1JBmEislL3dG4geAgS4MRrl4Eiw3dUuFaUYMFlohz-21ETOgadcCgAjWuvKyjgi0gS3rw4gp2cjRXsech39MAzttF8sJ8MQsv7dHRpnJlmPJrN66BYEdXCwVgr_x7FaGKoI_aYGQ-22bBRA7E"
sudo apt-get install libcudnn5_5.1.10-1+cuda8.0_amd64.deb
```

## Hello World

根据官网的列子，这里测试一下
```
weiwei@Lifar:~/opt/tensorflow$ cat hello_tensorflow.py 
#!/usr/bin/env python3

import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))

weiwei@Lifar:~/opt/tensorflow$ ./hello_tensorflow.py 
2017-05-25 01:10:43.850869: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.1 instructions, but these are available on your machine and could speed up CPU computations.
2017-05-25 01:10:43.850903: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
2017-05-25 01:10:43.850921: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
2017-05-25 01:10:43.888142: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:893] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
2017-05-25 01:10:43.888564: I tensorflow/core/common_runtime/gpu/gpu_device.cc:906] Found device 0 with properties: 
name: GeForce GT 645M
major: 3 minor: 0 memoryClockRate (GHz) 0.78
pciBusID 0000:01:00.0
Total memory: 1.95GiB
Free memory: 157.00MiB
2017-05-25 01:10:43.888601: I tensorflow/core/common_runtime/gpu/gpu_device.cc:927] DMA: 0 
2017-05-25 01:10:43.888622: I tensorflow/core/common_runtime/gpu/gpu_device.cc:937] 0:   Y 
2017-05-25 01:10:43.888652: I tensorflow/core/common_runtime/gpu/gpu_device.cc:996] Creating TensorFlow device (/gpu:0) -> (device: 0, name: GeForce GT 645M, pci bus id: 0000:01:00.0)
2017-05-25 01:10:43.891299: E tensorflow/stream_executor/cuda/cuda_driver.cc:924] failed to allocate 157.00M (164626432 bytes) from device: CUDA_ERROR_OUT_OF_MEMORY
b'Hello, TensorFlow!'
```

运行成功。

不过这里有几个提示，说使用的TensorFlow library 没有使用一些指令集， 但是机器支持这些指令，如果使用的话能提高速度。看来用Google提供的安装包出于兼容的考虑，没有使用这些。下次有机会试试重新再本地编译一个library。
