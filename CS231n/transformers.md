## 一、`train_imdb.py` 整体在做什么？

1. 加载 IMDb 数据集

真实评论 + 正面 / 负面标签

2. AutoTokenizer（自动分词器）

文本 → Token IDs（词元编号）

3. 加载预训练 DistilBERT

预训练语言模型 + 新的分类头

4. Trainer（训练管理器）微调

计算损失 → 反向传播 → 更新模型参数

5. 验证、测试、保存模型

比较指标，保存微调后的模型和分词器

`distilbert-base-uncased` 是一个已经通过大规模文本预训练的 DistilBERT 模型。它已经学到了很多通用的语言模式，因此不用从随机参数开始训练整个网络。