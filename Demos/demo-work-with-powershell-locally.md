# <a name="demonstration-work-with-powershell-locally"></a>演示：在本地使用 PowerShell

In this demonstration, we will install Azure Az PowerShell module. The Az module is available from a global repository called the <bpt id="p1">*</bpt>PowerShell Gallery<ept id="p1">*</ept>. You can install the module onto your local machine through the <bpt id="p1">**</bpt>Install-Module<ept id="p1">**</ept> command. You need an elevated PowerShell shell prompt to install modules from the PowerShell Gallery. 

## <a name="install-the-az-module"></a>安装 Az 模块

1. 打开“开始”菜单，然后键入“Windows PowerShell”。
2. 右键单击“Windows PowerShell”图标，然后选择“以管理员身份运行”。
3. 在“用户帐户控制”对话框中，选择“是”。
4. Type the following command, and then press Enter. This command installs the module for all users by default. (It's controlled by the scope parameter.) AllowClobber overwrites the previous PowerShell module. 

    ```
    Install-Module -Name Az -AllowClobber
    ```

## <a name="install-nuget-if-needed"></a>安装 NuGet（如果需要）

1. 根据安装的 NuGet 版本，可能会收到下载并安装最新版本的提示。
2. 如果出现提示，请安装并导入 NuGet 提供程序。

## <a name="trust-the-repository"></a>信任存储库

1. 在本演示中，我们将安装 Azure Az PowerShell 模块。

    ```
    You are installing the modules from an untrusted repository. If you trust this repository, change its
    InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from
    'PSGallery'?
    ```

2. 根据提示安装模块。 

## <a name="connect-to-azure-and-view-your-subscription-information"></a>连接至 Azure 并查看订阅信息

1. 登录到 Azure

    ```
    Login-AzAccount
    ```

2. 出现提示时，请提供凭据。
3. 验证订阅信息。

    ```
    Get-AzSubscription
    ```

## <a name="create-resources"></a>创建资源

1. Az 模块可从名为“PowerShell 库”的全局存储库中获得。

    ```
    New-AzResourceGroup -name <name> -location <location>
    ```

2. 验证资源组。 
  
    ```
    Get-AzResourceGroup
    ```

3. 可通过 Install-Module 命令该将模块安装到本地计算机上。 

    ```
    Remove-AzResourceGroup -Name Test
    ```