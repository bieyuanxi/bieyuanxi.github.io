+++
title = "内部排序"
date = "2023-11-18"

[taxonomies]
tags = ["排序"]
+++
# 内部排序
1. 稳定、不稳定
1. 

```c
typedef int ElemType;

typedef struct {
    ElemType *buf;
    int len;
} SqList;

// a是index较小的元素的地址，b是index较大的元素的地址
// 返回值>0： 交换a、b的值
// 返回值=0： a、b相等
// 返回值<0： 不交换a、b的值
typedef int (*cmp)(const void *a, const void *b);
```

## 分类
按照排序过程中依据的不同原则：
1. 插入排序：直接插入排序、折半插入排序、希尔排序
1. 交换排序：冒泡排序、快速排序
1. 选择排序：简单选择排序、堆排序
1. 归并排序
1. 计数排序：基于分配和收集。



## 插入排序
将序列看成已排序+未排序两部分，每次从未排序部分选取一个插入到已排序的序列，初始约定序列第一个元素已排序。
### 直接插入排序
1. 分类：插入排序
1. 适用范围：适合长度n较小的序列、基本有序的序列
1. 适用数据结构：适用于顺序表、链表
1. 时间复杂度：
    * 平均：O($n^2$)
    * 最好：O($n$)：原本有序
    * 最坏：O($n^2$)：原本逆序
1. 空间复杂度：O(1)
1. 稳定性：稳定
```c
// 直接插入排序
void insert_sort(SqList *list, cmp compare) {
    ElemType temp;
    for (int i = 1; i < list->len; ++i) {   //从i=1开始
        temp = list->buf[i];    //暂存
        int j = i;
        for (; j > 0 && compare(list->buf + j - 1, &temp) > 0; --j) {
            list->buf[j] = list->buf[j - 1];
        }
        list->buf[j] = temp;    //插入
    }
}

### 折半插入排序
1. 分类：插入排序
1. 适用范围：初始序列基本无序
1. 适用数据结构：仅适用于顺序表
1. 时间复杂度：
    * 平均：O($n^2$)
1. 空间复杂度：O(1)
1. 稳定性：稳定
// 折半插入排序
void binary_insert_sort(SqList *list, cmp cmp) {
    ElemType temp;
    for (int i = 1; i < list->len; ++i) {   //从i=1开始
        temp = list->buf[i];    //暂存
        int low = 0, high = i - 1;
        
        while (low <= high) {
            int mid = (low + high) >> 1;
            if (compare(list->buf + mid, &temp) > 0) {
                high = mid - 1;
            }else {
                low = mid + 1;
            }
        }
        int j = i;
        for (; j > low; --j) {  
            list->buf[j] = list->buf[j - 1];
        }
        list->buf[j] = temp;    //  在low(或者high+1)的位置插入
    }
}
```

## 希尔排序/缩小增量排序
增量序列最好是质数序列（避免前几次都是无效排序），且序列最后一位是1

1. 分类：插入排序
1. 适用范围：适合长度n较大的序列
1. 适用数据结构：仅适用于顺序表
1. 时间复杂度：
    * 平均：O($n^{1.3}$)
1. 空间复杂度：O(1)
1. 稳定性：不稳定

不怎么考察代码，常考的题目：给出增量序列，分析每一趟排序后的状态
```c
void shell_sort(SqList *list, cmp cmp) {
    for (int step = list->len >> 1; step > 0; step >>= 1) {
        for (int i = step; i < list->len; i++) {   //从i=1开始
            int temp = list->buf[i];    //暂存
            int j = i;
            for (; j > 0 && compare(list->buf + j - step, &temp) > 0; j -= step) {
                list->buf[j] = list->buf[j - step];
            }
            list->buf[j] = temp;    //插入
        }
    }
}
```

## 冒泡排序
1. 分类：交换排序
1. 适用范围：n较小时
1. 适用数据结构：适用于顺序表、链表
1. 时间复杂度：
    * 平均：O($n^2$)
    * 最好：O($n$)：原本有序
    * 最坏：O($n^2$)：原本逆序
1. 空间复杂度：O(1)
1. 稳定性：稳定
```c
void bubble_sort(SqList *list, cmp cmp) {
    for (int i = list->len - 1; i > 0; --i) {
        for (int j = 0; j < i; ++j) {
            if (cmp(list->buf + j, list->buf + j + 1) > 0) {
                ElemType temp = list->buf[j + 1];
                list->buf[j + 1] = list->buf[j];
                list->buf[j] = temp;
            }
        }
    }
}
```

