---
tags:
  - 工具
  - Python
  - numpy
  - 深度学习
---

# numpy 常用函数速查

> 适用：读 PyTorch / D2L 代码时需要补的 numpy 手感。环境 `numpy 2.4.3`。
> 一句话定位：**numpy 里 shape 是主角**。看代码时先盯 `.shape`，再猜它在算什么。

---

## ⚡ 速查表（最常用，先看这张）

| 想做什么 | 写法 |
| --- | --- |
| 建数组 | `np.array([[1,2],[3,4]])`、`np.zeros((2,3))`、`np.ones`、`np.arange(6)`、`np.linspace(0,1,5)` |
| 看形状/类型 | `x.shape`、`x.dtype`、`x.ndim`、`x.size` |
| 变形 | `x.reshape(2,3)`、`x.reshape(-1)`（-1 自动推）、`x.T` |
| 加一维 | `x[:, None]` 或 `x[..., None]` 或 `np.expand_dims(x, 1)` |
| 降一维 | `x.squeeze()`、`x.reshape(-1)` |
| 矩阵乘 | `A @ B`（推荐）、`np.matmul(A,B)` |
| 点积/内积 | `np.dot(u, w)`（1D 是内积） |
| 爱因斯坦求和 | `np.einsum('ij,jk->ik', A, B)` |
| 沿轴归约 | `x.sum(axis=0)`、`x.mean(axis=1)`、`x.max(axis=-1)` |
| 保持维度归约 | `x.max(axis=1, keepdims=True)` ← softmax 必备 |
| 逐元素 | `x + y`、`x * y`（**不是**矩阵乘！矩阵乘是 `@`） |
| 条件取值 | `np.where(cond, a, b)` |
| 最大位置 | `x.argmax(-1)` |
| 裁剪 | `np.clip(x, 0, 1)` |
| 拼接 | `np.concatenate([a,b], axis=0)`、`np.stack([a,b])` |
| 转置指定轴 | `x.transpose(1,0,2)`、`x.swapaxes(0,2)` |
| 新随机数 | `rng = np.random.default_rng(42)`；`rng.normal(size=(2,2))` |
| 判相等（浮点） | `np.allclose(a, b)` |
| 复制 | `x.copy()` |

---

## 1. 核心心智模型

- **shape 决定一切**。`(3,)`、`(3,1)`、`(1,3)` 是三个不同的东西，它们的运算结果完全不同。
- **numpy 的操作分两类**：
  - **view（共享内存）**：`reshape`、`ravel`、`T`、切片 `a[0]`、`resize` —— 改一个另一个也变。
  - **copy（独立内存）**：`flatten`、花式索引 `a[[0,1]]`、布尔索引 `a[a>0]`、`np.copy` —— 改一个不影响另一个。
  - 实测（`np.shares_memory`）：

  | 操作 | 共享内存？ |
  | --- | --- |
  | `a.ravel()` | ✅ 是（尽量返回 view） |
  | `a.flatten()` | ❌ 否（总是 copy） |
  | `a.reshape(6)` | ✅ 是 |
  | `a.T` | ✅ 是 |
  | `a[0]`（切片） | ✅ 是 |
  | `a[[0,1]]`（花式） | ❌ 否 |
  | `a[a>2]`（布尔） | ❌ 否 |

  > 踩坑场景：`sub = a[0]` 后改 `sub`，原数组跟着变。要独立就 `.copy()`。

---

## 2. 创建与属性

```python
np.array([1,2,3])                    # 从 list
np.zeros((2,3)); np.ones((2,3))
np.full((2,3), 7); np.eye(3)
np.arange(6)          # [0 1 2 3 4 5]
np.arange(6).reshape(2,3)
np.linspace(0, 1, 5)  # 等间距 5 个点
x.astype(np.float32)  # 类型转换（DL 里最常用 float32）
```

- 默认浮点是 **float64**，深度学习里通常要 `astype(np.float32)` 或直接 `np.float32(...)`。
- `np.arange(3) + 0.5` 结果 dtype 是 float64。

---

## 3. 索引与切片

