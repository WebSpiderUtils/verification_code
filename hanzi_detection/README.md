## 汉字位置检测模型文档

模型我已经训练好了，直接可以用。使用的darknet，模型使用方法见官网。**有问题咨询或者需要训练数据的童鞋可以加首页的微信跟我要，Git传不了这么多文件。**

> 现已更新QQ群，请看首页

提示：backup/ 文件夹下缺少权重文件，不是我不想传，是大于100M的文件Git传不了。如果需要，微信跟我要吧。

Git star 过500+，微信关注人数1000+会推出油管视频，谢谢支持！

### 使用方法

1. 编译

   打开终端

   进入项目文件夹：`cd hanzi_detection`

   编译：`make`

2. 测试

   编译成功后执行：`./darknet detector test data/jiyan.data model/jiyan_yolov3.cfg backup/jiyan_yolov3_final.weights `

   出现如下日志：

   ```shell
     100 conv    256  3 x 3 / 1    76 x  76 x 128   ->    76 x  76 x 256  3.407 BFLOPs
     101 conv    128  1 x 1 / 1    76 x  76 x 256   ->    76 x  76 x 128  0.379 BFLOPs
     102 conv    256  3 x 3 / 1    76 x  76 x 128   ->    76 x  76 x 256  3.407 BFLOPs
     103 conv    128  1 x 1 / 1    76 x  76 x 256   ->    76 x  76 x 128  0.379 BFLOPs
     104 conv    256  3 x 3 / 1    76 x  76 x 128   ->    76 x  76 x 256  3.407 BFLOPs
     105 conv     18  1 x 1 / 1    76 x  76 x 256   ->    76 x  76 x  18  0.053 BFLOPs
     106 yolo
   Loading weights from backup/jiyan_yolov3_final.weights...Done!
   
   ```

   模型加载完成，输入一个图片地址进行测试：

   ```shell
   Enter Image Path: ./809859eb7dbf70338f6c0dfa7c510c3a.jpg: Predicted in 18.303259 seconds.
   hanzi: 100%
   hanzi: 100%
   hanzi: 100%
   hanzi: 99%
   ```

   **识别结果会保存到当前文件夹下，名叫：predictions.jpg**

   可以看到识别这个结果用了18.3秒，以及识别到四个结果。

   因为目前是直接用CPU识别的，速度慢，用GPU速度会提升几百倍。关于GPU的使用待稍后整理发出，也可加微信交流。

   贴一个效果图：

   ![predicted img](./predictions.jpg)
   
   ### 关于环境搭建以及GPU训练方面文档稍后整理发出
   
   内容涉及：
   
   - Ubuntu18.04下的环境搭建
     - cuda + cudnn + opencv
     - Python3
     - NVIDIA 显卡驱动
   - GPU环境下模型的编译和使用
   
   GPU的使用在生产环境是必须的，速度提升了几百倍...
   
   
   
   **如果您看到这里，请给我一个star:star:谢谢，如果有遗漏，请留言指正，感激！**
   
   