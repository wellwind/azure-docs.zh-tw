---
title: "在 Azure 中建立執行 SQL&#92;IIS&#92;.NET 堆疊的 VM | Microsoft Docs"
description: "教學課程 - 在 Windows 虛擬機器上安裝 Azure SQL、IIS、.NET 堆疊。"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: ad84d6e8f74fa184ac2359ff7f08e6c8143d419a
ms.sourcegitcommit: 83ea7c4e12fc47b83978a1e9391f8bb808b41f97
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
---
# <a name="install-a-sql92iis92net-stack-in-azure"></a>在 Azure 中安裝 SQL&#92;IIS&#92;.NET 堆疊

在本教學課程中，我們會使用 Azure PowerShell 來安裝 SQL&#92;IIS&#92;.NET 堆疊。 此堆疊是由兩個執行 Windows Server 2016 的 VM 所組成，一個包含 IIS 和 .NET 而另一個則是包含 SQL Server。

> [!div class="checklist"]
> * 建立 VM 
> * 在 VM 上安裝 IIS 和 .NET Core SDK
> * 建立執行 SQL Server 的 VM
> * 安裝 SQL Server 擴充功能

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

本教學課程需要 AzureRM.Compute 模組 4.3.1 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM.Compute` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

## <a name="create-a-iis-vm"></a>建立 IIS VM 

在此範例中，我們會使用 PowerShell Cloud Shell 中的 [New-AzureRMVM](/powershell/module/azurerm.compute/new-azurermvm) Cmdlet 快速建立 Windows Server 2016 VM，然後再安裝 IIS 和 .NET Framework。 IIS 和 SQL VM 會共用資源群組和虛擬網路，因此我們要建立那些名稱的變數。


```azurepowershell-interactive
$vmName = "IISVM"
$vNetName = "myIISSQLvNet"
$resourceGroup = "myIISSQLGroup"
New-AzureRmVm `
    -ResourceGroupName $resourceGroup `
    -Name $vmName `
    -Location "East US" `
    -VirtualNetworkName $vNetName `
    -SubnetName "myIISSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -AddressPrefix 192.168.0.0/16 `
    -PublicIpAddressName "myIISPublicIpAddress" `
    -OpenPorts 80,3389 
```

使用自訂指令碼擴充功能安裝 IIS 和 .NET Framework。

```azurepowershell-interactive
Set-AzureRmVMExtension `
    -ResourceGroupName $resourceGroup `
    -ExtensionName IIS `
    -VMName $vmName `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server,Web-Asp-Net45,NET-Framework-Features"}' `
    -Location EastUS
```

## <a name="create-another-subnet"></a>建立另一個子網路

為 SQL VM 建立第二個子網路。 使用 [Get-AzureRmVirtualNetwork]{/powershell/module/azurerm.network/get-azurermvirtualnetwork} 取得 vNet。

```azurepowershell-interactive
$vNet = Get-AzureRmVirtualNetwork `
   -Name $vNetName `
   -ResourceGroupName $resourceGroup
```

使用 [Add-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/add-azurermvirtualnetworksubnetconfig) 建立子網路的組態。


```azurepowershell-interactive
Add-AzureRmVirtualNetworkSubnetConfig `
   -AddressPrefix 192.168.0.0/24 `
   -Name mySQLSubnet `
   -VirtualNetwork $vNet `
   -ServiceEndpoint Microsoft.Sql
```

使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork) 將 vNet 更新為新的子網路資訊
   
```azurepowershell-interactive   
$vNet | Set-AzureRmVirtualNetwork
```

## <a name="azure-sql-vm"></a>Azure SQL VM

使用 SQL Server 的預先設定 Azure Marketplace 映像來建立 SQL VM。 我們會先建立 VM，然後在 VM 上安裝 SQL Server 擴充功能。 


```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName $resourceGroup `
    -Name "mySQLVM" `
    -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
    -Location eastus `
    -VirtualNetworkName $vNetName `
    -SubnetName "mySQLSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "mySQLPublicIpAddress" `
    -OpenPorts 3389,1401 
```

使用 [Set-AzureRmVMSqlServerExtension](/powershell/module/azurerm.compute/set-azurermvmsqlserverextension) 將 [SQL Server 擴充功能](/sql/virtual-machines-windows-sql-server-agent-extension.md)新增至 SQL VM。

```azurepowershell-interactive
Set-AzureRmVMSqlServerExtension `
   -ResourceGroupName $resourceGroup  `
   -VMName mySQLVM `
   -Name "SQLExtension" `
   -Location "EastUS"
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您會使用 Azure PowerShell 來安裝 SQL&#92;IIS&#92;.NET 堆疊。 您已了解如何︰

> [!div class="checklist"]
> * 建立 VM 
> * 在 VM 上安裝 IIS 和 .NET Core SDK
> * 建立執行 SQL Server 的 VM
> * 安裝 SQL Server 擴充功能

前進到下一個教學課程，以了解如何使用 SSL 憑證保護 IIS Web 伺服器。

> [!div class="nextstepaction"]
> [使用 SSL 憑證保護 IIS Web 伺服器](tutorial-secure-web-server.md)

