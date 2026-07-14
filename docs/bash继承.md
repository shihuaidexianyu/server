# Bash 继承

服务器上部分特殊 Bash 命令通过全局 Bash 配置继承到各个用户，例如 `pon` 和 `poff`。默认假设用户使用 Bash 作为 shell，尚未在 Zsh、Fish 等其它 shell 中完整测试。

本文是进阶原理页，不是普通用户的第一入口。普通用户只需要知道如何开启或关闭代理时，应先阅读 [代理网络连接](代理网络连接.md)；只有遇到 `pon`、`poff` 不存在、登录 shell 行为异常、VS Code 远程终端不加载函数等问题时，再阅读本文。

## 全局配置位置

Ubuntu 的全局 Bash 配置通常位于：

```text
/etc/bash.bashrc
```

写入该文件的函数、别名和环境变量，会影响交互式 Bash 用户。

查看文件：

```bash
sudo less /etc/bash.bashrc
```

编辑前建议备份：

```bash
sudo cp /etc/bash.bashrc /etc/bash.bashrc.bak.$(date +%Y%m%d)
```

## pon 和 poff 的原理

`pon` 和 `poff` 本质上是 Bash 函数：

- `pon` 设置代理相关环境变量。
- `poff` 取消代理相关环境变量。

它们只影响当前 shell 以及从当前 shell 启动的子进程，不会自动影响已经运行的其它终端。

代理网络说明参见 [代理网络连接.md](代理网络连接.md)。

## 查看函数是否存在

登录服务器后执行：

```bash
type pon
type poff
```

如果输出显示 `pon is a function`，说明当前 shell 已经加载函数。

也可以检查当前 shell：

```bash
echo "$SHELL"
echo "$0"
```

## 修改全局 Bash 配置的注意事项

全局 Bash 配置影响所有用户，因此要保持轻量、安静、可预测：

- 不要在登录时输出大段文字。
- 不要执行耗时命令。
- 不要自动激活某个 Conda 环境。
- 不要修改用户私有配置。
- 不要在全局配置中写入 token、密码或私有代理订阅。
- 不要假设所有用户目录结构完全相同。

如果需要复杂逻辑，建议在 `/etc/profile.d/` 或独立脚本中维护，然后在 Bash 配置中按需加载。

## 修改后的测试

修改 `/etc/bash.bashrc` 后，先开一个新的 SSH 会话测试，不要关闭当前管理员会话。

检查：

```bash
ssh <server_alias>
type pon
type poff
echo "$SHELL"
```

测试代理函数：

```bash
pon
env | grep -i proxy
poff
env | grep -i proxy
```

确认不会影响普通命令：

```bash
ls
pwd
conda --version
```

## 用户自己的 Bash 配置

用户个人配置通常在：

```text
~/.bashrc
~/.bash_profile
~/.profile
```

用户可以在自己的 `~/.bashrc` 中设置个人别名、Conda 初始化、项目路径等。但不要把只适合个人的配置写入全局文件。

## 常见问题

### pon 命令不存在

可能原因：

- 当前 shell 不是 Bash。
- `/etc/bash.bashrc` 没有被加载。
- 函数定义写入位置不对。
- VS Code Remote SSH 使用的启动方式没有加载交互式 Bash。

先检查：

```bash
echo "$SHELL"
type pon
```

### 登录时出现奇怪输出

可能是全局 Bash 配置中有 `echo`、调试输出或命令报错。需要清理这些输出，因为它们可能影响 VS Code、scp、rsync 等工具。

### 非交互脚本中函数不可用

非交互脚本不一定加载 `/etc/bash.bashrc`。如果脚本依赖 `pon`，应显式说明需要在交互 shell 中先开启代理，或者在脚本中明确加载必要配置。
