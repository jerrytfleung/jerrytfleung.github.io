---
title: "Rotate 2D square array"
date: 2020-8-01 20:30:00 -07:00
categories:
  - tricks
tags:
  - c++
  - array
---

# Problem
Rotate 2D n by n array to 90 degree clockwise and 90 degree anti-clockwise in-place.

e.g.
```
1 2 3    7 4 1
4 5 6 => 8 5 2
7 8 9    9 6 3
```

```
1 2 3    3 6 9
4 5 6 => 2 5 8
7 8 9    1 4 7
```

## The trick is to do it in 2 steps.
Reverse whole array and then swap to symmetry
```
1 2 3    7 8 9    7 4 1
4 5 6 => 4 5 6 => 8 5 2
7 8 9    1 2 3    9 6 3
```

Reverse within and then swap to symmetry
```
1 2 3    3 2 1    3 6 9
4 5 6 => 6 5 4 => 2 5 8
7 8 9    9 8 7    1 4 7
```

## Is the operation order important?
Swap to symmetry first and reverse whole array  (Will become 90 anticlockwise)
```
1 2 3    1 4 7    3 6 9
4 5 6 => 2 5 8 => 2 5 8
7 8 9    3 6 9    1 4 7
```

Swap to symmetry first and reverse within  (Will become 90 clockwise)
```
1 2 3    1 4 7    7 4 1
4 5 6 => 2 5 8 => 8 5 2
7 8 9    3 6 9    9 6 3
```

## How about rotate 180 degree?
Reverse whole and reverse within
```
1 2 3    7 8 9    9 8 7
4 5 6 => 4 5 6 => 6 5 4
7 8 9    1 2 3    3 2 1
```

Reverse within and reverse whole (Same result)
```
1 2 3    3 2 1    9 8 7
4 5 6 => 6 5 4 => 6 5 4
7 8 9    9 8 7    3 2 1
```
