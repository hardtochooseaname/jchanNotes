# conda常用命令

以下是一些常用的 Conda 命令，按照不同的功能分类列出，方便你查阅和使用：

------

### 🛠️ **环境管理**

1. **创建环境：**

   > 创建的环境与目录无关，环境激活过后，不管当前工作目录在哪，都会把激活的那个环境当作默认的python环境

   ```bash
   # -n 名称模式（默认方式，方便环境复用）
   conda create -n myenv python=3.9
   
   # -p 路径模式（方便环境部署或分享）
   conda create -p myenv python=3.9
   
   # 然后在环境内配置pip镜像源
   pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
   pip config set global.trusted-host pypi.tuna.tsinghua.edu.cn
   ```

   - `-n myenv`：指定环境名称为 `myenv`
   - `python=3.9`：指定 Python 版本

2. **查看已有环境：**

   ```bash
   conda env list
   ```

   或者

   ```bash
   conda info --envs
   ```

3. **激活环境：**

   ```bash
   conda activate myenv
   ```

4. **退出环境：**

   ```bash
   conda deactivate
   ```

5. **删除环境：**

   ```bash
   conda remove -n myenv --all
   ```

   - `--all`：删除整个环境

------

### 📦 **包管理**

1. **搜索包：**

   ```bash
   conda search numpy
   ```

2. **安装包：**

   ```bash
   conda install numpy
   ```

   - 安装特定版本：

     ```bash
     conda install numpy=1.21.2
     ```

   - 从指定渠道安装：

     ```bash
     conda install -c conda-forge numpy
     ```

3. **更新包：**

   ```bash
   conda update numpy
   ```

   - 更新 conda 本身：

     ```bash
     conda update conda
     ```

4. **卸载包：**

   ```bash
   conda remove numpy
   ```

5. **查看已安装包：**

   ```bash
   conda list
   ```

   - 查看特定包信息：

     ```bash
     conda list numpy
     ```

------

### 📝 **环境导出与恢复**

1. **导出环境：**

   ```bash
   conda env export > environment.yml
   ```

   - 生成 `environment.yml` 文件，便于分享或备份

2. **从环境文件创建：**

   ```bash
   conda env create -f environment.yml
   ```

3. **克隆环境：**

   ```bash
   conda create --name newenv --clone oldenv
   ```

------

### 🚀 **清理操作**

1. **清理冗余包：**

   ```bash
   conda clean --all
   ```

   - 删除缓存包和未使用的环境

2. **仅清理包缓存：**

   ```bash
   conda clean --packages
   ```

3. **清理索引缓存：**

   ```bash
   conda clean --index-cache
   ```

------

### 🧩 **常见问题处理**

1. **修复环境：**

   ```bash
   conda update --all
   ```

   - 更新环境中的所有包

2. **修复权限错误：**

   ```bash
   conda update -n base -c defaults conda
   ```

   - 更新 base 环境中的 conda

------

如果有更多问题或遇到错误，随时联系我！😊