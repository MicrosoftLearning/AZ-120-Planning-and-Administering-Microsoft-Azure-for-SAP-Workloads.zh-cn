# 演示：连接到 Linux 虚拟机

在本演示中，我们将创建一台 Linux 机器并使用 SSL 访问该机器。

>**备注：** 确保端口 22 打开，以便连接正常工作。 

## 创建 SSH 密钥

1. 下载 PuTTY 工具。这将包括 [PuTTYgen](https://putty.org/)。 
2. 安装完成后，找到并打开 **PuTTYgen** 程序。
3. 在“**参数**”选项组中选择“**RSA**”。
4. 单击“**生成**”按钮。
5. 将鼠标移动到窗口中的空白区域以生成一些随机值。
6. 复制“**供粘贴到授权密钥的公钥**”文件中的文本。
7. 可以选择指定**密钥密码**，随后**确认密码**。使用私有 SSH 密钥对 VM 进行身份验证时，系统将提示你输入密码。如果没有设置密码，一旦有人获得了你的私钥，他们就可以登录使用该密钥的任何 VM 或服务。建议你创建密码。但是，如果忘记了密码，是无法恢复该密码的。
8. 单击“**保存私钥**”。
9. 选择位置和文件名，然后单击“**保存**”。需要此文件来访问 VM。 

## 创建 Linux 虚拟机并分配公共 SSH 密钥

1. 在门户中创建所选的 Linux 虚拟机。
2. 对于“**身份验证类型**”，请选择“**SSH 公钥**”（而不是“**密码**”）。
3. 提供**用户名**。
4. 将 PuTTY 中的公共 SSH 密钥粘贴到“**SSH 公钥**”文本区域中。确保密钥验证带有复选标记。 
5. 创建 VM。等待其进行部署。
6. 访问正在运行的 VM。 
7. 在“**概述**”边栏选项卡中，单击“**连接**”。
8. 记下你的登录信息，包括用户和公共 IP 地址。

## 使用 SSH 访问服务器

1. 打开 **PuTTY** 工具。
2. 输入“**username@publicIpAddress**”，其中 username 是你在创建 VM 时分配的值，而 publicIpAddress 是你从 Azure 门户获得的值。
3. 为“**端口**”指定“**22**”。
4. 在“**连接类型**”选项组中选择“**SSH**”。
5. 在“类别”面板中导航到“**SSH**”，然后单击“**身份验证**”。
6. 单击“**用于身份验证的私钥文件**”旁的“**浏览**”按钮。
7. 导航到生成 SSH 密钥时保存的私钥文件，然后单击“**打开**”。
8. 在主 PuTTY 屏幕上，单击“**打开**”。
9. 现在将连接到服务器命令行。 
