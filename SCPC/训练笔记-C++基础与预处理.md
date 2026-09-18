---
tags:
  - 竞赛
  - SCPC
  - C++
---

# 训练笔记:C++ 基础 · sort · 预处理 · gcd/lcm

> 训练简介(密码:2026)
> 新增内容:gcd 和 lcm、C++ 语法(string、pair、sort、cmp)、字符串、排序、预处理、普通思维
>
> 配套阅读:`C到C++竞赛编程入门.md`(本篇是该训练的具体落地笔记,重合内容这里不再展开)

---

## 0. 本次训练学什么

| 模块 | 要求 |
| --- | --- |
| C++ 基础 | 头文件、cin/cout、pair、string、指针和引用(了解即可) |
| sort 与 cmp | 熟练掌握,尤其 cmp 的写法 |
| 预处理 | 理解"先算好存下来,查询时直接输出"的思想 |
| gcd 和 lcm | 新增算法,会写会用 |
| 题目类型 | 字符串、排序、普通思维题 |

---

## 1. C++ 基础

C++ 可以看作 C 语言的升级版,包含 C 语言所有内容。做题只需要学习方便做题的新增内容:**输入输出、字符串、指针和引用**。

### 1.1 头文件

```cpp
#include <iostream>        // 输入输出
#include <algorithm>       // 算法库(sort 等)
#include <bits/stdc++.h>   // 万能头文件,包含几乎所有常用头文件,写了这个就不用写别的了
using namespace std;       // 做题加上就行
```

### 1.2 输入输出 cin / cout

```cpp
int a;
cin >> a;            // 相当于 scanf("%d", &a)
cout << a << endl;   // endl 相当于换行 '\n'
```

**提速(建议固定写)**:

```cpp
std::ios::sync_with_stdio(false);   // 关闭输入输出同步流,使 cin/cout 效率与 scanf/printf 相当
cin.tie(0);
cout.tie(0);
```

> 🔔 关闭同步流后**不能再使用 scanf 和 printf**,两者混用会出问题,选一套用到底。
> 🔔 `endl` 速度慢(会强制刷新缓冲区),建议统一用 `cout << '\n';`

**保留小数位数**:

```cpp
cout << fixed << setprecision(n);   // 之后输出的值均保留 n 位小数
// 例:cout << fixed << setprecision(2) << 3.14159;  => 3.14
```

### 1.3 pair 类型

相当于写好的二元结构体。`pair<int, int>` 表示两个 int 打包;第一个值用 `first`,第二个值用 `second`。

```cpp
pair<int, int> p;
p = make_pair(1, 2);   // 也可以 p = {1, 2};
cout << p.first << ' ' << p.second << '\n';  // 1 2
p.first = 5;           // 将第一个值改为 5
cout << p.first;       // 5
```

典型用途:

- 存坐标 `pair<int, int> p = {x, y};`
- 存"权值 + 编号"再排序:`vector<pair<int, int>>`
- `sort` 默认按 first 优先、second 其次排序,配合 pair 排序非常方便

### 1.4 字符串 string

新增 string 类型,**非常重要**,处理字符串比 char 数组好用得多。

```cpp
string s = "hello";

s.size();             // 或 s.length(),求长度
s[i];                 // 像数组一样访问第 i 个字符

s += " world";        // 拼接,直接 +,不用 strcat
if (s == t) ...       // 直接比较,不用 strcmp
if (s < t) ...        // 按字典序比较

s.substr(pos, len);   // 从 pos 开始取 len 个字符的子串
s.find(t);            // 找 t 在 s 中的位置,找不到返回 string::npos

reverse(s.begin(), s.end());   // 翻转字符串
sort(s.begin(), s.end());      // 字符排序(需要 <algorithm>)

stoi(s);              // 字符串转 int(stoll 转 long long)
to_string(x);         // 数字转字符串
```

输入输出:

```cpp
string s;
cin >> s;             // 读一个单词,遇到空格/换行停止
getline(cin, s);      // 读一整行,可含空格
cout << s << '\n';    // 直接输出整个字符串
```

> 坑:前面 `cin >> x` 之后残留一个换行符,接着 `getline` 会读到空串;先 `cin.ignore();` 再 `getline`。

### 1.5 指针和引用(了解即可)

```cpp
void f(int &x) { x++; }   // 引用:传引用,函数内修改直接影响外面
int a = 1;
f(a);                     // a 变成 2,不用传指针
```

觉得难理解可以先跳过,做题多了慢慢就明白了。现阶段记住:传参写 `int &x` 可以省去指针解引用的麻烦。

---

## 2. sort 与 cmp(重点掌握)

```cpp
#include <algorithm>

int a[100] = {...};
sort(a, a + n);                     // 数组排序:从小到大,区间 [a, a+n)
sort(a + 1, a + n + 1);             // 下标从 1 开始存时的写法

vector<int> v;
sort(v.begin(), v.end());           // vector 排序
```

### 2.1 从大到小:cmp 函数

```cpp
bool cmp(int a, int b) {
    return a > b;        // a 排在 b 前面 => 大的在前
}
sort(a, a + n, cmp);
```

### 2.2 结构体 / pair 多关键字排序

```cpp
struct Stu { int id, score; };

bool cmp(const Stu &x, const Stu &y) {
    if (x.score != y.score) return x.score > y.score;  // 分数高的在前
    return x.id < y.id;                                // 同分按 id 升序
}
sort(a, a + n, cmp);
```

也可以直接写 lambda(匿名函数):

```cpp
sort(a, a + n, [](const Stu &x, const Stu &y) {
    if (x.score != y.score) return x.score > y.score;
    return x.id < y.id;
});
```

