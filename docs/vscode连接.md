# VS Code 连接服务器

可以直接使用终端 SSH 连接服务器，也可以使用 VS Code Remote SSH 连接服务器。VS Code 的优点是跨平台体验一致，便于编辑文件、运行终端、管理项目；缺点是会在服务器上启动 VS Code Server，有一定内存和磁盘开销。

> **【说明】适用对象与完成标准**
>
> - **适用对象**：希望使用图形编辑器进行远程开发的普通用户。
> - **前置条件**：普通终端已经能执行 `ssh <server_alias>` 成功登录。
> - **所需权限**：服务器普通用户权限，不需要 `sudo`。
> - **完成标准**：VS Code 左下角显示远程主机，远程终端中的 `hostname` 和 `whoami` 输出正确，并能打开自己的项目目录。

## 前提条件

本机需要安装：

- Visual Studio Code。
- Remote - SSH 扩展。
- 可用的 SSH 密钥。
- 已配置好的 `~/.ssh/config`。

SSH 配置方法参见 [SSH 登录](ssh登录.md)。

## 配置 SSH Host

本机 `~/.ssh/config` 中应有类似配置：

```sshconfig
Host <server_alias>
    HostName <server_address>
    User <username>
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

配置完成后，先在普通终端测试：

```bash
ssh <server_alias>
```

终端能登录后，再用 VS Code 连接。这样更容易区分是 SSH 配置问题还是 VS Code 问题。

## 使用 VS Code Remote SSH

1. 打开 VS Code。
2. 安装 `Remote - SSH` 扩展。
3. 打开命令面板。
4. 选择 `Remote-SSH: Connect to Host...`。
5. 选择 `~/.ssh/config` 中配置的服务器别名。
6. 连接成功后，在远程窗口中打开服务器目录。

打开 VS Code 远程终端并执行：

```bash
hostname
whoami
pwd
```

确认输出与普通 SSH 登录一致，再开始编辑文件。

常用工作目录一般是：

```text
/home/<username>
/data/<username>
/data/project_name
```

## 服务器端开销

VS Code 会在远程服务器上安装并运行 VS Code Server，通常位于：

```text
~/.vscode-server
```

如果 VS Code 连接异常、版本冲突或占用空间过多，可以查看：

```bash
du -sh ~/.vscode-server
ps aux | grep vscode
```

清理前关闭相关远程窗口，并优先使用命令面板中的 `Remote-SSH: Kill VS Code Server on Host...` 来排查；它会中断该账户正在使用的 VS Code Server，须先保存工作。目录布局随版本变化，不要依据过期路径批量删除。

## 推荐设置

为了减少无关文件的监视和搜索开销，可以在项目中配置 `.vscode/settings.json`：

```json
{
  "files.watcherExclude": {
    "**/.git/objects/**": true,
    "**/node_modules/**": true,
    "**/.conda/**": true,
    "**/data/**": true
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/.conda": true,
    "**/data": true
  }
}
```

实际排除路径应按项目结构调整。`files.watcherExclude` 控制文件监视，`search.exclude` 控制搜索范围；二者不负责上传、下载或完整控制语言服务器索引。Remote SSH 直接编辑远端文件，也不会自动同步整个本地项目。

## 常见问题

### 终端能 SSH，但 VS Code 连不上

先查看 VS Code 的 Remote SSH 日志。常见原因包括：

- 本机 VS Code 扩展版本问题。
- 服务器端 `~/.vscode-server` 损坏。
- 服务器磁盘空间不足。
- 登录脚本额外输出或启动了其它 shell，影响启动协议。
- 服务器无法取得对应版本的 VS Code Server，或 SSH TCP 转发被策略禁止；先查日志，管理员再决定是否调整。

检查磁盘：

```bash
df -h
du -sh ~/.vscode-server
```

### 连接后卡顿

可能是打开了太大的目录。建议只打开项目目录，不要直接打开 `/data` 根目录或 `/`。

### 远程终端环境不对

VS Code 远程终端默认使用服务器上的 shell。服务器主要按 Bash 配置，特殊函数如 `pon`、`poff` 依赖 Bash 全局配置，参见 [Bash 继承](bash继承.md)。

## 使用建议

- 写代码和改脚本适合用 VS Code。
- 长时间任务仍建议放在 `tmux` 中运行。
- 不要在 VS Code 中打开超大数据目录。
- 如果只是执行几个命令，普通终端 SSH 更轻量。

依据：[VS Code Remote SSH 官方说明](https://code.visualstudio.com/docs/remote/ssh)。实际目录、版本与网络条件应按目标机器确认。
