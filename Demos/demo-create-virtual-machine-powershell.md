---
ms.openlocfilehash: 8253f093bc365b521180798165190e879b37a9af
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857533"
---
# <a name="demonstration-create-a-virtual-machine-with-powershell"></a>演示：使用 PowerShell 创建虚拟机

在本演示中，将使用 PowerShell 创建一个虚拟机。

## <a name="create-the-virtual-machine"></a>创建虚拟机

>注意：您可以使用 Cloud Shell 或本地版本的 PowerShell。 

1. 启动 Cloud Shell。
2. 运行以下代码：

```
    # create a resource group
    New-AzResourceGroup -Name myResourceGroup -Location EastUS

    # create the virtual machine 
    # when prompted, provide a username and password to be used as the logon credentials for the VM
    New-AzVm `
        -ResourceGroupName "myResourceGroup" `
        -Name "myVM" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -PublicIpAddressName "myPublicIpAddress" `
        -OpenPorts 80,3389
```

## <a name="verify-the-machine-creation-in-the-portal"></a>验证在门户中创建的计算机

1. 访问门户并查看您的虚拟机。
2. 验证已创建 **我的VM**。
3. 评价VM设置。 
4. 请注意，此Windows设备已连入新虚拟网和子网 
5. 请注意，该命令启动了本机。 
6. 此时，您可使用门户或管理员进行更改。 

## <a name="connect-to-the-virtual-machine"></a>连接到虚拟机

1. 检索计算机的公用 IP 地址。

```
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```
2. 从本机创建 RDP 会话。 将 IP 地址替换为你的 VM 的公共 IP 地址。 该命令在 cmd 窗口中运行。

```
mstsc /v:publicIpAddress
```

3. 出现提示时，请提供计算机的登录凭据。 请务必 **使用不同的帐户**。 键入 localhost \username 作为用户名，输入你为虚拟机创建的密码，然后选择“确定”。 你可能会在登录过程中收到证书警告。 选择 **是** 或 **继续** 创建连接
4. 完成后，关闭到 VM 的 RDP 连接。
5. 清理资源。 这将需要几分钟时间，然后删除资源组和虚拟机。

```
Remove-AzResourceGroup -Name myResourceGroup 
```