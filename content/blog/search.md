+++
title = "查找"
date = "2023-11-20"

[taxonomies]
tags = ["查找"]
+++
# 查找
![](/asset/search.png)
```c
typedef float KeyType;  //浮点类型
typedef int KeyType;    //整型
typedef char* KeyType;  //字符串类型
```

1. 基于比较的查找
    1. 静态查找表（线性表）：顺序表、有序表、索引顺序表
    1. 动态查找表（树）：二叉排序树、平衡二叉树、B-树
1. 基于计算的查找：哈希表

## 基本概念
1. 查找表：同一类型的数据元素构成的集合。
1. 静态查找表/动态查找表：
1. 关键字：数据元素中某个数据项的值
1. 主关键字：若某关键字可以唯一地标识一个数据元素，称之为主关键字
1. 次关键字：称用以识别若干数据元素的关键字为次关键字
1. 查找：根据给定的值在查找表中确定一个其关键字等于给定值的数据元素
1. 查找成功时的平均查找长度（Avergae Search Length）：为确定数据元素在查找表中的位置，和给定值进行比较的关键字个数的期望值。
1. 查找算法的平均查找长度：查找成功时的平均查找长度 + 查找不成功时的平均查找长度（通常很小）

## 静态查找表与动态查找表
1. 查询某个特定的数据元素是否在查找表中;
1. 检索某个特定的数据元素的各种属性;
1. 在查找表中插入一个元素;
1. 从查找表中删去一个元素;

若对查找表只做前两种“查找”的操作，则称为**静态查找表**；若在查找过程中同时插入查找表中不存在的元素或者删除已存在的元素，则称为**动态查找表**。

## 静态查找表
主要操作：
1. search()
1. traverse()

### 顺序表查找
以**顺序表**或者**线性链表**表示静态查找表

从表中最后一个元素开始，逐个进行数据关键字和给定值的比较，若相等，则查找成功；若一直到第一个元素都不相等，则查找失败。

时间复杂度：O((n + 1) / 2) = O(n)
```c
//静态查找表的顺序存储结构
typedef struct {
    ElemType *buf;  //数据元素存储空间基址，0号单元留空（哨兵，免去判断查找完毕）
    int len;    //表长度
} SSTable;

//静态查找表的链式存储结构
typedef struct _Node {
    struct _Node *next;
    ElemType elem;
} Node;

typedef struct {
    Node *head;
    int len;
} LSTable;
```

```c
//顺序存储结构的查找
int search_seq(SSTable *table, KeyType key) {   //查找失败返回0,否则返回对应下标
    table->buf[0].key = key;    //监视哨
    int i = 0;
    for(i = table->len; !EQ(table->buf[i].key, key); --i);  //从后往前
    return i;
}

//链式存储结构的查找
Node* search_seq(LSTable *table, KeyType key) {   //查找失败返回NULL,否则返回对应指针
    Node *p = table->head;
    while(p && p->elem.key != key) p = p->next;
    return p;
}
```

### 有序表查找
有折半查找（Binary Search）、斐波那契查找、插值查找

#### 折半查找/二分查找
以有序表表示静态查找表时，search函数可以用折半查找，仅适用于顺序表。
先确定待查记录所在区间，逐步缩小范围知道找到或找不到为止。

设置指针low、high,分别指示区间的下界和上界，指针mid指示区间的中间位置。

时间复杂度：O(|logN| + 1)，O(logN)

```c
// 折半查找仅适用于顺序表
int binary_search(SSTable *table, KeyType key) {
    int low = 1;            //0未启用
    int high = table->len;  //这里指示最后一个元素，即闭区间
    int mid;

    while(low <= high) {
        mid = (low + high) >> 1;
        if(EQ(table->buf[mid].key, key)) {
            return mid;
        }else if(LT(table->buf[mid].key, key)) {
            low = mid + 1;  //mid也被排除
        }else {
            high = mid - 1; // mid也被排除
        }
    }
    return 0;
}
```

