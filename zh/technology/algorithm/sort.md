---
title: 常用排序算法总结
description: 本文大部分内容为《大话数据结构》第九章的笔记，每个算法都附有笔者实现的 JavaScript 版本，暂供参考，后续会把笔记替换成总结输出。
published: true
date: 2021-01-14T07:38:20.634Z
tags: algorithm, sort
editor: markdown
dateCreated: 2020-12-05T15:39:16.357Z
---

# 9.2 排序的基本概念与分类

假设含有 `n` 个记录的序列为 `{ r1, r2, ......, rn }`，其相应的关键字分别为 `{ k1, k2, ......, kn }`，需确定 1, 2, ......, n 的一种排列 p1, p2, ......, pn，使其相应的关键字满足 `kp1 ≤ kp2 ≤ ...... ≤ kpn`（非递减或非递增）关系，即使得序列成为一个按关键字有序的序列 `{ rp1, rp2, ......, rpn }`，这样的操作就称为排序。

## 排序的稳定性
假设 `ki = kj ( 1 ≤ i ≤ n, 1 ≤ j ≤ n, i ≠ j ）`，且在排序前的序列中 ri 领先于 rj（即 i < j）。
如果排序后 ri 仍领先于 rj，则称所用的排序方法是稳定的；反之，若可能使得排序后的序列中rj领先ri，则称所用的排序方法是不稳定的。

只要有一组关键字实例发生类似情况，就可认为此排序方法是不稳定的。排序算法是否稳定的，要通过分析后才能得出。

## 9.2.2　内排序与外排序

根据在排序过程中<u>待排序的记录是否全部被放置在内存中</u>，排序分为：__内排序和外排序__。

1．时间性能
  在内排序中，主要进行两种操作：**比较**和**移动**。
  比较指关键字之间的比较，这是要做排序最起码的操作。
  移动指记录从一个位置移动到另一个位置，事实上，移动可以通过改变记录的存储方式来予以避免。
  总之，高效率的内排序算法应该是具有<u>尽可能少的关键字比较次数和尽可能少的记录移动次数</u>。

2．辅助空间
  辅助存储空间是除了存放待排序所占用的存储空间之外，执行算法所需要的其他存储空间。

3．算法的复杂性
  注意这里指的是*算法本身的复杂度*，而不是指算法的时间复杂度。
  显然算法过于复杂也会影响排序的性能。

根据排序过程中借助的主要操作，我们把内排序分为：**插入排序**、**交换排序**、**选择排序**和**归并排序**。

## 9.2.3　排序用到的结构与函数

为了讲清楚排序算法的代码，我先提供一个用于排序用的顺序表结构，此结构也将用于之后我们要讲的所有排序算法。
```c
#define MAXSIZE 10       /* 用于要排序数组个数最大值，可根据需要修改 */
typedef struct {      
    int r[MAXSIZE + 1];  /* 用于存储要排序数组，r[0] 用作哨兵或临时变量 */      
    int length;          /* 用于记录顺序表的长度 */  
} SqList;
```

另外，由于排序最最常用到的操作是数组两元素的交换，我们将它写成函数，在之后的讲解中会大量的用到。

```c
/* 交换L中数组r的下标为i和j的值 */
void swap ( SqList *L, int i, int j ) {
    int temp = L->r[i];
    L->r[i] = L->r[j];
    L->r[j] = temp;
}
```


# 9.3　冒泡排序

## 9.3.1　最简单排序实现
冒泡排序（Bubble Sort）一种<u>交换排序</u>，它的基本思想是：两两比较相邻记录的关键字，如果反序则交换，直到没有反序的记录为止。

```c
/* 对顺序表L作交换排序(冒泡排序初级版) */
void BubbleSort0 ( SqList *L ) {
    int i, j;
    for (i = 1; i < L->length; i++) {
        for (j = i + 1; j <= L->length; j++) {
            if (L->r[i] > L->r[j]) {
                swap(L, i, j);  /* 交换 L->r[i] 与 L->r[j] 的值 */
            }
        }
    }
}
```
每次 i 锁定一位，j 就循环之后其余的与之对比，一次只能排序好一位。

## 9.3.2　冒泡排序算法
```c
/* 对顺序表L作冒泡排序 */
void BubbleSort ( SqList *L ) {
    int i, j;
    for (i = 1; i < L->length; i++) {       
        for (j = L->length - 1; j >= i;j--) {   /* 注意j是从后往前循环 */
            if (L->r[j] > L->r[j + 1]) {        /* 若前者大于后者(注意这里与上一算法差异) */
                swap(L, j, j + 1);              /* 交换L->r[j]与L->r[j+1]的值 */ 
            }
        }
    }
}
```

