---
layout: post
title: "정렬 알고리즘 정복하기"
tags: [알고리즘]
comments: true
---

> 여러가지 정렬 알고리즘 코드  

여러가지 정렬 알고리즘들의 개념을 확실히 복습 할 겸 직접 구현해보았다.

## O(n²)

### 1. bubble sort

```cpp
void bubble_sort() {
    // 인접한 두 수를 비교하여 swap 하는 과정 한 번을 할 때마다 가장 큰 수가 특정된다.
    // 그러므로 이 과정을 n번 반복하면 정렬이 완성된다.

    for (int i = 0; i < n; i++) {
        // 인접한 두 수를 0부터 끝까지 비교하며 위치를 바꾼다 (거품처럼 swapping이 일어날 것)
        for (int j = 0; j < n - 1; j++) {
            if (arr[j] > arr[j + 1]) swap(&arr[j], &arr[j + 1]);
        }
    }
}
```

### 2. selection sort

```cpp
void selection_sort() {
    for (int i = 0; i < n; i++) {
        int idx = i;
        for (int j = i; j < n; j++) {
            // 가장 작은 수를 "선택"한다
            if (arr[j] < arr[idx]) {
                idx = j;
            }
        }
        // 선택한 가장 작은 수를 i자리(가장 앞 자리)에 갖다 놓는다
        swap(&arr[i], &arr[idx]);
    }
}
```

### 3. insertion sort

```cpp
void insertion_sort() {
    for (int i = 0; i < n; i++) {
        int key = arr[i];
        int j = i - 1;

        while(j >= 0 && key < arr[j]) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

## O(n logn)

### 1. quick sort

```cpp
void quick_sort(int * arr, int l, int r) {
    if (l >= r) return;

    // pivot을 잡는다
    int mid = (l + r) / 2;
    int pivot = get_mdeian(l, mid, r);

    // pivot 값을 오른쪽 끝으로 보내놓고 시작하자
    swap(&arr[pivot], &arr[r]);
    pivot = r;

    int x = l;
    for (int i = l; i < r; i++) {
        // pivot 보다 작으면 swap 후 x를 전진시킨다
        // pivot 보다 크다면 아무것도 하지 않는다
        if (arr[i] <= arr[pivot]) swap(&arr[x++], &arr[i]);
    }

    // x - 1까지가 pivot보다 작은 수들, x부터 pivot - 1까지가 pivot보다 큰 수들이므로
    swap(&arr[x], &arr[pivot]);

    quick_sort(arr, l, x - 1);
    quick_sort(arr, x + 1, r);
}
```

### 2. merge sort (top down)

```cpp
void merge_sort_top_down(int * arr, int l, int r) {
    if (l == r) return;

    int mid = (l + r) / 2;

    // divide
    merge_sort_top_down(arr, l, mid);
    merge_sort_top_down(arr, mid + 1, r);

    // conquer
    int x = l, y = mid + 1, k = l;

    while(x <= mid && y <= r) {
        if (arr[x] < arr[y]) temp[k++] = arr[x++];
        else temp[k++] = arr[y++];
    }

    while(x <= mid) temp[k++] = arr[x++];
    while(y <= r) temp[k++] = arr[y++];

    for (int i = l; i <= r; i++) arr[i] = temp[i];
}
```

### 3. merge sort (bottom up)

```cpp
void merge_sort_bottom_up() {
    for (int size = 1; size < n; size *= 2) {
        for (int left = 0; left < n; left += size * 2) {
            int mid = min(left + size - 1, n - 1);
            int right = min(left + (size * 2) - 1, n - 1);

            // merge left...mid, mid + 1...right
            int x = left, y = mid + 1, k = left;
            while(x <= mid && y <= right) {
                if (arr[x] < arr[y]) temp[k++] = arr[x++];
                else temp[k++] = arr[y++];
            }

            while(x <= mid) temp[k++] = arr[x++];
            while(y <= right) temp[k++] = arr[y++];

            for (int i = left; i <= right; i++) arr[i] = temp[i];
        }
    }
}
```

## O(n)

### 1. counting sort

```cpp
#include <cstdio>

using namespace std;

#define MAXN 10000000

int counting[10001] = { 0, };

int main() {
    int n;
    scanf("%d", &n);

    int max = 0;
    for (int i = 0; i < n; i++) {
        int temp;
        scanf("%d", &temp);

        // 최대값을 미리 구하고,
        if (max < temp) max = temp;
        // 계수 카운트
        counting[temp]++;
    }

    for (int i = 0; i <= max; i++) {
        // 메모리 효율을 위해 배열을 선언하기보다 counting 개수만큼 출력
        for (int j = 0; j < counting[i]; j++) {
            printf("%d\n", i);
        }
    }

    return 0;
}
```
