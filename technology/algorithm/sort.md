---
title: Common Sorting Algorithms, Summarized
description: Notes on chapter 9 of the Chinese textbook 《大话数据结构》, with the author's own JavaScript version of each algorithm; for reference until the notes are rewritten as a summary.
published: true
date: 2026-09-30T13:35:19.000Z
tags: algorithm, sort
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/algorithm/sort.md)

*The Chinese page is mostly notes on chapter 9 of the textbook 《大话数据结构》 (Data Structures, Plainly Told). This English page only sums up the key points in our own words and keeps the author's JavaScript implementations; for the book-based notes and the C listings, see the [Chinese page](/zh/technology/algorithm/sort.md).*

# 9.2 Basic concepts and classification of sorting

Sorting rearranges records so that their keys are in order (non-decreasing or non-increasing).

## Stability of sorting
In a stable sort, records with equal keys keep their original relative order.
An unstable sort may swap them.

One counterexample is enough to show that a sort is unstable.

## 9.2.2 Internal and external sorting

<u>If all the records fit in memory</u> it's __internal sorting__; otherwise it's external.

(1) Time performance
  The cost comes from **comparisons** and **moves**; a good internal sort keeps <u>both low</u>.

(2) Auxiliary space
  Extra memory beyond the data itself.

(3) Algorithm complexity
  *How complicated the algorithm is*, not how long it runs.

Internal sorts fall into **insertion**, **exchange**, **selection** and **merge** sorts.

## 9.2.3 Structures and functions used for sorting

The JavaScript below sorts plain arrays with a `swap(arr, i, j)` helper; the book's C versions work on a list struct (see the Chinese page).


# 9.3 Bubble sort

## 9.3.1 The simplest version
An <u>exchange sort</u>: keep swapping neighbors that are out of order until none are.

## 9.3.2 The bubble sort algorithm

![bubblesort.jpg](/technology/algorithm/sort/bubblesort.jpg)
Small values float up like bubbles, hence the name.

## 9.3.3 Optimizing bubble sort

A flag stops the loop as soon as a pass makes no swaps.

```js
/* My bubble sort in JS */
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

## 9.3.4 Complexity of bubble sort
Best case (already sorted): n-1 comparisons, O(n).
Worst case (reversed): n(n-1)/2 comparisons and about as many moves.
Overall: O(n^2).


# 9.4 Simple selection sort

Each pass picks the smallest remaining key and puts it in the next position.

## 9.4.1 The simple selection sort algorithm
Find the minimum of the unsorted part with n - i comparisons, then swap it into place.

```js
/* My selection sort in JS */
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
## 9.4.2 Complexity of simple selection sort
Always n(n-1)/2 comparisons, but at most n - 1 swaps.
Still O(n^2), yet usually a bit faster than bubble sort thanks to the fewer swaps.


# 9.5 Straight insertion sort

## 9.5.1 The straight insertion sort algorithm
Insert each record into the already sorted part in front of it.

```js
/* My insertion sort in JS */
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

## 9.5.2 Complexity of straight insertion sort

It needs extra space for one record only; the running time depends on how ordered the input already is.
![insertsorto(n).jpg](/technology/algorithm/sort/insertsorto(n).jpg)


# 9.6 Shell sort

Insertion sort on subsequences of elements an increment apart, with the increment shrinking down to 1.

The long jumps move elements close to their places early, so the final pass has little left to do.

## 9.6.2 The shell sort algorithm
```js
// My shell sort
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

## 9.6.3 Complexity of shell sort

Performance depends on the increment sequence, and the best sequence is still an open question.
Some sequences reach about **O(n^(3/2))**, better than O(n^2); the last increment must be 1.
The long jumps make shell sort unstable.

- Best case: T(n) = O(nlog2 n)
- Worst case: T(n) = O(nlog2 n)
- Average case: T(n) =O(nlog n)


# 9.7 Heap sort

An improvement on selection sort, built on a **complete binary tree**:
- max-heap: every node is at least as large as its children;
- min-heap: every node is at most as large as its children.

## 9.7.1 The heap sort algorithm
Two questions: how to build a heap, and how to restore it after taking the top off.

Two loops:
- build a max-heap from the array;
- repeatedly swap the root to the end and re-heapify the rest.

![heapsort.jpg](/technology/algorithm/sort/heapsort.jpg)

Heapifying starts from the last node that has children and works back to the root.

