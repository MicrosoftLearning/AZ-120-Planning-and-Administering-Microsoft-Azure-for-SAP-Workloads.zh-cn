# <a name="demonstration-set-up-site-recovery-for-virtual-machines-in-proximity-placement-groups---azure-to-azure"></a>演示：在“邻近放置组”为虚拟机设置站点恢复 - Azure 到 Azure

> **注意：** 此功能目前通过 PowerShell 提供，并支持任何使用托管磁盘的 Azure VM。

## <a name="prerequisites"></a>先决条件

1. 确保已有 Azure PowerShell Az 模块。 如需进安装或升级 Azure PowerShell，请遵循此[安装和配置 Azure PowerShell 指南](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-5.0.0)。
2. 最小的 Azure PowerShell Az 版本应为 4.1.0。 要查看当前版本，请使用以下命令：

    ```azurepowershell
    Get-InstalledModule -Name Az
    ```

>**注意：** 确保你拥有可用的目标邻近放置组的唯一 ID。 如果要创建新的邻近放置组，请查看[此处](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#create-a-proximity-placement-group)的命令；如果使用的是现有邻近放置组，请使用[此处](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups#list-proximity-placement-groups)的命令。

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>登录到 Microsoft Azure 订阅

1. 使用 `Connect-AzAccount` cmdlet 登录到你的 Azure 订阅。

    ```azurepowershell
    Connect-AzAccount
    ```

1. 选择 Azure 订阅。 使用 `Get-AzSubscription` cmdlet 获取你有权访问的 Azure 订阅的列表。 使用 `Set-AzContext` cmdlet 选择要使用的 Azure 订阅。

    ```azurepowershell
    Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ```

## <a name="get-details-of-the-virtual-machine-to-be-replicated"></a>获取要复制的虚拟机的详细信息

1. 在此演示中，将美国东部区域的虚拟机复制到美国西部 2 区域并进行恢复。 要复制的虚拟机具有 OS 磁盘和单个数据磁盘。 本示例中使用的虚拟机名称为 `AzureDemoVM`。

    ```azurepowershell
    # Get details of the virtual machine
    $VM = Get-AzVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

    Write-Output $VM
    ```

    ```Output
    ResourceGroupName  : A2AdemoRG
    Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
    VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
    Name               : AzureDemoVM
    Type               : Microsoft.Compute/virtualMachines
    Location           : eastus
    Tags               : {}
    DiagnosticsProfile : {BootDiagnostics}
    HardwareProfile    : {VmSize}
    NetworkProfile     : {NetworkInterfaces}
    OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
    ProvisioningState  : Succeeded
    StorageProfile     : {ImageReference, OsDisk, DataDisks}
    ```

1. 获取虚拟机磁盘的磁盘详细信息。 稍后在开始复制虚拟机时，将使用磁盘详细信息。

    ```azurepowershell
    $OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
    $DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
    ```

## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库

1. 创建要在其中创建恢复服务保管库的资源组。

    > **重要提示：**
    > * 恢复服务保管库和受保护的虚拟机必须位于不同的 Azure 位置。
    > * 恢复服务保管库的资源组和受保护的虚拟机必须位于不同的 Azure 位置。
    > * 恢复服务保管库及其所属的资源组可以位于相同的 Azure 位置。

1. 在此演示中，受保护的虚拟机位于美国东部区域。 为灾难恢复选择的恢复区域为美国西部 2 区域。 恢复服务保管库及其资源组都位于恢复区域“美国西部 2”。

    ```azurepowershell
    #Create a resource group for the recovery services vault in the recovery Azure region
    New-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"
    ```

    ```Output
    ResourceGroupName : a2ademorecoveryrg
    Location          : westus2
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg
    ```

1. 创建恢复服务保管库。 本示例在“美国西部 2”区域创建名为 `a2aDemoRecoveryVault` 的恢复服务保管库。

    ```azurepowershell
    #Create a new Recovery services vault in the recovery region
    $vault = New-AzRecoveryServicesVault -Name "a2aDemoRecoveryVault" -ResourceGroupName "a2ademorecoveryrg" -Location "West US 2"

    Write-Output $vault
    ```

    ```Output
    Name              : a2aDemoRecoveryVault
    ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoRecoveryVault
    Type              : Microsoft.RecoveryServices/vaults
    Location          : westus2
    ResourceGroupName : a2ademorecoveryrg
    SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
    ```

## <a name="set-the-vault-context"></a>设置保管库上下文

1. 设置 PowerShell 会话中使用的保管库上下文。 设置保管库上下文后，PowerShell 会话中的后续 Azure Site Recovery 操作将在所选保管库的上下文中执行。

    ```azurepowershell
    #Setting the vault context.
    Set-AzRecoveryServicesAsrVaultContext -Vault $vault
    ```

    ```Output
    ResourceName         ResourceGroupName ResourceNamespace          ResourceType
    ------------         ----------------- -----------------          -----------
    a2aDemoRecoveryVault a2ademorecoveryrg Microsoft.RecoveryServices Vaults
    ```

    ```azurepowershell
    #Delete the downloaded vault settings file
    Remove-Item -Path $Vaultsettingsfile.FilePath
    ```

1. 对于 Azure 到 Azure 迁移，可以将保管库上下文设置为新创建的保管库：

    ```azurepowershell
    #Set the vault context for the PowerShell session.
    Set-AzRecoveryServicesAsrVaultContext -Vault $vault
    ```

## <a name="prepare-the-vault-to-start-replicating-azure-virtual-machines"></a>准备保管库以开始复制 Azure 虚拟机

### <a name="create-a-site-recovery-fabric-object-to-represent-the-primary-source-region"></a>创建站点恢复结构对象以表示主要（源）区域

保管库中的结构对象表示 Azure 区域。 创建主要结构对象，以表示要在保管库中保护的虚拟机所属的 Azure 区域。 在此演示中，受保护的虚拟机位于美国东部区域。

- 每个区域只能创建一个结构对象。
- 如果先前已在 Azure 门户中为 VM 启用了站点恢复复制，则站点恢复会自动创建一个结构对象。 如果区域存在结构对象，则无法创建新结构对象。

开始前，请了解站点恢复操作是异步执行的。 启动操作时，将提交 Azure Site Recovery 作业，并返回跟踪对象的作业。

1. 使用作业跟踪对象获得最新的状态作业 (`Get-AzRecoveryServicesAsrJob`) 和监视操作状态。

    ```azurepowershell
    #Create Primary ASR fabric
    $TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'East US'  -Name "A2Ademo-EastUS"

    # Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            #If the job hasn't completed, sleep for 10 seconds before checking the job status again
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State

    $PrimaryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-EastUS"
    ```

1. 如果在同一个保管库中保护来自多个 Azure 区域的虚拟机，请为每个源 Azure 区域创建一个结构对象。

### <a name="create-a-site-recovery-fabric-object-to-represent-the-recovery-region"></a>创建站点恢复结构对象以表示恢复区域

恢复结构对象表示 Azure 恢复位置。 如果发生故障转移，虚拟机将复制并恢复到该恢复结构表示的恢复区域。 本示例使用的 Azure 恢复区域是“美国西部 2”。

```azurepowershell
#Create Recovery ASR fabric
$TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'West US 2'  -Name "A2Ademo-WestUS"

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$RecoveryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-WestUS"
```

## <a name="create-site-recovery-protection-containers"></a>创建站点恢复保护容器

### <a name="create-a-site-recovery-protection-container-in-the-primary-fabric"></a>在主结构中创建站点恢复保护容器

保护容器是用于在一个结构中分组复制项的容器。

```azurepowershell
#Create a Protection container in the primary Azure region (within the Primary fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $PrimaryFabric -Name "A2AEastUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

Write-Output $TempASRJob.State

$PrimaryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $PrimaryFabric -Name "A2AEastUSProtectionContainer"
```

### <a name="create-a-site-recovery-protection-container-in-the-recovery-fabric"></a>在恢复结构中创建站点恢复保护容器

```azurepowershell
#Create a Protection container in the recovery Azure region (within the Recovery fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $RecoveryFabric -Name "A2AWestUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"

Write-Output $TempASRJob.State

$RecoveryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $RecoveryFabric -Name "A2AWestUSProtectionContainer"
```

## <a name="create-a-replication-policy"></a>创建复制策略

```azurepowershell
#Create replication policy
$TempASRJob = New-AzRecoveryServicesAsrPolicy -AzureToAzure -Name "A2APolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$ReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "A2APolicy"
```

## <a name="create-protection-container-mappings"></a>创建保护容器映射

### <a name="create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container"></a>在主要保护容器与恢复保护容器之间创建保护容器映射

保护容器映射将主要保护容器映射到恢复保护容器和复制策略。 针对在保护容器对之间复制虚拟机时所用的每个复制策略各创建一个映射。

```azurepowershell
#Create Protection container mapping between the Primary and Recovery Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2APrimaryToRecovery" -Policy $ReplicationPolicy -PrimaryProtectionContainer $PrimaryProtContainer -RecoveryProtectionContainer $RecoveryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$EusToWusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $PrimaryProtContainer -Name "A2APrimaryToRecovery"
```

### <a name="create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover"></a>创建用于故障回复（故障转移后的反向复制）的保护容器映射

在故障转移后，准备好将故障转移的虚拟机恢复到原始 Azure 区域时，执行故障回复。 为了进行故障回复，故障转移的虚拟机将从故障转移的区域反向复制到原始区域。 反向复制时，原始区域和恢复区域的角色将会切换。 原始区域现在变成新的恢复区域，而最初的恢复区域现在会变成主要区域。 反向复制的保护容器映射表示原始和恢复区域的已切换角色。

```azurepowershell
#Create Protection container mapping (for fail back) between the Recovery and Primary Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2ARecoveryToPrimary" -Policy $ReplicationPolicy -PrimaryProtectionContainer $RecoveryProtContainer -RecoveryProtectionContainer $PrimaryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$WusToEusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $RecoveryProtContainer -Name "A2ARecoveryToPrimary"
```

## <a name="create-cache-storage-account-and-target-storage-account"></a>创建缓存存储帐户和目标存储帐户

缓存存储帐户是复制的虚拟机所在的同一 Azure 区域中的标准存储帐户。 缓存存储帐户用于在将复制更改移至恢复 Azure 区域之前临时保留这些更改。 可以为虚拟机的不同磁盘指定不同的缓存存储帐户，但这不是必需的。

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$EastUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestorage" -ResourceGroupName "A2AdemoRG" -Location 'East US' -SkuName Standard_LRS -Kind Storage
```

对于**未使用托管磁盘**的虚拟机，目标存储帐户是虚拟机磁盘复制到的恢复区域中的存储帐户。 目标存储帐户可以是标准存储帐户，也可以是高级存储帐户。 根据磁盘的数据更改率（IO 写入率）以及 Azure Site Recovery 对存储类型支持的变动限制，来选择所需的存储帐户类型。

```azurepowershell
#Create Target storage account in the recovery region. In this case a Standard Storage account
$WestUSTargetStorageAccount = New-AzStorageAccount -Name "a2atargetstorage" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -SkuName Standard_LRS -Kind Storage
```

## <a name="create-network-mappings"></a>创建网络映射

网络映射将主要区域中的虚拟网络映射到恢复区域中的虚拟网络。 网络映射指定主要虚拟网络中的虚拟机应故障转移到的恢复区域中的 Azure 虚拟网络。 一个 Azure 虚拟网络只能映射到恢复区域中的单个 Azure 虚拟网络。

1. 在恢复区域中创建要故障转移到的 Azure 虚拟网络：

   ```azurepowershell
    #Create a Recovery Network in the recovery region
    $WestUSRecoveryVnet = New-AzVirtualNetwork -Name "a2arecoveryvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.0.0.0/16"

    Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $WestUSRecoveryVnet -AddressPrefix "10.0.0.0/20" | Set-AzVirtualNetwork

    $WestUSRecoveryNetwork = $WestUSRecoveryVnet.Id
   ```

1. 检索主要虚拟网络。 虚拟机连接到的 VNet：

   ```azurepowershell
    #Retrieve the virtual network that the virtual machine is connected to

    #Get first network interface card(nic) of the virtual machine
    $SplitNicArmId = $VM.NetworkProfile.NetworkInterfaces[0].Id.split("/")

    #Extract resource group name from the ResourceId of the nic
    $NICRG = $SplitNicArmId[4]

    #Extract resource name from the ResourceId of the nic
    $NICname = $SplitNicArmId[-1]

    #Get network interface details using the extracted resource group name and resource name
    $NIC = Get-AzNetworkInterface -ResourceGroupName $NICRG -Name $NICname

    #Get the subnet ID of the subnet that the nic is connected to
    $PrimarySubnet = $NIC.IpConfigurations[0].Subnet

    # Extract the resource ID of the Azure virtual network the nic is connected to from the subnet ID
    $EastUSPrimaryNetwork = (Split-Path(Split-Path($PrimarySubnet.Id))).Replace("\","/")
   ```

1. 在主要虚拟网络与恢复虚拟网络之间创建网络映射：

   ```azurepowershell
    #Create an ASR network mapping between the primary Azure virtual network and the recovery Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AEusToWusNWMapping" -PrimaryFabric $PrimaryFabric -PrimaryAzureNetworkId $EastUSPrimaryNetwork -RecoveryFabric $RecoveryFabric -RecoveryAzureNetworkId $WestUSRecoveryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
   ```

1. 为反向复制（故障回复）创建网络映射：

    ```azurepowershell
    #Create an ASR network mapping for fail back between the recovery Azure virtual network and the primary Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AWusToEusNWMapping" -PrimaryFabric $RecoveryFabric -PrimaryAzureNetworkId $WestUSRecoveryNetwork -RecoveryFabric $PrimaryFabric -RecoveryAzureNetworkId $EastUSPrimaryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
    ```

## <a name="replicate-an-azure-virtual-machine-with-managed-disks"></a>复制包含托管磁盘的 Azure 虚拟机

使用以下 PowerShell cmdlet 复制具有托管磁盘的 Azure 虚拟机：

    ```azurepowershell
    #Get the resource group that the virtual machine must be created in when it's failed over.
    $RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

    #Specify replication properties for each disk of the VM that will be replicated (create disk replication configuration).
    #Make sure to replace the variable $OSdiskName with the OS disk name.

    #OS Disk
    $OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
    $OSdiskId = $OSdisk.Id
    $RecoveryOSDiskAccountType = $OSdisk.Sku.Name
    $RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

    $OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

    #Make sure to replace the variable $datadiskName with the data disk name.

    #Data disk
    $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
    $datadiskId1 = $datadisk[0].Id
    $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
    $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name

    $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $datadiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

    #Create a list of disk replication configuration objects for the disks of the virtual machine that will be replicated.

    $diskconfigs = @()
    $diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

    #Start replication by creating a replication protected item. Use a GUID for the name of the replication protected item to ensure uniqueness of the name.

    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```

    When you're enabling replication for multiple data disks, use the following PowerShell cmdlet:

    ```azurepowershell
    #Get the resource group that the virtual machine must be created in when it's failed over.
    $RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

    #Specify replication properties for each disk of the VM that will be replicated (create disk replication configuration).
    #Make sure to replace the variable $OSdiskName with the OS disk name.

    #OS Disk
    $OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
    $OSdiskId = $OSdisk.Id
    $RecoveryOSDiskAccountType = $OSdisk.Sku.Name
    $RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

    $OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

    $diskconfigs = @()
    $diskconfigs += $OSDiskReplicationConfig

    #Data disk

    # Add data disks
    Foreach( $disk in $VM.StorageProfile.DataDisks)
    {
        $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
        $dataDiskId1 = $datadisk[0].Id
        $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
        $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name
        $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
             -DiskId $dataDiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType `
             -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType
        $diskconfigs += $DataDisk1ReplicationConfig
    }

    #Start replication by creating a replication protected item. Use a GUID for the name of the replication protected item to ensure uniqueness of the name.

    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```

    When you're enabling zone-to-zone replication with a proximity placement group, the command to start replication will be exchanged with the PowerShell cmdlet:

    ```azurepowershell
    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id -RecoveryAvailabilityZone "2"
    ```

    After the operation to start replication succeeds, virtual machine data is replicated to the recovery region.

    The replication process starts by initially seeding a copy of the replicating disks of the virtual machine in the recovery region. This phase is called the *initial replication* phase.

    After initial replication finishes, replication moves to the *differential synchronization* phase. At this point, the virtual machine is protected, and you can perform a test failover operation on it. The replication state of the replicated item that represents the virtual machine goes to the protected state after initial replication finishes.

    Monitor the replication state and replication health for the virtual machine by getting details of the replication protected item that corresponds to it:

    ```azurepowershell
    Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
    ```

## <a name="perform-validate-and-clean-up-a-test-failover"></a>执行、验证和清理测试故障转移

在虚拟机复制进入受保护状态后，可以针对该虚拟机（针对虚拟机的复制保护项）执行测试故障转移操作。

```azurepowershell
#Create a separate network for test failover (not connected to my DR network)
$TFOVnet = New-AzVirtualNetwork -Name "a2aTFOvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.3.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $TFOVnet -AddressPrefix "10.3.0.0/20" | Set-AzVirtualNetwork

$TFONetwork= $TFOVnet.Id
```

执行测试故障转移。

```azurepowershell
$ReplicationProtectedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem -FriendlyName "AzureDemoVM" -ProtectionContainer $PrimaryProtContainer

$TFOJob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -AzureVMNetworkId $TFONetwork -Direction PrimaryToRecovery
```

等待测试故障转移操作完成。

```azurepowershell
Get-AzRecoveryServicesAsrJob -Job $TFOJob
```

```Output
Name             : 3dcb043e-3c6d-4e0e-a42e-8d4245668547
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/3dcb043e-3c6d-4e0e-a42e-8d4245668547
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : TestFailover
DisplayName      : Test failover
ClientRequestId  : 1ef8515b-b130-4452-a44d-91aaf071931c ActivityId: 907bb2bc-ebe6-4732-8b66-77d0546eaba8
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:29:43 AM
EndTime          : 4/25/2018 4:33:06 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisites check for test failover, Create test virtual machine, Preparing the virtual machine, Start the virtual machine}
Errors           : {}
```

测试故障转移作业成功完成后，可以连接到测试故障转移虚拟机，并验证测试故障转移。

对测试故障转移虚拟机完成测试后，通过启动清理测试故障转移操作来清理测试副本。 此操作会删除测试故障转移创建的虚拟机测试副本。

```azurepowershell
$Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $Job_TFOCleanup | Select State
```

```Output
State
-----
Succeeded
```

## <a name="fail-over-the-virtual-machine"></a>故障转移虚拟机

将虚拟机故障转移到特定的恢复点。

```azurepowershell
$RecoveryPoints = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $ReplicationProtectedItem

#The list of recovery points returned may not be sorted chronologically and will need to be sorted first, in order to be able to find the oldest or the latest recovery points for the virtual machine.
"{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[-1].RecoveryPointTime
```

```Output
CrashConsistent 4/24/2018 11:10:25 PM
```

```azurepowershell
#Start the fail over job
$Job_Failover = Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[-1]

do {
        $Job_Failover = Get-AzRecoveryServicesAsrJob -Job $Job_Failover;
        sleep 30;
} while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))

$Job_Failover.State
```

```Output
Succeeded
```

当故障转移作业成功时，你可以提交故障转移操作。

```azurepowershell
$CommitFailoverJOb = Start-AzRecoveryServicesAsrCommitFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $CommitFailoverJOb
```

```Output
Name             : 58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : CommitFailover
DisplayName      : Commit
ClientRequestId  : 10a95d6c-359e-4603-b7d9-b7ee3317ce94 ActivityId: 8751ada4-fc42-4238-8de6-a82618408fcf
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:50:58 AM
EndTime          : 4/25/2018 4:51:01 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisite check, Commit}
Errors           : {}
```

## <a name="reprotect-and-fail-back-to-the-source-region"></a>重新保护并故障回复到源区域

使用以下 PowerShell cmdlet 重新保护并故障回复到源区域：

```azurepowershell
#Create a cache storage account for replication logs in the primary region.
$WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage

#Use the recovery protection container, the new cache storage account in West US, and the source region VM resource group. 
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure -ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId -RecoveryProximityPlacementGroupId $vm.ProximityPlacementGroup.Id
```

## <a name="disable-replication"></a>禁用复制

可以使用 `Remove-AzRecoveryServicesAsrReplicationProtectedItem` cmdlet 禁用复制。

```azurepowershell
Remove-AzRecoveryServicesAsrReplicationProtectedItem -ReplicationProtectedItem $ReplicationProtectedItem
```