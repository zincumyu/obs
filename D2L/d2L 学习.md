---
tags:
  - 深度学习
  - d2l
---

![[Pasted image 20260730213443.png]]

Q1 "axis"怎么区分维度

detach()去梯度计算
![[Pasted image 20260801233607.png]]
![[Pasted image 20260801233623.png]]
1. 尝试添加不同数量的隐藏层（也可以修改学习率），怎么样设置效果最好？

2. 尝试不同的激活函数，哪个效果最好？

3. 尝试不同的方案来初始化权重，什么方法效果最好？
4. **![[Pasted image 20260802120507.png]]![[Pasted image 20260802120517.png]]**
5. ## 4.7 前向传播、反向传播、计算图

### 1. 三个概念一句话

- **前向传播（forward）**：从输入 → 输出，按计算图顺序算并**存中间变量**（z、h、o、L）。
    
- **反向传播（backward）**：从损失 → 参数，用**链式法则**反着算梯度，复用前向存的中间值。
    
- **计算图**：方块=变量，圆圈=算子，箭头表示依赖；正向向右上，反向向左下。
    

### 2. 单隐层 MLP 的公式链（必会）

```python
z = W₁x
h = ϕ(z)            # 激活
o = W₂h
L = l(o, y)         # 损失
s = λ/2 (‖W₁‖²+‖W₂‖²)   # L2 正则
J = L + s           # 目标函数
```

反向时关键几步（链式法则）：

```python
∂J/∂W₂ = (∂J/∂o) hᵀ + λW₂
∂J/∂h = W₂ᵀ (∂J/∂o)
∂J/∂z = (∂J/∂h) ⊙ ϕ′(z)
∂J/∂W₁ = (∂J/∂z) xᵀ + λW₁
```

⊙ 是逐元素乘，ϕ′ 是激活导数。

### 3. 训练时的两个事实（容易考）

- 前向和反向**交替进行**，反向依赖前向存的中间值。
    
- 训练比推理占显存：中间变量要留到反向结束才能释放；**层数深 × batch 大 → 容易 OOM**。
    

---

## 4.8 数值稳定性和模型初始化

### 1. 根本矛盾：梯度是连乘

深网反向梯度 ≈ 多层雅可比连乘：

- 每层因子 **>1**​ 连乘 → 梯度爆炸（loss 变 nan/inf）
    
- 每层因子 **<1**​ 连乘 → 梯度消失（前几层不更新）
    

举例：0.2³⁰ ≈ 1e-21，5³⁰ ≈ 9e20，都是数值灾难。

### 2. 激活函数的锅

- **Sigmoid / Tanh**：两端饱和，导数→0，深网必消失。
    
- **ReLU 族**：正区间导数=1，不放大不缩小，默认选它；负区间死神经元用 LeakyReLU/PReLU 救。
    
- 结论：现在 CNN 用 ReLU 系，Transformer 用 GELU，sigmoid 只在门控/注意力里做概率用。
    

### 3. 为什么不能全初始化成一样

全 0 或全相同值 → 对称破碎失败：

- 前向每个隐单元输出一样
    
- 反向每个隐单元梯度一样
    
- 更新后还一样 → **隐层等效于 1 个单元**，表达能力崩塌
    
    → 必须**随机初始化**打破对称性。
    

### 4. 初始化怎么选（核心结论）

目标：**让每层输出方差 ≈ 1，反向梯度方差 ≈ 1**。

- **Xavier / Glorot**：适合 tanh/sigmoid
    
    `Var(W) = 2/(n_in + n_out)`，均匀版 `a = sqrt(6/(n_in+n_out))`
    
- **He / Kaiming**：专为 ReLU 设计（ReLU 砍掉一半激活，方差减半要补回来）
    
    `Var(W) = 2/n_in`
    
- 框架默认（PyTorch `nn.Linear`）就是 He 均匀，中等问题不用手调；自己写层再手动指定。
    

