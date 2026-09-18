---
tags:
  - 竞赛
  - SCPC
---

## 矩阵染色
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
[[matrix_teleport_solution]]


auto it = lower_bound(a.begin(), a.end(), b);

```cpp
#include <algorithm>
vector<int> a = {1, 3, 5, 7, 9};
// 返回迭代器
auto it = lower_bound(a.begin(), a.end(), 5);
int idx = it - a.begin();  // 转下标

// 如果没找到，it == a.end()
```
---
```cpp
int lowerBound(vector<int>& a, int x) {
    int l = 0, r = a.size();  // r 开区间
    while (l < r) {
        int mid = (l + r) / 2;
        if (a[mid] >= x) r = mid;
        else l = mid + 1;
    }
    return l;  // 返回下标
}
```

## aya字符串1183L re!!!

[j == n - 1];
这是一个很经典的 **C++ 紧凑写法（小技巧）**，用来在一行代码里同时处理“空格”和“换行”。

我来给你拆开讲一下：

### 原理拆解

`" \n"` 在 C++ 里是一个**字符串字面量**，本质上就是一个字符数组：

- `" \n"[0]` 是空格 `' '`
- `" \n"[1]` 是换行符 `'\n'`
- `" \n"[2]` 是字符串结尾的 `\0`

而 `j == n - 1` 是一个**布尔表达式**，在 C++ 里：

- 如果 `j != n - 1`（不是行末），结果为 `false`，相当于整数 **`0`**
- 如果 `j == n - 1`（是行末），结果为 `true`，相当于整数 **`1`**

所以：

- 当 `j != n - 1` 时，访问 `" \n"[0]`，输出**空格**
- 当 `j == n - 1` 时，访问 `" \n"[1]`，输出**换行**

### 等价写法

它完全等价于下面这段更直观的代码：

```cpp
if (j == n - 1) {
    cout << '\n';
} else {
    cout << ' ';
}
```

set 容器用下标
```cpp
p=*(next(s.begin(), mid));
```

# [26NAILOONG只删一个] 前后缀数组trick

我的one piece在哪里SPA_分类：_点赞：0浏览：3_发布时间：_1 个月前

如果暴力枚举要删除的元素，每次重新计算整个数组的最大公约数，时间复杂度为 O(n2)，效率过低，无法通过数据范围限制。

观察性质：删掉第 i 个数字后，整个数组的 gcd 等于前缀区间 [1,i−1] 的最大公约数 与 后缀区间 [i+1,n] 的最大公约数二者再求 gcd。

据此预处理两个数组：

- pre：前缀 gcd 数组，pre[i] 表示区间 [1,i] 的最大公约数
- suf：后缀 gcd 数组，suf[i] 表示区间 [i,n] 的最大公约数

两个数组均可通过递推在 O(n) 时间内求得。  
之后依次枚举每个下标 i，计算 gcd(prei−1​, sufi+1​)，所有结果中的最大值即为本题答案。

前后缀数组是经典常用技巧，遇到**删除单个元素、删除一段区间**求整体最值的题目时，大多可以套用该思路。

```cpp
for(int i = 1; i <= n; i++) node[i].reserve(10);
```
### 预分配n个为10的空间



10484
```cpp
#include<bits/stdc++.h>

using namespace std;

typedef long long ll;
int main(){
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int n,c;cin >> n >>c ;
    vector<int> a(n);
    for(int i=0;i<n;i++) cin >> a[i];
    sort(a.begin(), a.end());
    ll ans = 0;
    for(int i = 0; i < n; i++){
        ll tar= a[i] - c;
        auto left = lower_bound(a.begin(), a.end(), tar);
        auto right = upper_bound(a.begin(), a.end(), tar);
        ans += right - left;
    }
    cout << ans << endl;
    return 0;

}
```
把数组分为小于目标tar的左和右,右减左就是目标个数

10076

```cpp
#include <bits/stdc++.h>

using namespace std;

  

int main() {

    ios::sync_with_stdio(false);

    cin.tie(0);

    cout.tie(0);

    int n, m;

    cin >> n >> m;

    vector<int> a(n);

  

    for (int i = 0; i < n; ++i) {

        cin >> a[i];

    }

    if(m<10){

        cout << m << endl;

        return 0;

    }

    sort(a.begin(), a.end());

    int last = a.back();

    a.pop_back();

    int cap = m - 10;

    vector<int> dp(cap + 1, 0);

    for (int p : a) {

        for (int c = cap; c >= p; --c) {

            dp[c] = max(dp[c], dp[c - p] + p);

        }

    }

    int ans = m - dp[cap] - last;

    cout << ans << '\n';

    return 0;

}
```

