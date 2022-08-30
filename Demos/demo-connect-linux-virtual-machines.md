# <a name="demonstration-connect-to-linux-virtual-machines"></a>演示：连接到 Linux 虚拟机

在本演示中，我们将创建 Linux 虚拟机并使用 SSL 访问该虚拟机。

>**注意：** 确保端口 22 打开以便进行连接。 

## <a name="create-the-ssh-keys"></a>创建 SSH 密钥

1. Download the PuTTY tool. This will include <bpt id="p1">[</bpt>PuTTYgen<ept id="p1">](https://putty.org/)</ept>. 
2. 安装后，找到并打开 PuTTYgen 程序。
3. 在“参数”选项组中选择“RSA”。
4. 单击“生成”按钮。
5. 在窗口的空白区域中四处移动鼠标，以造成某种程度的随机性。
6. 复制“要粘贴到授权密钥文件中的公钥”的文本。
7. Optionally you can specify a <bpt id="p1">**</bpt>Key passphrase<ept id="p1">**</ept> and then <bpt id="p2">**</bpt>Confirm passphrase.<ept id="p2">**</ept> You will be prompted for the passphrase when you authenticate to the VM with your private SSH key. Without a passphrase, if someone obtains your private key, they can sign in to any VM or service that uses that key. We recommend you create a passphrase. However, if you forget the passphrase, there is no way to recover it.
8. 单击“保存私钥”。
9. Choose a location and filename and click <bpt id="p1">**</bpt>Save<ept id="p1">**</ept>. You will need this file to access the VM. 

## <a name="create-the-linux-machine-and-assign-the-public-ssh-key"></a>创建 Linux 计算机并分配 SSH 公钥

1. 在门户中创建所选的 Linux 虚拟机。
2. 为“身份验证类型”选择“SSH 公钥”（而不是“密码”）。
3. 提供一个用户名。
4. Paste the public SSH key from PuTTY into the <bpt id="p1">**</bpt>SSH public key<ept id="p1">**</ept> text area. Ensure the key validates with a checkmark. 
5. Create the VM. Wait for it to deploy.
6. 访问正在运行的 VM。 
7. 在“概述”边栏选项卡中，单击“连接”。
8. 记下你的登录信息，包括用户和公共 IP 地址。

## <a name="access-the-server-using-ssh"></a>使用 SSH 访问服务器

1. 打开 PuTTY 工具。
2. 输入 username@publicIpAddress，其中的 username 是在创建 VM 时分配的值，publicIpAddress 是从 Azure 门户获取的值。
3. 为“端口”指定“22”。
4. 在“连接类型”选项组中选择“SSH”。
5. 在“类别”面板中导航到“SSH”，然后单击“身份验证”。
6. 单击“用于身份验证的私钥文件”旁边的“浏览”按钮。
7. 导航到生成 SSH 密钥时保存的私钥文件，然后单击“打开”。
8. 在 PuTTY 主屏幕上单击“打开”。
9. 现在将连接到服务器命令行。 