#### 斐波那契查找
平均性能比折半查找好，但最坏情况性能（也是O(logN)）比折半查找差，优点是分割时只需进行加减运算。
```c
//假设已经有斐波那契的数列
const int F[] = {0, 1, 1, 2, 3, 5, 8, 13, 21, 34};

int fibonacci_search(int *array, int len, int key) {
    int low = 1;    //0未启用
    
    int n;
    for (n = 0; F[n] < len; n++);   //找到合适的n使得F[n-1] < (table->len) < F[n]
    int mid;
    
    while(n > 0) {
        mid = low + F[n - 1];
        while(mid > len) {
            n--;
            mid = low + F[n - 1];
        }
        if(array[mid] == key) return mid;
        else if(array[mid] < key) { //key在0.618右边
            n -= 2;
            low = mid;
        }else {
            n -= 1;
        }
    }

    return 0;
}
```

#### 插值查找
思想是假设表内数据元素是均匀的，则key所在位置可以由最小关键字low和最大关键字high的比例直接得到，逐步缩小范围。
i = l + (key - arr[l]) / (arr[h] - arr[l]) * (h - l + 1)
**只适用于关键字均匀分布的表**。

### 索引顺序表查找
以索引顺序表表示静态查找表，search函数可以用分块查找实现

设表长为n,将分块均匀分成b块,则索引表长度为b，每个分块长度为s=n/b，ASL = 搜索索引表ASL + 搜索分块ASL = (b+1)/2 + (s+1)/2 = (n/s + s)/2 + 1，显然s = $\sqrt[2]{n}$ 时平均查找长度最小为($\sqrt[2]{n}$ + 1)

若使用折半查找确定所在分块，O($log_2$($\frac {n}{s}$ + 1) + $\frac {s}{2}$)

时间复杂度：O((n/s + s)/2) => 分块长度s = $\sqrt[2]{n}$ 时，O($\sqrt[2]{n}$),

空间复杂度：O(s)，建立索引表，索引表按照关键字有序，则表要么有序要么分块有序。
```c
//索引表：
//关键字    | key1  |key2   |key3   |
//起始地址  |addr1  | addr2 | addr3 |
typedef struct {
    KeyType key;
    int index;  //ElemType *p;
} INode, *IndexTable;

typedef struct {
    ElemType *buf;
    IndexTable itable;  //索引表指针
    int len;
} ISTable;

//为索引顺序表生成索引
int gen_itable(ISTable *table) {
    int s = sqrt(2, table->len);    //令分块长度=n开方
}

int index_sort(ISTable *table, KeyType key) {
    //1. 索引表内查找分块
    int start;  //start表示索引表内最后一个不大于key的起始地址
    int found = binary_search(table->itable, key, &start);  //折半查找索引表
    if(found != 0) return found;    //说明索引表内的正好是要找的

    //1. 分块内查找
    int block_len = sqrt(2, table->len);    //分块长度=n开方
    //顺序表搜索[start, start+block_len -1]的范围
    return search_seq(table, key, start, start + block_len - 1);
}
```
### 静态树表
适用于查找概率不等的有序表TODO

## 动态查找表
动态查找表的特点是表结构本身是在查找过程中动态生成的，即对于给定的key,若表中存在关键字等于key的记录，则查找成功返回，否则插入关键字等于key的记录。


### 二叉排序树(Binary Sort Tree)/二叉查找树
二叉排序树即拥有类似折半查找的特性，又采用了链表作存储结构。

中序遍历二叉排序树可以得到一个关键字的有序序列。

二叉排序树或者是一棵空树，或者是具有下列性质的二叉树：
1. 若他的左子树不为空，则左子树上的所有结点的值均小于它的根节点的值;
1. 若他的右子树不为空，则左子树上的所有结点的值均大于它的根节点的值;
1. 它的左右子树也分别是二叉排序树;

#### ASL
二叉排序树的平均查找长度和形成的树的形态有关，最好情况下O($log_2n$);
最坏情况是退化成线性链表（以有序序列建树时），此时查找、插入、删除时间复杂度为O(n)
```c
无序序列A=(45,24,53,12,37,93)
有序序列B=(12,24,37,45,53,97)
形成的二叉排序树TA和TB,TA高度为3，TB高度为6
ASL(TA) = 1/6*(1+2*2+3*3)   = 14/6
ASL(TB) = 1/6*(1+2+3+4+5+6) = 21/6
```