<[区间和恰好为 k - SCPC西南科技大学算法竞赛实验室OJ](http://scpc.fun/problem/10478)>
![[Pasted image 20260908230758.png]]


tuple<int, int, int> 三元
大于三元自己定义把

<[家谱 - SCPC西南科技大学算法竞赛实验室OJ](http://scpc.fun/problem/10111)>

```cpp
#include<iostream>

#include<map>

using namespace std;

map<string,string> f;

string find(string a){

    if(f[a] == a) return a;

    else return f[a] = find(f[a]);

}

void unin(string a,string b){

    string x = find(a),y = find(b);

    if(x != y)f[x] = y;

}

  

int main(){

    ios::sync_with_stdio(false);

    cin.tie(0);cout.tie(0);

    string s,ss;char c;

    while(cin>>c){

        if(c == '$')break;

        cin>>s;

       if(c == '#') {

           ss = s;

           if(f[s].empty())f[s] = s;

       }

        if(c == '+'){

            if(f[s].empty())f[s] = s;

            unin(s,ss);

        }

        if(c == '?'){

            cout<<s<<" "<<find(s)<<"\n";

        }

  

    }

    return 0;

}
```
干学长 学并查
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> p;

int find(int x) {
    if (p[x] != x)
        p[x] = find(p[x]);
    return p[x];
}

void unite(int x, int y) {
    int rX = find(x);
    int rY = find(y);
    if (rX != rY) p[rY] = rX;
}

int main(){
    ios::sync_with_stdio(false);
    cin.tie(0);cout.tie(0);
    int N,M;cin>>N>>M;
    p.resize(N+1);
    for(int i=1;i<N;i++) p[i]=i;
    while (M--) {
        int Z,X,Y;
        cin>>Z>>X>>Y;
        if (Z == 1) unite(X, Y);
        else {
            if (find(X) == find(Y)) cout << "Y\n";
            else cout << "N\n";
        }
    }
    return 0;
}

```

ST表

#作业 俄罗斯方块
```cpp
#include <bits/stdc++.h>

using namespace std;

  

int main() {

    ios::sync_with_stdio(false);

    cin.tie(0);

    cout.tie(0);

    vector<vector<int>> a(15, vector<int>(10));

    for (int i = 0; i < 15; i++) {

        for (int j = 0; j < 10; j++)

            cin >> a[i][j];

    }

    vector<vector<int>> pattern(4, vector<int>(4));

    for (int i = 0; i < 4; i++) {

        for (int j = 0; j < 4; j++)

            cin >> pattern[i][j];

    }

    int start = 0;

    cin >> start;

    start--;

    int down = 15;

    while (down > 0 && down--) {

        bool flag = true;

        for (int i = 0; i < 4; i++) {

            for (int j = 0; j < 4; j++) {

                if (pattern[i][j] == 1) {

                    if ((i + down) > 14 || (start + j) >= 10 ||

                        (start + j) < 0 || a[i + down][j + start] == 1) {

                        flag = false;

                        break;

                    }

                }

            }

            if (!flag)

                break;

        }

        if (flag)

            break;

    }

    // cout << down << "\n";

    for (int i = 0; i < 4; i++) {

        for (int j = 0; j < 4; j++) {

            if (pattern[i][j] == 1)

                a[down + i][start + j] = 1;

        }

    }

    for (int i = 0; i < 15; i++) {

        for (int j = 0; j < 10; j++) {

            cout << a[i][j] << "\n "[j != 9];

        }

    }

    return 0;

}

```


```cpp
begin() //返回指向容器第一个元素的迭代器
end() //返回指向容器最后一个元素之后的迭代器（尾后迭代器
```
它们共同定义了容器的遍历范围：`[begin, end)`
begin()+索引决定第几元素
```cpp
	cout<<fixed<<setprecision(3) << double << '\n';
```
    C++ 输出格式控制,小数点后保留 **3 位小数**。