---
title: Aktywowanie i konfigurowanie lokalnej konsoli zarządzania
description: Aktywowanie konsoli zarządzania gwarantuje, że czujniki są zarejestrowane na platformie Azure i wysyłają informacje do lokalnej konsoli zarządzania oraz że lokalna Konsola zarządzania wykonuje zadania zarządzania w połączonych czujników.
ms.date: 4/6/2021
ms.topic: how-to
ms.openlocfilehash: db0d2a84feeb5bf52932842badda8c126994c05d
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/06/2021
ms.locfileid: "106492158"
---
# <a name="activate-and-set-up-your-on-premises-management-console"></a>Aktywowanie i konfigurowanie lokalnej konsoli zarządzania 

Aktywacja i konfiguracja lokalnej konsoli zarządzania zapewniają następujące działania:

- Urządzenia sieciowe monitorowane za pomocą połączonych czujników są rejestrowane przy użyciu konta platformy Azure.

- Czujniki wysyłają informacje do lokalnej konsoli zarządzania.

- Lokalna Konsola zarządzania wykonuje zadania zarządzania w połączonych czujników.

- Zainstalowano certyfikat SSL.

## <a name="sign-in-for-the-first-time"></a>Logowanie się po raz pierwszy

Aby zalogować się do konsoli zarządzania:

1. Podczas instalacji systemu przejdź do adresu IP otrzymanego w lokalnej konsoli zarządzania.
 
1. Wprowadź nazwę użytkownika i hasło otrzymane dla lokalnej konsoli zarządzania podczas instalacji systemu. 


