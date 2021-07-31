---
title: "Sorting Algorithms"
categories:
  - blog
tags:
  - learn
---

Since I'm learning algorithms these days (cranmming Leetcode actually), I find it quite necessary and interesting to do some notes about sorting algorithms, a series of common knowledge for every programmer.

I'll be realizing these algorithms myself using Python, in order to put the process in practice for better familiarity.

And here is the GitHub [repo](https://github.com/ffyuanda/ffyuanda_learn/tree/main/Leetcode/Sort) for these algorithms.

# Quicksort
```python
""" self-realized quicksort python version """
# https://www.youtube.com/watch?v=COk73cpQbFQ


def partition(l: list, start: int, end: int, reverse: bool = False):
    # last element as pivot
    pivot = l[end]
    pIndex = start
    for i in range(start, end):
        if reverse:

            if l[i] >= pivot:
                # swap larger ones to left of the pIndex
                l[pIndex], l[i] = l[i], l[pIndex]
                pIndex += 1
        else:

            if l[i] <= pivot:
                # swap smaller ones to left of the pIndex
                l[pIndex], l[i] = l[i], l[pIndex]
                pIndex += 1
    # swap the pivot
    l[pIndex], l[end] = l[end], l[pIndex]
    return pIndex


def quicksort(l: list, start: int, end: int):

    if start >= end:
        return
    pIndex = partition(l, start, end)
    quicksort(l, start, pIndex-1)
    quicksort(l, pIndex+1, end)


c = [2, 1, 3, 4, 8, 10, 7, 6]
quicksort(c, 0, len(c)-1)
print(c)
```
Quicksort is a Divide and Conquer algorithm (just like Mergesort). It implements a technique called *partition* to sort everything. Partition basically uses a pivot (could be any index, I used the last index of every array), and put elements that are lesser to the left of it and larger to the right of it (during a ascending sort).  
<br>
Then, it uses the [start, pivot_index - 1] and [pivot_index + 1, end] as the array to be sort into the recursive children functions. The algorithm uses index to identify the array and thus it is a **in-place** (does not create extra data structure to run the algorithm) algorithm.  
<br>
It is also a **non-stable** algorithm (a sorting algorithm is said to be stable if it maintains the relative order of records in the case of equality of keys).  
<br>
It's average time complexity is <span>$O(nlogn)$</span>
<br>

![Big-O cheatsheet](/assets/images/Big_O_cheat_sheet.jpeg)

---
Stay tuned! More content are coming.