![bubblesort.jpg](/technology/algorithm/sort/bubblesort.jpg)
较小的数字如同气泡般慢慢浮到上面，因此就将此算法命名为冒泡算法

## 9.3.3　冒泡排序优化

增加一个标记变量 flag
```c
/* 对顺序表L作改进冒泡算法 */
void BubbleSort2 (SqList *L) {
    int i, j;
    Status flag = TRUE;             /* flag 用来作为标记 */
    /* 若 flag 为 true 说明有过数据交换，否则停止循环 */
    for (i = 1; i < L->length && flag; i++) {
        flag = FALSE;               /* 初始为 false */
        for (j = L->length - 1; j >= i; j--) {
            if (L->r[j] > L->r[j + 1]) {
                swap(L, j, j + 1);  /* 交换 L->r[j] 与 L->r[j+1] 的值 */                 
                flag = TRUE;        /* 如果有数据交换，则 flag 为 true */
            }
        }
    }
}
```

```js
/* 我的冒泡排序 JS 实现 */
const bubbleSort = (arr) => {
    if (arr == null || arr.length < 2) {
        return;
    }
    let i, j;
    let flag = true;
    for (i = 0; i < arr.length - 1 && flag; i++) {
        flag = false;
        for (j = arr.length - 2; j >= i; j--) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j + 1);
                flag = true;
            }
        }
    }
}
```

## 9.3.4　冒泡排序复杂度分析
当最好的情况，也就是要排序的表本身就是有序的，那么我们比较次数，根据最后改进的代码，可以推断出就是 n-1 次的比较，没有数据交换，时间复杂度为 O(n)。
当最坏的情况，即待排序表是逆序的情况，此时需要比较 sigma( i = 2, n, i - 1 ) = 1 + 2 + 3 + ... + ( n - 1 ) = n ( n - 1 ) / 2 次，并作等数量级的记录移动。
因此，总的时间复杂度为O( n^2 )。


# 9.4　简单选择排序

选择排序的基本思想是每一趟在 n - i ＋ 1 ( i = 1, 2, ..., n - 1 )个记录中选取关键字最小的记录作为有序序列的第i个记录。

## 9.4.1　简单选择排序算法
简单选择排序法（Simple Selection Sort）就是通过 n - i 次关键字间的比较，从 n - i ＋ 1 个记录中选出关键字最小的记录，并和第 i（1 ≤ i ≤ n）个记录交换之。

```c
/* 对顺序表L作简单选择排序 */
void SelectSort (SqList *L) {
    int i, j, min;
    for (i = 1; i < L->length; i++) {
        min = i;    /* 将当前下标定义为最小值下标 */
        for (j = i + 1; j <= L->length; j++) {  /* 循环之后的数据 */
            if (L->r[min] > L->r[j]) {  /* 如果有小于当前最小值的关键字 */
                min = j;    /* 将此关键字的下标赋值给min */
            }
        }
        if (i != min) { /* 若min不等于i，说明找到最小值，交换 */
            swap(L, i, min);    /* 交换L->r[i]与L->r[min]的值 */
        }
    }
}
```

```js
/* 我的选择排序 JS 实现 */
const selectSort =  (arr) => {
    if (arr == null || arr.length < 2) {
        return;
    }
    let i, j, min;
    for (i = 0; i < arr.length - 1; i++) {
        min = i;
        for (j = i + 1; j < arr.length; j++) {
            if (arr[min] > arr[j]) {
                min = j;
            }
        }
        if (min !== i) {
            swap(arr, i, min);
        }
    }
}
```
## 9.4.2　简单选择排序复杂度分析
从简单选择排序的过程来看，它最大的特点就是交换移动数据次数相当少，这样也就节约了相应的时间。
分析它的时间复杂度发现，无论最好最差的情况，其比较次数都是一样的多，第i趟排序需要进行 n-i 次关键字的比较，此时需要比较 sigma(i=1, n-1, n-i) = ( n - 1 ) + ( n - 2 ) + ... + 1 = n ( n - 1 ) / 2 次。而对于交换次数而言，当最好的时候，交换为 0 次，最差的时候，也就初始降序时，交换次数为 n - 1 次，基于最终的排序时间是比较与交换的次数总和，因此，总的时间复杂度依然为O( n^2 )。
应该说，尽管与冒泡排序同为O( n^2 )，但简单选择排序的性能上还是要略优于冒泡排序。