```python
a = np.arange(12).reshape(3,4)

a[1, 2]        # 第 1 行第 2 列（逗号写法，不是 a[1][2] 那样）
a[:, 1]        # 第 1 列 → shape (3,)
a[1]           # 第 1 行 → shape (4,)
a[0:2, 1:3]    # 子块

# 花式索引（结果是 copy）
a[[0, 2]]          # 取第 0、2 行
a[:, [3, 0]]       # 列换序

# 布尔索引（结果是 copy）
a[a > 5]           # 展平后的满足元素
a[a % 2 == 0] = 0  # 也可以赋值

# 省略号：中间维度全都要
c = np.zeros((2,3,4,5))
c[..., 0].shape     # (2,3,4)
c[..., None].shape  # (2,3,4,5,1)  ← 末尾加一维
c[:, None].shape    # (2,1,3,4,5)  ← 第 1 维后插一维
```

> `a[0]` 是 view，`a[[0]]` 是 copy —— 差别在"索引里有没有 list/array"。

---

## 4. 广播（Broadcasting）★ 最容易踩坑

**规则**：从**最右边**的维度开始逐个比对，两维要么相等，要么其中一个是 1，要么其中一个不存在；都不满足就报错。

```python
a = np.arange(6).reshape(2,3)   # (2,3)

a + np.arange(3)     # (2,3)+(3,)  → (2,3) ✅ 最后一维都是 3
a + np.arange(2)     # (2,3)+(2,)  → ValueError ❌
# operands could not be broadcast together with shapes (2,3) (2,)

np.random.randn(3,1) + np.random.randn(1,4)   # → (3,4) ✅
```

### 经典静默 bug：形状差一维，不报错但算错

```python
pred  = np.random.randn(5)      # (5,)    ← 少了一维
label = np.random.randn(5,1)    # (5,1)

pred - label                    # (5,5)  ⚠️ 本该是逐样本相减，却变成交叉相减
```

**DL 里的防御习惯**：
- 写完一行运算，**立刻打印 `.shape`**，或者用 assertion 卡住。
- 想逐样本对齐：`pred[:, None] - label` 或 `label.ravel()`，两边都变成 `(5,)`。

### 扩维三种等价写法

```python
x = np.arange(3)      # (3,)
x[:, None].shape              # (3,1)
np.expand_dims(x, 1).shape    # (3,1)
x.reshape(-1, 1).shape        # (3,1)

x[None, :].shape              # (1,3)  ← 注意方向
```

---

## 5. 形状操作

```python
x.reshape(2, 3)          # 变形（view 优先）
x.reshape(-1)            # 拉平，长度自动推
x.reshape(2, -1)

x.T                      # 全转置（反转所有轴）
x.transpose(1, 0, 2)     # 指定轴顺序
x.swapaxes(0, 2)         # 只换两个轴

np.concatenate([a, b], axis=0)   # 拼接（维度数不变）
np.vstack([a, b])                # 垂直拼
np.hstack([a, b])                # 水平拼
np.stack([a, b])                 # 堆叠：新增一维
np.stack([a, b], axis=1)         # 新增维插在位置 1

np.squeeze(x)            # 去掉所有长度为 1 的维度
```

实测对比（`a`、`b` 都是 `(2,3)`）：

| 写法 | 结果 shape |
| --- | --- |
| `np.vstack([a,b])` | `(4,3)` |
| `np.hstack([a,b])` | `(2,6)` |
| `np.stack([a,b])` | `(2,2,3)` |
| `np.stack([a,b], axis=1)` | `(2,2,3)` |

> `concatenate` = 在已有维度上接；`stack` = 造一个新维度。读代码时这个区别很关键。

---

## 6. 数学与归约：axis 是关键

```python
x = np.arange(6).reshape(2,3)
# [[0 1 2]
#  [3 4 5]]

x.sum()              # 15        全加
x.sum(axis=0)        # [3 5 7]   ↓ 压掉第 0 维 → 按"列"求和，shape (3,)
x.sum(axis=1)        # [3 12]    → 按"行"求和，shape (2,)
x.mean(axis=1)       # [1. 4.]
x.max(axis=-1)       # [2 5]     -1 = 最后一维
x.sum(axis=1, keepdims=True).shape   # (2,1) ← 保住维度，方便后面广播
```

其余常用：

