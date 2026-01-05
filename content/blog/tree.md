+++
title = "树与二叉树"
date = "2023-11-23"
draft = false

[taxonomies]
tags = ["树"]
+++
# 树与二叉树

树是n个结点的有限集。在任意一棵非空树中：
1. 有且只有一个被称为根的结点
1. 当n>1时，其余结点可分为m个互不相交的有限集，其中每一个集合都是一棵树，被称为根的子树。

## 基本概念
1. 树的结点：包含一个数据元素及若干指向其子树的分支。
1. 结点的度：结点拥有的子树的个数。
1. 树的度：树内结点的度的最大值。
1. 叶子结点：度为0的结点。
1. 孩子结点：结点的子树的根，即该结点的直接后继。
1. 双亲结点：该结点的直接前驱。
1. 兄弟结点：同一个双亲的孩子之间互称兄弟结点。
1. 祖先结点：从根到该结点的所经分支上的所有结点。
1. 子孙结点：以该节点为根的子树中的任意结点称为该结点的子孙。
1. 结点的层次：从开始定义，根为第一层，根的直接后继为第二层，以此类推。
1. 树的高度/深度：树中结点的最大层次。
1. 有序树：如果将树中各子树看成从左到右是有次序的（即不能互换），称该树为有序树，否则称为无序树。
1. 森林：森林是m棵**互不相交的树**的集合。

## 二叉树
二叉树每个结点最多只有两棵子树（结点度<=2），且有左右之分，次序不能颠倒。

根据访问根结点(D)、左孩子(L)、右孩子(R)的不同顺序分为：先序遍历(DLR)、中序遍历(LDR)、后续遍历(LRD)；分别得到前缀表示（波兰式）、中缀表示、后缀表示（逆波兰式）。

1. 满二叉树：深度为k且有2^k-1个结点的树。
1. 完全二叉树：深度为k有n个结点的二叉树，当且仅当其每一个结点都与深度为k的满二叉树中编号从1到n的结点一一对应时，称为完全二叉树。

## 二叉树的性质
1. 二叉树的第i层上最多有2^(i-1)个结点。
1. 深度为k的二叉树最多有2^k-1个结点。
1. 设二叉树的叶子结点、度为1的结点、度为2的结点个数分别为n0,n1,n2，则有公式n0=n2+1。
1. 具有n个结点的完全二叉树的深度为[log2(n)] + 1 (这里取下限)
1. 对具有n个结点的完全二叉树，按层序编号，则对任一结点i,有
    1. 若i=1,则i无双亲结点；若i>1,则i的双亲结点为i/2；
    1. 若2 * i>n,则i没有左孩子；否则左孩子结点为2 * i;
    1. 若2 * i+1>n，则i没有右孩子;否则右孩子结点为2 * i + 1;

## 二叉树的存储结构
### 顺序存储结构
用一组地址连续的存储单元依次自上而下，自左至右存储完全二叉树上的结点元素，仅适用于完全二叉树。
```c
#define MAX_TREE_SIZE 128

typedef int TElemType;

typedef TElemType sqBTree[MAX_TREE_SIZE];
```

### 链式存储结构
数据域+左右指针，二叉链表、三叉链表

```c
//二叉链表
typedef struct _BNode {
    struct _BNode *lchild, *rchild;
    TElemType data;
} BNode, *LinkBtree;

//三叉链表
typedef struct _BDNode {
    struct _BDNode *lchild, *rchild;
    struct _BDNode *parent;
    TElemType data;
} BDNode, *DLinkBtree;
```

## 创建二叉树
* 已知前序表示和中序表示，可以唯一确定一棵二叉树；
* 已知后续表示和中序表示，可以唯一确定一棵二叉树；
* 已知前序表示和后续表示，不能唯一确定一棵二叉树；
### 先序序列递归创建
以先序遍历的方式一个一个输入结点（空结点以'#'代替）
```c
BNode* _make_node() {
    BNode *t = (BNode*)malloc(sizeof(BNode));
    t->data = 0;
    t->lchild = NULL;
    t->rchild = NULL;
    return t;
}

void CreateLinkBTreeRecursive(LinkBtree *btree) {
    char in = getchar();
    if(in == '#') {*btree = NULL; return;}
    BNode *node = _make_node();
    if(!node) exit(1);
    
    node->data = in;
    *btree = node;
    CreateLinkBTreeRecursive(&node->lchild);
    CreateLinkBTreeRecursive(&node->rchild);
}
```

### 先序序列+中序序列
给出先序序列和中序序列，得到唯一二叉树
例子：
```c
//先序：ABCDEF
//中序：CBAEDF
// 先序是DLR,所以A是根节点；中序是LDR,所以A左边都是A的左子孙，A右边的都是A的右子孙；递归地处理即可。
```

### 后序序列+中序序列
给出后序序列和中序序列，得到唯一二叉树

## 遍历二叉树
时间复杂度：O(n)；

空间复杂度：树的深度，最坏情况为O(n)；



### 先序遍历
* 若二叉树为NULL,空操作，否则：
    1. 访问根结点；
    1. 先序遍历左子树；
    1. 先序遍历右子树；

