---
lab:
  title: 01b - 在 Azure VM 上实现 Windows 群集
  module: Module 01 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 模块 1：了解 Azure 上的 SAP 的 IaaS 的基础
# 实验室 1b：在 Azure VM 上实现 Windows 群集

估计时间：120 分钟

本实验室中的所有任务都是从 Azure 门户中执行的（包括 PowerShell Cloud Shell 会话）  

   > **注意**：如果没有使用 Cloud Shell，实验室虚拟机必须安装 Az PowerShell 模块：[**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi)。

实验室文件：无

## 方案
  
为了准备在 Azure 上部署 SAP NetWeaver，使用 SQL Server 作为数据库管理系统，Adatum Corporation 需要了解在运行 Windows Server 2022 的 Azure VM 上实现群集的过程。

## 目标
  
完成本实验室后，你将能够：

-   预配必要的 Azure 计算资源以支持高度可用的 SAP NetWeaver 部署。

-   配置运行 Windows Server 2022 的 Azure VM 的操作系统，以支持高可用性 SAP NetWeaver 部署。

-   预配必要的 Azure 网络资源以支持高度可用的 SAP NetWeaver 部署。

## 要求

-   在打算用于本实验室的 Azure 区域中，具有数量充足的可用 DSv2 和 Dsv3 vCPU（一个 Standard_DS1_v2 VM，1 个 vCPU；四个 Standard_D4s_v3 VM，每个 VM 4 个 vCPU）的 Microsoft Azure 订阅

-   安装了兼容 Azure Cloud Shell 的 Web 浏览器且具有 Azure 访问权限的实验室计算机

