# <a name="demonstration-create-and-manage-file-shares-and-snapshots"></a>演示：创建和管理文件共享与快照

>**注意**：这些步骤需要一个存储帐户。 

## <a name="create-a-file-share-and-upload-a-file"></a>创建文件共享和上传文件

1. 访问你的存储帐户，然后单击“文件”。
2. 单击“+文件共享”，然后提供新文件共享的“名称”和“配额”  。
3. 创建文件共享后，上传文件。 
4. 注意，可以添加目录、删除共享和编辑配额。

## <a name="manage-snapshots"></a>管理快照

1. 访问文件共享。
1. 选择“创建快照”。
1. 选择“查看快照”并验证已创建快照。
1. 单击快照并验证其是否包含上传的文件。
1. 单击属于快照的文件，并查看文件属性。 
1. 注意“下载”和“恢复”快照文件的选项。 
1. 访问文件共享并删除之前上传的文件。
1. 从快照恢复文件。 
 
## <a name="create-a-file-share-powershell"></a>创建文件共享 (PowerShell)

1. 为存储帐户和密钥创建上下文：上下文封装存储帐户名称和帐户密钥。

    ```PowerShell
    $storageContext = New-AzStorageContext storage-account-name storage-account-key
    ```

2. Create the file share. The name of your file share must be all lowercase.

    ```PowerShell
    $share = New-AzStorageShare logs -Context $storageContext
    ```

## <a name="mount-a-file-share-powershell"></a>装载文件共享 (PowerShell)

1. Run the following commands from a regular (i.e. not an elevated) PowerShell session to mount the Azure file share. Remember to replace <bpt id="p1">**</bpt>your-resource-group-name<ept id="p1">**</ept>, <bpt id="p2">**</bpt>your-storage-account-name<ept id="p2">**</ept>, <bpt id="p3">**</bpt>your-file-share-name<ept id="p3">**</ept>, and <bpt id="p4">**</bpt>desired-drive-letter<ept id="p4">**</ept> with the proper information.

    ```PowerShell
    $resourceGroupName = "your-resource-group-name"
    $storageAccountName = "your-storage-account-name"
    $fileShareName = "your-file-share-name"

    # These commands require you to be logged into your Azure account, run Login-AzAccount if you haven't
    # already logged in.
    $storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
    $storageAccountKeys = Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName
    $fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object { 
        $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
    }

    if ($fileShare -eq $null) {
        throw [System.Exception]::new("Azure file share not found")
    }

    # The value given to the root parameter of the New-PSDrive cmdlet is the host address for the storage account, 
    # storage-account.file.core.windows.net for Azure Public Regions. $fileShare.StorageUri.PrimaryUri.Host is 
    # used because non-Public Azure regions, such as sovereign clouds or Azure Stack deployments, will have different 
    # hosts for Azure file shares (and other storage resources).
    $password = ConvertTo-SecureString -String $storageAccountKeys[0].Value -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "AZURE\$($storageAccount.StorageAccountName)", $password
    New-PSDrive -Name desired-drive-letter -PSProvider FileSystem -Root "\\$($fileShare.StorageUri.PrimaryUri.Host)\$($fileShare.Name)" -Credential $credential -Persist
    ```

2. 完成后，可以通过运行以下命令卸除文件共享：

    ```PowerShell
    Remove-PSDrive -Name desired-drive-letter
    ```