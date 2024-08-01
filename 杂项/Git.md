# Git的概念与原理

## 三个基本概念

- **工作区**：就是项目文件所在的当前目录。
- **暂存区**：英文叫 stage 或 index。一般存放在 .git 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。add了又尚未commit的修改都暂存在index中。
- **版本库**：工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库，有时候可以把器看作“本地仓库”。版本控制的核心文件和数据信息都在此处。commit就是提交到版本库中，版本库中存放了所有的历史版本。

![](https://www.runoob.com/wp-content/uploads/2015/02/1352126739_7909.jpg)

## 一些其他重要概念

- **commits**：每次commit提交一次修改，而这个修改的快照就是一个commit。版本控制就是通过一系列commits来完成的。通过定位到时间线上的某个commit，就可以定位到某个历史版本，即该commit完成后的版本。每次修改的commit都会被创建为一个提交对象存放到版本库中，并且使用提交对象的哈希值用作commit（版本）的索引和唯一标识。
- **分支branches**：几乎每一种版本控制系统都以某种形式支持分支，一个分支代表一条独立的开发线。使用分支意味着你可以从开发主线上分离开来，然后在不影响主线的同时继续工作。分支的本质是指对项目中某个特定的提交（commit）的**引用**，是一种可移动的**指针**，指向了某个历史版本。在某个commit创建分支，相当于创建一个新的指针，做出的新的修改都会commit到到分支这条新的开发线上。有些操作既能在commit上执行也能在branch上执行，在branch上执行的时候有时候就相当于在其指向的commit上执行。（this is how we use 指针&引用）

        <img src="..\images/2024-02-03-10-23-52-image.png" title="" alt="" width="308">

- **默认分支（主分支）：** 当你初始化一个新的Git仓库时，会自动创建一个默认的分支，通常被命名为 `master`。这个分支指向你的第一个提交，即初始提交（initial commit）。

- **HEAD**：在Git中，`HEAD` 是一个指向当前所在分支的引用，也可以指向某个具体的提交（commit）。它是一个特殊的指针，用于标识你正在工作的本地仓库中的当前位置。
  
  - HEAD 表示当前版本
  
  - HEAD^ 上一个版本
  
  - HEAD^^ 上上一个版本
  
  - HEAD^^^ 上上上一个版本
  
  - 以此类推...
  
  可以使用 ～数字表示
  
  - HEAD~0 表示当前版本
  
  - HEAD~1 上一个版本
  
  - HEAD^2 上上一个版本
  
  - HEAD^3 上上上一个版本
  
  - 以此类推...

# Git仓库与配置

> **创建或克隆一个仓库是使用git来进行版本控制的第一步** 

## git init：创建仓库并初始化

可以直接指定仓库创建的目录，不指定的话就默认创建在当前目录

```git
git init <newrepo>
```

## git clone：直接克隆一个仓库到本地

同样可以指定或不指定克隆的目录

```git
git clone <repo> <directory>
```

## Git配置

Git的配置是指设置和管理Git的各种属性和选项，包括用户信息、编辑器偏好、别名等。Git配置文件保存在每个仓库的`.git/config`文件中，也可以在全局和系统范围内配置。

`git config` 是用于配置Git的命令。以下是一些常用的`git config`命令及其用法：

1. **全局配置用户信息：**
   
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```
   
   这会在全局范围内设置用户的姓名和邮箱地址。这些信息会在每个提交中作为作者信息记录。

2. **检查配置信息：**
   
   ```bash
   git config --list
   ```
   
   这会列出当前配置的所有信息。

3. **配置默认文本编辑器：**
   
   ```bash
   git config --global core.editor "your-editor"
   ```
   
   用于设置在Git中打开提交消息的文本编辑器。

4. **配置别名：**
   
   ```bash
   git config --global alias.co checkout
   ```
   
   这会将 `git co` 映射为 `git checkout` 的别名。你可以创建其他有用的别名，简化常用的Git命令。

5. **配置换行符处理：**
   
   ```bash
   git config --global core.autocrlf true
   ```
   
   用于在不同操作系统之间处理换行符的方式，例如在Windows上自动转换CRLF为LF。

6. **配置彩色输出：**
   
   ```bash
   git config --global color.ui true
   ```
   
   用于启用彩色输出，使得Git在终端中更易读。

7. **配置忽略文件：**
   
   ```bash
   git config --global core.excludesfile ~/.gitignore_global
   ```
   
   用于指定一个全局的`.gitignore`文件，定义了不希望Git跟踪的文件和目录。

可以通过不同的`--global`、`--local`、和`--system`参数来选择配置的范围。

- `--global` 表示全局配置，针对用户的整个系统环境，配置信息保存在用户的 home 目录下的 `.gitconfig` 文件中。这些配置对用户在系统上所有 Git 仓库都有效。

- `--local` 针对当前仓库，配置信息保存在仓库的 `.git/config` 文件中。这些配置只对当前仓库有效，对其他仓库不起作用。

- `--system` 针对整个系统，配置信息保存在 Git 安装目录下的 `etc/gitconfig` 文件中。这些配置对系统上的所有用户和所有 Git 仓库都有效。需要管理员权限才能修改系统级别的配置。

git checkout <commit>：回溯到特定历史版本

git checkout HEAD^：回溯到当前版本的上一个版本

# 基本命令

## git reset

该命令用于**回退版本**到指定历史版本/commit，但实际上主要执行的是**移动当前branch指针**的操作，并没有删除或取消任何commit（回退的是指针不是版本）。移动指针到某次commit后，从工作区和暂存区进行的修改和提交都是更新到指针所指的commit，这会开启新的timeline。

1. 仅仅移动branch指针，保留工作区和暂存区现状不做修改。

```bash
git reset --soft <commit>
```

2. 撤销add到**暂存区**中的修改，可以指定撤销哪些文件。（默认--mixed）

```bash
git reset <commit>
```

3. 丢弃**工作区和暂存区**中的修改，直接回退到指定版本/commit的状态。

```bash
git reset --hard <commit>
```

## git status

查看当前仓库的状态，可以查看上次commit之后是否有新的修改。

git status 命令会显示以下信息：

- 当前分支的名称。
- 当前分支与远程分支的关系（例如，是否是最新的）。
- 未暂存的修改：显示已修改但尚未使用 `git add` 添加到暂存区的文件列表。
- 未跟踪的文件：显示尚未纳入版本控制的新文件列表。

## git tag

**标签**通常用于标记项目的重要时刻，比如发布版本、里程碑或者某个特殊的状态。标签是用于标识项目的静态快照。

类似于branch，标签本质上也是一种引用/指针。但与branch不同的是，tag是静态的，一旦创建就不可移动不可更改。因为tag的目的就是标识一个特定的历史时刻。

```bash
git tag <tag-name> -a -m 'info'
```

-a 选项意为"创建一个带注解的标签"。 不用 -a 选项也可以执行的，但它不会记录这标签是啥时候打的，谁打的，也不会让你添加个标签的注解。 执行 git tag -a 命令时，Git 会打开你的编辑器，让你写一句标签注解，就像你给commit写注解一样。因此可以直接在-m后面加上注解。

```bash
git show <tag-name>
```

该命令可以查看关于tag和tag指向commit的详细信息，包括git tag -a加上的注解信息（如果有的话）。

```bash
git tag <commit>
```

给指定commit打标签。



# git remote 与 GitHub

## GitHub公私钥验证

使用GitHub首先就需要配置用于身份验证的ssh公私钥（或者GPG）

#### 密钥配置过程：

```bash
$ ssh-keygen -t rsa -C "youremail@example.com"
```

在本地生成密钥对，并通过-C选项把GitHub账户的邮箱注释到生成的密钥对信息中。

该邮箱用于本地ssh客户端与GitHub服务器通信时，标识本地与哪一个GitHub账户绑定（以哪个账户的身份发起的访问）。

然后将公钥添加到对应GitHub账号中。

#### 密钥的作用：身份验证与加密传输：

1. **是否所有仓库都需要身份验证？**
   
   - 对于公开仓库，通常不需要进行身份验证就可以访问它们。任何人都可以克隆（或拉取）和查看公开仓库的内容，因此不需要身份验证。
   - 对于私有仓库，如果你是仓库的合法用户，你需要进行身份验证才能访问它们。这是因为私有仓库的内容只能被授权用户访问。

2. **即便不需要身份验证，SSH 加密传输是否需要核对公私钥？**
   
   - 在 SSH 连接中，无论是否需要身份验证，都会使用公私钥对进行加密传输，确保通信的安全性。这个过程是在建立连接时进行的，以防止中间人攻击或窃听。因此，在 SSH 连接建立时，确实需要核对公私钥。

#### 核对密钥（身份验证）的过程：

1. **请求访问：** 当你尝试通过 SSH 协议访问 GitHub 上的仓库时，你的 SSH 客户端会向 GitHub 发送一个访问请求，包含用于唯一标识GitHub账户的邮箱。

2. **返回公钥列表：** GitHub 服务器会返回与你的账户关联的公钥列表。这些公钥是你在 GitHub 账户设置中添加的公钥，用于 SSH 密钥身份验证。

3. **匹配公钥：** 你的 SSH 客户端会将本地存储的私钥与返回的公钥进行比较。如果你尝试访问的仓库需要身份验证，那么你的 SSH 客户端会尝试使用本地私钥与 GitHub 返回的公钥进行匹配。

4. **验证身份：** 如果你本地存储的私钥与 GitHub 返回的任何一个公钥匹配成功，GitHub 将验证你的身份。这意味着你拥有与该公钥关联的账户权限。

5. **授予权限：** 如果你的身份验证成功，GitHub 将授予你对所请求资源的访问权限。这包括克隆仓库、拉取代码、推送修改以及访问受保护的资源等操作。



## git remote

### git remote -v

查看当前配置的所有远程仓库

```powershell
PS E:\Learning\notes\LearningNotes> git remote -v
origin  https://github.com/hardtochooseaname/LearningNotes.git (fetch)
origin  https://github.com/hardtochooseaname/LearningNotes.git (push)
```

### git branch -vv

查看本地分支与远程repo的关联

```powershell
PS E:\Learning\notes\LearningNotes> git branch -vv
* master 9056053 [origin/master] add local notes
```

- *master：当前所处master分支

- 9056053：当前分支最新提交的 SHA-1 标识

- [origin/master]：当前分支与远程的origin仓库的master分支相关联

- add local note：当前分支最新提交的commit信息
