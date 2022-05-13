---
ms.openlocfilehash: c24d0da0cd66795235076080e14c561c7399a16f
ms.sourcegitcommit: 30dae3c49fe96a790479d08844a71fcb7851aa46
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/15/2022
ms.locfileid: "139867894"
---
# <a name="az-120-module-4-deploy-sap-on-azure"></a>AZ 120 模块 4：部署 Azure 上的 SAP
# <a name="lab-3b-implement-sap-architecture-on-azure-vms-running-windows"></a>实验 3b：在运行 Windows 的 Azure VM 上实施 SAP 体系结构

预计时间：150 分钟

从 Azure 门户执行本实验室中的所有任务（包括 PowerShell Cloud Shell 会话）  

   > **注意**：如果没有使用 Cloud Shell，实验室虚拟机必须安装 Az PowerShell 模块：[ **https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi** ](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi)。

实验室文件：无

## <a name="scenario"></a>方案
  
为了准备在 Azure 上部署 SAP NetWeaver，Adatum Corporation 想要实现一个演示，将演示在运行 Windows Server 2016 的 Azure VM 上实现 SAP NetWeaver 的高可用性。

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

-   预配支持高可用性 SAP NetWeaver 部署所需的 Azure 资源

-   配置运行 Windows 的 Azure VM 的操作系统，以支持高可用性 SAP NetWeaver 部署

-   在运行 Windows 的 Azure VM 上配置群集，以支持高可用性 SAP NetWeaver 部署

## <a name="requirements"></a>要求

-   在支持可用性区域的 Azure 区域中，具有数量充足的可用 Dsv3 vCPU（四个 Standard_D2s_v3 VM，每个 VM 2 个 vCPU；六个 Standard_D4s_v3 VM，每个 VM 4 个 vCPU）的 Microsoft Azure 订阅

-   安装了兼容 Azure Cloud Shell 的 Web 浏览器且具有 Azure 访问权限的实验室计算机


## <a name="exercise-1-provision-azure-resources-necessary-to-support-highly-available-sap-netweaver-deployments"></a>练习 1：预配支持高度可用的 SAP NetWeaver 部署所需的 Azure 资源

持续时间：60 分钟

在本练习中，你将部署配置 Windows 群集所必需的 Azure 基础结构计算组件。 这将涉及在同一可用性集中创建一对运行 Windows Server 2016 的 Azure VM。

### <a name="task-1-deploy-a-pair-of-azure-vms-running-highly-available-active-directory-domain-controllers-by-using-an-azure-resource-manager-template"></a>任务 1：使用 Azure 资源管理器模板，部署一对运行高可用性 Active Directory 域控制器的 Azure VM

1.  从实验室计算机中启动网页浏览器，并导航到 Azure 门户网站： https://portal.azure.com

1.  如果出现提示，则使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1.  在 Azure 门户接口上，单击 **新建资源**。

1.  从 **新建** 边栏选项卡，开始创造一个新的 **模板部署（使用自定义模板部署）**