### 5. 工程上还有哪些补丁

初始化只是第一道：

- **BatchNorm / LayerNorm**：把每层输入拉回稳定分布
    
- **残差连接**：梯度多一条 `+1` 直通路，防消失
    
- **梯度裁剪**：爆炸时按范数截断
    
- **选 ReLU 系 + 合适初始化**：解决大部分基础稳定性问题


pytorch
![[Pasted image 20260804115143.png]]
**MLP = Multi-Layer Perceptron**

|词|人话|
|---|---|
|Multi|不止一层|
|Layer|Linear + 激活|
|Perceptron|神经元（最原始叫法）|

nn.Module的__call__调用了forward

“卷积”这个词承载了：

- 局部加权求和
    
- 平移等变
    
- 频域乘积


PyTorch 干的事：

> **从 Loss 出发，用链式法则一路倒推：
> 
> “Loss 变大，每个权重该负多少责？”**

得到：

```python
net.weight.grad   # ∇W
```

## 2. 梯度：损失对权重的“敏感度”

梯度是损失函数对每个权重的偏导数组成的向量：

它的物理意义：

- 每个分量告诉你：**稍微动一下这个权重，损失会往哪个方向、以多大速率变化**
    
- 梯度方向 = 损失**上升最快**的方向
    
- 负梯度方向 = 损失**下降最快**的方向

## 3. 权重更新：用梯度“下山”

梯度下降的更新规则：

w←w−η∇w​L


卷积尺寸公式：

(H+2p−k)/s+1

卷积（**Conv2d**）在 PyTorch 里的**输出尺寸公式**（最常用、面试/写网络必背）：

---

## 一、标准步幅（stride）计算公式

对于 **正方形输入 + 正方形卷积核**（最常见）：

Hout​=⌊SHin​+2P−K​⌋+1

同理：

Wout​=⌊SWin​+2P−K​⌋+1

### 符号含义

|符号|含义|
|---|---|
|Hin​,Win​|输入高 / 宽|
|K|kernel_size（如 3）|
|P|padding（每边）|
|S|stride（步幅）|
|⌊⋅⌋|向下取整（PyTorch 默认）|
```
def corr2d_multi_in_out(X, K):

    # 迭代“K”的第0个维度，每次都对输入“X”执行互相关运算。

    # 最后将所有结果都叠加在一起

    return torch.stack([corr2d_multi_in(X, k) for k in K], 0) //0维堆叠
```
![[Pasted image 20260805211303.png]]
___

![[Pasted image 20260805211401.png]]
```python
def corr2d_multi_in_out_1x1(X, K):

    c_i, h, w = X.shape

    c_o = K.shape[0]

    print(c_i, h, w, c_o,c_i)

    X = X.reshape((c_i, h * w))  ### 把空间维度“压扁” h*w 直接变一维度

    K = K.reshape((c_o, c_i))

    # 全连接层中的矩阵乘法

    Y = torch.matmul(K, X)

    return Y.reshape((c_o, h, w)) #⑤ reshape 回特征图
```

# 举例卷积操作
假设：

- 输入：`96 × H × W`（96 通道）
    
- 卷积层：`Conv2d(96, 128, 3, padding=1)`
    
- 看其中**第 k 个输出通道**（k = 0…127）
    

这一路的计算是：

1. 有一个卷积核：形状 `3 × 3 × 96`
    
2. 在输入的每一个空间位置 (i, j)：
    
    - 取输入以 (i,j) 为中心的 3×3×96 小块
        
    - 和这个 3×3×96 的核做**逐元素相乘再求和**
        
    - 得到一个**标量**
        
    
3. 所有空间位置都算完
    
4. 得到一张 `H × W` 的图

# 深度卷积神经网络（AlexNet）
```python
X = torch.rand(size=(1, 1, 224, 224))

for layer in net:

    X = layer(X)

    print(layer.__class__.__name__,'output shape:\t', X.shape)
```
 # re 批量归一化

