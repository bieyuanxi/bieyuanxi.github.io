+++
title = "数组和广义表"
date = "2023-11-01"

[taxonomies]
tags = ["数组", "广义表"]
+++
# 数组和广义表
数组一旦定义，它的维数和维界就不再改变。因此除了结构的初始化和销毁，数组只有存取元素和修改元素值的操作，因此采用顺序存储结构，随机存取结构。

# 数组的顺序表示和实现
对于数组，规定了维数和各维的长度，便可为其分配空间。

```c
typedef int Elem;

// array[h][m][l]
typedef struct {
    Elem *base;     //映射的一维数组基址
    int dim;        //维数
    int *bounds;    //各维的长度，越界判断
    int *constants; //各维的步长，步长 = 上一个维度的长度 * 上一个维度的步长
} Array, *Array_p;
```

```c
// 初始化
void init(Array array, int dim, ...) {
    if(dim < 1 || dim > MAX_DIMONSION) exit(1);

    array->dim = dim;
    array->bounds = (int*)malloc(sizeof(int) * dim);
    array->constants = (int*)malloc(sizeof(int) * dim);

    if(!(array->bounds && array->constants)) exit(1);

    va_list ap;
    va_start(ap, dim);

    int sum = 1;
    for (int i = 0; i < dim; ++i) {
        array->bounds[i] = va_arg(ap, int);
        sum *= array->bounds[i];
    }大小;  //显然最后一级数组步长为1
    for (int i = dim - 1; i > 0; --i) {
        array->constants[i - 1] = array->constants[i] * array->bounds[i];
    }

    array->base = (int*)malloc(sizeof(int) * sum);
    if(!array) exit(1);
    va_end(ap);
}
```
# 矩阵的压缩存储
压缩存储：对多个值相同的元只分配一个存储空间；对零元不分配空间。

特殊矩阵：值相同的元素或者零元素在矩阵中的分部有一定规律。反之称为稀疏矩阵。

## 特殊矩阵
1. 三角矩阵：对称矩阵/上/下三角矩阵：为每一对对称元分配一个存储空间，n^2 -> (n+1)*n/2
    ```c
    则对矩阵A[i][j]（i、j从1开始），一维数组Array[index], 
    index = (i-1)*i/2+j-1 (i >= j)
    index = (j-1)*j/2+i-1 (i < j) //实际就是i、j互换
    ```
    对上/下三角矩阵可以多存储一个常数c表示另外区域的值。

1. 对角矩阵：如三对角矩阵。
    ```c
    +--- i=1, j = 1, 2
    |
    +--- 1<i<n, j = i-1, i, i+1
    |
    +--- i=n, j = i-1, i
    
    index = 3(i-1) - 1 + (j - i) + 1 = 2*i + j - 3
    ```

## 稀疏矩阵
假设在m * n的矩阵中有t各元素不为0,令a=t/(m*n)，称a为矩阵的**稀疏因子**。
通常认为a<=0.05时称为稀疏矩阵。

存储稀疏矩阵时，除了要存储值，还要存储值的位置，即用三元组(i, j, v)表示一个非零元。

根据对三元组表的不同表示方法，分为：三元组顺序表、行逻辑链接的顺序表、十字链表。

### 三元组顺序表
以顺序存储结构表示三元组表。

优点：按行序有序存储，便于按行顺序处理的矩阵运算。
缺点：不能按行号随机存取某一行的非零元，需要从头查找。

```c
typedef struct {
    int row;
    int col;
    Elem val;
} Triple;

typedef struct {
    Triple data[MAXSIZE + 1];
    int row_num;
    int col_num;
    int total_num;
} TSMatrix;
```

#### 转置
普通矩阵的转置，O(row*col)

三元组两种算法：
1. 列序递增转置法：O(row*num) 适用于num << row*col
1. 一次定位快速转置法：O(row + num) 设立num[col]表，遍历三元组表得到每一列个数，时间复杂度O(num)。随后生成每一列第一个元的位置，时间复杂度O(col)。之后遍历三元组表得到转置结果。

### 行逻辑链接的顺序表
为了随机存取**任意一行**的非零元，需要知道每一行的第一个非零元在三元组表中的位置。因此在三元组表基础上加入了指示行信息的辅助数组。
优点：便于随机存取任意一行的非零元。

```c
typedef struct {
    Triple data[MAXSIZE + 1];
    int row_pos[MAXRC + 1]; //记录每一行第一个元素的位置
    int row_num;
    int col_num;
    int total_num;
} TSMatrix;
```

### 十字链表
当矩阵的非零元个数和位置在操作过程中变化较大时（插入删除），不宜采用顺序存储结构来表示三元组的线性表。