## 快速排序
1. 分类：交换排序
1. 适用范围：n较大、无序（不适用于逆序或基本有序）
1. 适用数据结构：仅适用于顺序表
1. 时间复杂度：
    * 平均：O($n\log_2{n}$)：平均性能最优
    * 最好：O($n\log_2{n}$)：原本有序
    * 最坏：O($n^2$)：原本逆序，退化为冒泡排序
1. 空间复杂度：取决于递归深度，每次“划分”越均匀，递归深度越低
    * 最好：O($\log_2{n}$)
    * 最坏：O($n$)
1. 稳定性：不稳定
1. 优化方法：
    1. n较小时改用直接插入；
    1. 事先遍历确保无序（或者做标记，找到枢纽时标记未反转表示有序或逆序）；
    1. 三者取中：用首位、中间、尾位元素的中间值替换首个元素，避免有序情况；
    1. 针对分割后长度较短的先排序，空间复杂度可降至O($\log_2{n}$)

手写提技巧：给出每一趟排序结果时，只需要

```c
// [start, end) 左闭右开
void _quick_sort_rec(ElemType *buf, const int start, const int end, cmp cmp) {
    // 优化：长度小于8,改用直接插入TODO

    if (end -start <= 1) return;    // 长度为1,结束

    // 优化：事先遍历，确保不是有序TODO

    int lp = start, rp = end - 1;
    // 优化：交换中间元素和头元素，随机化，避免有序情况
    int mid = (start + end) >> 1;
    ElemType temp = buf[mid];
    buf[mid] = buf[start];
    buf[start] = temp;

    temp = buf[start]; // 暂存第一个，初始设置start位置为空

    while (lp < rp) {
        while (lp < rp && cmp(&temp, buf + rp) <= 0) rp--;
        if(lp != rp) buf[lp] = buf[rp];

        while (lp < rp && cmp(buf + lp, &temp) <= 0) lp++;
        if (lp != rp) buf[rp] = buf[lp];
    }
    buf[lp] = temp;
    // 得到枢纽lp=rp
    _quick_sort_rec(buf, start, lp, cmp);
    _quick_sort_rec(buf, lp + 1, end, cmp);
}

void quick_sort_rec(SqList *list, cmp cmp) {
    _quick_sort_rec(list->buf, 0, list->len, cmp);
}
```


## (简单)选择排序
每次选择一个最大/最小的加入有序序列。

1. 分类：选择排序
1. 适用范围：
1. 适用数据结构：适用于顺序表、链表
1. 时间复杂度：
    * 平均：O($n^2$)：平均性能最优
    * 最好：O($n^2$)：原本有序
    * 最坏：O($n^2$)：原本逆序
1. 空间复杂度：O(1)
1. 稳定性：不稳定
```c
void select_sort(SqList *list, cmp cmp) {
    for (int i = 0; i < list->len; ++i) {
        int min = i;
        for (int j = i + 1; j < list->len; ++j) {
            if (compare(list->buf + min, list->buf + j) > 0) {
                min = j;
            }
        }
        ElemType temp = list->buf[min];
        list->buf[min] = list->buf[i];
        list->buf[i] = temp;
    }
}
```
## 堆排序
堆排序的一些概念
1. 大顶堆：升序排序
1. 小顶堆：降序排序

----

1. 分类：选择排序
1. 适用范围：n较大
1. 适用数据结构：适用于顺序表
1. 时间复杂度：
    * 平均：O($n\log_2{n}$)：建堆：O(n)，排序（n次维护，每次$\log_2n$）：O($n\log_2{n}$)
    * 最好：O($n\log_2{n}$)：
    * 最坏：O($n\log_2{n}$)：
1. 空间复杂度：O(1)
1. 稳定性：不稳定



