# AI 使用

服务器上可以安装 Codex、Claude Code 等 AI CLI 工具。此类工具通常依赖 Node.js、网络代理和个人 API token。使用时应注意环境隔离、网络配置和密钥安全。

## 基本依赖

多数 AI CLI 工具需要 Node.js 运行时。推荐使用 `nvm` 安装 Node.js，而不是修改系统 Node.js。

推荐安装到用户自己的目录中：

```text
~/.nvm
```

不要使用 `sudo npm install -g` 安装个人 CLI 工具，避免污染系统环境。

## 安装 nvm

`nvm` 是按用户安装的 Node.js 版本管理器，适合服务器上的普通用户自己管理 Node.js 和 npm。不要用 `sudo` 安装 `nvm`，也不要把个人 Node.js 装进系统目录。

官方文档：

```text
https://github.com/nvm-sh/nvm
```

以下命令按 nvm 官方 README 当前安装脚本版本 `v0.40.5` 整理。后续如果版本更新，可以到官方仓库确认最新版本号。

如果访问 GitHub 需要代理，先开启代理：

```bash
pon
```

使用 `curl` 安装：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
```

如果没有 `curl`，也可以使用 `wget`：

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
```

安装脚本通常会把 nvm 下载到：

```text
~/.nvm
```

并尝试把下面类似内容写入 `~/.bashrc`、`~/.bash_profile`、`~/.zshrc` 或 `~/.profile`：

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

服务器默认使用 Bash，因此安装完成后优先重新加载 `~/.bashrc`：

```bash
source ~/.bashrc
```

如果仍然找不到 `nvm`，可以手动加载：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

检查 nvm 是否可用：

```bash
command -v nvm
nvm --version
```

安装当前 LTS 版本 Node.js：

```bash
nvm install --lts
nvm use --lts
node --version
npm --version
```

把 LTS 版本设为新 shell 默认使用的 Node.js：

```bash
nvm alias default 'lts/*'
```

之后新开终端，检查：

```bash
node --version
npm --version
```

如果只在当前 shell 中临时使用，不需要写入全局配置；如果希望每次登录都可用，确认 `~/.bashrc` 中已经有加载 `nvm.sh` 的内容。

安装完成并确认不再需要代理后，可以关闭代理：

```bash
poff
```

## 安装 AI CLI 工具

不同工具的安装命令不同，应以对应官方文档为准。一般形式类似：

```bash
npm install -g <package_name>
```

如果网络不通：

```bash
pon
npm install -g <package_name>
poff
```

代理说明参见 [代理网络连接.md](代理网络连接.md)。

## Token 和密钥管理

AI CLI 通常需要登录或配置 API token。注意：

- 不要把 token 写入共享目录。
- 不要把 token 写进脚本并提交到仓库。
- 不要把 token 贴到聊天记录或公共文档。
- 如果怀疑泄露，立即撤销并重新生成。

建议使用工具自带的登录流程，或把环境变量写入个人 shell 配置：

```bash
export EXAMPLE_API_KEY="只放在个人配置中"
```

如果写入 `~/.bashrc`，确认文件权限不对其他人开放：

```bash
chmod 600 ~/.bashrc
```

## 使用代理

AI 工具经常需要访问外部服务。使用前可以开启代理：

```bash
pon
```

检查：

```bash
env | grep -i proxy
curl ipinfo.io
```

使用完成后关闭：

```bash
poff
```

## 长时间任务

AI agent 可能运行较久，建议放在 `tmux` 中。tmux 的完整用法见 [终端](终端.md)。

```bash
tmux new -s ai
```

断开后恢复：

```bash
tmux attach -t ai
```

终端使用建议参见 [终端.md](终端.md)。

## 使用规范

- 不要让 AI agent 修改不了解的共享配置。
- 执行大范围代码修改前先确认当前目录。
- 不要把服务器密钥、BMC 密码、数据访问凭据交给 AI。
- 对生成的脚本和命令先阅读再执行。
- 涉及删除、覆盖、递归权限修改时要特别谨慎。

## 常见问题

### npm 安装失败

先检查 Node.js 和 npm：

```bash
node --version
npm --version
```

再检查网络：

```bash
curl ipinfo.io
```

必要时开启代理：

```bash
pon
```

### 命令安装后找不到

检查 npm 全局路径：

```bash
npm config get prefix
npm root -g
echo "$PATH"
```

如果使用 nvm，确认当前 shell 已加载 nvm：

```bash
command -v nvm
which node
which npm
```

### AI 工具无法访问网络

检查代理变量是否存在：

```bash
env | grep -i proxy
```

如果 `curl` 能访问但 AI 工具不行，说明该工具可能需要单独配置代理，需查看对应官方文档。

如果 `pon` 或 `poff` 命令不存在，先阅读 [代理网络连接](代理网络连接.md) 的排查部分；如果问题仍然存在，再阅读 [Bash 继承](bash继承.md)。