```python
np.exp(x); np.log(x); np.sqrt(x); np.abs(x)
x ** 2;  x / x.sum()          # 归一化
np.clip(x, 0, 6)
np.round(x, 2)
np.sort(x, axis=-1)
np.argsort(x, axis=-1)
x.argmax(-1)
np.where(x > 2, x, 0)
np.unique(x)                  # 去重
np.cumsum(x); np.diff(x)
np.allclose(a, b, atol=1e-6)  # 浮点比较，别用 ==
```

**为什么 `keepdims=True` 重要**（softmax 的稳定性写法）：

```python
z = np.array([[1.,5.,3.],[7.,2.,9.]])
m = z.max(axis=1, keepdims=True)   # shape (2,1) → [3,7] 变竖着的
# 不 keepdims 的话 m 是 (2,)，z - m 会广播错，变成 (2,2) 的交叉相减
```

---

## 7. 矩阵运算：`@` / `matmul` / `dot` / `einsum`

```python
A = np.arange(6).reshape(2,3)     # (2,3)
B = np.arange(12).reshape(3,4)    # (3,4)

A @ B                 # (2,4)  ← 推荐写法，等价 np.matmul(A,B)
A.T @ A               # (3,3)
np.dot(u, w)          # 1D 向量内积 → 标量
u @ w                 # 同样
np.outer(u, w)        # 外积 → (3,3)
```

### ⚠️ `np.dot` 和 `@` 在 3D 上不一样

实测 `m (3,2,4)`、`n (3,4,5)`：

| 写法 | 结果 shape | 说明 |
| --- | --- | --- |
| `m @ n` = `np.matmul(m,n)` | `(3,2,5)` | ✅ 批量矩阵乘（batch matmul） |
| `np.dot(m, n)` | `(3,2,3,5)` | ⚠️ 是张量缩并，**不是**你想要的 |

**结论：多维一律用 `@` 或 `np.matmul`，别用 `np.dot`。**

### einsum：看懂就够了

```python
np.einsum('ij,jk->ik', A, B)      # 矩阵乘，等于 A @ B   ✅ 实测一致
np.einsum('ij->ji', A)            # 转置，等于 A.T       ✅
np.einsum('i,i->', u, w)          # 内积 → 标量         ✅ 实测 = 5
np.einsum('bij,bjk->bik', X, Y)   # 批量矩阵乘           ✅ 等于 matmul
np.einsum('ij,ij->', A, A)        # 逐元素相乘再全加 = Frobenius 内积
np.einsum('i,j->ij', u, w)        # 外积
```

读法：**箭头左边是输入下标，右边是输出下标；没出现在输出里的下标会被求和掉。**

---

## 8. 随机数（新 API）

```python
rng = np.random.default_rng(42)     # 推荐：独立的生成器，不污染全局
rng.normal(size=(2,2))              # 标准正态
rng.normal(loc=0.0, scale=1.0, size=(3,))
rng.uniform(0, 1, size=(2,3))
rng.integers(0, 10, size=3)         # 整数 [0,10)
rng.permutation(5)                  # 打乱/抽排列
rng.choice([1,2,3], size=2, replace=False)
rng.shuffle(arr)                    # 原地打乱

np.random.seed(42)                  # 老 API，读旧代码会遇到
```

> 复现性：固定 seed 是好事，但 `default_rng(42)` 和 `np.random.seed(42)` 产生的**不是同一串数**。

---

## 9. NumPy ↔ PyTorch 对照表 ★

写 PyTorch 时最容易忘的就是这个映射关系。

