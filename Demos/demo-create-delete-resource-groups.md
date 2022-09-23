# <a name="demonstration-create-and-delete-resource-groups"></a>演示：创建和删除资源组

>**注意**：只有所有者和用户访问管理员角色才能管理资源上的锁。

## <a name="manage-resource-groups-in-the-portal"></a>管理门户中的资源组

1. 访问 Azure 门户。
1. Create a resource group. Remember the name of this resource group. 
1. 在资源组的“设置”边栏选项卡中选择“锁”。
1. To add a lock, select <bpt id="p1">**</bpt>Add<ept id="p1">**</ept>. If you want to create a lock at a parent level, select the parent. The currently selected resource inherits the lock from the parent. For example, you could lock the resource group to apply a lock to all its resources.
1. Give the lock a <bpt id="p1">**</bpt>name<ept id="p1">**</ept> and <bpt id="p2">**</bpt>lock type<ept id="p2">**</ept>. Optionally, you can add notes that describe the lock.
1. 若要删除锁，请从可用选项中选择省略号中的“删除”****。

## <a name="manage-resource-groups-with-powershell"></a>使用 PowerShell 管理资源组

1. 访问 Cloud Shell。
2. 创建资源锁，并确认操作。

    ```
        New-AzResourceLock -LockName <lockName> -LockLevel CanNotDelete -ResourceGroupName <resourceGroupName>
    ```

3. View resource lock information. Notice the LockId that will be used in the next step to delete the lock.

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