1.  从“自定义部署”边栏选项卡的“快速启动模板(免责声明)”下拉列表中，选择条目“application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones”，然后单击“选择模板”。   

    > **注意**：或者，可以启动部署，方法是导航到 Azure 快速启动模板页面 (<https://github.com/Azure/azure-quickstart-templates>)，找到名为“在单独的可用性区域中新建 2 个 Windows VM、1 个 AD 林、域和 2 个 DC”的模板，并通过单击“部署到 Azure”按钮启动部署。 

1.  在“使用可用性区域新建具有 2 个 DC 的 AD 域”边栏选项卡中，指定以下设置，然后单击“检查 + 创建”，然后单击“创建”以启动部署：  

    -   订阅：你的 Azure 订阅的名称

    -   资源组：新资源组名称 az12003b-ad-RG

    -   位置：可在其中部署 Azure VM 的 Azure 区域

    > **注意**：请考虑使用“美国东部”或“美国东部 2”区域来部署资源。 

    -   管理员用户名：**Student**

    -   位置：你在上方指定的同一 Azure 区域

    -   密码：Pa55w.rd1234

    -   域名：adatum.com

    -   DnsPrefix：使用任何唯一的有效 DNS 前缀

    -   VM 大小：Standard D2s\_v3

    -   _artifacts 位置： *https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/*

    -   _artifacts 位置 Sas 令牌：留空


    > **注意**：该部署大约需要 35 分钟。 等待部署完成后，再继续执行下一个任务。

    > **注意**：如果在 CustomScriptExtension 组件部署期间，部署失败并出现“冲突”错误消息，请按以下步骤修正此问题：

       - 在 Azure 门户中的“部署”边栏选项卡上，查看部署详细信息并确定 CustomScriptExtension 安装失败的 VM

       - 在 Azure 门户中，导航到上一步中确定的 VM 的边栏选项卡，选择“扩展”，从“扩展”边栏选项卡中删除 CustomScript 扩展

       - 在 Azure 门户中，导航到“az12003b-sap-RG”资源组边栏选项卡，选择“部署”，选择失败部署的链接，然后选择“重新部署”，选择目标资源组 (az12003b-sap-RG) 并提供根帐户的密码 (Pa55w.rd1234)。    

### <a name="task-2-provision-subnets-that-will-host-azure-vms-running-highly-available-sap-netweaver-deployment-and-the-s2d-cluster"></a>任务 2：提供将托管可运行高可用性 SAP NetWeaver 部署和 S2D 群集的 Azure VM 的子网。

1.  在 Azure 门户中，导航到 **az12003b-RG** 资源组的边栏选项。

1.  在 **az12003b-AD-RG** 资源组边栏选项卡的资源列表中，找到 **adVNET** 虚拟网络并单击相应条目以显示 **adVNET** 边栏选项卡。

1.  从 **adVNET** 边栏选项卡导航到 **adVNET - 子网** 边栏选项卡。 

1.  从 **adVNET - 子网** 边栏选项卡，创建一个新的子网，设置如下：

    -   名称：sapSubnet

    -   地址范围（CIDR 块）：10.0.1.0/24

1.  从 **adVNET - 子网** 边栏选项卡，创建一个新的子网，设置如下：

    -   名称：s2dSubnet

    -   地址范围（CIDR 块）：10.0.2.0/24

1.  在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

    > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。 如果是，接受默认设置，这样会在自动生成的资源组中创建存储账户。

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在上一个任务中预配的资源所在的资源组的名称：

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1.  在 Cloud Shell 窗格中运行以下命令，标识在上一个任务中创建的虚拟网络：

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1.  在 Cloud Shell 窗格中，运行以下命令，标识新建子网的资源 ID：

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1.  将生成的值复制到剪贴板。 稍后在下一个任务中将用到它。

### <a name="task-3-deploy-azure-resource-manager-template-provisioning-azure-vms-running-windows-server-2016-that-will-host-a-highly-available-sap-netweaver-deployment"></a>任务 3：部署可对运行 Windows Server 2016 的 Azure VM 进行预配的 Azure 资源管理器模板，而 Windows Server 2016 可托管高可用性 SAP NetWeaver 部署

1.  在实验室计算机上，在 Azure 门户中搜索并选择“模板部署（使用自定义模板进行部署）”。

1.  在“自定义部署”边栏选项卡的“快速启动模板(免责声明)”下拉列表中，键入“application-workloads/sap/sap-3-tier-marketplace-image-md”，然后单击“选择模板”。   

    > **注意**：确保使用 Microsoft Edge 或第三方浏览器。 请勿使用 Internet Explorer。

1.  在“SAP NetWeaver 3 层(托管磁盘)”边栏选项卡上，选择“编辑模板” 。

1.  在“编辑模板”边栏选项卡上，应用以下更改并选择“保存”： 

    -   在第 197 行中，将 `"dbVMSize": "Standard_E8s_v3",` 替换为 `"dbVMSize": "Standard_D4s_v3",`

1.  返回到“SAP NetWeaver 3 层(托管磁盘)”边栏选项卡上，指定以下设置，单击“查看 + 创建”，然后单击“创建”以启动部署：

    -   订阅：你的 Azure 订阅的名称

    -   资源组：新资源组名称 az12003b-sap-RG

    -   位置：你在本练习的第一个任务中指定的 Azure 区域

    -   SAP 系统 ID：I20

    -   堆栈类型：ABAP

    -   OS 类型：**Windows Server 2016 Datacenter**

    -   Dbtype：SQL

    -   SAP 系统大小：**演示**

    -   系统可用性：高可用性

    -   管理员用户名：**Student**

    -   身份验证类型：密码

    -   管理员密码或密钥：Pa55w.rd1234

    -   子网 ID：你在上一个任务中复制到剪贴板的值

    -   可用性区域：1,2

    -   位置：[resourceGroup().location]

    -   _artifacts 位置： **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/**

    -   _artifacts 位置 Sas 令牌：留空


1.  请勿等待部署完成，而是继续进行下一项任务。 

### <a name="task-4-deploy-the-scale-out-file-server-sofs-cluster"></a>任务 4：部署横向扩展文件服务器 (SOFS) 群集

在此任务中，你将使用 GitHub 中提供的 Azure 资源管理器快速启动模板部署将为 SAP ASCS 服务器托管文件共享的横向扩展文件服务器 (SOFS) 群集：[ **https://github.com/robotechredmond/301-storage-spaces-direct-md** ](https://github.com/robotechredmond/301-storage-spaces-direct-md) 

1.  在实验室计算机上，启动浏览器并浏览到 [ **https://github.com/robotechredmond/301-storage-spaces-direct-md** ](https://github.com/robotechredmond/301-storage-spaces-direct-md)。 

    > **注意**：确保使用 Microsoft Edge 或第三方浏览器。 请勿使用 Internet Explorer。

1.  在标题为 **使用托管磁盘创建 Windows Server 2016 的 Storage Spaces Direct (S2D) 横向扩展文件服务器（SOFS）群集** 的页面上，点击 **部署到 Azure**。 这样会自动将你的浏览器重定向到 Azure 门户并显示 **自定义部署** 边栏选项卡。

1.  在“自定义部署”边栏选项卡中，指定以下设置，单击“查看 + 创建”，然后单击“创建”以启动部署：

    -   订阅：**你的 Azure 订阅名**.

    -   资源组：新资源组名称 az12003b-s2d-RG

    -   区域：在本练习的上一个任务中部署 Azure VM 的相同 Azure 区域

    -   名称前缀：i20

    -   VM 大小：Standard D4s\_v3

    -   启用加速网络：**是**

    -   映像 SKU：2016-Datacenter-Server-Core

    -   VM 数量：**2**

    -   VM 磁盘大小：128

    -   VM 磁盘数量：**3**

    -   现有域名：**adatum.com**

    -   管理员用户名：**Student**

    -   管理员密码：Pa55w.rd1234

    -   现有虚拟网络 RG 名：az12003b-ad-RG

    -   现有虚拟网络名：**adVNet**

    -   现有子网名：**s2dSubnet**

    -   Sofs 名：**sapglobalhost**

    -   共享名：**sapmnt**

    -   计划更新日：**星期日**

    -   计划更新时间：上午 3:00

    -   已启用实时反恶意软件： **false**

    -   已启用预定反恶意软件： **false**

    -   计划反恶意软件时间：120

    -   \__artifacts 位置：接受默认值

    -   \__artifacts 位置 Sas 令牌：保留默认值

1.  部署可能大约需要 20 分钟。 请勿等待部署完成，而是继续进行下一项任务。

### <a name="task-5-deploy-a-jump-host"></a>任务 5：部署跳板机

   > **注意**：由于无法从 Internet 访问你在上一个任务中部署的 Azure VM，因此你将部署运行 Windows Server 2016 Datacenter 的 Azure VM 作为跳板机。 

1.  在实验室计算机的 Azure 门户界面中，单击“+ 创建资源”。

1.  在“新建”边栏选项卡中，基于“Windows Server 2019 Datacenter - Gen1”图像，开始创建新的 Azure VM 。

1.  使用以下设置预配 Azure VM：

    -   订阅：你的 Azure 订阅的名称

    -   资源组：新资源组名称 az12003b-dmz-RG

    -   虚拟机名称：az12003b-vm0

    -   区域：在本练习的上一个任务中部署 Azure VM 的相同 Azure 区域

    -   可用性选项：不需要基础结构冗余

    -   映像:Windows Server 2019 Datacenter Gen2

    -   大小：Standard_D2s_v3

    -   用户名：Student

    -   密码：Pa55w.rd1234

    -   公共入站端口：“允许选定的端口”

    -   选择入站端口：RDP (3389)

    -   是否已拥有 Windows 许可？：**否**

    -   操作系统磁盘类型：**标准 HDD**

    -   虚拟网络：adVNET

    -   子网：名为“dmzSubnet (10.0.255.0/24)”的新子网

    -   公共 IP：名为“az12003b-vm0-ip”的新 IP 地址

    -   NIC 网络安全组：**基本**

    -   公共入站端口：“允许选定的端口”

    -   选择入站端口：RDP (3389)

    -   加速网络：关闭

    -   将此虚拟机置于现有负载均衡解决方案之后：**否**

    -   启动诊断：关闭

    -   OS 来宾诊断：关闭

    -   系统分配的托管标识：关闭

    -   使用 AAD 凭据登录（预览）：关闭

    -   启用自动关闭：关闭

    -   启用备份：关闭

    -   扩展：无

    -   标签：无

1.  等待预配完成。 这可能需要几分钟。

> **Result**：完成本练习后，你已配置了支持高可用性 SAP NetWeaver 部署所需的 Azure 资源


## <a name="exercise-2-configure-operating-system-of-azure-vms-running-windows-to-support-a-highly-available-sap-netweaver-deployment"></a>练习 2：配置运行 Windows 的 Azure VM 的操作系统，以支持高度可用的 SAP NetWeaver 部署

持续时间：60 分钟

在本练习中，你将配置运行 Windows Server 的 Azure VM 的操作系统，以适应高可用性 SAP NetWeaver 部署。

### <a name="task-1-join-windows-server-2016-azure-vms-to-the-active-directory-domain"></a>任务 1：将 Windows Server 2016 Azure VM 加入 Active Directory 域。

   > **注意**：在开始此任务之前，请确保你在之前练习中启动的模板部署已完成。 

1.  在 Azure 门户中，导航到名为 **adVNET** 的虚拟网络边栏选项卡，该虚拟网络已在本实验室的第一个练习中自动预配。

1.  显示“adVNET - DNS 服务器”边栏选项卡，请注意，虚拟网络配置了专用 IP 地址，该地址作为 DNS 服务器分配给本实验第一个练习中部署的域控制器。

1.  在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在上一个任务中预配的资源所在的资源组的名称：

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1.  在 Cloud Shell 窗格中运行以下命令，将你在上一个练习的第三个任务中部署的 Windows Server Azure VM 加入到 adatum.com Active Directory 域：

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "Pa55w.rd1234"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### <a name="task-2-examine-the-storage-configuration-of-the-database-tier-azure-vms"></a>任务 2：检查 Azure VM 的数据库层的存储配置。

1.  在实验室计算机的 Azure 门户中，导航到“az12003b-vm0”边栏选项卡。

1.  在“az12003b-vm0”边栏选项卡中，使用远程桌面连接到 Azure VM az12003b-vm0。 出现提示时，请提供以下凭据：

    -   登录身份： **学生**

    -   密码：Pa55w.rd1234

1.  从与 az12003b-vm0 的 RDP 会话中，使用远程桌面连接“i20-db-0.adatum.com”Azure VM。 出现提示时，请提供以下凭据：

    -   登录为：ADATUM\\Student

    -   密码：Pa55w.rd1234

1.  在远程桌面中使用相同凭据连接 i20-db-1.adatum.com Azure VM。

1.  在 i20-db-0.adatum.com 的 RDP 会话中，使用服务器管理器中的文件和存储服务检查磁盘配置。 请注意，已通过卷装载配置了一个数据磁盘，以提供用于数据库和日志文件的存储。 

1.  在 i20-db-1.adatum.com 的 RDP 会话中，使用服务器管理器中的文件和存储服务检查磁盘配置。 请注意，已通过卷装载配置了一个数据磁盘，以提供用于数据库和日志文件的存储。 


### <a name="task-3-prepare-for-configuration-of-failover-clustering-on-azure-vms-running-windows-server-2016-to-support-a-highly-available-sap-netweaver-installation"></a>任务 3：准备在运行 Windows Server 2016 的 Azure VM 上配置故障转移群集，以支持高可用性 SAP NetWeaver 安装。

1.  在与 i20-db-0.adatum.com 的 RDP 会话中，通过在分别将成为 ASCS 和 SQL 服务器群集节点的一对 ASCS 和 DB 服务器上运行以下命令来启动 Windows PowerShell ISE 会话并安装故障转移群集和远程管理工具功能：

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注意**：这可能重启所有四个 Azure VM 的来宾操作系统。

1.  在实验室计算机的 Azure 门户上单击“+ 创建资源”。

1.  从 **新建** 边栏选项卡，使用以下设置新建 **存储帐户**：

    -   订阅：你的 Azure 订阅的名称

    -   资源组：将托管高可用性 SAP NetWeaver 部署的 Azure VM 部署到的资源组的名称

    -   存储帐户名称：任何由 3 到 24 个字母和数字组成的唯一名称

    -   位置：*在上一个任务中部署 Azure VM 的相同 Azure 区域*

    -   性能：“标准”

    -   复制：本地冗余存储 (LRS)

    -   连接方式：公共终结点（所有网络）

    -   需要安全传输才能进行 REST API 操作：**已启用**

    -   大文件共享：**已禁用**

    -   Blob、容器和文件共享软删除：**已禁用**

    -   分层命名空间：**已禁用**


### <a name="task-4-configure-failover-clustering-on-azure-vms-running-windows-server-2016-to-support-a-highly-available-database-tier-of-the-sap-netweaver-installation"></a>任务 4：在运行 Windows Server 2016 的 Azure VM 上配置故障转移群集，以支持 SAP NetWeaver 安装的高可用性数据库层。

1.  如果需要，在与 az12003b-vm0 的 RDP 会话中，使用远程桌面重新连接到 i20-db-0.adatum.com Azure VM。 出现提示时，请提供以下凭据：

    -   登录为：ADATUM\\Student

    -   密码：Pa55w.rd1234

1.  在与 i20-db-0.adatum.com 的 RDP 会话中，在服务器管理器中导航到“本地服务器”视图并关闭“IE 增强安全配置” 。

1.  在与 i20-db-0.adatum.com 的 RDP 会话中，从服务器管理器的 **工具** 菜单中，启动 **Active Directory 管理中心**。

1.  在 Active Directory 管理中心，在 adatum.com 域的根目录下创建一个名为“群集”的新组织单位。

1.  在 Active Directory 管理中心中，将 i20-db-0 和 i20-db-1 的计算机帐户从“计算机”容器移动到“群集”组织单位。 

1.  在与 i20-db-0 的 RDP 会话中，通过运行以下命令启动 Windows PowerShell ISE 会话并创建新群集：

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.15
    ```

1.  在 i20-db-0.adatum.com 的 RDP 会话中，切换到 **Active Directory 管理中心** 控制台。

1.  在 Active Directory 管理中心，导航到“群集”组织单位并显示其“属性”窗口 。 

1.  在 **群集** 组织单位 **属性** 窗口中，导航到 **扩展** 部分，显示 **安全** 选项卡。 

1.  在 **安全** 选项卡，单击 **高级** 按钮，打开 **群集高级安全设置** 窗口。 

1.  在“群集高级安全设置”窗口的“权限”选项卡，单击“添加”。  

1.  在 **群集权限条目** 窗口，单击 **选择主体**

1.  在 **选择用户、服务帐户或组** 对话框，单击 **对象类型**，启用 **计算机** 条目旁边的复选框，然后单击 **确认**。 

1.  回到 **选择用户、计算机、服务帐户或组** 对话框，在 **输入要选择的对象名称**，输入 **az12003b-db-cl0** 并单击 **确定**。

1.  在“群集权限条目”窗口中，确保“允许”显示在“类型”下拉列表中  。 接下来，在 **适用于** 下拉列表，选择 **此对象和所有后代对象**。 在 **权限** 列表中，选择 **创建计算机对象** 和 **删除计算机对象** 复选框，然后单击 **确认** 两次。

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令安装 Azure PowerShell 模块：

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令使用 Azure AD 凭据进行身份验证：

    ```
    Add-AzAccount
    ```

    > **注意**：出现提示时，使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1.  在 Windows PowerShell ISE 会话中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在上一个任务中预配的存储帐户所在的资源组的名称：

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1.  在 Windows PowerShell ISE 会话中运行以下命令，设置新群集的 Cloud Witness Quorum：

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1.  如果要在 i20-db-0.adatum.com 的 RDP 会话中验证生成的配置，需在服务管理器的 **工具** 菜单中，启动 **故障转移群集管理器**。

1.  在故障转移群集管理器控制台中，审查 az12003b-db-cl0 群集配置，包括其节点、证明和网络设置。  请注意，群集没有任何共享存储。


### <a name="task-6-configure-failover-clustering-on-azure-vms-running-windows-server-2016-to-support-a-highly-available-ascs-tier-of-the-sap-netweaver-installation"></a>任务 6：在运行 Windows Server 2016 的 Azure VM 上配置故障转移群集，以支持 SAP NetWeaver 安装的高可用性 ASCS 层。

> **注意**：确保在开始此任务之前，已成功完成在练习 1 的任务 4 中启动的 S2D 群集部署。

1.  从与 az12003b-vm0 的 RDP 会话中，使用远程桌面连接 **i20-ascs-0.adatum.com** Azure VM。 出现提示时，请提供以下凭据：

    -   登录为：ADATUM\\Student

    -   密码：Pa55w.rd1234

1.  在与 i20-ascs-0.adatum.com 的 RDP 会话中，在服务器管理器中，导航到“本地服务器”视图并暂时关闭“IE 增强安全配置” 。

1.  在 i20-ascs-0.adatum.com 的 RDP 会话中，在服务器管理器的 **工具** 菜单中，启动 **Active Directory 管理中心**。

1.  在 Active Directory 管理中心中，导航到“计算机”容器。 

1.  在 Active Directory 管理中心中，将 i20-ascs-0 和 i20-ascs-1 的计算机帐户从“计算机”容器移动到“群集”组织单位。 

1.  在 i20-ascs-0.adatum.com 的 RDP 会话中，启动 Windows PowerShell ISE 会话，并通过运行以下命令新建群集：

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.16
    ```

1.  在 i20-ascs-0.adatum.com 的 RDP 会话中，切换到 **Active Directory 管理中心** 控制台。

1.  在 Active Directory 管理中心，导航到“群集”组织单位并显示其“属性”窗口 。 

1.  在 **群集** 组织单位 **属性** 窗口中，导航到 **扩展** 部分，显示 **安全** 选项卡。 

1.  在 **安全** 选项卡，单击 **高级** 按钮，打开 **群集高级安全设置** 窗口。 

1.  在 **计算机高级安全设置** 窗口的 **权限** 选项卡，单击 **添加**。

1.  在 **群集权限条目** 窗口，单击 **选择主体**

1.  在 **选择用户、服务帐户或组** 对话框，单击 **对象类型**，启用 **计算机** 条目旁边的复选框，然后单击 **确认**。 

1.  回到 **选择用户、计算机、服务帐户或组** 对话框，在 **输入要选择的对象名称**，输入 **az12003b-ascs-cl0** 并单击 **确定**。

1.  在“群集权限条目”窗口中，确保“允许”显示在“类型”下拉列表中  。 接下来，在 **适用于** 下拉列表，选择 **此对象和所有后代对象**。 在 **权限** 列表中，选择 **创建计算机对象** 和 **删除计算机对象** 复选框，然后单击 **确认** 两次。

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令安装 Azure PowerShell 模块：

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令使用 Azure AD 凭据进行身份验证：

    ```
    Add-AzAccount
    ```

    > **注意**：出现提示时，使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1.  在 Windows PowerShell ISE 会话中运行以下命令，将变量 `$resourceGroupName` 的值设置为你之前在本练习中预配的存储帐户所在的资源组的名称：

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1.  在 Windows PowerShell ISE 会话中运行以下命令，设置群集的 Cloud Witness Quorum：

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1.  如果要在 i20-ascs-0.adatum.com 的 RDP 会话中验证生成的配置，需在服务器管理器的 **工具** 菜单中，启动 **故障转移群集管理器**。

1.  在 **故障转移群集管理器** 控制台，审查 **az12003b-ascs-cl0** 群集配置，包括其节点、证明和网络设置。 请注意，群集没有任何共享存储。


### <a name="task-7-set-permissions-on-the-globalhostsapmnt-share"></a>任务 7：设置\\\\\\ 共享的权限

在本任务中，你将在 \\\\GLOBALHOST\\sapmnt 共享上设置共享级别权限。

> **注意**：默认情况下，完全控制权限只授予 ADATUM\Student 帐户。 

1.  在与 i20-ascs-0.adatum.com 的远程桌面会话中，从 **Windows PowerShell ISE** 窗口运行以下命令：

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### <a name="task-8-configure-operating-system-prerequisites-for-installing-sap-netweaver-ascs-and-database-components"></a>任务 8：配置安装 SAP NetWeaver ASCS 和数据库组件的操作系统先决条件

1.  在与 i20-ascs-0.adatum.com 的远程桌面会话中，从 Windows PowerShell ISE 会话中运行以下命令以配置便于安装 SAP ASCS 组件和使用虚拟名所需的注册条目：

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **Result**：完成本练习后，你已配置运行 Windows 的 Azure VM 操作系统，以支持高可用性 SAP NetWeaver 部署


## <a name="exercise-3-remove-lab-resources"></a>练习 3：删除实验室资源

持续时间：10 分钟

在本练习中，你将删除本实验室中预配的所有资源。

#### <a name="task-1-open-cloud-shell"></a>任务 1：打开 Cloud Shell

1. 在门户顶部，单击 Cloud Shell 图标打开“Cloud Shell”窗格，然后选择“PowerShell”作为 Shell。

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `$resourceGroupName` 的值设置为你在本实验的第一个练习中预配的那对 Windows Server 2019 Datacenter Azure VM 所在的资源组的名称：

    ```
    $resourceGroupNamePrefix = 'az12003b-'
    ```

1. 在 Cloud Shell 窗格中运行以下命令，列出你在本实验室中创建的所有资源组：

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. 验证输出是否只包含在本实验室中创建的资源组。 这些组将在下一个任务中删除。

#### <a name="task-2-delete-resource-groups"></a>任务 2：删除资源组

1. 在 Cloud Shell 窗格中运行以下命令，删除你在本实验室中创建的资源组

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. 关闭 Cloud Shell 窗格。

> **Result**：完成本练习后，你已经删除了本实验中使用的资源。