# 9.5　直接插入排序

## 9.5.1　直接插入排序算法
直接插入排序（Straight Insertion Sort）的基本操作是将一个记录插入到已经排好序的有序表中，从而得到一个新的、记录数增 1 的有序表。

```c
/* 对顺序表L作直接插入排序 */
void InsertSort (SqList *L) {
    int i, j;
    for (i = 2; i <= L->length; i++) {
        if (L->r[i] < L->r[i - 1]) {        /* 需将L->r[i]插入有序子表 */
            L->r[0] = L->r[i];              /* s设置哨兵 */
            for (j = i - 1; L->r[j] > L->r[0]; j--) {
                L->r[j + 1] = L->r[j];      /* 记录后移 */
            }
            L->r[j + 1] = L->r[0];          /* 插入到正确位置 */
        }
    }
}
```

```js
/* 我的插入排序 JS 实现 */
const insertSort = (arr) => {
    if (arr == null || arr.length < 2) {
        return;
    }
    let i, j, temp;
    for (i = 1; i < arr.length; i++) {
        if (arr[i] < arr[i-1]) {
            temp = arr[i];
            for (j = i-1; arr[j] > temp; j--) {
                arr[j+1] = arr[j];
            }
            arr[j+1] = temp;
        }
    }
}

```

## 9.5.2　直接插入排序复杂度分析

从空间上来看，它只需要一个记录的辅助空间，因此关键是看它的时间复杂度。
![insertsorto(n).jpg](/technology/algorithm/sort/insertsorto(n).jpg)


# 9.6　希尔排序

将相距某个增量的记录组成一个子序列。
根据插入排序改进。

分割待排序记录的目的是减少待排序记录的个数，并使整个序列向基本有序发展。
需要采取跳跃分割的策略：<u>将相距某个“增量”的记录组成一个子序列</u>，这样才能保证在子序列内分别进行直接插入排序后得到的结果是基本有序而不是局部有序。

## 9.6.2　希尔排序算法
```c
/* 对顺序表L作希尔排序 */
void ShellSort(SqList *L) {
    int i, j;
    int increment = L->length;     
    do {
        increment = increment / 3 + 1;        /* 增量序列 */
        for (i = increment + 1; i <= L->length; i++) {
            if (L->r[i] < L->r[i - increment]) {
                /* 需将L->r[i]插入有序增量子表暂存在L->r[0] */
                L->r[0] = L->r[i];
                for (j = i - increment; j > 0 && 
                    L->r[0] < L->r[j]; j -= increment) {
                    /* 记录后移，查找插入位置 */
                    L->r[j + increment] = L->r[j];
                }
                L->r[j + increment] = L->r[0];  /* 插入 */
            }
        }
    } while (increment > 1);
}
```

```js
// 我的 shell 排序实现
const shellSort = (arr) => {
    let i, j, temp;
    let gap = arr.length;
    do {
        gap = Math.floor(gap / 3);
        for (i = gap; i < arr.length; i++) {
            if (arr[i] < arr[i - gap]) {
                temp = arr[i];
                for (j = i - gap; j > -1 && arr[j] > temp; j -= gap) {
                    arr[j + gap] = arr[j];
                }
                arr[j + gap] = temp;
            }
        }
    } while (gap > 0);
}

```

## 9.6.3　希尔排序复杂度分析

希尔排序的关键并不是随便分组后各自排序，而是将相隔某个“增量”的记录组成一个子序列，实现跳跃式的移动，使得排序的效率提高。

这里“增量”的选取就非常关键了。
我们在代码中，是用increment=increment/3+1;的方式选取增量的，可究竟应该选取什么样的增量才是最好，目前还是一个数学难题，迄今为止还没有人找到一种最好的增量序列。
不过大量的研究表明，当增量序列为 **dlta[k]=2^(t-k+1)-1（0 ≤ k ≤ t ≤ |log2(n+1)|）** 时，可以获得不错的效率，其时间复杂度为 **O(n^(3/2))** ，要好于直接排序的O(n^2)。
需要注意的是，增量序列的最后一个增量值必须等于1才行。
另外由于记录是跳跃式的移动，希尔排序并不是一种稳定的排序算法。

- 最佳情况：T(n) = O(nlog2 n)
- 最坏情况：T(n) = O(nlog2 n)
- 平均情况：T(n) =O(nlog n)


# 9.7　堆排序

