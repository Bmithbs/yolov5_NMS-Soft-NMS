# Soft NMS

NMS 算法首先按照得分从高到低对建议框进行排序，然后分数最高的检测框M被选中，其他框与被选中建议框有明显重叠的框被抑制。该过程被不断递归的应用于其余检测框。

## Soft NMS的优势

1. 它仅需要对传统的NMS算法进行简单的改动且不增额外的参数。该Soft-NMS算法在标准数据集PASCAL VOC2007（较R-FCN和Faster-RCNN提升1.7%）和MS-COCO（较R-FCN提升1.3%，较Faster-RCNN提升1.1%）上均有提升。
2. Soft-NMS具有与传统NMS相同的算法复杂度，使用高效。
3. 
Soft-NMS不需要额外的训练，并易于实现，它可以轻松的被集成到任何物体检测流程中。

## Soft NMS原理

见下图伪代码，整个改进只需要使用绿色虚线表示的Soft-NMS替换红色虚线表示的NMS。

B集合是检测到的所有建议框，S集合是各个建议框得分（分数是指建议框包含物体的可能性大小），Nt是指手动设置的阈值。M为当前得分最高框，bi 为待处理框。

![](https://pic2.zhimg.com/v2-17d64df1108abd9794a9305828160825_r.jpg)

综上，soft-nms的核心就是降低置信度。比如一张人脸上有3个重叠的bounding box, 置信度分别为0.9, 0.7, 0.85 。选择得分最高的建议框，经过第一次处理过后，得分变成了0.9, 065, 0.55（此时将得分最高的保存在D中）。这时候再选择第二个bounding box作为得分最高的，处理后置信度分别为0.65, 0.45（这时候3个框也都还在），最后选择第三个，处理后得分不改变。最终经过soft-nms抑制后的三个框的置信度分别为0.9, 0.65, 0.45。最后设置阈值，将得分si小于阈值的去掉。


## $f(iou(M,b_i))$ 权重函数的形式

原来的NMS可以描述如下：将IOU大于阈值的窗口的得分全部置为0

$$
s_{i}= \begin{cases}s_{i}, & \operatorname{iou}\left(\mathcal{M}, b_{i}\right)<N_{t} \\ 0, & \operatorname{iou}\left(\mathcal{M}, b_{i}\right) \geq N_{t}\end{cases}
$$

Soft-NMS的改进有两种形式，一种是线性加权的：

$$
s_{i}= \begin{cases}s_{i}, & \operatorname{iou}\left(\mathcal{M}, b_{i}\right)<N_{t} \\ s_i(1-\operatorname{iou}\left(\mathcal{M}, b_{i}\right)), & \operatorname{iou}\left(\mathcal{M}, b_{i}\right) \geq N_{t}\end{cases}
$$

一种是高斯加权的：
$$
s_{i}=s_{i} e^{-\frac{\mathrm{iou}\left(\mathcal{\mathcal { M }}, b_{i}\right)^{2}}{\sigma}}, \forall b_{i} \notin \mathcal{D}
$$


