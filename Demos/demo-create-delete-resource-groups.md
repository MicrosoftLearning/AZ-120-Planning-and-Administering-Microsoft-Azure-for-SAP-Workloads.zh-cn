---
ms.openlocfilehash: f925b00bee5e45d5636ec833a64601cfe8df7475
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857514"
---
# <a name="demonstration-create-and-delete-resource-groups"></a>演示：创建和删除资源组

>**注意**：只有所有者和用户访问管理员角色才能管理资源上的锁。

## <a name="manage-resource-groups-in-the-portal"></a>管理门户中的资源组

1. 访问 Azure 门户。
1. 创建资源组。 记住该资源组的名称。 
1. 在资源组的“设置”边栏选项卡中选择“锁”。
1. 要添加锁，请选择“添加”。 如果要在父级别创建锁，请选择父级。 当前选定的资源将从父级继承锁。 例如，可以锁定资源组，以便向其所有资源应用锁。
1. 设定锁的“名称”和“锁类型” 。 （可选）可以添加注释来描述该锁。
1. 若要删除锁，请从可用选项中选择省略号中的“删除”。

## <a name="manage-resource-groups-with-powershell"></a>使用 PowerShell 管理资源组

1. 访问 Cloud Shell。
2. 创建资源锁，并确认操作。

    ```
        New-AzResourceLock -LockName <lockName> -LockLevel CanNotDelete -ResourceGroupName <resourceGroupName>
    ```

3. 查看资源锁信息。 注意 LockId，将在下一步中用它来删除该锁。

    ```
        Get-AzResourceLock
    ```

4. 删除资源锁，并确认操作。 

    ```
        Remove-AzResourceLock -LockName <Name> -ResourceGroupName <Resource Group>
    ```

5. 验证资源锁是否已删除。


    ```
        Get-AzResourceLock
    ```

>**注意：** 配置资源锁、跨资源组移动资源以及删除资源组是认证考试的一部分。