根据选择排序改进。

堆是具有下列性质的**完全二叉树**：
- 每个结点的值都大于或等于其左右孩子结点的值，称为大顶堆；
- 每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆。

## 9.7.1 堆排序算法
1.如何由一个无序序列构建成一个堆？
2.如果在输出堆顶元素后，调整剩余元素成为一个新的堆？

```c
/* 对顺序表L进行堆排序 */
void HeapSort (SqList *L) {
    int i;
    for (i = L->length / 2; i > 0; i--)  /* 把L中的r构建成一个大顶堆 */        
        HeapAdjust(L, i, L->length);
    for (i = L->length; i > 1; i--) {
        swap(L, 1, i);  /* 将堆顶记录和当前未经排序子序列的最后一个记录交换 */
        HeapAdjust(L, 1, i - 1);   /* 将L->r[1..i-1]重新调整为大顶堆 */
    }
}
```
整个排序过程分为两个for循环：
- 第一个循环要完成的就是将现在的待排序序列构建成一个大顶堆。
- 第二个循环要完成的就是逐步将每个最大值的根结点与末尾元素交换，并且再调整其成为大顶堆。

![heapsort.jpg](/technology/algorithm/sort/heapsort.jpg)

第一个循环为何是 arr.length / 2 呢？如图，从 4 循环到 1，它们都是有孩子的结点，其实就是每次将有孩子的结点和其孩子调整好。

我们所谓的将待排序的序列构建成为一个大顶堆，其实就是从下往上、从右到左，将每个非终端结点（非叶结点）当作根结点，将其和其子树调整成大顶堆。

```c
/* 已知L->r[s..m]中记录的关键字除L->r[s]之外均满足堆的定义 */
/* 本函数调整L->r[s]的关键字，使L->r[s..m]成为一个大顶堆 */
void HeapAdjust (SqList *L,int s,int m) {
    int temp, j;
    temp = L->r[s];
    for (j = 2 * s; j <= m; j *= 2) {   /* 沿关键字较大的孩子结点向下筛选 */
        if (j < m && L->r[j] < L->r[j + 1])
            ++j;                    /* j为关键字中较大的记录的下标 */
        if (temp >= L->r[j])
            break;                  /* rc应插入在位置s上 */
        L->r[s] = L->r[j];
        s = j;
    }
    L->r[s] = temp;                 /* 插入 */
}
```

```js
// 我的堆排序 JS 实现
const heapSort = (arr) => {
    let i;
    for (i = Math.floor((arr.length - 1) / 2); i > -1; i--) {
        heapAdjust(arr, i, arr.length - 1);
    }
    for (i = arr.length - 1; i > 0; i--) {
        swap(arr, i, 0);
        heapAdjust(arr, 0, i - 1);
    }
}

const heapAdjust = (arr, s, m) => {
    let j;
    let temp = arr[s];
    for (j = 2 * s; j <=m; j *= 2) {
        if (j < m && arr[j] < arr[j + 1]) {
            j++;
        }
        if (temp >= arr[j]) {
            break;
        }
        arr[s] = arr[j];
        s = j;
    }
    arr[s] = temp;
}
```

## 9.7.2 堆排序复杂度分析

运行时间主要消耗：
- 初始构建堆
- 在重建堆时的反复筛选

在构建堆的过程中，因为是完全二叉树从**最下层最右边的非终端结点**开始构建，将它与其孩子进行比较和若有必要的互换，对于每个非终端结点来说，其实最多进行两次比较和互换操作，因此整个构建堆的时间复杂度为O(n)。
在正式排序时，第i次取堆顶记录重建堆需要用O(logi)的时间（完全二叉树的某个结点到根结点的距离为|log(2)n| + 1），并且需要取 n-1 次堆顶记录，因此，重建堆的时间复杂度为O(nlogn)。

所以总体来说，堆排序的时间复杂度为**O(nlogn)**。

由于堆排序对原始记录的排序状态并不敏感，因此它无论是最好、最坏和平均时间复杂度均为O(nlogn)。这在性能上显然要远远好过于冒泡、简单选择、直接插入的O(n2)的时间复杂度了。 空间复杂度上，它只有一个用来交换的暂存单元，也非常的不错。不过由于记录的比较与交换是跳跃式进行，因此堆排序也是一种不稳定的排序方法。
另外，由于初始构建堆所需的比较次数较多，因此，它并不适合待排序序列个数较少的情况。


# 9.8　归并排序

## 9.8.1 归并排序算法

