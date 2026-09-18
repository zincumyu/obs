---
tags:
  - 竞赛
  - SCPC
  - 题解
---

# 矩阵传送与步行问题 - 最简解题方案

## 问题描述

给定一个 n×m 的彩色矩阵，每格有一个颜色编号。从起点到终点可以执行两种操作：
- **传送**：移动到矩阵中任意一个颜色与当前格子相同的格子，不消耗体力
- **步行**：移动到上下左右相邻的格子，消耗 1 点体力

求从起点 (sx,sy) 到终点 (tx,ty) 所需的最小体力。

**数据范围**：1 ≤ n,m ≤ 1000，1 ≤ A[i][j] ≤ 10^6

---

## 核心算法思路

采用 **SPFA（Shortest Path Faster Algorithm）+ 颜色列表优化**：

1. **普通队列 BFS**：每次从队头取出一个格子处理
2. **颜色传送优化**：用 `last_teleport[c]` 记录上次传送颜色 c 时使用的距离，只有当前距离更小时才重新传送（避免重复无效传送，同时保证正确性）
3. **步行**：上下左右四个方向，距离 +1
4. **提前存好每种颜色的格子列表**：避免每次传送时遍历整个矩阵找同色格子

**为什么不用"每种颜色只传送一次"？**
因为普通 FIFO 队列不保证按距离从小到大出队，可能出现：先用较大的距离激活了颜色传送，之后更小距离到达同色格子时却无法再次传送，导致 WA。

---

## 完整C代码

```c
#include <stdio.h>
#include <stdlib.h>

int a[1005][1005];
int sp[1005][1005];
int qx[5000005], qy[5000005];
char inq[1005][1005];

// 存每种颜色的格子列表
typedef struct { int *x, *y, cnt; } List;
List cs[1000005];
int last_teleport[1000005]; // 上次传送该颜色时用的最小距离

int main() {
    int n, m, sx, sy, tx, ty;
    scanf("%d%d", &n, &m);

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++) {
            scanf("%d", &a[i][j]);
            int c = a[i][j];
            cs[c].x = realloc(cs[c].x, sizeof(int) * (cs[c].cnt + 1));
            cs[c].y = realloc(cs[c].y, sizeof(int) * (cs[c].cnt + 1));
            cs[c].x[cs[c].cnt] = i;
            cs[c].y[cs[c].cnt] = j;
            cs[c].cnt++;
        }

    scanf("%d%d%d%d", &sx, &sy, &tx, &ty);

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            sp[i][j] = 1000000000;
    for (int i = 0; i < 1000005; i++)
        last_teleport[i] = 1000000000;

    int head = 0, tail = 0;
    qx[tail] = sx; qy[tail] = sy; tail++;
    sp[sx][sy] = 0;
    inq[sx][sy] = 1;

    int dx[] = {-1, 1, 0, 0};
    int dy[] = {0, 0, -1, 1};

    while (head < tail) {
        int x = qx[head];
        int y = qy[head];
        head++;
        inq[x][y] = 0;

        int c = a[x][y];
        // 传送：只有当前距离比上次传送该颜色时更小，才重新传送
        if (sp[x][y] < last_teleport[c]) {
            last_teleport[c] = sp[x][y];
            for (int k = 0; k < cs[c].cnt; k++) {
                int i = cs[c].x[k], j = cs[c].y[k];
                if (sp[i][j] > sp[x][y]) {
                    sp[i][j] = sp[x][y];
                    if (!inq[i][j]) { qx[tail] = i; qy[tail] = j; tail++; inq[i][j] = 1; }
                }
            }
        }

        // 步行
        for (int d = 0; d < 4; d++) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx < 1 || nx > n || ny < 1 || ny > m) continue;
            if (sp[nx][ny] > sp[x][y] + 1) {
                sp[nx][ny] = sp[x][y] + 1;
                if (!inq[nx][ny]) { qx[tail] = nx; qy[tail] = ny; tail++; inq[nx][ny] = 1; }
            }
        }
    }

    printf("%d\n", sp[tx][ty]);
    return 0;
}
```

---

## 逐行代码讲解

### 变量说明

| 变量 | 作用 |
|------|------|
| `a[1005][1005]` | 存储矩阵中每个格子的颜色 |
| `sp[1005][1005]` | `sp[i][j]` 表示起点到 (i,j) 的最小体力消耗 |
| `qx[], qy[]` | 普通队列，存储待处理格子的坐标 |
| `inq[1005][1005]` | 标记格子是否在队列中，防止重复入队 |
| `List cs[1000005]` | 每种颜色对应的格子列表（通讯录） |
| `last_teleport[1000005]` | 上次传送该颜色时使用的距离 |

### 预处理阶段：建立颜色"通讯录"

```c
for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++) {
        scanf("%d", &a[i][j]);
        int c = a[i][j];
        // 把坐标(i,j)加入颜色c的列表末尾
        cs[c].x = realloc(cs[c].x, sizeof(int) * (cs[c].cnt + 1));
        cs[c].y = realloc(cs[c].y, sizeof(int) * (cs[c].cnt + 1));
        cs[c].x[cs[c].cnt] = i;
        cs[c].y[cs[c].cnt] = j;
        cs[c].cnt++;
    }
```

