---
tags:
  - 项目
  - SAA
---

1.单词卡
已经实现了

2.思考大问题
原本就是围绕AI对话,很多功能都可以只用AI对话板块+保存板块
是否将大部分功能,类似AI出题,直接打包到插件里,
优点1.更核心通用,更能统一输出

缺点:可以引入相同程序而在不同页面有相同效果,研究成本低,并且功能明显可用

+插件指南
0. 编写json输出结构
1. 写插件处理器
新建 backend/services/plugins/math_plugin.py：

```
python
"""数学公式插件示例"""
import re

def handle(reply_text: str, message: str) -> dict | None:
    # 检测回复中是否有数学表达式
    if '\\frac' not in reply_text and 'x=' not in reply_text:
        return None  # 未触发，前端不渲染

    # 提取公式
    formulas = re.findall(r'\$([^$]+)\$', reply_text)
    if not formulas:
        return None

    return {
        "type": "math_card",
        "formulas": formulas,
    }
```

2. 注册插件
编辑 backend/services/plugins/__init__.py：
```
python
from .word_plugin import handle as word_handler
from .math_plugin import handle as math_handler   # ① 新增导入

_PLUGINS = {
    "word": word_handler,
    "math": math_handler,                          # ② 注册处理器
}

_PLUGIN_OPTIONS = [
    {"key": "word", "label": "单词卡", "icon": "📖"},
    {"key": "math", "label": "数学公式", "icon": "📐"},  # ③ 前端下拉选项
]
```


  3.在ai.py增加提示词

4. 前端渲染插件数据
在 frontend/index.html 的聊天消息模板中添加对应的渲染逻辑\


新增学习计插件,
与新页面学习计划<--放在学习看板
学习计划插件
功能:定时提醒学习 time 库+稳定的任务 -->提醒方式暂定为右下角弹窗
会添加在学习看板目录,可以修改   
暂定新增
1.个人分析报告,参与学习计划

json文件覆盖
explain

task-
	标签
	描述
 - {list} 某日 某小时 干什么 描述

目前就
类热力图,30天内数据
每日 每天时间段会有
┌────────────────────────────┬────────────────┐
│◁ 2026年7月            ▷   │ 2026-07-23   + │
├────────────────────────────┼────────────────┤
│ 一 二 三 四 五 六 日        │ 📝 复习数学      │
│         1  2  3  4  5      │    30 分钟  ✎ ✕ │
│  6  7  8  9 10 11 12      │                │
│ 13 14 15 16 17 18 19      │ 📝 背单词        │
│ 20 21 22[23]24 25 26      │    15 分钟  ✎ ✕ │
│ 27 28 29 30 31            │                │
│                 ·         │ 当天暂无安排      │
└────────────────────────────┴────────────────┘

新增
---
作文批改功能 英语
评分,大作文25,小作文15
0.整体评价,打分
1.语法错误英语
	语法错误修改/错别字病句 列表
2针对性评价
	好词 
	好句 
	修改建议
3.针对性推荐
	好词
	好句
4.示范作文
新增二次调用,第一次生成评析,二次生成范文
缺点:增长了生成时间

[[SAA项目清单]]