归并排序（Merging Sort）就是利用归并的思想实现的排序方法。它的原理是假设初始序列含有n个记录，则可以看成是n个有序的子序列，每个子序列的长度为1，然后两两归并，得到|n/2|（|x|表示不小于x的最小整数）个长度为2或1的有序子序列；再两两归并，……，如此重复，直至得到一个长度为n的有序序列为止，这种排序方法称为2路归并排序。
```c
/* 对顺序表L作归并排序 */
void MergeSort(SqList *L) {
    MSort(L->r, L->r, 1, L->length);
}
```
```c
/* 将SR[s..t]归并排序为TR1[s..t] */
void MSort(int SR[], int TR1[], int s, int t) {
    int m;
    int TR2[MAXSIZE + 1];
    if (s == t)
        TR1[s] = SR[s];
    else {
        m = (s + t) / 2;            /* 将SR[s..t]平分为SR[s..m]和SR[m+1..t] */
        MSort(SR, TR2, s, m);       /* 递归将SR[s..m]归并为有序的TR2[s..m] */
        MSort(SR, TR2, m + 1, t);   /* 递归将SR[m+1..t]归并为有序TR2[m+1..t] */
        Merge(TR2,TR1, s, m, t);    /* 将TR2[s..m]和TR2[m+1..t]归并到TR1[s..t] */
    }
}
```
```c
/* 将有序的SR[i..m]和SR[m+1..n]归并为有序的TR[i..n] */
void Merge(int SR[], int TR[], int i, int m, int n) {
    int j, k, l;
    for (j = m + 1, k = i; i <= m && j <= n; k++) {    /* 将SR中记录由小到大归并入TR */
        if (SR[i] < SR[j])
            TR[k] = SR[i++];
        else
            TR[k] = SR[j++];
    }
    if (i <= m) {
        for (l = 0; l <= m - i; l++)
            TR[k + l]=SR[i + l];      /* 将剩余的SR[i..m]复制到TR */
    }
    if (j<=n) {
        for (l = 0; l <= n - j; l++)
            TR[k + l] = SR[j + l];    /* 将剩余的SR[j..n]复制到TR */
    }
}
```
```js
// 我的 JS 归并排序实现
const mergeSort = (arr) => {
    MSort(arr, arr, 0, arr.length - 1);
}

const MSort = (SR, TR, s, t) => {
    let m;
    const TR1 = [];
    if (s === t) {
        TR[s] = SR[s];
    } else {
        m = Math.floor((s + t) / 2);
        MSort(SR, TR1, s, m);
        MSort(SR, TR1, m + 1, t);
        Merge(TR1, TR, s, m, t);
    }
}

const Merge = (SR, TR, i, m, n) => {
    let j, k, l;
    // 要非常注意这个 k 的起始点！！！
    for (j = m + 1, k = i; i <= m && j <= n; k++) {
        if (SR[i] < SR[j]) {
            TR[k] = SR[i++];
        } else {
            TR[k] = SR[j++];
        }
    }
    if (i <= m) {
        for (l = 0; l <= m - i; l++) {
            TR[k + l] = SR[i + l];
        }
    }
    if (j <= n) {
        for (l = 0; l <= n - 1; l++) {
            TR[k + l] = SR[j + l];
        }
    }
}
```

## 9.8.2 归并排序复杂度分析

时间复杂度：
一趟归并需要将SR[1]～SR[n]中相邻的长度为h的有序序列进行两两归并。并将结果放到TR1[1]～TR1[n]中，这需要将待排序序列中的所有记录扫描一遍，因此耗费O(n)时间；
而由完全二叉树的深度可知，整个归并排序需要进行 |log(2)n| 次(（|x|表示不小于x的最小整数）)；
因此，总的时间复杂度为**O(nlogn)** ，而且这是归并排序算法中最好、最坏、平均的时间性能。

空间复杂度：
由于归并排序在归并过程中需要<u>与原始记录序列同样数量的存储空间存放归并结果</u>以及<u>递归时深度为log2n的栈空间</u>，因此空间复杂度为**O(nlogn)**。

另外，对代码进行仔细研究，发现Merge函数中有if(SR[i]<SR[j])语句，这就说明它需要两两比较，不存在跳跃，因此归并排序是一种稳定的排序算法。
也就是说，归并排序是一种比较占用内存，但却效率高且稳定的算法。

## 9.8.3 非递归实现归并排序