```js
// My heap sort in JS
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

## 9.7.2 Complexity of heap sort

The time goes into:
- building the heap
- re-heapifying after each removal

Building is O(n), and each of the n-1 re-heapify steps costs O(log n).

Total: **O(nlogn)** in every case.

It needs only one temporary slot, but it's unstable, and building the heap costs enough comparisons that it isn't worth it for tiny inputs.


# 9.8 Merge sort

## 9.8.1 The merge sort algorithm

Treat the input as runs of length 1 and merge them in pairs, again and again, until one sorted run is left (2-way merge sort).
```js
// My merge sort in JS
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
    // Watch out for where k starts!!!
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

## 9.8.2 Complexity of merge sort

Each pass is O(n) and there are about log2n passes: **O(nlogn)** in every case.

The notes put its extra space at **O(nlogn)** (merge buffers plus the recursion stack).

It only compares neighbors, so it's stable: memory-hungry, but fast and stable.

## 9.8.3 Merge sort without recursion

The same merging, done bottom-up with a loop instead of recursion.
```js
// My merge sort in JS [non-recursive]
const mergeSort2 = (arr) => {
    let k = 1; // k is the gap
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


# 9.9 Quick sort
An exchange sort like bubble sort, but it moves elements across long distances, which cuts down comparisons and swaps.

## 9.9.1 The quick sort algorithm
Partition the array around a pivot so smaller keys end up on one side and larger keys on the other, then sort each side the same way; the pivot reaches its final place during partitioning.

## 9.9.2 Complexity of quick sort
### In the best case:
Even splits give a recursion depth of about log2n, with O(n) work per level:

T(n) ≤ 2T(n / 2) + n, T(1) = 0 T(n) ≤ 2(2T(n / 4) + n / 2) + n = 4T(n / 4)+2n
T(n) ≤ 4(2T(n / 8) + n / 4) + 2n = 8T(n / 8)+3n
……
T(n) ≤ nT(1) + (log2n) × n = O(nlogn)

So the best case is **O(nlogn)**.

### In the worst case:
Sorted or reversed input makes every split lopsided, giving n(n-1)/2 comparisons: **O(n2)**.

### On average:
With the pivot equally likely to land at any position k:

![quicksorto(n).jpg](/technology/algorithm/sort/quicksorto(n).jpg)

Induction gives **O(nlogn)**.

Long-distance swaps make it unstable.

## 9.9.3 Optimizing quick sort
### 1. A better pivot
Use the median of the first, middle and last elements.

### 2. Skipping unnecessary swaps
Keep the pivot aside and shift values into place instead of swapping, then drop the pivot into its slot at the end.

### 3. A better plan for small arrays

Below a small size threshold (suggestions range from 7 to 50), switch to insertion sort, which beats quick sort on tiny inputs.

### 4. Cutting down recursion

Badly unbalanced splits can push the recursion depth toward n.

Tail-recursion optimization turns the second recursive call into a loop, which keeps the stack shallow.

```js
// My quick sort in JS (with the optimizations)
const quickSort = (arr) => {
    QSort(arr, 0, arr.length - 1);
};

// Optimization 4: for small inputs, use straight insertion sort
const MAX_LENGTH_INSERT_SORT = 7;
const QSort = (arr, low, high) => {
    if (arr.length > MAX_LENGTH_INSERT_SORT) {
        let pivot;
        // Optimization 1
        // if (low < high) {
        while (low < high) {
            pivot = partition(arr, low, high);
            if (pivot > low) {
                QSort(arr, low, pivot - 1);
            }
            // Optimization 1: iterating instead of recursing cuts the stack depth
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
    // Optimization 2: pick a pivot close to the middle value, making arr[low] the median
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

    // Optimization 3: skip unnecessary swaps, replace instead of swapping pairs
    let temp = pivotKey;
    while (low < high) {
        while (low < high && arr[high] >= pivotKey) {
            high--;
        }
        // Optimization 3
        // swap(arr, low, high);
        arr[low] = arr[high];

        while (low < high && arr[low] <= pivotKey) {
            low++;
        }
        // Optimization 3
        // swap(arr, low, high);
        arr[high] = arr[low];
    }
    // Optimization 3
    arr[low] = temp;
    return low;
};
```


# 9.10 Summary

![sortsclassify.jpg](/technology/algorithm/sort/sortsclassify.jpg)
![sortssummary.jpg](/technology/algorithm/sort/sortssummary.jpg)
