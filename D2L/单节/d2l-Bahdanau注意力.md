---
tags:
  - 深度学习
  - d2l
  - 注意力
---

# D2L《Bahdanau 注意力》大白话笔记

> 原文链接:https://zh.d2l.ai/chapter_attention-mechanisms/bahdanau-attention.html
> 前置:9.6 编码器-解码器架构、9.7 seq2seq、10.3 注意力评分函数
> 后置:10.5 多头注意力、10.7 Transformer

## 这节在解决什么问题

回忆 9.7 节的 seq2seq 翻译模型:

- 编码器把**整句英文**读进去,最后只留下**一个固定形状的上下文变量 c**;
- 解码器翻译**每个词**时,用的都是这**同一个 c**。

问题就出在这:翻译不同位置的词,需要关注原文里**不同位置**的信息,但每个解码步却只能看到同一个"压缩包"。句子一长,信息全挤在一个固定大小的向量里,容易丢——这就是著名的**信息瓶颈**。

> **大白话类比**:普通 seq2seq 像翻译员**只听一遍**英文,在纸条上写下一句固定长度的"中心思想",然后整场翻译只盯着这张纸条,不允许再看原文。句子短还行,句子一长,纸条记不下,翻译就歪了。

**这一节的方案**:翻译每个词的时候,允许翻译员**回头重新扫一遍原文**,并且重点看"和当前要翻译的词最相关的那部分"。这个"动态回头看、按相关性取信息"的机制,就是 Bahdanau 注意力(2014 年提出,是注意力机制第一次被大规模用在机器翻译上)。

### 一点历史背景

- Graves(2013)在"根据文本生成手写笔迹"任务里设计了可微注意力模型,把文字和笔迹**对齐**,但只允许对齐方向**单向移动**(从左到右,不能回头)。
- Bahdanau 等人(2014)受"学习对齐"启发,去掉"只能单向"的限制:预测每个词元时,**对齐输入序列中和当前预测相关的任意部分**。

## 核心改动:上下文变量从"一个"变成"每个解码步一个"

与 9.7 的模型相比,Bahdanau 注意力模型**只改一处**:把原来固定的上下文变量 c,换成**每个解码时间步 t' 各有一个**的 c_{t'}:

`c_{t'} = Σₜ α(s_{t'-1}, hₜ) · hₜ   (t 从 1 到 T,遍历整个输入序列)`

其中三个角色的分工(和 10.3 节的三步流程对号入座):

