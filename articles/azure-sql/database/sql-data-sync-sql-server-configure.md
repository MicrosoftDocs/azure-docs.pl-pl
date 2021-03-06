---
title: Konfiguracja funkcji SQL Data Sync
description: W tym samouczku przedstawiono sposób SQL Data Sync platformy Azure
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/14/2019
ms.openlocfilehash: 77073d21f982e82e567e517b7d9eca061cb91859
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/21/2021
ms.locfileid: "107812957"
---
# <a name="tutorial-set-up-sql-data-sync-between-databases-in-azure-sql-database-and-sql-server"></a>Samouczek: konfigurowanie SQL Data Sync baz danych w Azure SQL Database i SQL Server
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Z tego samouczka dowiesz się, jak skonfigurować SQL Data Sync przez utworzenie grupy synchronizacji, która zawiera zarówno Azure SQL Database, jak i SQL Server wystąpienia. Grupa synchronizacji jest skonfigurowana niestandardowo i synchronizowana zgodnie z ustawionym harmonogramem.

W samouczku założono, że masz co najmniej pewne doświadczenie w pracy z SQL Database i SQL Server.

Aby uzyskać omówienie SQL Data Sync, zobacz [Sync data across cloud and on-premises databases with SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)(Synchronizowanie danych między bazami danych w chmurze i lokalnymi bazami danych przy użyciu SQL Data Sync).

Przykłady programu PowerShell dotyczące sposobu konfigurowania SQL Data Sync, zobacz Jak synchronizować bazy danych w programie [SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md) lub między bazami danych w Azure SQL Database [i SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)

> [!IMPORTANT]
> SQL Data Sync **nie obsługuje** Azure SQL Managed Instance w tej chwili.

## <a name="create-sync-group"></a>Tworzenie grupy synchronizacji

