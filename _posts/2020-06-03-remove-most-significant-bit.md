---
title: "Remove most significant bit"
date: 2020-06-03 14:00:00 -07:00
categories:
  - Tricks
tags:
  - C++
  - Bit Operation
---

To remove the most significant bit of a number ```x``` with its bit size ```k```, we could apply

```
x &= (1 << (k-1))-1;
```
```
e.g. x = 0b11111111, k = 8

(1 << (8-1)) = 0b10000000
(1 << (8-1))-1 = 0b01111111

0b11111111 & (1 << (8-1))-1
= 0b11111111 & 0b01111111
= 0b01111111 (most significant bit removed!)
```
