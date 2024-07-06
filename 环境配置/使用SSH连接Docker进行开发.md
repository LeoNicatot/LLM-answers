# SSH连接Docker开发

## Q：如果我希望用docker搭一个实验环境，然后在宿主机的vscode上通过ssh连接到docker容器进行开发，我应该怎么操作？

要在宿主机上的 Visual Studio Code (VSCode) 通过 SSH 连接到 Docker 容器进行开发，你需要按照以下步骤操作：

### 准备工作

1. **确保你有 Docker 和 VSCode：** 确保你的宿主机上已安装 Docker 和 Visual Studio Code。
2. **安装 VSCode 插件：** 在 VSCode 中安装必要的插件，如 Remote - SSH 和 Docker 扩展。

### 创建 Docker 镜像和容器

1. **创建 Dockerfile：** 创建一个 Dockerfile，定义你的开发环境，例如 Python、Node.js 等。

2. **构建 Docker 镜像：** 使用 `docker build` 命令构建 Dockerfile 中定义的镜像。

3. **运行 Docker 容器：** 使用 `docker run` 命令启动容器，并确保暴露 SSH 端口（通常是 22）到宿主机的某个端口，例如：

   ```
   docker run -d -p <host_port>:22 --name <container_name> <image_name>
   ```

   其中 `<host_port>` 是宿主机上的端口号，`<container_name>` 是你的容器名称，`<image_name>` 是你之前构建的 Docker 镜像名称。

### 在容器内安装 SSH 服务

1. **启动容器：** 如果你的容器未运行，使用 `docker start <container_name>` 启动它。

2. **进入容器：** 使用 `docker exec -it <container_name> /bin/bash` 命令进入容器的 shell。

3. **安装 SSH 服务：** 在容器内安装 SSH 服务，例如使用 `apt-get`（Debian/Ubuntu）或 `yum`（CentOS/RHEL）安装 `openssh-server`：

   ```
   apt-get update && apt-get install -y openssh-server
   ```

   或

   ```
   yum install -y openssh-server
   ```

4. **配置 SSH 服务：** 编辑 `/etc/ssh/sshd_config` 文件，确保以下配置项正确：

   - `PermitRootLogin yes`（允许 root 登录，但推荐创建非 root 用户）
   - `ListenAddress 0.0.0.0`（监听所有地址）

5. **启动 SSH 服务：** 使用 `service ssh start` 或 `systemctl start sshd` 启动 SSH 服务。

6. **确保防火墙允许 SSH：** 如果你的容器或宿主机有防火墙，确保它允许 SSH 流量。

### 配置 VSCode 进行 SSH 连接

1. **生成 SSH 密钥：** 在宿主机上使用 `ssh-keygen` 命令生成 SSH 密钥对，如果没有生成过的话。

2. **将公钥复制到容器：** 使用 `ssh-copy-id` 命令将宿主机的 SSH 公钥复制到容器中，或者手动添加公钥到容器用户的 `.ssh/authorized_keys` 文件。

3. **配置 VSCode 的 SSH 连接：** 使用 VSCode 的 Remote - SSH 扩展配置 SSH 连接，通过编辑 `~/.ssh/config` 文件或在 VSCode 中直接配置，如下所示：

   ```
   Host <container_name_or_alias>
       HostName <container_ip_or_name>
       User <username>
       Port <host_port>
       IdentityFile ~/.ssh/<your_private_key>
   ```

4. **测试 SSH 连接：** 在 VSCode 中使用 Remote Explorer 打开 SSH 连接，或者直接通过终端使用 `ssh` 命令测试连接。

### 开始开发

现在你已经设置好了 SSH 连接到 Docker 容器的环境，可以在 VSCode 中通过远程连接开发容器内的项目了。

请记住，将 SSH 服务暴露给外部网络可能会增加安全风险，所以在生产环境中谨慎使用。在开发环境中，确保你的网络环境安全，并定期检查和更新 SSH 配置和密钥。