---
title: Szyfrowanie danych za pomocą klucza zarządzanego przez klienta — Azure Database for PostgreSQL — pojedynczy serwer
description: Azure Database for PostgreSQL szyfrowanie danych na jednym serwerze przy użyciu klucza zarządzanego przez klienta umożliwia Bring Your Own Key (BYOK) w celu ochrony danych w spoczynku. Umożliwia to również organizacjom rozdzielanie obowiązków związanych z zarządzaniem kluczami i danymi.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: 08c66fe33da78d9b07931c37653b07ba07b22746
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813798"
---
# <a name="azure-database-for-postgresql-single-server-data-encryption-with-a-customer-managed-key"></a>Azure Database for PostgreSQL szyfrowania danych na jednym serwerze przy użyciu klucza zarządzanego przez klienta

Usługa Azure PostgreSQL używa szyfrowania [usługi Azure Storage](../storage/common/storage-service-encryption.md) do domyślnego szyfrowania danych magazynowych przy użyciu kluczy zarządzanych przez firmę Microsoft. W przypadku użytkowników usługi Azure PostgreSQL jest ona bardzo podobna do Transparent Data Encryption (TDE) w innych bazach danych, takich jak SQL Server. Wiele organizacji wymaga pełnej kontroli nad dostępem do danych przy użyciu klucza zarządzanego przez klienta. Szyfrowanie danych za pomocą kluczy zarządzanych przez klienta dla Azure Database for PostgreSQL pojedynczego serwera umożliwia korzystanie z własnego klucza (BYOK) w celu ochrony danych w spoczynku. Umożliwia to również organizacjom rozdzielanie obowiązków związanych z zarządzaniem kluczami i danymi. W przypadku szyfrowania zarządzanego przez klienta odpowiadasz za cykl życia klucza, uprawnienia do użycia klucza i inspekcje operacji na kluczach oraz w pełni kontrolujesz te elementy.

Szyfrowanie danych przy użyciu kluczy zarządzanych przez klienta dla Azure Database for PostgreSQL pojedynczego serwera jest ustawiane na poziomie serwera. W przypadku danego serwera klucz zarządzany przez klienta nazywany kluczem szyfrowania klucza (KEK) jest używany do szyfrowania klucza szyfrowania danych (DEK) używanego przez usługę. Klucz KEK jest kluczem asymetrycznym przechowywanym w wystąpieniu zarządzanym przez klienta [Azure Key Vault](../key-vault/general/security-features.md) klienta. Klucz szyfrowania klucza (KEK) i klucz szyfrowania danych (DEK) zostały szczegółowo opisane w dalszej części tego artykułu.

Key Vault to oparty na chmurze, zewnętrzny system zarządzania kluczami. Jest wysoce dostępna i zapewnia skalowalny, bezpieczny magazyn kluczy kryptograficznych RSA, opcjonalnie zabezpieczony przez sprzętowe moduły zabezpieczeń (HSM) zweryfikowane w wersji FIPS 140-2 poziom 2. Nie zezwala na bezpośredni dostęp do przechowywanego klucza, ale zapewnia autoryzowanym jednostkom usługi szyfrowania i odszyfrowywania. Key Vault wygenerować klucz, zaimportować go lub przenieść z [lokalnego urządzenia HSM.](../key-vault/keys/hsm-protected-keys.md)

