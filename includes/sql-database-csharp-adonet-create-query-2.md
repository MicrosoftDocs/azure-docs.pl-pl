---
author: rothja
ms.author: jroth
ms.service: sql-database
ms.topic: include
ms.date: 12/10/2018
ms.openlocfilehash: 5509c640fbf232cb6c613d96c30cb0e8d9d10ec5
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 04/07/2021
ms.locfileid: "106551694"
---
## <a name="c-program-example"></a>Przykład programu w języku C#

Kolejne sekcje tego artykułu zawierają program języka C#, który używa ADO.NET do wysyłania instrukcji Transact-SQL (T-SQL) do SQL Database. Program w języku C# prezentuje następujące akcje:

- [Nawiązywanie połączenia z SQL Database przy użyciu ADO.NET](#cs_1_connect)
- [Metody, które zwracają instrukcje języka T-SQL](#cs_2_return)
    - Tworzenie tabel
    - Wypełnianie tabel danymi
    - Aktualizowanie, usuwanie i wybieranie danych
- [Przesyłanie kodu języka T-SQL do bazy danych](#cs_3_submit)

### <a name="entity-relationship-diagram-erd"></a>Diagram relacji jednostek

Wcześniejsze instrukcje `CREATE TABLE` zawierają słowo kluczowe **REFERENCES** w celu utworzenia relacji między dwiema tabelami za pomocą *klucza obcego*. Jeśli używasz bazy danych *tempdb*, wykomentuj słowo kluczowe `--REFERENCES`, wpisując na początku dwa łączniki.

Ten diagram relacji jednostek przedstawia relację między dwiema tabelami. Wartości w kolumnie *podrzędnej* **tabEmployee. DepartmentCode** są ograniczone do wartości z kolumny *nadrzędnej* **tabDepartment. DepartmentCode** .

![Diagram relacji jednostek pokazujący klucz obcy](./media/sql-database-csharp-adonet-create-query-2/erd-dept-empl-fky-2.png)

> [!NOTE]
> Istnieje możliwość edytowania kodu w języku T-SQL w celu dodania znaku `#` na początku nazw tabel, co powoduje ich utworzenie jako tabel tymczasowych w *bazie danych tempdb*. Jest to przydatne dla celów demonstracyjnych, gdy nie jest dostępna testowa baza danych. Podczas ich używania nie są wymuszane żadne odwołania do kluczy obcych. Ponadto tabele tymczasowe są usuwane automatycznie, gdy połączenie zostanie zamknięte po zakończeniu działania programu.

### <a name="to-compile-and-run"></a>Kompilowanie i uruchamianie

Program w języku C# jest logicznie jednym plikiem cs, ale fizycznie jest podzielony na kilka bloków kodu, aby poprawić czytelność. Aby skompilować i uruchomić program, wykonaj następujące czynności:

1. Utwórz projekt w języku C# w programie Visual Studio. Typ projektu powinien mieć wartość *Konsola*, którą można znaleźć w obszarze **Szablony** > **Visual C#** > **Aplikacja klasyczna systemu Windows** > **Aplikacja konsoli (.NET Framework)**.

1. W pliku *Program.cs* zamień startowe wiersze kodu, wykonując następujące czynności:

    1. Skopiuj i wklej poniższe bloki kodu w tej samej kolejności, w której są zaprezentowane, zobacz sekcje [Połączenie z bazą danych](#cs_1_connect), [Generowanie kodu języka T-SQL](#cs_2_return) i [Przesyłanie do bazy danych](#cs_3_submit).

    1. Zmień następujące wartości w metodzie `Main`:

        - *cb.DataSource*
        - *cb.UserID*
        - *cb.Password*
        - *cb.InitialCatalog*

1. Upewnij się, że przywoływany jest zestaw *System.Data.dll*. Aby to sprawdzić, rozwiń węzeł **Odwołania** w okienku **Eksplorator rozwiązań**.

1. Aby skompilować i uruchomić program z poziomu programu Visual Studio, kliknij przycisk **Uruchom**. Dane wyjściowe raportu są wyświetlane w oknie programu, a wartości identyfikatora GUID będą różnić się w różnych przebiegach testu.

    ```Output
    =================================
    T-SQL to 2 - Create-Tables...
    -1 = rows affected.

    =================================
    T-SQL to 3 - Inserts...
    8 = rows affected.

    =================================
    T-SQL to 4 - Update-Join...
    2 = rows affected.

    =================================
    T-SQL to 5 - Delete-Join...
    2 = rows affected.

    =================================
    Now, SelectEmployees (6)...
    8ddeb8f5-9584-4afe-b7ef-d6bdca02bd35 , Alison , 20 , acct , Accounting
    9ce11981-e674-42f7-928b-6cc004079b03 , Barbara , 17 , hres , Human Resources
    315f5230-ec94-4edd-9b1c-dd45fbb61ee7 , Carol , 22 , acct , Accounting
    fcf4840a-8be3-43f7-a319-52304bf0f48d , Elle , 15 , NULL , NULL
    View the report output here, then press any key to end the program...
    ```

<a name="cs_1_connect"></a>

### <a name="connect-to-sql-database-using-adonet"></a>Nawiązywanie połączenia z SQL Database przy użyciu ADO.NET

```csharp
using System;
using System.Data.SqlClient;   // System.Data.dll
//using System.Data;           // For:  SqlDbType , ParameterDirection

namespace csharp_db_test
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                var cb = new SqlConnectionStringBuilder();
                cb.DataSource = "your_server.database.windows.net";
                cb.UserID = "your_user";
                cb.Password = "your_password";
                cb.InitialCatalog = "your_database";

                using (var connection = new SqlConnection(cb.ConnectionString))
                {
                    connection.Open();

                    Submit_Tsql_NonQuery(connection, "2 - Create-Tables", Build_2_Tsql_CreateTables());

                    Submit_Tsql_NonQuery(connection, "3 - Inserts", Build_3_Tsql_Inserts());

                    Submit_Tsql_NonQuery(connection, "4 - Update-Join", Build_4_Tsql_UpdateJoin(),
                        "@csharpParmDepartmentName", "Accounting");

                    Submit_Tsql_NonQuery(connection, "5 - Delete-Join", Build_5_Tsql_DeleteJoin(),
                        "@csharpParmDepartmentName", "Legal");

                    Submit_6_Tsql_SelectEmployees(connection);
                }
            }
            catch (SqlException e)
            {
                Console.WriteLine(e.ToString());
            }

            Console.WriteLine("View the report output here, then press any key to end the program...");
            Console.ReadKey();
        }
```

<a name="cs_2_return"></a>

### <a name="methods-that-return-t-sql-statements"></a>Metody, które zwracają instrukcje języka T-SQL

```csharp
static string Build_2_Tsql_CreateTables()
{
    return @"
        DROP TABLE IF EXISTS tabEmployee;
        DROP TABLE IF EXISTS tabDepartment;  -- Drop parent table last.

        CREATE TABLE tabDepartment
        (
            DepartmentCode  nchar(4)          not null    PRIMARY KEY,
            DepartmentName  nvarchar(128)     not null
        );

        CREATE TABLE tabEmployee
        (
            EmployeeGuid    uniqueIdentifier  not null  default NewId()    PRIMARY KEY,
            EmployeeName    nvarchar(128)     not null,
            EmployeeLevel   int               not null,
            DepartmentCode  nchar(4)              null
            REFERENCES tabDepartment (DepartmentCode)  -- (REFERENCES would be disallowed on temporary tables.)
        );
    ";
}

static string Build_3_Tsql_Inserts()
{
    return @"
        -- The company has these departments.
        INSERT INTO tabDepartment (DepartmentCode, DepartmentName)
        VALUES
            ('acct', 'Accounting'),
            ('hres', 'Human Resources'),
            ('legl', 'Legal');

        -- The company has these employees, each in one department.
        INSERT INTO tabEmployee (EmployeeName, EmployeeLevel, DepartmentCode)
        VALUES
            ('Alison'  , 19, 'acct'),
            ('Barbara' , 17, 'hres'),
            ('Carol'   , 21, 'acct'),
            ('Deborah' , 24, 'legl'),
            ('Elle'    , 15, null);
    ";
}

static string Build_4_Tsql_UpdateJoin()
{
    return @"
        DECLARE @DName1  nvarchar(128) = @csharpParmDepartmentName;  --'Accounting';

        -- Promote everyone in one department (see @parm...).
        UPDATE empl
        SET
            empl.EmployeeLevel += 1
        FROM
            tabEmployee   as empl
        INNER JOIN
            tabDepartment as dept ON dept.DepartmentCode = empl.DepartmentCode
        WHERE
            dept.DepartmentName = @DName1;
    ";
}

static string Build_5_Tsql_DeleteJoin()
{
    return @"
        DECLARE @DName2  nvarchar(128);
        SET @DName2 = @csharpParmDepartmentName;  --'Legal';

        -- Right size the Legal department.
        DELETE empl
        FROM
            tabEmployee   as empl
        INNER JOIN
            tabDepartment as dept ON dept.DepartmentCode = empl.DepartmentCode
        WHERE
            dept.DepartmentName = @DName2

        -- Disband the Legal department.
        DELETE tabDepartment
            WHERE DepartmentName = @DName2;
    ";
}

static string Build_6_Tsql_SelectEmployees()
{
    return @"
        -- Look at all the final Employees.
        SELECT
            empl.EmployeeGuid,
            empl.EmployeeName,
            empl.EmployeeLevel,
            empl.DepartmentCode,
            dept.DepartmentName
        FROM
            tabEmployee   as empl
        LEFT OUTER JOIN
            tabDepartment as dept ON dept.DepartmentCode = empl.DepartmentCode
        ORDER BY
            EmployeeName;
    ";
}
```

<a name="cs_3_submit"></a>

### <a name="submit-t-sql-to-the-database"></a>Przesyłanie kodu języka T-SQL do bazy danych

```csharp
static void Submit_6_Tsql_SelectEmployees(SqlConnection connection)
{
    Console.WriteLine();
    Console.WriteLine("=================================");
    Console.WriteLine("Now, SelectEmployees (6)...");

    string tsql = Build_6_Tsql_SelectEmployees();

    using (var command = new SqlCommand(tsql, connection))
    {
        using (SqlDataReader reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine("{0} , {1} , {2} , {3} , {4}",
                    reader.GetGuid(0),
                    reader.GetString(1),
                    reader.GetInt32(2),
                    (reader.IsDBNull(3)) ? "NULL" : reader.GetString(3),
                    (reader.IsDBNull(4)) ? "NULL" : reader.GetString(4));
            }
        }
    }
}

static void Submit_Tsql_NonQuery(
    SqlConnection connection,
    string tsqlPurpose,
    string tsqlSourceCode,
    string parameterName = null,
    string parameterValue = null
    )
{
    Console.WriteLine();
    Console.WriteLine("=================================");
    Console.WriteLine("T-SQL to {0}...", tsqlPurpose);

    using (var command = new SqlCommand(tsqlSourceCode, connection))
    {
        if (parameterName != null)
        {
            command.Parameters.AddWithValue(  // Or, use SqlParameter class.
                parameterName,
                parameterValue);
        }
        int rowsAffected = command.ExecuteNonQuery();
        Console.WriteLine(rowsAffected + " = rows affected.");
    }
}
} // EndOfClass
}
```