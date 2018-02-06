---
title: 多元线性回归（以医疗费用为例）
description: 了解多元线性回归的基本原理以及步骤。
categories:
 - machine learning
tags:
 - 多元回归
---





相较于一元线性回归，多元线性回归是用来确定2个或2个以上变量间的统计分析方法，其基本的分析方法和一元线性回归是类似的。
> **优点：**
> >- 可适用于几乎所有的数据；
> >- 提供了特征与结果之间关系的强度和大小的估计。 
>
> **缺点：**
> >- 对数据做出了很强的假设；
> >- 模型形式必须事先指定；
> >- 不能很好地处理缺失数据。
>
> **基本步骤：**对选取的多元数据集定义数学模型，进行参数估计，对参数进行显著性检验、残差分析、异常点检验，最后确定回归方程进行模型预测。特别地，对于多元回归方程有一项很重要的操作就是自变量的优化，要挑选出相关性最显著的自变量，同时去除不显著的自变量。



接下来以医疗费用为例，通过分析病人的数据，来预测这部分群体的平均医疗费用，从而来为年度保费价格的设定提供参考。

---
#### 收集和观察数据

>数据来源：https://github.com/stedy/Machine-Learning-with-R-datasets/find/master里的insurance.csv文件，它是美国人口普查局（U.S. Census Bureau）的人口统计资料。

文件包含1338个案例，即目前已经登记过的保险计划受益者以及表示病人特点和历年计划计入的总医疗费用的特征。这些特征分别是：
>- age：整数，受益者的年龄（不包括超过64岁的人，因为这部分人群的费用由政府承担）；
>- sex：保单持有人的性别，male或者female；
>- bmi：身体质量指数（Body Mass Index, BMI），它等于体重（公斤）除以身高（米）的平方，理想的BMI指数在18.5和24.9之间；
>- children：整数，保险计划中包括的孩子或者受抚养者的数量；
>- smoker：被保险人是否吸烟，yes或者no；
>- region：根据受益人在美国的居住地，分为4个地理区域，分别是northeast, southeast, southwest和northwest。

如何将这些变量与已结算的医疗费用联系起来很重要，在回归分析中，特征之间的关系通常是由使用者指定，而不是自动检测出来的。

---

#### 探索和准备数据

```R
> insurance <- read.csv("insurance.csv", stringsAsFactors = T)
> str(insurance)

'data.frame':   1338 obs. of  9 variables:
 $ age     : int  19 18 28 33 32 31 46 37 37 60 ...
 $ sex     : Factor w/ 2 levels "female","male": 1 2 2 2 2 1 1 1 2 1 ...
 $ bmi     : num  27.9 33.8 33 22.7 28.9 ...
 $ children: int  0 1 3 0 0 0 1 3 2 0 ...
 $ smoker  : Factor w/ 2 levels "no","yes": 2 1 1 1 1 1 1 1 1 1 ...
 $ region  : Factor w/ 4 levels "northeast","northwest",..: 4 3 3 2 2 3 3 2 1 2 ...
 $ charges : num  16885 1726 4449 21984 3867 ...
 $ age2    : num  361 324 784 1089 1024 ...
 $ bmi30   : num  0 1 1 0 0 0 1 0 0 0 ...
```

>- read.csv()函数读入数据，用stringsAsFactors = T将名义变量转换成因子变量；
>- str()函数查看数据结构，确认各个变量的读取是否有异常；
>- summary()函数查看charges的分布情况；
>- hist()函数可得到数据分布的直方图，表明保险费用的分布是右偏的，其平均数大于中位数，同时我们可以知道绝大数的人每年的费用在0到15000美元之间。


接下来面临的一个问题是如何处理因子类型的特征，因为回归模型需要每一个特征值都是数值型的。



##### 探索特征之间的关系（相关系数矩阵）

在使用回归模型拟合数据之前，有必要确定自变量与因变量之间以及自变量之间是如何相关的。相关系数矩阵（correlation matrix）可以为每一对变量之间的关系提供一个相关系数。
```R
> cor(insurance[c("age", "bmi", "children", "charges")])

               age       bmi   children    charges
age      1.0000000 0.1092719 0.04246900 0.29900819
bmi      0.1092719 1.0000000 0.01275890 0.19834097
children 0.0424690 0.0127589 1.00000000 0.06799823
charges  0.2990082 0.1983410 0.06799823 1.00000000
```
>- cor()函数为insurance中的4个数值型变量创建了一个相关系数矩阵。


相关系数矩阵中存在着一些显著的相关，例如age和bmi，这就说明随着年龄的增长，身体质量指数也会增加。此外，age和charges、bmi和charges以及children和charges也都呈现出相关性。



