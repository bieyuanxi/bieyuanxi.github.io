+++
title = "图"
date = "2023-11-22"
+++
# 图

## 基本概念
* 通用概念
    1. 顶点：图中的数据元素
    1. 权：与图的弧或者边有关的数
    1. 网：带权的图
    1. 子图
    1. 度

* 无向图
    1. 边：无序对(u,v)
    1. 无向图：不区分弧头弧尾的图
    1. 完全图：有n*(n-1)/2条边的无向图
    1. 邻接点：无向图中有边关系的两个顶点互称邻接点
    1. 回路/环
    1. 简单路径：序列中顶点不重复出现的路径
    1. 连通：无向图，从顶点v到顶点u有路径
    1. 连通图：无向图中任意两个顶点是连通的
    1. 连通分量：无向图中的极大连通子图
    1. 极小连通子图：含有图中全部顶点，但只有n-1条边
    1. 生成树：一个连通图的生成树是一个极小连通子图


* 有向图
    1. 弧<u, v>，指从u到v的一条弧
    1. 弧尾：u
    1. 弧头：v
    1. 有向图：区分弧头弧尾的图
    1. 有向完全图：有n*(n-1)条边的有向图
    1. 有向无环图（DAG图）：无环的有向图
    1. 入度/出度
    1. 强连通图：有向图中任意两个顶点双向连通
    1. 强连通分量：有向图中的极大强连通子图
    1. 有向树：一个有向图中恰有一个顶点的入度为0,其余顶点的入度均为1
    1. 生成森林：一个有向图的生成森林由若干棵有向树组成，含有图中全部顶点，但只有足以构成若干棵不相交的有向树的弧。连通分量的生成树组成非连通图的生成森林。




## 图的存储结构
**数组表示法（邻接矩阵）**、**邻接表**、十字链表（有向图的链式存储结构）、邻接多重表（无向图的链式存储结构）

### 邻接矩阵
用两个数组分别存储数据元素（顶点）信息和数据元素之间的关系（边或弧）的信息。

存储： n个结点空间，n^2个弧信息（无向图可以压缩至n*(n-1)/2）

适用于弧很多的情况，弧少时浪费空间

#### 建立邻接矩阵
时间复杂度：O(n^2 + n*e)    //n^2是初始化，n * e是因为对输入的每条边都要寻找对应的顶点

#### 度的计算
1. 入度：矩阵第j列之和
1. 出度：矩阵第i行之和

```c
typedef enum {DG, DN, UDG, UDN} GraphType;

typedef struct {
    int info;   //顶点信息
} Cell;

typedef struct {
    GraphType type;
    Cell vexs[MAX_VERTEX_NUM];  //可以用指针
    int arcs[MAX_VERTEX_NUM][MAX_VERTEX_NUM];   //无向图存1、0,有向图存权
    int vex_num;
} Graph;
```

### 邻接表
用数组存顶点信息，用链表存弧信息。

适用于边稀疏的情况。

缺点：
* 对于无向图，一个边会对应两个弧结点，分别在两个链表中，操作不便（只用一个的话得遍历全部链表）。
* 对于有向图，不能同时快速得出入度和出度。（可以通过建立逆邻接表加速）
#### 建立邻接表
时间复杂度：O(n + e)    //输入的顶点信息即为顶点编号
          O(n * e)      // 查找对应的顶点

#### 度的计算
1. 入度：需要遍历每个结点的出度（可以通过建立逆邻接表加速）
1. 出度：遍历该结点对应的链表皆可

```c
typedef struct {
    int info;   //顶点信息
    ArcNode *first_arc; //第一条弧的指针
} Cell;

typedef struct _ArcNode {
    int val;   //权
    int adjvex; //该弧指向的顶点
    struct _ArcNode *next;
} ArcNode;

typedef struct {
    Cell vexs[];
    int vex_num;
} Graph;
```


