---
tags:
  - 竞赛
  - SCPC
  - C++
---

# 从 C 到 C++:竞赛编程入门速查

> 面向:会 C 基础语法(变量、数组、函数、指针、结构体),准备用 C++ 打算法竞赛(洛谷 / Codeforces / AtCoder 等)。
> 竞赛用的 C++ 只是 C++ 的一小部分:**C 的语法 + STL(标准模板库)+ 少量新语法**。类继承、模板元编程、异常处理等高级特性竞赛里基本用不到,先不用学。
> 好消息:绝大多数 C 代码在 C++ 里可以直接编译运行,过渡很平滑。

---

## 0. 环境与编译

- 编译命令(平时练习和 OJ 基本一致):

```bash
g++ -std=c++17 -O2 main.cpp -o main
```

- 推荐标准:**C++17**(洛谷、Codeforces GNU++17、AtCoder 均支持),结构化绑定等新特性可用。
- 学习时先备好一套"最小模板",见 [第 7 节](#7-一个基础代码模板)。

---

## 1. 基础语法变化

### 1.1 头文件与命名空间

```cpp
#include <bits/stdc++.h>   // 万能头文件,包含几乎所有标准库(GCC 专有,所有主流 OJ 可用)
using namespace std;       // 竞赛必写,省去 std:: 前缀
```

> 想写标准代码也可以逐个 `#include <iostream>`、`#include <vector>` 等,但竞赛直接用万能头。

### 1.2 bool 类型

C++ 中 `bool` 是真正的类型,只有 `true` / `false`。C 里习惯用 `int` 表示真假,C++ 里直接用 `bool` 更清晰。

### 1.3 变量声明位置随意 + auto

```cpp
for (int i = 0; i < n; i++) { ... }   // for 内声明变量(C99 也支持,但 C++ 中更自然)

auto x = 1LL;          // auto 自动推导类型:此处 x 是 long long
auto it = s.begin();   // 迭代器类型又长又丑,用 auto 省事
```

### 1.4 引用(重点)

引用是变量的"别名",不拷贝数据。**传参首选引用,尤其是大对象(如 string、vector)**,否则每次调用都整体拷贝一份,非常慢。

```cpp
void myswap(int &a, int &b) {   // 引用传参,函数内修改直接影响外面
    int t = a; a = b; b = t;
}

int x = 1, y = 2;
myswap(x, y);   // x = 2, y = 1

const string &s   // 只读引用:不想被修改时加 const,避免拷贝
```

### 1.5 函数重载与默认参数

```cpp
int max2(int a, int b)      { return a > b ? a : b; }
int max3(int a, int b, int c) { return max2(max2(a, b), c); }  // 同名重载也可以

void print(int x, char end = '\n') {   // 默认参数
    cout << x << end;
}
print(5);        // 等价 print(5, '\n')
print(5, ' ');
```

### 1.6 范围 for

```cpp
vector<int> v = {1, 2, 3};
for (int x : v) cout << x << ' ';      // 遍历每个元素(拷贝)
for (int &x : v) x *= 2;               // 用引用才能修改元素
for (auto &x : v) x++;                 // 常用写法
```

### 1.7 struct 增强

C++ 的 struct 里可以直接写成员函数和构造函数,而且类型名直接用,不用写 `struct` 关键字:

```cpp
struct Point {
    int x, y;
    Point(int a, int b) : x(a), y(b) {}   // 构造函数
};

Point p(1, 2);          // C 里要写 struct Point p; 这里不用
```

### 1.8 lambda 表达式(匿名函数)

```cpp
auto cmp = [](int a, int b) { return a > b; };
sort(v.begin(), v.end(), cmp);
```

配合 sort 非常常用,见 [第 5 节](#5-algorithm-库核心武器)。

---

## 2. string:替代 char 数组

竞赛中最常用的字符串工具,告别 `strcpy/strcmp/strcat`:

```cpp
string s = "abc";
s += "def";                 // 拼接 => "abcdef"
s.push_back('g');           // 追加单个字符
int len = s.length();       // 或 s.size()

string t = s.substr(1, 3);  // 子串:从下标 1 开始取 3 个 => "bcd"
int pos = s.find("cd");     // 查找子串,找不到返回 string::npos

if (s == t) ...             // 直接 == < > 比较,不用 strcmp
reverse(s.begin(), s.end()); // 翻转

// 数字与字符串互转
string num = to_string(12345);
int a = stoi(num);            // 转 int,还有 stoll(转 long long)
```

遍历:`for (char c : s)`,需要改字符时用 `for (char &c : s)`。

> 注意:`substr` 是 O(n) 拷贝;`s[i]` 直接访问字符。索引越界是未定义行为,访问前确认 `i < s.size()`。

---

## 3. 输入输出

```cpp
ios::sync_with_stdio(false);   // 关闭与 C 标准输入输出的同步,让 cin/cout 提速
cin.tie(nullptr);              // 解除 cin 与 cout 的绑定
```

- **用 `'\n'` 而不是 `endl`**:`endl` 会强制刷新缓冲区,大量输出时慢很多。
- 关闭同步后,**不要混用** `cin/cout` 和 `scanf/printf`(输出顺序会乱);想用 printf 就用 `scanf/printf` 全家桶。
- 数据量在 10^6 级别以内,关闭同步后的 `cin/cout` 完全够用;超大输入(如 10^7 整数)再考虑 `scanf/printf` 或手写快读。
- `cout` 输出浮点小数位:`cout << fixed << setprecision(6) << x;`(或 `printf("%.6f", x)`)。

```cpp
string s;
getline(cin, s);   // 读一整行(含空格)
// 坑:前面刚 cin >> x 时,缓冲区还剩一个换行符,getline 会读到空串
// 解决:先 cin.ignore(); 再 getline
```

---

## 4. STL 容器(核心中的核心)

### 4.1 pair:打包两个值

```cpp
pair<int, int> p = {1, 2};
p.first;   // 1
p.second;  // 2
p = make_pair(3, 4);

auto [a, b] = p;   // C++17 结构化绑定,拆开 pair

// sort 默认先比 first,再比 second
vector<pair<int, int>> v = {{2, 1}, {1, 5}, {1, 3}};
sort(v.begin(), v.end());   // => (1,3) (1,5) (2,1)
```

### 4.2 vector:动态数组(代替 C 的手动 malloc 数组)

```cpp
vector<int> v;                 // 空数组
vector<int> v2(10, 0);         // 10 个 0
vector<int> v3 = {1, 2, 3};    // 初始化列表

v.push_back(x);                // 末尾加元素,均摊 O(1)
v.pop_back();                  // 删末尾
v.size(); v.empty();           // 大小 / 是否为空
v.front(); v.back();           // 首 / 尾元素
v[i];                          // 随机访问,和数组一样快
v.clear();                     // 清空
v.resize(n);                   // 调整大小

sort(v.begin(), v.end());      // 排序

// 二维:存图 / 动态规划常用
vector<vector<int>> g(n + 1);          // 邻接表:g[u] 存 u 能到达的点
g[u].push_back(v);

vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));  // 二维 DP 表
```

> 竞赛里**固定大小的数组仍然常用**:`const int N = 1e5 + 5; int a[N];` 且开在全局(全局数组在静态区,可以开很大;局部大数组会爆栈)。vector 胜在灵活、有 size()、能直接排序。

### 4.3 stack / queue / deque

```cpp
stack<int> st;  st.push(x);  st.top();  st.pop();
queue<int> q;   q.push(x);   q.front(); q.pop();
deque<int> dq;  dq.push_back(x); dq.push_front(x);
                dq.pop_back(); dq.pop_front(); dq[0];  // deque 还能随机访问
```

> **大坑:pop() 不返回值!** 要先 `top()` / `front()` 取出来,再 `pop()`。

### 4.4 priority_queue:堆

```cpp
priority_queue<int> pq;                                   // 大根堆(默认最大在顶)
priority_queue<int, vector<int>, greater<int>> pq2;       // 小根堆

// 存 pair 的小根堆(按 first 比较):Dijkstra 必备
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq3;

pq.push(x);   // O(log n)
pq.top();     // 取堆顶(最大/最小),不删除
pq.pop();     // 删堆顶
```

用途:贪心取最值、Dijkstra 最短路、动态维护第 k 大等。

### 4.5 set / map:有序,自带排序与去重

```cpp
set<int> s;            // 有序集合,自动去重,O(log n)
s.insert(x);
s.erase(x);            // 按值删除
if (s.count(x)) ...    // 判断是否存在
for (int x : s) ...    // 从小到大遍历

map<string, int> m;    // "字典":key 有序
m["apple"] = 3;        // 直接下标访问,不存在会自动插入
if (m.count("apple")) ...   // 判断 key 是否存在
m.erase("apple");
```

- `multiset` 允许重复元素;删一个重复值要用 `s.erase(s.find(x))`(按迭代器删),`s.erase(x)` 会删掉所有等于 x 的元素。
- 需要 O(1) 平均查询、不关心顺序时,用 `unordered_map` / `unordered_set`(哈希表)。
- 很多场景数组 + sort 比 set 更快,能用 vector + sort 就尽量不用 set。

### 4.6 bitset:二进制位集(选学)

```cpp
bitset<1000> b;        // 1000 位
b.set(3);              // 第 3 位置 1
b.test(3);             // 查第 3 位是否为 1
b.count();             // 1 的个数
b = b << 2;            // 整体移位
```

用途:标记数组压缩空间、超大二进制数、筛法优化。

---

## 5. algorithm 库:核心武器

头文件已包含在万能头里。**排序和二分是竞赛最高频操作**。

```cpp
// ---- 排序 O(n log n) ----
sort(v.begin(), v.end());                       // 从小到大
sort(v.begin(), v.end(), greater<int>());       // 从大到小
sort(a, a + n);                                 // 普通数组也可以

// ---- 自定义排序规则 ----
bool cmp(int a, int b) { return a > b; }        // 从大到小
sort(v.begin(), v.end(), cmp);

// 结构体排序:按分数降序,同分按 id 升序
struct Stu { int id, score; };
sort(v.begin(), v.end(), [](const Stu &x, const Stu &y) {
    if (x.score != y.score) return x.score > y.score;
    return x.id < y.id;
});

// ---- 二分(要求数组已有序)O(log n) ----
int pos = lower_bound(v.begin(), v.end(), x) - v.begin();  // 第一个 >= x 的下标
int pos = upper_bound(v.begin(), v.end(), x) - v.begin();  // 第一个 > x 的下标
// 个数 = upper_bound - lower_bound(等于 x 的元素个数)
if (binary_search(v.begin(), v.end(), x)) ...              // 是否存在
```

> **比较函数铁律**:必须满足"严格弱序",即 `a == b` 时 `cmp(a, b)` 必须返回 `false`,否则 sort 会越界崩溃。标准写法是"想排在前面就返回 true,平局返回 false"。

其他常用算法:

```cpp
unique(v.begin(), v.end());   // 去重:配合 sort 使用,返回新末尾迭代器
v.erase(unique(v.begin(), v.end()), v.end());  // 真正删除重复元素

reverse(v.begin(), v.end());  // 翻转
fill(a, a + n, 0);            // 批量赋值
min(a, b); max(a, b);         // 最值
min({a, b, c});               // 多个值
swap(a, b);                   // 交换(比手写三行省事)

__gcd(a, b);                  // 最大公约数(GCC 内置,不用手写欧几里得)

max_element(v.begin(), v.end());   // 返回最大元素位置的迭代器
count(v.begin(), v.end(), x);      // 统计等于 x 的个数
find(v.begin(), v.end(), x);       // 找第一个等于 x 的位置
accumulate(v.begin(), v.end(), 0LL);  // 求和,注意用 0LL 防溢出

next_permutation(v.begin(), v.end());  // 全排列:生成字典序下一个排列
// 用法:先 sort,再 do { ... } while (next_permutation(...));
```

---

## 6. 常用常量、类型与技巧

```cpp
using ll = long long;   // 竞赛习惯:几乎所有整型都用 long long,防溢出
// (老代码里常见 typedef long long ll;)

const int INF = 0x3f3f3f3f;             // 常用"无穷大" ≈ 1.06e9
const ll INFLL = 0x3f3f3f3f3f3f3f3f;    // long long 版
// 好处:memset(a, 0x3f, sizeof a) 能把 int 数组全填成 INF,且两倍 INF 不溢出

const int MOD = 1e9 + 7;   // 取模常用素数

const double EPS = 1e-9;   // 浮点比较:abs(a - b) < EPS 视为相等
```

- `int` 上限约 2.1e9,`long long` 上限约 9.2e18。中间结果可能爆 int 的(如乘法)一定用 long long。
- 浮点用 `double`,不要用 `float`。
- `NULL` → 用 `nullptr`。

---

## 7. 一个基础代码模板

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

const int INF = 0x3f3f3f3f;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // 在这里写代码

    return 0;
}
```

---

## 8. 常见坑速查

1. **pop() 不返回值**:先 `top()`/`front()` 再 `pop()`。
2. **比较函数平局必须返回 false**,否则 sort 崩溃。
3. **cin 与 scanf 混用**(关闭同步后)会乱序,选一套用。
4. **getline 前有残留换行**:用 `cin.ignore()` 吃掉。
5. **用 `'\n'` 不用 `endl`**。
6. **局部大数组爆栈**:大数组开全局,或开成 vector/static。
7. **数组越界 / string 越界是未定义行为**:OJ 上表现为玄学 WA/RE。
8. **乘法溢出**:先转 long long 再乘,或 `1LL * a * b`。
9. **multiset 里删一个重复值**:`s.erase(s.find(x))`,而不是 `s.erase(x)`。
10. **浮点判断相等用 eps**,不要用 `==`。

---

## 9. C → C++ 速查对照表

| C 的写法 | C++ 竞赛替代 |
| --- | --- |
| `char s[100]` | `string s` |
| `int a[100000]` + malloc | `vector<int> a` 或全局静态数组 |
| `malloc` / `free` | 直接用容器,几乎不再手动管理内存 |
| `qsort` + 函数指针 | `sort` + lambda / cmp 函数 |
| 手写二分 | `lower_bound` / `upper_bound` / `binary_search` |
| 手写栈 / 队列 / 堆 | `stack` / `queue` / `priority_queue` |
| 手写链表 | 数组模拟链表或 `list`(竞赛很少用 list) |
| `strcmp` / `strcat` / `strcpy` | `==` / `+` / `=`(string 直接运算) |
| `printf("%d", x)` | `cout << x`(关闭同步后也很快) |
| `NULL` | `nullptr` |
| 结构体传指针 | 引用 `&` |

---

## 10. 学习路线建议

以"会写 C 基础"为起点,按顺序推进:

1. **语法过渡**(1~2 周):把上面的 string / vector / sort / 二分 用熟,刷洛谷入门题(P1001~P2000 中的普及- 题)。
2. **基础算法**:模拟与暴力枚举、前缀和与差分、双指针、贪心、栈和队列应用。
3. **搜索与图论入门**:DFS / BFS、最短路(Dijkstra + 堆)。
4. **动态规划**:背包、线性 DP、区间 DP。
5. **进阶**:并查集、线段树/树状数组、图论进阶、数论基础。

推荐 OJ:

- **洛谷**:中文题面、题解丰富,新手首选;按难度标签刷题。
- **Codeforces**:全球最大,打 Div.4 / Div.3 上分;AtCoder 的 ABC 也适合入门。
- 刷题原则:先自己想,卡住看题解,然后把解法**自己默写一遍**,再隔几天重做。

核心心法:**能调库就调库**(STL 又对又快),把精力花在算法思路上,而不是重复造轮子。

> 相关笔记:[[C语言学习]] [[训练笔记-C++基础与预处理]]
