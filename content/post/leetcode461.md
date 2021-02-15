---
title: "hamming-distance"
date: "2019-05-28 11:18:18"
tags: ["LeetCode"]
categories: ["算法"]
description: "汉明码类的题，使用异或、与、移位进行操作"
---

The **Hamming distance** between two integers is the number of positions at which the corresponding bits are different.

Given two integers `x` and `y`, calculate the Hamming distance.

**Note:**
0 ≤ `x`, `y` < 231.

**Example:**

```
Input: x = 1, y = 4

Output: 2

Explanation:
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑

The above arrows point to positions where the corresponding bits are different.
```

**Consideration**

This problem is also have a relationship with '^'  , Think about it :

***1(0001)*** and ***4(0100)*** their Xor is ***5(0101)***  .  next we use **&** Operator to calculate the number of ***1  ,*** let the binary & 1 ,if result is 1 ,sum's up , use ***>>*** to move the postion.

```
5(0101)
0&1  = 0            pass
01&01 = 1           sum
010&001 = 0         pass
0101&0001 = 1       sum
```

so,the anwser is ***2***

**Solution**

```java
class Solution {
 public int hammingDistance(int x, int y) {
	         
         int sum = 0;
         int xor = x ^ y;
	 
	    for(int i = 0;i<32;i++){
	    	
             sum+=(xor>>i)&1;
	    }
	       
		return sum;
	    }
}
```

tips: if you don't understand the Xor ,you can see my[ older article](http://younge.group/2018/07/08/136-single-number/) which about Xor  .and the flowing is about **Hamming Distance** from wiki.

 

[![461. Hamming Distance](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b4/Hamming_distance_3_bit_binary.svg/140px-Hamming_distance_3_bit_binary.svg.png)](https://en.wikipedia.org/wiki/File:Hamming_distance_3_bit_binary.svg)

3-bit binary [cube](https://en.wikipedia.org/wiki/Cube) for finding Hamming distance

[![461. Hamming Distance](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6e/Hamming_distance_3_bit_binary_example.svg/140px-Hamming_distance_3_bit_binary_example.svg.png)](https://en.wikipedia.org/wiki/File:Hamming_distance_3_bit_binary_example.svg)

Two example distances: 100→011 has distance 3; 010→111 has distance 2



The minimum distance between any two vertices is the Hamming distance between the two binary strings.




