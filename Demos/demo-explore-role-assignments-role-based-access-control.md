---
ms.openlocfilehash: 5cf6ad2fc19b92d11ea26eee7dea0e40548d09da
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857558"
---
# <a name="demonstration-explore-role-assignments-and-role-based-access-control"></a>演示：了解角色分配和基于角色的访问控制

## <a name="locate-access-control-blade"></a>找到“访问控制”边栏选项卡

1. 访问 Azure 门户，选择一个资源组。 记下所使用的资源组。 
2. 选择“访问控制(IAM)”边栏选项卡。 
3. 此边栏选项卡将可用于许多不同的资源，因此你可以控制访问权限。

## <a name="review-role-permissions"></a>查看角色权限

1. 选择“角色”选项卡（顶部）。
2. 查看大量可用的内置角色。
3. 双击某个角色，然后选择“权限”（顶部）。
4. 继续钻取到角色中，直到可以查看该角色的“读取、写入和删除”操作。
5. 返回到“访问控制(IAM)”边栏选项卡。

## <a name="add-a-role-assignment"></a>添加角色分配

1. 选择“添加角色分配”。 

    + **角色**：*所有者*
    + **Select**：管理者
    + 单击“保存”以保存更改。 

2. 选择“检查访问”  。
3. 找到 Chris Green。
4. 请注意，他是“管理者”组的成员，并且是所有者。 
5. 请注意，你可以“拒绝分配”。 

## <a name="explore-powershell-commands"></a>探索 PowerShell 命令

1. 打开 Azure Cloud Shell。
2. 选择 PowerShell 下拉列表。
3. 列出角色定义。

    ```PowerShell
    Get-AzRoleDefinition | FT Name, Description
    ```

4. 列出角色的操作。

    ```PowerShell
    Get-AzRoleDefinition owner | FL Actions, NotActions
    ```

5. 列出角色分配。

    ```PowerShell
    Get-AzRoleAssignment -ResourceGroupName <resource group name>
    ```