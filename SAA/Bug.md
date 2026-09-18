---
tags:
  - 项目
  - SAA
---

#开发 
1.文本渲染问题
现在对话,pdf导出都是一样的渲染,并且不稳定
~~2.班级显示问题~~


3.登录不报存 直接打开网站具体栏目不会拦截
🟡 中等问题
4. 插件失败时裸 JSON 会暴露给用户 chat.py 第119-120行：如果 GPT-5-Nano 返回的 JSON 解析失败（dispatch_plugin 返回 None），原始的 {explain, words} JSON 字符串会原样显示给用户。

5. 异常静默吞没（多处）

python
# database.py:249  create_class()
except Exception: pass  # class_teachers 插入失败被忽略

# database.py:268, 297  get_classes()  
except Exception: pass  # class_teachers 查询失败被忽略
如果数据库损坏或表结构异常，这些关键错误会被完全隐藏。

6. 前端 apiFetch 对所有请求强设 Content-Type: application/json

javascript
// line 638-640
if (!(options.body instanceof FormData)) {
    headers['Content-Type'] = 'application/json';
}
DELETE 请求（无 body）也会被设上 Content-Type，虽然大多数后端不报错，但不规范。

7. PythonRequest 模型与端点不匹配 models.py:85-86 定义 PythonRequest 只有 code 字段，但 /api/re_python 和 /api/get_time 两个测试端点完全忽略了它。这些是遗留测试端点，可以清理。

🟢 轻微问题
8. ai.py 遗留的孤注注释

python
# line 1:  """渲染问题未统一"""    ← 这行应是模块文档字符串，但内容是待办备忘
# line 129: #没有修改              ← 无意义的注释
9. protectedRoutes 中 /pytest 重复

javascript
// line 2934
const protectedRoutes = ['/dashboard','/chat', '/errorbook',"/pytest",'/classes',
                          '/about','/generate','/quiz-lib','/search','/pytest','/notebook'];
//                                                                         ^^^^^^ 重复
10. backend/routes/image.py 已删除 这个文件被删除了，但 main.py 中也已移除了对应的 import。前端也不引用 /api/image/。当前一致，无 bug，只是 git 中有删除记录


现有的
~~1.分析页布局混乱,不能拍照上传

2.修改<aside class="sidebar"为透明>


4.导出模块会不正常渲染


![[Pasted image 20260725170816.png]]

[[SAA项目清单]]
