# Ubuntu18.04安装测试darknet yolo-v3|cuda|cudnn|opencv|anaconda 

> 机器配置：
>
> cpu：i3-9400
>
> gpu：gtx1650-4g
>
> 系统版本：Ubuntu18.04

前段时间接触到darknet目标检测的项目，由于需要做效率优化，之前在mac上使用yolo3，项目开发很顺利，最终瓶颈是cpu的运算速度，故需要在有显卡的机器进行测试。

[TOC]

---

## 一. Darknet项目

项目官网：https://pjreddie.com/darknet/yolo/

使用方式也很简单，官网都有介绍，这里就举个例子：

- 下载编译项目：

```shell
git clone https://github.com/pjreddie/darknet
cd darknet
make
```

- 下载权重文件：

```shell
wget https://pjreddie.com/media/files/yolov3.weights
```

- 测试

```shell
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg
```

然后在项目根目录下会生成`predictions.png`，打开即可看到效果。

到这里就可以进行二次开发了，但是都是基于CPU的，无论是前期的训练还是后期的测试，速度都是很慢的，GPU的速度在我测试下有200倍的提升（我的GPU一般，还有很大提升空间）。

---

## 二. Ubuntu安装NVIDIA显卡驱动|CUDA|CUDNN

要使用GPU进行项目的运算，除了改项目的配置以外，最重要，也是比较麻烦的部分就是环境的安装，坑比较多，需要细心一点。

### 2.1 显卡驱动

输入命令看一下系统推荐的驱动版本号，再进行相应版本的驱动安装即可。

命令：`ubuntu-drivers devices`

```shell
(base) iron@iron-H310M-S2-2-0:~/桌面/share/darknet$ ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001F82sv00001B4Csd00001320bc03sc00i00
vendor   : NVIDIA Corporation
driver   : nvidia-driver-435 - distro non-free
driver   : nvidia-driver-440 - third-party free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```

可以看到我的系统`recommended`版本是`nvidia-driver-440`，直接使用命令安装：

`sudo apt-get install nvidia-driver-440`

测试安装是否成功：

使用`nvidia-smi`命令，输出显卡信息即成功。

> 提示：显卡驱动也可以去NVIDIA官网查询对应的版本进行安装，下载Linux版本即可。我个人习惯使用上面的方式。

### 2.2 CUDA

CUDA官网下载页面：https://developer.nvidia.com/cuda-toolkit-archive

目前出到了`10.2`版本，不过我下载的`10.1`，为了求稳，也为了遇到问题能在网上找到更多的解决办法。

下载选择：Linux-->x86_64-->ubuntu-->18.04-->runfile

这里将我下载的`10.1`版本传到百度了：`链接: https://pan.baidu.com/s/1PyQJcR3G-kKIHqOFtFiBHg 提取码: ssp9`

- 安装依赖

`sudo apt-get install freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libgl1-mesa-glx libglu1-mesa libglu1-mesa-dev`

- 安装CUDA

进入CUDA的下载目录，执行安装：

`sudo ./cuda_10.1.105_418.39_linux.run `

安装过程没什么可说的，唯一要注意的是：在出现`Install NVIDIA Accelerated Graphics Driver for Linux-x86_64?`时，输入`no`，因为上一步我们装过显卡驱动了，这里不用安装，其他步骤都是`yes`或者`回车`。

- 配置环境变量

配置前确定指定的目录已经存在

```bash
sudo vi ~/.bashrc
#文件末尾添加
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-10.1/lib64
```
```shell
#刷新配置文件
sudo source ~/.bashrc
```

- 测试

检查是否安装成功

`cd  /usr/local/cuda-10.1/samples/1_Utilities/deviceQuery`
`sudo make `
`./deviceQuery`

输出CUDA	VERSION等信息即成功

### 2.3 CUDNN

cudnn安装要注意的是版本要和cuda一致，下载页面：https://developer.nvidia.com/rdp/cudnn-download

下载需要注册开发者账号，按照步骤注册即可。

百度云`10.1`版本：`链接: https://pan.baidu.com/s/1oAsw6LUMpjW9NfNL_JVIJA 提取码: zvui`

- 将cudann的压缩包解压
- 解压后得到cuda文件夹，文件夹和cuda安装目录中的文件夹是对应的，将文件复制到cuda对应的文件夹中即可

> 上一步的cuda安装完成以后路径为：/usr/local/cuda10.1

示例：

```shell
tar -zxvf cudnn-10.1-linux-x64-v7.6.5.32.tgz

sudo cp cuda/include/cudnn.h    /usr/local/cuda10.1/include
sudo cp cuda/lib64/libcudnn*    /usr/local/cuda10.1/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h   /usr/local/cuda10.1/lib64/libcudnn*
```

最后更新软链接，这步如果不执行，后面会报错`xxxxxx.so.x 不是符号链接`

> 这个so文件的版本要提前看好，我文件夹里的so文件是libcudnn.so.7.6.5

```shell
cd /usr/local/cuda-10.1/lib64/  

sudo chmod +r libcudnn.so.7.6.5
sudo ln -sf libcudnn.so.7.6.5 libcudnn.so.7  
sudo ln -sf libcudnn.so.7 libcudnn.so  
sudo ldconfig 
```

或者在遇到某某so文件不是符号链接的错误时，执行：`sudo ldconfig -v`，找到错误的so文件，创建一个软链。

---

## 三. 使用GPU测试运行项目

前面装好了所需的环境，接下来测试yolo项目是否可以在GPU下运行。

- 修改darknet项目配置文件

在darknet项目根目录下有`Makefile`文件，修改其中配置：

```shell
GPU=1
CUDNN=1
```

- 重新编译项目：

`make clean`

`make`