把同颜色的格子做成"通讯录"。比如颜色5有3个格子，`cs[5]` 里就存着 `(1,2), (4,7), (8,3)`，下次传送颜色5直接查通讯录，不用再翻遍整个矩阵找了。

### 初始化

```c
for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++)
        sp[i][j] = 1000000000;   // 初始距离都是无穷大

for (int i = 0; i < 1000005; i++)
    last_teleport[i] = 1000000000;  // 每种颜色还没传送过

int head = 0, tail = 0;
qx[tail] = sx; qy[tail] = sy; tail++;  // 起点入队
sp[sx][sy] = 0;      // 起点距离 = 0
inq[sx][sy] = 1;     // 标记起点在队列里
```

### 核心操作1：传送（防止WA的关键）

```c
int c = a[x][y];
// 如果用"这个距离"传送颜色c，比"上次传送c用的距离"还要小 → 值得再传一次！
if (sp[x][y] < last_teleport[c]) {
    last_teleport[c] = sp[x][y];   // 记录这次传送用的距离
    // 把所有颜色c的格子，全部免费更新距离
    for (int k = 0; k < cs[c].cnt; k++) {
        int i = cs[c].x[k], j = cs[c].y[k];
        if (sp[i][j] > sp[x][y]) {
            sp[i][j] = sp[x][y];     // 更新：免费传送过来！
            if (!inq[i][j]) {        // 不在队列里才入队
                qx[tail] = i; qy[tail] = j; tail++;
                inq[i][j] = 1;
            }
        }
    }
}
```

**为什么这样写不会WA？**
- ❌ 错误写法：`if (!col[c])` → 颜色c一辈子只传送一次，不管后来有没有更小的距离
- ✅ 正确写法：`if (sp[x][y] < last_teleport[c])` → 只要能带着更小的距离来，就允许再传送一次

举个例子：
```
第一次：颜色5用 sp=5 传送 → last_teleport[5] = 5
后来：有另一个颜色5的格子，sp=2 出队了 → 2 < 5 → 允许再传一次！
```

### 核心操作2：步行（上下左右）

```c
int dx[] = {-1, 1, 0, 0};
int dy[] = {0, 0, -1, 1};
// 4个方向：上 下 左 右

for (int d = 0; d < 4; d++) {
    int nx = x + dx[d], ny = y + dy[d];
    if (nx < 1 || nx > n || ny < 1 || ny > m) continue;  // 越界跳过
    
    if (sp[nx][ny] > sp[x][y] + 1) {   // 走过去更省力？
        sp[nx][ny] = sp[x][y] + 1;     // 更新：步行消耗1
        if (!inq[nx][ny]) {
            qx[tail] = nx; qy[tail] = ny; tail++;
            inq[nx][ny] = 1;
        }
    }
}
```

---

## 模拟运行示例

**输入：**
```
3 3
1 2 3
2 3 1
3 1 2
1 1 3 3
```

矩阵：
```
(1,1)=1  (1,2)=2  (1,3)=3
(2,1)=2  (2,2)=3  (2,3)=1
(3,1)=3  (3,2)=1  (3,3)=2
```

### 第1步：初始
```
sp[1][1]=0，其他=∞
队列 = [(1,1)]
```

### 第2步：出队(1,1) sp=0 颜色1
```
传送颜色1：0 < ∞，传送！
  → 颜色1的格子：(1,1) (2,3) (3,2)
  → sp全部=0，全部入队
  → last_teleport[1] = 0

步行：
  → (2,1) sp=1，入队
  → (1,2) sp=1，入队

队列 = [(2,3), (3,2), (2,1), (1,2)]
```

### 第3步：出队(2,3) sp=0 颜色1
```
传送：sp=0 < last_teleport[1]=0？不，跳过（省时间！）

步行：
  → (1,3) sp=1，入队
  → (3,3) sp=1 ← 终点就是它！
  → (2,2) sp=1，入队

队列 = [(3,2), (2,1), (1,2), (1,3), (3,3), (2,2)]
```

### 后续步骤
继续出队处理，但 `sp[3][3]` 已经是1，后续无法再优化。

**答案 = 1** ✓

---

## 复杂度分析

- **时间复杂度**：O(nm)，每个格子入队次数有限（SPFA特性），每种颜色的传送次数也有限（距离只能越来越小，最小是0）
- **空间复杂度**：O(nm + C)，C是颜色种类数

**性能优化对比**：

| 版本 | 传送找同色格子的方式 | 单次传送耗时 | 最坏总耗时 |
|------|---------------------|-------------|-----------|
| 朴素版 | 遍历整个矩阵 | O(nm) = 100万 | 10亿+（超时） |
| 本代码 | 查提前做好的列表 | O(该颜色格子数) | 100万（AC） |

---

## 本地编译运行

```bash
# 编译（-O2 优化速度）
gcc solution.c -o solution -O2

# 运行（手动输入）
./solution

# 或者用文件输入输出
./solution < input.txt > output.txt
```

**input.txt 示例：**
```
3 3
1 2 3
2 3 1
3 1 2
1 1 3 3
```

**output.txt 期望：**
```
1
```
