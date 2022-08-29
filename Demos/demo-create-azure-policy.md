# <a name="demonstration-create-an-azure-policy"></a>演示：创建 Azure Policy

## <a name="assign-a-policy"></a>分配策略

1. Launch the Azure Policy service in the Azure portal by clicking <bpt id="p1">**</bpt>All services<ept id="p1">**</ept>, then searching for and selecting <bpt id="p2">**</bpt>Policy<ept id="p2">**</ept>. This service is under <bpt id="p1">**</bpt>Management and Governance<ept id="p1">**</ept>.
2. Select <bpt id="p1">**</bpt>Assignments<ept id="p1">**</ept> on the left side of the Azure Policy page. An assignment is a policy that has been assigned to take place within a specific scope.
3. 选择“策略 - 分配”页面顶部的“分配策略”。
4. 请注意“范围”，它用于确定对哪些资源或资源分组施加策略分配。
5. Select the <bpt id="p1">**</bpt>Policy definition ellipsis<ept id="p1">**</ept> to open the list of available definitions. Take some time to review the built-in policy definitions.
6. 在 Azure 门户中单击“所有服务”，然后搜索并选择“策略”，启动 Azure Policy 服务。 
7. 将“创建托管标识”保留为未选中状态。 
8. 单击**分配**，以创建您的策略。

## <a name="create-and-assign-an-initiative-definition"></a>创建并分配计划定义

1. 选择 Azure Policy 页面左侧“创作”下的“定义”。
2. 选择页面顶部的“+ 计划定义”，打开“计划定义”页面。
3. 提供名称和说明 。
4. “新建”类别。
5. 从右侧面板添加“需要 SQL Server 12.0 版”策略 。
6. 添加你选择的一项其他策略。
7. “保存”更改，然后将计划定义“分配”给订阅。

## <a name="check-for-compliance"></a>检查合规性

1. 返回到“Azure Policy 服务”页面。
2. 选择“合规性”。
3. 查看策略和定义的状态。 