| 角色 | 是什么 | 说明 |
| --- | --- | --- |
| 查询(query) | 解码器**上一步**的隐状态 s_{t'-1} | "我现在翻译到哪了",决定该看原文哪里 |
| 键(key) | 编码器**所有时间步**的隐状态 h₁…h_T | 原文每个位置的"标签",用来和查询打分 |
| 值(value) | 同样是 h₁…h_T | 每个位置携带的信息,按权重取加权和 |
| 打分函数 | **加性注意力**(10.3 节学的) | 查询和键长度相同时也可换点积,见练习 2 |

> **一句话总结**:每个解码步,拿"上一步的翻译进度"当查询,和原文每个位置打分,加权求和出一个**专属于这一步**的上下文 c_{t'},再据此翻译下一个词。

## 模型长什么样

```
解码第 t' 个词时:
  s_{t'-1}(上一步解码器状态) ──查询──▶ 与每个 hₜ 打分(加性注意力)
                                          │ softmax
                                          ▼
                                 权重 α₁…α_T(热力图里能看到"对齐")
                                          │ 加权求和
                                          ▼
                              c_{t'} = Σ αₜ·hₜ(这一步专属的上下文)
                                          │ 与当前词嵌入拼接
                                          ▼
                                  喂给 GRU 解码器 → 输出词元
```

编码器部分**完全不用改**,只需重新定义解码器:

- 初始化状态时,要保存三样东西:编码器所有时间步的隐状态(当键和值)、编码器最后一层隐状态(初始化解码器)、编码器有效长度(掩蔽 padding);
- 每个解码步:用上一步隐状态当查询 → 算出上下文 c_{t'} → 和当前词嵌入**拼接** → 送进 GRU。

解码器核心循环(PyTorch 版,骨架):

```python
class Seq2SeqAttentionDecoder(AttentionDecoder):
    def __init__(self, vocab_size, embed_size, num_hiddens, num_layers, dropout=0):
        super().__init__()
        self.attention = d2l.AdditiveAttention(num_hiddens, num_hiddens, num_hiddens, dropout)
        self.embedding = nn.Embedding(vocab_size, embed_size)
        self.rnn = nn.GRU(embed_size + num_hiddens, num_hiddens, num_layers, dropout=dropout)
        self.dense = nn.Linear(num_hiddens, vocab_size)

    def forward(self, X, state):
        enc_outputs, hidden_state, enc_valid_lens = state
        X = self.embedding(X).permute(1, 0, 2)
        outputs, self._attention_weights = [], []
        for x in X:
            query = torch.unsqueeze(hidden_state[-1], dim=1)          # 查询 = 上一步隐状态
            context = self.attention(query, enc_outputs, enc_outputs, enc_valid_lens)  # 键=值=编码器输出
            x = torch.cat((context, torch.unsqueeze(x, dim=1)), dim=-1)  # 上下文和词嵌入拼接
            out, hidden_state = self.rnn(x.permute(1, 0, 2), hidden_state)
            outputs.append(out)
            self._attention_weights.append(self.attention.attention_weights)
        outputs = self.dense(torch.cat(outputs, dim=0))
        return outputs.permute(1, 0, 2), [enc_outputs, hidden_state, enc_valid_lens]
```

## 训练与效果

- 超参数和 9.7 节基本一样(embed=32、隐藏 32、2 层、250 轮);但**训练明显变慢**——因为每个解码步都要对整句算一遍注意力(代价约 T×T 次打分),这是注意力的"出场费"。
- 训练后的翻译示例(英 → 法):"go ." → "va !","i lost ." → "j'ai perdu .","i'm home ." → "je suis chez moi ." 都能翻对,BLEU 分数高。
- 关键证据:**注意力权重热力图**。把每个查询(解码位置)对每个键(原文位置)的权重画成热力图,能看到一条清晰的"对角线"——翻译第几个词,注意力就集中在原文第几个词附近。这证明模型**真的学会了对齐**,翻译每个词时看的是原文不同部分。

> 这就是注意力带来的质变:模型不再是"背一句总结",而是**每输出一个词,重新定位原文**,长句翻译能力大幅提升。

## 和普通 seq2seq 对比(速查表)

| | 9.7 普通 seq2seq | 10.4 Bahdanau 注意力 |
| --- | --- | --- |
| 上下文变量 | 固定一个 c,每步都用它 | 每步一个 c_{t'},各不相同 |
| 解码时能否"看原文" | 不能,只能看 c | 能,直接和所有 hₜ 交互 |
| 关键部件 | 编码器 + 解码器 | 编码器 + **带注意力的**解码器 |
| 对齐能力 | 隐式、易丢信息 | 显式学出对齐(热力图可见) |
| 训练速度 | 快 | 慢(每步都要算注意力) |

## 看完这节你该有的感觉

1. Bahdanau 注意力 = **把 10.3 的加性注意力装进 seq2seq 解码器**:查询 = 上一步解码器隐状态,键 = 值 = 编码器所有隐状态,输出 = 每一步专属的上下文 c_{t'}。
2. 它解决了 seq2seq 的**信息瓶颈**:每步都能"回头全文检索",并学会自动对齐。
3. 代价是**训练变慢**(每步都要打分),后面 10.7 的 Transformer 会用"自注意力 + 并行化"把效率问题解决掉。
4. 术语:这种"查询来自解码器、键值来自编码器"的注意力,常被称为**交叉注意力(cross-attention)**;热力图里那条对角线就是"对齐"的直观证据。

## 书后练习(可以先想想)

1. **把 GRU 换成 LSTM**:LSTM 隐状态和 GRU 类似,直接替换后重训即可,目的是体会"注意力机制与具体 RNN 单元解耦"。
2. **把加性注意力换成缩放点积注意力**:这里查询(解码器隐状态)和键(编码器隐状态)长度相同,完全可以用点积;训练会更快,但效果不一定更好——点积注意力没有可学习参数(10.3 讲过它只有 dropout),打分能力比带参数的加性注意力弱。

## 参考

- [10.4 Bahdanau 注意力(zh.d2l.ai)](https://zh.d2l.ai/chapter_attention-mechanisms/bahdanau-attention.html)
- [上一节:10.3 注意力评分函数](https://zh.d2l.ai/chapter_attention-mechanisms/attention-scoring-functions.html)
- [下一节:10.5 多头注意力](https://zh.d2l.ai/chapter_attention-mechanisms/multihead-attention.html)
- [Bahdanau et al., 2014. Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473)
