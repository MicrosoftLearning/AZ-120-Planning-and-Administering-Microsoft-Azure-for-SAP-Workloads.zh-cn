---
lab:
  title: 00 - 实验先决条件
  module: Module 00 - Lab prerequisites
---

# AZ 120:实验先决条件

## vCPU 核心要求

> **重要提示**：vCPU 核心要求取决于要实现的实验室。

- 要完成实验室 04b - 在运行 Windows 的 Azure VM 上实现 SAP 体系结构，你需要有订阅 Microsoft Azure，而且在支持可用性区域（本实验室中部署的 Azure VM 将会驻留在这里）的 Azure 区域中有至少 28 个 vCPU 可用。

    - 4 x Standard_DS1_v2（每个 1 个 vCPU）= 4
    - 6 x Standard_D4s_v3（每个 4 个 vCPU）= 24

    > **注意**：请考虑使用“美国东部”**** 或“美国东部 2”**** 区域来部署资源。

    > **注意**：要确定支持可用性区域的 Azure 区域，请参阅 <https://docs.microsoft.com/en-us/azure/availability-zones/az-overview>

- 要完成实验室 05 - 使用 Azure SAP 解决方案中心自动部署，你需要有 Microsoft Azure 订阅，而且在支持可用性区域（本实验室中部署的 Azure VM 将会驻留在这里）的 Azure 区域中有以下数量的 vCPU 可用。

    - 4 x Standard_E4ds_v4（每个 4 个 vCPU）或 4 X Standard_D4ds_v4（每个 4 个 vCPU）= 8
    - 2 x Standard_M64ms（每个 64 个 vCPU）= 128

>**备注**：要尽量减少 vCPU 和内存要求，可以使用 Standard_M32ts（每个 32 个 vCPU 和 192 GiB 内存），而不是 Standard_M64m。

>**备注**：虽然本课程的前三个实验室的 vCPU 要求较低，但我们建议你请求增加配额以满足所有实验室的要求，因为增加配额可能需要一些时间（即使通常在工作日当天就能应要求完成增加配额的工作）。

## 动手实验室准备工作

时间范围：30 分钟

### 任务 1：验证实验室是否有足够数量的 vCPU 核心，以在运行 Windows 的 Azure VM 上实现 SAP 体系结构

1. 从实验室计算机中启动 Web 浏览器，并导航到 Azure 门户 ( )。
1. 在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

    > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。 如果是，接受默认设置，这样会在自动生成的资源组中创建存储账户。

1. 在 Azure 门户中，在 Cloud Shell**** 窗格中的 PowerShell 提示符处运行以下命令：其中 `<Azure_region>` 指定了你打算用于本实验室的目标 Azure 区域（例如 `eastus`）：

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **注意**：若要确定 Azure 区域的名称，请在 Cloud Shell 的 Bash 提示符下运行 `(Get-AzLocation).Location`。****
   
1. 查看上一步执行命令输出中的当前值和限制条目，并确保目标 Azure 区域中有足够数量的 vCPU。
1. 如果 vCPU 数量不足，请在 Azure 门户中导航回“订阅”边栏选项卡，然后单击“使用情况 + 配额”****。 
1. 在“订阅”的“使用情况 + 配额”边栏选项卡上，单击“请求增加”********。
1. 在“问题描述”边栏选项卡上，指定以下内容****：

    -   问题类型：“服务和订阅限制(配额)”****
    -   订阅：在本次实验室中你将使用的 Azure 订阅名
    -   配额类型：计算/VM（核心数/vCPU 数）订阅限制提高****

1. 单击“管理配额”****。
1. 使用位置下拉菜单将结果筛选到计划使用的 Azure 区域。 建议使用“美国东部”或“美国东部 2”********。
1. 找到“标准 DSv3 系列 vCPU”配额类型，然后选择编辑铅笔****。
1. 在“新限制”字段中，指定 40，然后单击“保存并继续”********。
1. 找到“区域 vCPU 总数”配额类型，然后选择编辑铅笔****。
1. 在“新限制”字段中，指定 40，然后单击“保存并继续”********。

   > **注意**：应自动批准配额增加请求。 如果请求被拒绝，请继续创建支持工单以请求增加配额。

