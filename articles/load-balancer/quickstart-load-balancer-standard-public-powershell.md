---
title: 'Szybki start: tworzenie publicznego równoważenia obciążenia — Azure PowerShell'
titleSuffix: Azure Load Balancer
description: W tym przewodniku Szybki start pokazano, jak utworzyć usługę równoważenia obciążenia przy użyciu Azure PowerShell
services: load-balancer
documentationcenter: na
author: asudbring
ms.author: allensu
manager: KumudD
ms.date: 11/22/2020
ms.assetid: ''
ms.topic: quickstart
ms.service: load-balancer
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom:
- mode-api
ms.openlocfilehash: 0ddaf0eede59053cd8022fef24d37a37c6d7db5a
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529560"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-azure-powershell"></a>Szybki start: tworzenie publicznego równoważenia obciążenia w celu równoważenia obciążenia maszyn wirtualnych przy użyciu Azure PowerShell

Rozpoczynanie pracy z Azure Load Balancer przy użyciu Azure PowerShell do utworzenia publicznego równoważenia obciążenia i trzech maszyn wirtualnych.

## <a name="prerequisites"></a>Wymagania wstępne

- Konto platformy Azure z aktywną subskrypcją. [Utwórz bezpłatne konto.](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Azure PowerShell zainstalowane lokalnie lub Azure Cloud Shell

Jeśli postanowisz zainstalować program PowerShell i używać go lokalnie, ten artykuł wymaga modułu Azure PowerShell w wersji 5.4.1 lub nowszej. Uruchom polecenie `Get-Module -ListAvailable Az`, aby dowiedzieć się, jaka wersja jest zainstalowana. Jeśli konieczne będzie uaktualnienie, zobacz [Instalowanie modułu Azure PowerShell](/powershell/azure/install-Az-ps). Jeśli używasz programu PowerShell lokalnie, musisz też uruchomić polecenie `Connect-AzAccount`, aby utworzyć połączenie z platformą Azure.

## <a name="create-a-resource-group"></a>Tworzenie grupy zasobów

Grupa zasobów platformy Azure to logiczny kontener przeznaczony do wdrażania zasobów platformy Azure i zarządzania nimi.

Utwórz grupę zasobów za [pomocą new-AzResourceGroup:](/powershell/module/az.resources/new-azresourcegroup)

```azurepowershell-interactive
New-AzResourceGroup -Name 'CreatePubLBQS-rg' -Location 'eastus'

```
---

# <a name="standard-sku"></a>[**Standardowy SKU**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>W przypadku obciążeń produkcyjnych zaleca się użycie standardowego równoważenia obciążenia SKU. Aby uzyskać więcej informacji na temat jednostki SKU, **[zobacz Azure Load Balancer SKU](skus.md)**.

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram.png" alt-text="Zasoby standardowego równoważenia obciążenia utworzone na podstawie przewodnika Szybki start." border="false":::

## <a name="create-a-public-ip-address---standard"></a>Tworzenie publicznego adresu IP — Standardowa

Użyj [new-AzPublicIpAddress,](/powershell/module/az.network/new-azpublicipaddress) aby utworzyć publiczny adres IP.

```azurepowershell-interactive
$publicip = @{
    Name = 'myPublicIP'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'static'
    Zone = 1,2,3
}
New-AzPublicIpAddress @publicip

```

Aby utworzyć strefowy publiczny adres IP w strefie 1, użyj następującego polecenia:

```azurepowershell-interactive
$publicip = @{
    Name = 'myPublicIP'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'static'
    Zone = 1
}
New-AzPublicIpAddress @publicip

```

## <a name="create-standard-load-balancer"></a>Tworzenie standardowego równoważenia obciążenia

W tej sekcji opisano szczegółowo procedurę tworzenia i konfigurowania następujących składników modułu równoważenia obciążenia:

* Utwórz adres IP frontonia za [pomocą polecenie New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) dla puli adresów IP frontonia. Ten adres IP odbiera ruch przychodzący do usługi równoważenia obciążenia

* Utwórz pulę adresów zaplecza za pomocą polecenie [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) dla ruchu wysyłanego z frontony usługi równoważenia obciążenia. W tej puli są wdrażane maszyny wirtualne zaplecza.

* Utwórz sondę kondycji za pomocą polecenie [Add-AzLoadBalancerProbeConfig,](/powershell/module/az.network/add-azloadbalancerprobeconfig) które określa kondycję wystąpień maszyn wirtualnych zaplecza.

* Utwórz regułę równoważenia obciążenia za pomocą [poleceniem Add-AzLoadBalancerRuleConfig,](/powershell/module/az.network/add-azloadbalancerruleconfig) która definiuje sposób dystrybucji ruchu do maszyn wirtualnych.

* Utwórz publiczny równoważenie obciążenia za pomocą [new-AzLoadBalancer.](/powershell/module/az.network/new-azloadbalancer)


```azurepowershell-interactive
## Place public IP created in previous steps into variable. ##
$publicIp = Get-AzPublicIpAddress -Name 'myPublicIP' -ResourceGroupName 'CreatePubLBQS-rg'

## Create load balancer frontend configuration and place in variable. ##
$feip = New-AzLoadBalancerFrontendIpConfig -Name 'myFrontEnd' -PublicIpAddress $publicIp

## Create backend address pool configuration and place in variable. ##
$bepool = New-AzLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'

## Create the health probe and place in variable. ##
$probe = @{
    Name = 'myHealthProbe'
    Protocol = 'http'
    Port = '80'
    IntervalInSeconds = '360'
    ProbeCount = '5'
    RequestPath = '/'
}
$healthprobe = New-AzLoadBalancerProbeConfig @probe

## Create the load balancer rule and place in variable. ##
$lbrule = @{
    Name = 'myHTTPRule'
    Protocol = 'tcp'
    FrontendPort = '80'
    BackendPort = '80'
    IdleTimeoutInMinutes = '15'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
}
$rule = New-AzLoadBalancerRuleConfig @lbrule -EnableTcpReset -DisableOutboundSNAT

## Create the load balancer resource. ##
$loadbalancer = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myLoadBalancer'
    Location = 'eastus'
    Sku = 'Standard'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
    LoadBalancingRule = $rule
    Probe = $healthprobe
}
New-AzLoadBalancer @loadbalancer

```

## <a name="configure-virtual-network---standard"></a>Konfigurowanie sieci wirtualnej — standardowa

Przed wdrożeniem maszyn wirtualnych i przetestowanie usługi Load Balancer utwórz zasoby sieci wirtualnej.

Utwórz sieć wirtualną dla maszyn wirtualnych zaplecza.

Utwórz sieciową grupę zabezpieczeń, aby zdefiniować połączenia przychodzące do sieci wirtualnej.

### <a name="create-virtual-network-network-security-group-and-bastion-host"></a>Tworzenie sieci wirtualnej, sieciowej grupy zabezpieczeń i hosta bastionu

* Utwórz sieć wirtualną przy użyciu polecenia [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

* Utwórz regułę sieciowej grupy zabezpieczeń za [pomocą polecenie New-AzNetworkSecurityRuleConfig.](/powershell/module/az.network/new-aznetworksecurityruleconfig)

* Utwórz hosta Azure Bastion pomocą [new-AzBastion.](/powershell/module/az.network/new-azbastion)

* Utwórz sieciową grupę zabezpieczeń przy użyciu polecenia [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'myBackendSubnet'
    AddressPrefix = '10.1.0.0/24'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create Azure Bastion subnet. ##
$bastsubnet = @{
    Name = 'AzureBastionSubnet' 
    AddressPrefix = '10.1.1.0/24'
}
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig @bastsubnet

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig,$bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @net

## Create public IP address for bastion host. ##
$ip = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @ip

## Create bastion host ##
$bastion = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @bastion -AsJob

## Create rule for network security group and place in variable. ##
$nsgrule = @{
    Name = 'myNSGRuleHTTP'
    Description = 'Allow HTTP'
    Protocol = '*'
    SourcePortRange = '*'
    DestinationPortRange = '80'
    SourceAddressPrefix = 'Internet'
    DestinationAddressPrefix = '*'
    Access = 'Allow'
    Priority = '2000'
    Direction = 'Inbound'
}
$rule1 = New-AzNetworkSecurityRuleConfig @nsgrule

## Create network security group ##
$nsg = @{
    Name = 'myNSG'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    SecurityRules = $rule1
}
New-AzNetworkSecurityGroup @nsg

```

## <a name="create-virtual-machines---standard"></a>Tworzenie maszyn wirtualnych — standardowa

W tej sekcji utworzysz trzy maszyny wirtualne dla puli zaplecza usługi równoważenia obciążenia.

* Utwórz trzy interfejsy sieciowe za [pomocą new-AzNetworkInterface.](/powershell/module/az.network/new-aznetworkinterface)

* Ustaw nazwę użytkownika i hasło administratora dla maszyn wirtualnych za pomocą [get-credential.](/powershell/module/microsoft.powershell.security/get-credential)

* Utwórz maszyny wirtualne za pomocą:
    * [New-AzVM](/powershell/module/az.compute/new-azvm)
    * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
    * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
    * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
    * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)

```azurepowershell-interactive
# Set the administrator and password for the VMs. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreatePubLBQS-rg'

## Place the load balancer into a variable. ##
$lb = @{
    Name = 'myLoadBalancer'
    ResourceGroupName = 'CreatePubLBQS-rg'
}
$bepool = Get-AzLoadBalancer @lb  | Get-AzLoadBalancerBackendAddressPoolConfig

## Place the network security group into a variable. ##
$nsg = Get-AzNetworkSecurityGroup -Name 'myNSG' -ResourceGroupName 'CreatePubLBQS-rg'

## For loop with variable to create virtual machines for load balancer backend pool. ##
for ($i=1; $i -le 3; $i++)
{
## Command to create network interface for VMs ##
$nic = @{
    Name = "myNicVM$i"
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    NetworkSecurityGroup = $nsg
    LoadBalancerBackendAddressPool = $bepool
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myVM$i"
    VMSize = 'Standard_DS1_v2'  
}
$vmos = @{
    ComputerName = "myVM$i"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    VM = $vmConfig
    Zone = "$i"
}
New-AzVM @vm -AsJob
}

```

Wdrożenia maszyn wirtualnych i hosta bastionu są przesyłane jako zadania programu PowerShell. Aby wyświetlić stan zadań, użyj [get-job:](/powershell/module/microsoft.powershell.core/get-job)

```azurepowershell-interactive
Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzBastion
2      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
3      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
4      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-outbound-rule-configuration"></a>Tworzenie konfiguracji reguły ruchu wychodzącego
Reguły ruchu wychodzącego usługi równoważenia obciążenia konfigurują translatora źródłowych adresów sieciowych (SNAT) ruchu wychodzącego dla maszyn wirtualnych w puli zaplecza. 

Aby uzyskać więcej informacji na temat połączeń wychodzących, zobacz [Połączenia wychodzące na platformie Azure.](load-balancer-outbound-connections.md)

### <a name="create-outbound-public-ip-address"></a>Tworzenie publicznego adresu IP ruchu wychodzącego

Użyj [new-AzPublicIpAddress,](/powershell/module/az.network/new-azpublicipaddress) aby utworzyć standardowy strefowo nadmiarowy publiczny adres IP o nazwie **myPublicIPOutbound.**

```azurepowershell-interactive
$publicipout = @{
    Name = 'myPublicIPOutbound'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'static'
    Zone = 1,2,3
}
New-AzPublicIpAddress @publicipout

```

Aby utworzyć strefowy publiczny adres IP w strefie 1, użyj następującego polecenia:

```azurepowershell-interactive
$publicipout = @{
    Name = 'myPublicIPOutbound'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'static'
    Zone = 1
}
New-AzPublicIpAddress @publicipout

```

### <a name="create-outbound-configuration"></a>Tworzenie konfiguracji ruchu wychodzącego

* Utwórz nową konfigurację adresu IP frontonia za pomocą [polecenie Add-AzLoadBalancerFrontendIpConfig.](/powershell/module/az.network/add-azloadbalancerfrontendipconfig)

* Utwórz nową pulę adresów zaplecza ruchu wychodzącego za pomocą [polecenie Add-AzLoadBalancerBackendAddressPoolConfig.](/powershell/module/az.network/add-azloadbalancerbackendaddresspoolconfig) 

* Zastosuj adres IP puli i frontonia do usługi równoważenia obciążenia za pomocą [zestawu AzLoadBalancer.](/powershell/module/az.network/set-azloadbalancer)
*  Utwórz nową regułę ruchu wychodzącego dla puli zaplecza ruchu wychodzącego za pomocą [polecenie Add-AzLoadBalancerOutboundRuleConfig.](/powershell/module/az.network/new-azloadbalanceroutboundruleconfig) 

```azurepowershell-interactive
## Place public IP created in previous steps into variable. ##
$pubip = @{
    Name = 'myPublicIPOutbound'
    ResourceGroupName = 'CreatePubLBQS-rg'
}
$publicIp = Get-AzPublicIpAddress @pubip

## Get the load balancer configuration ##
$lbc = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myLoadBalancer'
}
$lb = Get-AzLoadBalancer @lbc

## Create the frontend configuration ##
$fe = @{
    Name = 'myFrontEndOutbound'
    PublicIPAddress = $publicIP
}
$lb | Add-AzLoadBalancerFrontendIPConfig @fe | Set-AzLoadBalancer

## Create the outbound backend address pool ##
$be = @{
    Name = 'myBackEndPoolOutbound'
}
$lb | Add-AzLoadBalancerBackendAddressPoolConfig @be | Set-AzLoadBalancer

## Apply the outbound rule configuration to the load balancer. ##
$rule = @{
    Name = 'myOutboundRule'
    AllocatedOutboundPort = '10000'
    Protocol = 'All'
    IdleTimeoutInMinutes = '15'
    FrontendIPConfiguration = $lb.FrontendIpConfigurations[1]
    BackendAddressPool = $lb.BackendAddressPools[1]
}
$lb | Add-AzLoadBalancerOutBoundRuleConfig @rule | Set-AzLoadBalancer

```

### <a name="add-virtual-machines-to-outbound-pool"></a>Dodawanie maszyn wirtualnych do puli ruchu wychodzącego

Dodaj interfejsy sieciowe maszyny wirtualnej do puli ruchu wychodzącego usługi Load Balancer za pomocą [polecenie Add-AzNetworkInterfaceIpConfig:](/powershell/module/az.network/add-aznetworkinterfaceipconfig)

```azurepowershell-interactive
## Get the load balancer configuration ##
$lbc = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myLoadBalancer'
}
$lb = Get-AzLoadBalancer @lbc

# For loop with variable to add virtual machines to backend outbound pool. ##
for ($i=1; $i -le 3; $i++)
{
$nic = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = "myNicVM$i"
}
$nicvm = Get-AzNetworkInterface @nic

## Apply the backend to the network interface ##
$be = @{
    Name = 'ipconfig1'
    LoadBalancerBackendAddressPoolId = $lb.BackendAddressPools[0].id,$lb.BackendAddressPools[1].id
}
$nicvm | Set-AzNetworkInterfaceIpConfig @be | Set-AzNetworkInterface
}

```

# <a name="basic-sku"></a>[**Podstawowy SKU**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>W przypadku obciążeń produkcyjnych zalecany jest standardowy sku równoważenia obciążenia. Aby uzyskać więcej informacji na temat jednostki SKU, **[zobacz Azure Load Balancer SKU](skus.md)**.

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram-basic.png" alt-text="Podstawowe zasoby równoważenia obciążenia utworzone w przewodniku Szybki start." border="false":::

## <a name="create-a-public-ip-address---basic"></a>Tworzenie publicznego adresu IP — podstawowa

Użyj [new-AzPublicIpAddress,](/powershell/module/az.network/new-azpublicipaddress) aby utworzyć publiczny adres IP.

```azurepowershell-interactive
$publicip = @{
    Name = 'myPublicIP'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Basic'
    AllocationMethod = 'static'
}
New-AzPublicIpAddress @publicip

```

## <a name="create-basic-load-balancer"></a>Tworzenie podstawowego równoważenia obciążenia

W tej sekcji opisano szczegółowo procedurę tworzenia i konfigurowania następujących składników modułu równoważenia obciążenia:

* Utwórz adres IP frontendu za [pomocą polecenie New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) dla puli adresów IP frontonia. Ten adres IP odbiera ruch przychodzący do usługi równoważenia obciążenia

* Utwórz pulę adresów zaplecza za pomocą polecenie [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) dla ruchu wysyłanego z frontona usługi równoważenia obciążenia. W tej puli są wdrażane maszyny wirtualne zaplecza.

* Utwórz sondę kondycji za pomocą polecenie [Add-AzLoadBalancerProbeConfig,](/powershell/module/az.network/add-azloadbalancerprobeconfig) które określa kondycję wystąpień maszyn wirtualnych zaplecza.

* Utwórz regułę równoważenia obciążenia za pomocą pliku [Add-AzLoadBalancerRuleConfig,](/powershell/module/az.network/add-azloadbalancerruleconfig) która definiuje sposób dystrybucji ruchu do maszyn wirtualnych.

* Utwórz publiczny równoważenie obciążenia za [pomocą new-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).

```azurepowershell-interactive
## Place public IP created in previous steps into variable. ##
$publicIp = Get-AzPublicIpAddress -Name 'myPublicIP' -ResourceGroupName 'CreatePubLBQS-rg'

## Create load balancer frontend configuration and place in variable. ##
$feip = New-AzLoadBalancerFrontendIpConfig -Name 'myFrontEnd' -PublicIpAddress $publicIp

## Create backend address pool configuration and place in variable. ##
$bepool = New-AzLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'

## Create the health probe and place in variable. ##
$probe = @{
    Name = 'myHealthProbe'
    Protocol = 'http'
    Port = '80'
    IntervalInSeconds = '360'
    ProbeCount = '5'
    RequestPath = '/'
}
$healthprobe = New-AzLoadBalancerProbeConfig @probe

## Create the load balancer rule and place in variable. ##
$lbrule = @{
    Name = 'myHTTPRule'
    Protocol = 'tcp'
    FrontendPort = '80'
    BackendPort = '80'
    IdleTimeoutInMinutes = '15'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
}
$rule = New-AzLoadBalancerRuleConfig @lbrule

## Create the load balancer resource. ##
$loadbalancer = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myLoadBalancer'
    Location = 'eastus'
    Sku = 'Basic'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
    LoadBalancingRule = $rule
    Probe = $healthprobe
}
New-AzLoadBalancer @loadbalancer

```

## <a name="configure-virtual-network---basic"></a>Konfigurowanie sieci wirtualnej — podstawowa

Przed wdrożeniem maszyn wirtualnych i testowaniem usługi równoważenia obciążenia utwórz zasoby sieci wirtualnej.

Utwórz sieć wirtualną dla maszyn wirtualnych zaplecza.

Utwórz sieciową grupę zabezpieczeń, aby zdefiniować połączenia przychodzące do sieci wirtualnej.

### <a name="create-virtual-network-network-security-group-and-bastion-host"></a>Tworzenie sieci wirtualnej, sieciowej grupy zabezpieczeń i hosta bastionu

* Utwórz sieć wirtualną przy użyciu polecenia [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

* Utwórz regułę sieciowej grupy zabezpieczeń za [pomocą polecenie New-AzNetworkSecurityRuleConfig.](/powershell/module/az.network/new-aznetworksecurityruleconfig)

* Utwórz hosta Azure Bastion za [pomocą new-AzBastion](/powershell/module/az.network/new-azbastion).

* Utwórz sieciową grupę zabezpieczeń przy użyciu polecenia [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'myBackendSubnet'
    AddressPrefix = '10.1.0.0/24'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create Azure Bastion subnet. ##
$bastsubnet = @{
    Name = 'AzureBastionSubnet' 
    AddressPrefix = '10.1.1.0/24'
}
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig @bastsubnet

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig,$bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @net

## Create public IP address for bastion host. ##
$ip = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @ip

## Create bastion host ##
$bastion = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @bastion -AsJob

## Create rule for network security group and place in variable. ##
$nsgrule = @{
    Name = 'myNSGRuleHTTP'
    Description = 'Allow HTTP'
    Protocol = '*'
    SourcePortRange = '*'
    DestinationPortRange = '80'
    SourceAddressPrefix = 'Internet'
    DestinationAddressPrefix = '*'
    Access = 'Allow'
    Priority = '2000'
    Direction = 'Inbound'
}
$rule1 = New-AzNetworkSecurityRuleConfig @nsgrule

## Create network security group ##
$nsg = @{
    Name = 'myNSG'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    SecurityRules = $rule1
}
New-AzNetworkSecurityGroup @nsg

```

## <a name="create-virtual-machines---basic"></a>Tworzenie maszyn wirtualnych — podstawowa

W tej sekcji utworzysz maszyny wirtualne dla puli zaplecza usługi równoważenia obciążenia.

* Utwórz trzy interfejsy sieciowe za [pomocą interfejsu New-AzNetworkInterface.](/powershell/module/az.network/new-aznetworkinterface)

* Ustaw nazwę użytkownika i hasło administratora dla maszyn wirtualnych za pomocą [get-credential.](/powershell/module/microsoft.powershell.security/get-credential)

* Użyj [new-AzAvailabilitySet,](/powershell/module/az.compute/new-azvm) aby utworzyć zestaw dostępności dla maszyn wirtualnych.

* Utwórz maszyny wirtualne za pomocą:
    * [New-AzVM](/powershell/module/az.compute/new-azvm)
    * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
    * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
    * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
    * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)

```azurepowershell-interactive
# Set the administrator and password for the VMs. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreatePubLBQS-rg'

## Place the load balancer into a variable. ##
$lb = @{
    Name = 'myLoadBalancer'
    ResourceGroupName = 'CreatePubLBQS-rg'
}
$bepool = Get-AzLoadBalancer @lb  | Get-AzLoadBalancerBackendAddressPoolConfig

## Place the network security group into a variable. ##
$nsg = Get-AzNetworkSecurityGroup -Name 'myNSG' -ResourceGroupName 'CreatePubLBQS-rg'

## Create availability set for the virtual machines. ##
$set = @{
    Name = 'myAvSet'
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Sku = 'Aligned'
    PlatformFaultDomainCount = '2'
    PlatformUpdateDomainCount =  '2'
}
$avs = New-AzAvailabilitySet @set

## For loop with variable to create virtual machines. ##
for ($i=1; $i -le 3; $i++)
{
## Command to create network interface for VMs ##
$nic = @{
    Name = "myNicVM$i"
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    NetworkSecurityGroup = $nsg
    LoadBalancerBackendAddressPool = $bepool
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myVM$i"
    VMSize = 'Standard_DS1_v2'
    AvailabilitySetId = $avs.Id   
}
$vmos = @{
    ComputerName = "myVM$i"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Location = 'eastus'
    VM = $vmConfig
}
New-AzVM @vm -AsJob
}

```

Wdrożenia maszyn wirtualnych i hosta bastionu są przesyłane jako zadania programu PowerShell. Aby wyświetlić stan zadań, użyj [get-job:](/powershell/module/microsoft.powershell.core/get-job)

```azurepowershell-interactive
Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzBastion
2      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
3      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
4      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

---

## <a name="install-iis"></a>Instalowanie usług IIS

Zainstaluj rozszerzenie niestandardowego skryptu przy użyciu polecenia [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension). 

To rozszerzenie uruchamia polecenie `PowerShell Add-WindowsFeature Web-Server`, aby zainstalować serwer internetowy usług IIS, a następnie aktualizuje stronę Default.htm w celu wyświetlenia nazwy hosta maszyny wirtualnej:

> [!IMPORTANT]
> Przed przystąpieniem upewnij się, że wdrożenia maszyn wirtualnych zostały ukończone w poprzednich krokach.  Użyj `Get-Job` , aby sprawdzić stan zadań wdrażania maszyny wirtualnej.

```azurepowershell-interactive
## For loop with variable to install custom script extension on virtual machines. ##
for ($i=1; $i -le 3; $i++)
{
$ext = @{
    Publisher = 'Microsoft.Compute'
    ExtensionType = 'CustomScriptExtension'
    ExtensionName = 'IIS'
    ResourceGroupName = 'CreatePubLBQS-rg'
    VMName = "myVM$i"
    Location = 'eastus'
    TypeHandlerVersion = '1.8'
    SettingString = '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
}
Set-AzVMExtension @ext -AsJob
}
```

Rozszerzenia są wdrażane jako zadania programu PowerShell. Aby wyświetlić stan zadań instalacji, użyj [get-job:](/powershell/module/microsoft.powershell.core/get-job)


```azurepowershell-interactive
Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
8      Long Running O… AzureLongRunni… Running       True            localhost            Set-AzVMExtension
9      Long Running O… AzureLongRunni… Running       True            localhost            Set-AzVMExtension
10     Long Running O… AzureLongRunni… Running       True            localhost            Set-AzVMExtension
```

## <a name="test-the-load-balancer"></a>Testowanie modułu równoważenia obciążenia

Użyj [get-AzPublicIpAddress,](/powershell/module/az.network/get-azpublicipaddress) aby uzyskać publiczny adres IP usługi równoważenia obciążenia:

```azurepowershell-interactive
$ip = @{
    ResourceGroupName = 'CreatePubLBQS-rg'
    Name = 'myPublicIP'
}  
Get-AzPublicIPAddress @ip | select IpAddress

```

Skopiuj publiczny adres IP, a następnie wklej go na pasku adresu przeglądarki. W przeglądarce jest wyświetlana domyślna strona internetowego serwera usług IIS.

   ![Internetowy serwer usług IIS](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)

Aby zobaczyć, jak równoważenie obciążenia dystrybuuje ruch na wszystkich trzech maszynach wirtualnych, można dostosować domyślną stronę serwera sieci Web usług IIS każdej maszyny wirtualnej, a następnie wymusić odświeżenie przeglądarki internetowej z maszyny klienckiej.

## <a name="clean-up-resources"></a>Czyszczenie zasobów

Gdy grupa zasobów, usługa równoważenia obciążenia i pozostałe zasoby nie będą już potrzebne, można je usunąć za pomocą polecenia [Remove-AzResourceGroup.](/powershell/module/az.resources/remove-azresourcegroup)

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'CreatePubLBQS-rg'

```

## <a name="next-steps"></a>Następne kroki

W tym przewodniku Szybki start przyjęto następujące założenia:

* Utworzono publiczny lub standardowy publiczny równoważenie obciążenia
* Dołączone maszyny wirtualne. 
* Skonfigurowano regułę ruchu i sondę kondycji usługi równoważenia obciążenia.
* Przetestowano równoważenie obciążenia.

Aby dowiedzieć się więcej o Azure Load Balancer, przejdź do:
> [!div class="nextstepaction"]
> [Co to jest usługa Azure Load Balancer?](load-balancer-overview.md)