Jeśli nie pamiętasz hasła, wybierz opcję **Odzyskaj hasło**  i zobacz [Odzyskiwanie hasła](how-to-manage-the-on-premises-management-console.md#password-recovery) , aby uzyskać instrukcje dotyczące sposobu odzyskania hasła.

## <a name="activate-the-on-premises-management-console"></a>Aktywowanie lokalnej konsoli zarządzania

Po zalogowaniu się po raz pierwszy należy aktywować lokalną konsolę zarządzania przez pobranie i przekazanie pliku aktywacji. 

Aby aktywować lokalną konsolę zarządzania:

1. Zaloguj się do lokalnej konsoli zarządzania.

1. W oknie powiadomienia o alertach w górnej części ekranu wybierz łącze **Wykonaj akcję** .

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/take-action.png" alt-text="Wybierz łącze Przejmij akcję z alertu w górnej części ekranu.":::

1. Na ekranie podręcznym aktywacji wybierz łącze **Azure Portal** .

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/azure-portal.png" alt-text="Wybierz łącze Azure Portal z komunikatu podręcznego.":::
 
1. Wybierz subskrypcję, aby skojarzyć lokalną konsolę zarządzania z usługą, a następnie wybierz przycisk **Pobierz plik aktywacji lokalnej konsoli zarządzania** . Plik aktywacji zostanie pobrany.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/cloud_download_opm_activation_file.png" alt-text="Pobierz plik aktywacji.":::

   Jeśli subskrypcja nie została jeszcze dołączona, Dołącz [do niej subskrypcję](how-to-manage-subscriptions.md#onboard-a-subscription).

1. Przejdź z powrotem do ekranu podręcznego **aktywacji** i wybierz pozycję **Wybierz plik**.

1. Wybierz pobrany plik.

Po początkowej aktywacji liczba monitorowanych urządzeń może przekroczyć liczbę zatwierdzonych urządzeń zdefiniowanych podczas dołączania. Dzieje się tak, jeśli połączysz więcej czujników z konsolą zarządzania. Jeśli występuje rozbieżność między liczbą monitorowanych urządzeń i liczbą zatwierdzonych urządzeń, w konsoli zarządzania zostanie wyświetlone ostrzeżenie. Jeśli tak się stanie, Przekaż nowy plik aktywacji.

## <a name="set-up-a-certificate"></a>Konfigurowanie certyfikatu

Po zainstalowaniu konsoli zarządzania zostanie wygenerowany lokalny certyfikat z podpisem własnym. Ten certyfikat służy do uzyskiwania dostępu do konsoli programu. Gdy administrator zaloguje się po raz pierwszy, użytkownik zostanie poproszony o dołączenie certyfikatu SSL/TLS. 

Dostępne są dwa poziomy zabezpieczeń:

- Zapoznaj się z określonymi wymaganiami dotyczącymi certyfikatów i szyfrowania, przekazując certyfikat podpisany przez urząd certyfikacji.
- Zezwalaj na sprawdzanie poprawności między konsolą zarządzania i połączonymi czujnikami. Sprawdzanie poprawności jest oceniane względem listy odwołania certyfikatów i daty wygaśnięcia certyfikatu. *Jeśli walidacja nie powiedzie się, komunikacja między konsolą zarządzania i czujnikiem zostanie zatrzymana, a w konsoli zostanie wyświetlony błąd sprawdzania poprawności.* Ta opcja jest domyślnie włączona po instalacji.  

Konsola obsługuje następujące typy certyfikatów:

- Infrastruktura kluczy prywatnych i korporacyjnych (prywatna infrastruktura PKI)

- Infrastruktura kluczy publicznych (publiczna infrastruktura PKI)

- Wygenerowane lokalnie na urządzeniu (lokalnie z podpisem własnym) 

  > [!IMPORTANT]
  > Zalecamy, aby nie używać certyfikatu z podpisem własnym. Certyfikat nie jest zabezpieczony i powinien być używany tylko w środowiskach testowych. Nie można zweryfikować właściciela certyfikatu, a zabezpieczenia systemu nie są obsługiwane. Nie należy używać tej opcji w przypadku sieci produkcyjnych.

Aby przekazać certyfikat:

1. Po wyświetleniu monitu po zalogowaniu Zdefiniuj nazwę certyfikatu.

1. Przekaż pliki CRT i Key.

1. Wprowadź hasło i w razie potrzeby Przekaż plik PEM.

Po przekazaniu certyfikatu podpisanego przez urząd certyfikacji może być konieczne odświeżenie ekranu.

Aby wyłączyć weryfikację między konsolą zarządzania i połączonymi czujnikami:

1. Wybierz opcję **Dalej**.

1. Wyłącz opcję **Włącz tryb walidacji całego systemu** .

Aby uzyskać informacje o przekazywaniu nowego certyfikatu, obsługiwanych plikach certyfikatów i powiązanych elementach, zobacz [Zarządzanie lokalną konsolą zarządzania](how-to-manage-the-on-premises-management-console.md).

## <a name="connect-sensors-to-the-on-premises-management-console"></a>Łączenie czujników z lokalną konsolą zarządzania

Należy upewnić się, że czujniki wysyłają informacje do lokalnej konsoli zarządzania i że lokalna Konsola zarządzania może wykonywać kopie zapasowe, zarządzać alertami i wykonywać inne działania na czujników. Aby to zrobić, należy wykonać poniższe procedury, aby sprawdzić, czy nastąpiło wstępne połączenie między czujnikami i lokalną konsolą zarządzania.

Dostępne są dwie opcje łączenia usług Azure Defender for IoT z lokalną konsolą zarządzania:

- Łączenie z konsolą czujnika

- Nawiązywanie połączenia przy użyciu tunelowania

Po nawiązaniu połączenia należy skonfigurować lokację za pomocą tych czujników.

### <a name="connect-sensors-to-the-on-premises-management-console-from-the-sensor-console"></a>Łączenie czujników z lokalną konsolą zarządzania z poziomu konsoli czujnika

Czujniki można połączyć z lokalną konsolą zarządzania z poziomu konsoli czujnika:

1. W lokalnej konsoli zarządzania wybierz pozycję **Ustawienia systemowe**.

1. Skopiuj **Parametry połączenia Kopiuj**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/connection-string.png" alt-text="Skopiuj parametry połączenia dla czujnika.":::

1. Na czujniku przejdź do **ustawień systemowych** i wybierz pozycję **połączenie z konsolą zarządzania**:::image type="icon" source="media/how-to-manage-sensors-from-the-on-premises-management-console/connection-to-management-console.png" border="false":::

1. Wklej skopiowane parametry połączenia z lokalnej konsoli zarządzania do pola **Parametry połączenia** .

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/paste-connection-string.png" alt-text="Wklej skopiowany ciąg połączenia do pola parametry połączenia.":::

1. Wybierz pozycję **Połącz**.

### <a name="connect-sensors-by-using-tunneling"></a>Łączenie czujników przy użyciu tunelowania

Włącz zabezpieczone połączenie tunelowania między czujnikami organizacyjnymi a lokalną konsolą zarządzania. Ta konfiguracja omija interakcje z zaporą organizacyjną i w efekcie zmniejsza obszar narażony na ataki.

Używanie tunelowania umożliwia nawiązanie połączenia z lokalną konsolą zarządzania przy użyciu adresu IP i jednego portu (czyli 9000) do dowolnego czujnika.

Aby skonfigurować tunelowanie w lokalnej konsoli zarządzania:

- Zaloguj się do lokalnej konsoli zarządzania i uruchom następujące polecenia:

  ```bash
  cyberx-management-tunnel-enable
  service apache2 reload
  sudo cyberx-management-tunnel-add-xsense --xsenseuid <sensorIPAddress> --xsenseport 9000
  service apache2 reload
  ```

Aby skonfigurować tunelowanie na czujniku:

1. Ręcznie otwórz port TCP 9000 na czujniku (Network. Properties). Jeśli port nie jest otwarty, czujnik odrzuci połączenie z lokalnej konsoli zarządzania.

2. Zaloguj się do każdego czujnika i uruchom następujące polecenia:

   ```bash
   sudo cyberx-xsense-management-connect -ip <centralmanagerIPAddress>
   sudo cyberx-xsense-management-tunnel
   sudo vi /var/cyberx/properties/network.properties
   opened_tcp_incoming_ports=22,80,443,102,9000
   sudo cyberx-xsense-network-validation
   sudo /etc/network/if-up.d/iptables-recover
   sudo iptables -nvL
   ```

## <a name="set-up-a-site"></a>Konfigurowanie lokacji

Domyślna mapa przedsiębiorstwa zapewnia ogólny widok swoich urządzeń w oparciu o kilka poziomów lokalizacji geograficznych.

Widok urządzeń może być wymagany w przypadku, gdy struktura organizacyjna i uprawnienia użytkownika są złożone. W takich przypadkach Konfiguracja lokacji może być określona przez globalną strukturę organizacyjną, oprócz standardowej struktury lokacji lub strefy.

Aby można było obsługiwać to środowisko, należy utworzyć globalną topologię biznesową opartą na jednostkach, regionach, lokacjach i strefach firmy w organizacji. Należy również zdefiniować uprawnienia dostępu użytkowników wokół tych jednostek za pomocą grup dostępu.

Grupy dostępu umożliwiają lepszą kontrolę nad tym, gdzie użytkownicy zarządzają i analizują urządzenia w usłudze Defender for IoT.

### <a name="how-it-works"></a>Jak to działa

Można zdefiniować jednostkę biznesową i region dla każdej lokacji w organizacji. Następnie można dodać strefy, które są jednostkami logicznymi istniejącymi w sieci. 

Należy przypisać co najmniej jeden czujnik dla każdej strefy. Model pięciu poziomów zapewnia elastyczność i stopień szczegółowości, który jest wymagany do zapewnienia systemu ochrony, który odzwierciedla strukturę organizacji.

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/diagram-of-sensor-showing-relationships.png" alt-text="Diagram przedstawiający czujniki i relacje regionalne.":::

Za pomocą widoku przedsiębiorstwa można edytować witryny bezpośrednio. Po wybraniu witryny z widoku przedsiębiorstwa liczba otwartych alertów pojawia się obok każdej strefy.

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/console-map-with-data-overlay-v2.png" alt-text="Zrzut ekranu przedstawiający lokalną mapę konsoli zarządzania z nakładką danych Berlin.":::

Aby skonfigurować lokację:

1. Dodaj nowe jednostki biznesowe, aby odzwierciedlały strukturę logiczną organizacji.

   1. W widoku przedsiębiorstwa wybierz pozycję **Wszystkie lokacje**  >  **Zarządzaj jednostkami biznesowymi**.

      :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/manage-business-unit.png" alt-text="Wybierz pozycję Zarządzaj jednostką biznesową z menu rozwijanego Wszystkie lokacje na ekranie widok przedsiębiorstwa.":::

   1. Wprowadź nową nazwę jednostki biznesowej i wybierz pozycję **Dodaj**.

1. Dodawanie nowych regionów w celu odzwierciedlenia regionów organizacji.

   1. W widoku przedsiębiorstwa wybierz pozycję **wszystkie regiony**  >  **Zarządzaj regionami**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/manage-regions.png" alt-text="Wybierz pozycję Wszystkie regiony, a następnie Zarządzaj regionami, aby zarządzać regionami w przedsiębiorstwie.":::

   1. Wprowadź nazwę nowego regionu i wybierz pozycję **Dodaj**.

1. Dodaj lokację.

   1. W widoku przedsiębiorstwa wybierz :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/new-site-icon.png" border="false"::: górny pasek. Kursor zostanie wyświetlony jako znak plusa ( **+** ).

   1. Umieść kursor **+** w miejscu nowej lokacji i wybierz ją. Zostanie otwarte okno dialogowe **Utwórz nową lokację** .

      :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/create-new-site-screen.png" alt-text="Zrzut ekranu przedstawiający widok Tworzenie nowej witryny.":::

   1. Zdefiniuj nazwę i adres fizyczny nowej witryny, a następnie wybierz pozycję **Zapisz**. Nowa lokacja zostanie wyświetlona na mapie witryny.

4. [Dodaj strefy do lokacji](#create-enterprise-zones).

5. [Połącz czujniki](how-to-manage-individual-sensors.md#connect-a-sensor-to-the-management-console).

6. [Przypisywanie czujnika do stref lokacji](#assign-sensors-to-zones).

### <a name="delete-a-site"></a>Usuwanie witryny

Jeśli lokacja nie jest już potrzebna, można ją usunąć z lokalnej konsoli zarządzania.

Aby usunąć lokację:

1. W oknie **Zarządzanie lokacją** wybierz :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: z paska, który zawiera nazwę lokacji, a następnie wybierz pozycję **Usuń lokację**. Zostanie wyświetlone okno dialogowe potwierdzające, czy chcesz usunąć lokację.

2. W polu potwierdzenia wybierz pozycję **Potwierdź**.

## <a name="create-enterprise-zones"></a>Utwórz strefy przedsiębiorstwa

Strefy są jednostkami logicznymi, które umożliwiają dzielenie urządzeń w obrębie lokacji na grupy według różnych właściwości. Można na przykład utworzyć grupy dla linii produkcyjnych, podstacji, obszarów lokacji lub typów urządzeń. Strefy można definiować na podstawie wszelkich cech, które są odpowiednie dla Twojej organizacji.

Strefy można konfigurować w ramach procesu konfiguracji lokacji.

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/site-management-zones-screen-v2.png" alt-text="Zrzut ekranu przedstawiający widok strefy zarządzania lokacją.":::

W poniższej tabeli opisano parametry w oknie **Zarządzanie lokacją** .

| Parametr | Opis |
|--|--|
| Nazwa | Nazwa czujnika. Tę nazwę można zmienić tylko z czujnika. Aby uzyskać więcej informacji, zobacz Podręcznik użytkownika usługi Defender for IoT. |
| Adres IP | Adres IP czujnika. |
| Wersja | Wersja czujnika. |
| Łączność | Stan łączności czujnika. Stan może być **połączony** lub **odłączony**. |
| Ostatnie uaktualnienie | Data ostatniego uaktualnienia. |
| Postęp uaktualniania | Pasek postępu pokazuje stan procesu uaktualniania w następujący sposób:<br />-Przekazywanie pakietu<br />— Przygotowywanie do instalacji<br />-Zatrzymywanie procesów<br />— Tworzenie kopii zapasowej danych<br />— Tworzenie migawki<br />-Aktualizowanie konfiguracji<br />-Aktualizowanie zależności<br />-Aktualizowanie bibliotek<br />-Stosowanie poprawek baz danych<br />-Uruchamianie procesów<br />-Walidacja Sanity systemu<br />-Walidacja zakończyła się pomyślnie<br />-Sukces<br />-Niepowodzenie<br />-Rozpoczęto uaktualnianie<br />— Rozpoczynanie instalacjiogress bar shows the status of the upgrade process, as follows:<br />- Uploading package<br />- Preparing to install<br />- Stopping processes<br />- Backing up data<br />- Taking snapshot<br />- Updating configuration<br />- Updating dependencies<br />- Updating libraries<br />- Patching databases<br />- Starting processes<br />- Validating system sanity<br />- Validation succeeded<br />- Success<br />- Failure<br />- Upgrade started<br />- Starting installation<br /></br >Aby uzyskać szczegółowe informacje o uaktualnianiu programu, zapoznaj się z tematem [Pomoc techniczna firmy Microsoft](https://support.microsoft.com/) . |
| Urządzenia | Liczba urządzeń, które są monitorowane przez czujnik. |
| Alerty | Liczba alertów w czujniku. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/assign-icon.png" border="false"::: | Umożliwia przypisanie czujnika do stref. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/delete-icon.png" border="false":::| Włącza usuwanie odłączonego czujnika z lokacji. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/sensor-icon.png" border="false"::: | Wskazuje, ile czujników jest obecnie połączonych ze strefą. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/ot-assets-icon.png" border="false"::: | Wskazuje, ile z zasobów jest obecnie połączonych ze strefą. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/number-of-alerts-icon.png" border="false"::: | Wskazuje liczbę alertów wysyłanych przez czujniki, które są przypisane do strefy. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/unassign-sensor-icon.png" border="false"::: | Cofa przypisanie czujników ze stref. |

Aby dodać strefę do lokacji:

1. W oknie **Zarządzanie lokacją** wybierz :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: z paska, który zawiera nazwę lokacji, a następnie wybierz pozycję **Dodaj strefę**. Pojawi się okno dialogowe **Utwórz nową strefę** .

    :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/create-new-zone-screen.png" alt-text="Zrzut ekranu przedstawiający widok Utwórz nową strefę.":::

1. Wprowadź nazwę strefy.

1. Wprowadź opis nowej strefy, która jasno określa charakterystykę, która została użyta do podziału lokacji na strefy.

1. Wybierz pozycję **Zapisz**. Nowa strefa zostanie wyświetlona w oknie **Zarządzanie lokacją** w ramach lokacji, do której należy Ta strefa.

Aby edytować strefę:

1. W oknie **Zarządzanie lokacją** wybierz :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: z paska, który zawiera nazwę strefy, a następnie wybierz pozycję **Edytuj strefę**. Zostanie wyświetlone okno dialogowe **Edytuj strefę** .

   :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/zone-edit-screen.png" alt-text="Zrzut ekranu przedstawiający okno dialogowe Edytowanie strefy.":::

1. Edytuj parametry strefy i wybierz pozycję **Zapisz**.

Aby usunąć strefę:

1. W oknie **Zarządzanie lokacją** wybierz :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: z paska, który zawiera nazwę strefy, a następnie wybierz pozycję **Usuń strefę**.

1. W polu potwierdzenia wybierz pozycję **tak**.

Aby filtrować według stanu łączności:

- W lewym górnym rogu wybierz pozycję :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/down-pointing-icon.png" border="false"::: Dalej, aby nawiązać **połączenie**, a następnie wybierz jedną z następujących opcji:

  - **Wszystkie**: przedstawia wszystkie czujniki, które są raportowane do tej lokalnej konsoli zarządzania.

  - **Połączono**: prezentuje tylko czujniki połączone.

  - **Rozłączono**: prezentuje tylko czujniki rozłączone.

Aby filtrować według stanu uaktualnienia:

- W lewym górnym rogu wybierz pozycję :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/down-pointing-icon.png" border="false"::: Dalej, aby **uaktualnić stan** , a następnie wybierz jedną z następujących opcji:

  - **Wszystkie**: przedstawia wszystkie czujniki, które są raportowane do tej lokalnej konsoli zarządzania.

  - **Prawidłowy**: przedstawia czujniki z prawidłowym stanem uaktualnienia.

  - **W toku**: przedstawia czujniki, które są w trakcie uaktualniania.

  - **Niepowodzenie**: przedstawia czujniki, których proces uaktualniania nie powiódł się.

## <a name="assign-sensors-to-zones"></a>Przypisywanie czujników do stref

Dla każdej strefy należy przypisać czujniki, które wykonują analizę ruchu lokalnego i alerty. Można przypisać tylko czujniki połączone z lokalną konsolą zarządzania.

Aby przypisać Czujnik:

1. Wybierz pozycję **Zarządzanie lokacją**. Nieprzypisane czujniki pojawiają się w lewym górnym rogu okna dialogowego.

   :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/unassigned-sensors-view.png" alt-text="Zrzut ekranu przedstawiający widok czujników nieprzypisane.":::

1. Sprawdź, czy stan **łączności** jest połączony. Jeśli nie, zobacz sekcję [łączenie czujników z lokalną konsolą zarządzania,](#connect-sensors-to-the-on-premises-management-console) Aby uzyskać szczegółowe informacje na temat nawiązywania połączenia. 

1. Wybierz :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/assign-icon.png" border="false"::: dla czujnika, który chcesz przypisać.

1. W oknie dialogowym **przypisywanie czujnika** wybierz jednostkę biznesową, region, witrynę i strefę do przypisania.

   :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/assign-sensor-screen.png" alt-text="Zrzut ekranu przedstawiający widok czujnika przypisywania.":::

1. Wybierz pozycję **Przypisz**.

Aby cofnąć przypisanie i usunięcie czujnika:

1. Odłącz czujnik od lokalnej konsoli zarządzania. Aby uzyskać szczegółowe informacje [, zobacz czujniki połączeń z lokalną konsolą zarządzania](#connect-sensors-to-the-on-premises-management-console) .

1. W oknie **Zarządzanie lokacją** wybierz czujnik i wybierz opcję :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/unassign-sensor-icon.png" border="false"::: . Czujnik pojawia się na liście nieprzypisanych czujników po kilku chwilach.

1. Aby usunąć nieprzypisany czujnik z lokacji, wybierz czujnik z listy nieprzypisanych czujników i wybierz opcję :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/delete-icon.png" border="false"::: .

## <a name="see-also"></a>Zobacz też

[Rozwiązywanie problemów z czujnikiem i lokalną konsolą zarządzania](how-to-troubleshoot-the-sensor-and-on-premises-management-console.md)
