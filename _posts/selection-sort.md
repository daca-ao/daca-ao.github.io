---
title: 选择排序
date: 2022-11-11 21:53:08
mathjax: true
tags:
- 算法
- 排序算法
---

[选择排序](https://zh.wikipedia.org/wiki/%E9%80%89%E6%8B%A9%E6%8E%92%E5%BA%8F)，也称直接选择排序。

<!-- more -->

# 基本思想

选择排序属于比较排序，大体步骤如下：
1. 在待排序序列中**选择**最小（大）的元素，将其**移动**到该序列的起始位置（或末尾）；
2. 再从剩余未排序元素中继续选择最小（大）元素，然后移动到已排序部分的末尾（或前面）；
3. 重复操作，直到序列有序。

选择排序是不稳定的排序：如果当前元素比某一个元素小，而该较小的元素又出现在一个和当前元素相等的元素后面，则稳定性就被破坏。


# 复杂度评估

* 交换操作：0~(n-1) 次
* 比较操作：n(n-1)/2
* 赋值操作（元素移动次数）：0~3(n-1)

在数据量（n）比较小的时候，选择排序比冒泡排序快：然而，这种实际适用的场合非常罕见。
* 可以说，选择排序是综合性能最差的排序算法。


# 代码实现

```java
public static void selectionSort(int[] a) {
    final int length = a.length;
    int minIndex;
    for (int i = 0; i < length; i++) {
        minIndex = i;
        for (int j = i + 1; j < length; j++) {
            if (a[j] < a[minIndex]) {
                minIndex = j;
            }
        }
        if (minIndex != i) {
            final int temp = a[minIndex];
            a[minIndex] = a[i];
            a[i] = temp;
        }
    }
}
```

上述程序的缺点：即使元素已经按序排列，程序依然继续运行。如即使在第二次循环后数组元素可能已经按序排列，首层 for 循环还需要执行 n-1 次。

为了终止不必要的循环，在查找最大元素期间，可顺便检查数组是否已按序排列。

改进版：最好情况仅比较 n-1 次

```java
public static void sortEarlyTerm(int[] a) {

    final int length = a.length;
    boolean sorted = false;
    for (int size = length; !sorted && size > 1; size--) {
        int maxIndex = 0;
        sorted = true;
        // 如果经历一轮遍历后 sorted 还是 true，说明元素都是顺序的，就不需要再排序了
        for (int i = 1; i < size; i++) {
            if (a[i] > a[maxIndex]) {
                maxIndex = i;
            } else {
                sorted = false;
            }
        }
        if (maxIndex != size - 1) {
            final int temp = a[maxIndex];
            a[maxIndex] = a[size - 1];
            a[size - 1] = temp;
        }
    }
}
```
