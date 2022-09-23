---
lab:
  title: 00 - 实验先决条件
  module: Module 00 - Lab prerequisites
ms.openlocfilehash: 67327d20cd107870c74a44b27ada759083e07aca
ms.sourcegitcommit: 7f50177c1d1fc8925a1b296f89b243b8427a593e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/25/2022
ms.locfileid: "145868736"
---
# <a name="az-120-lab-prerequisites"></a>AZ 120:实验先决条件

## <a name="vcpu-core-requirements"></a>vCPU 核心要求

-   要完成本课程的最后一个实验室，你需要一个 Microsoft Azure 订阅，而且在支持本实验室部署的 Azure VM 所在可用性区域的 Azure 区域中至少有 28 个 vCPU 可用。

    -   4 x Standard_DS1_v2（每个 1 个 vCPU）= 4

    -   6 x Standard_D4s_v3（每个 4 个 vCPU）= 24

    > **注意**：请考虑使用“美国东部”或“美国东部 2”区域来部署资源。

    > **注意**：要确定支持可用性区域的 Azure 区域，请参阅 <https://docs.microsoft.com/en-us/azure/availability-zones/az-overview>

虽然本课程前三个实验室的 vCPU 要求较低，但我们建议你请求增加配额以满足所有实验室的要求，因为增加配额可能需要一些时间（即使配额增加请求通常是在请求当天的工作日完成）。

## <a name="before-the-hands-on-lab"></a>动手实验室准备工作

时间范围：30 分钟

### <a name="task-1-validate-sufficient-number-of-vcpu-cores"></a>任务 1：验证 vCPU 核心数量是否足够

1.  在 Azure 门户 (<http://portal.azure.com>) 中， 

1.  在 Azure 门户中，在 Cloud Shell 中启动 PowerShell 会话。 

    > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。 如果是，接受默认设置，这样会在自动生成的资源组中创建存储账户。

1.  在 Azure 门户的 Cloud Shell 窗格中，在 PowerShell 提示符处运行以下命令：其中 `<Azure_region>` 指定你打算用于本实验室的目标 Azure 区域（例如 `eastus`）：

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **注意**：若要确定 Azure 区域的名称，请在 Cloud Shell 的 Bash 提示符下运行 `(Get-AzLocation).Location`。
   
1.  查看上一步执行命令输出中的当前值和限制条目，并确保目标 Azure 区域中有足够数量的 vCPU。

1.  如果 vCPU 数量不足，请在 Azure 门户中导航回"订阅"边栏选项卡，然后单击 **使用 + 配额**。 

1.  在"订阅"的 **使用 + 配额** 边栏选项卡，单击 **请求增加**。

1.  在“问题描述”边栏选项卡上，指定以下内容：

    -   问题类型：“服务和订阅限制(配额)”

    -   订阅：在本次实验室中你将使用的 Azure 订阅名

    -   配额类型：计算/VM（核心数/vCPU 数）订阅限制提高

1. 单击“管理配额”。

1. 使用位置下拉菜单将结果筛选到计划使用的 Azure 区域。 建议使用“美国东部”或“美国东部 2” 。

1. 找到“标准 DSv3 系列 vCPU”配额类型，然后选择编辑铅笔。

1. 在“新限制”字段中，指定 40，然后单击“保存并继续” 。

1. 找到“区域 vCPU 总数”配额类型，然后选择编辑铅笔。

1. 在“新限制”字段中，指定 40，然后单击“保存并继续” 。

   > **注意**：应自动批准配额增加请求。 如果请求被拒绝，请继续创建支持工单以请求增加配额。
