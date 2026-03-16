# 函数是对象
python里一切都是对象，函数也是。意味着函数可以：作为返回值，作为参数传入，赋值给变量，放进容器

def的时候到底做了什么：
1. 创建一个函数对象
2. 把函数名绑定到这个函数对象

即函数定义不是纯编译期概念，而是运行时构造对象并绑定名字。
# 参数传递机制

Python 的参数传递不是简单的“值传递”或“引用传递”教科书二分法，而是：

> **对象引用按值传递**  
> 或者更 Pythonic 地说：**函数调用时进行名字绑定**

本质：
传的是可变对象还是不可变对象。如果是不可变对象，函数内部改的是局部名字的绑定（新对象），不是外部变量本身。如果是可变对象，那么注意函数修改的是局部名字绑定到对象还是原地修改了传入的可变对象
# 默认参数和可变参数

默认参数的坑：
> 默认参数在函数定义时求值，而且通常只求值一次。

```python
def append_item(item, lst=[]):  
lst.append(item)  
return lst  
  
print(append_item(1))  
print(append_item(2))  
print(append_item(3))
```
输出
```python
[1]
[1, 2]
[1, 2, 3]
```
也就是说，`lst=[]` 这个列表对象，在 `def` 执行时就创建好了。  
以后每次不传 `lst`，都会复用同一个列表对象。

**深一层理解：默认值是函数对象的一部分**
函数对象会保存默认参数信息。
你可以理解为：
- `def` 时把默认值记录到函数对象里
- 调用时如果没传对应参数，就取那里记录的对象
所以默认参数是“定义期绑定”，不是“调用期重新计算”。

# 位置参数、关键字参数、仅限位置、仅限关键字
python的参数类型汇总：
- 位置参数
- 关键字参数
- 默认参数
- 可变位置参数 `*args`，`*args` 会把多余的位置参数收集成一个元组。
- 可变关键字参数 `**kwargs`，`**kwargs` 会把多余的关键字参数收集成字典。
- 仅限位置参数 `/`
- 仅限关键字参数 `*`

`*` 和 `**` 不只用于定义，也用于调用
```python
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
print(add(*nums

def greet(name, age):  
print(name, age)  
  
info = {"name": "Alice", "age": 20}  
greet(**info)
```
等价于
```python
print(add(1, 2, 3))

greet(name="Alice", age=20)
```
# 返回值与多返回值本质
Python 函数始终只返回一个对象。  
这里返回的是一个 tuple。

Python 函数如果没有显式 `return`，默认返回 `None
Python是没有真正意义上的void函数

# 作用域、闭包、LEGB

## 作用域规则
L  Local        当前函数
E  Enclosing    外层函数
G  Global       模块全局
B  Builtin      内置作用域


## 闭包

## 闭包定义

> **函数 + 捕获的外部变量 = 闭包**

换句话说：
函数记住了它创建时的环境。


`global` vs `nonlocal`

| 关键字      | 作用       |
| -------- | -------- |
| global   | 修改全局变量   |
| nonlocal | 修改外层函数变量 |
一个闭包的例子
```python
def counter():
    count = 0

    def inc():
        nonlocal count
        count += 1
        return count

    return inc

c = counter()

print(c())
print(c())
print(c())
```
工程上的使用场景：
配置函数


函数工厂

装饰器 = 闭包+wrapper
#  函数注解与类型提示

# 高阶函数、lambda

如果一个函数：接受函数作为参数或者返回函数
它就是高阶函数

lambda就是Python 的匿名函数，使用场景是创建一次性的小函数
# 装饰器

# 生成器函数与yield

# 异步函数

