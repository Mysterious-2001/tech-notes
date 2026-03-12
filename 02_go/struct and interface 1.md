在 Go 里，**结构体（struct）和接口（interface）是整个语言抽象能力的核心**。如果你真的理解了它们的底层原理，你会发现 Go 的面向对象其实比传统 OOP 更干净。

我会按下面结构讲清楚：

1. 结构体的本质与原理
2. 方法与接收者（值 vs 指针）
3. 接口的底层原理（最关键）
4. 接口的使用模式
5. 结构体 + 接口的高级设计思想

---

# 一、结构体（struct）

## 1️⃣ 结构体的本质

结构体在 Go 里就是：

> 一块连续内存的复合类型

```
typeUserstruct {
    Namestring
    Ageint
}
```

内存示意：

```
| Name指针 | Age |
```

Go 的结构体：

- 没有继承
- 只是数据聚合
- 类型是强类型

创建方式：

```
u1:=User{"Tom",18}
u2:=User{Name:"Tom",Age:18}
u3:=new(User)
u4:=&User{Name:"Tom",Age:18}
```

---

## 2️⃣ 方法（method）

Go 没有类，但可以给类型绑定方法。

```
func (uUser) SayHello() {
fmt.Println("Hello",u.Name)
}
```

这叫 **接收者（receiver）**。

等价于：

```
funcSayHello(uUser)
```

---

## 3️⃣ 值接收者 vs 指针接收者

### 值接收者

```
func (uUser) SetName(namestring) {
u.Name=name
}
```

这是拷贝一份结构体。

### 指针接收者

```
func (u*User) SetName(namestring) {
u.Name=name
}
```

这是直接操作原对象。

### 什么时候用指针？

✅ 结构体大

✅ 需要修改数据

✅ 保持一致性

实际上：

> 90% 场景应该用指针接收者

---

# 二、接口（interface）

这才是 Go 的精髓。

## 1️⃣ 接口的定义

```
typeAnimalinterface {
    Speak()string
}
```

接口只定义行为，不定义数据。

---

## 2️⃣ 最重要的原理

在 Go 里：

> 一个类型只要实现了接口的所有方法，它就自动实现该接口。

不用写：

```
implements Animal
```

这叫：

> 隐式接口

示例：

```
typeDogstruct {}

func (dDog) Speak()string {
return"wang"
}
```

Dog 自动实现 Animal。

---

## 3️⃣ 接口的底层结构

接口本质上是：

```
(type, data)
```

接口变量内部有两个指针：

1. 指向类型信息
2. 指向具体数据

当你写：

```
varaAnimal=Dog{}
```

实际发生：

```
a = (Dog类型信息, Dog实例数据)
```

---

## 4️⃣ nil 接口陷阱

这是 Go 面试高频题。

```
vard*Dog=nil
varaAnimal=d

fmt.Println(a==nil)// false
```

为什么？

因为：

```
a = (Dog类型, nil)
```

接口不为 nil，因为类型不为 nil。

真正为 nil 必须：

```
(type=nil, data=nil)
```

---

# 三、接口的设计思想

Go 的哲学是：

> 面向接口编程，而不是面向实现

例如标准库里的：

- io.Reader
- io.Writer
- error

举例：

```
typeReaderinterface {
    Read(p []byte) (nint,errerror)
}
```

只要实现 Read，就能接入整个 IO 生态。

---

# 四、结构体组合（替代继承）

Go 没有继承，但有组合。

```
typeAnimalstruct {
    Namestring
}

typeDogstruct {
Animal
}
```

这叫：

> 内嵌字段（匿名字段）

效果类似继承：

```
d:=Dog{}
d.Name="旺财"
```

---

# 五、多态示例

```
funcMakeSound(aAnimal) {
fmt.Println(a.Speak())
}
```

调用：

```
MakeSound(Dog{})
MakeSound(Cat{})
```

这就是多态。

---

# 六、接口最佳实践

### 小接口原则

Go 推崇：

> 接口越小越好

例如：

```
typeCloserinterface {
    Close()error
}
```

不要设计：

```
typeBigInterfaceinterface {
    Read()
    Write()
    Close()
    Flush()
}
```

---

# 七、接口类型断言

```
a:=Animal(Dog{})

d,ok:=a.(Dog)
```

底层是检查接口的 type 指针。

---

# 八、结构体 vs 接口总结

|结构体|接口|
|---|---|
|描述数据|描述行为|
|有具体内存|是抽象|
|可以有方法|只声明方法|
|不能多态|实现多态|

---

# 九、高手级理解

### Go 的对象模型本质是：

```
数据 + 函数绑定 + 接口鸭子类型
```

Go 的面向对象不是传统 OOP，而是：

> 组合 + 接口驱动

---

# 十、给你一个进阶思考

在 Go 里：

- 接口由使用方定义
- 实现方不知道自己实现了什么接口

这是一种 **解耦设计哲学**。