##### 可视化特征之间的关系（散点图矩阵）

散点图矩阵（sactterplot matrix）是一个简单地将散点图集合排列在网格中，并且包含相互紧邻的多种因素的图表。
```R
> pairs(insurance[c("age", "bmi", "children", "charges")])
```

>- pairs()函数可用于绘制散点图矩阵。


在散点图矩阵中，每个行和列的交叉点所在的散点图表示其所在的行与列的两个变量的相关关系，在对角线上方的图和下方的图是互为转置的。大部分的散点图看上去像是随机密布的点，但还是有一些会呈现出某种趋势，例如age和charges之间的关系呈现出几条相对的直线，而bmi和charges则构成了两个不同的群体。
```R
> pairs.panels(insurance[c("age", "bmi", "children", "charges")])
```

>- pairs.panels()函数属于R包psych，其产生的散点图矩阵有着更多的信息。


对角线的上方表示的是相关系数矩阵，对角线上的直方图描绘了每个特征的数值分布，对角线下方的散点图带有更多的可视化信息。

散点图中的椭圆被称为**相关椭圆** （correlation ellipse），它提供了一种变量之间是如何密切相关的可视化信息。相关椭圆中心的点表示x轴变量的均值和y轴变量的均值。两个变量的相关性由椭圆的形状所表示，椭圆越被拉伸，其相关性越强。若形状近似于圆，如bmi和children，则表示一种非常弱的相关性。

散点图中的曲线是**局部回归平滑** （loess smooth）曲线，它表示x轴和y轴变量之间的一般关系。例如，对于age和bmi而言，局部回归平滑曲线是一条倾斜的逐步上升的线，这说明bmi会随着age的增长而增长，这个结论从相关系数矩阵中也能得出。

---
#### 基于数据训练模型

```R
> ins_model <- lm(charges ~ age + children + bmi + sex + smoker + region, data = insurance)
> ins_model

Call:
lm(formula = charges ~ age + children + bmi + sex + smoker + 
    region, data = insurance)

Coefficients:
    (Intercept)              age         children              bmi          sexmale        smokeryes  regionnorthwest  
       -11938.5            256.9            475.5            339.2           -131.3          23848.5           -353.0  
regionsoutheast  regionsouthwest  
        -1035.0           -960.1  
```
>- lm()函数用于对数据拟合线性回归模型；
>- 直接输入模型对象名称ins_model，就可以输出估计的系数。


需要注意的是，我们在模型公式中仅仅指定了6个变量，但是却输出8个系数（除截距外）。这是因为lm()函数自动将**虚拟编码** （dummy coding）应用于模型所包含的每一个因子类型的变量中。当添加一个虚拟编码的变量到回归模型中时，一个类别总是被排除在外作为参照类别，然后估计的系数就是相对于参照类别解释的。在我们的模型中，自动保留了sexfemale、smokerno和regionnortheast变量，使东北地区的女性非吸烟者作为参照组。因此，相对于女性来说，男性每年的医疗费用要少131.30美元，吸烟者平均多花费23848.50美元，远远超过非吸烟者。此外，模型中另外3个地区的系数是负的，这意味着东北地区倾向于具有最高的平均医疗费用。

线性回归模型的结果是合乎逻辑的，因为高龄、吸烟和肥胖往往与其它健康问题联系在一起，而额外的家庭成员或者受抚养者可能会导致就诊次数增加和预防保健费用的增加。

---



#### 评估模型的性能

```R
> summary(ins_model)    ### summary()给出评估模型性能的信息。

Call:

lm(formula = charges ~ age + children + bmi+ sex + smoker +

   region, data = insurance)
   

Residuals:

    Min       1Q   Median      3Q      Max

-11304.9 -2848.1   -982.1   1393.9 29992.8


Coefficients:

                Estimate Std. Error t valuePr(>|t|)   

(Intercept)     -11938.5      987.8 -12.086  < 2e-16 ***

age                256.9       11.9 21.587  < 2e-16 ***

children           475.5      137.8  3.451 0.000577 ***

bmi                339.2       28.6 11.860  < 2e-16 ***

sexmale           -131.3      332.9 -0.394 0.693348   

smokeryes        23848.5      413.1 57.723  < 2e-16 ***

regionnorthwest   -353.0     476.3  -0.741 0.458769   

regionsoutheast  -1035.0     478.7  -2.162 0.030782 * 

regionsouthwest   -960.0     477.9  -2.009 0.044765 * 

---
```

>评估模型的性能主要有3个方面：

>>- Residuals（残差）部分提供了预测误差的主要统计量；