### 2.3 铁律

> ⚠️ **不要写 `a <= b`,只能写 `a < b`。**
> 比较函数必须满足:两个元素相等时返回 false。写了 `<=` 会让 sort 认为"a 排在 b 前"和"b 排在 a 前"同时成立,导致越界崩溃(RE)。
> 口诀:**想让 a 排前面就返回 true,平局必须返回 false。**

---

## 3. 预处理

有些题要求输出的内容是**重复的**。例如:给你 n 个数,进行 q 次查询,每次查询输入一个数字 x,输出前 x 个数的和。如果每次查询都重新计算,有 TLE 的风险。

解法:**先进行一次预处理,把每个答案都存下来,查询时直接输出。**

```cpp
int a[10010];
int pre[10010];   // 预处理数组:pre[i] = 前 i 个数的和
int main()
{
    int n, q, x;
    scanf("%d", &n);
    for (int i = 1; i <= n; i ++) scanf("%d", &a[i]);          // 输入数组
    for (int i = 1; i <= n; i ++) pre[i] = pre[i - 1] + a[i];  // 预处理部分:O(n) 一次算完
    scanf("%d", &q);
    while (q --)
    {
        scanf("%d", &x);              // 输入要查询的位置
        printf("%d\n", pre[x]);       // O(1) 直接输出预处理的答案
    }
}
```

### 3.1 前缀和的进阶用法:区间和

`pre[i]` 存"前 i 个数的和"之后,任意区间 `[l, r]` 的和可以 O(1) 求出:

```cpp
// 区间 [l, r] 的和 = pre[r] - pre[l - 1]
printf("%d\n", pre[r] - pre[l - 1]);
```

### 3.2 预处理思想的其他例子

- 阶乘表:`f[i] = f[i-1] * i % MOD`,查询阶乘直接输出
- 斐波那契:`fib[i] = fib[i-1] + fib[i-2]`,先算到最大 n 再回答查询
- 答案打表:把 n 个询问的答案全部算好存数组,统一输出

核心:**一次性 O(n)(或 O(n log n)) 算出所有可能被问到的答案,之后每个查询 O(1) 回答。**

---

## 4. gcd 和 lcm(新增内容)

### 4.1 定义

- **gcd(a, b)**:a 和 b 的最大公约数(Greatest Common Divisor)
- **lcm(a, b)**:a 和 b 的最小公倍数(Least Common Multiple)

### 4.2 gcd 的求法:辗转相除法(欧几里得算法)

原理:`gcd(a, b) = gcd(b, a % b)`,直到 b == 0 时答案为 a。

```cpp
// 手写版(递归)
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}

// 手写版(循环)
int gcd(int a, int b) {
    while (b != 0) {
        int t = a % b;
        a = b;
        b = t;
    }
    return a;
}

// 直接调库:GCC 内置函数,不用手写
int g = __gcd(a, b);
```

### 4.3 lcm 的求法

公式:`lcm(a, b) = a * b / gcd(a, b)`

```cpp
int lcm(int a, int b) {
    return a / gcd(a, b) * b;   // 先除后乘,防止 a*b 中途溢出
}

// 注意数据范围:如果 a、b 可能很大,返回值和参数都改成 long long
long long lcm(long long a, long long b) {
    return a / __gcd(a, b) * b;
}
```

### 4.4 常见题型

- 直接问 gcd / lcm 的值
- 分数约分:分子分母同除 `__gcd(分子, 分母)`
- 判断两个数互质:`__gcd(a, b) == 1`
- 周期相遇类问题 → 求 lcm
- 多组询问 → 用函数反复调用即可,不用每次重写

---

## 5. 字符串、排序与普通思维题

### 5.1 字符串题套路

- 逐字符处理:`for (int i = 0; i < s.size(); i++)` 或 `for (char c : s)`
- 判断回文:`reverse` 一份比较,或双指针两头比
- 统计字符出现次数:开 `int cnt[26]` 数组,`cnt[s[i] - 'a']++`
- 字典序比较直接用 `<`
- 数字 ↔ 字符串互转:`to_string` / `stoi`

### 5.2 排序题套路

- 结构体多关键字排序(成绩、编号)
- 排序后再处理(排序后相邻元素比较、去重等)
- 排序 + 贪心:很多题先 sort 就能看出做法

### 5.3 普通思维题心法

1. **先模拟**:拿样例手动走一遍,搞清楚"规则到底是什么"
2. **会暴力**:数据小就先写最朴素的三重循环,拿到部分分再想优化
3. **找规律**:小数据打表,观察输出规律
4. **分类讨论**:把情况列全,每种情况单独处理
5. **想清楚再写**:代码写一半发现思路不对,浪费的时间比想的时间多得多

---

## 6. 训练自测清单

学完本次训练,逐条确认:

- [ ] 能默写带加速的头文件 + 主函数模板
- [ ] 会用 `pair` 存二元数据,知道 `first` / `second`
- [ ] `string` 的 `size / += / == / substr / find` 都亲手敲过
- [ ] 能写出"从大到小"的 cmp,并说清为什么**不能写 `<=`**
- [ ] 能独立写出结构体多关键字排序
- [ ] 能默写前缀和预处理 + 区间和 `pre[r] - pre[l-1]`
- [ ] 能默写递归版 gcd,会用 `__gcd`
- [ ] 会求 lcm,且知道要**先除后乘**防溢出
- [ ] 遇到多组询问的题,第一反应是"能不能预处理"

---

## 附:本训练通用模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    // 在这里写代码

    return 0;
}
```

> 相关笔记:[[C到C++竞赛编程入门]] [[C语言学习]]
