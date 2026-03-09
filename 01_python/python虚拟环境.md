`uv` 是 Astral 团队用 Rust 开发的超高速 Python 包 / 环境管理工具，速度比 pip/pipenv/poetry 快几十到上百倍

# 安装

```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
```

# 验证

```shell
uv --version
```

# uv的作用

## 1.全局 Python 版本管理

```shell
#查看所有可安装的Python版本 
uv python list 
#安装指定版本（支持模糊指定，比如3.10会自动装3.10最新小版本） 
uv python install 3.11 
#查看本地已安装的所有Python版本 
uv python list --installed 
#设置全局默认Python版本 
uv python default 3.11 
#卸载指定版本 
uv python uninstall 3.9
```
## 2.项目虚拟环境管理


不需要每次都激活环境，在运行项目的时候，所有命令前面加`uv run`就行，比如`uv run python main.py`，uv 会自动找当前项目的`.venv`环境。

## 3.依赖安装

基于`pyproject.toml`标准
```shell
uv add
uv remove
uv install
uv sync
```

任何命令加`--help`就能看详细参数，不用死记