### 十字链表
**有向图**的另一种链式存储结构。相当于有向图的邻接表+逆邻接表。

由顶点结点和弧结点组成。
每个顶点结点都存储该结点对应的第一个弧头弧尾的指针；弧结点保存了其弧头弧尾指针的下一个。

```c
typedef struct _ArcNode {
    int val;
    int vex_tail;   //弧尾对应的结点
    int vex_head;   //弧头对应的结点
    struct _ArcNode tail_next;  //弧尾相同的下一个
    struct _ArcNode head_next;  //弧头相同的下一个
} ArcNode;

typedef struct {
    int data;
    ArcNode *first_out; //出度，从该结点出发的弧
    ArcNode *first_in;  //入度，以该结点结束的弧
} Cell;

typedef struct {
    Cell *nodes;    //顶点表
    int vex_num;
} Graph;
```

#### 建立十字链表
```c
//0. input:  <u,v>, arc_val
//1.生成一个弧结点，补充弧头、弧尾
//2.将相应顶点表对应的first_out、first_in指针存到弧结点
//3.将对应first_out、first_in更新成该弧结点地址
ArcNode *p = (ArcNode*)malloc(sizeof(ArcNode));
p->val = arc_val;
p->vex_tail = u;
p->vex_head = v;

int i = locate(graph, u);
int j = locate(graph, v);
p->tail_next = graph.nodes[i].first_out;
p->head_next = graph.nodes[j].first_in;

graph.nodes[i].first_out = p;
graph.nodes[j].first_in = p;
```


### 邻接多重表
是**无向图**的另一种链式存储结构。
```c
typedef struct _ArcNode {
    int mark;   //标记
    int ivex, jvex; //边对应的顶点
    struct _ArcNode ilink, jlink;   //边对应的下一个边的指针
    int val；   //边对应的权值
} ArchNode;

typedef struct {
    int data;   //结点信息
    ArcNode *first_edge;    //结点对应的第一条边的地址
} Cell;

typedef struct {
    Cell *nodes;
    int vex_num;
} Graph;
```

## 图的遍历
从图中某一顶点出发访遍图中其余顶点，且使每一个顶点仅被访问一次。

图的遍历算法是求解图的连通性问题、拓扑排序和求关键路径等算法的基础。

图的搜索需要设置一个visit数组以避免同一顶点被访问多次。
深度优先搜索（DFS）、广度优先搜索（BFS）
### DFS
是树的先根遍历的推广。

```c
typedef void (*visit)(int v);

int visited[];

void DFSTraverse(Graph g, visit callback) {
    // 设置visited为0
    for(int i = 0; i < g.vex_num;++i) {
        visited[i] = 0;
    }
    for(int vex = 0; vex < g.vex_num;++vex) {
        if(!visited[vex]) DFS(g, vex, callback);
    }
}

void DFS(Graph g, int vex, visit callback) {
    visited[vex] = 1;
    callback(vex);
    
    for(int vex = first_adj_vex(g, vex); vex >= 0;vex = next_adj_vex(g, vex)) {
        if(!visited[vex]) DFS(g, vex, callback);
    }
}
```
#### 时间复杂度分析
初始化visited数组，时间复杂度O(n)

1. 邻接矩阵：查找一个顶点的边/弧，时间复杂度o(n)，对于整个图时间复杂度：O(n^2)
1. 邻接表：遍历n个链表可以直接得到全部弧，时间复杂度(e)，加上初始化visited数组，为O(n+e)

### BFS
类似于树的层次遍历。

```c
typedef void (*visit)(int v);

int visited[];

void BFSTraverse(Graph g, visit callback) {
    // 设置visited为0
    for(int i = 0; i < g.vex_num;++i) visited[i] = 0;
    Queue queue; init_queue(&queue);
    int vex;
    for (int i = 0; i < g.vex_num;++i) {
        if(visited[i]) continue;
        visited[i] = 1; callback(vex);  //先访问，再push
        push(queue, i);
        
        while (!empty(queue)) {
            pop(queue, &vex);
            for(int i = first_adj_vex(g, vex); i>= 0; i = next_adj_vex(g, vex)) {
                if(visited[i]) continue;
                visited[i] = 1; callback(vex);
                push(queue, i); 
            }
        }
    }

}
```

