# <a name="demonstration-review-common-storage-explorer-tasks"></a>演示：查看常见的存储资源管理器任务

>**注意**：
>- If you have an older version of the Storage Explorer, be sure to upgrade. These steps use version 1.6.2.
>- 就此演示而言，我们将只进行基本的存储帐户连接。

## <a name="download-and-install-storage-explorer"></a>下载并安装存储资源管理器

1. 下载并安装 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/) 
2. 安装后，启动该工具。
3. 查看发行说明和菜单选项。

## <a name="connect-to-an-azure-subscription"></a>连接到 Azure 订阅

1. In Storage Explorer, select <bpt id="p1">**</bpt>Manage Accounts<ept id="p1">**</ept>, second icon top left. This will take you to the Account Management Panel.
2. The left pane now displays all the Azure accounts you've signed in to. To connect to another account, select <bpt id="p1">**</bpt>Add an account<ept id="p1">**</ept>.
3. 若要登录到某个国家/地区云或 Azure Stack，请单击“Azure 环境”下拉列表并选择要使用的 Azure 云。 
4. 选择环境后，单击“登录...”按钮。**** 
5. 使用 Azure 帐户成功登录后，该帐户以及与该帐户关联的 Azure Stack 订阅会添加到左窗格中。 
6. 选择要使用的 Azure 订阅，并选择“应用”。
7. 左窗格会显示与所选 Azure 订阅关联的存储帐户。

>**注意**：下一节需要 Azure 存储帐户。 

## <a name="attach-an-azure-storage-account"></a>附加 Azure 存储帐户

1. 访问 Azure 门户和存储帐户。
2. 浏览“存储资源管理器”的选项。
3. 选择“访问密钥”并阅读有关使用密钥的信息。 
4. 若要在存储资源管理器中连接，需要存储帐户名称和 Key1 信息 。
5. 在存储资源管理器中，添加一个帐户。
6. 在“帐户名称”文本框中粘贴帐户名称，在“帐户密钥”文本框中粘贴帐户密钥（从 Azure 门户获取的“密钥 1”值），然后选择“下一步”。****
7. 如果你有旧版存储资源管理器，请务必升级。 
8. 右键单击存储帐户，并注意选项包括“在门户中打开”、“复制主键”和“添加到快速访问”。