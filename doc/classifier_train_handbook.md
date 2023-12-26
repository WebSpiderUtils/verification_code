# 汉字分类识别模型训练手册

理论上可以不用`汉字检测+汉字识别`分两步，一步就可以完成，目前局限在数据集上，我手里的一步模型识别效果不是很理想。

此次我会挑几个字出来训练，训练数据我会放在`files/train`目录下，供大家练习和参考，更多讨论和需求欢迎加入我刚刚建立的QQ群：:rocket:`1040784971`:rocket:

---

[环境的安装和准备看这里](./Ubuntu18.04%20install%20darknet%20yolo-v3%7Ccuda%7Ccudnn%7Copencv%7Canaconda.md)

[汉字检测定位模型训练手册看这里](./detector_train_handbook.md)

---

## 一、数据准备

这次的数据集比较简单，我都放好了在`files/train/hanzi_classification`目录下，每个文件的文件名即标签。

文件名如下：

```shell
0092e7d888a1ab411eabd50e9cc5b2fa旅_u65c5.jpg  8b9c6569c7e55291f1dcff7026da1806区_u533a.jpg
036cac9f0db46babad19a9f129ed7587旅_u65c5.jpg  90e9fbca52b7faa571d430f9be27bfdd态_u6001.jpg
...
...
```

训练时读取的是`下划线_`后面的`Unicode`部分作为标签（中文支持不好），前面一段是为了方便区分。

训练数据包含了`生态旅游区`五个字，每个字`50`张图。

![](../media/0b7bbb3595309f7f9123704ef354a52d%E6%97%85_u65c5.jpg)

接下来配置几个文件即可开始训练：

- data文件：`data/hanzi_clas.data`

  ```txt
  classes = 5				# 分类（标签）数
  train = data/hanzi_clas_train.txt		# 训练数据路径
  labels = data/hanzi_clas.labels		# 标签文件
  backup = backup/		# 模型保存文件
  ```

  - `hanzi_clas_train.txt`文件的生成：

    这个文件内容是每个训练图片的绝对路径，一共250条记录对应250张图片（修改成自己电脑上的路径）

    ```txt
    /Users/hanzi_clas_train/719ce7a4febff78be03c33a6e11a3231生_u751f.jpg
    /Users/hanzi_clas_train/fedb1de7214d3c429bcb05ea28e92862生_u751f.jpg
    ...
    ...
    ```

  - `data/hanzi_clas.labels`文件的生成：

    记录需要分类的标签名称，我们有5个汉字需要分类，这里用`Unicode`编码来表示五个汉字

    ```txt
    u6001
    u65c5
    u6e38
    u751f
    u533a
    ```

- cfg文件：模型网络配置文件`cfg/hanzi_clas.cfg`，具体配置是什么意思Google吧，有感兴趣的童鞋再留言更新。

---

## 模型训练

1. 训练

   `./darknet classifier train data/hanzi_clas.data cfg/hanzi_clas.cfg`

   ```shell
   ...
   ...
   ...
   99, 25.344: 1.554884, 1.496595 avg, 0.000000 rate, 0.353925 seconds, 6336 images
   Loaded: 0.000017 seconds
   100, 25.600: 1.523866, 1.499322 avg, 0.000000 rate, 0.419332 seconds, 6400 images
   Saving weights to backup/hanzi_clas.weights
   ```

   训练完成后，会在`backup`文件夹下生成`weights`文件

2. 测试

   `./darknet classifier predict data/hanzi_clas.data cfg/hanzi_clas.cfg backup/hanzi_clas.weights `

   ```shell
   ...
   ...
   ...
   Loading weights from backup/hanzi_clas.weights...Done!
   top: Using default '1'
   Enter Image Path: 
   ```

   输入要测试的图片地址即可。

---

**如果您看到了这里，给个star:star:，谢谢，如有遗漏请留言，感激**

