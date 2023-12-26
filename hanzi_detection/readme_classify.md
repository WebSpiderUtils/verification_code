## 汉字分类识别模型文档

和检测模型一样使用yolo算法，同样在检测模型项目下进行。

提示：backup/ 文件夹下缺少权重文件，不是我不想传，是大于100M的文件Git传不了。如果需要，微信跟我要吧。

### 使用方法

预测：`./darknet classify predict data/jiyan_classify.data model/jiyan_classify.cfg backup/jiyan_classify_final.weights`



![classify_test](./classify_test.jpg)



```shell
[('池', 0.8325800895690918), ('施', 0.027770591899752617)]
```

