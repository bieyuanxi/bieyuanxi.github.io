+++
title = "串"
date = "2023-11-14"

[taxonomies]
tags = ["串"]
+++
# 串的表示
1. 定长顺序存储：用一组地址连续的存储单元存储串值的字符序列，但是长度静态分配。
1. 堆分配存储：用一组地址连续的存储单元存储串值字符序列，其存储空间在程序执行过程中**动态分配**。
1. 块链存储表示：用链表方式存储串值。

## 定长顺序存储表示
按照预定义的大小，为串分配一个固定长度的存储区。

超过预定义长度的串值将被舍去，即“截断”。

对串长有2种表示方法：在下标0处存串长；在结尾处加入一个不计入串长的结束标记字符，如‘\0’。

实现串操作主要为**字符序列的复制**，时间复杂度基于复制的字符长度。操作时超过上限时约定用截尾法处理。

```c
#define MAXSTRLEN 255
typedef unsigned char SString[MAXSTRLEN + 1];
```


## 块链存储表示
链表存储：存在结点大小的问题：每个结点可以存放1个字符或多个字符(所有结点大小相同)。链表最后一个结点不一定全满，可以用‘#’或非串值字符填充标记。

对于联结操作（联结时需要处理串尾的无效字符，如何处理？TODO）等有一定方便之处，但占用存储量大且操作复杂，总体不如另外两种存储结构灵活。

```c
#define CHUNKSIZE 0x40
typedef struct Chunk {
    char ch[CHUNKSIZE];
    struct Chunk *next;
} Chunk;
typedef stuct {
    Chunk *head;
    Chunk *tail;    //串尾结点指针，便于联结操作
    int curlen;
} LString;
```
### 存储密度
链式存储方式中，结点大小的选择直接影响串处理的效率。串的字符集的大小也是一个重要因素。

字符集小，字符的机内编码就短。

存储密度=串值所占的存储位/实际分配的存储位

存储密度小（结点小），运算处理方便，但存储占用大。


## 堆分配存储表示
既有顺序存储结构的特点，对串长又没有任何限制。
```c
typedef struct {
    char *ch;   //若是非空串，按串长length分配存储区，否则ch为NULL
    int length; //串长
} HString;
```

## 串的模式匹配算法
子串的定位操作通常称作串的模式匹配。
### BF
O(m*n)

### KMP TODO
O(m+n)
```c
// next_arr表示模式串前index+1个元素组成的子串的前后缀相同的长度
void get_next_arr(char *pattern, int *next_arr) {
    int len = strlen(pattern);
    next_arr[0] = 0;
    for(int i = 1, j = 0; i < len; ++i) {
        while(j && pattern[i] != pattern[j]) j = next_array[j - 1];
        if(pattern[i] == pattern[j]) j++;
        next_arr[i] = j;
    }
}
```
## 串操作举例
1. 文本编辑
1. 建立词索引表






