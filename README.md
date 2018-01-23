### Segementation

本次项目主要实现语义分割，内容有两个部分：准备数据制作tfrecord和修改fcn_8s

##### 一、准备数据

1.下载数据集VOCtrainval_11-May-2012.tar

​    VOC网址：http://host.robots.ox.ac.uk/pascal/VOC/voc2012/

2.修改代码convert_fcn_dataset.py

​    主要修改以下两个函数

​    *dict_to_tf_example()* 中补上了feature_dict的内容，可以根据dataset中的read_and_decode()函数知道feature_dict中每项填的内容和数据类型等信息。

​    *create_tf_record()* 是调用dict_to_tf_example()方法把数据转化成example格式，再用TFRecordWriter生成tfrecord格式。



3.生成数据上传到tinymind

​    由于上传tfrecord到tinymind太慢且网络不稳定，所以生成每个tfrecord文件500张图片，失败了不用整个训练集或验证集重传。修改了read_images_names()和tfrecord文件读取相关的代码，添加一个参数FLAGS.data_dir。



##### 二、修改fcn_8s

修改代码train.py

主要过程：

1.获取pool3、pool4的feature_map

2.对vgg_16的logit进行2X上采样和反卷积

3.将结果与pool4合并，再进行2X上采样和反卷积

4.将结果与pool3合并，再进行8X上采样和反卷积

5.对结果与ground_truth计算loss，进行优化



##### 三、结果

​    经过2000个step的训练后，发现对于目标物体和背景颜色区别大的情况分割的效果比较好，目标物体与背景颜色相近的情况分割效果不理想。继续在之前的结果上训练了2000个step，效果没有明细改善。

tinymind运行结果：https://www.tinymind.com/executions/b6lc9zc9

·step2000:

原图

![val_2000_img](https://github.com/Baageer/quiz_segmentation/blob/master/val_2000_img.jpg)

标签

![val_2000_annotation](https://github.com/Baageer/quiz_segmentation/blob/master/val_2000_annotation.jpg)

预测

![val_2000_prediction](https://github.com/Baageer/quiz_segmentation/blob/master/val_2000_prediction.jpg)

CRF之后的预测

![val_2000_prediction_crfed](https://github.com/Baageer/quiz_segmentation/blob/master/val_2000_prediction_crfed.jpg)



·step3600:

原图

![val_3600_img](https://github.com/Baageer/quiz_segmentation/blob/master/val_3600_img.jpg)

标签

![val_3600_annotation](https://github.com/Baageer/quiz_segmentation/blob/master/val_3600_annotation.jpg)

预测

![val_3600_prediction](https://github.com/Baageer/quiz_segmentation/blob/master/val_3600_prediction.jpg)

CRF之后的预测

![val_3600_prediction_crfed](https://github.com/Baageer/quiz_segmentation/blob/master/val_3600_prediction_crfed.jpg)



##### 四、问题

1、dict_to_tf_example()中'image/encoded'数据类型写错。

​    发现读取数据时，需要的是string类型，把float_list_feature()改为bytes_feature()。