## 图的连通性问题
### 无向图的连通分量和生成树
极小连通子图=连通图的一棵生成树（深度优先生成树、广度优先生成树）
TODO
### 有向图的强连通分量
TODO

## 最小生成树MST
研究在n个结点间建立边，使得构造出的连通网代价最小，得到最小代价生成树。

### Prim算法
时间复杂度：O(n^2)，只与结点个数有关，适用于求**边稠密**的网的最小生成树。

空间复杂度：O(n)

思路：将图中结点划分为已被联结的集合U和尚未联结的集合V，每次选择V中边代价最小的结点划入U，一共循环n次。需要设置一个辅助数组close_edge[vex_num]记录V中结点与U中结点最近的边（0xFFFFFFFF表示不存在边或弧，0x0表示顶点已被加入集合U）。

```c
#define MAX_COST (0xFFFFFFFF)
typedef struct {
    int arc_tail;   //弧尾或边
    unsigned int low_cost;   //最小代价（0xFFFFFFFF表示不存在边或弧，0x0表示顶点已被加入集合U）
} CloseEdge;
void Prim(Graph g) {
    CloseEdge *close_edge = (CloseEdge*)malloc(g.vex_num * sizeof(CloseEdge));
    for(int i = 0; i < g.vex_num; ++i) {
        close_edge[i] = MAX_COST;
    }

    int p = 0;
    int p_next = 0;
    unsigned int low_cost = MAX_COST;
    close_edge[p].low_cost = 0; //设置为0表示加入集合U，结点0已加入
    for(int i = 1; i < g.vex_num; ++i) {    //一次找一个边，循环n-1次
        low_cost = MAX_COST;
        for(int j = 0; j < g.vex_num; ++j) {    //遍历寻找顶点p（也即集合U）最短的弧
            //if(g.arcs[p][j] == MAX_COST) continue;
            if(g.arcs[p][j].low_cost < close_edge[j]){  //更新集合U到顶点j的弧信息
                close_edge[j].arc_tail = p; //是从p出发到j
                close_edge[j].low_cost = g.arcs[p][j];
            }
            if(close_edge[j].low_cost > 0 && close_edge[j].low_cost < low_cost){
                p_next = j;
                low_cost = close_edge[j].low_cost;
            }
        }
        close_edge[p_next].low_cost = 0;    //设置加入集合U
        p = p_next;
    }
}
```
教科书：
```c
void Prim(Graph g, VertexType u) {
    int k = LocateVex(g, u);
    CloseEdge *close_edge = (CloseEdge*)malloc(g.vex_num * sizeof(CloseEdge));

    close_edge[k].low_cost = 0; //设置顶点k加入集合U
    for(int i = 0; I <g.vex_num; ++i) {
        if(k != i) {
            close_edge[i].arc_tail = k;
            close_edge[i].low_cost = g.arcs[k][i];  //初始设置顶点k的弧
        }
    }
    
    for(int i = 1; i < g.vex_num; ++i) {    //选择其余n-1个顶点
        k = min(close_edge);    //遍历选择代价最小的顶点(且代价>0)
        close_edge[k].low_cost = 0; //并入k到集合U

        for(int j = 0; j < g.vex_num; ++j) {    //遍历顶点更新到集合U的代价
            if(g.arcs[k][j] < close_edge[j].low_cost) {
                close_edge[j].arc_tail = k;
                close_edge[j].low_cost = g.arcs[k][j];
            }
        }
    }
}
```
### Kruskal算法（加边法）
时间复杂度：O(e * loge)，与顶点数无关，适用于求**边稀疏**的网的最小生成树。

