---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/21/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 067b099bee447c559d7196e0f7e72bcaa1d00bf5
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/22/2021
ms.locfileid: "107867760"
---
|Nazwa<br /><sub>(Azure Portal)</sub> |Opis |Efekt(y) |Wersja<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Zalecenia dotyczące adaptacyjnego wzmacniania zabezpieczeń sieci powinny być stosowane na maszynach wirtualnych skierowanych do Internetu](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Azure Security Center analizuje wzorce ruchu maszyn wirtualnych skierowanych do Internetu i udostępnia zalecenia dotyczące reguł sieciowej grupy zabezpieczeń, które zmniejszają ryzyko ataku |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Maszyny wirtualne z Internetem powinny być chronione za pomocą sieciowych grup zabezpieczeń](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Chroń maszyny wirtualne przed potencjalnymi zagrożeniami, ograniczając dostęp do nich za pomocą sieciowych grup zabezpieczeń. Dowiedz się więcej na temat kontrolowania ruchu za pomocą sieciowych sieciowych sieciowych [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc) |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[Przekazywanie ip na maszynie wirtualnej powinno być wyłączone](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbd352bd5-2853-4985-bf0d-73806b4a5744) |Włączenie przekazywania ip na kartach sieciowych maszyny wirtualnej umożliwia maszynie odbieranie ruchu skierowanego do innych miejsc docelowych. Przekazywanie ip jest rzadko wymagane (np. w przypadku używania maszyny wirtualnej jako wirtualnego urządzenia sieciowego), dlatego powinno to zostać przejrzane przez zespół ds. zabezpieczeń sieci. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_IPForwardingOnVirtualMachines_Audit.json) |
|[Porty zarządzania maszyn wirtualnych powinny być chronione za pomocą kontroli dostępu do sieci just in time](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Możliwy dostęp do sieci just in time (JIT) będzie monitorowany przez Azure Security Center zgodnie z zaleceniami |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |
|[Porty zarządzania powinny być zamknięte na maszynach wirtualnych](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22730e10-96f6-4aac-ad84-9383d35b5917) |Otwarte porty zdalnego zarządzania narażają maszynę wirtualną na wysoki poziom ryzyka związanego z atakami internetowymi. Te ataki usiłują atak siłowy na poświadczenia w celu uzyskania dostępu administratora do maszyny. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OpenManagementPortsOnVirtualMachines_Audit.json) |