#### 查找
```c
// 查找成功返回值不为0,p指向该结点
// 查找失败时返回值为0,p指向访问的最后一个结点
int searchBST(LinkBtree tree, TElemType e, BNode **p) {
    if(!tree) return 0;
    *p = tree;  //p始终指向访问的最后一个有效结点
    if(tree->data == e) return 1;
    else if(tree->data < e) return searchBST(tree->rchild, e, p);
    else return searchBST(tree->lchild, e, p);
}
```
#### 插入
一个无序序列可以通过构造一棵二叉排序树而变成一个有序序列，构造树的过程即为对无序序列进行排序的过程。
```c
int insertBST(LinkBtree *tree, TElemType e) {
    BNode *found = NULL;
    if(!searchBST(*tree, e, &found)) {
        BNode *s = (BNode*)malloc(sizeof(BNode));
        s->data = e; s->lchild = s->rchild = NULL;

        if(!found) *tree = s;   //tree本身是空指针
        else if(found->data < e) found->rchild = s;
        else found->lchild = s;
        return 1;
    }
    return 0;
}
```
#### 删除
```c
int deleteBST(LinkBtree *tree, KeyType key) {
    BNode *node = *tree;
    if(!node) return 0; // 如果tree本身为空
    if(node->data == key) return _delete(tree);
    else if(node->data < key) return deleteBST(&node->rchild, key);
    else return deleteBST(&node->lchild, key);
}

int _delete(LinkBtree *tree) {
    BNode *node = *tree;
    if(!node->lchild) *tree = node->rchild;
    else if(!node->rchild) *tree = node->lchild;
    else {
        BNode *p = node;    //指向s的父节点
        BNode *s = node->rchild;
        while(s->lchild) {
            p = s; s = s->lchild;  //寻找node结点的右子树的最左的结点s
        }
        node->data = s->data;   //实际上删除的是s结点，所以要存储s结点的数据和孩子
        if(p == node) p->rchild = s->rchild;    //如果s就是node的孩子结点
        else p->lchild = s->rchild;
        free(s);
    }
    return 1;
}
```




### 平衡二叉树(Balanced Binary Tree)/AVL树
1. 结点的平衡因子(Balance Factor)：该结点的左子树的深度减去右子树的深度。AVL树的全部结点的平衡因子只可能是-1、0、1
1. 最小不平衡子树：距离插入结点最近的、且以平衡因子绝对值大于1的结点为根的树

AVL树是一种特殊的**二叉排序树**，其平均查找长度：O($log_2n$)

深度为$h$的AVL树的最小结点数$n_h$：
$n_0$ = 0，$n_1$ = 1, $n_2$ = 2, $n_h$ = $1 + n_{h-1} + n_{h-2}$ 

AVL树，或者是一棵空树，或者是具有下列性质的二叉树：
1. 它的左子树和右子树都是AVL树；
1. 且左右子树的深度之差的**绝对值**不超过1.

稳态：平衡因子从-1变为0或从1变为0为进入稳态，此时不会影响该结点的父节点；如果从0变为1或-1,则需要更新父节点的平衡因子；如果平衡因子从1变为2或从-1变为-2,则代表失衡，此结点为最小不平衡子树。

在插入新结点导致平衡被打破时，寻找最小不平衡子树，通过“旋转”转为平衡：
1. LL：以结点2为中心，右旋结点3（根节点顺时针旋转）
1. RR：以结点2为中心，左旋结点1（根节点逆时针旋转）
1. LR：以结点2为中心，左旋1,之后以结点2为中心，右旋3（即先RR后LL）
1. RL：以结点2为中心，右旋3,之后以结点2为中心，左旋1（即先LL后RR）

个人理解：所谓旋转，其本质是在不影响二叉排序树性质的前提下，对子树进行扁平化处理，

