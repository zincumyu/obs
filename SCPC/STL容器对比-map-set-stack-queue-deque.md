---
tags:
  - 竞赛
  - SCPC
  - C++
  - STL
---

# STL 容器对比速查:map / set / stack / queue / deque

> 配套阅读:[[C到C++竞赛编程入门]] 第 4 节、[[训练笔记-C++基础与预处理]]
> 全部在 `#include <bits/stdc++.h>` 里,不用单独记头文件。

## ⚡ 速查表(先看这张)

| 容器              | 大白话              | 是否自动排序  | 允许重复   | 随机访问          | 增删查复杂度          |
| --------------- | ---------------- | ------- | ------ | ------------- | --------------- |
| `stack`         | 一摞盘子,后进先出        | 否       | —      | 否(只能看顶)       | O(1)            |
| `queue`         | 排队,先进先出          | 否       | —      | 否(只能看头尾)      | O(1)            |
| `deque`         | 双开门队列,两头都能进出     | 否       | 是      | ✅ `d[i]` O(1) | 头尾 O(1),中间 O(n) |
| `set`           | 自动排序 + 去重的袋子     | ✅ 从小到大  | 否      | 否             | O(log n)        |
| `multiset`      | set 的"允许重复"版     | ✅       | 是      | 否             | O(log n)        |
| `map`           | 按 key 排序的字典(电话簿) | ✅ 按 key | key 唯一 | 否             | O(log n)        |
| `unordered_map` | 哈希字典,只求查得快       | 否       | key 唯一 | 否             | 平均 O(1)         |

> **一句话怎么选**:后进先出用 stack;先进先出用 queue;两头都要用 deque;要"自动排序+去重"用 set;要"自动排序但可重复"用 multiset;要"键值对且 key 有序"用 map;要"键值对只求 O(1) 查询"用 unordered_map。

---

## 1. stack:栈(一摞盘子)

只能从"顶"进出,**后进先出(LIFO)**。

```cpp
stack<int> st;
st.push(x);      // 入栈
st.top();        // 看栈顶(不删除)
st.pop();        // 弹栈(删除栈顶)
st.empty();      // 是否为空
st.size();       // 元素个数
```

> ⚠️ `pop()` **不返回值**,要先 `top()` 取出再 `pop()`。
> 用途:括号匹配、表达式求值、DFS 模拟、单调栈。

## 2. queue:队列(排队)

队尾进、队头出,**先进先出(FIFO)**。

```cpp
queue<int> q;
q.push(x);       // 入队(队尾)
q.front();       // 看队头(不删除)
q.back();        // 看队尾
q.pop();         // 出队(删队头)
q.empty(); q.size();
```

> ⚠️ 同样,`pop()` 不返回值,先 `front()` 再 `pop()`。
> 用途:BFS、任务排队。

## 3. deque:双端队列(双开门)

`stack` 和 `queue` 的**功能超集**:头尾都能进出,还能随机访问。`stack`/`queue` 底层默认就是它。

```cpp
deque<int> dq;
dq.push_back(x);   dq.push_front(x);   // 尾/头插入
dq.pop_back();     dq.pop_front();     // 尾/头删除
dq.front(); dq.back();                 // 看头/尾
dq[i];                                  // 随机访问,和 vector 一样
```

> 用途:滑动窗口(双端队列维护最值)、需要两头操作的序列。竞赛里滑动窗口最值非常常用。

## 4. set:自动排序 + 去重

底层是**红黑树**,元素**从小到大自动排列**,重复元素会被忽略。没有下标,靠迭代器遍历。

```cpp
set<int> s;
s.insert(x);          // 插入 O(log n),重复的插不进去
s.erase(x);           // 删除 O(log n)
s.count(x);           // 是否存在(返回 0 或 1)
s.find(x);            // 返回迭代器,找不到返回 s.end()
s.lower_bound(x);     // 第一个 >= x 的位置
for (int x : s) ...   // 从小到大遍历
```

> ⚠️ set 里的元素**不能修改**——想改只能 `erase` 旧的再 `insert` 新的。
> ⚠️ 迭代器只能 `++`/`--` 一步步走,不能 `it + 1`(红黑树不是连续内存)。

**multiset:允许重复的 set**