```c
//递归
void PreOrderTraverse(LinkBtree btree, visit callback) {
    if (!btree) return;

    callback(btree->data);
    PreOrderTraverse(btree->lchild, callback);
    PreOrderTraverse(btree->rchild, callback);
}

//非递归
void PreOrderTraverseNonRec(LinkBtree btree, visit callback) {
    BNode* stack[128];
    int top = 0;
    BNode *p = NULL;

    p = btree;
    while (top > 0 || p) {
        if (p) {
            callback(p->data);   //pre-order
            stack[top++] = p;
            p = p->lchild;
        } else {
            p = stack[--top];
            p = p->rchild;
        }
    }
}
```

### 中序遍历
```c
//递归
void InOrderTraverse(LinkBtree btree, visit callback) {
    if (!btree) return;
    
    InOrderTraverse(btree->lchild, callback);
    callback(btree->data);
    InOrderTraverse(btree->rchild, callback);
}

//非递归
void PreOrderTraverseNonRec(LinkBtree btree, visit callback) {
    BNode* stack[128];
    int top = 0;
    BNode *p = NULL;

    p = btree;
    while (top > 0 || p) {
        if (p) {
            stack[top++] = p;
            p = p->lchild;
        } else {
            p = stack[--top];
            callback(p->data);   //in-order
            p = p->rchild;
        }
    }
}
```

### 后序遍历
```c
//递归
void PostOrderTraverse(LinkBtree btree, visit callback) {
    if (!btree) return;
    
    PostOrderTraverse(btree->lchild, callback);
    PostOrderTraverse(btree->rchild, callback);
    callback(btree->data);
}

//非递归
void PostOrderTraverseNonRec(LinkBtree btree, visit callback) {
    BNode* stack[128];
    int top = 0;
    BNode *p = NULL;
    BNode *pre = NULL;

    p = btree;
    while (top > 0 || p) {
        if (p) {
            stack[top++] = p;
            p = p->lchild;
        } else {
            p = stack[top - 1];
            if (!p->rchild || p->rchild == pre) {
                callback(p->data);
                pre = p;
                top--;
                p = NULL;
            } else {
                p = p->rchild;
            }
        }
    }
}
```

## 线索二叉树
时间复杂度：O(n)

若所用二叉树需要经常遍历或查找结点在遍历所得线性序列中的前驱和后继，应采用线索链表作存储结构。

将二叉树按照某种遍历方式储存起来（线索化），使其能像二叉链表一样知道一个结点便可遍历其余结点，提高效率。

在有n个结点的二叉链表中必定有n+1个空链域，可以利用这些空链域存放结点的前驱和后继的信息。
```c
+--------+-------+-------+-------+-------+
|lchild  |Ltag   |data   |RTag   |rchild |
+--------+-------+-------+-------+-------+

LTag = 0 (lchild指示结点的左孩子)
     = 1 (lchild指示结点的前驱)

RTag = 0 (lchild指示结点的右孩子)
     = 1 (lchild指示结点的后继)
```
以这种结点结构构成的二叉链表，叫做**线索链表**。
其中指向结点前驱和后继的指针叫做**线索**。
加上线索的二叉树称之为**线索二叉树**。
对二叉树以某种次序遍历使其变为线索二叉树的过程叫做**线索化**。

```c
typedef enum PointerTag {Link, Thread}; //Link指针，Thread线索
typedef struct BiThrNode {
    TElemType data;
    struct BiThrNode *lchild, *rchild;
    PointerTag ltag, rtag;
} BiThrNode *BiThrTree;
```
### 中序线索树
1. 树中所有叶子结点的右链是线索，右链指示结点的后继；
1.结点的后继是遍历其右子树时访问的第一个结点，即右子树中最左下的结点；
1. 结点的前驱是遍历左子树时最后访问的一个结点，即左子树中最右下的结点； 

```c
//在二叉树的线索链表上添加一个头结点，令其lchild域的指针指向二叉树的根节点，其rchild域的指针指向中序遍历时访问的最后一个结点。

void InOrderThreading(BiThrTree *Thrt, BiThrTree T) {
    //中序遍历二叉树T,并将其中序线索化，Thrt指向头结点
    *Thrt = (BiThrTree)malloc(sizeof(BiThrNode));
    if(!*Thrt) exit(1);
    (*Thrt)->ltag = Link;
    (*Thrt)->rtag = Thread;
    if(!T) (*Thrt)->lchild = (*Thrt);
    else {
        (*Thrt)->lchild = T;
        pre = Thrt;
        InThreading(T);
        pre->rchild = (*Thrt);
        pre->rtag = Thread;
        (*Thrt)->rchild = pre;
    }
}

void InThreading(BiThrTree p) {
    if(p) {
        InThreading(p->lchild);
        if(!p->lchild) {
            p->ltag = Thread;
            p->lchild = pre;
        }
        if(!pre->rchild) {
            pre->rtag = Thread;
            pre->rchild = p;
        }
        pre = p;
        InThreading(p->rchild);
    }
}

void InOrderTranverse_Thr(BiThrTree tree, visit callback) {
    p = tree->lchild;   //令p指向根节点
    while (p != tree) {
        while (p->ltag == Link) p = p->lchild;
        callback(p->data);
        while (p->rtag == Thread && p->rchild != tree) {
            p = p->rchild;
            visit(p->data);
        }
        p = p->rchild;
    }
}


```




