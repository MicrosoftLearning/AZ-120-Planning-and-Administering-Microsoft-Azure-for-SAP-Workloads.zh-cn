# <a name="demonstration-back-up-azure-file-shares"></a>演示：备份 Azure 文件共享

在本演示中，我们将探讨在 Azure 门户中备份文件共享。

>**注意：** 此演示需要 Azure 文件共享和保管库可以使用的存储帐户。 

## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库

1. 在 Azure 门户中，输入“恢复服务”并单击“恢复服务保管库”。
3. 单击“添加”。
4. 提供“名称”、“订阅”、“资源组”和“位置”   。 
5. 新建的保管库应与文件共享位于同一位置。 
5. Click <bpt id="p1">**</bpt>Create<ept id="p1">**</ept>. It can take several minutes for the Recovery Services vault to be created. Monitor the status notifications in the upper right-hand area of the portal. Once your vault is created, it appears in the list of Recovery Services vaults. 
6. 如果几分钟后未添加保管库，请单击“刷新”。

## <a name="configure-the-vault"></a>配置保管库

1. 打开恢复服务保管库。 
2. 单击“备份”并创建一个新的备份实例。 
3. 从“你的工作负载在何处运行?”下拉菜单中，选择“Azure”。
4. 从**需备份什么？** 菜单，选择 **Azure FileShare**。
5. 单击“备份”****。
6. From the list of Storage accounts, <bpt id="p1">**</bpt>select a storage account<ept id="p1">**</ept>, and click <bpt id="p2">**</bpt>OK<ept id="p2">**</ept>. Azure searches the storage account for files shares that can be backed up. If you recently added your file shares, allow a little time for the file shares to appear.
7. 从“文件共享”列表中，**选择一个或多个需备份的文件共享**，然后单击“确定”。
8. On the Backup Policy page, choose <bpt id="p1">**</bpt>Create New backup policy<ept id="p1">**</ept> and provide Name, Schedule, and Retention information. Click <bpt id="p1">**</bpt>OK<ept id="p1">**</ept>.
9. 完成配置备份后，请单击“启用备份”。 

## <a name="explore-recovery-services-vault-information"></a>浏览恢复服务保管库信息

1. Explore the <bpt id="p1">**</bpt>Backup items<ept id="p1">**</ept> blade. There is information on backed up items and replicated items.
2. Explore the <bpt id="p1">**</bpt>Backup policies<ept id="p1">**</ept> blade. You can add or delete backup policies. 
3. Explore the <bpt id="p1">**</bpt>Backup jobs<ept id="p1">**</ept> blade. Here you can review the status of your backup jobs.