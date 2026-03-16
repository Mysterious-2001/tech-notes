@propetry
@dataclass
@classmethod

```python
dict.setdefault(key, default_value)
```
参数说明：

- `key`：要查找的键
- `default_value`：如果 key 不存在时，插入的默认值

返回值：

- 如果 `key` 存在 → 返回对应的 value
- 如果 `key` 不存在 → 插入 `key: default_value`，并返回 `default_value`


# `sys.path` 是 **Python 用来查找模块（module）和包（package）的路径列表**。  
当你执行 `import xxx` 时，Python 就会按顺序在 `sys.path` 里的目录中查找这个模块。
简单说：
> **sys.path = Python 的模块搜索路径列表**
> `sys.path` 是 Python 解释器在执行 `import` 时用来查找模块和包的路径列表。

特点：

- 是 `list`
    
- 按顺序查找
    
- 可以动态修改
    
- 包含
    

当前目录  
PYTHONPATH  
标准库  
site-packages