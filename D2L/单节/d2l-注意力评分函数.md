---
tags:
  - 深度学习
  - d2l
  - 注意力
---

# D2L《注意力评分函数》大白话笔记

> 原文链接:https://zh.d2l.ai/chapter_attention-mechanisms/attention-scoring-functions.html
> 前置:10.1 注意力提示、10.2 注意力汇聚(Nadaraya-Watson 核回归)
> 后置:10.4 Bahdanau 注意力、10.5 多头注意力、10.7 Transformer

## 这节在解决什么问题

上一节(10.2)用"高斯核"来计算查询和键之间的关系,并且发现:高斯核里那个指数部分,可以单独拎出来看成一个**打分函数**。

这一节就把这件事正式化:**注意力机制里"怎么给每个键打分"这一步,抽象成一个函数——注意力评分函数(attention scoring function,简称评分函数)**。打分方式不同,注意力就不同。这一节介绍两种最常用的打分方式,它们也是后面 Transformer 的零件来源。

## 先回忆:注意力机制的三步流程

不管用什么评分函数,注意力的流程都是固定的三步:

```
1. 打分:    用评分函数 a 给"查询 q"和"每个键 k_i"打一个分
2. 归一化:  把这些分数送进 softmax,变成一组加起来等于 1 的权重 α_i(概率分布)
3. 加权求和:用权重 α_i 对每个值 v_i 加权求和,得到最终输出
```

用数学写出来就是书里的两个公式:

- 输出 = 加权和(本质是加权平均):

  `f(q, (k₁,v₁), …, (kₘ,vₘ)) = Σᵢ α(q, kᵢ) · vᵢ`

- 权重 = 分数经过 softmax:

  `α(q, kᵢ) = softmax(a(q, kᵢ)) = exp(a(q,kᵢ)) / Σⱼ exp(a(q,kⱼ))`

> **大白话类比**:考试排名。评分函数 = 改卷老师(给每科打多少分,规则可以换);softmax = 把分数换算成"占比权重";加权求和 = 算加权平均总分。换一个"改卷规则"(评分函数),最终排名就会不同。

记住这个框架,后面所有内容都只是"给第一步换打分方式"。

## 零件一:掩蔽 softmax(先解决一个小麻烦)

**问题**:实际处理文本时,每句话长短不一,为了凑成整齐的小批量,短句子会被"填充(padding)"一些没意义的词元。这些填充位置**不应该参与**注意力的计算。

**做法**:给每个样本记一个"有效长度" `valid_lens`,计算 softmax 之前,把超出有效长度的位置的分**改成非常大的负数**(书里用 -1e6)。因为 `exp(-1e6) ≈ 0`,softmax 之后这些位置的权重自然变成 0——相当于被"掩蔽"掉了。

```python
# 核心思想(PyTorch 版)
def masked_softmax(X, valid_lens):
    if valid_lens is None:
        return nn.functional.softmax(X, dim=-1)
    # 无效位置替换成超大负数 -1e6,softmax 后自动为 0
    X = d2l.sequence_mask(X, valid_lens, value=-1e6)
    return nn.functional.softmax(X, dim=-1)
```

> **大白话**:投票时,没资格投票的人不是删掉,而是把他们的票数改成"负无穷",归一化后权重自动归零。

书里的例子:`X` 形状 (2, 2, 4),两个样本的有效长度分别是 2 和 3。输出矩阵里,每行只有前 2(或 3)个位置有非零值,其余全是 0。有效长度还可以是二维的,给每一行单独指定。

## 零件二:加性注意力

**什么时候用**:查询 q 和键 k 的**长度不一样**时。

**公式**:

`a(q, k) = wᵥᵀ · tanh(W_q·q + W_k·k)`

里面有三个可学习参数:`W_q`、`W_k`(把 q、k 投影到同一个 h 维空间)和 `wᵥ`(投影成一个数字)。

**大白话拆解**:

1. q 和 k 长度不同没法直接相加,先各自乘一个矩阵 `W_q`、`W_k`,都投影到长度为 h 的空间(对齐了);
2. 两个投影结果**相加**——所以叫"加性"注意力;
3. 过 tanh 激活;
4. 再用 `wᵥ` 压成一个数字,就是分数。

> 等价理解:把 q 和 k 拼起来,送进一个**只有一个隐藏层的 MLP** 打分,隐藏单元数 h 是超参数,激活函数 tanh,不要偏置项。

**加性注意力的实现骨架**(PyTorch 版,核心几行):

