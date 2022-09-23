# <a name="demonstration-create-storage-accounts"></a>演示：创建存储帐户

## <a name="create-a-storage-account-in-the-portal"></a>在门户中创建存储帐户

1.  In the Azure portal, select <bpt id="p1">**</bpt>All services<ept id="p1">**</ept>. In the list of resources, type Storage Accounts. As you begin typing, the list filters based on your input. Select <bpt id="p1">**</bpt>Storage Accounts<ept id="p1">**</ept>.
2.  在出现的“存储帐户”窗口中，选择“添加”。
3.  选择要在其中创建存储帐户的订阅。
4.  Under the Resource group field, select <bpt id="p1">**</bpt>Create new<ept id="p1">**</ept>. Enter a name for your new resource group.
5.  Enter a <bpt id="p1">**</bpt>name<ept id="p1">**</ept> for your storage account. The name you choose must be unique across Azure. The name also must be between 3 and 24 characters in length, and can include numbers and lowercase letters only.
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

在 Azure 门户中，选择“所有服务”。

```PowerShell
Get-AzLocation | select Location 
$location = "westus" 
$resourceGroup = "storage-demo-resource-group" 
New-AzResourceGroup -Name $resourceGroup -Location $location 
New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "storagedemo" -Location $location -SkuName Standard_LRS -Kind StorageV2 
```

## <a name="create-a-storage-account-using-azure-cli"></a>使用 Azure CLI 创建存储帐户

在资源列表中，键入“存储帐户”  。

```PowerShell
az group create --name storage-resource-group --location westus 
az account list-locations --query "[].{Region:name}" --out table 
az storage account create --name storagedemo --resource-group storage-resource-group --location westus --sku Standard_LRS --kind StorageV2 
```