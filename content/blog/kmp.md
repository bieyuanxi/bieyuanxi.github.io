+++
title = "KMP算法"
date = "2024-05-11"
description = ""
draft = false

[taxonomies]
tags = ["KMP", "字符串匹配"]
+++

KMP 算法的核心是利用模式串的前缀函数（部分匹配表）跳过无效匹配，将字符串匹配的时间复杂度优化到 `O(n+m)`（n 是文本串长度，m 是模式串长度）。

前缀函数 `next[i]` 表示 `pattern[0..=i]` 的最长相等**真前缀**和**真后缀**的长度。
| pattern | a | a | a | b | b | a | b |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| next | 0 | 1 | 2 | 0 | 0 | 1 | 0 |

| pattern | a | b | e | a | b | a | b | e | a | b | f |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| next | 0 | 0 | 0 | 1 | 2 | 1 | 2 | 3 | 4 | 5 | 0 |

> 真前缀：不包含最后一个元素

> 真后缀：不包含第一个元素

```rust
fn kmp(haystack: String, pattern: String) -> i32 {
    let haystack: Vec<char> = haystack.chars().collect();
    let pattern: Vec<char> = pattern.chars().collect();
    
    let m = pattern.len();
    let n = haystack.len();

    // 1. 构建 next 数组
    let mut next = vec![0; m];
    let mut nxt = 0;
    for i in 1..m {
        while nxt > 0 && pattern[nxt] != pattern[i] {
            nxt = next[nxt - 1];
        }
    
        if pattern[nxt] == pattern[i] {
            nxt += 1;
        }
        next[i] = nxt;
    }
    
    // 2. KMP 匹配
    let mut nxt = 0;
    for i in 0..n {
        while nxt > 0 && haystack[i] != pattern[nxt] {
            nxt = next[nxt - 1];
        }
    
        if haystack[i] == pattern[nxt] {
            nxt += 1;
        }
    
        if nxt >= m {
            return (i - m + 1) as i32;
        }
    }
    
    -1
}
```

## 例题
- [28.找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)
- [259.重复的子字符串](https://leetcode.cn/problems/repeated-substring-pattern/description/)