```python
class AdditiveAttention(nn.Module):
    def __init__(self, key_size, query_size, num_hiddens, dropout):
        super().__init__()
        self.W_k = nn.Linear(key_size, num_hiddens, bias=False)
        self.W_q = nn.Linear(query_size, num_hiddens, bias=False)
        self.w_v = nn.Linear(num_hiddens, 1, bias=False)
        self.dropout = nn.Dropout(dropout)

    def forward(self, queries, keys, values, valid_lens):
        queries, keys = self.W_q(queries), self.W_k(keys)
        features = queries.unsqueeze(2) + keys.unsqueeze(1)  # 广播相加
        features = torch.tanh(features)
        scores = self.w_v(features).squeeze(-1)              # 打分
        self.attention_weights = masked_softmax(scores, valid_lens)
        return torch.bmm(self.dropout(self.attention_weights), values)
```

## 零件三:缩放点积注意力

**什么时候用**:查询 q 和键 k 的**长度相同**(都是 d 维)时。点积算得快,但要求两边维度一致。

**公式**:

`a(q, k) = qᵀk / √d`

**大白话**:直接算 q 和 k 的**内积**(天然衡量两个向量的相似程度),再除以 √d。

**为什么要除以 √d(这是"缩放"两个字的由来,也是本节最值得记的一个细节)**:

假设 q 和 k 的每个元素都是独立随机变量,零均值、单位方差。那么两个 d 维向量点积的结果:

- 均值为 0;
- **方差为 d**。

也就是说,d 越大,分数数值越"飘",送进 softmax 后容易极端化(梯度饱和,不好训练)。除以 √d 后,方差被拉回到 1,与向量长度无关,softmax 更稳定。

**批量形式**(实际计算时一次算完所有查询,矩阵乘法,GPU 飞快):

`输出 = softmax(Q·Kᵀ / √d) · V`

其中 Q 是 n 个查询、K 是 m 个键、V 是 m 个值。

> 这也就是后来 **Transformer 里 `QKᵀ/√d` 的来源**——看到这个式子就不用陌生了。

**实现骨架**(PyTorch 版,核心就一行):

```python
class DotProductAttention(nn.Module):
    def __init__(self, dropout):
        super().__init__()
        self.dropout = nn.Dropout(dropout)

    def forward(self, queries, keys, values, valid_lens=None):
        d = queries.shape[-1]
        # 核心:点积 → 除以 √d → 掩蔽 softmax → 对值加权
        scores = torch.bmm(queries, keys.transpose(1, 2)) / math.sqrt(d)
        self.attention_weights = masked_softmax(scores, valid_lens)
        return torch.bmm(self.dropout(self.attention_weights), values)
```

## 两种评分函数怎么选(速查表)

| 情况 | 选哪个 | 为什么 |
| --- | --- | --- |
| 查询和键**长度不同** | 加性注意力 | 先投影对齐再相加,天然支持不同维度 |
| 查询和键**长度相同** | 缩放点积注意力 | 纯矩阵乘法,计算效率更高 |

## 一张图串起整节

```
查询 q ──────┐
             ├──▶ 评分函数 a(q, kᵢ) ──▶ softmax ──▶ 权重 αᵢ ──┐
所有键 kᵢ ───┘       (加性 / 缩放点积)         (无效位置先掩蔽)  ├──▶ Σ αᵢ·vᵢ 输出
所有值 vᵢ ──────────────────────────────────────────────────┘
```

## 看完这节你该有的感觉

1. 注意力机制 = **打分 → softmax → 加权平均**;评分函数只是"打分方式",换汤不换药。
2. 掩蔽 softmax:填充位置填超大负数,softmax 后自动权重为 0。
3. 加性注意力:`wᵥᵀ·tanh(W_q q + W_k k)`,q、k 长度不同时用,相当于一个小 MLP 打分。
4. 缩放点积注意力:`qᵀk/√d`,q、k 长度相同时用,更快;除以 √d 是为了把点积方差从 d 拉回 1,防止 softmax 饱和。Transformer 用的就是它。

下一节(Bahdanau 注意力)会把**加性注意力**装进 seq2seq 的翻译模型里,让解码器翻译每个词时都能"看"整个输入句子——到那时这两个评分函数就派上实际用场了。

## 书后练习(可以先想想)

1. 修改小例子中的键并可视化注意力权重:加性注意力和缩放点积注意力还会产生相同结果吗?为什么?
2. 只用矩阵乘法,能不能为长度不同的查询和键设计新的评分函数?
3. 当 q、k 长度相同时,用"向量求和"当评分函数,会比点积更好吗?为什么?

## 参考

- [10.3 注意力评分函数(zh.d2l.ai)](https://zh.d2l.ai/chapter_attention-mechanisms/attention-scoring-functions.html)
- [上一节:10.2 注意力汇聚:Nadaraya-Watson 核回归](https://zh.d2l.ai/chapter_attention-mechanisms/nadaraya-watson.html)
- [下一节:10.4 Bahdanau 注意力](https://zh.d2l.ai/chapter_attention-mechanisms/bahdanau-attention.html)
