面向对象本质上不是“会写 class”，而是：

- 把数据和行为组织到一起
    
- 让代码更容易抽象、复用、扩展
    
- 用“对象”去描述现实或业务中的实体、状态和动作

# 类定义的时候发生了什么

```python
class Student:
    school = "No.1 School"

    def __init__(self, name):
        self.name = name

    def study(self):
        print(self.name, "is studying")
        
```

Python 会执行 `class` 代码块，把里面定义的名字收集起来，再构造出一个类对象。
也就是说，类定义是“执行出来的”。
类体里的这些名字：
- `school`
- `__init__`
- `study`    
最终会进入类的命名空间里。

# 实例化的时候发生了什么
## 实例对象是怎么来的

```python
s = Student("Alice")
```

表面上是“调用类创建对象”，本质上是：
1. 先创建一个新的实例对象
2. 再调用 `__init__` 初始化它
3. 把这个对象赋值给 `s`

# self到底是什么

实例方法本质上还是函数，只不过当你通过对象调用时，Python 会自动把当前对象传进去。

# 属性查找规则：Python 对象模型的核心
大致规则是：
1. 先找实例自己的属性    
2. 实例没有，再找类属性
3. 类没有，再沿继承链往父类找
4. 都没有，抛 `AttributeError`