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

任务：把模型封装成 API
```json
{
  "text": "The movie was absolutely wonderful."
}
```

服务返回：

```json
{
  "label": "POSITIVE",
  "score": 0.9724
}
```

# FastAPI + Requests 学习笔记

## 一、它们分别是干什么的？

FastAPI（Web API 框架）负责提供服务，Requests（HTTP 客户端库）负责调用服务。

这里先理解四件事：

- `app = FastAPI()`：创建 Web 应用。
- `@app.post("/predict")`：注册一个 POST（提交数据）接口。
- `PredictRequest`：定义请求的数据结构，并通过 Pydantic 校验输入。
- `return {...}`：返回 Python 字典，FastAPI 会将它转换为 JSON（常见的网络数据格式）。

可以看看`\test_api_errors.py` 和\client_test 和\app.py