手写题方法：
1. 按照二叉排序树的规则插入或者删除结点;
1. 找到最小不平衡二叉树，在根节点到插入的结点的路径上取根节点开始的三个相邻结点，以二叉排序树的性质重排这三个顶点，之后把其余结点分别插入即可。

![4种旋转纠正类型](/asset/AVL.jpg "4种旋转纠正类型")

```c
#define LH (+1) //左子树高
#define EH (0)
#define RH (-1)
typedef struct _Node {
    struct _Node *lchild, *rchild;
    ElemType data;
    int bf; // balance factor range{-1, 0, 1}
} Node, *AVLTree;

```

### B-树
B-树是一种平衡的多路查找树，在文件系统中很有用（作文件的索引）。

一棵m阶的B-树，或为空树，或为满足以下特性的m叉树：
1. 树中每个结点最多有m棵子树；
1. 若根节点不是终端结点，则至少有两棵子树；
1. 除根节点之外的所有非终端结点至少有$\lceil m/2 \rceil$棵子树;
1. 所有非叶子结点中包含信息数据：(n, A0, K1, A1, K2, A2, ..., Kn, An)，其中Ki为关键字，按递增排列，Ai为指向子树根节点的指针，且Ai指针所指向的子树所有结点的关键字介于Ki-1和Ki之间。
1. 所有叶子结点都出现在同一层次上，并且不带信息（看作查找失败的结点，实际上指向这些结点的指针为空）

#### 性质
根节点非空时key的个数：[1, $m - 1$]

除根节点外非叶子结点key的个数：[$\lceil m/2 \rceil - 1$, $m - 1$]

含有n个非叶子结点的m阶B树，
1. 最少有$(n - 1) * (\lceil m/2 \rceil - 1) + 1$个关键字：$n - 1$个非叶子结点至少有$\lceil m/2 \rceil - 1$个关键字，根节点至少有1个关键字；
1. 最多有$n*(m-1)$个关键字：全满的时候。

高度为h的m阶B树，????
1. 至少$(m-1)^h - 1$个结点，
1. 至多有$m^h - 1$个结点

含N个关键字的m阶B树的最大深度：每层分配尽可能少的关键字
1. 第一层（根节点）1个关键字;
1. 第二层分配$2*(\lceil m/2 \rceil - 1)$个关键字;
1. 第三层分配$2*(\lceil m/2 \rceil - 1)^2$个，...
1. 第h层分配$2*(\lceil m/2 \rceil - 1)^{h-1}$个

一棵高度为h的m阶B树，其关键字最少为$2*\lceil m/2 \rceil^{h-1}-1$
```c
#define m (3)   //阶数
typedef struct _Node {
    struct *_Node *ptr[m];
    int key[m];
    struct *_Node *parent;  //双亲
    int num;    //关键字个数
    // int 
} Node, BTree;
```

#### 查找
```c
int searchBTree(BTree T, KeyType key) {
    
}
```

#### 插入
针对m阶高度h的B树，插入一个元素时，首先检查在B树中是否存在，如果不存在，即在叶子结点处结束，然后在叶子结点中插入该新的元素。

1. 若该节点元素个数小于m-1，直接插入；
1. 若该节点元素个数等于m-1，插入后引起节点分裂；以该节点中间元素为分界分成$\big\{K_1, K_2, ..., K_{\lceil \frac{m}{2} \rceil - 1} \big\}$，$\big\{ K_{\lceil \frac{m}{2} \rceil} \big\}$、$\big\{K_{\lceil \frac{m}{2} \rceil + 1, ..., K_m} \big\}$三部分，取中间元素插入到父节点中；
1. 重复上面动作，直到所有节点符合B树的规则；最坏的情况一直分裂到根节点，生成新的根节点，高度增加1；

#### 删除
首先查找B树中需删除的元素,如果该元素在B树中存在，则将该元素在其结点中进行删除；删除该元素后，首先判断该元素是否有左右孩子结点，如果有，则上移孩子结点中的某相近元素(“左孩子最右边的节点”或“右孩子最左边的节点”)到父节点中，然后是移动之后的情况；如果没有，直接删除。
1. 某结点中元素数目小于（m/2）-1,(m/2)向上取整，则需要看其某相邻兄弟结点是否丰满；
1. 如果丰满（结点中元素个数大于(m/2)-1），则向父节点借一个元素来满足条件；
1. 如果其相邻兄弟都不丰满，即其结点数目等于(m/2)-1，则该结点与其相邻的某一兄弟结点进行“合并”成一个结点；