### 后序线索树
TODO

## 二叉树的其他操作
### 二叉树的高
递归地表示为max{左子树高度, 右子树高度} + 1。

### 统计二叉树叶子结点个数
遍历二叉树，callback访问结点是否是叶子结点，是的话总数加一。

## 树和森林
### 树的存储结构
1. 双亲表示法：易于表示双亲，遍历寻找孩子；
1. 孩子表示法：易于表示孩子，遍历寻找双亲；
1. **孩子兄弟表示法/二叉树表示法**


#### 双亲表示法
以一组**连续空间（数组）**存储树的结点，同时在每个结点中附设一个指示器指示其双亲结点在链表中的位置。

优点：
* 常量时间内求双亲

缺点：
* 求结点的孩子需要遍历表
```c
#define MAX_TREE_SIZE 100
typedef struct PTNode {
    TElemType data;
    int parent; //双亲位置index
} PTNode;

typedef struct {
    PTNode nodes[MAX_TREE_SIZE];
    int root_pos, num;   //根的位置和结点数
}
```

#### 孩子表示法
使用**多重链表**，即每个结点有多个指针域，其中每个指针指向一棵子树的根节点。易于找到孩子，不易找到双亲。

纯链表可以采用两种结点格式：
1. 结点大小都相同，指针个数等于树的度。则在一棵有n个结点度为k的树中必有nk-(n-1)个空链域，缺点是浪费空间。
1. 结点大小不确定，设置一个字段degree用来确定该结点的孩子个数，缺点是操作不方便。

第三种方法是把每个结点的孩子结点排列起来，看成线性表，且以单链表作存储结构，则n个结点有n个孩子链表。n个头指针又组成一个线性表（采用顺序表）。

```c
第三种：
typedef struct CTNode { //孩子结点
    int child;
    struct CTNode *next;
} *ChildPtr;

typedef struct {
    TElemType data;
    ChildPtr first_child;
} CTBox;

typedef struct {
    CTBox nodes[MAX_TREE_SIZE];
    int root_pos, num;
}

```

#### 孩子兄弟表示法
以二叉链表作树的存储结构。又称二叉树表示法/二叉链表表示法。

**链表中结点的两个链域分别指向该结点的第一个孩子结点和下一个兄弟结点。**
```c
typedef struct CSNode {
    ElemType data;
    struct CSNode *first_child; //第一个孩子
    struct CSNode *nextsibling; //第一个兄弟
} CSNode, *CSTree;
```

1. 访问结点x的第i个结点：
```c
int index = 1;  //若以0计数则修改为0
node = x->first_child;

while(index < i && node) {
    node = node->nextsibling;
    index++;
}

if(node) {
    visit(node);
}else{
    return;
}
```

### 森林与二叉树的转换
由于二叉树和树都可以用二叉链表作为存储结构，则以**二叉链表**作为媒介可以导出树与二叉树之间的对应关系。
#### 树转换为二叉树
一棵树对应的二叉树，其右子树为空。

#### 二叉树转换为树

#### 森林转换为二叉树

#### 二叉树转换为森林

### 树和森林的遍历
#### 树的遍历
1. 先根（次序）遍历：二叉树的先序遍历
1. 后根（次序）遍历：二叉树的中序遍历

#### 森林的遍历
1. 先序遍历森林：二叉树的先序遍历
1. 中序遍历森林：二叉树的中序遍历

## 树与等价问题
TODO




## 哈夫曼树
又称最优二叉树，是一类带权路径长度最短的树。

哈夫曼树中没有度为1的结点。则一棵有n个叶子结点的哈夫曼树一共有2*n-1个结点，可以存储在一个大小为2*n-1的一维数组中。

```c
typedef char** HuffmanCode;
typedef struct {
    unsigned int weight;
    unsigned int parent, lchild, rchild;
} HTNode, *HuffmanTree;
```

### 基本概念
1. 路径：从树中一个结点到另一个结点之间的分支构成这两个结点之间的路径。
1. 路径长度：路径上的分支数目。
1. 树的路径长度：从树根到每一结点的路径长度之和。
1. 结点的权：结点的权重？
1. 结点的带权路径长度：从该结点到树根之间的路径长度与结点上的权的乘积。
1. 树的带权路径长度：树中所有**叶子结点**的带权路径长度之和。记作：WPL=w1l1+w2l2+...
1. 最优二叉树/哈夫曼树：使得有n个叶子结点的二叉树的带权路径长度最小的二叉树。

### 哈夫曼编码
若要设计长短不等的编码，则必须是任一个字符的编码都不是另一个字符的编码的前缀，这种编码称作**前缀编码**。
可以利用二叉树来设计二进制的前缀编码。









