# Conda 环境

Conda 用于管理 Python、R、命令行工具及其依赖。服务器上建议每个用户把 Conda 安装在自己的用户目录下，并为不同项目创建独立环境，避免互相影响。

## 安装位置

推荐安装到用户目录，例如：

```text
~/miniconda3
```

不要把个人 Conda 安装到系统目录，也不要修改其他用户的 Conda 环境。

## 安装前准备

如果下载安装脚本需要代理，先开启代理：

```bash
pon
```

网络说明参见 [代理网络连接.md](代理网络连接.md)。

## 安装 Miniconda

Miniconda 是较轻量的 Conda 安装方式，适合服务器普通用户自己安装和维护。不要使用 `sudo` 安装 Miniconda，也不要安装到系统目录。

官方 Linux 安装文档：

```text
https://www.anaconda.com/docs/getting-started/miniconda/install/linux-install
```

以下命令适用于当前服务器常见的 Linux x86_64 架构。

进入一个临时目录：

```bash
cd /tmp
```

使用 `curl` 下载最新版安装脚本：

```bash
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

如果没有 `curl`，可以使用 `wget`：

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

可选但推荐：计算安装脚本的 SHA-256，并到官方仓库页面核对：

```bash
sha256sum Miniconda3-latest-Linux-x86_64.sh
```

官方校验值页面：

```text
https://repo.anaconda.com/miniconda/
```

运行安装脚本：

```bash
bash Miniconda3-latest-Linux-x86_64.sh
```

安装过程中按提示操作：

1. 阅读许可协议，输入 `yes` 接受。
2. 安装路径建议使用默认的 `~/miniconda3`。
3. 询问是否初始化 conda 时，建议输入 `yes`，让安装器自动修改 `~/.bashrc`。

如果你不想交互式安装，也可以明确指定安装目录：

```bash
bash Miniconda3-latest-Linux-x86_64.sh -b -p "$HOME/miniconda3"
```

静默安装后手动初始化 Bash：

```bash
"$HOME/miniconda3/bin/conda" init bash
source ~/.bashrc
```

确认安装成功：

```bash
conda --version
conda list
which conda
```

如果安装前开启了代理，安装完成后可以关闭：

```bash
poff
```

## 初始化 Conda

安装完成后，通常需要初始化 Bash：

```bash
conda init bash
```

然后重新登录，或执行：

```bash
source ~/.bashrc
```

确认 Conda 可用：

```bash
conda --version
which conda
```

## 创建环境

为每个项目创建独立环境：

```bash
conda create -n project_name python=3.11
conda activate project_name
```

安装包：

```bash
conda install numpy scipy pandas
pip install package_name
```

原则上优先用 `conda install` 安装二进制依赖复杂的包，再用 `pip` 补充 Conda 中没有的包。

## 查看和删除环境

查看环境：

```bash
conda env list
```

删除不再使用的环境：

```bash
conda env remove -n 环境名
```

删除前确认没有项目仍依赖该环境。

## 导出环境

为了保证实验可重复，建议导出环境文件：

```bash
conda env export > environment.yml
```

从环境文件重建：

```bash
conda env create -f environment.yml
```

如果只想记录手动安装的包，可以使用：

```bash
conda env export --from-history > environment.yml
```

## 缓存和磁盘空间

Conda 包缓存可能占用大量空间。查看缓存：

```bash
conda clean --dry-run --all
```

确认后清理：

```bash
conda clean --all
```

如果用户主目录空间紧张，可以考虑把环境放到 `/data/<username>` 下，但要注意备份和权限。

## 不建议的做法

- 不要长期在 `base` 环境中安装项目依赖。
- 不要使用 `sudo pip install`。
- 不要把 Conda 环境复制给别人后直接当作可重复环境。
- 不要在同一个环境里混装多个项目的大量冲突依赖。

## 常见问题

### conda 命令不存在

检查是否初始化：

```bash
which conda
echo "$PATH"
```

如果安装目录存在但没有进入 PATH，可以临时加载：

```bash
source ~/miniconda3/etc/profile.d/conda.sh
```

`~/miniconda3` 按实际安装位置调整。

### 安装很慢或失败

可能是网络问题。先确认是否需要代理：

```bash
curl ipinfo.io
```

再根据需要执行：

```bash
pon
```

### 激活环境后 Python 不是期望版本

检查：

```bash
which python
python --version
conda info --envs
```

确认当前环境名前面是否有 `*`。
