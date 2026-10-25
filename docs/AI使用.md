# AI 使用

服务器上可以安装 Codex、Claude Code 等 AI CLI 工具。依赖和认证方式随工具、安装方式与版本变化：Codex 和 Claude Code 都提供独立/原生安装方式，不能把 Node.js、代理或 API token 当作共同必需条件。使用时应注意环境隔离、网络配置和密钥安全。

> **【说明】适用对象与完成标准**
>
> - **适用对象**：需要在服务器上使用个人 AI CLI 工具的普通用户。
> - **前置条件**：基础网络和代理配置正常，并拥有个人合法账号或 token。
> - **所需权限**：只安装到个人目录，不需要也不应使用 `sudo npm install -g` 修改系统环境。
> - **完成标准**：目标 CLI 能输出版本（选 npm 安装时再检查 Node.js/npm），登录或 token 配置仅对当前用户可见，并能在 `tmux` 中完成一次测试调用。

## 基本依赖

先读目标工具的官方安装说明。选择 npm 发行方式或项目本身需要 Node.js 时，才使用个人 `nvm` 管理 Node.js；独立安装器不需要先完成整套 nvm 流程。

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

以下命令以 nvm 安装脚本版本 `v0.40.8` 为例；其他版本的安装方式与要求应以官方仓库为准。

如果访问 GitHub 需要代理，先开启代理：

```bash
pon
```

先下载到自己的临时目录，避免覆盖已有安装脚本：

```bash
ai_download_dir=$(mktemp -d) && cd "$ai_download_dir"
curl -fLo nvm-install-v0.40.8.sh https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.8/install.sh
```

如果没有 `curl`，也可以使用 `wget`：

```bash
wget -O nvm-install-v0.40.8.sh https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.8/install.sh
```

确认下载成功、查看脚本及其将修改的文件后，再单独执行下面一步。下载失败时不要执行，也不要改用目录里的旧脚本：

```bash
bash nvm-install-v0.40.8.sh
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

[Codex CLI 官方文档](https://learn.chatgpt.com/docs/codex/cli) 提供 macOS/Linux 独立安装器及其它发行方式；[Claude Code 官方安装](https://code.claude.com/docs/en/setup) 区分原生与 npm 方式，当前 npm 包要求 Node.js 22 或更高。安装前记录选用方式和版本，不以 `nvm install --lts` 的变化结果作为实验固定版本。选择 npm 方式时，一般形式类似：

```bash
npm install -g <package_name>
```

如果网络不通：

```bash
pon
npm install -g <package_name>
poff
```

代理说明参见 [代理网络连接](代理网络连接.md)。

## Token 和密钥管理

AI CLI 通常需要登录或配置 API token。注意：

- 不要把 token 写入共享目录。
- 不要把 token 写进脚本并提交到仓库。
- 不要把 token 贴到聊天记录或公共文档。
- 如果怀疑泄露，立即撤销并重新生成。

优先按工具自带的登录流程配置认证，或从受控秘密管理工具临时注入所需环境变量；不要默认把长期 token 写入 `~/.bashrc`。文件权限限制也不能阻止该账户运行的程序读取秘密，导出变量会被子进程继承。

Codex 支持 ChatGPT 登录及 API key 等认证方式，功能和费用依认证方式与计划而定。无浏览器的服务器可按 [官方认证文档](https://learn.chatgpt.com/docs/auth) 使用 `codex login --device-auth`，前提是账户/工作区允许；不要分享一次性设备码。文件型凭据存储中的 `~/.codex/auth.json` 含敏感 token，应像密码一样保护，不读出或粘贴到日志。

## 使用代理

AI 工具经常需要访问外部服务。使用前可以开启代理：

```bash
pon
```

检查：

```bash
env | cut -d= -f1 | grep -i proxy
curl --max-time 10 https://ipinfo.io/ip
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

终端使用建议参见 [终端](终端.md)。

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
curl --max-time 10 https://ipinfo.io/ip
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
env | cut -d= -f1 | grep -i proxy
```

如果 curl 能访问而 AI 工具失败，还要比较目标主机、TLS、认证和程序自己的代理配置，不能直接确定是代理变量问题。

如果 `pon` 或 `poff` 命令不存在，先阅读 [代理网络连接](代理网络连接.md) 的排查部分；如果问题仍然存在，再阅读 [Bash 继承](bash继承.md)。

安装 nvm 前使用自己的下载目录并确认文件名不会覆盖原文件；其默认位置也可能受 `XDG_CONFIG_HOME` 影响，按安装输出核对 `NVM_DIR`。每次记录 `codex --version` 或 `claude --version`；最小调用应在临时项目中、按用户允许的账户范围进行。

参考：[Codex CLI](https://learn.chatgpt.com/docs/codex/cli)、[Codex 认证](https://learn.chatgpt.com/docs/auth)、[nvm 官方 README](https://github.com/nvm-sh/nvm) 和 [Claude Code setup](https://code.claude.com/docs/en/setup)。