- 测试：

`./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg`

这里是我的测试速度对比：质的飞跃

- CPU

```shell
Loading weights from yolov3.weights...Done!
data/dog.jpg: Predicted in 19.692001 seconds.
dog: 6379%
truck: 1609%
truck: 226%
bicycle: 3912%
```

- GPU

```shell
Loading weights from yolov3.weights...Done!
data/dog.jpg: Predicted in 0.131651 seconds.
dog: 100%
truck: 92%
bicycle: 99%
```

---

## 四. 配置OPENCV

出于对darknet视频实时检测的好奇，决定自己做一个目标检测的样例视频，自己拍摄剪辑然后做目标检测，所以又需要安装OPENCV来支持视频处理。

### 4.1 文件下载

文件需要下载两个，一个是`opencv`一个是`opencv_contrib`，版本要对应，我这里下载的是4.3版本

- 官方

opencv：https://opencv.org/releases/

opencv_contrib：https://github.com/opencv/opencv_contrib/releases

- 百度盘

opencv：`链接: https://pan.baidu.com/s/163SbilBZdsS2MxqTdMaeEg 提取码: asja`

opencv_contrib：`链接: https://pan.baidu.com/s/1LhMkqBr5U5b8b7qXTZDAtA 提取码: niqq`

### 4.2 安装

- 安装依赖

```shell
    sudo apt-get install build-essential
    sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
    sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

- 安装cmake-gui

```shell
    sudo apt-get install cmake-qt-gui
```

- 将`opencv`和`opencv_contrib`解压后放到统一目录下
- 在`opencv`目录下创建`build`目录，用来存放编译文件
- 进入`build`目录，执行：`cmake-gui`
- 在界面上方配置源码和编译目录
  - where is the source code：# 指定opencv的目录
  - where to build the binaries：# 指定编译目录（即上一步创建的build目录）
- 点击`configure`
  - Unix Makefiles - > Use default native compilers- > Finish

- 编译参数配置
  - CMAKE_BUILD_TYPE：RELEASE
  - CMAKE_INSTALL_PREFIX：# 配置安装目录
  - OPENCV_EXTRA_MODULES_PATH：# 配置opencv_contrib的modules目录

- 点击`Generate`
- 在**终端**进入`build`目录，执行`make`，开始编译，没有意外会编译成功进行下一步，而现实是不可能没有意外...
- 编译成功后，再执行`make install `进行安装

#### 4.2.1 编译时的意外

- fatal error：boostdesc_bgm.i 没有那个文件或目录

文件下载：`链接: https://pan.baidu.com/s/1cjEtx0dIrmeUJdFDmb-KHw 提取码: ir4h `

全部文件放入`opencv_contrib/modules/xfeatures2d/src/`目录下

- fatal error : feature2d/test/test_detectors_regression.impl.hpp

这是一类头文件的问题，提示找不到文件，可能错误地方不止一个，但是解决方法是一样的

> 定位到出错的`cpp`文件
>
> 编辑这个文件，将`#include "feature2d/test/test_detectors_regression.impl.hpp"`改为`#include "test_detectors_regression.impl.hpp"`
>
> 然后将`test_detectors_regression.impl.hpp`文件放到当前`cpp`文件目录中就可以了
>
> 那这个文件在哪呢？
>
> 所有缺失的hpp文件在opencv4.3/modules/features2d/test文件夹中都能找到

### 4.3 配置环境变量
-   编辑文件sudo vim /etc/ld.so.conf.d/opencv.conf ，写入opencv安装目录下的lib目录路径
-   编辑文件sudo vim /etc/bash.bashrc ，末尾添加两行内容：
    - `PKG_CONFIG_PATH=$PKG_CONFIG_PATH:opencv安装目录/lib/pkgconfig `
    - `export PKG_CONFIG_PATH`
- 最后执行：source /etc/profile

### 4.4 测试

测试是否安装成功：pkg-config --cflags --libs opencv

```shell
(base) iron@iron-H310M-S2-2-0:/usr/local/cuda/lib64$ pkg-config --cflags --libs opencv
-I/home/iron/opencv-4.3.0/include/opencv4 -L/home/iron/opencv-4.3.0/lib -lopencv_shape -lopencv_stitching -lopencv_objdetect -lopencv_superres -lopencv_videostab -lopencv_calib3d -lopencv_features2d -lopencv_highgui -lopencv_videoio -lopencv_imgcodecs -lopencv_video -lopencv_photo -lopencv_ml -lopencv_imgproc -lopencv_flann -lopencv_core
```

---

## 五. 再次编译darknet项目

我们这次加入了opencv的支持，和之前一样，先改配置文件，再重新编译项目

- 改darknet项目下：Makefile文件

```shell
GPU=1
CUDNN=1
OPENCV=1
```

- 重新编译项目：

`make clean`

`make`

不出意外的话直接编译成功

### 5.1 darknet + opencv编译的意外

- image_opencv.cpp:12:1: error: ‘IplImage’

这看似是代码兼容问题，在我们的darknet项目下可以找到这个出错的文件：darknet/src/image_opencv.cpp

最简单的解决办法是把文件替换成我的这个，我已经改好了：

`链接: https://pan.baidu.com/s/1bns2HLAFd--pvBHJpTZaLQ 提取码: 929p`



---

## 六、参考文献

> [opencv4.2.0+darknet安装 ./src/image_opencv.cpp:12:1: error: ‘IplImage’ 报错](https://blog.csdn.net/a135013563/article/details/104859395)
>
> [YOLO: Real-Time Object Detection](https://pjreddie.com/darknet/yolo/)

---

**如果您看到了这里，请给我一个star:star:谢谢，如果有遗漏，请留言指正，感激！** 