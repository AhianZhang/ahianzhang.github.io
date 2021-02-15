---
title: "条件概率、全概率与贝叶斯公式"
date: "2019-05-18 12:17:40"
tags: ["概率论"]
categories: ["算法"]
mathjax: true
---

# 条件概率公式

设事件A 发生的概率为 P(A), 事件B 发生的概率为 P(B),则在``事件B发生的情况下事件A发生的概率（A given B 的概率）``为：

$$ P(A|B)=\frac{P(AB)}{P(B)} $$

# 全概率公式
如果直接求事件A 的概率比较困难的时候，可以将事件A发生的概率分成一个个小的事件B的概率

$$ P(A)=\sum \limits_n{P(B_{n})P(A|B_{n})} $$

# 贝叶斯公式

$$ P(B_{n}|A)=\frac{P(A|B_{n})P(B_{n})}{\sum \limits_n P(A|B_{n})P(B_{n})} $$