| 功能 | NumPy | PyTorch |
| --- | --- | --- |
| 数组/张量 | `np.array(x)` | `torch.tensor(x)` |
| 互转 | `x.numpy()` | `torch.from_numpy(a)` |
| 沿轴归约 | `a.sum(axis=1)` | `t.sum(dim=1)` ← **axis→dim** |
| 保持维度 | `keepdims=True` | `keepdim=True`（少个 s） |
| 加一维 | `x[:, None]` / `np.expand_dims` | `t.unsqueeze(1)` |
| 降一维 | `x.squeeze()` | `t.squeeze()` |
| 变形 | `x.reshape(2,3)` | `t.reshape(2,3)` / `t.view(2,3)` |
| 转置 | `x.T` | `t.T` / `t.transpose(1,0)` |
| 换轴 | `x.permute`→`x.transpose(1,0,2)` | `t.permute(1,0,2)` ← **permute 对元组** |
| 矩阵乘 | `A @ B` | `A @ B` / `torch.matmul` |
| 逐元素 | `a * b` | `a * b` |
| 爱因斯坦 | `np.einsum(...)` | `torch.einsum(...)` 写法完全一样 |
| 拼接 | `np.concatenate` | `torch.cat` |
| 堆叠 | `np.stack` | `torch.stack` |
| 广播 | ✅ 同规则 | ✅ 同规则 |
| 就地修改 | `a += 1` | 要 `t.add_(1)`（带下划线） |
| 设备 | 只有 CPU | `t.to('cuda')` |
| 求梯度 | ❌ | `t.requires_grad_(True)` |

**关键差异（读代码时会卡住的点）**：

1. **`axis` → `dim`**：这是最高频的翻译错误。numpy 里是 `axis=`，torch 里是 `dim=`。
2. **`keepdims` → `keepdim`**：torch 少一个 s。
3. **`permute` vs `transpose`**：torch 里 `transpose` 只换**两个**轴，`permute` 才能重排全部轴（numpy 里 `transpose` 就能重排全部）。numpy 的 `x.transpose(1,0,2)` 对应 torch 的 `x.permute(1,0,2)`。
4. **就地操作**：torch 里原地修改必须显式带下划线（`add_`、`mul_`），numpy 里 `a += 1` 就是就地。
5. **view 语义**：torch 的 `view` 要求内存连续，不连续会报错，得先 `.contiguous()`；numpy 的 `reshape` 会自动处理。

---

## 10. 常见坑清单

| 坑 | 说明 | 对策 |
| --- | --- | --- |
| 形状差一维不报错 | `(5,)` − `(5,1)` → `(5,5)` 静默算错 | 打印 `.shape`，或 `x[:, None]` 显式对齐 |
| `*` 不是矩阵乘 | `A * B` 是逐元素 | 矩阵乘用 `@` |
| `np.dot` 用在多维 | 结果 shape 和 `@` 不同 | 统一用 `@` / `np.matmul` |
| view 改了污染原数组 | `a[0]`、`reshape`、`T` 都是 view | 要独立就 `.copy()` |
| 整数除法 | numpy 2.x 里 `/` 总是真除法，得到 float | 整数除法用 `//` |
| 整数溢出 | `np.int8` 等窄类型会回绕 | DL 里统一 `float32` |
| 浮点相等判断 | `a == b` 常常是 False | 用 `np.allclose` |
| `reshape` 元素数对不上 | `(2,3)` → `reshape(4)` 报错 | 用 `-1` 让它自己推（但只能有一个 -1） |
| 广播方向反了 | 想按列对齐全写成了按行 | 记住"从最右维开始比对" |
| 忘了 `keepdims` | 归约后维度丢了，后面广播出错 | softmax/normalize 一律带 keepdims |

---

## 11. 一段最小可跑的自检脚本

复制到本地跑一遍，手感就有了：

```python
import numpy as np

a = np.arange(6).reshape(2, 3)
print(a.shape, a.dtype)              # (2, 3) int64

print(a.sum(axis=1))                 # [ 3 12]
print(a.sum(axis=1, keepdims=True).shape)   # (2, 1)

print((a + np.arange(3)).shape)      # (2, 3)   广播
print((np.random.randn(3,1) + np.random.randn(1,4)).shape)   # (3, 4)

A = np.arange(6).reshape(2,3); B = np.arange(12).reshape(3,4)
print((A @ B).shape)                                   # (2, 4)
print(np.allclose(np.einsum('ij,jk->ik', A, B), A @ B))  # True

m = np.zeros((3,2,4)); n = np.zeros((3,4,5))
print((m @ n).shape, np.dot(m, n).shape)   # (3, 2, 5) (3, 2, 3, 5) ← 记住这个区别

rng = np.random.default_rng(0)
print(rng.normal(size=(2,2)))
```

---

相关：[[d2L 学习]] · [[python tips]] · [[conda 常用指令]]
