# SSH 登录

服务器全面禁止密码登录，统一使用 SSH 公钥登录。这样既方便，也能减少密码泄露、弱口令和暴力破解风险。用户登录服务器前，需要在本机生成密钥，并把公钥加入服务器账户的 `authorized_keys`。

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

一路回车即可使用默认路径：

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

建议给私钥设置 passphrase。这样即使私钥文件泄露，也多一层保护。

## 查看公钥

把 `.pub` 文件内容发给管理员：

```bash
cat ~/.ssh/id_ed25519.pub
```

只发送公钥，不要发送没有 `.pub` 后缀的私钥。

## 服务器端配置

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

`userkit` 会自动设置 `.ssh` 和 `authorized_keys` 权限，并在替换已有 key 前做备份。详细说明参见 [服务器管理.md](服务器管理.md)。

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
- `IdentitiesOnly yes` 表示只使用指定私钥，避免本机尝试太多其它密钥。

## 第一次登录

第一次登录会看到类似提示：

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

确认服务器地址无误后输入：

```text
yes
```

SSH 会把服务器指纹记录到本机 `~/.ssh/known_hosts`。

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

确认安全后，可以删除旧记录：

```bash
ssh-keygen -R <server_address>
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
- 离组或不再使用服务器时，应删除对应公钥。
- 如果怀疑私钥泄露，立即生成新密钥并通知管理员移除旧公钥。