因此采用链式存储结构表示三元组的线性表。
```c
typedef struct OLNode {
    int i, j;
    Elem val;
    struct OLNode *right, *down;    //记录下一个的指针，right是同一行，down是同一列
} OLNode, *OLink;

typedef struct {
    OLink *rhead, *chead;
    int row_num, col_num, total_num;
} CrossList;

//插入算法实现：
//先在行表中插入，之后在列表中插入。
// 1.行插入
OLink p = list->rhead[node->i];
if(p == NULL || p->j > node->j) {
    node->right = p;
    list->rhead[node->i] = node;
} else {
    OLink p = list->rhead[node->i];
    while(p->right && p->right->j < node->j) p = p->right;  //循环找到合适的位置
    node->right = p->right;
    p->right = node;
}
// 2.列插入......

//删除算法实现：
//先在行表删除，之后在列表删除
QLink del = NULL;
//行表删除
OLink p = list->rhead[node->i];
if(p == NULL) return;       //是空的
if(p->j == node->j) {       //删除的是第一个
    del = p;
    list->rhead[node->i] = p->right;
} else {    //删除的不是第一个，设p为要删除的父结点
    while(p->right && p->right->j != node->j) p = p->right;
    if(p->right) {  //如果找到了
        del = p->right;
        p->right = p->right->right;
    }
}
//列表删除......

// 最后返回del即可

```
在建立十字链表时，如果插入的位置随机，时间复杂度：O(total_num * max{row_num, col_num});
如果按照行序为主序顺序插入，则时间复杂度：O(total_num)。（两个数组记录行列最后一个结点地址）


# 广义表的定义
广义表是线性表的推广，也被称为列表(lists，用复数形式与通称的表list区别)。

广义表一般记作： LS = (a1, a2, ..., an)，LS是表名，ai可以是单个元素（广义表的原子）也可以是广义表（广义表的子表）。

1. 列表的元素可以是子表，子表的元素也可以是子表。
1. 列表可以为其他列表共享。
1. 列表可以是一个递归的表，即列表也可以是其本身的一个子表。

## 基本概念
1. 广义表：n个表元素组成的有限序列。
1. 表头(Head)：称第一个元素a1为LS的表头。
1. 表尾(Tail)：称其余元素组成的(a2, a3, ..., an)是LS的表尾。（注意没有去掉最外一层的括号）
1. 长度：最外层括号内元素的个数（第一个括号里用逗号分割出的元素的个数）。
1. 深度：括号的层次最大数。

## 性质
任何一个非空列表其表头可能是原子，也可能是列表，而**其表尾必定为列表**。

```c
//example:
A=()    //A是空表，长度为0
B=(e)   //B只有一个原子，长度为1
C=(a, (b, c, d))    // C长度为2,两个元素分别为原子a，子表(b, c, d)
D=(A, B, C) //D长度为3，3个元素都是子表
E=(a, E)    //E是递归表，长度为2

//列表()和列表(())不同，前者为空表，长度为0，后者长度为1
GetHead(B) = e;         GetTail(B) = ();
GetHead(C) = a;         GetTail(C) = ((b, c, d));   //*注意这里两个括号
GetHead(D) = A;         GetTail(D) = (B, C);
GetHead((B, C)) = B;    GetTail((B, C)) = (C);
```

## 广义表的存储结构

### 广义表的头尾链表存储表示
1. 列表的表头指针，除指向空表时表头指针为空外，指向任何非空列表时，其表头指针均指向一个表结点，且该结点中的hp指向列表表头；tp指向列表表尾，tp为空代表表尾为空。
1. 最高层的表结点的个数即为列表的长度。



```c
typedef enum {ATOM, LIST} ElemTag;  //ATOM:原子；LIST: 子表
typedef int AtomType;

/** 广义表的头尾链表存储表示 */
typedef struct GLNode {
    ElemTag tag;    //区分原子结点还是子表结点
    union {
        AtomType atom;  //原子结点的值
        struct {struct GLNode *hp, *tp;} ptr;   //表结点的指针域
    };
} *GList;   //广义表类型

void GetHead(GList lists, GLNode **node) {
    if(lists) *node = lists->ptr.hp;
    else *node = NULL;
}

void GetTail(GList lists, GLNode **node) {
    if(lists) *node = lists->ptr.tp;
    else *node = NULL;
}

ElemTag is_lists(GLNode *node) {
    if(!node || !node->tag) return 0;
    else return 1;
}

```
    
### 广义表的拓展线性链表表示

```c
typedef struct GLNode {
    ElemTag tag;    //区分原子结点还是子表结点
    union {
        AtomType atom;  //原子结点的值
        struct GLNode *head_ptr;   //表结点的指针域
    };
    struct GLNode *tail_ptr;
} *GList;   //广义表类型
```

