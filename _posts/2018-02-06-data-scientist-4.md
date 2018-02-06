---
title: 辨Data Scientist之真假（四）
description: 查准率，查全率，ROC曲线。
categories:
 - Questions for data scientist
tags:
 - 模型性能指标
---



##### 问题四：什么是查准率和查全率？它们与ROC曲线有什么关系？

> - 真阳性（True Positive， TP）：样本是阳性的，分类器将样本也分类为阳性；
> - 假阴性（False Negative， FN）：样本是阳性的，分类器将样本分类为阴性；
> - 真阴性（True Negative， TN）：样本是阴性的，分类器将样本也分类为阴性；
> - 假阳性（False Positive， FP）：样本是阴性的，分类器将样本分类为阳性。
> - **查准率**（Precision）：TP / (TP + FP)
> - **查全率**/ 召回率（Recall）： TP / (TP + FN)
> - 真阳性率（True Positive Rate）： TPR = TP / (TP + FN)
> - 假阳性率（False Positive Rate）： FPR = FP / (FP + TN)

- **ROC曲线**（receiver operating characteristic）是一种用于衡量二值分类器的功能图像，表现了敏感性（查全率）和特异性（不准确）之间的关系，x轴为**FRP**，y轴为**TPR**，有时候也被称为“灵敏度 vs. 1-特异度”曲线图。
  <img src="http://img.blog.csdn.net/20180204002523255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTHV6X0RhdGFfU2NpZW50aXN0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="60%" height="60%" />

> - 最好的预测方式是在（0,1），这个点被称为“完美分类器”；
> - 点A比点B更加保守，因为A的假阳性率比B低；
> - 点C代表一个随机分类器；
> - 点E表示该分类器比随机分类器要差，但是如果将其分类决策反转，那么它就好于随机分类器。

---

- **PR曲线**：当处理的数据集表现出高度不均衡时，可以用**PR曲线**来评估分类器的可信度。在PR曲线中，x轴是**Recall**，y轴是**Precision**。
  <img src="http://img.blog.csdn.net/20180204002713502?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTHV6X0RhdGFfU2NpZW50aXN0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="70%" height="70%" />
> ROC曲线越凸向左上方，分类器的效果越好，而PR曲线是越凸向右上，效果越好。


- **AUC**（Area Under Curve）：ROC曲线下的面积。
> 在一般情况下，ROC曲线都处在y=x这条直线的上方，所以其取值范围在0.5和1之间。AUC的值越大，说明分类器的效果越好。
