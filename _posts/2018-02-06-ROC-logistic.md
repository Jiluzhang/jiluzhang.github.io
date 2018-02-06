---
title: 用ROC曲线评估logistic回归模型性能
categories:
 - machine learning
tags:
 - ROC曲线 
 - logistic回归
---



ROC曲线被广泛用于二分类输出模型的性能评估。这里我们将给出一个简单的例子，使用数据集“diamonds”创建logistic回归模型，然后通过绘制ROC曲线来确定carat、cut和clarity这三个因素中哪个最能预测钻石的昂贵与否。

---
#### 探索和准备数据

```R
> library(ggplot2)
> str(diamonds)    # diamonds是ggplot2里的数据集

Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	53940 obs. of  10 variables:
 $ carat  : num  0.23 0.21 0.23 0.29 0.31 0.24 0.24 0.26 0.22 0.23 ...
 $ cut    : Ord.factor w/ 5 levels "Fair"<"Good"<..: 5 4 2 4 2 3 3 3 1 3 ...
 $ color  : Ord.factor w/ 7 levels "D"<"E"<"F"<"G"<..: 2 2 2 6 7 7 6 5 2 5 ...
 $ clarity: Ord.factor w/ 8 levels "I1"<"SI2"<"SI1"<..: 2 3 5 4 2 6 7 3 4 5 ...
 $ depth  : num  61.5 59.8 56.9 62.4 63.3 62.8 62.3 61.9 65.1 59.4 ...
 $ table  : num  55 61 65 58 58 57 57 55 61 61 ...
 $ price  : int  326 326 327 334 335 336 336 337 337 338 ...
 $ x      : num  3.95 3.89 4.05 4.2 4.34 3.94 3.95 4.07 3.87 4 ...
 $ y      : num  3.98 3.84 4.07 4.23 4.35 3.96 3.98 4.11 3.78 4.05 ...
 $ z      : num  2.43 2.31 2.31 2.63 2.75 2.48 2.47 2.53 2.49 2.39 ...

> summary(diamonds)

     carat               cut        color        clarity          depth           table           price             x                y         
 Min.   :0.2000   Fair     : 1610   D: 6775   SI1    :13065   Min.   :43.00   Min.   :43.00   Min.   :  326   Min.   : 0.000   Min.   : 0.000  
 1st Qu.:0.4000   Good     : 4906   E: 9797   VS2    :12258   1st Qu.:61.00   1st Qu.:56.00   1st Qu.:  950   1st Qu.: 4.710   1st Qu.: 4.720  
 Median :0.7000   Very Good:12082   F: 9542   SI2    : 9194   Median :61.80   Median :57.00   Median : 2401   Median : 5.700   Median : 5.710  
 Mean   :0.7979   Premium  :13791   G:11292   VS1    : 8171   Mean   :61.75   Mean   :57.46   Mean   : 3933   Mean   : 5.731   Mean   : 5.735  
 3rd Qu.:1.0400   Ideal    :21551   H: 8304   VVS2   : 5066   3rd Qu.:62.50   3rd Qu.:59.00   3rd Qu.: 5324   3rd Qu.: 6.540   3rd Qu.: 6.540  
 Max.   :5.0100                     I: 5422   VVS1   : 3655   Max.   :79.00   Max.   :95.00   Max.   :18823   Max.   :10.740   Max.   :58.900  
                                    J: 2808   (Other): 2531                                                                                    
       z         
 Min.   : 0.000  
 1st Qu.: 2.910  
 Median : 3.530  
 Mean   : 3.539  
 3rd Qu.: 4.040  
 Max.   :31.800  
```
> - carat（克拉数），数值型变量
> - cut（划分等级），有序因子，分为Fair、Good、Very Good、Premiun、Ideal五个等级
> - clarity（洁净度），有序因子，分为SI1、VS2、SI2、VS1、VVS2等
> - price（价格），整型变量，介于326和18823之间

##### 创建一个二进制变量：isExpensive
在原始数据集中没有二分类变量，所以我们将price大于2400（**中位数**）定义为“**EXPENSIVE**”。
```R
> plot(density(diamonds$price), main = "Distribution of Diamond Price")
> polygon(density(diamonds$price), col = "pink")  # polygon()用于曲线内填充颜色
> abline(v = 2400, lwd = 3, col = "skyblue")  # abline（a, b）表示在图上添加一条y=a+bx的直线
```
<img src="http://img.blog.csdn.net/20180204153809674?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTHV6X0RhdGFfU2NpZW50aXN0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="80%" height="80%" />
```R
> diamonds$isExpensive <- diamonds$price > 2400
> summary(diamonds$isExpensive)

   Mode   FALSE    TRUE 
logical   26959   26981
```

#### 训练模型
因为样本的数目很多，所以通常情况下，我们选取其中的70%用于训练，剩余的30%用于测试。

```R
> set.seed(1)
> isTest <- runif(nrow(diamonds)) > 0.70  
# 生成和样本数量相同的[0, 1]的均匀分布随机数
> train <- diamonds[isTest == F, ]
> test <- diamonds[isTest == T, ]
```
##### 模型一：以cut作为预测因素
```R
> fit1 <- glm(isExpensive ~ cut, data = train, family = binomial())
> prob1 <- predict(fit1, newdata = test, type = "response")

# type = "link", 缺省值，给出线性函数预测值 
# type = "response", 给出概率预测值
# type = "terms"，给出各个变量的预测值

> library(ROCR)     # ROCR包提供多种评估分类执行效果的方法及可视化

载入需要的程辑包：gplots

载入程辑包：‘gplots’

The following object is masked from ‘package:stats’:

    lowess
    
> pred1 <- prediction(prob1, test$isExpensive)  # 转换prob1的格式
> performance(pred1, "auc")@y.values[[1]]
[1] 0.5806602
```
##### 模型二：以clarity作为预测因素
```R
> fit2 <- glm(isExpensive ~ clarity, data = train, family = binomial())
> prob2 <- predict(fit2, newdata = test, type = "response")
> pred2 <- prediction(prob2, test$isExpensive)
> performance(pred2, "auc")@y.values[[1]]
[1] 0.6551634
```
##### 模型三：以carat作为预测因素
```R
> fit3 <- glm(isExpensive ~ carat, data = train, family = binomial())
Warning message:
glm.fit:拟合機率算出来是数值零或一 

> prob3 <- predict(fit3, newdata = test, type = "response")
> pred3 <- prediction(prob3, test$isExpensive)
> performance(pred3, "auc")@y.values[[1]]
[1] 0.9905482
```


#### 评估模型性能
绘制ROC曲线。
```R
> plot(performance(pred1, "tpr", "fpr"), colorize = T, lwd = 3, main = "ROC Curves")
> plot(performance(pred2, "tpr", "fpr"), add = T, colorize = T, lwd = 3)
> plot(performance(pred3, "tpr", "fpr"), add = T, colorize = T, lwd = 3)
> abline(a = 0, b = 1, lty = 2, lwd = 3, col = "black")
```
<img src="http://img.blog.csdn.net/20180204172557511?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTHV6X0RhdGFfU2NpZW50aXN0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="70%" height="70%" />

#### 结论
-  可以通过`fit <- glm() `创建logistic回归模型，并设置参数`family = binomial()`；
-  从ROC曲线以及AUC均可以看出，用**carat**作为预测因素的效果最佳。

#### 参考
> - http://yaojenkuo.io/diamondsROC.html#model-2-clarity-as-the-only-predictor
> - http://blog.sina.com.cn/s/blog_72f52d290102w3gv.html
> - http://www.qingpingshan.com/m/view.php?aid=361677
