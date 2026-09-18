---
tags:
  - 工具
  - Python
---


## 二、最基础示例：人 → 学生

### 1️⃣ 父类：`Person`

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        print("Person 初始化完成")

    def info(self):
        return f"姓名: {self.name}, 年龄: {self.age}"
```

### 2️⃣ 子类：`Student`

```python
class Student(Person):
    def __init__(self, name, age, student_id):
        # 调用父类的 __init__
        super().__init__(name, age)

        # 子类自己的属性
        self.student_id = student_id
        print("Student 初始化完成")

    def info(self):
        return f"{super().info()}, 学号: {self.student_id}"
```

### 3️⃣ 使用

```python
s = Student("张三", 20, "2023001")
print(s.info())
```

### 4️⃣ 输出

```
Person 初始化完成
Student 初始化完成
姓名: 张三, 年龄: 20, 学号: 2023001
```

[[python tips]]
