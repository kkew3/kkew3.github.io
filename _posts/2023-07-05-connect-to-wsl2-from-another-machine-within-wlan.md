---
layout: post
title:  "从另一台计算机 SSH 连接到 WSL2 服务器"
date:   2023-07-05 12:59:03 +0800
categories: network
---

# 原理

1. 在 Windows 上设置防火墙允许接入端口 (本文暂定为 4000)
2. 在 Windows 上设置端口转发从 4000 至 WSL2 的 IP 地址的端口 22 (即 SSH 的默认端口)
3. 在 WSL2 中设置 SSH 服务器, 监听其端口 22
4. 从另一台计算机 SSH 到 Windows 的 IP 地址的端口 4000

# 具体流程

## 设置 WSL2 中的 SSH 服务器

*以下命令在 WSL2 的终端中执行*

安装 `openssh-server`:

```
sudo apt update
sudo apt install openssh-server
```

设置开机启动 `systemd`.
方法是在 `/etc/wsl.conf` 中写入:

```
[boot]
systemd=true
```

可以直接用 `vim`, `nano` 等编辑, 也可

```bash
{ echo '[boot]'; echo 'systemd=true'; } | sudo tee /etc/wsl.conf
```

但注意不要覆盖已存在的 `/etc/wsl.conf` 文件.

*以下命令在 Windows 终端中执行 (可能需要管理员权限)*

关闭 WSL2:

```
wsl --shutdown
```

*以下命令在 WSL2 的终端中执行*

然后打开 WSL2, 执行

```bash
sudo service ssh status
```

如果输出中包含 "Active: active (running)", 说明 SSH 服务器安装成功.

## 设置 Windows 防火墙以允许从其它计算机接入端口 (例如 4000)

*以下命令在 Windows 终端中执行 (需要管理员权限)*

```
netsh advfirewall firewall add rule name="WSL SSH" dir=in action=allow protocol=TCP localport=4000
```

其中 `name="WSL SSH"` 部分的名字可任选.
如果输出为 "确定" (或其它 locale 下的同等含义的输出), 说明设置成功.
日后若想删除可以去控制面板的 "高级安全 Windows Defender 防火墙" 的 "入站规则" 中查看/编辑/删除.

## 设置 Windows 的端口转发

*以下命令在 Windows 终端中执行 (需要管理员权限)*

查看 WSL2 的 IP 地址:

```
wsl hostname -I
```

本文假设该 IP 地址为 `172.21.199.198`.
旧版本的 `wsl` 可能会返回两个 IP 地址, 此时选择第一个.

设置从 4000 (见上文) 到 22 的端口转发:

```
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=4000 connectaddress=172.21.199.198 connectport=22
```

确定设置成功:

```
netsh interface portproxy show v4tov4
```

## 查看 Windows 的 IP 地址和 WSL 的用户名

*以下命令在 Windows 终端中执行*

```
ipconfig
```

在 "无线局域网适配器 WLAN:" 一节中可见 "IPv4 地址", 本文为 `192.168.0.105`.

*以下命令在 WSL2 的终端中执行*

```
echo "$USER"
```

可得 WSL 用户名, 本文为 `ubuntu`.

## 从另一台计算机 SSH 接入

```bash
ssh -p 4000 ubuntu@192.168.0.105
```

会提示输入密码, 此时输入 WSL2 的密码即可.

## 免密码登录 (适用于 macOS)

*以下命令在 macOS 终端中执行*

```bash
ssh-copy-id -p 4000 ubuntu@192.168.0.105
```

按提示确认并输入密码.
然后打开 `~/.ssh/config`, 并输入以下内容

```
Host my-wsl
  User ubuntu
  Port 4000
  HostName 192.168.0.105
  IdentityFile ~/.ssh/id_rsa
  UseKeychain yes
```

其中 `Host my-wsl` 处的名字随意.
`UseKeychain yes` 是免密码的关键所在.
`ubuntu`, `4000`, `192.168.0.105` 三个值的选用见上文.

然后就可以

```bash
ssh my-wsl
```

登录 Windows 的 WSL2 了.

# 脚本

## 自动更新 Windows 的端口转发

WSL 的 IP 地址可能会变化, 因此每次重启 Windows 后可能需要更新端口转发规则.
Powershell 脚本:

```powershell
$wsl_ip = wsl hostname -I
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=4000 connectaddress=$wsl_ip connectport=22
```

运行时需要管理员权限.

# 参考

- [这篇博客](https://medium.com/geekculture/enable-ssh-access-into-wsl-from-a-remote-computer-f2e4a962430)
- [微软 WSL 文档](https://learn.microsoft.com/en-us/windows/wsl/networking#accessing-a-wsl-2-distribution-from-your-local-area-network-lan)
- [微软 netsh 文档](https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-interface-portproxy)