将递归转化成迭代。
```c
/* 对顺序表L作归并【非递归】排序 */
void MergeSort2(SqList *L) {
    int * TR = (int *)malloc(L->length * sizeof(int));  /* 申请额外空间 */
    int k = 1;
    while (k < L->length) {
        MergePass(L->r, TR, k, L->length);
        k = 2 * k;  /* 子序列长度加倍 */
        MergePass(TR, L->r, k, L->length);
        k = 2 * k;  /* 子序列长度加倍 */
    }
}
```
```c
/* 将SR[]中相邻长度为s的子序列两两归并到TR[] */
/* k 为间隔 */
void MergePass(int SR[], int TR[], int k, int n) {
    int i = 1;
    int j;
    while (i <= n - 2 * k + 1) {    // (n - 2k + 1)为最后能递归的下标
        Merge(SR, TR, i, i + k - 1, i + 2 * k - 1); /* 两两归并 */
        i = i + 2 * k;
    }
    if (i < n - k + 1)                      /* 归并最后两个序列 */
        Merge(SR, TR, i, i + k - 1, n);
    else                                    /* 若最后只剩下单个子序列 */
        for (j = i; j <= n; j++)
            TR[j] = SR[j];
}
```
```js
// 我的归并排序【非递归】JS 实现
const mergeSort2 = (arr) => {
    let k = 1; // k 为间隔
    const TR = [];
    while (k < arr.length) {
        mergePass(arr, TR, k);
        k *= 2;
        mergePass(TR, arr, k);
        k *= 2;
    }
}

const mergePass = (SR, TR, k) => {
    let i = 0;
    let j;
    const n = SR.length - 1;
    while (i <= n - 2 * k + 1) {
        Merge(SR, TR, i, i + k - 1, i + 2 * k - 1);
    }
    if (i < n - k + 1) {
        Merge(SR, TR, i, i + k - 1, n);
    } else {
        for (j = i; j <= n; j++) {
            TR[j] = SR[j];
        }
    }
}
```


# 9.9　快速排序
快速排序其实就是我们前面认为最慢的冒泡排序的升级，它们都属于交换排序类。即它也是通过不断比较和移动交换来实现排序的，只不过它的实现，增大了记录的比较和移动的距离，将关键字较大的记录从前面直接移动到后面，关键字较小的记录从后面直接移动到前面，从而减少了总的比较次数和移动交换次数。

## 9.9.1 快速排序算法
快速排序（Quick Sort）的基本思想是：
通过一趟排序将待排记录分割成独立的两部分，其中一部分记录的关键字均比另一部分记录的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序的目的。
```c
/* 对顺序表L作快速排序 */
void QuickSort(SqList *L) {
    QSort(L, 1, L->length);    // 需递归调用，所以封装了一个函数
}

/* 对顺序表L中的子序列L->r[low..high]作快速排序 */
/* 参数：数组，最小下标值、最大下标值 */
void QSort(SqList *L, int low, int high) {
    int pivot;
    if (low < high) {
        pivot = Partition(L, low, high);   /* 将L->r[low..high]一分为二，算出枢轴值pivot */
        QSort(L, low, pivot - 1);     /* 对低子表递归排序 */
        QSort(L, pivot + 1, high);    /* 对高子表递归排序 */
    }
}
```
这一段代码的核心是“pivot=Parti-tion(L,low,high);”
Partition函数要做的，就是先选取当中的一个关键字，然后想尽办法将它放到一个位置，使得它左边的值都比它小，右边的值比它大，我们将这样的关键字称为枢轴（pivot）。
```c
/* 交换顺序表L中子表的记录，使枢轴记录到位，并返回其所在位置 */
/* 此时在它之前（后）的记录均不大（小）于它。 */
int Partition(SqList *L, int low, int high) {
    int pivotkey;
    pivotkey = L->r[low];       /* 用子表的第一个记录作枢轴记录 */
    while (low < high) {        /* 从表的两端交替向中间扫描 */
        while (low < high && L->r[high] >= pivotkey)             
            high--;
        swap(L, low, high);     /* 将比枢轴记录小的记录交换到低端 */
        while (low < high && L->r[low] <= pivotkey)             
            low++;
        swap(L, low, high);     /* 将比枢轴记录大的记录交换到高端 */
    }
    return low;                 /* 返回枢轴所在位置 */
}
```
通过这段代码的模拟，大家应该能够明白，Partition函数，其实就是将选取的pivotkey不断交换，将比它小的换到它的左边，比它大的换到它的右边，它<u>也在交换中不断更改自己的位置</u>，直到完全满足这个要求为止。

