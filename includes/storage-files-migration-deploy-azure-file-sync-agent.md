---
title: Plik dyrektywy include
description: Plik dyrektywy include
services: storage
author: fauhse
ms.service: storage
ms.topic: include
ms.date: 2/20/2020
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: 7c1ea3fbf8e5cef4b1e8f8a2b0963fdd66584f5a
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/31/2021
ms.locfileid: "106075610"
---
W tej sekcji instalujesz agenta Azure File Sync w wystąpieniu systemu Windows Server.

[Przewodnik wdrażania](../articles/storage/files/storage-sync-files-deployment-guide.md) pokazuje, że należy wyłączyć **konfigurację zwiększonych zabezpieczeń programu Internet Explorer**. Ta miara zabezpieczeń nie ma zastosowania do Azure File Sync. Wyłączenie tej opcji umożliwia uwierzytelnianie na platformie Azure bez żadnych problemów.

Otwórz program PowerShell i zainstaluj wymagane moduły programu PowerShell, używając następujących poleceń. Po wyświetleniu monitu upewnij się, że zainstalowano pełen moduł i dostawcę NuGet.

```powershell
Install-Module -Name Az -AllowClobber
Install-Module -Name Az.StorageSync
```

Jeśli masz problemy z nawiązaniem połączenia z Internetem z serwera, możesz teraz rozwiązać ten problem. Azure File Sync korzysta z dowolnego dostępnego połączenia sieciowego z Internetem. Obsługiwane jest również wymaganie, aby serwer proxy mógł nawiązać połączenie z Internetem. Można skonfigurować serwer proxy dla całego komputera lub określić serwer proxy, który będzie używany przez Azure File Sync podczas instalacji agenta.

W przypadku konfigurowania serwera proxy oznacza to, że należy otworzyć zapory dla tego serwera, takie podejście może być akceptowane. Po zakończeniu instalacji serwera po zakończeniu rejestracji serwera zostanie wyświetlony raport o łączności sieciowej z dokładnymi adresami URL punktów końcowych na platformie Azure, które Azure File Sync muszą komunikować się z wybranym regionem. Raport informuje również o przyczynach niepotrzebnej komunikacji. Możesz użyć raportu, aby zablokować zapory otaczające ten serwer do określonych adresów URL.

Można również zastosować bardziej rozpuszczalną metodę, w której nie można otwierać zapór, ale zamiast tego ograniczyć serwer do komunikacji z przestrzeniami nazw DNS wyższego poziomu. Aby uzyskać więcej informacji, zobacz [Azure File Sync ustawień serwera proxy i zapory](../articles/storage/files/storage-sync-files-firewall-and-proxy.md). Postępuj zgodnie z najlepszymi rozwiązaniami dotyczącymi sieci.

Po zakończeniu działania Kreatora instalacji serwera zostanie otwarty Kreator rejestracji serwera. Zarejestruj serwer w ramach zasobu platformy Azure usługi synchronizacji magazynu z wcześniejszych wersji.

Te kroki są opisane bardziej szczegółowo w przewodniku wdrażania, który obejmuje moduły programu PowerShell, które należy zainstalować najpierw: [Azure File Sync instalacji agenta](../articles/storage/files/storage-sync-files-deployment-guide.md).

Użyj najnowszego agenta. Można go pobrać z centrum pobierania Microsoft: [Agent Azure File Sync](https://aka.ms/AFS/agent "Pobieranie agenta Azure File Sync").

Po pomyślnej instalacji i rejestracji serwera można sprawdzić, czy ten krok został pomyślnie ukończony. Przejdź do zasobu usługi synchronizacji magazynu w Azure Portal. W menu po lewej stronie przejdź do pozycji **zarejestrowane serwery**. Zobaczysz serwer w tym miejscu.
