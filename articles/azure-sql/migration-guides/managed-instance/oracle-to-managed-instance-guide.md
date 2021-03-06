---
title: 'Wystąpienie zarządzane Oracle do usługi Azure SQL: Przewodnik migracji'
description: W tym przewodniku dowiesz się, jak migrować schematy programu Oracle do wystąpienia zarządzanego usługi Azure SQL przy użyciu Asystent migracji do programu SQL Server dla programu Oracle.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 2bb019a692178c5b44c3589d401d3b2b34c3dccb
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553927"
---
# <a name="migration-guide-oracle-to-azure-sql-managed-instance"></a>Przewodnik migracji: wystąpienie zarządzane programu Oracle do usługi Azure SQL

[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqlmi.md)]

W tym przewodniku dowiesz się, jak migrować schematy programu Oracle do wystąpienia zarządzanego usługi Azure SQL przy użyciu Asystent migracji do programu SQL Server dla programu Oracle (ASYSTENCIE migracji for Oracle).

Aby poznać inne przewodniki dotyczące migracji, zobacz [przewodniki dotyczące migracji bazy danych Azure](https://docs.microsoft.com/data-migration).

## <a name="prerequisites"></a>Wymagania wstępne

Przed rozpoczęciem migrowania schematu Oracle do wystąpienia zarządzanego SQL:

- Sprawdź, czy środowisko źródłowe jest obsługiwane.
- Pobierz [Asystencie migracji dla programu Oracle](https://www.microsoft.com/en-us/download/details.aspx?id=54258).
- Mieć obiekt docelowy [wystąpienia zarządzanego SQL](../../managed-instance/instance-create-quickstart.md) .
- Uzyskaj odpowiednie [uprawnienia dla Asystencie migracji dla programu Oracle](/sql/ssma/oracle/connecting-to-oracle-database-oracletosql) i [dostawcy](/sql/ssma/oracle/connect-to-oracle-oracletosql).
 
## <a name="pre-migration"></a>Przed migracją

Po spełnieniu wymagań wstępnych można przystąpić do odnajdywania topologii środowiska i oceny wykonalności migracji. Ta część procesu obejmuje przeprowadzenie spisu baz danych, które należy zmigrować, ocenianie tych baz danych pod kątem potencjalnych problemów z migracją lub blokowaniem, a następnie rozwiązanie wszystkich elementów, które mogły zostać odkryte.

### <a name="assess"></a>Ocena

Korzystając z programu ASYSTENCIE migracji for Oracle, można przeglądać obiekty bazy danych i dane, oceniać bazy danych do migracji, migrować obiekty bazy danych do wystąpienia zarządzanego SQL, a następnie na koniec migrować dane do bazy danych.

Aby utworzyć ocenę:

1. Otwórz [Asystencie migracji dla programu Oracle](https://www.microsoft.com/en-us/download/details.aspx?id=54258).
1. Wybierz pozycję **plik**, a następnie wybierz pozycję **Nowy projekt**.
1. Wprowadź nazwę projektu i lokalizację, w której ma zostać zapisany projekt. Następnie wybierz pozycję **wystąpienie zarządzane Azure SQL** jako miejsce docelowe migracji z listy rozwijanej i wybierz **przycisk OK**.

   ![Zrzut ekranu pokazujący nowy projekt.](./media/oracle-to-managed-instance-guide/new-project.png)

1. Wybierz pozycję **Połącz z bazą danych Oracle**. Wprowadź wartości w polu Szczegóły połączenia Oracle w oknie dialogowym **łączenie z programem Oracle** .

   ![Zrzut ekranu przedstawiający połączenie z bazą danych Oracle.](./media/oracle-to-managed-instance-guide/connect-to-oracle.png)

1. Wybierz schematy Oracle, które chcesz zmigrować.

   ![Zrzut ekranu pokazujący wybór schematu Oracle.](./media/oracle-to-managed-instance-guide/select-schema.png)

1. W **Eksploratorze metadanych Oracle** kliknij prawym przyciskiem myszy schemat Oracle, który chcesz zmigrować, a następnie wybierz polecenie **Utwórz raport** , aby wygenerować raport HTML. Alternatywnie możesz wybrać bazę danych, a następnie wybrać kartę **Utwórz raport** .

   ![Zrzut ekranu przedstawiający tworzenie raportu.](./media/oracle-to-managed-instance-guide/create-report.png)

1. Przejrzyj raport HTML, aby poznać statystyki konwersji oraz błędy lub ostrzeżenia. Możesz również otworzyć raport w programie Excel, aby uzyskać spis obiektów Oracle i nakład pracy wymagany do przeprowadzenia konwersji schematu. Domyślna lokalizacja raportu znajduje się w folderze raportów w SSMAProjects.

   Na przykład zobacz `drive:\<username>\Documents\SSMAProjects\MyOracleMigration\report\report_2020_11_12T02_47_55\`.

   ![Zrzut ekranu przedstawiający Raport z oceny.](./media/oracle-to-managed-instance-guide/assessment-report.png)

### <a name="validate-the-data-types"></a>Sprawdzanie poprawności typów danych

Sprawdź poprawność domyślnych mapowań typu danych i zmień je w zależności od wymagań, jeśli jest to konieczne. W tym celu wykonaj następujące czynności:

1. W programie ASYSTENCIE migracji for Oracle wybierz pozycję **Narzędzia**, a następnie wybierz pozycję **Ustawienia projektu**.
1. Wybierz kartę **Mapowanie typu** .

   ![Zrzut ekranu pokazujący mapowanie typu.](./media/oracle-to-managed-instance-guide/type-mappings.png)

1. Można zmienić mapowanie typu dla każdej tabeli, wybierając tabelę w **Eksploratorze metadanych Oracle**.

### <a name="convert-the-schema"></a>Konwertuj schemat

Aby skonwertować schemat:

1. Obowiązkowe Dodaj zapytania dynamiczne lub ad-hoc do instrukcji. Kliknij prawym przyciskiem myszy węzeł, a następnie wybierz polecenie **Dodaj instrukcje**.
1. Wybierz kartę **łączenie z wystąpieniem zarządzanym usługi Azure SQL** .
    1. Wprowadź szczegóły połączenia, aby połączyć swoją bazę danych w **wystąpieniu zarządzanym SQL Database**.
    1. Wybierz docelową bazę danych z listy rozwijanej lub wprowadź nową nazwę. w takim przypadku baza danych zostanie utworzona na serwerze docelowym.
    1. Wprowadź szczegóły uwierzytelniania i wybierz pozycję **Połącz**.

    ![Zrzut ekranu przedstawiający połączenie z wystąpieniem zarządzanym usługi Azure SQL.](./media/oracle-to-managed-instance-guide/connect-to-sql-managed-instance.png)

1. W **Eksploratorze metadanych Oracle** kliknij prawym przyciskiem myszy schemat Oracle, a następnie wybierz polecenie **Konwertuj schemat**. Alternatywnie możesz wybrać schemat, a następnie wybrać kartę **Konwertuj schemat** .

   ![Zrzut ekranu pokazujący schemat konwersji.](./media/oracle-to-managed-instance-guide/convert-schema.png)

1. Po zakończeniu konwersji Porównaj i przejrzyj przekonwertowane obiekty do oryginalnych obiektów, aby zidentyfikować potencjalne problemy i rozwiązać je na podstawie zaleceń.

   ![Zrzut ekranu pokazujący porównanie zaleceń dotyczących tabel.](./media/oracle-to-managed-instance-guide/table-comparison.png)

1. Porównaj przekonwertowany tekst języka Transact-SQL z oryginalnym kodem i zapoznaj się z zaleceniami.

   ![Zrzut ekranu pokazujący porównanie zaleceń dotyczących procedur.](./media/oracle-to-managed-instance-guide/procedure-comparison.png)

1. W okienku danych wyjściowych wybierz pozycję **Przejrzyj wyniki** i Przejrzyj błędy w okienku **Lista błędów** .
1. Zapisz projekt lokalnie dla ćwiczenia korygowania schematu w trybie offline. W menu **plik** wybierz polecenie **Zapisz projekt**. Ten krok umożliwia ocenę schematów źródłowych i docelowych w trybie offline oraz korygowanie danych przed opublikowaniem schematu w wystąpieniu zarządzanym bazy danych SQL.

## <a name="migrate"></a>Migrate

Po zakończeniu oceniania baz danych i rozwiązaniu jakichkolwiek rozbieżności następnym krokiem jest uruchomienie procesu migracji. Migracja obejmuje dwie czynności: publikowanie schematu i Migrowanie danych.

Aby opublikować schemat i zmigrować dane:
1. Opublikuj schemat, klikając prawym przyciskiem myszy bazę danych w węźle **bazy danych** w **Eksploratorze metadanych wystąpienia zarządzanego Azure SQL** i wybierając pozycję **Synchronizuj z bazą danych**.

   ![Zrzut ekranu pokazujący synchronizację z bazą danych.](./media/oracle-to-managed-instance-guide/synchronize-with-database.png)
   

1. Przejrzyj mapowanie między projektem źródłowym a obiektem docelowym.

   ![Zrzut ekranu pokazujący synchronizację z przeglądem bazy danych.](./media/oracle-to-managed-instance-guide/synchronize-with-database-review.png)

1. Migruj dane, klikając prawym przyciskiem myszy schemat lub obiekt, który chcesz zmigrować w **Eksploratorze metadanych Oracle** , i wybierając pozycję **Migruj dane**. Alternatywnie możesz wybrać kartę **Migrowanie danych** . Aby migrować dane dla całej bazy danych, zaznacz pole wyboru obok nazwy bazy danych. Aby przeprowadzić migrację danych z pojedynczych tabel, rozwiń bazę danych, rozwiń węzeł **tabele**, a następnie zaznacz pola wyboru obok tabel. Aby pominąć dane z poszczególnych tabel, wyczyść pola wyboru.

   ![Zrzut ekranu przedstawiający Migrowanie danych.](./media/oracle-to-managed-instance-guide/migrate-data.png)

1. Wprowadź szczegóły połączenia dla wystąpienia zarządzanego Oracle i SQL.
1. Po zakończeniu migracji Wyświetl **raport dotyczący migracji danych**.

   ![Zrzut ekranu przedstawiający raport dotyczący migracji danych.](./media/oracle-to-managed-instance-guide/data-migration-report.png)

1. Połącz się z wystąpieniem wystąpienia zarządzanego SQL przy użyciu [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)i Zweryfikuj migrację, przeglądając dane i schemat.

   ![Zrzut ekranu pokazujący weryfikację w programie ASYSTENCIE migracji for Oracle.](./media/oracle-to-managed-instance-guide/validate-data.png)

Alternatywnie można także przeprowadzić migrację za pomocą SQL Server Integration Services. Aby dowiedzieć się więcej, zobacz:

- [Wprowadzenie do SQL Server Integration Services](/sql/integration-services/sql-server-integration-services)
- [SQL Server Integration Services na platformę Azure i hybrydowe przenoszenie danych](https://download.microsoft.com/download/D/2/0/D20E1C5F-72EA-4505-9F26-FEF9550EFD44/SSIS%20Hybrid%20and%20Azure.docx)

## <a name="post-migration"></a>Po migracji

Po pomyślnym zakończeniu etapu *migracji* należy wykonać serię zadań po migracji, aby upewnić się, że wszystko działa jak najszybciej i efektywnie.

### <a name="remediate-applications"></a>Koryguj aplikacje

Po przeprowadzeniu migracji danych do środowiska docelowego wszystkie aplikacje, które wcześniej korzystały ze źródła, muszą zacząć zużywać miejsce docelowe. Wykonanie tego kroku wymaga wprowadzenia zmian w aplikacjach w niektórych przypadkach.

[Zestaw narzędzi do migracji dostępu do danych](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit) to rozszerzenie dla Visual Studio Code, które pozwala analizować kod źródłowy Java oraz wykrywać wywołania i zapytania interfejsu API dostępu do danych. Zestaw narzędzi zawiera jednookienkowy widok, w którym należy się rozmieścić, aby zapewnić obsługę nowego zaplecza bazy danych. Aby dowiedzieć się więcej, zobacz wpis w blogu [Migrowanie naszej aplikacji Java z firmy Oracle](https://techcommunity.microsoft.com/t5/microsoft-data-migration/migrate-your-java-applications-from-oracle-to-sql-server-with/ba-p/368727) .

### <a name="perform-tests"></a>Wykonaj testy

Podejście testowe do migracji bazy danych obejmuje następujące działania:

1. **Opracowywanie testów weryfikacyjnych**: Aby przetestować migrację bazy danych, należy użyć zapytań SQL. Należy utworzyć zapytania walidacji do uruchomienia względem źródłowej i docelowej bazy danych. Zapytania weryfikacyjne powinny obejmować zdefiniowany zakres.
2. **Konfigurowanie środowiska testowego**: środowisko testowe powinno zawierać kopię źródłowej bazy danych i docelowej bazy danych. Należy pamiętać o odizolowaniu środowiska testowego.
3. **Uruchom testy weryfikacyjne**: Uruchom testy weryfikacyjne względem źródła i celu, a następnie Przeanalizuj wyniki.
4. **Uruchom testy wydajnościowe**: Uruchom testy wydajności względem źródła i celu, a następnie Przeanalizuj i Porównaj wyniki.

### <a name="optimize"></a>Optymalizacja

Faza po migracji jest kluczowa do uzgadniania problemów z dokładnością danych, sprawdzania kompletności i rozwiązywania problemów z wydajnością w ramach obciążenia.

> [!NOTE]
> Więcej informacji o tych problemach i krokach, które należy rozwiązać, można znaleźć w [przewodniku po sprawdzeniu poprawności po migracji i optymalizacji](/sql/relational-databases/post-migration-validation-and-optimization-guide).

## <a name="migration-assets"></a>Zasoby migracji

Aby uzyskać więcej pomocy przy wykonywaniu tego scenariusza migracji, zobacz następujące zasoby. Zostały one opracowane w ramach obsługi rzeczywistego projektu migracji.

| **Tytuł/link**                                                                                                                                          | **Opis**                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Model i narzędzie oceny obciążenia danych](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool) | To narzędzie zapewnia sugerowane Platformy docelowe, gotowość chmury oraz poziom korygowania aplikacji lub bazy danych dla danego obciążenia. Oferuje proste, oparte na jednym kliknięcie Obliczanie i generowanie raportów, które ułatwiają przyspieszenie oceny dużych ilości, zapewniając zautomatyzowany i jednolity proces podejmowania decyzji platformy docelowej.                                                          |
| [Artefakty skryptu spisu firmy Oracle](https://github.com/Microsoft/DataMigrationTeam/tree/master/Oracle%20Inventory%20Script%20Artifacts)                 | Ten element zawartości zawiera zapytanie PL/SQL, które trafi w tabele systemowe Oracle i udostępnia liczbę obiektów według typu schematu, typu obiektu i stanu. Zapewnia także przybliżone oszacowanie danych pierwotnych w każdym schemacie oraz zmianę rozmiarów tabel w każdym schemacie, z wynikami przechowywanymi w formacie CSV.                                                                                                               |
| [Automatyzowanie & konsolidowania danych oceny rozwiązań firmy Oracle ASYSTENCIE migracji](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Automate%20SSMA%20Oracle%20Assessment%20Collection%20%26%20Consolidation)                                             | Ten zestaw zasobów używa pliku CSV jako wpisu (sources.csv w folderach projektu) do tworzenia plików XML, które są konieczne do uruchamiania oceny ASYSTENCIE migracji w trybie konsoli. source.csv jest udostępniana przez klienta w oparciu o spis istniejących wystąpień programu Oracle. Pliki wyjściowe to AssessmentReportGeneration_source_1.xml, ServersConnectionFile.xml i VariableValueFile.xml.|
| [ASYSTENCIE migracji dla typowych błędów Oracle i sposoby ich naprawiania](https://aka.ms/dmj-wp-ssma-oracle-errors)                                                           | Za pomocą programu Oracle można przypisać nieskalarny warunek w klauzuli WHERE. Jednak SQL Server nie obsługuje tego typu warunku. W związku z tym ASYSTENCIE migracji for Oracle nie konwertuje zapytań z nieskalarnym warunkiem w klauzuli WHERE. Zamiast tego generuje błąd O2SS0001. Ten oficjalny dokument zawiera więcej informacji o problemie i sposobach jego rozwiązania.          |
| [Podręcznik migracji firmy Oracle do SQL Server](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20SQL%20Server%20Migration%20Handbook.pdf)                | Ten dokument koncentruje się na zadaniach skojarzonych z migrowaniem schematu programu Oracle do najnowszej wersji bazy danych SQL Server Database. Jeśli migracja wymaga wprowadzenia zmian w funkcjach lub funkcjach, możliwym wpływem każdej zmiany w aplikacjach, które używają bazy danych, należy rozważyć ostrożnie.                                                     |

Zespół inżynierów danych SQL Data opracował te zasoby. Podstawowa karta tego zespołu ma odblokować i przyspieszyć kompleksową modernizację projektów migracji platformy danych do platformy danych platformy Microsoft Azure.

## <a name="next-steps"></a>Następne kroki

- W przypadku macierzy usług i narzędzi firmy Microsoft i innych firm, które są dostępne, aby pomóc w różnych scenariuszach migracji bazy danych i danych oraz zadaniach specjalnych, zobacz temat [usługi i narzędzia do migracji danych](../../../dms/dms-tools-matrix.md).

- Aby dowiedzieć się więcej o wystąpieniu zarządzanym SQL, zobacz:
  - [Omówienie wystąpienia zarządzanego usługi Azure SQL](../../managed-instance/sql-managed-instance-paas-overview.md)
  - [Kalkulator całkowitego kosztu posiadania (TCO) na platformie Azure](https://azure.microsoft.com/en-us/pricing/tco/calculator/)

- Aby dowiedzieć się więcej na temat cyklu i wdrożenia migracji do chmury, zobacz:
   -  [Przewodnik Cloud Adoption Framework dla platformy Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Najlepsze rozwiązania dotyczące kosztów i rozmiarów obciążeń na potrzeby migracji na platformę Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs)

- Aby uzyskać zawartość wideo, zobacz:
    - [Przegląd podróży migracji i narzędzi i usług zalecanych do przeprowadzania oceny i migracji](https://azure.microsoft.com/resources/videos/overview-of-migration-and-recommended-tools-services/)