```cpp
multiset<int> ms;
ms.erase(ms.find(x));   // 只删一个 x
ms.erase(x);            // ⚠️ 删掉所有等于 x 的元素!
```

## 5. map:键值对字典(电话簿)

底层也是红黑树,**按 key 自动排序**,每个 key 唯一,对应一个 value。

```cpp
map<string, int> m;
m["apple"] = 3;           // 直接下标赋值,不存在会自动插入
m["apple"] = 5;           // 覆盖旧值
m.count("apple");         // key 是否存在(0 或 1),判断存在用它
m.erase("apple");         // 删除
m.find("apple");          // 迭代器,找不到返回 m.end()
for (auto &p : m)         // 按 key 从小到大遍历
    cout << p.first << ' ' << p.second << '\n';
```

> ⚠️ **大坑**:`m[key]` 访问**不存在**的 key 会自动插入一个默认值(0/空串),统计/判断时别用 `[]`;要判断 key 是否存在用 `count()` 或 `find()`。
> ⚠️ key **不能改**,value 可以改。
> 只需 O(1) 平均查询、不关心顺序 → 用 `unordered_map`(哈希表)。

## 6. 五容器全面对比表

| 维度       | stack  | queue  | deque    | set      | multiset | map            | unordered_map |
| -------- | ------ | ------ | -------- | -------- | -------- | -------------- | ------------- |
| 底层结构     | deque  | deque  | 分段数组     | 红黑树      | 红黑树      | 红黑树            | 哈希表           |
| 进出规则     | 只出顶    | 头出尾进   | 两头自由     | —        | —        | —              | —             |
| 自动排序     | ❌      | ❌      | ❌        | ✅        | ✅        | ✅(按 key)       | ❌             |
| 去重       | —      | —      | ❌        | ✅        | ❌        | key 唯一         | key 唯一        |
| 随机访问     | ❌      | ❌      | ✅ `d[i]` | ❌        | ❌        | ❌ `m[key]` 算查找 | ❌             |
| 插入/删除/查找 | O(1)   | O(1)   | 头尾 O(1)  | O(log n) | O(log n) | O(log n)       | 平均 O(1)       |
| 能否遍历     | ❌ 无迭代器 | ❌ 无迭代器 | ✅        | ✅ 有序     | ✅ 有序     | ✅ 按 key 有序     | ✅ 无序          |

> 💡 **能用 vector + sort 就不一定用 set**:set 每次插入都 O(log n) 且常数大;如果数据是"一次性读入、排序后使用",`vector` + `sort` 往往更快。需要**动态维护有序**(边插入边查询)时才用 set/multiset。

## 7. 常见坑速查

1. `pop()` 不返回值 → 先 `top()`/`front()` 再 `pop()`(stack、queue、priority_queue 都一样)。
2. `map` 用 `m[key]` 判断存在会**意外插入** → 用 `count()` / `find()`。
3. `multiset` 删一个重复元素 → `erase(find(x))`,`erase(x)` 是全删。
4. set/map 的迭代器只能 `++`/`--`,没有 `it + k`。
5. set 元素、map 的 key **不可修改**。
6. stack/queue **不能遍历**,想要能遍历的双端队列用 deque。
7. `unordered_map` 最坏情况(被卡哈希)会退化成 O(n),竞赛卡常时留意。
8. 遍历 set/map 时删元素会导致迭代器失效 → 记下要删的再删,或边遍历边 `it = s.erase(it)`。

## 8. 自测清单

- [ ] 能不看笔记说出 5 个容器各自"一句话适用场景"
- [ ] 会写 stack 模拟括号匹配
- [ ] 会写 queue 做 BFS
- [ ] 会用 deque 维护滑动窗口最值
- [ ] 会用 set 去重排序;知道 multiset 怎么只删一个
- [ ] 会用 map 统计词频,且判断存在用 `count()` 而不是 `[]`
- [ ] 能解释 set/map 为什么是 O(log n),unordered_map 为什么平均 O(1)

## 相关笔记

- [[C到C++竞赛编程入门]] — 4.3~4.5 节有各容器基本用法
- [[训练笔记-C++基础与预处理]] — 训练配套笔记
- [[C语言学习]] — C 语言手写栈/队列的对照
