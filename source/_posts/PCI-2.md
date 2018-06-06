---
title: PCI-2
date: 2016-07-24 21:52:10
categories: programming
tags: course
mathjax: true
---

### 2.提供推荐

#### 主要内容
**1.相似度评价体系：**
**1.1欧几里得距离评价(Euclidean distance)**

本书中，先取出共同评分的电影，将两个对象对于同一电影评分的差值的平方求和加一再取倒数，值越接近1越相关。

```python
    def sim_distance(prefs,person1,person2):
      # Get the list of shared_items
      si={} 
      for item in prefs[person1]:
        if item in prefs[person2]:
            si[item]=1
      # if they have no ratings in common, return 0
      if len(si)==0: return 0
      # Add up the squares of all the differences
      sum_of_squares=sum([pow(prefs[person1][item]-prefs[person2][item],2)
                          for item in prefs[person1] if item in prefs[person2]])
      return 1/(1+sum_of_squares)
```

**1.2皮尔逊相关度评价(Pearson correlation)**

<!-- more -->

 修正了夸大分值（一方普遍比另一方评分高），本书中，先取出共同评分的电影，将各自的电影评分求和sum1,sum2，求平方和sp1,sp2，并求对同一电影的评分之积ps, 

```python

     # Returns the Pearson correlation coefficient for p1 and p2
     def sim_pearson(prefs,p1,p2):
       # Get the list of mutually rated items
       si={}
       for item in prefs[p1]:
         if item in prefs[p2]: si[item]=1
       # Find the number of elements
       n=len(si)
       # if they are no ratings in common, return 0
       if n==0: return 0
       # Add up all the preferences
       sum1=sum([prefs[p1][it] for it in si])
       sum2=sum([prefs[p2][it] for it in si])
       # Sum up the squares
       sum1Sq=sum([pow(prefs[p1][it],2) for it in si])
       sum2Sq=sum([pow(prefs[p2][it],2) for it in si])
       # Sum up the products
       pSum=sum([prefs[p1][it]*prefs[p2][it] for it in si])
       # Calculate Pearson score
       num=pSum-(sum1*sum2/n)
       den=sqrt((sum1Sq-pow(sum1,2)/n)*(sum2Sq-pow(sum2,2)/n))
       if den==0: return 0
r=num/den
return r
```

![无奈啊](https://github.com/BaichuanWu/prictures/raw/master/PCI/2-2png.png)

HEXO解析math出错无奈传图

**2 协同过滤(Collaborative Filtering)是推荐系统中应用最为广泛的技术之一**：

2.1	基于用户协同过滤(user base Collaborative Filtering)

​	找到和各个用户的相似度，推荐时，按下表

| 相似用户 | 目标用户相似度 | 对项目A评分  | 计算项目A评分 |
| ---- | ------- | ------- | ------- |
| 用户1  | 0.1     | 3       | 0.3     |
| 用户2  | 0.2     | 4       | 0.8     |
| 用户3  | 0.3     | 5       | 1.5     |
| 汇总   | 0.6     |         | 2.7     |
| 预计评分 |         | 2.7/0.6 | 4.5     |



2.2  基于项目协同过滤(item base Collaborative Filtering)

​	找到各项目相似度，推荐时，按下表

| 已评分项目 | 评分   | 项目A相似度 | 计算项目A评分 |
| ----- | ---- | ------ | ------- |
| 项目1   | 2    | 0.1    | 0.2     |
| 项目2   | 3    | 0.2    | 0.6     |
| 项目3   | 4    | 0.3    | 1.2     |
| 汇总    |      | 0.6    | 2       |
| 预计评分  |      | 2/0.6  | 3.333   |

2.3 (model base)

待补充

> Item-based filtering usually outperforms user-based filtering in sparse datasets, and the two per-
> form about equally in dense datasets.


​				
​			
​		
​	
