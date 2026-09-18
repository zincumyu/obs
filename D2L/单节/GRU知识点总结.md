---
tags:
  - 深度学习
  - d2l
  - RNN
---

# 门控循环单元(GRU)知识点总结

> 📺 **视频来源**:[李沐《动手学深度学习 v2》第 56 讲 · 门控循环单元(GRU)](https://www.bilibili.com/video/BV1mf4y157N2/?spm_id_from=333.1387.collection.video_card.click&vd_source=d01609d3cc689e55b51b1518a2154104)
> 📖 **配套教材**:[d2l-zh · 9.1 门控循环单元(GRU)](https://zh.d2l.ai/chapter_recurrent-modern/gru.html)
> 💬 **课后讨论**:[discuss.d2l.ai 讨论帖](http://discuss.d2l.ai/t/gru/2763)

---

## 一、为什么需要 GRU?(背景与动机)

### 1. 回顾 RNN 的隐状态更新

$$
H_t = \phi(X_t W_{xh} + H_{t-1} W_{hh} + b_h)
$$

- RNN 通过隐状态在时间步之间循环传递信息。
- 输出层、状态更新共用同一套循环结构。

### 2. RNN 存在的问题

| 问题 | 说明 |
| --- | --- |
| **梯度问题** | 长序列上梯度消失/爆炸,难以学习长距离依赖 |
| **隐状态被不断重写** | 每个时间步都用新状态整体覆盖旧状态,没有"选择性",无法主动决定保留什么、丢弃什么 |
| **并非每个观察值都同等重要** | 序列中某些 token 很关键(如句首词、动词),某些只是辅助(如虚词、标点);RNN 无法自适应地"关注"重要位置 |

### 3. GRU 的核心思想

- **"关注一个序列"(attend over one sequence)**:门控机制其实是**注意力机制的早期形态**,早于注意力机制被提出。
- **引入"门"(Gate)显式控制信息流**:
  - 哪些过去信息该**记住**?
  - 哪些该**忘记**?
  - 当前输入有多少价值值得写入状态?

---

## 二、GRU 的核心:门控隐状态

GRU 通过 **重置门(Reset Gate)** 与 **更新门(Update Gate)** 两个门来控制隐状态,是 LSTM 的精简变体。

### 1. 重置门 $R_t$

$$
R_t = \sigma(X_t W_{xr} + H_{t-1} W_{hr} + b_r)
$$

- **作用**:控制上一个时间步的隐状态 $H_{t-1}$ 对**候选隐状态**的贡献程度。
- 值越接近 0,越多地"遗忘"过去信息 → 有助于捕获序列中的**短期依赖关系**。

### 2. 更新门 $Z_t$

$$
Z_t = \sigma(X_t W_{xz} + H_{t-1} W_{hz} + b_z)
$$

- **作用**:控制隐状态被更新的程度——有多少来自旧隐状态、多少来自候选隐状态。
- 值越接近 1,越多地保留旧状态 → 有助于捕获序列中的**长期依赖关系**。

### 3. 候选隐状态 $\tilde{H}_t$

$$
\tilde{H}_t = \tanh(X_t W_{xh} + (R_t \odot H_{t-1}) W_{hh} + b_h)
$$

- 用**重置门加权后的过去状态**与当前输入一起计算。
- $\odot$ 为 Hadamard 逐元素乘积;$\tanh$ 保证取值在 $(-1, 1)$。

### ==4. 隐状态更新==

$$
H_t = Z_t \odot H_{t-1} + (1 - Z_t) \odot \tilde{H}_t
$$

- 本质是**==加权平均==**:
  - $Z_t \to 1$:保留旧隐状态(记忆过去);
  - $Z_t \to 0$:采用候选隐状态(写入新信息)。

### 5. 关键细节

- 两个门的**输入相同**(都是 $X_t$ 和 $H_{t-1}$),只是**参数不同**(各自的权重矩阵 $W_{x*}$、$W_{h*}$ 与偏置 $b_*$)。
- $\sigma$ 输出在 $(0,1)$,充当"软开关";$\tanh$ 输出在 $(-1,1)$,充当状态取值。
- **退化情形**:当重置门全为 1、更新门全为 0 时,GRU 退化为普通 RNN:
  - 重置门全 1 → 候选状态等价于 RNN 的状态更新;
  - 更新门全 0 → $H_t = \tilde{H}_t$。

---

## 三、从零实现(d2l 代码要点)

> 数据集:《时间机器》(The Time Machine),训练**字符级语言模型**——给定前面的字符,预测下一个字符。

1. **初始化参数**:
   - 三组权重与偏置:更新门 $(W_{xz}, W_{hz}, b_z)$、重置门 $(W_{xr}, W_{hr}, b_r)$、候选隐状态 $(W_{xh}, W_{hh}, b_h)$;
   - 权重均用**正态分布**初始化(标准差 0.01),偏置置 0。
2. **模型前向计算**(三层循环 `for`):
   - `Z = sigmoid(X @ W_xz + state @ W_hz + b_z)`
   - `R = sigmoid(X @ W_xr + state @ W_hr + b_r)`
   - `H_tilda = tanh(X @ W_xh + (R * state) @ W_hh + b_h)`
   - `state = Z * state + (1 - Z) * H_tilda`
   - 注意:`@` 为矩阵乘法,`*` 为逐元素乘法。
3. **训练**:沿用 `d2l.train_ch8`:
   - 评估指标:**困惑度(perplexity)**;
   - 序列采样方式:**随机采样**;
   - 使用**梯度裁剪**防止梯度爆炸;
   - 与上一讲(循环神经网络从零实现)的流程完全一致,只替换模型定义。

---

## 四、简洁实现

```python
gru_layer = nn.GRU(input_size, hidden_size, num_layers)
# 或使用封装好的 d2l.GRU / rnn.GRU
output, state = gru_layer(X) 
```

- 一行代码即可构建 GRU 层,训练代码与 RNN 相同(`d2l.train_ch8`)。
- 训练结果与从零实现**几乎一致**,验证了手写实现的正确性。
- PyTorch 中 `nn.GRU` 返回输出序列和最终隐状态。

---

## 五、小结(核心考点)

1. 门控循环神经网络(GRU)可以更好地捕获**时间步距离很长的序列上的依赖关系**。
2. **重置门**有助于捕获序列中的**短期依赖**——控制过去状态对候选状态的贡献。
3. **更新门**有助于捕获序列中的**长期依赖**——控制新旧状态的混合比例。
4. 当**重置门打开(全 1)、更新门关闭(全 0)**时,GRU 退化为基础 RNN。
5. GRU 与 LSTM 的关系(预告下一讲 LSTM):
   - GRU:2 个门(重置门 + 更新门),无独立的记忆单元,参数更少、更精简;
   - LSTM:3 个门(输入门 + 遗忘门 + 输出门),有独立的记忆单元 $C_t$。

---

## 六、参考资料

| 资料 | 链接 |
| --- | --- |
| 视频原片 | [B 站 · BV1mf4y157N2(李沐《动手学深度学习 v2》56 · 门控循环单元 GRU)](https://www.bilibili.com/video/BV1mf4y157N2/) |
| 配套教材 | [d2l-zh · 门控循环单元(GRU)](https://zh.d2l.ai/chapter_recurrent-modern/gru.html) |
| 讨论区 | [discuss.d2l.ai/t/gru](http://discuss.d2l.ai/t/gru/2763) |
| 社区笔记 | [GitHub · DeepLearning-MuLi-Notes/56-GRU](https://github.com/liu-yang-maker/DeepLearning-MuLi-Notes/blob/main/notes/56-GRU.md) |
| 上一讲(前置知识) | [d2l-zh · 循环神经网络(RNN)](https://zh.d2l.ai/chapter_recurrent-neural-networks/rnn.html) |
| 下一讲(延伸学习) | [d2l-zh · 长短期记忆网络(LSTM)](https://zh.d2l.ai/chapter_recurrent-modern/lstm.html) |
