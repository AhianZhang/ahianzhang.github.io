---
title: "Golang Array 和 Slice 区别"
date: 2022-04-22T16:30:54+08:00
tags: ["总结"]
categories: ["Golang"]
draft: true
---

开始学习 Golang 时，对于 array 和 slice 的理解并不深入，还有很多疑惑的困扰着我。于是就深入的研究了一下。

# Array 与 Slice 的 区别
1. array 是固定长度的，声明时必须是常量大小，定义好后不可更改，更不会扩容，超出访问地址后会 panic；
2. array 是值类型，将旧的 array 赋值给新的 array 相当于两个完全独立的 array；
3. slice 是 struct 类型，可动态声明 length 和 capacity 大小，底层基于 array ，但是 slice 可以表示 array 的部分元素，修改 slice 内容会修改所引用 array 的值。

# Array/Slice 的 len() 和 cap()

array 的 length 和 capacity 是相同的，不能进行扩容；

slice 中的 len()：是指当前 slice 能够访问的元素；

slice 中的 cap()：是指当前 slice 指针的开头到数组的最后一位，slice 使用 append 进行函数追加时，能够进行动态扩容

# Slice 的 length 和 capacity 计算

```go
arr := [3]int{1,2,3} // 假设此数组的 cap 为 k
sli := arr[i:j] // [i,j)
len: j - i
cap: k - i

```



# Slice 扩容

# Slice 的 len() 和 cap()

slice 中的 len()：是指当前 slice 能够访问的元素；

slice 中的 cap()：是指当前 slice 指针的开头到数组的最后一位



# 参考文档

https://go.dev/blog/slices
