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
2. Create an RDP session from your local machine. Replace the IP address with the public IP address of your VM. This command runs from a cmd window.

```
mstsc /v:publicIpAddress
```

3. When prompted, provide your login credentials for the machine. Be sure to <bpt id="p1">**</bpt>Use a different account<ept id="p1">**</ept>. Type the username as localhost\username, enter password you created for the virtual machine, and then select <bpt id="p1">**</bpt>OK<ept id="p1">**</ept>. You may receive a certificate warning during the sign-in process. Select <bpt id="p1">**</bpt>Yes<ept id="p1">**</ept> or <bpt id="p2">**</bpt>Continue<ept id="p2">**</ept> to create the connection
4. 完成后，关闭到 VM 的 RDP 连接。
5. Clean up your resources. This will take a few minutes and remove the resource group and virtual machine.

```
Remove-AzResourceGroup -Name myResourceGroup 
```