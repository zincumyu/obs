---
tags:
  - 工具
  - Python
---


1.super().__init__()类
	_call_
构造函数,调用父类的函数,多继承
```
class A:
    def __init__(self):
        print("A")

class B:
    def __init__(self):
        print("B")

class C(A, B):
    def __init__(self):
        super().__init__()   # ⚠️ 只调一个
```

[[「继承 + super() 调用父类构造器」示例]] [[pip 源]]
