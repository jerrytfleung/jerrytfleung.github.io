---
title: "Rotating 2D square array"
date: 2020-07-30 20:30:00 -07:00
categories:
  - tricks
tags:
  - c++
  - array
---
#Problem  
Rotate 2D n by n array to 90 degree clockwise and 90 degree anti-clockwise in-place.  
[[1, 2, 3],  
[4, 5, 6],  
[7, 8, 9]]   

e.g.  
1 2 3    7 4 1  
4 5 6 => 8 5 2  
7 8 9    9 6 3  

1 2 3    3 6 9  
4 5 6 => 2 5 8  
7 8 9    1 4 7  

The trick is to do it in 2 steps.  

1. Reverse whole array and then swap to symmetry  
1 2 3    7 8 9    7 4 1
4 5 6 => 4 5 6 => 8 5 2
7 8 9    1 2 3    9 6 3  

2. Reverse within and then swap to symmetry  
1 2 3    3 2 1    3 6 9  
4 5 6 => 6 5 4 => 2 5 8  
7 8 9    9 8 7    1 4 7  

Is the operation order important?  
1.
1 2 3    7 4 1
4 5 6 => 8 5 2
7 8 9    9 6 3