思路：将各个顶点看作不同的集合，递增遍历边，判断边并合并顶点所在的不同集合，最终得到一个集合（连通图）。

1. 将边按照从小到大的顺序排序（堆排序）；
1. 每次选取代价最小的边，判断对应的两个顶点是否在同一个集合内：
    1. 若是则放弃，继续选取下一个边；
    1. 否则将两个顶点对应的集合合并；

使用并查集存储遍历过程中顶点与边之间的关系。

```c
typedef struct {
    int tail;   //弧尾
    int head;   //弧头
    int val;    //权值、代价
} Edge;
//1. 堆排序
heap_sort(heap, g.arcs);    //辅助数组1：排序好的弧

int set[] = {0, 1, 2, ...}; //辅助数组2：顶点的并查集

//2. 遍历全部弧
int cost = 0;
for(Edge e = first(heap); e >= 0; e = next(heap)) {
    if(find(e.tail) == find(e.head)) continue;
    merge(e.tail, e.head);  //合并结点，e是使用的边
    printf(e);  //do sth.
    cost += e.val;
}
```

#### 并查集示例
```c
int set_parent[];  //每一个元素对应的父节点，如果没有父节点，约定为自己（也可以舍弃index0,规定0表示没有父节点）
// 找顶点i所在的集合
int find(int i) {   //递归地寻找顶点i的父节点
    if(set_parent[i] == i) {    //若规定index=0表示无父节点：set_parent[i] == 0
        return i;
    }else {
        return find(set_parent[i]);
    }
}

int find(int i) {   //非递归地寻找顶点i的父节点，同时做并查集搜索的优化
    int p = i;
    while(set_parent[p] != p) {
        p = set_parent[p];
        set_parent[i] = p;  //并查集搜索优化
    }
    return p;
}

//合并i、j所在的集合
int merge(int i, int j) {
    int pi = find(i);
    int pj = find(j);
    if(pi != pj) set_parent[pi] = pj;
    return pj;
}
```

## 有向无环图（DAG图）及其应用
有向无环图是描述含有公共子式的表达式的有效工具。

1. 偏序：若集合X上的关系R是自反的、**反对称的**和传递的，则称R是集合X上的偏序关系。（集合中仅有部分成员之间可以比较）
1. 全序：（全体成员之间均可比较）
1. 拓扑有序：
1. 拓扑排序：由偏序定义得到拓扑有序的操作便是拓扑排序。由某个集合上的一个偏序得到该集合上的一个全序，这个操作称之为拓扑排序。
1. AOV网(Activity On Vertex Network)：用顶点表示活动，用弧表示活动间的优先关系的有向图称为顶点表示活动的网。用于流程图设计、子工程之间的次序关系。
1. AOE网(Activity On Edge Network)：用顶点表示事件，弧表示活动，权表示活动持续时间的网，带权的有向无环图。用于估算工程完成时间、哪些活动是影响工程进度的关键。
1. 关键路径：AOE网中路径长度最长的路径。

### 拓扑排序（AOV网）
AOV网不应存在有向环，首先判定网中是否存在环：对有向图构建拓扑有序序列，若网中所有顶点都在它的拓扑有序序列中，则不存在环。

拓扑排序方法：
1. 在有向图中选一个没有前驱的顶点并输出；
1. 从图中删除该顶点和所有以它为尾的弧;
1. 重复上述步骤，直至全部顶点输出（有向无环图）或图中不存在无前驱的顶点（有向图中存在环）为止。

时间复杂度：O(n + e)

