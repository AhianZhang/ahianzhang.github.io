---
title: "master 公式"
date: "2018-05-26 11:26:46"
tags: ["算法"]
categories: ["算法"]
---
**master公式**
****
T(N) = a*T(N/b) + O(N<sup>d</sup>)
****
N：样本量
T：时间复杂度
a：样本量发生的次数
b：将样本量进行分治
c：执行子过程之外其余过程的时间复杂度

**用途**：计算递归算法的时间复杂度

**快速计算**
- log<sub>b</sub>a > d -> 复杂度为O(N<sup>log<sub>b</sub>a</sup>) 
- log<sub>b</sub>a = d -> 复杂度为O(N<sup>d</sup> * logN) 
- log<sub>b</sub>a < d -> 复杂度为O(N<sup>d</sup>)

**适用范围**
递归调用使，划分的子过程规模一样，
奇数问题：递归不看常数

任何递归行为都可以变成非递归
递归的本质是 压栈