### B+树
TODO


## 哈希表/散列表
1. 哈希函数： f(key) = index
1. 哈希表：根据设定好的哈希函数H(key)和处理冲突的方法将一组关键字映射到一个有限的连续的地址区间上，并以关键字在地址集中的“像”作记录在表中的位置，这种表便是哈希表。
1. 同义词：具有相同函数值的关键字对该哈希函数来说称作同义词
1. 冲突：key1!=key2, f(key1) = f(key2)
1. 均匀的哈希函数：
1. 哈希表的装填因子：$\alpha = \frac{表中填入的记录数}{哈希表的长度}$,标志着哈希表的装满程度

查找过程中需要比较的次数取决于：哈希函数、处理冲突的方法、哈希表的装填因子

### 构造方法
1. 直接定址法：取关键字或关键字的某个线性函数值为哈希地址。
即H(key) = key, H(key) = a*key+b，适合关键字分布基本连续的情况，若分布不连续较多会浪费空间。
1. 数字分析法：设关键字是r进制的，并且哈希表中可能出现的关键字都是事先知道的，则可以通过分析特征取关键字的若干位数组成哈希地址。取的位数由表长决定。
1. 平方取中法：取关键字平方后的中间几位作为哈希地址。一个数平方后的中间几位数和数的每一位都有关系。取的位数由表长决定。
1. 折叠法：将关键字分割成位数相同的几部分（最后一部分位数可以不同），取这几部分的叠加和（舍去进位）作为哈希地址。适合关键字位数很多，且关键字中每一位上数字分布大致均匀。
1. 除留余数法：取关键字被某个不大于哈希表表长m的数p除后所得余数为哈希地址。即H(key)=key MOD p, p <= m.一般情况下，可以选p为质数或不包含小于20的质因数的合数。
1. 随机数法：H(key) = random(key), random为随机函数。当关键字长度不等时适合。

#### 除留余数法
可以取最接近m的质数

### 处理冲突的方法
1. 开放定址法：$H_i=(H(key)+d_i)$ MOD $m$ （注意这里以哈希表长度m为模）
1. 再哈希法：$H_i=RH_i(key)$，$RH_i$均是不同的哈希函数。在产生地址冲突时计算另一个地址直到不再冲突。
1. 链地址法：将所有关键字为同义词的记录存储在同一线性链表中。
1. 建立公共溢出区：

#### 开放定址法
1. 线性探测再散列:$d_i=1, 2, 3, ..., m-1$，能保证只要哈希表未满，总能找到不冲突的地址，但是会慢
1. 二次探测再散列:$d_i=1^2, -1^2, 2^2, -2^2, ..., +-k^2$(k<=m/2)。平法探测法要想能探测到所有位置，对于散列表的长度有要求：4*j+3
1. 伪随机探测再散列:$d_i=$ 伪随机序列


#### 链地址法
优化：使线性链表保持有序。


## 各种算法在查找成功和查找失败时的平均查找长度

| Syntax      | 查找成功ASL | 查找失败ASL | 备注 |
| ----------- | ----------- | ----------- | ----------- |
| 无序顺序表      | $\frac {n+1}{2}$       | $n+1$       | 失败时n+1是因为设置了哨兵        |
| 有序表折半查找   | $log_2(n+1) - 1$        | Text        | Text        |
| 索引顺序表   | $\sqrt{n} + 1$        | Text        | 取决于分块大小s，当$s=\sqrt{n} $时       |
| Header      | Title       | Title       |

### 散列表
取决于哈希函数、处理冲突的方法、装填因子

#### 开放定址法
对于空位置的判断也要比较一次
ASL成功：
ASL失败：空的位置也要比较一次

#### 链地址法
ASL失败：空位置不算，ASL失败=装填因子$\alpha$