### 任务 2：验证实验室是否有足够数量的 vCPU 核心，以使用 Azure SAP 解决方案中心进行自动部署

> **注意**：若要按照说明完成此实验室，需要一个 Microsoft Azure 订阅，其 vCPU 配额可容纳以下 VM 的部署：

- 对于 ASCS 层，2 个 Standard_E4ds_v4 VM（每个 VM 4 个 vCPU 和 32 GiB 内存）或 2 个 Standard_D4ds_v4 VM（每个 VM 4 个 vCPU 和 16 GiB 内存）
- 对于应用层，2 个 Standard_E4ds_v4 VM（每个 VM 4 个 vCPU 和 32 GiB 内存）或 2 个 Standard_D4ds_v4 VM（每个 VM 4 个 vCPU 和 16 GiB 内存） 
- 对于数据层，2 个 Standard_M64ms VM（每个 VM 64 个 vCPU 和 1750 GiB 内存）

1. 从实验室计算机中启动 Web 浏览器，并导航到 Azure 门户 ( )。
1. 在 Azure 门户中，选择“Cloud Shell”图标，在 Cloud Shell 中打开一个 PowerShell 会话****。 
1. 在 Azure 门户的 Cloud Shell 窗格中，在 PowerShell 提示符处运行以下命令：其中 `<Azure_region>` 指定你打算在本实验室中部署资源的 Azure 区域（例如 `eastus`）：****

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **注意**：若要确定 Azure 区域的名称，请在 Cloud Shell 的 Bash 提示符下运行 `(Get-AzLocation).Location`。****

1. 查看输出以确定当前 vCPU 使用情况和 vCPU 限制。 确保它们之间的差异足以容纳将在本实验室中部署的 Azure VM 的 vCPU。 同时考虑特定于 VM 系列的 vCPU 数和区域 vCPU 总数。 
1. 如果 vCPU 数不足，请关闭“Cloud Shell”窗格，在 Azure 门户的“搜索”文本框中，搜索并选择“配额”********。
1. 在“配额”页上，选择“计算”********。
1. 在“配额 \| 计算”页上，使用“区域”筛选器选择要在此实验室中部署资源的 Azure 区域********。
1. 在“配额名称”列中，找到并选择需要增加配额的 VM SKU 名称****。 
1. 在同一行中，查看“可调整”列中的条目****。 下一步取决于该列中包含的条目是“是”还是“否”********。

   - 如果该条目设置为“是”，请选择“请求调整”图标，在“新建配额请求”的“新限制”文本框中，输入新的配额限制，然后选择“提交”********************。
   - 如果该条目设置为“否”，请选择“请求访问权限或获取建议”图标，在“配额建议”窗格中，选择“联系支持人员”选项，然后选择“下一步”********************。 

1. 在“新建支持请求”页的“问题描述”选项卡上，指定以下设置，然后选择“下一步”************：

    |设置|“值”|
    |---|---|
    |你的问题与哪些内容相关？|**Azure 服务**|
    |问题类型|“服务和订阅限制(配额)”****|
    |订阅|你在此实验室中使用的 Azure 订阅的名称|
    |配额类型|计算/VM（核心数/vCPU 数）订阅限制提高****|

1. 在“其他详细信息”选项卡上，选择“输入详细信息”********。
1. 在“配额详细信息”选项卡的“部署模型”下拉列表中，选择“资源管理器”，在“位置”下拉列表中选择目标 Azure 区域，在“配额”下拉列表中，选择需要增加其配额限制的 Azure VM 系列，在“新限制”文本框中，输入新的配额限制，然后选择“保存并继续”****************************。
1. 返回“其他详细信息”选项卡，在“高级诊断信息”选项卡中，选择“是(建议)”************。
1. 在“支持方法”部分中，选择“电子邮件”或“电话”作为首选联系方法，然后选择“下一步”****************。
1. 在“查看 + 创建”选项卡上，选择“创建”。

    > **备注**：在开始使用实验室“使用 Azure SAP 解决方案中心自动部署”之前，请耐心等待增加配额限制的请求完全得到满足。