空间复杂度：O(n)，设置辅助数组indegree[]，存储顶点的入度值，入度为0表示顶点没有前驱。另设一栈，度为0的顶点入栈。
```c
//采用邻接表作有向图的存储结构
int TopologicalSort(ALGraph g) {
    int indegree[g.vex_num] = {0};
    ArcNode *p = NULL;
    for(int i = 0; i < g.vex_num; ++i) {    //更新各个结点的入度：O(e)
        p = g.vexs[i].first_arc;
        while(p) {
            indegree[i]++;
            p = p->next;
        }
    }
    Stack s; init_stack(&s);
    for(int i = 0; i < g.vex_num; ++i) {    //设置栈存储没有前驱的顶点:O(n)
        if(!indegree[i]) push(&stack, i);
    }

    int i, count = 0;
    ArcNode *del;
    while(!empty(&stack)) { //O(e)
        pop(&stack, &i); count++;
        printf("%d, ", i);  //拓扑排序输出序列
        p = g.vexs[i].first_arc;
        while(p) {
            if(!(--indegree[p->adjvex])){   //入度减为0
                push(&stack, p->adjvex);
            }
            p = p->next;
        }
    }

    if(count < g.vex_num) return ERROR;
    else return OK;
}
```

### 关键路径（AOE网）
1. 最早开始时间ve(i)：max{前驱们的最早开始时间+弧长}
1. 最晚开始时间vl(i)：min{后继们的最晚开始时间-弧长}
1. 时间余量： vl(i) - ve(i)
1. 关键活动：ve(i) = vl(i)的活动

关键路径上的所有活动都是关键活动

求事件最早开始时间同拓扑排序，求最晚开始时间利用vl(i) = vl[j] - dur<i, j>

时间复杂度：O(n + e)

空间复杂度：O(n)：
1. 使用了辅助数组indegree[]；
1. 栈T存储拓扑排序的输出顺序；

```c
//求事件（弧）最早开始时间，使用辅助数组indegree[]

int vex_e[];    //最早开始时间
int vex_l[];    //最晚开始时间
// t存储拓扑排序的一个结果，用于记录结点之间的先后次序，便于找前驱
// vex_e存储最早开始时间
int _TopologicalOrder(Graph g, Stack *t) {
    int indegree[g.vex_num];
    FindIndegree(g, indegree);

    vex_e[0..g.vex_num] = 0;    //初始化最早开始时间全为0

    Stack s; init_stack(&s);
    //将入度为0的顶点push到s内
    int count = 0;
    int i, j;
    ArcNode *p;
    while(!empty(&s)) {
        pop(&s, i); push(&t, i); count++;   //按拓扑排序顺序输出
        for(ArcNode *p = g.vex[i].first_arc; p; p = p->next) {
            j = p->adjvex;  //顶点i指向顶点j
            if(p->val + vex_e[i] > vex_e[j]) {
                vex_e[j] = p->val + vex_e[i];   //最早开始时间取决于最晚(大)的那个
            }
            if(!(--indegree[j])) push(&s, j);
        }
    }

    if(count < g.vex_num) return ERR;
    return OK;
}

int criticalPath(ALGraph g) {
    Stack T; init_stack(&T);

    if(!_TopologicalOrder(g, &T)) return ERR;

    vex_l[0..g.vex_num] = vex_e[g.vex_num - 1]; //初始化最晚开始时间全为最大值

    ArcNode *p;
    int i, j;
    while(!empty(&T)) {
        pop(&T, i);
        for(p = g.vexs[i].first_arc; p; p = p->next) {
            j = p.adjvex;
            if(vex_l[j] - p->val < vex_l[i]) {
                vex_l[i] = vex_l[j] - p->val;   //最晚开始时间取决于最早(小)的那个
            }
        }
    }

    //echo vex_e[..], vex_l[..]
    return OK;
}
```


## 最短路径
一类带权有向图问题

1. 源点：路径上第一个顶点
1. 终点：路径上最后一个顶点

### 从某个源点到其余各顶点的最短路径——Dijkstra迪杰斯特拉算法

按路径长度递增的次序产生最短路径的方法，用于求解从源点v到其余各顶点v1、v2...的最短路径。