## 9.9.2 快速排序复杂度分析
### 在最优情况下：
Partition每次都划分得很均匀，如果排序n个关键字，其递归树的深度就为 |log(2)n| + 1（表示不大于x的最大整数），即仅需递归 log2n 次，需要时间为T(n)的话，第一次Partiation应该是需要对整个数组扫描一遍，做n次比较。然后，获得的枢轴将数组一分为二，那么<u>各自</u>还需要T(n/2)的时间（注意是最好情况，所以平分两半）。于是不断地划分下去……

T(n) ≤ 2T(n / 2) + n, T(1) = 0 T(n) ≤ 2(2T(n / 4) + n / 2) + n = 4T(n / 4)+2n
T(n) ≤ 4(2T(n / 8) + n / 4) + 2n = 8T(n / 8)+3n
……
T(n) ≤ nT(1) + (log2n) × n = O(nlogn)

也就是说，在最优的情况下，快速排序算法的时间复杂度为**O(nlogn)** 。

### 在最坏的情况下：
待排序的序列为正序或者逆序，每次划分只得到一个比上一次划分少一个记录的子序列，注意另一个为空。如果递归树画出来，它就是一棵斜树。
此时需要执行n-1次递归调用，且第i次划分需要经过n-i次关键字的比较才能找到第i个记录，也就是枢轴的位置，因此比较次数为sigma(i=1, n-1, n-i)=(n-1)+(n-2)+...+1=n(n-1)/2，最终其时间复杂度为**O(n2)** 。

### 平均的情况：
设枢轴的关键字应该在第k的位置（1≤k≤n），那么：

![quicksorto(n).jpg](/technology/algorithm/sort/quicksorto(n).jpg)

由数学归纳法可证明，其数量级为**O(nlogn)** 。

由于比较和交换是跳跃进行的，因此，快速排序是不稳定排序。

## 9.9.3 快速排序优化
### 1. 优化选取枢轴
我们来看看取左端、右端和中间三个数的实现代码，在Partition函数代码的第3行与第4行之间增加这样一段代码。
```c
int pivotkey;

int m = low + (high - low) / 2;    /* 计算数组中间的元素的下标 */
if (L->r[low] > L->r[high])
    swap(L, low, high);            /* 交换左端与右端数据，保证左端较小 */
if (L->r[m] > L->r[high])
    swap(L, high, m);              /* 交换中间与右端数据，保证中间较小 */
if (L->r[m] > L->r[low])
    swap(L, m, low);               /* 交换中间与左端数据，保证左端为中值 */
/* 此时L.r[low]已经为整个序列左中右三个关键字的中间值。 */

pivotkey = L->r[low];              /*用子表的第一个记录作枢轴记录 */
```

### 2．优化不必要的交换
观察数组{50,10,90,30,70,40,80,60,20}用上面的快排，发现50这个关键字，其位置变化是1→9→3→6→5，可其实它的最终目标就是5，当中的交换其实是不需要的。因此对Partition函数优化。
```c
/* 快速排序优化算法 */
int Partition1(SqList *L, int low, int high) {
    int pivotkey;
    /* 这里省略三数取中代码 */
    pivotkey = L->r[low];         /* 用子表的第一个记录作枢轴记录 */
!   L->r[0] = pivotkey;           /* 将枢轴关键字备份到L->r[0] */
    while (low < high) {          /* 从表的两端交替向中间扫描 */
        while (low < high && L->r[high] >= pivotkey)
            high--;
!       L->r[low] = L->r[high];   /* 采用替换而不是交换的方式进行操作 */
        while (low < high && L->r[low] <= pivotkey)
            low++;
!       L->r[high] = L->r[low];   /* 采用替换而不是交换的方式进行操作 */
    }
!   L->r[low] = L->r[0];          /* 将枢轴数值替换回L.r[low] */
    return low;                   /* 返回枢轴所在位置 */
}
```
注意代码中前面有！的行数的改变。我们事实将piv-otkey备份到L.r[0]中，然后在之前是swap时，只作替换的工作，最终当low与high会合，即找到了枢轴的位置时，再将L.r[0]的数值赋值回L.r[low]。因为这当中少了多次交换数据的操作，在性能上又得到了部分的提高。

### 3．优化小数组时的排序方案

