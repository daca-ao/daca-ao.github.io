---
title: 快速排序
date: 2023-02-26 18:41:14
mathjax: true
tags:
- 算法
- 排序算法
---

快速排序属于不稳定的比较排序算法，是[冒泡排序](/2022/11/08/bubble-sort)的改进版本。

<!-- more -->

快速排序又称作**分区交换排序**（partition-exchange sort）。

思路如下：

1. n 个元素被分成左（left）、中（middle）、右（right）三段
    * 其中中段仅包含一个元素，作为**支点**（**pivot**）
    * 支点不一定是最中间的元素，即：left 和 right 的元素数量可以不一样
2. 左段元素均小于或等于中段元素，右段元素均大于或等于中段元素
    * 相对于[归并排序](/2023/02/25/merge-sort)而言，left 和 right 的元素可独立排序，无需合并
3. 分别递归使用快排，对 left 和 right 排序
4. 所得序列为 left + middle + right

之所以叫**快排**，是因为在步骤 1 之后，支点以左的任一元素无需再与支点以右的元素再发生比较和交换，大大减少了比较和交换的次数；  
因此其速度通常明显比其他算法更快，其递归排序的循环可以在大部分的架构上很有效率地去完成。

在执行步骤 1 的时候，数组需要**先进行一次排序**，分别筛选比支点大和小的元素；而这一次的排序，也是需要**递归执行**的排序。

变种：中值快速排序（median-of-three quick sort）

* 取 {$a[1]$, $a[(1+r)/2]$, $a[r]$} 中大小居中的元素作为支点


# 代码实现

C++：

```c++
template<class T>
void QuickSort(T *a, int n) {
    // Sort a[0:n-1] using quick sort.
    // Requires a[n] must have largest key.
    quickSort(a, 0, n-1);  // 以数组第一个元素作为支点
}

template<class T>
void quickSort(T a[], int l, int r)
{// Sort a[l:r], a[r+1] has large value.

    if (l >= r) {
        return;
    }

    int i = l,  // left-to-right cursor
    j = r + 1;  // right-to-left cursor
    T pivot = a[l];

    // swap elements >= pivot on left side
    // with elements <= pivot on right side
    while (true) {
        while (a[++i] < pivot) {  // find >= element on left side
            if (i == r) {
                break;
            }
        }
        while (a[--j] > pivot) {  // find <= element on right side
            if (j == l) {
                break;
            }
        }

        if (i >= j) {
            break;  // swap pair not found
        }
        Swap(a[i], a[j]);
    }

    // place pivot
    a[l] = a[j];
    a[j] = pivot;

    quickSort(a, l, j-1);  // sort left segment
    quickSort(a, j+1, r);  // sort right segment
}
```

Java：

```java
public class QuickSort {

    public static void sort(int[] a) {
        quickSort(a, 0, a.length - 1);
    }

    private static void quickSort(int[] a, int begin, int end) {
        if (begin >= end) {
            return;
        }

        int left = begin;
        int right = end;
        int pivot = a[begin];  // 首元素为支点

        while (left != right) {

            // search element smaller than pivot from right to left
            while (a[right] >= pivot && left < right) {
                right--;
            }
            // then search element bigger than pivot from left to right
            while (a[left] <= pivot && left < right) {
                left++;
            }

            if (left < right) {
                Swapper.swap(a, left, right);
            }
        }

        // put pivot into middle
        a[begin] = a[left];
        a[left] = pivot;

        quickSort(a, begin, left - 1);  // sort left part
        quickSort(a, left + 1, end);  // sort right part
    }
}
```

改进版：随机选择支点，使每次递归快排尽可能均匀。

```java
public class QuickSort {

    ...

    private static void quickSort(int[] a, int begin, int end) {
        if (begin >= end) {
            return;
        }

        int left = begin;
        int right = end;
        // 随机确定支点的位置
        int pivotInt = RANDOM.nextInt(end - begin + 1) + begin;
        int pivot = a[pivotInt];
        //选定基准值后，将基准值的位置和最开头的位置交换，这样在右边留出连续空间，便于交换
        Swapper.swap(a, pivotInt, begin);

        while (left != right) {

            // search element smaller than pivot from right to left
            while (a[right] >= pivot && left < right) {
                right--;
            }
            // then search element bigger than pivot from left to right
            while (a[left] <= pivot && left < right) {
                left++;
            }

            if (left < right) {
                Swapper.swap(a, left, right);
            }
        }

        // put pivot into middle
        a[begin] = a[left];
        a[left] = pivot;

        quickSort(a, begin, left - 1);  // sort left part
        quickSort(a, left + 1, end);  // sort right part
    }
}
```


# 复杂度分析

* 最坏情况：left 为空—— $O(n^2)$
* 最好情况：left 和 right 数目大致相同—— $O(nlogn)$