# 32-35没有学习


## 标号混合
![[Pasted image 20260810145705.png]]

[[task]]
********
Scheduler（调度器）

## 锚框
Q1![[Pasted image 20260812103133.png]]

这段内容讲的是目标检测中非常核心的一步：**如何把生成的几万个“锚框”（候选框）和真实的“边界框”（GT）进行配对，从而制作出训练标签**。

因为一张图里通常只有几个真实目标（比如一只狗、一辆车），但会生成几万个锚框。我们必须定个规矩，告诉模型：“哪些锚框算正样本（有狗）？哪些算负样本（只是背景）？”

这个算法就是来解决“配对打标签”问题的。我们可以把它想象成一个**“相亲配对 + 背景筛查”**的过程，分两步走：

### 第一步：强制一对一匹配（保证每个真实目标都有人负责）

这一步对应文字中的步骤 1、2、3 和下面的三个矩阵图。

- **逻辑**：假设图里有 4 个真实目标（B1​ 到 B4​），为了防止漏掉任何一个目标，我们必须保证这 4 个目标至少各有一个锚框与之对应。
    
- **做法**：构建一个 IoU（交并比）矩阵 X，行是锚框（共9个），列是真实框（共4个），里面的数字是它们重叠的程度。
    
    1. 先在矩阵里找**全局最大**的 IoU（图中是 x23​，即锚框 A2​ 和真实框 B3​ 最吻合），直接把它们配对！然后把 A2​ 所在的第2行、B3​ 所在的第3列全部划掉（阴影部分），因为 A2​ 已经名花有主，B3​ 也找到了归宿，不再参与后续的竞争。
        
    2. 在剩下的空白区域里，继续找最大的 x71​（A7​ 和 B1​ 配对），划掉第7行和第1列。
        
    3. 接着找最大的 x54​（A5​ 和 B4​ 配对），划掉第5行和第4列。
        
    4. 最后找最大的 x92​（A9​ 和 B2​ 配对），划掉第9行和第2列。
        
    
- **结果**：经过这一步，4个真实目标全部有了专属的锚框（正样本）。这就是文字里说的“直到丢弃掉矩阵中 nb​ 列中的所有元素”。
    

### 第二步：剩余锚框的阈值筛查（处理背景和模糊地带）

这一步对应文字中的步骤 4。

- **逻辑**：刚才配对完，还剩下 A1​,A3​,A4​,A6​,A8​ 这些没人要的锚框。它们可能是背景，也可能刚好和某个真实框有一点点重叠。
    
- **做法**：遍历这些剩下的锚框，看它们和各自 IoU 最大的真实框重叠度有多高。
    
    - 如果 IoU **大于预设阈值**（比如 0.5）：算作正样本（虽然没抢到“最佳匹配”，但也算沾边）。
        
    - 如果 IoU **小于阈值**：算作负样本（纯背景），不参与回归训练。

----
![[Pasted image 20260813172051.png]]


## rnn-scratch 复杂,程序待定



## 转置思路

![[Pasted image 20260815231422.png]]



GRU门控见[[GRU知识点总结]]

```python
def gru(inputs, state, params):

    W_xz, W_hz, b_z, W_xr, W_hr, b_r, W_xh, W_hh, b_h, W_hq, b_q = params

    H, = state

    outputs = []

    for X in inputs:

        Z = torch.sigmoid((X @ W_xz) + (H @ W_hz) + b_z) #

        R = torch.sigmoid((X @ W_xr) + (H @ W_hr) + b_r) #

        H_tilda = torch.tanh((X @ W_xh) + ((R * H) @ W_hh) + b_h)

        H = Z * H + (1 - Z) * H_tilda

        Y = H @ W_hq + b_q

        outputs.append(Y)

    return torch.cat(outputs, dim=0), (H,)
```

reset gate update gate(Z)

![[Pasted image 20260817161547.png]]

