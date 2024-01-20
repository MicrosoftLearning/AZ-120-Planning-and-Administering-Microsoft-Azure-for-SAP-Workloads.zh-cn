---
lab:
  title: 02a - 在 Azure VM 上实现 Linux 群集
  module: Module 02 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 模块 2：了解 Azure 上的 SAP 的 IaaS 的基础
# 实验室 2a：在 Azure VM 上实现 Linux 群集

预计时间：90 分钟

本实验室中的所有任务都是从 Azure 门户执行的（包括 Bash Cloud Shell 会话）  

   > **注意**：如果没有使用 Cloud Shell，实验室虚拟机必须安装 Azure CLI [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows)，并包含 SSH 客户端（例如 PuTTY，可从 [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 获得）。

实验室文件：无

## 方案
  
为了准备在 Azure 上部署 SAP HANA，Adatum Corporation 需要了解在运行 Linux 的 SUSE 发行版的 Azure VM 上实现群集的过程。

## 目标
  
完成本实验室后，你将能够：

- 预配支持高可用性 SAP HANA 部署所需的 Azure 计算资源

- 配置运行 Linux 的 Azure VM 操作系统，以支持高可用性 SAP HANA 安装

- 预配支持高可用性 SAP HANA 部署所需的 Azure 网络资源

## 要求

- 具有足够数量的可用 DSv3 vCPU (2 x 4) 和 DSv2 (1 x 1) vCPU 的 Microsoft Azure 订阅

- 安装了兼容 Azure Cloud Shell 的 Web 浏览器且具有 Azure 访问权限的实验室计算机

## 练习 1：预配必要的 Azure 计算资源以支持高度可用的 SAP HANA 部署

持续时间：30 分钟

在本练习中，你将部署配置 Linux 群集所必需的 Azure 基础结构计算组件。 这将涉及在同一可用性集中创建一对运行 Linux SUSE 的 Azure VM 并预配 Azure Bastion。

### 任务 1：部署运行 Linux SUSE 的 Azure VM

1. 从实验室计算机中启动 Web 浏览器，并导航到 Azure 门户 (https://portal.azure.com )。

1. 如果出现提示，则使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1. 在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“邻近放置组”边栏选项卡，然后在“邻近放置组”边栏选项卡上选择“+ 创建”****************。

1. 在“创建邻近放置组”边栏选项卡的“基本信息”选项卡中，指定以下设置并选择“查看 + 创建”************：

    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称  |
    | “资源组”部分 | 选择“**新建**”，输入 **az12001a-RG**，然后选择“**确定**” |
    | **区域** | *具有足够 vCPU 配额的 Azure 区域* |
    | **邻近放置组名称** | 选择 **az12001a-ppg** |
    | **意向详细信息** | 标准 D4s v3**** |

   > **注意**：请考虑使用“美国东部”**** 或“美国东部 2”**** 区域来部署资源。

1. 在“创建邻近放置组”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”************。

   > **注意**：等待预配完成。 这应该可以在一分钟内完成。

1. 在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“虚拟机”边栏选项卡，然后在“虚拟机”边栏选项卡上选择“+ 创建”，并在下拉菜单中选择“Azure 虚拟机”********************。

1. 在“创建虚拟机”边栏选项卡的“基本信息”选项卡中，指定以下设置并选择“下一步:************ 磁盘 >”（保留所有其他设置的默认值）：

    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称  |
    | 资源组 | *选择先前在此任务中使用的资源组名称* |
    | **虚拟机名称** | *选择* **az12001a-vm0** |
    | **区域** | *在创建邻近放置组时选择的**相同 Azure 区域*** |
    | **可用性选项** | *选择*“**可用性集**” |
    | **可用性集** | *名为* **az12001a-avset** *的新可用性集，具有 2 个容错域和 5 个更新域* |
    | 安全类型**** | *选择*“**标准**” |
    | **图像** | *选择***SUSE Enterprise Linux for SAP 15 SP3 - BYOS - x64 Gen 2** |
    | **使用 Azure Spot 折扣运行** | **是** |
    | **大小** | 标准 D4s v3**** |
    | **身份验证类型** | **密码** |
    | **用户名** | student**** |
    | **密码** | 自行设定的任何复杂密码 |
   
    > **备注**：请确保记住在部署期间指定的密码。 本实验室中稍后会用到它。

    > **注意**：要查找映像，请单击“**查看所有映像**”链接，在“**选择映像**”边栏选项卡上的搜索文本框中，键入“**SUSE Enterprise Linux**”，然后在结果列表中单击“**SUSE Enterprise Linux for SAP 15 SP3 - BYOS**”并选择“**Generation 2**”。

1. 在“创建虚拟机”边栏选项卡的“磁盘”选项卡中，指定以下设置并选择“下一步:************ 网络 >”（保留所有其他设置的默认值）：

    | 设置 | Value |
    |   --    |  --   |
    | **OS 磁盘类型** | **高级 SSD（本地冗余存储）**  |
    | **密钥管理** | **平台管理的密钥** |

1. 在“创建虚拟机”边栏选项卡的“网络”选项卡中，指定以下设置并选择“下一步:************ 管理 >”（保留所有其他设置的默认值）：

    | 设置 | 值 |
    |   --    |  --   |
    | **虚拟网络** | *选择*“**新建**”*并创建名为* **az12001a-RG-vnet** 的新虚拟网络，继续“创建虚拟网络”中的后续步骤。  |
    | **地址空间** | *将新虚拟网络的地址空间设置为* **192.168.0.0/20** |
    | **子网名称** | **subnet-0** |
    | **子网地址范围** | **192.168.0.0/24**，选择“确定”以继续“创建虚拟机”。|
    | **公共 IP 地址** | **无** |
    | **NIC 网络安全组** | **高级**  |
    | **启用加速网络** | **开** |
    | 负载平衡选项 | **无** |
    
    > **注意**：该映像已预先配置了 NSG 规则。

1. 在“**创建虚拟机**”边栏选项卡的“**管理**”选项卡中，指定以下设置并选择“**下一步：监视 >** ”（保留所有其他设置的默认值）：
   
   | 设置 | “值” |
   |   --    |  --   |
   | **启用系统分配的托管标识** | 关闭 |
   | 启用自动关闭 | 关闭 |
   | 免费启用基本计划**** | **否**  |

   > **注意**：如果已在订阅中启用了 Microsoft Defender for Cloud，则**免费基本计划**设置不可用。

1. 在“**创建虚拟机**”边栏选项卡的“**监视**”选项卡中，选择“**下一步：高级 >** ”（保留所有其他设置的默认值）

1. 在“创建虚拟机”**** 边栏选项卡的“高级”**** 选项卡中，指定以下设置并选择“查看 + 创建”****（保留其他所有设置的默认值）：

   | 设置 | “值” |
   |   --    |  --   |
   | **邻近放置组** | **az12001a-ppg** |

1. 在“创建虚拟机”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”************。

   > **注意**：等待预配完成。 此操作应需不到 3 分钟的时间。

1. 在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“虚拟机”边栏选项卡，然后在“虚拟机”边栏选项卡上选择“+ 创建”，并在下拉菜单中选择“Azure 虚拟机”********************。

1. 在“创建虚拟机”边栏选项卡的“基本信息”选项卡中，指定以下设置并选择“下一步:************ 磁盘 >”（保留所有其他设置的默认值）：
   
    | 设置 | 值 |
    |   --    |  --   |
    | **订阅** | Azure 订阅的名称  |
    | 资源组 | *选择先前在此任务中使用的资源组名称* |
    | **虚拟机名称** | *选择* **az12001a-vm1** |
    | **区域** | *在创建邻近放置组时选择的相同 Azure 区域* |
    | **可用性选项** | *选择*“**可用性集**” |
    | **可用性集** | **az12001a-avset** |
    | 安全类型**** | *选择*“**标准**” |
    | **图像** | *选择* * **SUSE Enterprise Linux for SAP 15 SP3 - BYOS - x64 Gen 2** |
    | **使用 Azure Spot 折扣运行** | **是** |
    | **大小** | 标准 D4s v3**** |
    | **身份验证类型** | **密码** |
    | **用户名** | student**** |
    | **密码** | 在第一次部署期间指定的同一个密码 |
   
   > **注意**：要查找映像，请单击“**查看所有映像**”链接，在“**选择映像**”边栏选项卡上的搜索文本框中，键入“**SUSE Enterprise Linux**”，然后在结果列表中单击“**SUSE Enterprise Linux for SAP 15 SP3 - BYOS**”并选择“**Generation 2**”。

1. 在“创建虚拟机”边栏选项卡的“磁盘”选项卡中，指定以下设置并选择“下一步:************ 网络 >”（保留所有其他设置的默认值）：

    | 设置 | Value |
    |   --    |  --   |
    | **OS 磁盘类型** | **高级·SSD**  |
    | **密钥管理** | **平台管理的密钥** |

1. 在“创建虚拟机”边栏选项卡的“网络”选项卡中，指定以下设置并选择“下一步:************ 管理 >”（保留所有其他设置的默认值）：
    
    | 设置 | 值 |
    |   --    |  --   |
    | **虚拟网络** | **az12001a-RG-vnet** |
    | **子网** | **subnet-0 (192.168.0.0/24)** |
    | **公共 IP 地址** | **无** |
    | **NIC 网络安全组** | **高级**  |
    | **启用加速网络** | **开** |
    | 负载平衡选项 | **无** |

   > **注意**：该映像已预先配置了 NSG 规则。

1. 在“**创建虚拟机**”边栏选项卡的“**管理**”选项卡中，指定以下设置并选择“**下一步：监视 >** ”（保留所有其他设置的默认值）：
    
   | 设置 | “值” |
   |   --    |  --   |
   | **启用系统分配的托管标识** | 关闭 |
   | 启用自动关闭 | 关闭 |
   | 免费启用基本计划**** | **否**  |

   > **注意**：如果已经选择了 Azure 安全中心计划，则**免费基本计划**设置不可用。

1.  在“**创建虚拟机**”边栏选项卡的“**监视**”选项卡中，选择“**下一步：高级 >** ”（保留所有其他设置的默认值）

1.  在“创建虚拟机”**** 边栏选项卡的“高级”**** 选项卡中，指定以下设置并选择“查看 + 创建”****（保留其他所有设置的默认值）：
    
   | 设置 | “值” |
   |   --    |  --   |
   | **邻近放置组** | **az12001a-ppg** |

1.  在“创建虚拟机”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”************。

   > **注意**：等待预配完成。 此操作应需不到 3 分钟的时间。


### 任务 2：创建并配置 Azure VM 磁盘

1. 从 Azure 门户，在 Cloud Shell 启动 Bash 会话。 

   > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。 如果是，接受默认设置，这样会在自动生成的资源组中创建存储账户。

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `RESOURCE_GROUP_NAME` 的值设置为你在上一个任务中预配的资源所在的资源组的名称：

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. 在 Cloud Shell 窗格中运行以下命令，创建第一组共计 8 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第一个 Azure VM：

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. 在 Cloud Shell 窗格中运行以下命令，创建第二组共计 8 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第二个 Azure VM：

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. 在 Azure 门户中，导航到你在前一个任务 (az12001a-vm0) 中预配的第一个 Azure VM 边栏选项卡。****

1. 从“az12001a-vm0”边栏选项卡中，导航到“az12001a-vm0 \| 磁盘”边栏选项卡。********

1. 在“az12001a-vm0 \| 磁盘”边栏选项卡中，选择“附加现有磁盘”并将具有以下设置的数据磁盘附加到 az12001a-vm0：********
    
   | 设置 | 值 |
   |   --    |  --   |
   | **LUN** | **0** |
   | **磁盘名称** | **az12001a-vm0-DataDisk0** |
   | 资源组 | *选择先前在此任务中使用的资源组名称* |
   | **主机缓存** | **只读** |

2. 重复上一步，使用前缀 az12001a-vm0-DataDisk**** 附加剩余的 7 个磁盘（总共 8 个）。 分配与磁盘名称的最后一个字符匹配的 LUN 编号。 使用 LUN 1**** 将磁盘的主机缓存设置为“只读”****，而对于其他部分，将主机缓存设置为“无”****。

3. 保存所做更改。 

4. 在 Azure 门户中，导航到你在前一个任务 (az12001a-vm1) 中预配的第二个 Azure VM 边栏选项卡。****

5. 从“az12001a-vm1”边栏选项卡中，导航到“az12001a-vm1 \| 磁盘”边栏选项卡。********

6. 在“az12001a-vm1 \| 磁盘”边栏选项卡中，使用以下设置将数据磁盘附加到 az12001a-vm1：****
    
   | 设置 | 值 |
   |   --    |  --   |
   | **LUN** | **0** |
   | **磁盘名称** | **az12001a-vm1-DataDisk0** |
   | 资源组 | *选择先前在此任务中使用的资源组名称* |
   | **主机缓存** | **只读** |

7. 重复上一步，使用前缀“az12001a-vm1-DataDisk”附加剩余的 7 个磁盘（总共 8 个）。**** 分配与磁盘名称的最后一个字符匹配的 LUN 编号。 使用 LUN 1**** 将磁盘的主机缓存设置为“只读”****，而对于其他部分，将主机缓存设置为“无”****。

8. 保存所做的更改。 

#### 任务 3：预配 Azure Bastion 

> **注意**：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

> **注意**：请确保浏览器已启用弹出窗口功能。

1. 在显示 Azure 门户的浏览器窗口中，打开另一个选项卡，并在浏览器选项卡中导航到 [Azure 门户****](https://portal.azure.com)。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的虚拟网络 az12001a-RG-vnet 中********：

   ```powershell
   $resourceGroupName = 'az12001a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az12001a-RG-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 192.168.15.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中，搜索并选择“Bastion”，然后从“Bastion”边栏选项卡中选择“+ 创建”  。
1. 在“创建 Bastion”边栏选项卡的“基本”选项卡上，指定以下设置并选择“查看 + 创建”  ：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|**az12001a-RG**|
   |名称|**az12001a-bastion**|
   |区域|在本练习的先前任务中部署资源的同一 Azure 区域|
   |层|**基本**|
   |虚拟网络|**az12001a-RG-vnet**|
   |子网|**AzureBastionSubnet (192.168.15.0/24)**|
   |公共 IP 地址|**新建**|
   |公共 IP 名称|**az12001a-RG-vnet-ip**|

1. 在“创建 Bastion”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”  ：

   > **备注**：请等待部署完成再继续此练习的下一个任务。 部署可能需要大约 5 分钟时间。


> **Result**：完成此练习后，你已预配了支持高可用性 SAP HANA 部署所需的 Azure 计算资源。


## 练习 2：配置运行 Linux 的 Azure VM 的操作系统以支持高度可用的 SAP HANA 安装

持续时间：30 分钟

在本练习中，你将在运行 SUSE Linux Enterprise Server 的 Azure VM 上配置操作系统和存储，以适应 SAP HANA 的群集安装。

### 任务 1：连接到 Azure Linux VM

1. 在实验室计算机上，在 Azure 门户中搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az12001a-vm0”条目。************ 这将打开“az12001a-vm0”边栏选项卡。****

1. 在“az12001a-vm0”边栏选项卡中，选择“连接”，在下拉菜单中选择“通过 Bastion 连接”，在“az12001a-vm0”的“Bastion”选项卡上，将“身份验证类型”留为“VM 密码”，提供你部署“az12001a-vm0”虚拟机时设置的凭据，保留启用的复选框“在新浏览器选项卡中打开”，然后选择“连接”：

1. 重复上述两个步骤，通过 Bastion 连接到 **az12001a-vm1** Azure VM。

### 任务 2：配置运行 Linux 的 Azure VM 存储

1. 在与 **az12001a-vm0** Azure VM 的 Bastion 会话中，运行以下命令以提升特权： 

   ```cli
   sudo su -
   ```

1. 运行以下命令来标识新附加设备与其 LUN 编号之间的映射：
   
   ```cli
   lsscsi
   ```

1. 通过运行以下命令为 6 个（共 8 个）数据磁盘创建物理卷：
   
   ```cli
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. 通过运行以下命令创建卷组：
   
   ```cli
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. 通过运行以下命令创建逻辑卷：

   ```cli
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **注意**：我们为每个卷组创建一个逻辑卷

1. 通过运行以下命令设置逻辑卷的格式：

   ```cli
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **注意**：从 SUSE Linux Enterprise Server 12 开始，可以选择使用 XFS 文件系统的新磁盘上的格式 (v5)，该格式提供 XFS 元数据的自动校验和、文件类型支持，并增加了对每个文件的访问控制列表数量的限制。 使用 YaST 创建 XFS 文件系统时，新格式会自动应用。 出于兼容性原因，若要以较旧的格式创建 XFS 文件系统，请使用不含 `-m crc=1` 选项的 mkfs.xfs 命令。 

1. 通过运行以下命令对 **/dev/sdi** 磁盘进行分区：

   ```cli
   fdisk /dev/sdi
   ```

1. 出现提示时，请按顺序键入 `n`、`p`、`1`（每次键入后按 Enter 键）按 Enter 键两次，然后键入 `w` 以完成写入。********

1. 通过运行以下命令对 **/dev/sdj** 磁盘进行分区：

   ```cli
   fdisk /dev/sdj
   ```

1. 出现提示时，请按顺序键入 `n`、`p`、`1`（每次键入后按 Enter 键）按 Enter 键两次，然后键入 `w` 以完成写入。********

1. 通过运行以下命令设置新创建的分区的格式（在系统提示确认时键入 `y` 并按 **Enter** 键）：

   ```cli
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. 通过运行以下命令创建将用作装入点的目录：

   ```cli
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. 通过运行以下命令显示逻辑卷的 ID：

   ```cli
   blkid
   ```

   > **注意**：确定与新创建的卷组和分区关联的 UUID 值，包括 /dev/sdi（用于 /hana/shared）和 /dev/sdj（用于 /usr/sap）。********************


1. 通过运行以下命令在 vi 编辑器中打开 **/etc/fstab**（也可以使用任何其他编辑器）：

   ```cli
   vi /etc/fstab
   ```

1. 在编辑器中，将以下条目添加到 /etc/fstab（其中 `\<UUID of /dev/vg\_hana\_data-hana\_data\>`、`\<UUID of /dev/vg\_hana\_log-hana\_log\>`、`\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`、`\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>` 和 `\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>` 表示你在上一步中标识的 ID）：****

   ```cli
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. 保存更改并关闭编辑器。

1. 通过运行以下命令装载新卷：

   ```cli
   mount -a
   ```

1. 通过运行以下命令验证装载是否成功：

   ```cli
   df -h
   ```

1. 切换到与 az12001a-vm1 的 Bastion 会话，然后重复此任务中的所有步骤以在 **az12001a-vm1** 上配置存储。


### 任务 3：启用跨节点无密码 SSH 访问

1. 在与 **az12001a-vm0** Azure VM 的 Bastion 会话中，通过运行以下命令生成无通行短语 SSH 密钥：

   ```cli
   ssh-keygen -tdsa
   ```

1. 出现提示时，按 Enter**** 三次，然后运行以下命令以显示公钥： 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. 将密钥值复制到剪贴板中。

1. 切换到与 **az12001a-vm1** Azure VM 的 Bastion 会话，然后运行以下代码在 vi 编辑器中创建文件 **/root/.ssh/authorized\_keys**（也可以使用任何其他编辑器）：

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. 在编辑器窗口中，粘贴你在 az12001a-vm0 上生成的密钥。

1. 保存更改并关闭编辑器。

1. 在与 **az12001a-vm1** Azure VM 的 Bastion 会话中，通过运行以下命令生成无通行短语 SSH 密钥：

   ```cli
   ssh-keygen -tdsa
   ```

1. 出现提示时，按 Enter**** 三次，然后运行以下命令以显示公钥： 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. 将密钥值复制到剪贴板中。

1. 切换到与 **az12001a-vm0** Azure VM 的 Bastion 会话，然后运行以下代码在 vi 编辑器中创建文件 **/root/.ssh/authorized\_keys**（也可以使用任何其他编辑器）：

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. 在编辑器窗口中，粘贴你在 az12001a-vm1 上生成的密钥。

1. 保存更改并关闭编辑器。

1. 在与 **az12001a-vm0** Azure VM 的 Bastion 会话中，通过运行以下命令生成无通行短语 SSH 密钥：

   ```cli
   ssh-keygen -t rsa
   ```

1. 出现提示时，按 Enter**** 三次，然后运行以下命令以显示公钥： 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. 将密钥值复制到剪贴板中。

1. 切换到与 **az12001a-vm1** Azure VM 的 Bastion 会话，然后运行以下代码在 vi 编辑器中打开文件 **/root/.ssh/authorized\_keys**（也可以使用任何其他编辑器）：

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. 在编辑器窗口中，从新的一行开始，粘贴你在 az12001a-vm0 上生成的密钥。

1. 保存更改并关闭编辑器。

1. 在与 **az12001a-vm1** Azure VM 的 Bastion 会话中，通过运行以下命令生成无通行短语 SSH 密钥：

   ```cli
   ssh-keygen -t rsa
   ```

1. 出现提示时，按 Enter**** 三次，然后运行以下命令以显示公钥： 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. 将密钥值复制到剪贴板中。

1. 切换到与 **az12001a-vm0** Azure VM 的 Bastion 会话，然后运行以下代码在 vi 编辑器中打开文件 **/root/.ssh/authorized\_keys**（也可以使用任何其他编辑器）：

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. 在编辑器窗口中，从新的一行开始，粘贴你在 az12001a-vm1 上生成的密钥。

1. 保存更改并关闭编辑器。

1. 在与 **az12001a-vm0** Azure VM 的 Bastion 会话中，运行以下命令在 vi 编辑器中打开文件 **/etc/ssh/sshd\_config**（也可以使用任何其他编辑器）：

   ```cli
   vi /etc/ssh/sshd_config
   ```

1. 在 /etc/ssh/sshd\_config 文件中，找到 PermitRootLogin 和 AuthorizedKeysFile 条目，并按如下所示对其进行配置（如果需要，请删除前导 # 字符：****************

   ```cli
   PermitRootLogin yes
   AuthorizedKeysFile  /root/.ssh/authorized_keys
   ```

1. 保存更改并关闭编辑器。

1. 在与 **az12001a-vm0** Azure VM 的 Bastion 会话中，运行以下命令重启 sshd 守护程序：

   ```cli
   systemctl restart sshd
   ```

1. 在 az12001a-vm1 上重复前面的四个步骤。

1. 要验证配置是否成功，请在与 **az12001a-vm0** Azure VM 的 Bastion 会话中，通过运行以下命令建立从 az12001a-vm0 到 az12001a-vm1 的 SSH 会话，作为**根**： 

   ```cli
   ssh root@az12001a-vm1
   ```

1. 当系统提示你是否确定要继续连接时，请键入 `yes` 并按 Enter**** 键。 

1. 确保未提示你输入密码。

1. 通过运行以下命令，关闭从 az12001a-vm0 到 az12001a-vm1 的 SSH 会话： 

   ```cli
   exit
   ```

1. 通过运行以下命令两次，从 az12001a-vm0 注销：

   ```cli
   exit
   ```

1. 要验证配置是否成功，请在与 **az12001a-vm1** 的 Bastion 会话中，通过运行以下命令建立从 az12001a-vm1 到 az12001a-vm0 的 SSH 会话，作为**根**： 

   ```cli
   ssh root@az12001a-vm0
   ```

1. 当系统提示你是否确定要继续连接时，请键入 `yes` 并按 Enter**** 键。 

1. 确保未提示你输入密码。

1. 通过运行以下命令，关闭从 az12001a-vm1 到 az12001a-vm0 的 SSH 会话： 

   ```cli
   exit
   ```

1. 通过运行以下命令两次，从 az12001a-vm1 注销：

   ```cli
   exit
   ```

> **Result**：完成本练习后，你已配置运行 Linux 的 Azure VM 操作系统，以支持高可用性 SAP HANA 安装


## 练习 3：预配必要的 Azure 网络资源以支持高度可用的 SAP HANA 部署

持续时间：30 分钟

在本练习中，你将实施 Azure 负载均衡器以适应 SAP HANA 的群集安装。


### 任务 1：配置 Azure VM 以辅助负载均衡设置。

1. 在 Azure 门户中，导航到 az12001a-vm0 Azure VM 的边栏选项卡****。

1. 从“az12001a-vm0”边栏选项卡中，导航到“az12001a-vm0 \| 网络”边栏选项卡。******** 

1. 从“az12001a-vm0 \| 网络”边栏选项卡中，选择表示 az12001a-vm0 的网络接口的条目。**** 

1. 在 az12001a-vm0 的网络接口边栏选项卡中，导航到其 IP 配置边栏选项卡，并在该处显示其“ipconfig1”边栏选项卡****。

1. 在“ipconfig1”边栏选项卡中，将专用 IP 地址分配设置为“静态”，然后保存所做的更改。********

1. 在 Azure 门户中，导航到 az12001a-vm1 Azure VM 的边栏选项卡。****

1. 从“az12001a-vm1”边栏选项卡中，导航到“az12001a-vm1 \| 网络”边栏选项卡。******** 

1. 从“az12001a-vm1 \| 网络”边栏选项卡，导航到 az12001a-vm1 的网络接口。**** 

1. 在 az12001a-vm1 网络接口的边栏选项卡中，导航到其 IP 配置边栏选项卡，并在这里显示其“ipconfig1”边栏选项卡。****

1. 在“ipconfig1”边栏选项卡中，将专用 IP 地址分配设置为“静态”，然后保存所做的更改。********


### 任务 2：创建和配置处理入站流量的 Azure 负载均衡器

1. 在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“负载均衡器”边栏选项卡，然后在“负载均衡器”选项卡上，选择“创建”****************。

1. 在“创建负载均衡器”边栏选项卡的“基本信息”选项卡中，请指定以下设置并选择“查看 + 创建”（保留其他设置的默认值）************：
    
   | 设置 | 值 |
   |   --    |  --   |
   | **订阅** | Azure 订阅的名称 |
   | 资源组 | *先前在本实验室中使用的资源组名称* |
   | **Name** | **az12001a-lb0** |
   | **区域** | 在本实验室的第一项练习中部署 Azure VM 时所在的 Azure 区域 |
   | **SKU** | **标准** |
   | **类型** | **内部** |

1. 单击“下一步:**** 前端 IP 配置”。 在“前端 IP 配置”屏幕上，单击“添加前端 IP 配置”，然后单击“添加”************。
    
   | 设置 | 值 |
   |   --    |  --   |
   | **Name** | **frontend1** |
   | **虚拟网络** | **az12001a-RG-vnet** |
   | **子网** | **subnet-0** |
   | IP 地址分配 | **静态** |
   | IP 地址 | 192.168.0.240 |
   | **可用性区域** | **区域冗余** |

1. 选择“查看 + 创建”，然后选择“创建”。

   > **注意**：等待负载均衡器配置完成。 这应该可以在一分钟内完成。 

1. 在 Azure 门户中，导航到显示新配置的 az12001a-lb0**** 负载均衡器属性的边栏选项卡。 

1. 在“az12001a-lb0”**** 边栏选项卡中，选择“后端池”****，再选择“+ 添加”****，然后在“添加后端池”**** 中指定以下设置（将其他设置保留为默认值）：
    
   | 设置 | 值 |
   |   --    |  --   |
   | **Name** | **az12001a-lb0-bepool** |
   | **虚拟网络** | **az12001a-RG-vnet** |
   | **后端池配置** | IP 地址 |
   | IP 地址 | **192.168.0.4** 资源名称：**az12001a-vm0** |
   | IP 地址 | **192.168.0.5** 资源名称：**az12001a-vm1** |

1. 在“az12001a-lb0”**** 边栏选项卡中，选择“运行状况探测”****，再选择“+ 添加”****，然后在“添加运行状况探测”**** 边栏选项卡中，指定以下设置（将其他设置保留为默认设置）：
    
   | 设置 | 值 |
   |   --    |  --   |
   | **Name** | **az12001a-lb0-hprobe** |
   | **协议** | **TCP** |
   | 端口 | 62500**** |
   | **时间间隔** | 5 秒 |
   | **不正常阈值** | 2 次连续失败 |

1. 在“az12001a-lb0”**** 边栏选项卡中，选择“负载均衡规则”****， 再选择“+ 添加”****，然后在“添加负载均衡规则”**** 边栏选项卡中，指定以下设置（将其他设置保留为默认设置）：
    
   | 设置 | 值 |
   |   --    |  --   |
   | **Name** | **az12001a-lb0-lbruleAll** |
   | **IP 版本** | **IPv4** |
   | “前端 IP 地址” | 192.168.0.240 (LoadBalancerFrontEnd) |
   | HA 端口 | **已启用** |
   | 后端池 | **az12001a-cl-lb0-bepool（2 个虚拟机）** |
   | 运行状况探测 | **az12001a-lb0-hprobe (TCP:62500)** |
   | **会话暂留** | **无** |
   | **空闲超时(分钟)** | **4** |
   | **TCP 重置** | **已禁用** |
   | 浮动 IP (直接服务器返回) | **已启用** |

### 任务 3：创建和配置处理出站流量的 Azure 负载均衡器

1. 从 Azure 门户，在 Cloud Shell 启动 Bash 会话。 

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `RESOURCE_GROUP_NAME` 的值设置为你在本实验室的第一个练习中预配的资源所在的资源组的名称：

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. 在 Cloud Shell 窗格中运行以下命令，创建第二个负载均衡器将使用的公共 IP 地址：

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. 在 Cloud Shell 窗格中，运行以下命令以创建第二个负载均衡器：

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. 在 Cloud Shell 窗格中，运行以下命令以创建第二个负载均衡器的出站规则：

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. 关闭 Cloud Shell 窗格。

1. 在 Azure 门户中，导航到显示新创建的 Azure 负载均衡器 az12001a-lb1 的属性的边栏选项卡****。

1. 在 az12001a-lb1**** 边栏选项卡中，单击**后端池**。

1. 在“az12001a-lb1 \| 后端池”边栏选项卡中，单击“az12001a-lb1-bepool”。********

1. 在 az12001a-lb1-bepool**** 边栏选项卡中，指定以下设置并单击“保存”****：
   
   | 设置 | 值 |
   |   --    |  --   |
   | **虚拟网络** | **az12001a-rg-vnet (2 个 VM)** |
   | **虚拟机** | **az12001a-vm0** IP 配置：**ipconfig1 (192.168.0.4)** |
   | **虚拟机** | **az12001a-vm1** IP 配置：**ipconfig1 (192.168.0.5)** |

> **Result**：完成此练习后，你已预配了支持高可用性 SAP HANA 部署所需的 Azure 网络资源


## 练习 4：删除实验室资源

持续时间：10 分钟

在本练习中，你将删除本实验室中预配的所有资源。

#### 任务 1：列出要删除的资源组

1. 在门户顶部，单击 Cloud Shell 图标以打开 Cloud Shell 窗格，然后选择“Bash”作为 Shell****。

1. 在 Cloud Shell 窗格中运行以下命令，将变量 `RESOURCE_GROUP_PREFIX` 的值设置为你在本实验室中预配的资源所在的资源组的名称前缀：

   ```cli
   RESOURCE_GROUP_PREFIX='az12001a-'
   ```

1. 在 Cloud Shell 窗格中运行以下命令，列出你在本实验室中创建的所有资源组：

   ```cli
   az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
   ```

1. 验证输出结果是否仅包含在本实验室中创建的资源组。 在下一个任务中，将删除该资源组及其所有资源。

#### 任务 2：删除资源组

1. 在 Cloud Shell 窗格中运行以下命令，删除该资源组及其资源。

   ```cli
   az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 关闭 Cloud Shell 窗格。

> **Result**：完成本练习后，你已经删除了本实验中使用的资源。
