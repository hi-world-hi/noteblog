---
title: 验证chronyc ntp服务正常
abbrlink: 44a51621
date: 2026-02-02 18:19:10
tags:
---
在 Ubuntu 22.04 上，通过 chrony（服务名为 chrony.service）向其他主机提供 NTP 服务，验证其是否正常工作可以分为 服务端检查 和 远程客户端验证 两个阶段。

以下是具体步骤：
一、 服务端配置确认（前提条件）

首先确保你的 /etc/chrony/chrony.conf 文件中包含了允许其他主机访问的指令。

```
# 编辑配置文件
sudo nano /etc/chrony/chrony.conf

# 确保有类似以下配置（例如允许 192.168.1.0 网段）
allow 192.168.1.0/24

# 如果想允许所有地址（不安全，仅建议在受控内网使用）
# allow 0.0.0.0/0

# 修改后重启服务
sudo systemctl restart chrony
```

二、 服务端自我验证（本机检查）
1. 检查服务状态

确认 chrony 服务正在运行：
```
systemctl status chrony
```
2. 检查端口监听

NTP 服务使用 UDP 123 端口。检查该端口是否已打开并处于监听状态：
```
sudo ss -unlp | grep 123
```
* 正常输出： 应显示 0.0.0.0:123 或 *:123，表示服务正在监听外部请求。
3. 检查自身同步状态

如果服务端自己都没同步好时间，它可能不会向客户端提供可靠的时间服务（或 Stratum 层级过高）：
```
chronyc tracking
```
* 关注点： Leap status 应为 Normal，且 Stratum 通常应在 2-4 之间。
4. 查看已连接的客户端

这个命令可以实时看到有哪些 IP 正在请求本服务器的时间：
```
sudo chronyc clients
```
三、 远程客户端验证（最有效的方法）

在另一台主机（客户端）上运行以下测试：
方法 A：使用 ntpdate (简单直观)

如果客户端安装了 ntpdate 工具，可以使用 -q (query) 模式测试，这不会修改本地时间：
```
# 替换 <SERVER_IP> 为你的 Ubuntu 主机 IP
ntpdate -q <SERVER_IP>
```
* 成功： 输出 server <IP>, stratum 3, offset ..., delay ...。

    失败： 输出 no server suitable for synchronization found。

方法 B：使用 sntp (现代系统常用)

很多现代 Linux 系统自带 sntp：
```
sntp <SERVER_IP>
```
* 成功： 会返回服务器的时间戳和误差。
方法 C：使用 chronyc (如果客户端也是 chrony)

在客户端的 /etc/chrony/chrony.conf 中添加服务端 IP，然后重启服务。执行：
```
chronyc sources -v
```
* 查看状态： 观察对应 IP 前面的符号。
* ^*：表示同步成功且是当前选定的主来源。
* ^?：表示无法连接或同步中。
四、 常见问题排查

如果远程验证失败，通常是以下两个原因：

    防火墙拦截：
    Ubuntu 默认可能开启了 ufw。需要允许 UDP 123 端口：
```
    sudo ufw allow 123/udp
```
    配置未生效：
    确保 allow 指令中的子网范围涵盖了客户端的 IP。如果是云服务器，还需检查云平台的“安全组/网络 ACL”是否放行了 UDP 123 端口。