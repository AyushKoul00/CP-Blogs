---
title: "Search in Rotated Sorted Array"
categories: [Concepts, Binary Search]
tags: [binary search]     # TAG names should always be lowercase
author: Ayush Koul
layout: post
toc: true
math: true
---

[Problem Link](https://leetcode.com/problems/search-in-rotated-sorted-array/)

### Brute-Force
The problem tells us to find a `target` value in the array `nums` which is a sorted array that has been rotated
around an unknown `pivot`. The brute force approach would be to go through each element in the array and check if
it equals target. However, that solution is too trivial (Time: $\boldsymbol{O(n)}$) - the problem is asking to solve this in $\boldsymbol{O(\log(n))}$

### Prelude: Finding pivot
Before we try to solve this problem, first we will discuss how to find the `pivot` (the first element of the array in sorted ascending order - i.e. first element of the array if it was not rotated) using **binary seach**.

For binary search, if we are able to find a `condition` that assigns each element a `True` or `False` value, we can use it to find the **first/last** `True/False`:

```
Condition: a[i] >= a[0] for all indicies i
ex: [4,5,6,7,0,1,2]
 -> [T,T,T,T,F,F,F]
We can use Binary search to find pivot (look for first F)

However, this condition doesn't work if array is not rotated:
ex: [0,1,2,4,5,6,7]
 -> [T,T,T,T,T,T,T]

If we look for first F, our Binary search will give n as the result as we are 
expecting that the first F will come after the prefix of T's

Instead, we should use the condition: a[i] <= a[n-1] for all indicies i
ex: [4,5,6,7,0,1,2]
 -> [F,F,F,F,T,T,T]
We can use Binary search to find pivot (look for first T)

ex: [0,1,2,4,5,6,7]
 -> [T,T,T,T,T,T,T]
The logic still holds: We can use Binary search to find pivot (look for first T)
```
[Source](https://youtu.be/GU7DpgHINWQ?t=1089)

### Approach 1: Find Pivot Index + Binary Search
We will first find where the **pivot (minimum element)** is in the array (let's assume the pivot's index is `k`). Then we will use **2 binary searches** to look for target between `[0..k-1]` and `[k..n-1]`:

```cpp
int binarySearch(const vector<int> &a, int l, int r, int t)
{
    while (l <= r)
    {
        int mid = l + (r - l) / 2;
        if (a[mid] == t)
            return mid;
        if (a[mid] > t)
            r = mid - 1;
        else
            l = mid + 1;
    }
    return -1;
}

int search(vector<int> &a, int t)
{
    int n = a.size();
    int l = 0, r = n - 1;
    // Condition: a[i] <= a[n-1] for all i (find first T)
    while (l < r)
    {
        int mid = l + (r - l) / 2;
        if (a[mid] <= a[n - 1])
            r = mid;
        else
            l = mid + 1;
    }
    // Now l and r both equal the pivot's index
    if (l - 1 >= 0 && t >= a[0] && t <= a[l - 1])
        return binarySearch(a, 0, l - 1, t);
    else if (t >= a[l] && t <= a[n - 1])
        return binarySearch(a, l, n - 1, t);
    return -1;
}
```

### Approach 2: Find Pivot Index + Binary Search with Shift
The array we're working with has been rotated by a certain number of steps (`shift`), which means we can't apply a regular binary search to the modified array. However, if we can revert this array to its original sorted form, then a conventional binary search becomes a viable approach.

Once we find `pivot` (in code, this means the index of pivot element), the `shift` is equal to `n-pivot` i.e. if we rotate the array `n-pivot` times, we will get a sorted array. Therefore, if we map each element's "sorted" index to their index in the array then we can do normal binary search:

```cpp
int shiftedBinarySearch(const vector<int> &a, int pivInd, int t)
{
    int n = a.size();
    int shift = n - pivInd;
    int l = 0, r = n - 1;
    while (l <= r)
    {
        int mid = l + (r - l) / 2;
        int mappedInd = (mid - shift + n) % n;

        if (a[mappedInd] == t)
            return mappedInd;
        if (a[mappedInd] > t)
            r = mid - 1;
        else
            l = mid + 1;
    }
    return -1;
}

int search(vector<int> &a, int t)
{
    int n = a.size();
    int l = 0, r = n - 1;
    // Condition: a[i] <= a[n-1] for all i (find first T)
    while (l < r)
    {
        int mid = l + (r - l) / 2;
        if (a[mid] <= a[n - 1])
            r = mid;
        else
            l = mid + 1;
    }
    // Now l and r both equal the pivot's index
    return shiftedBinarySearch(a, l, t);
}
```

### Approach 3: One Binary Search
Even though the array isn't fully sorted, can we still use a normal binary search? If we think about it, in any iteration of binary seach, the pivot will only be on one side of the division. The other side will be a normally sorted [array](https://leetcode.com/problems/search-in-rotated-sorted-array/Figures/33/q0.png).

So in each iteration, if we just look at the sorted array section and ask: does `target` fall in this section?
If target falls in this section, that means we can discard the other section. If target doesn't fall in this section, then we know it has to be in the other half. In order to check if target falls in the sorted section from `[l..r]`, we just check: `a[l] <= target <= a[r]`.

```cpp
int search(vector<int> &a, int t)
{
    int n = a.size();
    int l = 0, r = n - 1;
    while (l <= r)
    {
        int mid = l + (r - l) / 2;
        if (a[mid] == t)
            return mid;
        if (a[mid] >= a[l])
        {
            if (t >= a[l] && t < a[mid])
                r = mid - 1;
            else
                l = mid + 1;
        }
        else
        {
            if (t > a[mid] && t <= a[r])
                l = mid + 1;
            else
                r = mid - 1;
        }
    }
    return -1;
}
```
**Note**: Instead of comparing to `a[l]` and `a[r]`, we could use `a[0]` and `a[n-1]` respectively. These gives the same answer as only one subarray (left or right) can be sorted.

**Note**: In the condition, `a[mid] >= a[l]`, the `=` is necessary. This is because of the edge case where `mid==l` (when there are 2 elements):
```
Test case:
[3, 1]
target = 1
```
Here: `l==0`, `r==1`, and `mid==0` (i.e. `mid==l`). If the condition was `a[mid] > a[l]`, it would fail and assume the right subarray, `[mid..r]`, is sorted - which it isn't. Basically, `a[mid]` will always be part of one sorted subarray, either left or right . Before assuming it is part of the right sorted subarray, we first confirm it is not part of the left sorted subarry (which is when `a[mid] >= a[l]` fails). Therefore, `[l..mid]` is sorted iff `a[mid] >= a[l]` (`=` because `mid==l`).