## 跳过LSTM(Long Short-Term Memory)

## 跳过双向深度神经网络Transformer 出来后，BiRNN 已经“退役”了

编码器 它定义了一个“规范”，规定编码器必须输出“解码器能用的东西”

编码器解码器
[[d2l-编码器解码器架构]]
![[Pasted image 20260819235852.png]]
FFN(三维全连接单隐藏MLP)后都有接
AddNorm = Add（残差连接）+ Norm（层归一化）

**继承 + 调用父类构造器（super）**​ 的 Python 类定义写法 编码器
```python
#@save

class TransformerEncoder(d2l.Encoder):

    """Transformer编码器"""

    def __init__(self, vocab_size, key_size, query_size, value_size,

                 num_hiddens, norm_shape, ffn_num_input, ffn_num_hiddens,

                 num_heads, num_layers, dropout, use_bias=False, **kwargs):

        super(TransformerEncoder, self).__init__(**kwargs)

        self.num_hiddens = num_hiddens

        self.embedding = nn.Embedding(vocab_size, num_hiddens)

        self.pos_encoding = d2l.PositionalEncoding(num_hiddens, dropout)

        self.blks = nn.Sequential()

        for i in range(num_layers):

            self.blks.add_module("block"+str(i),

                EncoderBlock(key_size, query_size, value_size, num_hiddens,

                             norm_shape, ffn_num_input, ffn_num_hiddens,

                             num_heads, dropout, use_bias))

  

    def forward(self, X, valid_lens, *args):

        # 因为位置编码值在-1和1之间，

        # 因此嵌入值乘以嵌入维度的平方根进行缩放，

        # 然后再与位置编码相加。

        X = self.pos_encoding(self.embedding(X) * math.sqrt(self.num_hiddens))

        self.attention_weights = [None] * len(self.blks)

        for i, blk in enumerate(self.blks):

            X = blk(X, valid_lens)

            self.attention_weights[

             i] = blk.attention.attention.attention_weights

        return X
```
----
```python
encoder = TransformerEncoder(
    200, 24, 24, 24, 24, [100, 24], 24, 48, 8, 2, 0.5)
encoder.eval()
encoder(torch.ones((2, 100), dtype=torch.long), valid_lens).shape
```

state = [
    enc_outputs,      # 0: 编码器的输出
    enc_valid_lens,   # 1: 编码器有效长度（padding mask）
    key_values_list   # 2: 每一层保存的历史 Key/Value
]
```python
if self.training:
        batch_size, num_steps, _ = X.shape
        # dec_valid_lens的开头:(batch_size,num_steps),
        # 其中每一行是[1,2,...,num_steps]
        dec_valid_lens = torch.arange(
            1, num_steps + 1, device=X.device).repeat(batch_size, 1)
    else:
        dec_valid_lens = None
```
**构造“因果掩码”的长度信息：**

- **训练时**：并行处理整句话，需要显式告诉自注意力：
    - 第 1 个位置只能看 1 个词
    - 第 2 个位置只能看 2 个词
    - …
- 所以生成一个 `[1,2,...,num_steps]` 的矩阵
- **预测时**：一次只来一个词，天然满足因果性，不需要这个掩码

### jump BERT +目标检测赛

Embedding 把“词 / 句子 / 东西”变成“模型能算的向量”

|**<br><br>对比<br><br>**|**<br><br>One-Hot<br><br>**|**<br><br>Embedding<br><br>**|
|---|---|---|
|维度|词表大小（巨大）|固定小维度|
|语义|无|有|
|相似度|全 0|可计算|
|是否可学习|否|是|

# re-Adam

![[Pasted image 20260820195023.png]]
![[Pasted image 20260820195108.png]]

## 相关笔记
- [[d2l-编码器解码器架构]]
- [[d2l-注意力评分函数]]
- [[d2l-Bahdanau注意力]]
- [[GRU知识点总结]]
- [[关于AI]]