> **注意**：确保为资源部署选择的 Azure 区域支持可用性区域。 有关此类区域的列表，请参阅 (https://docs.microsoft.com/en-us/azure/availability-zones/az-overview)。 考虑使用“美国东部”或“美国东部 2”********。

## 练习 1：预配必要的 Azure 计算资源以支持高度可用的 SAP NetWeaver 部署

持续时间：50 分钟

在本练习中，你将部署在运行 Windows Server 2022 的 Azure VM 上配置故障转移群集所需的 Azure 基础结构计算组件。 此过程涉及到部署一对 Active Directory 域控制器，然后部署一对运行 Windows Server 2022 的 Azure VM。 每对 VM 将在同一虚拟网络中的单独可用性区域中分开放置。 若要自动部署域控制器，你将使用 Azure 资源管理器快速启动模板，可从 <https://aka.ms/az120-1bdeploy> 获得

### 任务 1：使用 Bicep 模板，部署一对运行高可用性 Active Directory 域控制器的 Azure VM

1. 从实验室计算机中启动 Web 浏览器，并导航到 Azure 门户 (https://portal.azure.com )。

1. 如果出现提示，则使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1. 在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

    > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。 如果是，接受默认设置，这样会在自动生成的资源组中创建存储账户。

1. 在 Cloud Shell 窗格中，运行以下命令，为托管 Bicep 模板的存储库创建浅表克隆，以用于部署运行高度可用的 Active Directory 域控制器的一对 Azure VM，并将当前目录设置为该模板及其参数文件的位置：

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/
    ```

1. 在 Cloud Shell 窗格中运行以下命令，将 `$rgName` 变量的值设置为 `az12001b-ad-RG`：

    ```
    $rgName = 'az12001b-ad-RG'
    ```

1. 在 Cloud Shell 窗格中，运行以下命令，将 `$location` 变量的值设置为支持可用性区域并且你打算在其中部署实验室 VM 的 Azure 区域的名称（将 `<Azure_region>` 占位符替换为该区域的名称）：

    ```
    $location = '<Azure_region>'
    ```

1. 在 Cloud Shell 窗格中，运行以下命令以在选择的 Azure 区域中创建名为 az12001b-ad-RG 的资源组：

    ```
    New-AzResourceGroup -Name $rgName -Location $location
    ```

1. 在 Cloud Shell 窗格中，运行以下命令以设置变量 `$deploymentName` 的值：

    ```
    $deploymentName = 'az1201b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. 在 Cloud Shell 窗格中，运行以下命令以设置管理用户帐户的名称及其密码（分别将 `<username>` 和 `<password>` 占位符替换为管理用户帐户的名称及其密码的值）：

    ```
    $adminUsername = '<username>'
    $adminPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
    ```

    > **注意**：确保密码满足适用于部署运行 Windows 的 Azure VM 的复杂性要求（长度至少为 12 个字符，包含大小写字母、数字和特殊字符）。

1. 在 Cloud Shell 窗格中，运行以下命令以运行部署：

    ```
    New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $rgName -TemplateFile .\main.bicep -TemplateParameterFile .\azuredeploy.parameters.json -adminUsername $adminUsername -adminPassword $adminPassword -c
    ```

1. 查看命令的输出，并验证它是否不包含任何错误和警告。 出现提示时，按 Enter 键继续部署。

    > **注意**：该部署大约需要 30 分钟才能完成。 等待部署完成后再继续下一个任务。

    > **注意**：如果部署失败并出现包括语句 `PowerShell DSC resource MSFT_xADDomainController failed to execute Set-TargetResource functionality with error message: Domain 'adatum.com' could not be found` 的错误，请使用以下步骤修正此问题：

    - 在 Azure 门户中，导航到 **adVNET** 的边栏选项卡，在左侧垂直导航菜单的“设置”**** 部分，选择 **DNS 服务器**，在 **adVNET \| DNS 服务器**页上，删除 **10.0.0.5** 项，然后选择“保存”****。
      
    - 在 Azure 门户中导航到 **adBDC** VM 的边栏选项卡，在左侧垂直导航菜单的“设置”**** 部分中，选择**扩展 + 应用程序**，在**扩展 + 应用程序**窗格中，选择**PrepareBDC**，然后在**准备 BDC**窗格中选择“卸载”****。 

    - 导航回到“**adBDC**”VM 边栏选项卡并重启 Azure VM。

    - 导航到“**az1201b-ad-RG**”边栏选项卡，在左侧垂直导航菜单的“**设置**”部分中，选择“**部署**”。

    - 在“**az1201b-ad-RG \| 部署**”边栏选项卡上，选择以 **az1201b** 前缀开头的部署，然后在部署边栏选项卡上选择“**重新部署**”。

    - 在“**自定义部署**”边栏选项卡中的“**管理员密码**”文本框中，输入在原始部署期间使用的相同密码，选择“**查看 + 创建**”，然后选择“**创建**”。

    - 请勿等待重新部署完成，而是继续进行下一项任务。 重新部署大约需要 3 分钟时间才能完成。

### 任务 2：将一对运行 Windows Server 2022 的 Azure VM 部署到不同的可用性区域

1. 从实验室计算机登录 Azure 门户，导航到“虚拟机”边栏选项卡，单击“+ 创建”，然后从下拉菜单中选择“Azure 虚拟机”  。

1. 在“创建虚拟机”边栏选项卡中，使用以下设置启动“Windows Server 2022 Datacenter: Azure Edition - Gen2”Azure VM 的预配（所有其他 VM 保留默认值） ：
    
    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称  |
    | 资源组 | 新资源组的名称 az12001b-cl-RG |
    | **虚拟机名称** | az12001b-cl-vm0 |
    | **区域** | 在上一个任务中部署 Azure VM 时所在的 Azure 区域 |
    | **可用性选项** | **可用性区域** |
    | **可用性区域** | **区域 1** |
    | 安全类型**** | **标准** |
    | **图像** | 选择“Windows Server 2022 Datacenter: Azure Edition - Gen2” |
    | **大小** | 标准 D4s v3**** |
    | **用户名** | 在本练习的前面部分部署 Bicep 模板时指定的用户名 |
    | **密码** | 在本练习的前面部分部署 Bicep 模板时指定的密码 |
    | **公共入站端口** | **无** |
    | **是否要使用现有的 Windows Server 许可证？** | **是** |
    | **OS 磁盘类型** | **高级·SSD** |
    | **虚拟网络** | adVNET |
    | **子网名称** | 名为 clSubnet 的新子网 |
    | **子网地址范围** | **10.0.1.0/24** |
    | **公共 IP 地址** | **无** |
    | **NIC 网络安全组** | **无**  |
    | **启用加速网络** | **开** |
    | 负载平衡选项 | **无** |
    | **启用系统分配的托管标识** | 关闭 |
    | 通过 Azure AD 登录 | 关闭 |
    | 启用自动关闭 | 关闭 |
    | 补丁业务流程选项 | 手动更新 |
    | **启动诊断** | **禁用** |
    | **扩展** | *无* |
    | **标记** | *无* |

1. 请勿等待预配完成，而是继续进行下一步骤。

1. 使用以下设置预配另一个“Windows Server 2022 Datacenter: Azure Edition - Gen2”Azure VM：
     
    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称  |
    | 资源组 | 在此任务中部署第一个“Windows Server 2022 Datacenter: Azure Edition - Gen2”Azure VM 时使用的资源组的名称** |
    | **虚拟机名称** | az12001b-cl-vm1 |
    | **区域** | 在此任务中部署第一个“Windows Server 2022 Datacenter: Azure Edition - Gen2”Azure VM 时所在的 Azure 区域** |
    | **可用性选项** | **可用性区域** |
    | **可用性区域** | **区域 2** |
    | 安全类型**** | **标准** |
    | **图像** | 选择“Windows Server 2022 Datacenter: Azure Edition - Gen2” |
    | **大小** | 标准 D4s v3**** |
    | **用户名** | 在本练习的前面部分部署 Bicep 模板时指定的用户名 |
    | **密码** | 在本练习的前面部分部署 Bicep 模板时指定的密码 |
    | **公共入站端口** | **无** |
    | **是否要使用现有的 Windows Server 许可证？** | **是** |
    | **OS 磁盘类型** | **高级·SSD** |
    | **虚拟网络** | adVNET |
    | **子网名称** | clSubnet |
    | **公共 IP 地址** | **无** |
    | **NIC 网络安全组** | **无**  |
    | **启用加速网络** | **开** |
    | 负载平衡选项 | **无** |
    | 通过 Azure AD 登录 | 关闭 |
    | 启用自动关闭 | 关闭 |
    | 补丁业务流程选项 | 手动更新 |
    | **启动诊断** | **禁用** |
    | **扩展** | *无* |
    | **标记** | *无* |

1. 等待预配完成。 这可能需要几分钟。

### 任务 3：创建并配置 Azure VM 磁盘

1. 在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在上一个任务中预配的资源所在的资源组的名称：

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. 在 Cloud Shell 窗格中运行以下命令，创建第一组共计 4 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第一个 Azure VM：

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location
    
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm0').Zones

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1. 在 Cloud Shell 窗格中运行以下命令，创建第二组共计 4 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第二个 Azure VM：

    ```
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm1').Zones
    
    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone
        
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1. 在 Azure 门户中，导航到你在上一个任务中预配的第一个 Azure VM (az12001b-vm0) 的边栏选项卡。****

1. 从“az12001b-cl-vm0”边栏选项卡导航到“az12001b-cl-vm0 - 磁盘”边栏选项卡。********

1. 从“az12001b-cl-vm0 - 磁盘”边栏选项卡，将具有以下设置的数据磁盘附加到 az12001b-cl-vm0：****
   
   | 设置 | 值 |
   |   --    |  --   |
   | **LUN** | **0** |
   | **磁盘名称** | az12001b-cl-vm0-DataDisk0 |
   | 资源组 | 在上一个任务中部署那对“Windows Server 2022 Datacenter”Azure VM 时使用的资源组的名称** |
   | 主机缓存 | **只读** |

1. 重复上一步，附加前缀为 az12001b-cl-vm0-DataDisk 的其余 3 个磁盘（共 4 个）。**** 分配与磁盘名称的最后一个字符匹配的 LUN 编号。 对于最后一个磁盘 (LUN 3)，将主机缓存设置为“无”。********

1. 保存所做更改。 

1. 在 Azure 门户中，导航到你在上一个任务中预配的第二个 Azure VM (az12001b-cl-vm1) 的边栏选项卡。****

1. 从“az12001b-cl-vm1”边栏选项卡中，导航到“az12001b-cl-vm1 - 磁盘”边栏选项卡。********

1. 从“az12001b-cl-vm1 - 磁盘”边栏选项卡中，将具有以下设置的数据磁盘附加到 az12001b-cl-vm1：****
     
   | 设置 | 值 |
   |   --    |  --   |
   | **LUN** | **0** |
   | **磁盘名称** | az12001b-cl-vm1-DataDisk0 |
   | 资源组 | 在上一个任务中部署那对“Windows Server 2022 Datacenter”Azure VM 时使用的资源组的名称** |
   | 主机缓存 | **只读** |

1. 重复上一步，附加前缀为 az12001b-cl-vm1-DataDisk 的其余 3 个磁盘（共 4 个）。**** 分配与磁盘名称的最后一个字符匹配的 LUN 编号。 对于最后一个磁盘 (LUN 3)，将主机缓存设置为“无”。********

1. 保存所做更改。 

#### 任务 4：预配 Azure Bastion 

> **注意**：Azure Bastion 允许在没有公共终结点的情况下连接到 Azure VM（你在本练习上一个任务中部署的虚拟机），同时提供保护，以防范针对操作系统级别凭据的暴力攻击。

> **注意**：要使用 Azure Bastion，请确保浏览器已启用弹出窗口功能。

1. 在显示 Azure 门户的浏览器窗口中，打开另一个选项卡，并在浏览器选项卡中导航到 [**Azure 门户**](https://portal.azure.com)。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的虚拟网络 az12001a-RG-vnet 中********：

   ```powershell
   $resourceGroupName = 'az12001b-cl-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'adVNET'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中，搜索并选择“Bastion”，然后从“Bastion”边栏选项卡中选择“+ 创建”  。
1. 在“创建 Bastion”边栏选项卡的“基本”选项卡上，指定以下设置并选择“查看 + 创建”  ：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|**az12001b-cl-RG**|
   |名称|**az12001b-bastion**|
   |区域|在本练习的先前任务中部署资源的同一 Azure 区域|
   |层|**基本**|
   |虚拟网络|adVNET|
   |子网|**AzureBastionSubnet (10.0.255.0/24)**|
   |公共 IP 地址|**新建**|
   |公共 IP 名称|**adVNET-ip**|

1. 在“创建 Bastion”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”  ：

   > **注意**：不必等到部署完成，而是继续执行本次练习的下一项任务。 部署可能需要大约 5 分钟时间。

> **Result**：完成本练习后，你已预配了支持高可用性 SAP NetWeaver 部署所需的 Azure 计算资源。

## 练习 2：配置运行 Windows Server 2022 Datacenter 的 Azure VM 的操作系统，以支持高度可用的 SAP NetWeaver 安装

持续时间：40 分钟

### 任务 1：将 Windows Server 2022 Datacenter VM 加入 Active Directory 域。

   > **注意**：在开始此任务之前，请确保你在上一个练习的最后一个任务中启动的模板部署已完成。 

1. 在 Azure 门户中，导航到虚拟网络 adVNET 的边栏选项卡，该虚拟网络已在本实验室的第一个练习中自动预配。****

1. 显示“adVNET - DNS 服务器”边栏选项卡，请注意，虚拟网络配置了专用 IP 地址，该地址作为 DNS 服务器分配给本实验第一个练习中部署的域控制器。****

1. 在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为资源组的名称，其中包含你在上一个练习中预配的那对“Windows Server 2022 Datacenter: Azure Edition - Gen2”Azure VM：

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. 在 Cloud Shell 窗格中，运行以下命令，将上一练习的第二个任务中部署的 Windows Server 2022 Azure VM 加入到 adatum.com Active Directory 域（将 `<username>` 和 `<password>` 占位符替换为在本实验室的第一个练习中部署 Bicep 模板时指定的管理用户帐户的名称和密码）：

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\<username>", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1. 请等待脚本完成，再执行下一个任务。

### 任务 2：在运行 Windows Server 2022 的 Azure VM 上配置存储，以支持高可用性 SAP NetWeaver 安装。

1. 在 Azure 门户中，导航到你在本实验室的第一个练习中预配的虚拟机 az12001b-cl-vm0 的边栏选项卡。****

1. 在“az12001b-cl-vm0”边栏选项卡中，选择“连接”，在下拉菜单中选择“通过 Bastion 连接”，在“az12001b-cl-vm0”的“Bastion”选项卡上，将“身份验证类型”保留为“VM 密码”，提供部署“az12001b-cl-vm0”虚拟机时设置的凭据，保留启用的复选框“在新浏览器选项卡中打开”，然后选择“连接”****************************************。

    > 注意：请确保使用 ADATUM 域帐户而不是操作系统级别的帐户登录（即确保用户名前面有 ADATUM\\ 前缀）  。

1. 在与 az12001b-cl-vm0 的 Bastion 会话中，在服务器管理器中导航到“文件和存储服务” -> “服务器”节点********。 

1. 导航到“存储池”视图，验证是否显示了你在上一个练习中附加到 Azure VM 的所有磁盘。****

1. 使用“新建存储池向导”创建具有以下设置的新存储池：****
     
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | 数据存储池 |
    | **物理磁盘** | 选择磁盘编号与前三个 LUN 编号 (0-2) 对应的 3 个磁盘，并将其分配设置为“自动” |

    > **注意**：请使用“机箱”列中的条目来标识 LUN 编号。********

1. 使用“新建虚拟磁盘向导”创建具有以下设置的新虚拟磁盘：****
     
    | 设置 | “值” |
    |   --    |  --   |
    | 虚拟磁盘名称 | 数据虚拟磁盘 |
    | 存储布局 | **简单** |
    | **Provisioning** | **固定** |
    | **大小** | “最大大小”**** |

1. 使用“新建卷向导”创建具有以下设置的新卷：****
    
    | 设置 | “值” |
    |   --    |  --   |
    | 服务器和磁盘 | 接受默认值 |
    | **大小** | 接受默认值 |
    | **驱动器号** | **M** |
    | **文件系统** | ReFS**** |
    | 分配单位大小 | **默认** |
    | **卷标签** | **数据** |

1. 返回“存储池”视图，使用“新建存储池向导”创建具有以下设置的新存储池：********
    
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | 日志存储池 |
    | **物理磁盘** | 选择 4 个磁盘中的最后一个磁盘并将其分配设置为“自动” |

1. 使用“新建虚拟磁盘向导”创建具有以下设置的新虚拟磁盘：****
    
    | 设置 | “值” |
    |   --    |  --   |
    | 虚拟磁盘名称 | 日志虚拟磁盘 |
    | 存储布局 | **简单** |
    | **Provisioning** | **固定** |
    | **大小** | “最大大小”**** |

1. 使用“新建卷向导”创建具有以下设置的新卷：****
    
    | 设置 | “值” |
    |   --    |  --   |
    | 服务器和磁盘 | 接受默认值 |
    | **大小** | 接受默认值 |
    | **驱动器号** | **L** |
    | **文件系统** | ReFS**** |
    | 分配单位大小 | **默认** |
    | **卷标签** | **日志** |

1. 重复此任务中的上一步，配置 az12001b-cl-vm1 上的存储。

### 任务 3：准备在运行 Windows Server 2022 的 Azure VM 上配置故障转移群集，以支持高可用性 SAP NetWeaver 安装。

1. 在与 az12001b-cl-vm0 的 Bastion 会话中，启动 Windows PowerShell ISE 会话并运行以下命令，在 az12001b-cl-vm0 和 az12001b-cl-vm1 上安装故障转移群集和远程管理工具功能：

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注意**：这将导致重启两个 Azure VM 的来宾操作系统。

1. 在实验室计算机的 Azure 门户中单击“+ 创建资源”。****

1. 从“新建”边栏选项卡，使用以下设置新建存储帐户：********
    
    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称 |
    | 资源组 | 在上一个练习中预配的那对“Windows Server 2022 Datacenter”Azure VM 所在的资源组的名称** |
    | **存储帐户名称** | 任何由 3 到 24 个字母和数字组成的唯一名称 |
    | **位置** | 在上一个练习中部署 Azure VM 时所在的 Azure 区域 |
    | **“性能”** | **标准** |
    | **冗余** | **本地冗余存储 (LRS)** |
    | **连接方法** | 公共终结点（所有网络） |
    | **需要安全传输才能进行 REST API 操作** | **已启用** |
    | **大型文件共享** | **已禁用** |
    | blob、容器和文件的软删除 | **已禁用** |
    | **分层命名空间** | **已禁用** |

### 任务 4：在运行 Windows Server 2022 的 Azure VM 上配置故障转移群集，以支持高可用性 SAP NetWeaver 安装。

1. 在 Azure 门户中，导航到你在本实验室的第一个练习中预配的虚拟机 az12001b-cl-vm0 的边栏选项卡。****

1. 从“az12001b-cl-vm0”边栏选项卡，使用 Azure Bastion 连接到虚拟机来宾操作系统****。 当系统提示进行身份验证时，请提供在本实验室的第一个练习中部署 Bicep 模板时指定的管理用户帐户的凭据。

1. 在与 az12001b-cl-vm0 的 Bastion 会话中，从服务器管理器的“工具”菜单，启动 Active Directory 管理中心********。

1. 在 Active Directory 管理中心，在 adatum.com 域的根目录下创建一个名为“群集”的新组织单位。****

1. 在 Active Directory 管理中心，将 az12001b-cl-vm0 和 az12001b-cl-vm1 的计算机帐户从“计算机”容器移动到“群集”组织单位。****************

1. 在与 az12001b-cl-vm0 的 Bastion 会话中，启动 Windows PowerShell ISE 会话，并运行以下命令来创建新群集：

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1. 在与 az12001b-cl-vm0 的 Bastion 会话中，切换到“Active Directory 管理中心”控制台****。

1. 在 Active Directory 管理中心，导航到“群集”组织单位并显示其“属性”窗口。******** 

1. 在“群集”组织单位的“属性”窗口中，导航到“扩展”部分，显示“安全性”选项卡。**************** 

1. 在“安全”选项卡上，单击“高级”按钮，打开“群集的高级安全设置”窗口。************ 

1. 在“群集高级安全设置”窗口的“权限”选项卡，单击“添加”。************

1. 在“群集的权限条目”窗口中，单击“选择主体”********

1. 在“选择用户、服务帐户或组”对话框，单击“对象类型”，启用“计算机”条目旁边的复选框，然后单击“确定”。**************** 

1. 回到“选择用户、计算机、服务帐户或组”对话框，在“输入要选择的对象名称”中，键入“az12001b-cl-cl0”并单击“确定”。****************

1. 在“群集的权限条目”窗口中，确保“允许”显示在“类型”下拉列表中。************ 接下来，在“适用于”下拉列表中，选择“此对象和所有后代对象”。******** 在“权限”列表中，选中“创建计算机对象”和“删除计算机对象”复选框，然后单击“确定”两次。****************

1. 在 Windows PowerShell ISE 会话中，通过运行以下命令安装 Azure PowerShell 模块：

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. 在 Windows PowerShell ISE 会话中，通过运行以下命令使用 Azure AD 凭据进行身份验证：

    ```
    Add-AzAccount
    ```

    > **注意**：出现提示时，使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1. 在 Windows PowerShell ISE 会话中，通过运行以下命令设置新群集的云见证仲裁：

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. 要在与 az12001b-cl-vm0 的 Bastion 会话中验证生成的配置，需从服务管理器的“工具”菜单启动“故障转移群集管理器”********。

1. 在“故障转移群集管理器”控制台中，查看 az12001b-cl-cl0 群集配置，包括其节点、证明和网络设置。******** 请注意，群集没有任何共享存储。

1. 终止与 az12001b-cl-vm0 的 Bastion 会话。

> **结果**：完成本练习后，你已配置运行 Windows Server 2022 的 Azure VM 操作系统，以支持高可用性 SAP NetWeaver 安装


## 练习 3：预配必要的 Azure 网络资源以支持高度可用的 SAP NetWeaver 部署

持续时间：30 分钟

在本练习中，你将实现 Azure 负载均衡器，以适应 SAP NetWeaver 的群集安装。

### 任务 1：配置 Azure VM 以辅助负载均衡设置。

   > **注意**：由于你将设置一对 Stardard SKU 的 Azure 负载均衡器，因此你需要首先删除与两个 Azure VM 网络适配器关联的公共 IP 地址，这两个 Azure VM 将用作负载均衡后端池。

1. 在实验室计算机的 Azure 门户中，导航到 Azure VM az12001b-cl-vm0 的边栏选项卡。**** 

1. 从“az12001b-cl-vm0”边栏选项卡，导航到与其网络适配器关联的公共 IP 地址 az12001b-cl-vm0-ip 的边栏选项卡。********

1. 从“az12001b-cl-vm0-ip”边栏选项卡，先将公共 IP 地址与网络接口解除关联，然后将其删除。****

1. 在 Azure 门户中，导航到 Azure VM az12001b-cl-vm1 的边栏选项卡。**** 

1. 从“az12001b-cl-vm1”边栏选项卡中，导航到与其网络适配器关联的公共 IP 地址 az12001b-cl-vm1-ip 的边栏选项卡。********

1. 从“az12001b-cl-vm1-ip”边栏选项卡，先将公共 IP 地址与网络接口解除关联，然后将其删除。****

1. 在 Azure 门户中，导航到“az12001b-cl-vm0”Azure VM 的边栏选项卡****。

1. 从“az12001b-cl-vm0”边栏选项卡，导航到“网络”边栏选项卡********。 

1. 从“az12001b-cl-vm0 - 网络”边栏选项卡，导航到 az12001b-cl-vm0 的网络接口****。 

1. 在 az12001b-cl-vm0 的网络接口边栏选项卡中，导航到其 IP 配置边栏选项卡，并在该处显示其“ipconfig1”边栏选项卡****。

1. 在“ipconfig1”边栏选项卡中，将专用 IP 地址分配设置为“静态”，然后保存所做的更改。********

1. 在 Azure 门户中，导航到“az12001b-cl-vm1”Azure VM 的边栏选项卡****。

1. 从“az12001b-cl-vm1”边栏选项卡中，导航到“网络”边栏选项卡********。 

1. 从“az12001b-cl-vm1 - 网络”边栏选项卡，导航到 az12001b-cl-vm1 的网络接口****。 

1. 在 az12001b-cl-vm1 的网络接口边栏选项卡中，导航到其 IP 配置边栏选项卡，并在该处显示其“ipconfig1”边栏选项卡****。

1. 在“ipconfig1”边栏选项卡中，将专用 IP 地址分配设置为“静态”，然后保存所做的更改。********

### 任务 2：创建和配置处理入站流量的 Azure 负载均衡器

1. 在 Azure 门户中，单击“+ 创建资源”****。

1. 从“新建”边栏选项卡中，使用以下设置开始创建新的 Azure 负载均衡器：****
    
    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称 |
    | 资源组 | 在本实验室的第一个练习中预配的那对“Windows Server 2022 Datacenter”Azure VM 所在的资源组的名称** |
    | **Name** | az12001b-cl-lb0 |
    | **区域** | 在本实验室的第一项练习中部署 Azure VM 时所在的 Azure 区域* |
    | SKU | **标准** |
    | **类型** | **内部** |
    | **前端 IP 名称** | frontend-ip1 |
    | **虚拟网络** | adVNET |
    | **子网** | clSubnet |
    | IP 地址分配 | **静态** |
    | IP 地址 | 10.0.1.240 |
    | **可用性区域** | **区域冗余** |

1. 等待负载均衡器配置完毕，然后导航到它 Azure 门户中的边栏选项卡。

1. 从“az12001b-cl-lb0”边栏选项卡，添加具有以下设置的后端池：****
    
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | az12001b-cl-lb0-bepool |
    | **虚拟网络** | adVNET |
    | **后端池配置** | IP 地址 |
    | IP 地址 | 10.0.1.4 资源名称 az1201b-cl-vm0 |
    | IP 地址 | 10.0.1.5 资源名称 az1201b-cl-vm1 |

1. 从“az12001b-cl-lb0”边栏选项卡，添加具有以下设置的运行状况探测：****
    
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | az12001b-cl-lb0-hprobe |
    | **协议** | **TCP** |
    | 端口 | **59999** |
    | **时间间隔** | 5 秒 |
    | **不正常阈值** | 2 次连续失败 |

1. 在“az12001b-cl-lb0”边栏选项卡中，添加具有以下设置的网络负载均衡规则：****
     
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | az12001b-cl-lb0-lbruletcp1433 |
    | **IP 版本** | **IPv4** |
    | “前端 IP 地址” | 10.0.1.240 (LoadBalancerFrontEnd) |
    | HA 端口 | **已禁用** |
    | 协议 | **TCP** |
    | **端口** | **1433** |
    | 后端端口 | **1433** |
    | 后端池 | az12001b-cl-lb0-bepool（2 个虚拟机） |
    | 运行状况探测 | az12001b-cl-lb0-hprobe (TCP:59999) |
    | **会话暂留** | **无** |
    | **空闲超时(分钟)** | **4** |
    | **TCP 重置** | **已禁用** |
    | 浮动 IP (直接服务器返回) | **已启用** |

### 任务 3：创建和配置处理出站流量的 Azure 负载均衡器

1. 从 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在本实验的第一个练习中预配的那对 Windows Server 2022 Datacenter Azure VM 所在的资源组的名称：

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. 在 Cloud Shell 窗格中运行以下命令，创建第二个负载均衡器将使用的公共 IP 地址：

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1. 在 Cloud Shell 窗格中，运行以下命令以创建第二个负载均衡器：

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1. 关闭 Cloud Shell 窗格。

1. 在 Azure 门户中，导航到显示 Azure 负载均衡器 az12001b-cl-lb1 的属性的边栏选项卡。****

1. 在“az12001b-lb1”边栏选项卡中，单击“后端池”。********

1. 在“az12001b-cl-lb1 - 后端池”边栏选项卡中，单击“az12001b-cl-lb1-bepool”。********

1. 在“az12001b-cl-lb1-bepool”边栏选项卡中，指定以下设置并单击“保存”：********
    
    | 设置 | 值 |
    |   --    |  --   |
    | **虚拟网络** | adVNET (4 VM) |
    | **虚拟机** | az12001b-cl-vm0  IP 地址：ipconfig1 |
    | **虚拟机** | az12001b-cl-vm1  IP 地址：ipconfig1 |

1. 在“az12001b-cl-lb1”边栏选项卡中，单击“运行状况探测”。********

1. 从“az12001b-cl-lb1 - 运行状况探测”边栏选项卡，添加具有以下设置的运行状况探测：****
    
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | az12001b-cl-lb1-hprobe |
    | 协议 | **TCP** |
    | **端口** | **80** |
    | **时间间隔** | 5 秒 |
    | **不正常阈值** | 2 次连续失败 |

1. 在“az12001b-cl-lb1”边栏选项卡中，单击“负载均衡规则”。********

1. 从“az12001b-cl-lb1 - 负载均衡规则”边栏选项卡，添加具有以下设置的网络负载均衡规则：****
    
    | 设置 | 值 |
    |   --    |  --   |
    | **Name** | az12001b-cl-lb1-lbharule |
    | **IP 版本** | **IPv4** |
    | “前端 IP 地址” | *接受默认值* |
    | HA 端口 | **已禁用** |
    | 协议 | **TCP** |
    | **端口** | **80** |
    | 后端端口 | **80** |
    | 后端池 | az12001b-cl-lb1-bepool（2 个虚拟机） |
    | 运行状况探测 | az12001b-cl-lb1-hprobe (TCP:80) |
    | **会话暂留** | **无** |
    | **空闲超时(分钟)** | **4** |
    | **TCP 重置** | **已禁用** |
    | 浮动 IP (直接服务器返回) | **已禁用** |

> **Result**：完成本练习后，你已预配了支持高可用性 SAP NetWeaver 部署所需的 Azure 网络资源

## 练习 4：删除实验室资源

持续时间：10 分钟

在本练习中，你将删除本实验室中预配的所有资源。

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 Cloud Shell 图标打开“Cloud Shell”窗格，然后选择“PowerShell”作为 Shell。****

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在本实验的第一个练习中预配的那对 Windows Server 2022 Datacenter Azure VM 所在的资源组的名称：

    ```
    $resourceGroupNamePrefix = 'az12001b-'
    ```

1. 在 Cloud Shell 窗格中运行以下命令，列出你在本实验室中创建的所有资源组：

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. 验证输出是否只包含在本实验室中创建的资源组。 这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 Cloud Shell 窗格中运行以下命令，删除你在本实验室中创建的资源组

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. 关闭 Cloud Shell 窗格。

> **Result**：完成本练习后，你已经删除了本实验中使用的资源。
