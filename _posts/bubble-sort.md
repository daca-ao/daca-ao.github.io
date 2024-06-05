---
title: 冒泡排序
date: 2022-11-08 22:36:37
mathjax: true
tags:
- 算法
- 排序算法
---

学习数据结构的时候最早接触到的排序算法。

<!-- more -->

[冒泡排序](https://zh.wikipedia.org/wiki/冒泡排序)属于**比较排序**。比较过程中的“冒泡”，指的是：**重复**地访问要排序的数列，对相邻的元素进行比较，如果次序不符合要求，再将元素**交换顺序**。
* 升序排列：如果左元素 > 右元素，则进行交换
* 降序排列：如果左元素 < 右元素，则进行交换

名称由来：越小（大）的元素经过交换后会逐渐“浮”到数列的顶端。

在一次冒泡过程结束后，最大（小）的元素肯定在最后的位置上。

时间复杂度最好是 $O(n)$，最坏是 $O(n^2)$。


# 代码实现

伪代码：

```js
function bubble_sort(array, length) {
    var i, j;
    for (i from 0 to length-1) {
        for (j from 0 to length-1-i) {
            if (array[j] > array[j+1])
                swap(array[j], array[j+1])
        }
    }
} 
```

代码实现：

```java
public class BubbleSort {

    public static void sort(int[] a) {
        final int n = a.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - 1 - i; j++) {
                if (a[j] > a[j + 1]) {  // ascending order
                    final int temp = a[j + 1];
                    a[j + 1] = a[j];
                    a[j] = temp;
                }
            }
        }
    }
}
```

改进 1：引入提前中断位

```java
/**
 * 最好情况：序列本身有序，比较次数 n-1，无数据交换，时间复杂度为 O(n)
 * 最坏情况：序列本身逆序，需要比较以及交换 n(n-1)/2 次，时间复杂度为 O(n^2)
 */
public static void sortE(int[] a) {
    final int n = a.length;
    boolean needSwap = true;
    for (int i = 0; i < n - 1 && needSwap; i++) {
        needSwap = false;
        for (int j = 0; j < n - 1 - i; j++) {  // 从待排序元素开始遍历
        // 如果跑完一轮 needSwap 仍是 false，证明已经有序，就不再继续冒泡了。
            if (a[j] > a[j + 1]) {
                swap(a, j, j + 1);  // 交换
                needSwap = true;
            }
        }
    }
}
```

另一种写法：

```java
public static void sort(int[] a, int n) {
    for (int i = n; i > 1 && bubble(a, i); i--);
}

boolean bubble(int[] a, int n) {
    boolean swapped = false;
    for (int i = 0; i < n - 1; i++) {
        if (a[i] > a[i + 1]) {
            swap(a[i], a[i + 1]);
            swapped = true;
        }
    }
    return swapped;
}
```

改进 2：举一个极端的例子

```java
/*
 * 如果有 100 个数的数组，仅前面 10 个无序，后面 90 个都已经排好序且都大于前面 10 个数字
 * 那么第一遍遍历之后，最后发生交换的位置必小于 10，而且这个位置之后的数据必然有序
 * 记录下这个位置，第二次只要从数组头部遍历到这个位置就好了
 */ 
void bubbleSort(int[] a) {
    int k;
    int flag = a.length;
    while (flag > 0) {
        k = flag;
        flag = 0;
        for (int i = 1; i > k; ++i) {
            if (a[i - 1] > a[i]) {
                swap(a[i - 1], a[i]);
                flag = i;
            }
        }
    }
}
```