```c
#define LCHILD(p) (((p) << 1) + 1)
#define RCHILD(p) (((p) << 1) + 2)
#define PARENT(child) (((child) - 1) >> 1)
#define CHILD_VALID(child, len) (child < len)

void _sink(int *buf, int len, int index, cmp cmp) {
    int next = index;
    while (CHILD_VALID(LCHILD(index), len)) {
        if (compare(buf + LCHILD(index), buf + index) > 0) {
            ElemType temp = buf[index];
            buf[index] = buf[LCHILD(index)];
            buf[LCHILD(index)] = temp;
            next = LCHILD(index);
        }
        if (CHILD_VALID(RCHILD(index), len) 
            && compare(buf + RCHILD(index), buf + index) > 0) {
            ElemType temp = buf[index];
            buf[index] = buf[RCHILD(index)];
            buf[RCHILD(index)] = temp;
            next = RCHILD(index);
        }
        if (next == index) break;
        index = next;
    }
}

void heap_sort(SqList *list, cmp cmp) {
    for (int i = PARENT(list->len - 1); i >= 0; --i) {  //从最后一个有孩子的结点开始
        _sink(list->buf, list->len, i, cmp);    //构建堆
    }

    ElemType temp;
    for (int i = list->len - 1; i >= 1; --i) {
        temp = list->buf[i];
        list->buf[i] = list->buf[0];
        list->buf[0] = temp;
        _sink(list->buf, i, 0, cmp);    //维护堆
    }
}
```

## 归并排序
分治思想，将序列递归地分成两个已排好序的序列（递归到子序列长度为1），之后递归合并

1. 分类：归并排序
1. 适用范围：n较大
1. 适用数据结构：适用于顺序表
1. 时间复杂度：
    * 平均：O($n\log_2{n}$)：分治:O($\log_2{n}$),合并O($n$)
    * 最好：O($n\log_2{n}$)：
    * 最坏：O($n\log_2{n}$)：
1. 空间复杂度：O($n$)：需要存储序列的值，以便合并
1. 稳定性：稳定

```c
#define MIN(a, b) ((a) > (b) ? (b) : (a))

// 非递归
void merge_sort(SqList *list, cmp cmp) {
    ElemType *cache = (ElemType*)malloc(sizeof(ElemType) * list->len);
    
    for (int step = 1; step < list->len; step <<= 1) {
        memmove(cache, list->buf, sizeof(ElemType) * list->len);

        for (int start = 0, mid = start + step, end = MIN(mid + step, list->len);
            mid < list->len; 
            start = end, mid = start + step, end = MIN(mid + step, list->len)) {
            
            int i = start, p = start, j = mid;
            while (i < mid && j < end) {
                if (compare(cache + i, cache + j) > 0) list->buf[p++] = cache[j++];
                else list->buf[p++] = cache[i++];
            }

            while (i < mid) list->buf[p++] = cache[i++];

            while (j < end) list->buf[p++] = cache[j++];
        }
    }
}

void _merge(ElemType *buf, ElemType *cache, int start, int mid, int end, cmp cmp)；
void _merge_sort(ElemType *buf, ElemType *cache, int start, int end, cmp cmp)；

// 递归
void merge_sort_rec(SqList *list, cmp cmp){
    ElemType *cache = (ElemType*)malloc(sizeof(ElemType) * list->len);

    _merge_sort(list->buf, cache, 0, list->len, cmp);
}

void _merge(ElemType *buf, ElemType *cache, int start, int mid, int end, cmp cmp) {
    int i = start, p = start, j = mid;
    while (i < mid && j < end) {
        if (compare(cache + i, cache + j) > 0) buf[p++] = cache[j++];
        else buf[p++] = cache[i++];
    }

    while (i < mid) buf[p++] = cache[i++];

    while (j < end) buf[p++] = cache[j++];
}

void _merge_sort(ElemType *buf, ElemType *cache, int start, int end, cmp cmp) {
    if (end - start <= 1) return;

    int mid = (start + end) >> 1;
    
    // divide O(log2n)
    _merge_sort(buf, cache, start, mid, cmp);
    _merge_sort(buf, cache, mid, end, cmp);

    memmove(cache + start, buf + start, sizeof(ElemType) * (end - start));
    // merge O(n)
    _merge(buf, cache, start, mid, end, cmp);
}
```

## 基数排序
一种借助多关键字排序的思想对单逻辑关键字进行排序的方法。基于分配和收集。

1. 分类：基数排序
1. 适用范围：n较大，关键字d较小
1. 适用数据结构：适用于链表
1. 时间复杂度：
    * 平均：O($nd$)：分治:O($\log_2{n}$),合并O($n$)
    * 最好：O($nd$)：
    * 最坏：O($nd$)：
1. 空间复杂度：O($r$)：基
1. 稳定性：稳定




## 总结
勘误：下图快速排序空间复杂度为O($log_2{n}$)
![](/asset/sort.png)