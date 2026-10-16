# PyTorch 模型评估：Accuracy、Precision、Recall 与 F1

## 1. 为什么不只看 Accuracy？

Accuracy（准确率）只反映整体预测正确的比例，无法单独说明模型对每个类别的表现。因此，我们还可以查看 Confusion Matrix（混淆矩阵），以及每个类别的 Precision、Recall 和 F1-score。

## 2. 混淆矩阵

当前项目使用 ResNet18 对 `ants`（蚂蚁）和 `bees`（蜜蜂）进行二分类。

混淆矩阵的**行是真实类别，列是预测类别**：

| 真实类别 | 预测 ants | 预测 bees |
|---|---:|---:|
| ants | 66 | 4 |
| bees | 4 | 79 |

- 66 张蚂蚁被正确识别为蚂蚁；4 张蚂蚁被误判为蜜蜂。
- 79 张蜜蜂被正确识别为蜜蜂；4 张蜜蜂被误判为蚂蚁。
- 总样本数为 153，正确预测 145 张，错误预测 8 张。

因此：`Accuracy = (66 + 79) / 153 ≈ 94.8%`。

## 3. 三个分类指标

对于某一个类别，把它当作“正类”，其他类别都当作“负类”：

- **Precision（精确率）**：所有被模型预测为该类别的样本中，实际属于该类别的比例。
  - 公式：`Precision = TP / (TP + FP)`
- **Recall（召回率）**：所有真实属于该类别的样本中，被模型正确识别出来的比例。
  - 公式：`Recall = TP / (TP + FN)`
- **F1-score（F1 分数）**：Precision 和 Recall 的调和平均数。
  - 公式：`F1 = 2 * Precision * Recall / (Precision + Recall)`

其中：
- `TP`（True Positive，真正例）：真实类别和预测类别都为当前类别的数量。
- `FP`（False Positive，假正例）：被预测为当前类别，但真实类别不是它的数量。
- `FN`（False Negative，假负例）：真实属于当前类别，却被预测成其他类别的数量。

如果分母为 0，代码应显式处理，例如返回 `0.0`，以避免除零错误。

## 4. 本次实际结果

| 类别 | Precision | Recall | F1-score | 支持数（Support） |
|---|---:|---:|---:|---:|
| ants | 0.943 | 0.943 | 0.943 | 70 |
| bees | 0.952 | 0.952 | 0.952 | 83 |

两类的 Precision 与 Recall 恰好相等，是因为混淆矩阵中两个方向的误判数量都是 4；这不是所有分类问题都会出现的情况。

训练日志中最佳验证准确率为 `0.948`（约 94.8%）。验证集只有 153 张图片，这个结果说明当前实验运行良好，但不能保证模型在所有真实图片上都能达到同样准确率。

## 5. 用混淆矩阵手动计算指标

项目中的 `cm` 是混淆矩阵，类别 `i` 对应 `class_names[i]`：

```python
for i, name in enumerate(class_names):
    # 当前类别的真正例
    tp = cm[i, i].item()

    # 真实属于当前类别的样本总数（该行之和）
    actual = cm[i].sum().item()

    # 被预测为当前类别的样本总数（该列之和）
    predicted = cm[:, i].sum().item()

    precision = tp / predicted if predicted > 0 else 0.0
    recall = tp / actual if actual > 0 else 0.0
    f1 = (
        2 * precision * recall / (precision + recall)
        if precision + recall > 0 else 0.0
    )

    print(
        f"{name}: Precision={precision:.3f}, "
        f"Recall={recall:.3f}, F1={f1:.3f}"
    )
```

这里要记住：
- `cm[i, i]`：当前类别预测正确的数量。
- `cm[i].sum()`：真实属于当前类别的数量。
- `cm[:, i].sum()`：模型预测为当前类别的数量。

## 6. 这一步的工程意义

评估流程可以拆成三个职责：

1. **模型推理**：输入图片，输出 `logits`（模型输出的原始分类分数）。
2. **结果收集**：收集真实标签 `y_true` 和预测标签 `y_pred`。
3. **指标计算**：根据预测结果计算混淆矩阵、Precision、Recall 和 F1。

把这些职责分开后，换用其他模型时通常可以继续复用同一套评估流程。

## 7. 下一步

可以使用 `scikit-learn` 的 `classification_report()` 和 `confusion_matrix()` 生成标准分类评估报告，减少手写指标计算代码，并将评估流程整理成可单独复用的函数或 `evaluate.py`。
