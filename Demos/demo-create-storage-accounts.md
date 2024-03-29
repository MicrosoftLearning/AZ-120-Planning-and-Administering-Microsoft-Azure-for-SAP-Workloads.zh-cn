# <a name="demonstration-create-storage-accounts"></a>演示：创建存储帐户

## <a name="create-a-storage-account-in-the-portal"></a>在门户中创建存储帐户

1.  在 Azure 门户中，选择“所有服务”。 在资源列表中，键入“存储帐户”  。 开始键入时，会根据输入筛选该列表。 选择“存储帐户”  。
2.  在出现的“存储帐户”窗口中，选择“添加”。
3.  选择要在其中创建存储帐户的订阅。
4.  在“资源组”字段下，选择“新建”。 输入新资源组的名称。
5.  为你的存储帐户输入“名称”。 所选名称在 Azure 中必须唯一。 该名称还必须为 3 到 24 个字符，并且只能包含数字和小写字母。
6.  为你的存储帐户选择位置，或使用默认位置。
7.  将这些字段设置为其默认值：

     * 部署模型：**Resource Manager**
     * 性能：“标准”
     * 帐户类型：**存储 V2（常规用途 v2）**
     * 复制：“本地冗余存储(LRS)”
     * 访问层：**热访问层**

8.  选择“查看+创建”可查看存储帐户设置并创建帐户。
9.  选择“创建”  。

## <a name="create-a-storage-account-using-powershell"></a>使用 PowerShell 创建存储帐户

使用以下代码通过 PowerShell 创建存储帐户。 根据你的要求修改存储类型和名称。

```PowerShell
Get-AzLocation | select Location 
$location = "westus" 
$resourceGroup = "storage-demo-resource-group" 
New-AzResourceGroup -Name $resourceGroup -Location $location 
New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "storagedemo" -Location $location -SkuName Standard_LRS -Kind StorageV2 
```

## <a name="create-a-storage-account-using-azure-cli"></a>使用 Azure CLI 创建存储帐户

使用以下代码通过 Azure CLI 创建存储帐户。 更改存储类型和名称以满足你的要求。

```PowerShell
az group create --name storage-resource-group --location westus 
az account list-locations --query "[].{Region:name}" --out table 
az storage account create --name storagedemo --resource-group storage-resource-group --location westus --sku Standard_LRS --kind StorageV2 
```