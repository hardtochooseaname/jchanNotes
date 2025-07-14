# Python Poetry

### Python Poetry 的用法详解

[官方文档]: https://python-poetry.org/docs/basic-usage/

#### Poetry 简介

Poetry 是一个现代化的 Python 包管理工具，旨在简化 Python 项目的创建、依赖管理和发布。相较于传统的 `pip`，Poetry 提供了更直观、更强大的功能，使得 Python 项目的管理变得更加高效。

#### 安装 Poetry

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

#### 创建一个新的项目

```bash
# step1：创建新项目
poetry new my_project # 创建一个my_project目录，这个目录就是新项目的项目根目录，里面有一些poetry预先创建的配置文件

# step2：修改项目配置信息
cd my_project
vim pyproject.toml 

# step3：换国内源
poetry source add --priority=primary mirrors https://pypi.tuna.tsinghua.edu.cn/simple/
```

#### 查看依赖

- 查看所有依赖

```bash
poetry show
```

- 查看指定依赖（是否安装）

```sh
poetry show <pkg-name>
```

#### 安装依赖

- 根据 `pyproject.toml` 文件中的配置，安装所有所需的依赖（一般git clone的项目，就需要这样**初始化虚拟环境**以及安装项目依赖）

```bash
poetry install
```

- 安装指定依赖

```sh
poetry add <pkg-name>
```

#### 虚拟环境

Poetry 会为每个项目创建一个独立的虚拟环境，以隔离项目的依赖。

1. 激活虚拟环境


```bash
# version < 2.0.0
poetry shell

# version >=2.0.0
eval $(poetry env activate)
```

2. （在虚拟环境中）退出虚拟环境:

```bash
exit
```

3. 查看虚拟环境信息

```bash
poetry env list # 列出所有虚拟环境（范围在当前目录或者父目录中）
portry env info # 打印当前位置激活的虚拟环境信息
```

4. 删除虚拟环境

```bash
poetry env remove <env-name> # 删除指定的虚拟环境
poetry env remove --all # 删除当前位置能env list出来的所有虚拟环境
```

#### 更新依赖

```bash
poetry update
```

这条命令会更新所有可更新的依赖。

#### 锁定依赖

```bash
poetry lock
```

这条命令会将当前的依赖锁定到一个特定的版本，以便在不同的环境中保持一致。

#### 发布项目

```bash
poetry publish
```

这条命令会将项目发布到 PyPI（Python Package Index）上。

#### 其他常用命令

- **生成 requirements.txt:** `poetry export -f requirements.txt`
- **运行测试:** `poetry run pytest`
- **生成文档:** `poetry run sphinx-build docs build`

#### `pyproject.toml` 文件

这个文件是 Poetry 的配置文件，包含了项目的所有配置信息，例如：

- **依赖:** 项目所依赖的库及其版本。
- **虚拟环境:** 虚拟环境的配置。
- **构建系统:** 构建项目的配置。

#### 示例 `pyproject.toml` **文件**

```toml
[tool.poetry]
name = "my_project"
version = "0.1.0"
description = ""
authors = ["Your Name <your.email@example.com>"]

[too1l.poetry.dependencies]
python = "^3.7"
requests = "^2.28.1"

[tool.poetry.dev-dependencies]
pytest = "^7.1.2"
```

### poetry.lock 和 pyproject.toml 的区别与作用

#### pyproject.toml

- **项目配置的主文件：** 它包含了项目的元数据，如项目名称、版本、作者、依赖的版本范围等。
- **版本范围：** 这里的依赖版本通常是一个范围，比如 `requests >= 2.28.1`，表示要求 `requests` 包的版本大于等于 2.28.1。这个范围为依赖更新提供了灵活性。
- **可修改性高：** 开发者可以随时修改这个文件，调整依赖的版本范围。

#### poetry.lock

- **依赖的快照：** 它记录了项目在某个特定时刻所依赖的每个包的精确版本号。
- **版本锁定：** 这里的版本是固定的，比如 `requests==2.28.1`，表示必须使用 2.28.1 版本。
- **不可随意修改：** 一般情况下，不建议手动修改这个文件，因为它会破坏版本的锁定。

#### 为什么需要两个文件

- 灵活性与稳定性的平衡：
  - `pyproject.toml` 提供了对依赖的灵活配置，允许开发者在开发过程中调整依赖版本。
  - `poetry.lock` 则提供了稳定的依赖环境，确保项目在不同环境下的一致性。
- 版本控制:
  - `pyproject.toml` 可以提交到版本控制系统，但通常只提交 `poetry.lock`。
  - `poetry.lock` 保证了团队成员使用的依赖版本一致，避免了因为依赖版本不同导致的冲突。
- 自动化工具:
  - CI/CD 工具可以根据 `poetry.lock` 文件来构建和部署项目，确保每次构建的依赖环境都是一致的。

#### 总结

- **`pyproject.toml`** 是项目的配置文件，定义了项目的元数据和依赖的版本范围。
- **`poetry.lock`** 是项目的依赖快照，记录了项目的精确依赖版本。
- **两个文件的关系：** `pyproject.toml` 定义了依赖的范围，`poetry.lock` 则锁定了具体的版本。
- **作用：** `pyproject.toml` 提供了灵活性，`poetry.lock` 提供了稳定性。

**形象比喻：** 你可以将 `pyproject.toml` 看作是一个食谱，它列出了制作菜品的食材和大致的量。而 `poetry.lock` 则是一份购物清单，它精确地列出了每种食材的品牌、规格和数量。有了这份购物清单，无论是谁按照这个食谱做菜，做出来的菜品都应该是一样的。

**何时修改哪个文件？**

- **修改 `pyproject.toml`：** 当你需要添加新的依赖、修改依赖的版本范围时。
- **修改 `poetry.lock`：** 一般不建议手动修改，而是通过 `poetry update` 命令来更新。

### peotry换国内源

```bash
# tian'j--priority=primary是要把清华源的优先级设置得比官方源更高
poetry source add --priority=primary mirrors https://pypi.tuna.tsinghua.edu.cn/simple/

# 换源后pyproject.toml文件会发生改变，需要重新生成poetry.lock文件
poetry lock --no-update
```



