---
title: Konfigurowanie usługi Azure IoT Hub w celu wdrożenia aktualizacji za pośrednictwem środowiska AIR
description: Dowiedz się, jak skonfigurować usługę Azure IoT Hub w celu wdrażania aktualizacji na platformie Azure Percept DK
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 03/30/2021
ms.custom: template-how-to
ms.openlocfilehash: 5890ca90b0ad0dcb3d5141e62e986475fd386959
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/31/2021
ms.locfileid: "106064124"
---
# <a name="how-to-set-up-azure-iot-hub-to-deploy-over-the-air-updates-to-your-azure-percept-dk"></a>Jak skonfigurować usługę Azure IoT Hub w celu wdrożenia aktualizacji w środowisku AIR na platformie Azure Percept DK

Dbaj o bezpieczeństwo i aktualność systemu Azure Percept, korzystając z aktualizacji opartych na środowisku AIR. W kilku prostych krokach można skonfigurować środowisko platformy Azure z aktualizacją urządzenia dla IoT Hub i wdrożyć najnowsze aktualizacje na platformie Azure Percept.

## <a name="prerequisites"></a>Wymagania wstępne

- Azure Percept DK (Devkit)
- [Subskrypcja platformy Azure](https://azure.microsoft.com/free/)
- [Środowisko instalacji usługi Azure PERCEPT DK](./quickstart-percept-dk-set-up.md): nawiązano połączenie zestawu deweloperskiego z siecią Wi-Fi, utworzono IoT Hub i połączono swój zestaw deweloperski z IoT Hub

## <a name="create-a-device-update-account"></a>Utwórz konto aktualizacji urządzenia

1. Przejdź do [Azure Portal](https://portal.azure.com) i zaloguj się przy użyciu konta platformy Azure używanego z usługą Azure Percept.

1. Na pasku wyszukiwania w górnej części strony wprowadź **aktualizację urządzenia dla IoT Hub**.

1. Wybierz pozycję **Aktualizacja urządzenia dla IoT Hub** , gdy zostanie ona wyświetlona na pasku wyszukiwania.

1. Kliknij przycisk **+ Dodaj** w lewej górnej części strony.

1. Wybierz **subskrypcję platformy Azure** i **grupę zasobów** skojarzoną z urządzeniem Percept platformy Azure i jej IoT Hub.

1. Określ **nazwę** i **lokalizację** konta aktualizacji urządzenia.

1. Przejrzyj szczegóły i wybierz pozycję **Przegląd + Utwórz**.

1. Po zakończeniu wdrażania kliknij pozycję **Przejdź do zasobu**.

## <a name="create-a-device-update-instance"></a>Tworzenie wystąpienia aktualizacji urządzenia

1. W obszarze aktualizacja urządzenia dla IoT Hub zasobu kliknij pozycję **wystąpienia** w obszarze **Zarządzanie wystąpieniami**.

1. Kliknij pozycję **+ Utwórz**, określ nazwę wystąpienia i wybierz IoT Hub skojarzoną z urządzeniem Azure Percept. Może to potrwać kilka minut.

1. Kliknij pozycję **Utwórz**.

## <a name="configure-iot-hub"></a>Konfigurowanie IoT Hub

1. Na stronie **wystąpienia** zarządzania wystąpieniami poczekaj, aż wystąpienie aktualizacji urządzenia przejdzie do stanu " **powodzenie** ". Kliknij ikonę **odświeżania** , aby zaktualizować stan.

1. Wybierz utworzone wystąpienie, a następnie kliknij pozycję **konfiguruj IoT Hub**. W lewym okienku wybierz pozycję **Zgadzam się wprowadzić te zmiany** i kliknij przycisk **Aktualizuj**.

1. Poczekaj na pomyślne zakończenie procesu.

## <a name="configure-access-control-roles"></a>Konfigurowanie ról kontroli dostępu

Ostatni krok umożliwi przyznanie użytkownikom uprawnień do publikowania i wdrażania aktualizacji.

1. W obszarze aktualizacja urządzenia dla IoT Hub zasobu kliknij pozycję **Kontrola dostępu (IAM)**.

1. Kliknij pozycję **+ Dodaj** , a następnie wybierz pozycję **Dodaj przypisanie roli**.

1. W obszarze **rola** wybierz pozycję **administrator aktualizacji urządzenia**. Aby **przypisać dostęp do** wybranych **zasad użytkownika, grupy lub usługi**. W polu **Wybierz** wybierz swoje konto lub konto osoby, która będzie wdrażać aktualizacje. Kliknij pozycję **Zapisz**.

> [!TIP]
> Jeśli chcesz udzielić większej liczbie osób w organizacji, możesz powtórzyć ten krok i wprowadzić każdego z tych użytkowników jako **administratora aktualizacji urządzenia**.

## <a name="next-steps"></a>Następne kroki

Teraz możesz przystąpić do [aktualizowania zestawu Azure Percept dev Kit za pośrednictwem środowiska AIR](./how-to-update-over-the-air.md) przy użyciu aktualizacji urządzenia dla IoT Hub.