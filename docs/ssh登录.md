# SSH 登录

实验室文档约定使用 SSH 公钥登录，密码登录应由管理员禁用；这一约定是否生效，需检查服务器的实际 `sshd` 配置。这样既方便，也能减少密码泄露、弱口令和暴力破解风险。用户登录服务器前，需要在本机生成密钥，并把公钥加入服务器账户的 `authorized_keys`。

> **【说明】适用对象与完成标准**
>
> - **适用对象**：需要从个人电脑登录服务器的普通用户；“服务器端配置”小节仅供管理员操作。
> - **前置条件**：已获得 `<username>`、`<server_address>`、服务器 SSH 指纹和公钥提交渠道。
> - **所需权限**：本机普通用户权限；账户拥有者一般可管理自己的 `authorized_keys`，但首次开户按实验室流程交管理员处理。
> - **完成标准**：`ssh <server_alias>` 能登录，且 `hostname`、`whoami` 与管理员提供的信息一致。

## SSH 登录的基本概念

SSH 公钥登录包含两部分：

- 私钥：保存在自己电脑上，不能发给别人。
- 公钥：可以交给管理员，放到服务器用户目录下。

登录时，本机用私钥证明身份，服务器用公钥验证身份。服务器不需要知道你的私钥。

## 生成密钥

在本机终端执行：

```bash
ssh-keygen -t ed25519 -C "your_name@lab"
```

先检查已有密钥；看到覆盖提示时不要覆盖正在使用的密钥，可以另取文件名。接受默认保存路径时，文件分别为：

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

在口令提示处建议给私钥设置 passphrase，不要把“一路回车”理解为必须留空口令。这样即使私钥文件泄露，也多一层保护。

## 查看公钥

把 `.pub` 文件内容发给管理员：

```bash
cat ~/.ssh/id_ed25519.pub
```

只发送公钥，不要发送没有 `.pub` 后缀的私钥。

## 服务器端配置

> **【警告】仅限管理员**
>
> 普通用户只需要把公钥提交给管理员，不要尝试修改其他用户的 home、`.ssh` 或 `authorized_keys`。

管理员把用户公钥加入：

```text
/home/<username>/.ssh/authorized_keys
```

推荐管理员使用 `userkit` 写入或更新公钥：

```bash
cd /home/<admin_name>/userkit
sudo ./userkit set-key <username> --public-key '<ssh_public_key>'
```

如果要保留已有 key 并追加新 key：

```bash
sudo ./userkit set-key <username> --public-key '<ssh_public_key>' --append
```

部署接口记录中，`userkit` 负责设置权限并备份被替换的 key；实际行为须对照部署版本的实现、`--help` 和备份策略。详细说明参见 [服务器管理](服务器管理.md)。

权限建议如下：

```bash
sudo chmod 700 /home/<username>/.ssh
sudo chmod 600 /home/<username>/.ssh/authorized_keys
sudo chown -R <username>:<username> /home/<username>/.ssh
```

权限不正确时，SSH 可能拒绝使用公钥。

## 本机配置 SSH 别名

为了能用 `ssh server-a` 这类短命令登录，需要在本机 `~/.ssh/config` 中配置 Host。

示例：

```sshconfig
Host <server_alias>
    HostName <server_address>
    User <username>
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

配置后即可登录：

```bash
ssh <server_alias>
```

其中：

- `Host <server_alias>` 是本机使用的别名，可以自定义，例如 `server-a`。
- `HostName` 是服务器 IP 或域名。
- `User` 是服务器上的用户名。
- `IdentityFile` 是私钥路径。
- `IdentitiesOnly yes` 限制使用配置的身份，避免 SSH agent 中额外密钥全部被尝试；多个 `IdentityFile` 仍可能一起生效，可用 `ssh -G` 检查最终配置。

## 第一次登录

第一次登录会看到类似提示：

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

通过管理员提供的可信渠道逐字核对显示的 SHA256 指纹；地址正确并不能证明服务器身份。指纹一致后输入：

```text
yes
```

SSH 会把服务器指纹记录到本机 `~/.ssh/known_hosts`。

登录后立即核对：

```bash
hostname
whoami
pwd
```

如果主机名、用户名或 home 目录与管理员提供的信息不一致，应退出并联系管理员。

## 常见排查命令

查看详细登录过程：

```bash
ssh -vvv <server_alias>
```

检查本机配置是否被正确解析：

```bash
ssh -G <server_alias>
```

检查服务器 22 端口是否可达：

```bash
nc -vz <server_address> 22
```

## 常见问题

### Permission denied publickey

常见原因：

- 公钥没有加入服务器。
- 本机使用了错误私钥。
- `~/.ssh/config` 写错了用户名或地址。
- 服务器端 `.ssh` 或 `authorized_keys` 权限不正确。

先用：

```bash
ssh -vvv <server_alias>
```

查看 SSH 实际尝试了哪个私钥。

### WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED

说明服务器指纹和本机记录不一致。可能是服务器重装，也可能是连接到了错误机器。不要直接忽略，先联系管理员确认。

经管理员确认新指纹后，按警告中给出的真实记录名删除旧记录（可能为主机名、IP，非默认端口用 `[host]:port`）：

```bash
ssh-keygen -R <known_hosts_name>
```

### 私钥权限过宽

如果看到 `UNPROTECTED PRIVATE KEY FILE`，修复本机私钥权限：

```bash
chmod 600 ~/.ssh/id_ed25519
chmod 700 ~/.ssh
```

## 安全要求

- 不要共享私钥。
- 不要把私钥上传到服务器或网盘。
- 不要把私钥写入聊天记录。
- 离组或不再使用服务器时，由管理员从服务器账户的 `authorized_keys` 撤销公钥；只删除本机 `.pub` 文件不会撤销登录权限。
- 如果怀疑私钥泄露，立即生成新密钥并通知管理员移除旧公钥。

依据：[OpenSSH ssh_config](https://man.openbsd.org/ssh_config)、[ssh-keygen](https://man.openbsd.org/ssh-keygen)。提交排障日志前需遮盖内部地址、用户名和路径。