**v到vi的最短路径只可能是v直接到vi,或者经过了点vk到vi**，可以用反证法或数学归纳法证明。

时间复杂度：O(n^2)

空间复杂度：O(n)
1. 设置辅助数组dis[]，表示从源点v到vi的最短路径长度（规定弧不存在时无穷）；
1. 设置辅助数组in_set[]，表示顶点i是否已访问到。

思路：
1. 初始时，将v到各个顶点的弧赋值到dis[]；
1. 从dis中找到未访问的且距离v最短的顶点i,对于vi到各顶点vj的弧长，当且仅当dis[j] > dis[i] + arc_len<vi, vj>时，更新dis到各个顶点的距离为较小的那个;
1. 重复步骤2共n-1次。
```c
//假设使用邻接矩阵
#define INF (0xFFFFFFFF)
void dijkstra(Graph g, int start, int *path) {
    int num = g.vex_num;
    int dis[num]; // i到start的最短距离
    int in_set[num];  //已访问标记，访问到的结点不会再次选取
    for(int i = 0; i < num;++i) {
        dis[i] = g.arcs[start][i];
        path[i] = start;    // <start, i>
        in_set[i] = false;
    }
    in_set[start] = true;   //初始设置start顶点已访问
    int k;
    int min = INF;  //无穷
    for(int i = 1; i < num;++i) {   //从1开始，循环n-1次
        min = INF;
        for(int j = 0; j < num; ++j) {  //在未访问的结点集找距离start最短的顶点k 
            if(!in_set[j] && in_set[j] < min) {
                k = j; min = in_set[j];
            }
        }
        if(min == INF) break; //找不到任何可能路径，提前结束

        in_set[k] = true;   //访问顶点k，并将其放入已访问集合
        for(int j = 0; j < num;++j) {   //更新经结点k到达各顶点的最短路径
            if(g.arcs[k][j] == INF) continue;
            if(g.arcs[k][j] + dis[k] < dis[j]) {    //如果经k更近
                dis[j] = g.arcs[k][j] + dis[k];

                path[j] = k;    //偷偷记下经k到j: <k, i>
            }
        }
    }
}
```

### 每一对顶点之间的最短路径——Floyd弗洛伊德算法
ps: 可以执行n次迪杰斯特拉算法，时间复杂度为O(n^3)。弗洛伊德算法的优势是代码简单，易于实现。

也可以用floyd算法，其思想是用加点法，借助其他顶点间接表示任意两个顶点之间的最短距离。对于顶点vk,假设存在<vi, vk>, <vk, vj>, 则vi与vj之间的最短距离=min{<vi, vj>, <vi, vk> + <vk, vj>}。对于任意顶点vk（O(n)）,都可以找到以vk为中介的两个顶点（作为弧头和弧尾，O(n^2)）并获得对应的最短距离。

时间复杂度:O(n^3)

空间复杂度：O(n^2)，设置辅助数组dis[][]，用来存储当前各顶点之间的最短距离。

```c
#define INF (0xFFFFFFFF)
int dis[NUM][NUM]; //演示用，辅助数组
void Floyd(Graph g) {
    int num = g.vex_num;

    for(int i = 0; i < num; ++i) {
        for(int j = 0; j < num; ++j) {
            dis[i][j] = g.arcs[i][j];
            if(i == j) dis[i][j] = 0;  //自己到自己为0
        }
    }

    for(int k = 0; k < num; ++k) {  //对全部vk
        for(int i = 0; i < num; ++i) {  //顶点vi
            if(dis[i][k] == INF) continue;
            for(int j = 0; j < num; ++j) {  //顶点vj
                if(dis[k][j] == INF) continue;
                if(dis[i][k] + dis[k][j] < dis[i][j]) {
                    dis[i][j] = dis[i][k] + dis[k][j];
                }
            }
        }
    }
}
```



