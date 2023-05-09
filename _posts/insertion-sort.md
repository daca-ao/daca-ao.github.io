---
title: 插入排序
date: 2022-11-12 18:00:48
mathjax: true
tags:
- 算法
- 排序算法
---

插入排序也是比较排序的一种。

<!-- more -->

在通过插入排序构建有序序列时，采用 in-place 排序：

* 对于未排序数据，在已排序序列中从后向前扫描，找到相应的位置并**插入**未排序数据，直到全部元素按序插入完成；
* 从后向前扫描过程中，需要反复把已排序元素逐步**向后挪位**，为最新元素提供插入空间；
* 只需要用到 $O(1)$ 的额外空间


# 算法描述

以升序排序为例：
1. 从第一个元素开始，该元素可认为已经被排序
2. 从未排序序列中取出下一个元素 temp
3. 从后向前扫描已经排序的序列
4. 若已排序序列中某元素 A 大于 temp，则将 A 往后移动一个位置
5. 重复步骤 3 ~ 4，直到找到不大于 temp 的已排序元素
6. 将新元素 temp 插入到该位置（不大于 temp 的已排序元素）之后
7. 重复步骤 2 ~ 6，最终完成排序。

如果比较操作的代价比交换操作大的话，可以采用二分查找法来减少比较操作的数目。

```java
public static int binarySearch(int[] arr, int start, int end, int hkey) {

    while (start <= end) {
        int mid = start + (end - start)/2;  // 防止溢位
        if (arr[mid] > hkey) {
            end = mid - 1;
        } else if (arr[mid] < hkey) {
            start = mid + 1;
        } else {
            return mid;  
        }
    }

    return -1;
}
```

该算法可认为是插入排序的一个变种，称为二分查找插入排序。  
⚠️注意：**二分查找只对有序序列有效**。


# 复杂度评估

**空间复杂度**为 $O(1)$。

**时间复杂度**的最好情况：序列本身有序且顺序，只需比较 n-1 次

最坏情况：序列原本逆序
* 需要比较 $n(n-1)/2$ 次，赋值 $[n(n-1)/2]-(n-1) = (n-1)(n-2)/2$ 次
* 因此时间复杂度为 $O(n^2)$

插入排序并不适合对于数据量比较大的排序应用；不过若数量级小于千，插入排序还是一个不错的选择。


# 代码实现

中间变量保存插入数据，比较过程中不断右移（左移）已排序数据：

```java
public class InsertionSort {

    // 升序排列
    public static void sort(int[] a) {
        final int length = a.length;

        for (int i = 1; i < length; i++) {
            int toBeInserted = a[i];  // get the one to be inserted into the ordered list

            for (int j = i; j >= 0; j--) {
                // 找出不大于将要插入的元素的那个位置
                if (j > 0 && a[j - 1] > toBeInserted) {
                    a[j] = a[j - 1];  // 在 for 循环内，将比它大的元素往后移动
                } else {
                    a[j] = toBeInserted;  // set that one into the ordered list
                    break;
                }
            }
        }
    }
}
```

比较过程中交换数据：

```java
public class InsertionSort {

    public static void sortThenSwap(int[] a) {
        final int length = a.length;

        for (int i = 1; i < length; i++) {
            for (int j = i; j > 0; j--) {
                if (a[j] < a[j - 1]) {
                    final int temp = a[j];
                    a[j] = a[j - 1];
                    a[j - 1] = temp;
                }
            }
        }
    }
}
```
