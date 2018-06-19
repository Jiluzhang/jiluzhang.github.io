---
title: A Data Scientist Interview Question from Verizon
description: 一道关于推荐系统的面试题
categories:
 - case interview
tags:
 - recommendation system
---

> **If you were a data scientist at a web company that sells shoes, how would you build a system that recommends shoes to a visitor? **

- Break the answer to two components: **Data science** and **Data engineering**.
- Let's discuss the data science element first. If it is a new company that does not have much historical user data, go with item to item similarity. If the number of different items/shoes is extremely large, consider using **matrix factorization** techniques to reduce the dimensions.
- If you have historical data around user preferences (e.g. ratings of shoes), you can use a **collaborative filter** type approach. Mention specifically the rows and columns of the matrix you generate with either approach. Then discuss what kink of **similarity metrics** you would try. E.g. euclidean distance, Jaccard similarity, cosine distance.
- After explaining the algorithmic aspect, you would discuss the data engineering side. Propose an engineering infrastructure that scales to millions of users/shoes where recommendations are generated in real time. As an example, you can stream the user data to a **S3 bucket**. You can perform the matrix analysis on a nightly basis, precompute the entire set of recommendations on a per user basis, and store this in a inmemory database such as Redis. Then you could build a **REST API** that would query the database and respond with the recommendation given a user id.

**Notes**

1. matrix factorization, 矩阵分解，是指将矩阵拆解为数个矩阵的乘积，可分为三角分解、满秩分解、QR分解、Jordan分解和SVD（奇异值）分解等；

2. collaborative filter，协同过滤，是指利用某兴趣相投以及拥有共同经验的群体的喜好来推荐用户感兴趣的信息，个人通过**合作的机制**给予信息相当程度的回应（如评分），并记录下来以达到过滤的目的，进而帮助别人筛选信息。回应不一定局限于特别感兴趣的，特别不感兴趣信息的记录也相当重要；

3. euclidean distance，欧几里得距离，衡量空间各点的绝对距离，和各个点所在的位置坐标直接相关；

   Jaccard similarity，杰卡德相似度，衡量两个集合相似度的一种指标；

   cosine distance，余弦距离，用向量空间中两个夹角的余弦值作为衡量两个个体间差异的大小。





































