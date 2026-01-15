+++
title = "手搓系列：优先队列"
date = "2026-01-15"
description = "手搓系列"
draft = false

[taxonomies]
tags = ["优先队列", "Priority Queue", "堆", "Heap"]
+++

优先队列的核心是构建堆&维护堆，其中，插入操作和移除操作都需要维护堆。可以使用数组构建堆。

插入操作的主要步骤是：将要插入的元素放到末尾，之后向上进行堆化处理（`heaplify_up()`）。

移除操作的主要步骤是：将堆顶元素与末尾元素交换，之后向下堆化（`heaplify()`），并且堆化完成后将最后一个元素取出。

## 基本操作
|操作	|功能描述	|时间复杂度（堆实现）|
| ---- | ---- | ---- |
|push	|插入一个元素，并维护优先级顺序 |O(log n) |
|pop |移除并返回优先级最高的元素|O(log n)|
|top/peek|	查看（不移除）优先级最高的元素|O(1)|
|len |获取队列元素个数|O(1)|
|is_empty	|判断队列是否为空	|O(1)|

## 实现
下面实现的是`最小优先队列`(`小顶堆`)，每次`pop()`操作得到的是最小值。


```rust
#[derive(Default, Debug)]
pub struct PQueue<T> {
    inner: Vec<T>,
}

impl<T: PartialOrd> From<Vec<T>> for PQueue<T> {
    fn from(mut v: Vec<T>) -> Self {
        let n = v.len();
        if !is_parent(n - 1) {
            for i in (0..=parent(n - 1)).rev() {    // O(n)
                heaplify(&mut v, i, n);
            }
        }

        Self { inner: v }
    }
}

impl<T: PartialOrd> PQueue<T> {
    pub fn with_capacity(cap: usize) -> Self {
        Self { inner: Vec::with_capacity(cap) }
    }
    
    pub fn len(&self) -> usize {
        self.inner.len()
    }

    pub fn push(&mut self, v: T) {
        let len = self.inner.len();
        
        self.inner.push(v);
        heaplify_up(&mut self.inner, len);
    }

    pub fn pop(&mut self) -> Option<T> {
        let n = self.inner.len();
        if n > 0 {
            self.inner.swap(0, n - 1);
            heaplify(&mut self.inner, 0, n - 1);
            self.inner.pop()
        } else {
            None
        }
    }

    pub fn top(&self) -> Option<&T> {
        self.inner.first()
    }

}

fn heaplify<T: PartialOrd>(heap: &mut Vec<T>, mut i: usize, len: usize) {
    let len = len.min(heap.len());
    while exist(lchild(i), len) {
        let j = if exist(rchild(i), len) && less_than(&heap[rchild(i)], &heap[lchild(i)]) {
            rchild(i)
        } else {
            lchild(i)
        };

        if less_than(&heap[j], &heap[i]) {
            heap.swap(i, j);
            i = j;
        } else {
            break;
        }
    }
}

#[inline]
pub fn less_than<T: PartialOrd>(a: &T, b: &T) -> bool {
    *a < *b
}

#[inline]
pub fn more_than<T: PartialOrd>(a: &T, b: &T) -> bool {
    *a > *b
}

fn heaplify_up<T: PartialOrd>(heap: &mut Vec<T>, mut i: usize) {
    while !is_parent(i) && less_than(&heap[i], &heap[parent(i)]) {
        let j = parent(i);
        heap.swap(i, j);
        i = j;
    }
}

#[inline]
fn is_parent(i: usize) -> bool {
    i == 0
}

/// if i equals 0, it will panic
#[inline]
fn parent(i: usize) -> usize {
    assert!(i > 0);
    (i - 1) >> 1
}

#[inline]
fn lchild(i: usize) -> usize {
    2 * i + 1
}

#[inline]
fn rchild(i: usize) -> usize {
    lchild(i) + 1
}

#[inline]
fn exist(i: usize, n: usize) -> bool {
    i < n
}
```

测试用例：
- [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

```rust
#[cfg(test)]
mod test {
    use crate::priority_queue::PQueue;

    #[test]
    fn test1() {
        let mut pq = PQueue::default();
        
        pq.push(5);
        pq.push(3);
        pq.push(4);
        pq.push(2);
        pq.push(1);
        pq.push(6);
        
        println!("{:#?}", pq);
        
        assert_eq!(Some(1), pq.pop());
        assert_eq!(Some(2), pq.pop());
        assert_eq!(Some(3), pq.pop());
        assert_eq!(Some(4), pq.pop());
        assert_eq!(Some(5), pq.pop());
        assert_eq!(Some(6), pq.pop());
        assert_eq!(None, pq.pop());
    }
    
    #[test]
    fn test2() {
        let mut pq = PQueue::default();
        
        pq.push(5);
        pq.push(3);
        pq.push(4);
        pq.push(2);
        pq.push(1);
        pq.push(6);
        
        assert_eq!(Some(1), pq.pop());
        assert_eq!(Some(2), pq.pop());
        assert_eq!(Some(3), pq.pop());

        pq.push(-1);
        
        assert_eq!(Some(&-1), pq.top());
    }
    
    #[test]
    fn t3() {
        let mut pq = PQueue::from(Vec::from([1]));
        
    }
    
    #[test]
    fn t4() {
        
        let mut pq = PQueue::from(Vec::from([2,10,8,7,5,4,3,9,6,0,1]));
        let k = 9;
        let target = pq.len() - k;
        for _ in 0..target {
            println!("{:#?}", pq.pop());
            
        }
        assert_eq!(Some(2), pq.pop())
    }
}
```