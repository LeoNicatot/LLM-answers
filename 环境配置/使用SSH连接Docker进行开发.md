# SSH连接Docker开发

## 如果我希望用docker搭一个实验环境，然后在宿主机的vscode上通过ssh连接到docker容器进行开发，我应该怎么操作？

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

## 我如何在MacOS上查看我的ssh公钥？

在macOS上查看您的SSH公钥，您可以遵循以下步骤：

1. **打开终端**： 打开Mac上的终端应用。您可以在Launchpad中找到它，或者通过Spotlight搜索（按`Command + Space`，然后键入“终端”）。

2. **导航到SSH目录**： 在终端中输入以下命令来切换到包含SSH密钥的目录：

   ```
   cd ~/.ssh
   ```

3. **查看公钥**： 使用`cat`命令来显示您的公钥内容。通常公钥文件名为`id_rsa.pub`。如果您的密钥文件名不同，使用实际的文件名替换`id_rsa.pub`：

   ```
   cat id_rsa.pub
   ```

这将输出您的SSH公钥到终端窗口中。确保这个公钥与您要使用的远程服务（如GitHub、GitLab或其他SSH服务器）相匹配或已经上传到相应的服务中。

如果您之前没有生成过SSH密钥对，可以使用以下命令生成一个新的RSA密钥对：

```
ssh-keygen -t rsa -C "your_email@example.com"
```

这将创建一个RSA类型的密钥对，并将您的电子邮件地址作为注释添加到公钥中。在生成过程中，系统会提示您输入一个安全的密码短语，以保护您的私钥。请记住这个密码短语，因为如果没有它，您将无法使用私钥进行身份验证。

## 如何在ssh服务器上添加公钥？

在SSH服务器上添加公钥以便进行无密码登录，您需要完成以下几个步骤：

1. **生成SSH密钥对**： 确保您已经在本地机器上生成了SSH密钥对。如果您还没有生成，可以在本地计算机上使用以下命令来生成一对新的SSH密钥（以RSA为例）：

   ```
   ssh-keygen -t rsa -b 4096 -C "youremail@example.com"
   ```

   上面的命令将生成一个4096位的RSA密钥对，其中`youremail@example.com`会被用作识别标签。这一步会在本地机器的`~/.ssh/`目录下生成两个文件：`id_rsa`（私钥）和`id_rsa.pub`（公钥）。

2. **获取公钥**： 使用以下命令来查看您的公钥内容：

   ```
   cat ~/.ssh/id_rsa.pub
   ```

   这会输出您的公钥到终端。

3. **将公钥传输到远程服务器**： 您可以通过多种方式将公钥发送到远程服务器，但最常见的是直接使用`ssh-copy-id`命令，或者手动复制粘贴。下面是两种常见的方法：

   - **使用`ssh-copy-id`命令**（如果可用）：

     ```
     ssh-copy-id -i ~/.ssh/id_rsa.pub [username@]remote_host
     ```

     这个命令会自动将公钥添加到远程服务器的`~/.ssh/authorized_keys`文件中。

   - **手动复制粘贴公钥**：

     1. 复制公钥的内容。

     2. 登录到远程服务器：

        ```
        ssh [username@]remote_host
        ```

     3. 创建

        ```
        .ssh
        ```

        目录（如果不存在）：

        ```
        mkdir -p ~/.ssh
        ```

     4. 更改权限：

        ```
        chmod 700 ~/.ssh
        ```

     5. 创建或编辑

        ```
        authorized_keys
        ```

        文件：

        ```
        touch ~/.ssh/authorized_keys
        chmod 600 ~/.ssh/authorized_keys
        ```

     6. 手动粘贴公钥到`authorized_keys`文件中。

4. **测试SSH连接**： 为了确保一切正常，从本地机器尝试无密码SSH登录远程服务器：

   ```
   ssh [username@]remote_host
   ```

   如果一切设置正确，您应该能够成功登录而无需输入密码。

请注意，上面的命令中的`[username@]`和`remote_host`需要替换为您实际的用户名和远程服务器的主机名或IP地址。如果使用的是非标准端口，您还需要在`ssh`命令中添加`-p`参数指定端口号。例如：

```
ssh -p port_number [username@]remote_host
```

## 如何用ssh在输入密码的情况下连接服务器？

使用SSH（Secure Shell）在输入密码的情况下连接服务器，你可以遵循以下步骤：

1. **打开终端**： 首先，你需要在你的本地计算机上打开一个终端或命令行界面。

2. **使用SSH命令**： 接下来，使用SSH命令格式来连接到远程服务器。命令的基本语法是：

   ```
   ssh [options] [user@]hostname
   ```

   在这个命令中，`[user]`是你在远程服务器上的用户名，`hostname`可以是服务器的IP地址或其DNS名称。如果你的服务器使用非默认的SSH端口，你还需要包含`-p`选项来指定端口号。

   例如，如果你的用户名是`john`，服务器的IP地址是`192.168.1.100`，并且使用的是默认端口22，那么命令将是：
   
   ```
   ssh john@192.168.1.100
   ```
   如果服务器使用的是非标准端口，比如`2222`，则命令应该是：

   ```
   ssh -p 2222 john@192.168.1.100
   ```

3. **输入密码**： 当你执行SSH命令后，系统会提示你输入密码。输入密码时，屏幕上不会显示任何字符，这是因为这样可以防止密码被旁观者看到。输入正确的密码后，按回车键。

4. **确认服务器的指纹**： 如果你是第一次连接到该服务器，SSH客户端会警告你关于服务器的指纹，并询问你是否继续连接。你应该检查指纹与服务器管理员提供的指纹是否匹配，以确保你正在连接到正确的服务器。如果一切正确，输入`yes`并按回车键继续。

5. **登录成功**： 如果一切顺利，你会看到远程服务器的命令行提示符，表示你已经成功登录。

记住，为了安全起见，不要在不信任的网络环境下使用密码认证，而是考虑使用更安全的公钥认证或其他双因素认证方法。此外，频繁使用的密码应该定期更改，以增强安全性。