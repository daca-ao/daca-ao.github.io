---
title: 分而治之算法 Divide and Conquer
date: 2022-11-13 12:09:42
tags:
- 算法
---

分而治之算法是 MapReduce 方法的基石，是数据挖掘的核心。

<!-- more -->

分而治之的本质是：

1. 将大的问题分成两个以上小的问题
2. 分别解决每个小问题
3. 将各小问题解答组合起来，得到原问题的解答

拆分出来的小问题通常与大问题相似，因此可以通过**递归**解决。

非递归：

```c++
template<class T>
bool MinMax(T w[], int n, T& Min, T& Max)
{// Locate min and max of w[0:n-1].
 // Return false if fewer than one element.
 // special cases, n <= 1

    if (n < 1) {
        return false;
    }

    if (n == 1) {
        Min = Max = 0;
        return true;
    }

    // initialize Min and Max
    int s;  // start point for loop
    if (n % 2) {  // n is odd
        Min = Max = 0;
        s = 1;
    } else {  // n is even, compare first pair
        if (w[0] > w[1]) {
            Min = 1;
            Max = 0;
        } else {
            Min = 0;
            Max = 1;
        }
        s = 2;
    }

    // compare remaining pairs
    for (int i = s; i < n; i += 2) {

        // find larger of w[i] and w[i+1]
        // then compare larger with w[Max]
        // and smaller with w[Min]
        if (w[i] > w[i+1]) {
            if (w[i] > w[Max]) {
                Max = i;
            }
            if (w[i+1] < w[Min]) {
                Min = i + 1;
            }
        } else {
            if (w[i+1] > w[Max]) {
                Max = i + 1;
            }
            if (w[i] < w[Min]) {
                Min = i;
            }
        }
    }

    return true;
}
```

# 应用

由分而治之算法衍生出来的算法，常用的有[归并排序](/2023/02/25/merge-sort)和[快速排序](/2023/02/26/quick-sort)两种。

解决线性代数的矩阵乘法，可以使用分而治之算法。

分而治之算法可以应用到很多生活场景，比如求解递归方程、找出伪币、找最轻和最重、得出距离最近的点对等。
