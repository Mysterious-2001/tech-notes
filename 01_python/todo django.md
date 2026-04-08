## Django 模型注册机制
### 1. 为什么需要注册？
想象一下，Django 就像一个大仓库管理员，它需要知道：

- 项目里有哪些模型（数据库表）
- 这些模型属于哪个应用
- 如何管理这些模型的生命周期
### 2. Django 是如何找到模型的？
当 Django 启动时，它会按照以下步骤查找模型：

1. 读取 INSTALLED_APPS → 从 settings.py 中读取已注册的应用列表
2. 导入应用 → 逐个导入这些应用
3. 查找 models.py → 在每个应用目录下查找 models.py 文件
4. 注册模型 → 将 models.py 中定义的模型类自动注册到 Django 的注册表中
### 3. 你的项目的特殊情况
在你的项目中，结构有点特殊：

```
MysqlModels/           ← 这是在 INSTALLED_APPS 中注册的应用
  └── models.py        ← Django 会自动查找这个文件
      └── 只是从 ace.infra.django.models 导入模型

ace/infra/django/
  └── models.py        ← 模型实际定义在这里，但是这个目录不在 
  INSTALLED_APPS 中
```
问题所在 ：Django 只在 MysqlModels/models.py 里找模型定义，但你的模型实际定义在另一个地方。

### 4. app_label 的作用
当你在模型的 Meta 类中添加 app_label = 'MysqlModels' 时，就相当于告诉 Django：
 "嘿，虽然我定义在 ace/infra/django/models.py，但我实际上是属于 MysqlModels 应用的，请把我注册到那里去！"
### 5. 底层发生了什么？
在 Django 源码中（ django/db/models/base.py 的 ModelBase 元类（就是那个 __new__ 方法）会：

1. 检查模型定义位置
2. **如果模型不在已注册应用的 models.py 中
3. 就查看 Meta 类中是否有 app_label
4. 如果有，就用这个 app_label 来注册模型
5. 如果没有，就报错 （就是你遇到的那个错误）
### 6. 类比理解
可以把 Django 想象成一个学校：

- 应用（App） = 班级
- 模型（Model） = 学生
- INSTALLED_APPS = 学校的班级名单
- app_label = 学生证上的班级信息
如果一个学生不在班级名单里的班级上课，但学生证上写明了属于某个班级，学校还是会承认他是那个班级的学生。