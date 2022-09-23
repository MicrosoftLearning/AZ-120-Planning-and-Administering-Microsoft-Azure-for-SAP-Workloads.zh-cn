# <a name="demonstration-work-with-azure-cli-locally"></a>演示：在本地使用 Azure CLI

在本演示中，将安装并使用 CLI 来创建资源。

## <a name="install-the-cli-on-windows"></a>在 Windows 上安装 CLI

使用 MSI 安装程序在 Windows 操作系统上安装 Azure CLI：

1. 下载 [MSI installer](https://aka.ms/installazurecliwindows)，在浏览器安全对话框中，单击“运行”。
2. 在安装程序中，接受许可条款，然后单击“安装”。
3. 在“用户帐户控制”对话框中，选择“是”。

## <a name="verify-azure-cli-installation"></a>验证 Azure CLI 安装

通过打开 Bash shell（适用于 Linux 或 macOS）运行 Azure CLI，或从命令提示符或 PowerShell（适用于 Windows）中运行 Azure CLI。

启动 Azure CLI 并通过运行版本检查来验证安装：

```azurecli
az --version
 ```

><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Running Azure CLI from PowerShell has some advantages over running Azure CLI from the Windows command prompt. PowerShell provides more tab completion features than the command prompt.

## <a name="login-to-azure"></a>登录到 Azure

Because you're working with a local Azure CLI installation, you'll need to authenticate before you can execute Azure commands. You do this by using the Azure CLI <bpt id="p1">**</bpt>login<ept id="p1">**</ept> command:

```azurecli
az login
```

Azure CLI will typically launch your default browser to open the Azure sign-in page. If this doesn't work, follow the command-line instructions and enter an authorization code at <bpt id="p1">[</bpt><ph id="ph1">https://aka.ms/devicelogin</ph><ept id="p1">](https://aka.ms/devicelogin)</ept>.

成功登录后，将连接到 Azure 订阅。

## <a name="create-a-resource-group"></a>创建资源组

由于在创建新的 Azure 服务之前，通常需要创建新的资源组，因此我们将使用资源组作为示例来说明如何从 CLI 创建 Azure 资源。

Azure CLI <bpt id="p1">**</bpt>group create<ept id="p1">**</ept> command creates a resource group. You must specify a name and location. The <bpt id="p1">*</bpt>name<ept id="p1">*</ept> must be unique within your subscription. The <bpt id="p1">*</bpt>location<ept id="p1">*</ept> determines where the metadata for your resource group will be stored. You use strings like "West US", "North Europe", or "West India" to specify the location; alternatively, you can use single word equivalents, such as westus, northeurope, or westindia. The core syntax is:

```azurecli
az group create --name <name> --location <location>
```

## <a name="verify-the-resource-group"></a>验证资源组

For many Azure resources,  Azure CLI provides a <bpt id="p1">**</bpt>list<ept id="p1">**</ept> subcommand to view resource details. For example, the Azure CLI <bpt id="p1">**</bpt>group list<ept id="p1">**</ept> command lists your Azure resource groups. This is useful to verify whether resource group creation was successful:

```azurecli
az group list
```

若要获得更简洁的视图，可将输出的格式设为简单的表：

```azurecli
az group list --output table
```

If you have several items in the group list, you can filter the return values by adding a <bpt id="p1">**</bpt>query<ept id="p1">**</ept> option. Try this command:

```azurecli
az group list --query "[?name == '<rg name>']"
```

><bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> You format the query using <bpt id="p2">**</bpt>JMESPath<ept id="p2">**</ept>, which is a standard query language for JSON requests. Learn more about this powerful filter language at <bpt id="p1">[</bpt><ph id="ph1">http://jmespath.org/</ph><ept id="p1">](http://jmespath.org/)</ept>.