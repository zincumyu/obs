---
tags:
  - 竞赛
  - SCPC
  - C++
---

  ```c++
   vector<pair<int,int>> b(n);
  ```

[[C到C++竞赛编程入门]]


切片 strat 和 len ->substr  stoi 是转是数字
```cpp
stoi(s.substr(st, len));
```
差分
### 核心用途：区间加法

如果要对原数组的 `[l, r]` 区间每个元素都加上 `x`

gcd = 最大公约数（Greatest Common Divisor）

round() 四舍五入取整函数

map 
for(i:list) 类python for i in list
可以用 for(auto& i :list) 解引用list里元素,不用拷贝

^= 异或 可以排除相同元素
0^0=0，  
1^0=1，  
0^1=1，  
1^1=0,

LLONG_MIN和 LLONG_MAX
INT_MAX 和 INT_MIN

```
1 <<n  -->  2^n
它等价于计算 **2 的 n 次方**
```

```
int mask = 1 << 3;   // 得到二进制 00001000

// 1. 检查第3位是否为1
if (x & mask) {
    // 第3位是1
}

// 2. 将第3位设为1
x |= mask;

// 3. 将第3位清除为0
x &= ~mask;

// 4. 翻转第3位
x ^= mask;
```

