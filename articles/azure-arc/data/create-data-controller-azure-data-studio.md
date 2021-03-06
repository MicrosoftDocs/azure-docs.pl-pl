---
title: Utwórz kontroler danych w Azure Data Studio
description: Utwórz kontroler danych w Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 04/07/2021
ms.topic: how-to
ms.openlocfilehash: 6e61c8819e7ccd868ec92458cff69c37f9277d80
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029628"
---
# <a name="create-data-controller-in-azure-data-studio"></a>Utwórz kontroler danych w Azure Data Studio

Kontroler danych można utworzyć przy użyciu Azure Data Studio za pośrednictwem Kreatora wdrażania i notesów.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

W bieżącym momencie można utworzyć kontroler danych przy użyciu metody opisanej w tym artykule.

## <a name="prerequisites"></a>Wymagania wstępne

- Musisz mieć dostęp do klastra Kubernetes i mieć skonfigurowany plik kubeconfig, aby wskazywał klaster Kubernetes, który ma zostać wdrożony.
- Należy [zainstalować narzędzia klienckie](install-client-tools.md) , w tym **Azure Data Studio** rozszerzenia Azure Data Studio o nazwie **Azure Arc** i **[!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]** .
- Musisz zalogować się do platformy Azure w Azure Data Studio.  W tym celu: wpisz CTRL/Command + SHIFT + P, aby otworzyć okno tekstowe polecenia i wpisz **Azure**.  Wybierz pozycję **Azure: Zaloguj się**.   W panelu, który zostanie wyświetlony, kliknij ikonę + w prawym górnym rogu, aby dodać konto platformy Azure.

## <a name="use-the-deployment-wizard-to-create-azure-arc-data-controller"></a>Tworzenie kontrolera danych usługi Azure ARC przy użyciu Kreatora wdrażania

Wykonaj następujące kroki, aby utworzyć kontroler danych usługi Azure ARC przy użyciu Kreatora wdrażania.

1. W Azure Data Studio kliknij kartę połączenia na lewym pasku nawigacyjnym.
1. Kliknij przycisk **...** w górnej części panelu połączenia, a następnie wybierz pozycję **nowe wdrożenie...**
1. W Kreatorze nowego wdrożenia wybierz pozycję **Azure Arc Data Controller**, a następnie kliknij przycisk **Wybierz** u dołu.
1. Upewnij się, że wstępnie wymagane narzędzia są dostępne i są zgodne z wymaganymi wersjami. **Kliknij przycisk Dalej**.
1. Użyj domyślnego pliku kubeconfig lub wybierz inny.  Kliknij przycisk **Dalej**.
1. Wybierz kontekst klastra Kubernetes. Kliknij przycisk **Dalej**.
1. Wybierz profil konfiguracji wdrożenia w zależności od docelowego klastra Kubernetes. **Kliknij przycisk Dalej**.
1. Jeśli używasz platformy Azure Red Hat OpenShift lub Red Hat OpenShift Container platform, Zastosuj ograniczenia kontekstu zabezpieczeń. Postępuj zgodnie z instrukcjami w obszarze [Zastosuj ograniczenie kontekstu zabezpieczeń dla usług danych z obsługą usługi Azure Arc w systemie OpenShift](how-to-apply-security-context-constraint.md).

   >[!IMPORTANT]
   >Na platformie Azure Red Hat OpenShift lub Red Hat OpenShift kontenera należy zastosować ograniczenie kontekstu zabezpieczeń przed utworzeniem kontrolera danych.

1. Wybierz żądaną subskrypcję i grupę zasobów.
1. Wybierz lokalizację platformy Azure.
   
   Wybrana lokalizacja platformy Azure to lokalizacja na platformie Azure, w której będą przechowywane *metadane* dotyczące kontrolera danych i wystąpień bazy danych, którymi zarządza. Wystąpienia kontrolera danych i bazy danych zostaną utworzone w klastrze Kubernetes wszędzie tam, gdzie to możliwe.
   
   Po zakończeniu kliknij przycisk **dalej**.

1. Wprowadź nazwę kontrolera danych i dla przestrzeni nazw, w której zostanie utworzony kontroler danych.

    Kontroler danych i nazwa przestrzeni nazw będą używane do tworzenia zasobów niestandardowych w klastrze Kubernetes, aby musiały one być zgodne z [konwencjami nazewnictwa Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names).
    
    Jeśli przestrzeń nazw już istnieje, zostanie użyta, jeśli przestrzeń nazw nie zawiera jeszcze innych Kubernetes obiektów — pory itd.  Jeśli przestrzeń nazw nie istnieje, zostanie podjęta próba utworzenia przestrzeni nazw.  Tworzenie przestrzeni nazw w klastrze Kubernetes wymaga uprawnień administratora klastra Kubernetes.  Jeśli nie masz uprawnień administratora klastra Kubernetes, poproszenie administratora klastra Kubernetes o wykonanie kilku pierwszych kroków z artykułu [Tworzenie kontrolera danych przy użyciu narzędzia Kubernetes-Native Tools](./create-data-controller-using-kubernetes-native-tools.md) , które są wymagane do wykonania przez administratora Kubernetes przed ukończeniem tego kreatora.


1. Wybierz klasę magazynu, w której zostanie wdrożony kontroler danych. 
1.  Wprowadź nazwę użytkownika i hasło, a następnie potwierdź hasło dla konta użytkownika Administrator kontrolera danych. Kliknij przycisk **Dalej**.

1. Zapoznaj się z konfiguracją wdrożenia.
1. Kliknij przycisk **Wdróż** , aby wdrożyć żądaną konfigurację lub **skrypt do notesu** w celu przejrzenia instrukcji wdrożenia lub wprowadzić wszelkie niezbędne zmiany, takie jak nazwy klas magazynu lub typy usług. Kliknij pozycję **Uruchom wszystkie** w górnej części notesu.

## <a name="monitoring-the-creation-status"></a>Monitorowanie stanu tworzenia

Tworzenie kontrolera potrwa kilka minut. Postęp można monitorować w innym oknie terminalu przy użyciu następujących poleceń:

> [!NOTE]
>  W przykładowych poleceniach przyjęto założenie, że utworzono kontroler danych i przestrzeń nazw Kubernetes o nazwie "ARC".  Jeśli użyto innej nazwy obszaru nazw/kontrolera danych, można zastąpić "ARC" nazwą.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Możesz również sprawdzić stan tworzenia dowolnego określonego obszaru pod, uruchamiając polecenie podobne do poniższego.  Jest to szczególnie przydatne w przypadku rozwiązywania problemów.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Rozwiązywanie problemów z tworzeniem

W przypadku wystąpienia problemów z tworzeniem należy zapoznać się z [przewodnikiem rozwiązywania problemów](troubleshoot-guide.md).
