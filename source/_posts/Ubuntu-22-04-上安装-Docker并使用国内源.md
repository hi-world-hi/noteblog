---
title: Ubuntu 22.04 上安装 Docker并使用国内源
abbrlink: 9d464c1
date: 2026-01-23 19:54:08
tags:
---
Ubuntu 22.04 上安装 Docker 需要几个步骤。以下是一个详细的指南，包括所有必要的命令和解释：

#### 1. 更新软件包索引:

这是安装任何新软件包前的最佳实践，确保你拥有最新的软件包信息。

```bash
sudo apt update
```

Bash

#### 2. 安装必要的软件包以允许 apt 通过 HTTPS 使用仓库:

这些软件包允许你的系统安全地连接到 Docker 的仓库并下载软件包。

```bash
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
```

Bash

```
apt-transport-https: 允许 apt 使用 HTTPS 协议。
ca-certificates: 包含信任的证书颁发机构 (CA) 的证书。
curl: 一个用于传输数据的命令行工具。
gnupg: GNU Privacy Guard，用于安全地导入 Docker 的 GPG 密钥。
lsb-release: 提供关于 Linux 发行版的信息。
```

#### 3. 添加 Docker 的官方 GPG 密钥:

GPG 密钥用于验证下载的 Docker 软件包的完整性和真实性。

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Bash

```
sudo mkdir -p /etc/apt/keyrings: 如果 /etc/apt/keyrings 目录不存在，则创建它。

curl -fsSL https://download.docker.com/linux/ubuntu/gpg: 从 Docker 官方网站下载 GPG 密钥。
    -f: 如果服务器报告错误则退出。
    -s: 静默模式 (不显示进度条和错误消息)。
    -S: 显示错误消息。
    -L: 如果服务器返回重定向，则跟随重定向。

sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg: 将下载的 ASCII 格式的 GPG 密钥转换为二进制格式并保存到 /etc/apt/keyrings/docker.gpg。
```

#### 4. 设置 Docker 仓库:

你需要将 Docker 的仓库添加到你的 apt 源列表中，这样 apt 才能找到 Docker 软件包。

```bash
echo \
+  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
+  $(lsb_release -cs) stable"
```
错误：
```bash
echo \
  "deb [arch=(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


```
ARCH=(dpkg --print-architecture) 和 RELEASE=(lsb_release -cs): 首先，将 dpkg --print-architecture 和 lsb_release -cs 命令的输出分别存储在 ARCH 和 RELEASE 变量中。
"deb [arch=$ARCH signed-by=/etc/apt/keyrings/docker.gpg] ...": 然后在字符串中使用变量替换。 在双引号字符串中，变量会被 shell 展开。
为什么有效: 这种方法将子shell的执行与字符串的生成分离开，从而避免了在管道或 tee 命令中进行复杂的 shell 扩展问题。 这是最安全和最可读的方法。
```

#### 5. 再次更新软件包索引:

现在你的 apt 源列表中包含了 Docker 的仓库，需要更新软件包索引来获取新的软件包信息。

```bash
sudo apt update
```

Bash

#### 6. 安装 Docker Engine:

现在你可以安装 Docker Engine、Containerd 和 Docker Compose 插件。

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Bash

```
docker-ce: Docker Community Edition，即 Docker Engine。
docker-ce-cli: 用于与 Docker Engine 交互的命令行工具。
containerd.io: 一个行业标准的容器运行时。
docker-compose-plugin: 用于定义和运行多容器 Docker 应用程序的工具 (作为 Docker CLI 的插件存在)。
```

#### 6.5 修改为国内可用源(20250107测试可用)

如需代理加上 proxies

```bash
# cat /etc/docker/daemon.json 
{
   "registry-mirrors": [
  "https://docker.m.daocloud.io",
  "http://mirrors.ustc.edu.cn",
  "http://mirror.azure.cn"
   ],
   "proxies": {
           "http-proxy": "http://127.0.0.1:8080",
           "https-proxy": "http://127.0.0.1:8080"
      }

 }

sudo systemctl daemon-reload
sudo systemctl restart docker

```

Bash

#### 7. 验证 Docker 是否安装成功:

安装完成后，可以运行以下命令来检查 Docker 是否已正确安装并正在运行。

```bash
sudo systemctl status docker
```

Bash

你应该看到类似以下的输出，表明 Docker 服务正在运行 (active (running)):

```bash
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-10-24 10:00:00 UTC; 1min 30s ago
       Docs: https://docs.docker.com
   Main PID: 1234 (dockerd)
      Tasks: 50
     Memory: 100.0M
        CPU: 1.500s
     CGroup: /system.slice/docker.service
             └─1234 /usr/bin/dockerd -H fd://
```

Bash

你也可以运行一个简单的 Docker 镜像来验证安装：

```bash
sudo docker run hello-world
```

Bash

如果一切正常，你应该看到一条欢迎消息，表明 Docker 已经正确安装并正在运行。

#### 8. （可选）将你的用户添加到 docker 组，以便无需 sudo 运行 Docker 命令:

默认情况下，你需要使用 sudo 来运行 Docker 命令。为了避免这种情况，你可以将你的用户添加到 docker 组。

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Bash

```
sudo usermod -aG docker USER: 将当前用户 (USER) 添加到 docker 组。
newgrp docker: 使组更改立即生效，无需注销并重新登录。
```

重要提示: 将用户添加到 docker 组会赋予该用户 root 权限，因为他们可以控制 Docker 守护进程。请谨慎操作。

#### 9. Docker Compose (如果需要):

你已经安装了 Docker Compose 插件，它可以通过 docker compose 命令来使用。 你可以创建一个 docker-compose.yml 文件来定义你的多容器应用程序。

例如，创建一个简单的 docker-compose.yml 文件：

```bash
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
```

Bash

然后，在该文件所在的目录中运行：

```bash
docker compose up -d
```

Bash

这将启动一个运行 Nginx 的容器。

常用 Docker 命令:

```bash
docker ps: 列出正在运行的容器。
docker images: 列出本地可用的镜像。
docker pull <image_name>: 从 Docker Hub 下载镜像。
docker stop <container_id>: 停止一个正在运行的容器。
docker rm <container_id>: 删除一个已停止的容器。
```

Bash

### 总结:

以上就是在 Ubuntu 22.04 上安装 Docker 的详细步骤。按照这些步骤操作，你应该能够成功安装并运行 Docker。记得阅读 Docker 的官方文档以了解更多关于 Docker 的功能和用法。