如果数组非常小，其实快速排序反而不如直接插入排序来得更好（直接插入是简单排序中性能最好的）。其原因在于快速排序用到了<u>递归操作</u>，在大量数据排序时，这点性能影响相对于它的整体算法优势而言是可以忽略的，但如果数组只有几个记录需要排序时，这就成了一个大炮打蚊子的大问题。因此我们需要改进一下QSort函数。

```c
#define MAX_LENGTH_INSERT_SORT 7                    /* 数组长度阀值 */
/* 对顺序表L中的子序列L.r[low..high]作快速排序 */
void QSort(SqList &L, int low, int high) {
    int pivot;
!   if ((high - low) > MAX_LENGTH_INSERT_SORT) {    /* 当high-low大于常数时用快速排序 */
        pivot = Partition(L, low, high);            /* 将L.r[low..high]一分为二，并算出枢轴值pivot */
        QSort(L, low, pivot - 1);                   /* 对低子表递归排序 */
        QSort(L, pivot + 1, high);                  /* 对高子表递归排序 */
    } else {
!       InsertSort(L);                              /* 当high-low小于等于常数时用直接插入排序 */
    }
}
```
注意代码中前面有！的行数，增加了一个判断，当high-low不大于某个常数时（有资料认为7比较合适，也有认为50更合理，实际应用可适当调整），就用直接插入排序，这样就能保证最大化地利用两种排序的优势来完成排序工作。

### 4．优化递归操作

若待排序的序列划分极端不平衡，递归的深度将趋近于 n，而不是 log2n，既慢，又占用栈的空间，所以想办法减少递归。

于是我们对QSort实施尾递归优化。
```c
/* 对顺序表L中的子序列L.r[low..high]作快速排序 */
void QSort1(SqList *L, int low, int high) {
    int pivot;
    if ((high - low) > MAX_LENGTH_INSERT_SORT) {
!       while (low < high) {
            pivot = Partition1(L, low, high);   /* L.r[low..high]一分为二，算出枢轴值pivot */
            QSort1(L, low, pivot - 1);          /* 对低子表递归排序 */
!           low = pivot + 1;                    /* 尾递归 */
        }
    } else {
        InsertSort(L);
    }
}
```
当我们将if改成while后（见！代码部分），因为第一次递归以后，变量low就没有用处了，所以可以将pivot+1赋值给low，再循环后，来一次Partition(L,low,high)，其效果等同于“QSort(L,pivot+1,high);”。结果相同，但因采用迭代而不是递归的方法可以缩减堆栈深度，从而提高了整体性能。

```js
// 我的 JS 快速排序实现（包含优化点）
const quickSort = (arr) => {
    QSort(arr, 0, arr.length - 1);
};

// 优化点4：若待排序数量较小，用直接插入排序
const MAX_LENGTH_INSERT_SORT = 7;
const QSort = (arr, low, high) => {
    if (arr.length > MAX_LENGTH_INSERT_SORT) {
        let pivot;
        // 优化点1
        // if (low < high) {
        while (low < high) {
            pivot = partition(arr, low, high);
            if (pivot > low) {
                QSort(arr, low, pivot - 1);
            }
            // 优化点1：采用迭代而不是递归的方法可以缩减堆栈深度
            // if (pivot < high) {
            //     QSort(arr, pivot + 1, high);
            // }
            low = pivot + 1;
        }
    } else {
        // insertSort(arr);
    }
};

const partition = (arr, low, high) => {
    // 优化点2：枢轴的选取，尽量取靠近中间的值，让 arr[low] 为中间值
    // let pivotKey = arr[low];
    let pivotKey;
    let m = Math.floor(low + (high - low) / 2);
    if (arr[low] > arr[high]) {
        swap(arr, low, high);
    }
    if (arr[m] > arr[high]) {
        swap(arr, m, high);
    }
    if (arr[m] > arr[low]) {
        swap(arr, m, low);
    }
    pivotKey = arr[low];

    // 优化点3：优化不必要的交换，用替换，而不是两两交换
    let temp = pivotKey;
    while (low < high) {
        while (low < high && arr[high] >= pivotKey) {
            high--;
        }
        // 优化点3
        // swap(arr, low, high);
        arr[low] = arr[high];

        while (low < high && arr[low] <= pivotKey) {
            low++;
        }
        // 优化点3
        // swap(arr, low, high);
        arr[high] = arr[low];
    }
    // 优化点3
    arr[low] = temp;
    return low;
};
```


# 9.10　总结回顾

![sortsclassify.jpg](/technology/algorithm/sort/sortsclassify.jpg)
![sortssummary.jpg](/technology/algorithm/sort/sortssummary.jpg)
