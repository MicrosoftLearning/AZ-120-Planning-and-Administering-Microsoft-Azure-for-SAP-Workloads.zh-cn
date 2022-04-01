---
ms.openlocfilehash: 8ab7cb6b36f91aa53099436a1b67073cee8d09db
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857562"
---
# <a name="demonstration-explore-quickstart-templates"></a>演示：了解快速启动模板

## <a name="explore-the-gallery"></a>探索模板库

1. 你可以首先浏览到 [Azure 快速启动模板库](https://azure.microsoft.com/resources/templates?azure-portal=true)。 在该库中，你将找到许多最近更新的热门模板。 这些模板适用于 Azure 资源和常用软件包。
2. 浏览多种不同类型的可用模板。
3. 是否有你感兴趣的模板？

## <a name="explore-a-template"></a>探索模板

1. 假设我们无意中发现<a href="https://azure.microsoft.com/resources/templates/101-vm-simple-windows?azure-portal=true" target="_blank"><span style="color: #0066cc;" color="#0066cc">部署简单 Windows VM</span></a> 模板。

    ![此屏幕截图中显示了“部署一个简单的 Windows VM” 页面](Images/AZ103_Demo_QS_Templates2.png)

    >**注意：**
    >- 使用“部署到 Azure”按钮可根据需要直接通过 Azure 门户部署模板。
    >- 向下滚动到“使用模板 PowerShell 代码”。 下一个演示需要用到“TemplateURI”。 **复制该值。** 

```
https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
```

2. 单击“在 GitHub 中浏览”，导航到 GitHub 上的模板源代码。

    ![此屏幕截图中显示了资源管理器模板的 GitHub 自述文件](Images/AZ103_Demo_QS_Templates3.png)

3. 注意，在此页面上还可以选择“部署到 Azure”。 花一分钟查看自述文件。 这有助于确定模板是否适合你。  

4. 单击“可视化”导航到“Azure 资源管理器可视化工具” 。

    ![Azure 资源管理器可视化工具显示 Azure 资源。](Images/AZ103_Demo_QS_Templates4.png)

5. 注意构成部署的资源，包括 VM、存储帐户和网络资源。
6. 使用鼠标来排列资源。 此外，还可以使用鼠标的滚轮进行放大和缩小。
7. 单击标签为“SimpleWinVM”的 VM 资源。

    ![Azure 资源管理器可视化工具显示模板的源代码。](Images/AZ103_Demo_QS_Templates5.png)

8. 查看定义 VM 资源的源代码。

    * 资源类型为 `Microsoft.Compute/virtualMachines`。
    * 其位置或 Azure 区域来自名为 `location` 的模板参数。
    * VM 的大小为 Standard_A2。
    * 计算机名称读取自模板变量，VM 的用户名和密码读取自模板参数。

9. 返回显示模板中文件的“快速入门”页面。 将链接复制到 azuredeploy.json 文件。 其将采用以下格式：

>**注意：** 下一个演示需要用到该模板链接。