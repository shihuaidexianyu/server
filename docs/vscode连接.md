# VS Code 连接服务器

可以直接使用终端 SSH 连接服务器，也可以使用 VS Code Remote SSH 连接服务器。VS Code 的优点是跨平台体验一致，便于编辑文件、运行终端、管理项目；缺点是会在服务器上启动 VS Code Server，有一定内存和磁盘开销。

## 前提条件

本机需要安装：

- Visual Studio Code。
- Remote - SSH 扩展。
- 可用的 SSH 密钥。
- 已配置好的 `~/.ssh/config`。

SSH 配置方法参见 [ssh登录.md](ssh登录.md)。

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

清理前先关闭 VS Code 远程窗口，再按需删除旧版本目录。不要删除不确定的项目文件。

## 推荐设置

为了避免误把大文件、数据集、缓存同步到编辑器索引，可以在项目中配置 `.vscode/settings.json`：

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

实际排除路径应按项目结构调整。

## 常见问题

### 终端能 SSH，但 VS Code 连不上

先查看 VS Code 的 Remote SSH 日志。常见原因包括：

- 本机 VS Code 扩展版本问题。
- 服务器端 `~/.vscode-server` 损坏。
- 服务器磁盘空间不足。
- 登录 shell 输出了额外内容，影响 VS Code Server 启动。

检查磁盘：

```bash
df -h
du -sh ~/.vscode-server
```

### 连接后卡顿

可能是打开了太大的目录。建议只打开项目目录，不要直接打开 `/data` 根目录或 `/`。

### 远程终端环境不对

VS Code 远程终端默认使用服务器上的 shell。服务器主要按 Bash 配置，特殊函数如 `pon`、`poff` 依赖 Bash 全局配置，参见 [bash继承.md](bash继承.md)。

## 使用建议

- 写代码和改脚本适合用 VS Code。
- 长时间任务仍建议放在 `tmux` 中运行。
- 不要在 VS Code 中打开超大数据目录。
- 如果只是执行几个命令，普通终端 SSH 更轻量。
