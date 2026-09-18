---
tags:
  - 工具
  - conda
  - 环境
---

# conda 常用基础指令速查

> 适用:Anaconda / Miniconda,Windows(PowerShell / Anaconda Prompt)
> conda 两大功能:**环境管理**(隔离不同项目的 Python 环境)和**包管理**(安装、更新、卸载第三方库)

---

## ⚡ 速查表(最常用命令,先看这张)

| 想做什么 | 命令 |
| --- | --- |
| 看有哪些环境 | `conda env list` |
| 新建环境 | `conda create -n 名字 python=3.11` |
| 进入环境 | `conda activate 名字` |
| 退出环境 | `conda deactivate` |
| 删环境 | `conda env remove -n 名字` |
| 复制环境 | `conda create -n 新名 --clone 旧名` |
| 装包 | `conda install 包名`(版本用 `包名=1.26`) |
| 卸包 | `conda remove 包名` |
| 升级包 | `conda update 包名` |
| 看已装包 | `conda list` |
| 搜包版本 | `conda search 包名` |
| 配镜像 | `conda config --add channels 镜像地址` |
| 看渠道 | `conda config --show channels` |
| 清理缓存 | `conda clean --all` |
| 查版本 | `conda --version` |

> 详细说明和常见坑见下文各节。

---

## 0. 一个概念:为什么要用 conda

不同的项目可能依赖**不同版本的 Python 或不同版本的库**(比如 A 项目要 numpy 1.x,B 项目要 2.x)。conda 可以为每个项目建一个**独立环境**,互不干扰。装坏了一个环境,直接删掉重建,不影响别的环境。

**建议:永远不要在 base(默认环境)里乱装包**,每个项目建一个自己的环境。

---

## 1. 环境管理

### 查看所有环境

```bash
conda env list          # 列出所有环境,带 * 的是当前所在环境
conda info --envs       # 等价写法
```

### 创建环境

```bash
conda create -n 环境名 python=3.11          # 创建一个叫"环境名"的环境,指定 Python 版本
conda create -n myenv python=3.11 numpy pandas   # 创建的同时顺便装一些包
```

- `-n` 是 `--name` 的简写;
- 常用版本:`python=3.10`、`python=3.11`、`python=3.12`,按项目需求选。

### 激活 / 退出环境

```bash
conda activate 环境名     # 进入环境
conda deactivate          # 退出当前环境,回到 base
```

- 激活成功后,命令行提示符前面会出现 `(环境名)`;
- **Windows PowerShell 里第一次用 activate 可能报错**:先执行一次 `conda init powershell`,然后**关掉终端重开**;或直接用开始菜单里的 **Anaconda Prompt**。

### 删除环境

```bash
conda env remove -n 环境名        # 推荐写法
conda remove -n 环境名 --all      # 等价写法
```

### 克隆环境

```bash
conda create -n 新环境名 --clone 旧环境名    # 完整复制一个环境
```

### 导出 / 导入环境(复现项目)

```bash
conda env export > environment.yml       # 把当前环境所有包导出到文件
conda env create -f environment.yml      # 按文件重建一个完全相同的环境
```

---

## 2. 包管理

> 以下命令都要**先激活目标环境**(`conda activate 环境名`),否则装到 base 里去了。

### 安装包

```bash
conda install numpy                     # 安装最新版
conda install numpy=1.26                # 安装指定版本(用 = 号)
conda install numpy pandas matplotlib   # 一次装多个
conda install -c conda-forge 包名       # 从指定渠道装(conda-forge 社区渠道,包更全)
```

### 卸载包

```bash
conda remove numpy         # 或 conda uninstall numpy
```

### 更新包

```bash
conda update numpy         # 更新某个包
conda update --all         # 更新当前环境所有包(谨慎,可能引起版本冲突)
```

### 查看 / 搜索包

```bash
conda list                 # 列出当前环境已安装的所有包
conda list -n 环境名       # 列出指定环境(不激活也能看)
conda search numpy         # 搜索可安装的版本
```

---

## 3. 配置国内镜像(提速关键)

conda 默认从国外服务器下载,很慢。配置清华镜像后快很多。

### 方法一:命令行配置(推荐)

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --set show_channel_urls yes     # 下载时显示来源,方便排查
```

### 方法二:直接改配置文件 `.condarc`

Windows 位置:`C:\Users\你的用户名\.condarc`,内容:

```yaml
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

### 查看 / 删除渠道

```bash
conda config --show channels           # 查看当前渠道
conda config --remove channels 渠道名   # 删除某个渠道
```

> conda 装不了的包,在**激活环境内**用 pip 装,同样可以换国内源(见库里的 `pip 源.md`):
> `pip install 包名 -i https://pypi.tuna.tsinghua.edu.cn/simple`

---

## 4. conda 自身维护

```bash
conda --version        # 查看 conda 版本
conda update conda     # 更新 conda 本身
conda info             # 查看 conda 概况(版本、路径、渠道等)
conda clean --all      # 清理下载缓存和旧包,释放磁盘空间(建议偶尔执行)
```

---

## 5. 常用操作流程(新手照着做)

**新开一个项目:**

```bash
conda create -n myproj python=3.11     # 1. 建环境
conda activate myproj                  # 2. 激活
conda install numpy pandas             # 3. 装依赖
python main.py                         # 4. 跑代码
conda deactivate                       # 5. 用完退出
```

**项目不用了:**

```bash
conda deactivate
conda env remove -n myproj             # 整个环境删掉,干净利落
```

---

## 6. 常见坑

1. **包装错环境**:先 `conda activate` 确认提示符前面的环境名,再 install;不确定就用 `conda list -n 环境名` 检查。
2. **PowerShell 无法 activate**:执行 `conda init powershell` 后重开终端。
3. **下载慢 / 卡住**:配置清华镜像(第 3 节),或 `conda clean --all` 后重试。
4. **conda 与 pip 混用**:优先用 conda 装;conda 找不到的包再 pip。混装顺序不当时可能引起依赖冲突,报错就重建环境。
5. **别乱动 base**:base 环境装太多包容易引发版本冲突,而且删不掉(只能重装 Anaconda)。

> 相关:[[pip 源]] [[python tips]] [[硬技能（必须会）&&  指令技巧]]