>>- 星号（如***）表示模型中每个特征的预测能力；

>>- 多元R方值（也称为判定系数）提供度量模型性能的方式。 （**0.75是相当不错的**）

---

#### 提高模型的性能

因为回归模型通常是由使用者选择特征和设定模型，所以如果具备了关于特征是如何与结果相关的学科知识，我们就可以使用该信息对模型进行设定，从而提高模型的性能。   

##### 模型的设定——添加非线性关系

线性回归的假设是自变量和因变量之间的关系是线性的，但是这并不一定正确。例如，对于所有年龄值而言，年龄对医疗费用的影响可能不是恒定的，对于年龄最大的人群，治疗费用可能就会比较高。 
```R
> insurance$age2 <- insurance$age^2 
```

##### 转换——数据值变量转换为二进制指标

如果一个特征的影响不是累积的，而是取值达到一个给定的阈值后才产生影响的，我们就可以通过创建一个二进制指标变量来建立关系。例如，对于在正常体重范围内的人来说，BMI对医疗费用的影响可能为0，但是对于肥胖者（即BMI不低于30）来说，它可能与较高的费用密切相关。因此，当BMI大于等于30时，我们设定为1，否则设定为0。
```R
> insurance$bmi30 <-ifelse(insurance$bmi >= 30, 1, 0)
```

##### 模型的设定——加入相互作用的影响

如果某些特征对因变量有综合影响，例如，吸烟和肥胖可能分别都会产生有害的影响，但是假设它们的共同影响可能会比它们每一个单独影响之和更糟糕是合理的。当两个特征存在共同的影响时，称为相互作用（interaction）。如果怀疑两个变量相互作用，可以通过在模型中添加它们的相互作用来检验这一假设。

##### 改进的回归模型

基于医疗费用与患者特点相联系的学科知识，我们来改进回归模型：

- 增加一个非线性年龄项
- 为肥胖创建一个指标
- 指定肥胖与吸烟之间的相互作用   


```R
> ins_model2 <- lm(charges ~ age +age2 + children + bmi + sex + bmi30*smoker + region, data = insurance)

> ins_model2

Call:

lm(formula = charges ~ age + age2 +children + bmi + sex + bmi30 *

   smoker + region, data = insurance)

 
Coefficients:

   (Intercept)              age             age2         children              bmi          sexmale            bmi30 

       134.251          -32.685            3.732          678.561          120.020         -496.824        -1000.140 

     smokeryes  regionnorthwest  regionsoutheast  regionsouthwest  bmi30:smokeryes 

     13404.687         -279.204         -828.547        -1222.644        19810.753 

 
> summary(ins_model2)

Call:

lm(formula = charges ~ age + age2 +children + bmi + sex + bmi30 *

   smoker + region, data = insurance)

 
Residuals:

    Min       1Q   Median      3Q      Max

-17296.4 -1656.0  -1263.3   -722.1 24160.2

 
Coefficients:

                  Estimate Std. Error t valuePr(>|t|)   

(Intercept)       134.2509 1362.7511   0.099 0.921539   

age               -32.6851    59.8242 -0.546 0.584915   

age2                3.7316     0.7463  5.000 6.50e-07 ***

children          678.5612   105.8831  6.409 2.04e-10 ***

bmi               120.0196    34.2660  3.503 0.000476 ***

sexmale          -496.8245   244.3659 -2.033 0.042240 * 

bmi30           -1000.1403   422.8402 -2.365 0.018159 * 

smokeryes       13404.6866   439.9491 30.469  < 2e-16 ***

regionnorthwest  -279.2038  349.2746  -0.799 0.424212   

regionsoutheast  -828.5467  351.6352  -2.356 0.018604 * 

regionsouthwest -1222.6437   350.5285 -3.488 0.000503 ***

bmi30:smokeryes 19810.7533   604.6567 32.764  < 2e-16 ***

---

Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’1

Residual standard error: 4445 on 1326degrees of freedom

Multiple R-squared:  0.8664,   Adjusted R-squared:  0.8653

F-statistic: 781.7 on 11 and 1326 DF,  p-value: < 2.2e-16
```

相较于第一个模型，判定系数从0.75提高到0.87，这说明回归模型的性能得到了提高，现在的模型能够解释医疗费用变化87%。此外，我们关于模型函数形式的理论也得到了验证，高阶项age2和肥胖指标bmi30在统计学上都是显著的。肥胖和吸烟之间的相互作用影响很大，除了单独吸烟增加的超过13404美元的费用外，肥胖的吸烟者每年要另外花费19810美元，这说明吸烟可能会加剧与肥胖相关的疾病。