> [!NOTE]
> Ta funkcja jest dostępna we wszystkich regionach świadczenia usługi Azure, Azure Database for PostgreSQL jeden serwer obsługuje warstwy cenowe "Ogólnego przeznaczenia" i "Zoptymalizowane pod kątem pamięci". Aby uzyskać informacje o innych ograniczeniach, zapoznaj się z [sekcją ograniczenia.](concepts-data-encryption-postgresql.md#limitations)

## <a name="benefits"></a>Korzyści

Szyfrowanie danych przy użyciu kluczy zarządzanych przez klienta Azure Database for PostgreSQL pojedynczego serwera zapewnia następujące korzyści:

* Dostęp do danych jest w pełni kontrolowany przez użytkownika przez możliwość usunięcia klucza i uniemożliwienia dostępu do bazy danych 
*    Pełna kontrola nad cyklem życia klucza, w tym rotacja klucza w celu dostosowania go do zasad firmowych
*    Centralne zarządzanie kluczami i ich organizacja w Azure Key Vault
*    Możliwość wdrożenia podziału obowiązków między pracowników ochrony oraz administratorów baz danych i administratorów systemu

## <a name="terminology-and-description"></a>Terminologia i opis

**Klucz szyfrowania danych (DEK):** symetryczny klucz AES256 używany do szyfrowania partycji lub bloku danych. Szyfrowanie każdego bloku danych przy użyciu innego klucza utrudnia ataki analizy kryptograficzne. Dostawca zasobów lub wystąpienie aplikacji, które szyfruje i odszyfrowuje określony blok, musi mieć dostęp do kluczy DEK. W przypadku zastąpienia klucza szyfrowania danych nowym kluczem tylko dane w skojarzonym bloku muszą zostać ponownie zaszyfrowane przy użyciu nowego klucza.

**Klucz szyfrowania klucza (KEK):** klucz szyfrowania używany do szyfrowania deks. KEK, który nigdy nie opuszcza Key Vault umożliwia dek same są szyfrowane i kontrolowane. Jednostka, która ma dostęp do KEK może różnić się od jednostki, która wymaga tego. Ponieważ klucz szyfrowania kluczy jest wymagany do odszyfrowania deKs, klucz KEK jest faktycznie pojedynczym punktem, za pomocą którego można skutecznie usunąć dek przez usunięcie klucza szyfrowania kluczy.

Deks, zaszyfrowane za pomocą KEK, są przechowywane oddzielnie. Tylko jednostka z dostępem do klucza szyfrowania kluczy może odszyfrować te dek. Aby uzyskać więcej informacji, zobacz Security in encryption at rest (Zabezpieczenia [szyfrowania danych w spoczynku).](../security/fundamentals/encryption-atrest.md)

## <a name="how-data-encryption-with-a-customer-managed-key-work"></a>Jak działa szyfrowanie danych za pomocą klucza zarządzanego przez klienta

:::image type="content" source="media/concepts-data-access-and-security-data-encryption/postgresql-data-encryption-overview.png" alt-text="Diagram przedstawiający przegląd Bring Your Own Key":::

Aby serwer PostgreSQL mógł używać kluczy zarządzanych przez klienta przechowywanych w programie Key Vault na potrzeby szyfrowania klucza szyfrowania danych, administrator Key Vault zapewnia następujące prawa dostępu do serwera:

* **get**: do pobierania części publicznej i właściwości klucza w magazynie kluczy.
* **wrapKey:** aby można było zaszyfrować klucz szyfrowania danych. Zaszyfrowany DEK jest przechowywany w Azure Database for PostgreSQL.
* **unwrapKey:** aby można było odszyfrować klucz szyfrowania danych. Azure Database for PostgreSQL odszyfrowany klucz szyfrowania danych do szyfrowania/odszyfrowywania danych

Administrator magazynu kluczy może również włączyć rejestrowanie zdarzeń [Key Vault inspekcji,](../azure-monitor/insights/key-vault-insights-overview.md)aby można było je później poddać inspekcji.

Gdy serwer jest skonfigurowany do używania klucza zarządzanego przez klienta przechowywanego w magazynie kluczy, wysyła klucz szyfrowania do magazynu kluczy w celu szyfrowania. Key Vault zwraca zaszyfrowany DEK, który jest przechowywany w bazie danych użytkownika. Podobnie w razie potrzeby serwer wysyła chroniony klucz szyfrowania danych do magazynu kluczy w celu odszyfrowania. Audytorzy mogą używać Azure Monitor do przeglądania Key Vault inspekcji dzienników zdarzeń, jeśli rejestrowanie jest włączone.

## <a name="requirements-for-configuring-data-encryption-for-azure-database-for-postgresql-single-server"></a>Wymagania dotyczące konfigurowania szyfrowania danych dla Azure Database for PostgreSQL pojedynczego serwera

Poniżej przedstawiono wymagania dotyczące konfigurowania Key Vault:

* Key Vault i Azure Database for PostgreSQL jeden serwer musi należeć do tej samej dzierżawy Azure Active Directory (Azure AD). Interakcje między Key Vault i serwera nie są obsługiwane. Przeniesienie Key Vault zasobów wymaga ponownego skonfigurowania szyfrowania danych.
* Magazyn kluczy musi być ustawiony na 90 dni, aby "Dni zachowywały usunięte magazyny". Jeśli istniejący magazyn kluczy został skonfigurowany z niższą liczbą, należy utworzyć nowy magazyn kluczy, ponieważ nie można go zmodyfikować po utworzeniu.
* Włącz funkcję usuwania nie soft-delete w magazynie kluczy, aby chronić przed utratą danych w przypadku przypadkowego usunięcia Key Vault klucza. Zasoby nieużytkowo usunięte są przechowywane przez 90 dni, chyba że użytkownik odzyska je lub przeczyści w międzyczasie. Akcje odzyskiwania i przeczyszczania mają własne uprawnienia skojarzone w zasadach Key Vault dostępu. Funkcja usuwania nie programowego jest domyślnie wyłączona, ale można ją włączyć za pomocą programu PowerShell lub interfejsu wiersza polecenia platformy Azure (pamiętaj, że nie można jej włączyć za pośrednictwem Azure Portal). 
* Włącz ochronę przed przeczyszczaniem, aby wymusić obowiązkowy okres przechowywania usuniętych magazynów i obiektów magazynu
* Przyznaj Azure Database for PostgreSQL magazynu kluczy z uprawnieniami get, wrapKey i unwrapKey przy użyciu jego unikatowej tożsamości zarządzanej. W Azure Portal jest automatycznie tworzona unikatowa tożsamość usługi po włączeniu szyfrowania danych na pojedynczym serwerze PostgreSQL. Zobacz [Szyfrowanie danych dla Azure Database for PostgreSQL pojedynczego](howto-data-encryption-portal.md) serwera przy użyciu Azure Portal, aby uzyskać szczegółowe instrukcje krok po kroku, gdy używasz Azure Portal.

Poniżej przedstawiono wymagania dotyczące konfigurowania klucza zarządzanego przez klienta:

* Klucz zarządzany przez klienta do szyfrowania klucza szyfrowania danych może być tylko asymetryczny, RSA 2048.
* Data aktywacji klucza (jeśli jest ustawiona) musi być datą i godziną w przeszłości. Data wygaśnięcia (jeśli jest ustawiona) musi być przyszłą datą i godziną.
* Klucz musi mieć stan *Włączone.*
* Jeśli importujesz [istniejący klucz](/rest/api/keyvault/ImportKey/ImportKey) do magazynu kluczy, upewnij się, że jest on dostępny w obsługiwanych formatach plików ( , , `.pfx` `.byok` `.backup` ).

## <a name="recommendations"></a>Zalecenia

Jeśli używasz szyfrowania danych przy użyciu klucza zarządzanego przez klienta, poniżej znajdują się zalecenia dotyczące konfigurowania Key Vault:

* Ustaw blokadę zasobu na Key Vault, aby kontrolować, kto może usuwać ten zasób krytyczny i zapobiegać przypadkowemu lub nieautoryzowanemu usunięciu.
* Włącz inspekcję i raportowanie dla wszystkich kluczy szyfrowania. Key Vault udostępnia dzienniki, które można łatwo wstrzyknąć do innych narzędzi do zarządzania informacjami i zdarzeniami zabezpieczeń. Azure Monitor przykładową usługą, która jest już zintegrowana, jest usługa Log Analytics.
* Upewnij się, Key Vault i Azure Database for PostgreSQL jeden serwer znajdują się w tym samym regionie, aby zapewnić szybszy dostęp do zawijania i odpakowania operacji DEK.
* Zablokuj usługę Azure KeyVault tylko do prywatnego punktu końcowego i wybranych sieci, zezwalaj na zabezpieczanie zasobów tylko zaufanym usługom firmy *Microsoft.* 

    :::image type="content" source="media/concepts-data-access-and-security-data-encryption/keyvault-trusted-service.png" alt-text="trusted-service-with-AKV":::

Poniżej znajdują się zalecenia dotyczące konfigurowania klucza zarządzanego przez klienta:

* Przechowuj kopię klucza zarządzanego przez klienta w bezpiecznym miejscu lub usłudze escrow.

* Jeśli Key Vault klucz, utwórz kopię zapasową klucza przed użyciem klucza po raz pierwszy. Kopię zapasową można przywrócić tylko do Key Vault. Aby uzyskać więcej informacji o poleceniu kopii zapasowej, zobacz [Backup-AzKeyVaultKey](/powershell/module/az.keyVault/backup-azkeyVaultkey).

## <a name="inaccessible-customer-managed-key-condition"></a>Warunek niedostępny klucza zarządzanego przez klienta

W przypadku konfigurowania szyfrowania danych przy użyciu klucza zarządzanego przez klienta w programie Key Vault serwer musi mieć ciągły dostęp do tego klucza, aby serwer pozostał w trybie online. Jeśli serwer utraci dostęp do klucza zarządzanego przez klienta w Key Vault, serwer rozpocznie odmawianie wszystkich połączeń w ciągu 10 minut. Serwer wysyła odpowiedni komunikat o błędzie i zmienia stan serwera na *Niedostępny.* Niektóre z powodów, dla których serwer może osiągnąć ten stan, to:

* Jeśli utworzymy serwer przywracania do punktu w czasie dla serwera Azure Database for PostgreSQL z włączonym szyfrowaniem danych, nowo utworzony serwer będzie w stanie *Niedostępny.* Stan serwera można naprawić za pomocą polecenia [Azure Portal](howto-data-encryption-portal.md#using-data-encryption-for-restore-or-replica-servers) interfejsu [wiersza polecenia.](howto-data-encryption-cli.md#using-data-encryption-for-restore-or-replica-servers)
* Jeśli utworzymy replikę do odczytu dla serwera Azure Database for PostgreSQL z włączonym szyfrowaniem danych, serwer repliki będzie w stanie *Niedostępny.* Stan serwera można naprawić za pomocą polecenia [Azure Portal](howto-data-encryption-portal.md#using-data-encryption-for-restore-or-replica-servers) interfejsu [wiersza polecenia.](howto-data-encryption-cli.md#using-data-encryption-for-restore-or-replica-servers)
* Jeśli usuniesz usługę KeyVault, Azure Database for PostgreSQL pojedynczy serwer nie będzie mógł uzyskać dostępu do klucza i przeniesie się do *stanu Niedostępna.* [Odzyskaj Key Vault](../key-vault/general/key-vault-recovery.md) i ponownie sprawdź szyfrowanie danych, aby udostępnić *serwer*.
* Jeśli usuniemy klucz z usługi KeyVault, Azure Database for PostgreSQL pojedynczy serwer nie będzie mógł uzyskać dostępu do klucza i przeniesie się do stanu *Niedostępna.* [Odzyskaj klucz](../key-vault/general/key-vault-recovery.md) i ponownie sprawdź szyfrowanie danych, aby udostępnić *serwer .*
* Jeśli klucz przechowywany w usłudze Azure KeyVault wygaśnie, klucz stanie się nieprawidłowy, a Azure Database for PostgreSQL jeden serwer stanie się niedostępny.  Rozszerz datę wygaśnięcia klucza przy użyciu interfejsu [wiersza](/cli/azure/keyvault/key#az_keyvault_key_set_attributes) polecenia, a następnie ponownie sprawdź szyfrowanie danych, aby udostępnić *serwer*.

### <a name="accidental-key-access-revocation-from-key-vault"></a>Przypadkowe odwołanie dostępu do klucza z Key Vault

Może się zdarzyć, że ktoś z wystarczającymi uprawnieniami dostępu Key Vault przypadkowo wyłączy dostęp serwera do klucza przez:

* Odwołanie uprawnień get, wrapKey i unwrapKey magazynu kluczy z serwera.
* Usuwanie klucza.
* Usuwanie magazynu kluczy.
* Zmiana reguł zapory magazynu kluczy.

* Usuwanie tożsamości zarządzanej serwera w usłudze Azure AD.

## <a name="monitor-the-customer-managed-key-in-key-vault"></a>Monitorowanie klucza zarządzanego przez klienta w Key Vault

Aby monitorować stan bazy danych i włączyć alerty dotyczące utraty dostępu funkcji ochrony funkcji Transparent Data Encryption, skonfiguruj następujące funkcje platformy Azure:

* [Azure Resource Health:](../service-health/resource-health-overview.md)Niedostępna baza danych, która utraciła dostęp do klucza klienta, jest przedstawiana jako "Niedostępna" po odmowie pierwszego połączenia z bazą danych.
* [Dziennik aktywności:](../service-health/alerts-activity-log-service-notifications-portal.md)w przypadku niepowodzenia dostępu do klucza klienta w zarządzanym przez klienta Key Vault wpisy są dodawane do dziennika aktywności. Możesz przywrócić dostęp tak szybko, jak to możliwe, jeśli tworzysz alerty dla tych zdarzeń.

* [Grupy akcji:](../azure-monitor/alerts/action-groups.md)zdefiniuj te grupy, aby wysyłać powiadomienia i alerty na podstawie Twoich preferencji.

## <a name="restore-and-replicate-with-a-customers-managed-key-in-key-vault"></a>Przywracanie i replikowanie przy użyciu klucza zarządzanego klienta w Key Vault

Po Azure Database for PostgreSQL zaszyfrowaniu pojedynczego serwera przy użyciu klucza zarządzanego klienta przechowywanego w u Key Vault, każda nowo utworzona kopia serwera jest również szyfrowana. Możesz utworzyć nową kopię za pomocą operacji przywracania lokalnego lub geograficznego albo replik do odczytu. Kopię można jednak zmienić w celu odzwierciedlenia klucza zarządzanego nowego klienta na potrzeby szyfrowania. Po zmianie klucza zarządzanego przez klienta stare kopie zapasowe serwera zaczynają używać najnowszego klucza.

Aby uniknąć problemów podczas konfigurowania szyfrowania danych zarządzanego przez klienta podczas przywracania lub tworzenia repliki do odczytu, należy wykonać następujące kroki na serwerach podstawowych i przywróconych/replikach:

* Zainicjuj proces przywracania lub odczytu repliki z podstawowego Azure Database for PostgreSQL pojedynczego serwera.
* Zachowaj nowo utworzony serwer (przywrócony/replikowy) w stanie niedostępnym, ponieważ jego unikatowa tożsamość nie ma jeszcze uprawnień do Key Vault.
* Na przywróconym/replikowym serwerze ponownie przeszukaj klucz zarządzany przez klienta w ustawieniach szyfrowania danych. Dzięki temu nowo utworzony serwer będzie mieć uprawnienia do opakowywania i odpakowywania klucza przechowywanego w Key Vault.

## <a name="limitations"></a>Ograniczenia

Na Azure Database for PostgreSQL obsługa szyfrowania danych w spoczynku przy użyciu klucza zarządzanego (CMK) klientów ma kilka ograniczeń —

* Obsługa tej funkcji jest ograniczona **do Ogólnego przeznaczenia** i zoptymalizowane **pod kątem** pamięci.
* Ta funkcja jest obsługiwana tylko w regionach i na serwerach, które obsługują magazyn o pojemności do 16 TB. Aby uzyskać listę regionów świadczenia usługi Azure, które obsługują magazyn o pojemności do 16 TB, zapoznaj się z sekcją dotyczącą magazynu w dokumentacji [tutaj](concepts-pricing-tiers.md#storage)

    > [!NOTE]
    > - Wszystkie nowe serwery PostgreSQL utworzone w regionach wymienionych powyżej, obsługa szyfrowania przy użyciu kluczy menedżera klienta jest **dostępna.** Serwer przywracania do punktu w czasie (PITR) lub replika odczytu nie kwalifikują się, chociaż teoretycznie są one "nowe".
    > - Aby sprawdzić, czy aprowizowany serwer obsługuje do 16 TB, możesz przejść do bloku warstwy cenowej w portalu i zobaczyć maksymalny rozmiar magazynu obsługiwany przez aprowowany serwer. Jeśli suwak można przenieść do 4 TB, serwer może nie obsługiwać szyfrowania przy użyciu kluczy zarządzanych przez klienta. Jednak dane są przez cały czas szyfrowane przy użyciu kluczy zarządzanych przez usługę. Jeśli masz jakieś pytania, skontaktuj się AskAzureDBforPostgreSQL@service.microsoft.com z toem.

* Szyfrowanie jest obsługiwane tylko w przypadku klucza kryptograficznego RSA 2048.

## <a name="next-steps"></a>Następne kroki

Dowiedz się, jak skonfigurować szyfrowanie danych przy użyciu klucza zarządzanego przez klienta dla pojedynczego serwera usługi [Azure Database for PostgreSQL](howto-data-encryption-portal.md)przy użyciu Azure Portal.
