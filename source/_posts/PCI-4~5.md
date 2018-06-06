---
title: PCI-4~5
date: 2016-12-06 18:01:47
categories: programming
tags: course
mathjax: true
---

### 4.搜索引擎

#### 主要内容

```sql
select w0.urlid,w0.location,w1.location from wordlocation w0,wordlocation w1 where w0.urlid=w1.urlid
and w0.wordid=10
and w1.wordid=17//查找同一张表内同时有word_id=10和17的urlid的记录并组合各自的location展现
```

### 5.最优化

<!-- more -->

##### 5.5 simulated annealing

$p=e^{-{h-l}/T}$

公式怀疑，就是如果相差越大越不可能将比当前差的选择当做当前选择