1. Przejdź do [Azure Portal,](https://portal.azure.com) aby znaleźć bazę danych w SQL Database. Wyszukaj i wybierz pozycję **Bazy danych SQL.**

    ![Wyszukiwanie baz danych, Microsoft Azure Portal](./media/sql-data-sync-sql-server-configure/search-for-sql-databases.png)

1. Wybierz bazę danych, której chcesz użyć jako bazy danych centrum dla Data Sync.

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/select-sql-database.png" alt-text = "Select from the database list, Microsoft Azure portal":::

    > [!NOTE]
    > Baza danych centrum jest centralnym punktem końcowym topologii synchronizacji, w którym grupa synchronizacji ma wiele punktów końcowych bazy danych. Wszystkie inne członkowskie bazy danych z punktami końcowymi w grupie synchronizacji są synchronizowane z bazą danych centrum.

1. W menu **bazy danych SQL** dla wybranej bazy danych wybierz pozycję **Synchronizuj z innymi bazami danych.**

    :::image type="content" source="./media/sql-data-sync-sql-server-configure/sync-to-other-databases.png" alt-text = "Sync to other databases, Microsoft Azure portal":::

1. Na stronie **Synchronizuj z innymi bazami** danych wybierz pozycję **Nowa grupa synchronizacji.** Zostanie **otwarta strona Nowa grupa synchronizacji** z **grupę tworzenia synchronizacji.**

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/create-sync-group.png" alt-text = "Set up new sync group with private link":::

   Na **stronie Tworzenie Data Sync grupy** zmień następujące ustawienia:

   | Ustawienie                        | Opis |
   | ------------------------------ | ------------------------------------------------- |
   | **Nazwa grupy synchronizacji** | Wprowadź nazwę nowej grupy synchronizacji. Ta nazwa różni się od nazwy samej bazy danych. |
   | **Baza danych metadanych synchronizacji** | Wybierz utworzenie bazy danych (zalecane) lub użycie istniejącej bazy danych.<br/><br/>W przypadku wybrania opcji **Nowa baza danych** wybierz pozycję Utwórz **nową bazę danych.** Następnie na stronie **SQL Database** nazwij i skonfiguruj nową bazę danych, a następnie wybierz przycisk **OK.**<br/><br/>Jeśli wybierzesz **pozycję Użyj istniejącej bazy danych,** wybierz ją z listy. |
   | **Automatyczna synchronizacja** | Wybierz **pozycję Wł.** lub **Wył.**.<br/><br/>W przypadku wybrania **opcji Wł.** wprowadź liczbę i wybierz pozycję **Sekundy,** **Minuty,** **Godziny** **lub** Dni w sekcji **Częstotliwość synchronizacji.**<br/> Pierwsza synchronizacja rozpoczyna się po upływie wybranego okresu interwału od czasu zapisania konfiguracji.|
   | **Konfliktów** | Wybierz **pozycję Hub win lub** Member **win**.<br/><br/>**Hub win** oznacza, że gdy wystąpią konflikty, dane w bazie danych centrum zastępują dane powodujące konflikt w członkowskiej bazie danych.<br/><br/>**Wygranie członka** oznacza, że gdy wystąpią konflikty, dane w bazie danych członkowskiej zastępują dane powodujące konflikt w bazie danych centrum. |
   | **Korzystanie z linku prywatnego** | Wybierz prywatny punkt końcowy zarządzany przez usługę, aby nawiązać bezpieczne połączenie między usługą synchronizacji a bazą danych centrum. |

   > [!NOTE]
   > Firma Microsoft zaleca utworzenie nowej, pustej bazy danych do użycia jako baza danych **metadanych synchronizacji.** Data Sync tworzy tabele w tej bazie danych i uruchamia częste obciążenie. Ta baza danych jest udostępniana jako baza **danych metadanych synchronizacji** dla wszystkich grup synchronizacji w wybranym regionie i subskrypcji. Nie można zmienić bazy danych ani jej nazwy bez usunięcia wszystkich grup synchronizacji i agentów synchronizacji w regionie. Ponadto bazy danych zadań elastycznych nie można używać jako bazy danych SQL Data Sync metadanych i na odwrót.  

   Wybierz **przycisk OK** i poczekaj na utworzenia i wdrożenia grupy synchronizacji.
   
1. Jeśli na **stronie Nowa grupa synchronizacji** wybrano opcję Użyj **linku prywatnego,** należy zatwierdzić połączenie prywatnego punktu końcowego. Link w komunikacie informacyjnym umożliwia połączenie z prywatnym punktem końcowym, w którym można zatwierdzić połączenie. 

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/approve-private-link-update.png" alt-text = "Approve private link":::
   
   > [!NOTE]
   > Prywatne linki dla grupy syng i grupy członków synchronizacji, które mają zostać utworzone, zatwierdzone i wyłączone oddzielnie. 

## <a name="add-sync-members"></a>Dodawanie członków synchronizacji

Po utworzeniu i wdrożeniu nowej grupy synchronizacji otwórz grupę synchronizacji i uzyskaj dostęp do strony Bazy danych, na której wybierzesz członków synchronizacji. 

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/add-sync-members.png" alt-text = "Select sync members":::
   
   > [!NOTE]
   > Aby zaktualizować lub wstawić nazwę użytkownika i  hasło do bazy danych centrum, przejdź do sekcji Baza danych centrum na **stronie Wybieranie członków synchronizacji.** 

### <a name="to-add-a-database-in-azure-sql-database"></a>Aby dodać bazę danych w Azure SQL Database

W sekcji **Select sync members (Wybieranie** członków synchronizacji) opcjonalnie dodaj bazę danych w usłudze Azure SQL Database do grupy synchronizacji, wybierając **pozycję Add an Azure Database (Dodaj bazę danych platformy Azure).** Zostanie otwarta strona Konfigurowanie bazy danych platformy **Azure.**
  
   :::image type="content" source="./media/sql-data-sync-sql-server-configure/step-two-configure.png" alt-text = "Add a database to the sync group":::
   
  Na **stronie Azure SQL Database** ustawienia zmień następujące ustawienia:

  | Ustawienie                       | Opis |
  | ----------------------------- | ------------------------------------------------- |
  | **Nazwa członka synchronizacji** | Podaj nazwę nowego członka synchronizacji. Ta nazwa różni się od samej nazwy bazy danych. |
  | **Subskrypcja** | Wybierz skojarzoną subskrypcję platformy Azure na potrzeby rozliczeń. |
  | **Serwer usługi SQL Azure** | Wybierz istniejący serwer. |
  | **Azure SQL Database** | Wybierz istniejącą bazę danych w SQL Database. |
  | **Wskazówki dotyczące synchronizacji** | Wybierz **pozycję Synchronizacja dwukierunkowa**, **Do centrum** lub Z **centrum**. |
  | **Nazwa użytkownika** i **hasło** | Wprowadź istniejące poświadczenia dla serwera, na którym znajduje się członkowski bazy danych. Nie wprowadzaj *nowych* poświadczeń w tej sekcji. |
  | **Korzystanie z linku prywatnego** | Wybierz prywatny punkt końcowy zarządzany przez usługę, aby nawiązać bezpieczne połączenie między usługą synchronizacji a członcyjną bazą danych. |

  Wybierz **przycisk OK** i poczekaj, aż nowy członek synchronizacji zostanie utworzony i wdrożony.

<a name="add-on-prem"></a>

### <a name="to-add-a-sql-server-database"></a>Aby dodać bazę SQL Server danych

W sekcji **Baza danych członków** opcjonalnie dodaj bazę SQL Server do grupy synchronizacji, wybierając pozycję Dodaj **lokalną bazę danych.** Zostanie **otwarta strona Konfigurowanie lokalne,** na której można wykonać następujące czynności:

1. Wybierz **pozycję Wybierz bramę agenta synchronizacji.** Zostanie **otwarta strona Wybierz agenta** synchronizacji.

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/steptwo-agent.png" alt-text = "Creating a sync agent":::

1. Na stronie **Wybierz agenta synchronizacji** wybierz, czy chcesz użyć istniejącego agenta, czy utworzyć agenta.

   W przypadku wybrania **opcji Istniejący agenci** wybierz istniejącego agenta z listy.

   W przypadku wybrania **opcji Utwórz nowego agenta** wykonaj następujące czynności:

   1. Pobierz agenta synchronizacji danych z podanego linku i zainstaluj go na komputerze, SQL Server znajduje się. Agenta można również pobrać bezpośrednio z Azure SQL Data Sync [Agent.](https://www.microsoft.com/download/details.aspx?id=27693)

      > [!IMPORTANT]
      > Musisz otworzyć wychodzący port TCP 1433 w zaporze, aby pozwolić agentowi klienta na komunikację z serwerem.

   1. Wprowadź nazwę agenta.

   1. Wybierz **pozycję Utwórz i wygeneruj** klucz, a następnie skopiuj klucz agenta do schowka.

   1. Wybierz **przycisk OK,** aby zamknąć **stronę Wybierz agenta synchronizacji.**

1. Na komputerze SQL Server znajdź i uruchom aplikację Agent synchronizacji klienta.

   ![Aplikacja agenta klienta synchronizacji danych](./media/sql-data-sync-sql-server-configure/datasync-preview-clientagent.png)

    1. W aplikacji agenta synchronizacji wybierz pozycję **Prześlij klucz agenta.** Zostanie **otwarte okno dialogowe Konfiguracja bazy danych metadanych** synchronizacji.

    1. W **oknie dialogowym Konfiguracja** bazy danych metadanych synchronizacji wklej klucz agenta skopiowany z Azure Portal. Podaj również istniejące poświadczenia dla serwera, na którym znajduje się baza danych metadanych. (Jeśli utworzono bazę danych metadanych, ta baza danych znajduje się na tym samym serwerze co baza danych centrum). Wybierz **przycisk OK** i poczekaj na zakończenie konfiguracji.

        ![Wprowadź klucz agenta i poświadczenia serwera](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-enterkey.png)

        > [!NOTE]
        > Jeśli wystąpi błąd zapory, utwórz regułę zapory na platformie Azure, aby zezwolić na ruch przychodzący z SQL Server komputera. Regułę można utworzyć ręcznie w portalu lub w programie SQL Server Management Studio (SSMS). W programie SSMS połącz się z bazą danych centrum na platformie Azure, wprowadzając jej nazwę <hub_database_name>.database.windows.net.

    1. Wybierz **pozycję Zarejestruj,** aby zarejestrować SQL Server bazy danych za pomocą agenta. Zostanie **SQL Server okno dialogowe** Konfiguracja aplikacji.

        ![Dodawanie i konfigurowanie bazy SQL Server danych](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-adddb.png)

    1. W **oknie dialogowym SQL Server Configuration** (Konfiguracja komputera) wybierz opcję nawiązania połączenia przy użyciu SQL Server lub uwierzytelniania systemu Windows. Jeśli wybierzesz SQL Server, wprowadź istniejące poświadczenia. Podaj SQL Server i nazwę bazy danych, którą chcesz zsynchronizować, a następnie wybierz pozycję **Test connection,** aby przetestować ustawienia. Następnie wybierz **pozycję Zapisz,** aby wyświetlić zarejestrowaną bazę danych na liście.

        ![SQL Server bazy danych jest teraz zarejestrowana](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-dbadded.png)

    1. Zamknij aplikację Agent synchronizacji klienta.

1. W portalu na stronie **Konfigurowanie lokalne** wybierz pozycję Wybierz **bazę danych**.

1. Na stronie **Wybieranie bazy** danych w polu **Nazwa** członka synchronizacji podaj nazwę nowego członka synchronizacji. Ta nazwa różni się od nazwy samej bazy danych. Wybierz bazę danych z listy. W polu **Synchronizuj wskazówki** wybierz pozycję **Synchronizacja** dwukierunkowa, Do **centrum** lub **Z centrum**.

    ![Wybieranie lokalnej bazy danych](./media/sql-data-sync-sql-server-configure/datasync-preview-selectdb.png)

1. Wybierz **przycisk OK,** aby zamknąć **stronę Wybieranie bazy** danych. Następnie wybierz **przycisk OK,** aby zamknąć **stronę Konfigurowanie** lokalne i poczekać na utworzenia i wdrożenia nowego członka synchronizacji. Na koniec wybierz **przycisk OK,** aby zamknąć **stronę Wybieranie członków synchronizacji.**

> [!NOTE]
> Aby nawiązać SQL Data Sync z agentem lokalnym, dodaj nazwę użytkownika do roli DataSync_Executor *.* Data Sync tworzy tę rolę w SQL Server wystąpienia.

## <a name="configure-sync-group"></a>Konfigurowanie grupy synchronizacji

Po utworzeniu i wdrożeniu nowych członków grupy synchronizacji przejdź do sekcji **Tabele** na stronie **Grupa synchronizacji bazy** danych.

![Ustawienia kroku 3](./media/sql-data-sync-sql-server-configure/configure-sync-group.png)

1. Na stronie **Tabele** wybierz bazę danych z listy członków grupy synchronizacji i wybierz pozycję **Odśwież schemat.** Należy oczekiwać kilkuminutowego opóźnienia w schemacie odświeżania. W przypadku korzystania z linku prywatnego opóźnienie może być dłuższe o kilka minut.

1. Z listy wybierz tabele, które chcesz zsynchronizować. Domyślnie wszystkie kolumny są zaznaczone, dlatego należy wyłączyć pole wyboru dla kolumn, które nie mają być synchronizowane. Pamiętaj, aby pozostawić zaznaczoną kolumnę klucza podstawowego.

1. Wybierz pozycję **Zapisz**.

1. Domyślnie bazy danych nie są synchronizowane do czasu zaplanowanego lub ręcznego uruchomienia. Aby uruchomić synchronizację ręczną, przejdź do bazy danych w SQL Database w Azure Portal, wybierz pozycję Synchronizuj z innymi bazami danych **i** wybierz grupę synchronizacji. Zostanie **Data Sync** strona aplikacji. Wybierz pozycję **Synchronizuj**.

    ![Synchronizacja ręczna](./media/sql-data-sync-sql-server-configure/datasync-sync.png)

## <a name="faq"></a>Często zadawane pytania

**Czy SQL Data Sync w pełni tworzyć tabele?**

Jeśli w docelowej bazie danych brakuje tabel schematu synchronizacji, usługa SQL Data Sync tworzy je z wybranymi kolumnami. Jednak nie spowoduje to schematu pełnej wierności z następujących powodów:

- W tabeli docelowej są tworzone tylko wybrane kolumny. Niewybrane kolumny są ignorowane.
- W tabeli docelowej są tworzone tylko wybrane indeksy kolumn. Indeksy niewybranych kolumn są ignorowane.
- Indeksy w kolumnach typu XML nie są tworzone.
- Ograniczenia CHECK nie są tworzone.
- Wyzwalacze w tabelach źródłowych nie są tworzone.
- Widoki i procedury składowane nie są tworzone.

Ze względu na te ograniczenia zalecamy następujące rozwiązania:

- W środowiskach produkcyjnych należy samodzielnie utworzyć schemat o pełnej wierności.
- Podczas eksperymentowania z usługą użyj funkcji automatycznego aprowizowania.

**Dlaczego widzę tabele, które nie są przeze mnie tworzyć?**

Data Sync tworzy dodatkowe tabele w bazie danych na celu śledzenie zmian. Nie usuwaj tych ani nie Data Sync przestanie działać.

**Czy moje dane są zbieżne po synchronizacji?**

Niekoniecznie. Weź grupę synchronizacji z piastą i trzema szprychami (A, B i C), w której synchronizacje mają grupę piasty do A, piasty do B i piasty do C. W przypadku zmiany bazy danych *A* po synchronizacji Centrum na A ta zmiana nie zostanie zapisywana w bazie danych B lub bazie danych C do następnego zadania synchronizacji.

**Jak mogę zmiany schematu w grupie synchronizacji?**

Ręcznie wprowadzaj i propaguj wszystkie zmiany schematu.

1. Zreplikuj zmiany schematu ręcznie do centrum i do wszystkich elementów członkowskich synchronizacji.
1. Zaktualizuj schemat synchronizacji.

W przypadku dodawania nowych tabel i kolumn:

Nowe tabele i kolumny nie mają wpływu na bieżącą synchronizację i Data Sync je ignorują, dopóki nie zostaną dodane do schematu synchronizacji. Podczas dodawania nowych obiektów bazy danych postępuj zgodnie z sekwencją:

1. Dodaj nowe tabele lub kolumny do centrum i do wszystkich elementów członkowskich synchronizacji.
1. Dodaj nowe tabele lub kolumny do schematu synchronizacji.
1. Rozpocznij wstawianie wartości do nowych tabel i kolumn.

Aby zmienić typ danych kolumny:

Po zmianie typu danych istniejącej kolumny Data Sync będzie działać tak długo, jak nowe wartości pasują do oryginalnego typu danych zdefiniowanego w schemacie synchronizacji. Jeśli na przykład zmienisz typ w źródłowej bazie danych z **int** na **bigint,** polecenie Data Sync będzie nadal działać, dopóki nie wstawisz zbyt dużej wartości dla typu danych **int.** Aby ukończyć zmianę, należy ręcznie zreplikować zmianę schematu do centrum i do wszystkich elementów członkowskich synchronizacji, a następnie zaktualizować schemat synchronizacji.

**Jak wyeksportować i zaimportować bazę danych za pomocą Data Sync?**

Po wyeksportowaniu bazy danych jako pliku *bacpac* i zaimportowaniu pliku w celu utworzenia bazy danych wykonaj następujące czynności, aby użyć Data Sync w nowej bazie danych:

1. Wyczyść obiekty Data Sync i dodatkowe tabele w nowej bazie danych przy użyciu [tego skryptu](https://github.com/vitomaz-msft/DataSyncMetadataCleanup/blob/master/Data%20Sync%20complete%20cleanup.sql). Skrypt usuwa wszystkie wymagane obiekty Data Sync z bazy danych.
1. Utwórz ponownie grupę synchronizacji z nową bazą danych. Jeśli stara grupa synchronizacji nie jest już potrzebna, usuń ją.

**Gdzie można znaleźć informacje na temat agenta klienta?**

Często zadawane pytania dotyczące agenta klienta można znaleźć w artykule Agent — często [zadawane pytania.](sql-data-sync-agent-overview.md#agent-faq)

**Czy konieczne jest ręczne zatwierdzenie linku przed rozpoczęciem korzystania z niego?**

Tak, należy ręcznie zatwierdzić zarządzany przez usługę prywatny punkt końcowy na stronie Połączenia prywatnego punktu końcowego usługi Azure Portal podczas wdrażania grupy synchronizacji lub przy użyciu programu PowerShell.

**Dlaczego otrzymuję błąd zapory, gdy zadanie synchronizacji aprowizuje bazę danych platformy Azure?**

Może się tak zdarzyć, ponieważ zasoby platformy Azure nie mogą uzyskać dostępu do serwera. Upewnij się, że zapora w bazie danych platformy Azure ma ustawienie "Zezwalaj usługom i zasobom platformy Azure na dostęp do tego serwera" ustawione na wartość "Tak".


## <a name="next-steps"></a>Następne kroki

Gratulacje. Utworzono grupę synchronizacji, która obejmuje zarówno wystąpienie SQL Database, jak i SQL Server bazę danych.

Aby uzyskać więcej informacji na temat usługi SQL Data Sync, zobacz:

- [Data Sync Agent for Azure SQL Data Sync](sql-data-sync-agent-overview.md)
- [Najlepsze rozwiązania i](sql-data-sync-best-practices.md) [jak rozwiązywać problemy z Azure SQL Data Sync](sql-data-sync-troubleshoot.md)
- [Monitorowanie SQL Data Sync za pomocą Azure Monitor dzienników](./monitor-tune-overview.md)
- [Aktualizowanie schematu synchronizacji za pomocą języka Transact-SQL](sql-data-sync-update-sync-schema.md) lub [programu PowerShell](scripts/update-sync-schema-in-sync-group.md)

Aby uzyskać więcej informacji na temat usługi SQL Database, zobacz:

- [Omówienie usługi SQL Database](sql-database-paas-overview.md)
- [Database Lifecycle Management (Zarządzanie cyklem życia bazy danych)](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))
