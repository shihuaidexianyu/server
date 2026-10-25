# Bash 启动与环境继承

先区分“启动文件加载函数”和“子进程继承环境”，再定位 `pon`、`poff` 为什么可用或缺失。以下以函数写在 Ubuntu 全局 Bash 配置中为例；实际函数定义与加载位置取决于部署配置。

普通用户先看 [代理网络连接](代理网络连接.md)。本页供排查 shell、SSH、VS Code 与批处理环境的用户和管理员使用；修改系统文件仅限管理员，并须保留另一个可用会话和恢复副本。

## 当前 shell 与启动方式

```bash
printf '账户默认 shell: %s\n' "$SHELL"
ps -p $$ -o comm=
printf '当前参数标志: %s\n' "$-"
shopt -q login_shell && printf 'login shell\n'
type -t pon
type -t poff
```

`$SHELL` 通常是账户默认 shell，不一定是正在执行命令的 shell；`$-` 含 `i` 才表示交互式 Bash。`shopt` 是 Bash 专属内建命令，先确认当前进程是 Bash。`type -t` 只显示命令类型，避免把函数内的私有代理地址输出到共享日志。

## 启动文件不是全部依次读取

| Bash 启动方式 | 主要读取规则 |
| --- | --- |
| 登录 shell | `/etc/profile`，再从 `~/.bash_profile`、`~/.bash_login`、`~/.profile` 中按顺序读第一个存在且可读的文件 |
| 交互、非登录 shell | 上游 Bash 读 `~/.bashrc`；Ubuntu 的系统版还使用 `/etc/bash.bashrc`，受 `--norc` 等选项影响 |
| 普通非交互脚本 | 通常不读交互 rc，按 `BASH_ENV` 指定的文件加载；带 `--login` 时又有登录规则 |
| sshd 启动的非交互 Bash | Bash 识别远程启动时可能读 `~/.bashrc`；以 `sh` 启动、文件中的提前 return 或其它启动条件会改变结果 |

Ubuntu 的 `/etc/profile` 可以再加载系统 Bash 配置和 `/etc/profile.d/*.sh`；`profile.d` 并非 Bash 无条件扫描的内建机制。新建 `~/.bash_profile` 也可能使旧 `~/.profile` 不再被读取。应检查目标机器的实际文件和加载顺序，不要把一个文件写入成功等同所有终端都已生效。

## 环境变量与函数的区别

`pon` 在当前 Bash 执行 `export http_proxy=...` 等操作后，随后启动的程序继承这些导出变量。已经运行的终端或程序不受影响；子进程修改变量也不会反向修改父 shell。

Bash 函数本身默认不会传给新启动的 `bash script.sh`；`export -f` 是另一种机制，且不能据此假定 Zsh、Fish 或 `/bin/sh` 认识函数。需要代理的脚本通常只需继承导出的变量，不必获得 `pon` 定义。

```bash
# 在已经加载 pon 的交互 Bash 中启动任务
(pon && bash run_download.sh)
```

括号创建子 shell，代理变量只在该子 shell 及其后代中改变，任务结束后父 shell 保持原值。需要脚本自主加载环境时，应显式加载管理员提供的可信、轻量脚本，而不是无条件 source 整份交互 `~/.bashrc`。

## 管理员维护原则

`/etc/bash.bashrc` 通常可由普通用户读取；需要修改时才使用管理员权限。修改前备份文件，保留精确路径、时间和恢复方式；同一天的备份也不要互相覆盖。

```bash
less /etc/bash.bashrc
bash -n /etc/bash.bashrc
```

`bash -n` 只检查语法，不执行配置，不验证路径、命令、权限或行为。全局启动配置应安静且轻量，不默认激活个人 Conda、不改变别人私有路径、不存放凭据。交互提示只应在交互场景输出，环境设置与输出逻辑应分开。

修改后保留原管理员会话，新开 SSH 和 VS Code 终端分别检查函数类型、shell 标志和程序路径；再按实际使用方式验证非交互命令、SFTP/scp/rsync，避免启动输出污染协议。不要为测试加载真实代理而把其值写入公开日志。

## 常见排查

- **函数不存在**：检查部署是否存在、当前 shell 与加载方式；不是一概归因于没有“继承”。
- **登录慢或有额外输出**：检查启动文件中的联网、耗时操作和调试打印。
- **交互终端可用，脚本不可用**：检查脚本的解释器、是否需要函数，或只需父 shell 预先导出变量。
- **VS Code 环境不同**：记录 Remote SSH 日志与实际 shell，不盲目把所有配置 source 到所有启动路径。

依据：[GNU Bash 启动文件](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html)、[Bash 手册中的环境与函数](https://www.gnu.org/s/bash/manual/bash.html)、[Ubuntu 22.04 Bash 手册](https://manpages.ubuntu.com/manpages/jammy/man1/bash.1.html)。
