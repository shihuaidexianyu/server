# BMC

BMC 是服务器的板载管理控制器，可用于查看硬件状态、远程开关机、进入远程控制台等。BMC 通常独立于操作系统运行，即使 Ubuntu 无响应，也可能通过 BMC 查看机器状态或进行电源操作。

## BMC 地址信息

服务器主机名、管理 IP、BMC IP 和访问地址属于内部运维信息，不在公开文档中记录。管理员应在本地私有清单中维护这些映射。

其中“管理 IP”用于正常 SSH、服务访问和系统层面的管理；“BMC IP”用于硬件管理控制台。两者不是同一个入口，不要混用。

## BMC 能做什么

BMC 常用于：

- 查看电源状态。
- 查看温度、风扇、电压等硬件传感器。
- 远程开机、关机、重启。
- 查看硬件告警。
- 使用远程控制台排查系统无法启动的问题。
- 检查 RAID、磁盘或硬件异常。

## 登录方式

通常在浏览器中访问：

```text
https://<bmc_ip>
```

用户名和密码不应写在共享文档中，应由管理员通过安全方式保存和分发。

## 使用 ipmitool 检查

如果本机或服务器安装了 `ipmitool`，可以通过命令行查看状态。以下命令中的地址和 `<bmc_username>` 需要替换为实际值。

为避免密码进入 shell 历史，可以交互式读取密码：

```bash
read -rsp "BMC password: " IPMI_PASSWORD
export IPMI_PASSWORD
ipmitool -I lanplus -H <bmc_ip> -U <bmc_username> -E power status
unset IPMI_PASSWORD
```

查看传感器：

```bash
read -rsp "BMC password: " IPMI_PASSWORD
export IPMI_PASSWORD
ipmitool -I lanplus -H <bmc_ip> -U <bmc_username> -E sensor
unset IPMI_PASSWORD
```

不建议在 shell 历史、脚本或共享文档中留下 BMC 密码。

执行命令前，从本地私有清单获取目标 BMC 地址，并把 `<bmc_ip>` 和 `<bmc_username>` 替换为实际值。不要在共享文档或脚本中保存地址与账号的对应关系。

## 电源操作

!!! warning "强制断电风险"

    `power off` 和 `power cycle` 相当于拔掉物理电源线。非系统完全死锁且征得所有正在运行任务成员同意，严禁执行此操作。

电源操作影响所有用户，必须谨慎。执行前应确认：

- 是否有人正在运行任务。
- 是否有重要数据正在写入。
- 是否有其它恢复方式。
- 是否已经通知相关用户。

常见操作包括：

```bash
read -rsp "BMC password: " IPMI_PASSWORD
export IPMI_PASSWORD
ipmitool -I lanplus -H <bmc_ip> -U <bmc_username> -E power status
ipmitool -I lanplus -H <bmc_ip> -U <bmc_username> -E power on
ipmitool -I lanplus -H <bmc_ip> -U <bmc_username> -E power off
ipmitool -I lanplus -H <bmc_ip> -U <bmc_username> -E power cycle
unset IPMI_PASSWORD
```

其中 `power off` 和 `power cycle` 都可能导致数据丢失。只有在系统无法正常恢复时才使用。

## 推荐处理顺序

服务器异常时，建议按以下顺序处理：

1. 先尝试 SSH 登录。
2. 如果 SSH 不通，检查网络和防火墙。
3. 如果系统疑似卡死，登录 BMC 查看电源和硬件状态。
4. 如果还能进入远程控制台，优先尝试正常重启。
5. 只有在必要时才执行强制电源操作。

异常处理参见 [异常处理.md](异常处理.md)。

## 安全要求

- BMC 不应暴露到不可信网络。
- BMC 密码不能写入共享文档、脚本或聊天记录。
- 离组人员应及时移除 BMC 访问权限。
- 电源操作应记录时间、原因和执行人。
