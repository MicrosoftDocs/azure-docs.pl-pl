---
title: Tworzenie zestawu skalowania na użyciu wyspecjalizowanej wersji obrazu przy użyciu interfejsu wiersza polecenia platformy Azure
description: Utwórz zestaw skalowania przy użyciu wyspecjalizowanej wersji obrazu w Shared Image Gallery interfejsie wiersza polecenia platformy Azure.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5fc88c00d548c0a034984976557d316fdac7620f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/20/2021
ms.locfileid: "107792349"
---
# <a name="create-a-scale-set-using-a-specialized-image-version-with-the-azure-cli"></a>Tworzenie zestawu skalowania przy użyciu wyspecjalizowanej wersji obrazu za pomocą interfejsu wiersza polecenia platformy Azure

Utwórz zestaw skalowania na pomocą [wyspecjalizowanej wersji obrazu](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) przechowywanej w Shared Image Gallery. Jeśli chcesz utworzyć zestaw skalowania przy użyciu uogólnionych wersji obrazu, zobacz Create a scale set from a generalized image (Tworzenie zestawu skalowania na [pomocą uogólnionych obrazów).](instance-generalized-image-version-cli.md)

Jeśli zdecydujesz się zainstalować interfejs wiersza polecenia i używać go lokalnie, ten samouczek wymaga interfejsu wiersza polecenia platformy Azure w wersji 2.4.0 lub nowszej. Uruchom polecenie `az --version`, aby dowiedzieć się, jaka wersja jest używana. Jeśli konieczna będzie instalacja lub uaktualnienie, zobacz [Instalowanie interfejsu wiersza polecenia platformy Azure]( /cli/azure/install-azure-cli).

W razie potrzeby w tym przykładzie zastąp nazwy zasobów. 

Wyświetl listę definicji obrazów w galerii przy użyciu [narzędzia az sig image-definition list,](/cli/azure/sig/image-definition#az_sig_image_definition_list) aby wyświetlić nazwę i identyfikator definicji.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

Utwórz zestaw skalowania przy [`az vmss create`](/cli/azure/vmss#az_vmss_create) użyciu `--specialized` parametru , aby wskazać, że obraz jest wyspecjalizowanym obrazem.

Użyj identyfikatora definicji obrazu dla , aby utworzyć wystąpienia zestawu skalowania z `--image` najnowszej dostępnej wersji obrazu. Wystąpienia zestawu skalowania można również utworzyć z określonej wersji, podając identyfikator wersji obrazu dla `--image` . Należy pamiętać, że użycie określonej wersji obrazu oznacza, że automatyzacja może się nie powieść, jeśli ta wersja obrazu nie jest dostępna, ponieważ została usunięta lub usunięta z regionu. Zalecamy użycie identyfikatora definicji obrazu do utworzenia nowej maszyny wirtualnej, chyba że jest wymagana konk.

W tym przykładzie tworzymy wystąpienia z najnowszej wersji obrazu *myImageDefinition.*

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --specialized
```


## <a name="next-steps"></a>Następne kroki
[Usługa Azure Image Builder (wersja zapoznawcza)](../virtual-machines/image-builder-overview.md) może pomóc zautomatyzować tworzenie wersji obrazu. Można jej nawet użyć do zaktualizowania i utworzenia nowej wersji obrazu z [istniejącej wersji obrazu.](../virtual-machines/linux/image-builder-gallery-update-image-version.md) 

Możesz również utworzyć zasób Shared Image Gallery za pomocą szablonów. Dostępnych jest kilka szablonów szybkiego startu platformy Azure: 

- [Tworzenie galerii obrazów udostępnionych](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Tworzenie definicji obrazu w usłudze Shared Image Gallery](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Tworzenie wersji obrazu w usłudze Shared Image Gallery](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)