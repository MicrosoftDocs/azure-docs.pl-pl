---
title: 'Samouczek: masowe wyodrębnianie danych formularza przy użyciu Azure Data Factory'
titleSuffix: Azure Cognitive Services
description: Skonfiguruj działania Azure Data Factory, aby wyzwolić szkolenia i uruchamiać modele aparatów rozpoznawania formularzy, oraz cyfrowe, Duże zaległości dokumentów.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 01/04/2021
ms.author: lajanuar
ms.openlocfilehash: 0c009a87a5834997cdc489efc75ebb16f9459754
ms.sourcegitcommit: 3ea12ce4f6c142c5a1a2f04d6e329e3456d2bda5
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/15/2021
ms.locfileid: "103467106"
---
# <a name="tutorial-extract-form-data-in-bulk-by-using-azure-data-factory"></a>Samouczek: masowe wyodrębnianie danych formularza przy użyciu Azure Data Factory

W tym samouczku zawarto informacje na temat korzystania z usług platformy Azure w celu pozyskiwania dużych partii formularzy na nośniku cyfrowym. W tym samouczku przedstawiono sposób automatyzacji pozyskiwania danych z usługi Azure Data Lake Documents w usłudze Azure SQL Database. Będziesz w stanie szybko przeszkolić modele i przetwarzać nowe dokumenty za pomocą kilku kliknięć.

## <a name="business-need"></a>Potrzeby biznesowe

Większość organizacji ma teraz świadomość wartości danych przechowywanych w różnych formatach (PDF, obrazy, wideo). Poszukują najlepszych rozwiązań i najbardziej opłacalnych metod cyfrowego przesyłania tych zasobów.

Ponadto nasi klienci często mają różne typy formularzy, które pochodzą od wielu klientów i klientów. W przeciwieństwie do [przewodników szybki start](./quickstarts/client-library.md)w tym samouczku pokazano, jak automatycznie uczenie modelu z nowymi i różnymi typami formularzy przy użyciu metody opartej na metadanych. Jeśli nie masz istniejącego modelu dla danego typu formularza, system utworzy jeden i przekaże Identyfikator modelu. 

Przez wyodrębnienie danych z formularzy i połączenie ich z istniejącymi systemami operacyjnymi i magazynami danych firmy mogą uzyskiwać wgląd i dostarczać korzyści klientom i użytkownikom biznesowym.

Aparat rozpoznawania formularzy platformy Azure ułatwia organizacjom korzystanie z danych, automatyzowanie procesów (płatności faktury, przetwarzanie podatków itd.), oszczędność pieniędzy i czasu oraz lepszą dokładność danych.

Ten samouczek zawiera informacje na temat wykonywania następujących czynności:

> [!div class="checklist"]
> * Skonfiguruj Azure Data Lake do przechowywania formularzy.
> * Użyj usługi Azure Database, aby utworzyć tabelę parametrization.
> * Użyj Azure Key Vault, aby przechowywać poufne poświadczenia.
> * Przeszkol model aparatu rozpoznawania formularzy w notesie Azure Databricks.
> * Wyodrębnij dane formularza przy użyciu notesu datacegły.
> * Automatyzacja szkoleń i wyodrębniania formularzy przy użyciu Azure Data Factory.

## <a name="prerequisites"></a>Wymagania wstępne

* Subskrypcja platformy Azure. [Utwórz je bezpłatnie](https://azure.microsoft.com/free/cognitive-services/).
* Po utworzeniu subskrypcji platformy Azure <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title=" Utwórz zasób aparatu rozpoznawania formularzy "  target="_blank"> Utwórz zasób aparatu rozpoznawania formularza </a> w Azure Portal, aby uzyskać klucz i punkt końcowy. Po wdrożeniu zasobu wybierz pozycję **Przejdź do zasobu**.
    * Będziesz potrzebować klucza i punktu końcowego z zasobu, który utworzysz, aby połączyć aplikację z interfejsem API rozpoznawania formularzy. Klucz i punkt końcowy zostaną wklejone do kodu w dalszej części tego przewodnika Szybki Start.
    * Aby wypróbować usługę, możesz skorzystać z warstwy cenowej bezpłatna (F0). Następnie można przeprowadzić uaktualnienie później do warstwy płatnej dla środowiska produkcyjnego.
* Zestaw składający się z co najmniej pięciu form tego samego typu. W idealnym przypadku ten przepływ pracy jest przeznaczony do obsługi dużych zestawów dokumentów. Zobacz [Tworzenie zestawu danych szkoleniowych](./build-training-data-set.md) dla wskazówek i opcji tworzenia zestawu danych szkoleniowych. W tym samouczku można użyć plików w folderze uczenie [przykładowego zestawu danych](https://go.microsoft.com/fwlink/?linkid=2128080).


## <a name="project-architecture"></a>Architektura projektu 

Ten projekt przedstawia zestaw potoków Azure Data Factory, aby wyzwolić notesy języka Python, które pociągną, analizują i wyodrębniają dane z dokumentów na koncie magazynu Azure Data Lake.

Interfejs API REST aparatu rozpoznawania formularzy wymaga pewnych parametrów jako danych wejściowych. Ze względów bezpieczeństwa niektóre z tych parametrów będą przechowywane w magazynie kluczy platformy Azure. Inne, mniej poufne parametry, takie jak nazwa folderu obiektów blob magazynu, będą przechowywane w tabeli parametryzacja w bazie danych SQL Azure.

Dla każdego typu formularza, który ma być analizowany, inżynierowie danych lub analityki danych wypełniają wiersz tabeli parametrów. Następnie użyją Azure Data Factory do iteracji na liście wykrytych typów formularzy i przekazania odpowiednich parametrów do notesu datacegły, aby szkolić lub ponownie szkolić modele aparatów rozpoznawania formularzy. W tym miejscu można również użyć funkcji platformy Azure.

W notesie Azure Databricks są następnie wykorzystywane przeszkolone modele do wyodrębnienia danych formularza. Eksportuje te dane do usługi Azure SQL Database.

:::image type="content" source="./media/tutorial-bulk-processing/architecture.png" alt-text="Diagram przedstawiający architekturę projektu.":::


## <a name="set-up-azure-data-lake"></a>Skonfiguruj Azure Data Lake

Zaległości formularzy mogą znajdować się w środowisku lokalnym lub na serwerze FTP. Ten samouczek używa formularzy na koncie Azure Data Lake Storage Gen2. Pliki można przenieść za pomocą Azure Data Factory, Eksplorator usługi Azure Storage lub AzCopy. Zestawy danych szkoleniowych i oceniających mogą znajdować się w różnych kontenerach, ale zestawy danych do szkolenia dla wszystkich typów formularzy muszą znajdować się w tym samym kontenerze. (Mogą znajdować się w różnych folderach).

Aby utworzyć nową usługą Data Lake, postępuj zgodnie z instrukcjami w temacie [Tworzenie konta magazynu do użycia z Azure Data Lake Storage Gen2](../../storage/blobs/create-data-lake-storage-account.md).

## <a name="create-a-parameterization-table"></a>Tworzenie tabeli parametryzacja

Następnie utworzymy tabelę metadanych w bazie danych SQL Azure. Ta tabela będzie zawierać dane niepoufne wymagane przez interfejs API REST aparatu rozpoznawania formularzy. Za każdym razem, gdy w zestawie danych jest nowy typ formularza, w tej tabeli zostanie wstawiony nowy rekord i zostanie wyzwolone potoki szkoleń i oceniania. (Te potoki zostaną zaimplementowane później).

Te pola zostaną użyte w tabeli:

* **form_description**. Niewymagane w ramach szkolenia. To pole zawiera opis typu formularzy, które są szkoleniowe dla modelu (na przykład "klient A formularze", "formularze hotelu B").
* **training_container_name**. Nazwa kontenera konta magazynu, w którym zapisano zestaw danych szkoleniowych. Może to być ten sam kontener co **scoring_container_name**.
* **training_blob_root_folder**. Folder na koncie magazynu, w którym będą przechowywane pliki do szkolenia modelu.
* **scoring_container_name**. Nazwa kontenera konta magazynu, w którym przechowywane są pliki, z których mają zostać wyodrębnione pary klucz/wartość. Może to być ten sam kontener co **training_container_name**.
* **scoring_input_blob_folder**. Folder na koncie magazynu, w którym będą przechowywane pliki do wyodrębnienia danych.
* **model_id**. Identyfikator modelu, który ma zostać przemieszczony ponownie. Dla pierwszego uruchomienia należy ustawić wartość-1, co spowoduje, że Notes szkoleniowy utworzy nowy model niestandardowy do uczenia. Notes szkoleniowy zwróci nowy identyfikator modelu do wystąpienia Azure Data Factory. Korzystając z działania procedury składowanej, zaktualizujemy tę wartość w usłudze Azure SQL Database.

  W każdym przypadku, gdy chcesz uzyskać nowy typ formularza, musisz ręcznie zresetować Identyfikator modelu do-1 przed nauczeniem modelu.

* **file_type**. Obsługiwane typy formularzy to `application/pdf` , `image/jpeg` , `image/png` , i `image/tif` .

  Jeśli masz formularze w innych typach plików, musisz zmienić tę wartość i **model_id** podczas uczenia nowego typu formularza.
* **form_batch_group_id**. W miarę upływu czasu może istnieć wiele typów formularzy, które są naliczane względem tego samego modelu. Pole **form_batch_group_id** umożliwi określenie wszystkich typów formularzy, które zostały szkoleniowe z określonym modelem.

### <a name="create-the-table"></a>Tworzenie tabeli


[Utwórz bazę danych Azure SQL](https://ms.portal.azure.com/#create/Microsoft.SQLDatabase), a następnie uruchom ten skrypt SQL w [Edytorze zapytań](../../azure-sql/database/connect-query-portal.md) , aby utworzyć wymaganą tabelę:


```sql
CREATE TABLE dbo.ParamFormRecogniser(
    form_description varchar(50) NULL,
  training_container_name varchar(50) NOT NULL,
    training_blob_root_folder varchar(50) NULL,
    scoring_container_name varchar(50) NOT NULL,
    scoring_input_blob_folder varchar(50) NOT NULL,
    scoring_output_blob_folder varchar(50) NOT NULL,
    model_id varchar(50) NULL,
    file_type varchar(50) NULL
) ON PRIMARY
GO
```

Uruchom ten skrypt, aby utworzyć procedurę, która automatycznie aktualizuje **model_id** po przeszkoleniu.

```SQL
CREATE PROCEDURE [dbo].[update_model_id] ( @form_batch_group_id  varchar(50),@model_id varchar(50))
AS
BEGIN 
    UPDATE [dbo].[ParamFormRecogniser]   
        SET [model_id] = @model_id  
    WHERE form_batch_group_id =@form_batch_group_id
END
```

## <a name="use-azure-key-vault-to-store-sensitive-credentials"></a>Używanie Azure Key Vault do przechowywania poufnych poświadczeń

Ze względów bezpieczeństwa nie chcemy przechowywać pewnych poufnych informacji w tabeli parametryzacja w bazie danych SQL Azure. Będziemy przechowywać poufne parametry jako Azure Key Vault wpisy tajne.

### <a name="create-an-azure-key-vault"></a>Tworzenie magazynu kluczy platformy Azure

[Utwórz zasób Key Vault](https://ms.portal.azure.com/#create/Microsoft.KeyVault). Następnie przejdź do zasobu Key Vault po jego utworzeniu i w sekcji **Ustawienia** wybierz pozycję wpisy **tajne** , aby dodać parametry.

Zostanie wyświetlone nowe okno. Wybierz pozycję **Generuj/Importuj**. Wprowadź nazwę parametru i jego wartość, a następnie wybierz pozycję **Utwórz**. Wykonaj te kroki dla następujących parametrów:

* **CognitiveServiceEndpoint**. Adres URL punktu końcowego interfejsu API rozpoznawania formularza.
* **CognitiveServiceSubscriptionKey**. Klucz dostępu dla usługi rozpoznawania formularzy. 
* **StorageAccountName**. Konto magazynu, w którym są przechowywane zestawy danych szkoleniowych i formularzy, z których mają zostać wyodrębnione pary klucz/wartość. Jeśli te elementy znajdują się na różnych kontach, wprowadź nazwę każdego konta jako odrębne hasło. Należy pamiętać, że zestawy danych szkoleniowe muszą znajdować się w tym samym kontenerze dla wszystkich typów formularzy. Mogą one znajdować się w różnych folderach.
* **StorageAccountSasKey**. Sygnatura dostępu współdzielonego (SAS) konta magazynu. Aby pobrać adres URL sygnatury dostępu współdzielonego, przejdź do zasobu magazynu. Na karcie **Eksplorator usługi Storage** przejdź do swojego kontenera, kliknij prawym przyciskiem myszy, a następnie wybierz pozycję **Pobierz sygnaturę dostępu współdzielonego**. Ważne jest, aby uzyskać sygnaturę dostępu współdzielonego dla kontenera, a nie dla samego konta magazynu. Upewnij się, że uprawnienia do **odczytu** i **listy** są zaznaczone, a następnie wybierz pozycję **Utwórz**. Następnie skopiuj wartość z sekcji **URL** . Powinien on mieć następującą formę: `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>` .

## <a name="train-your-form-recognizer-model-in-a-databricks-notebook"></a>Uczenie modelu aparatu rozpoznawania formularzy w notesie datacegły

Będziesz używać Azure Databricks do przechowywania i uruchamiania kodu języka Python, który współdziała z usługą rozpoznawania formularzy.

### <a name="create-a-notebook-in-databricks"></a>Tworzenie notesu w kostkach

[Utwórz zasób Azure Databricks](https://ms.portal.azure.com/#create/Microsoft.Databricks) w Azure Portal. Przejdź do zasobu po jego utworzeniu i Otwórz obszar roboczy.

### <a name="create-a-secret-scope-backed-by-azure-key-vault"></a>Utwórz zakres tajny, którego kopia zapasowa ma Azure Key Vault


Aby odwołać się do wpisów tajnych w magazynie kluczy platformy Azure utworzonym powyżej, musisz utworzyć zakres tajny w kostkach danych. Wykonaj kroki opisane tutaj: [Utwórz zakres tajny Azure Key Vault-kopia zapasowa](/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope).

### <a name="create-a-databricks-cluster"></a>Tworzenie klastra usługi Databricks

Klaster jest kolekcją zasobów obliczeniowych. Aby utworzyć klaster:

1. W lewym okienku wybierz przycisk **klastry** .
1. Na stronie **klastry** wybierz pozycję **Utwórz klaster**.
1. Na stronie **Tworzenie klastra** Określ nazwę klastra, a następnie wybierz pozycję **7,2 (Scala 2,12, Spark 3.0.0)** na liście **wersja Databricks Runtime** .
1. Wybierz pozycję **Utwórz klaster**.

### <a name="write-a-settings-notebook"></a>Napisz Notes ustawień

Teraz możesz dodać notesy języka Python. Najpierw utwórz Notes o nazwie **Settings (ustawienia**). Ten Notes spowoduje przypisanie wartości w tabeli parametryzacja do zmiennych w skrypcie. Azure Data Factory będzie później przekazać wartości w postaci parametrów. Dodatkowo przypiszemy wartości z wpisów tajnych w magazynie kluczy do zmiennych. 

1. Aby utworzyć Notes **ustawień** , wybierz przycisk **obszaru roboczego** . Na nowej karcie Wybierz listę rozwijaną i wybierz pozycję **Utwórz** , a następnie **Notes**.
1. W oknie podręcznym wprowadź nazwę notesu, a następnie wybierz opcję **Python** jako język domyślny. Wybierz klaster datakostki, a następnie wybierz pozycję **Utwórz**.
1. W pierwszej komórce notesu pobieramy parametry przesłane przez Azure Data Factory:

    ```python 
    dbutils.widgets.text("form_batch_group_id", "","")
    dbutils.widgets.get("form_batch_group_id")
    form_batch_group_id = getArgument("form_batch_group_id")
    
    dbutils.widgets.text("model_id", "","")
    dbutils.widgets.get("model_id")
    model_id = getArgument("model_id")
    
    dbutils.widgets.text("training_container_name", "","")
    dbutils.widgets.get("training_container_name")
    training_container_name = getArgument("training_container_name")
    
    dbutils.widgets.text("training_blob_root_folder", "","")
    dbutils.widgets.get("training_blob_root_folder")
    training_blob_root_folder=  getArgument("training_blob_root_folder")
    
    dbutils.widgets.text("scoring_container_name", "","")
    dbutils.widgets.get("scoring_container_name")
    scoring_container_name=  getArgument("scoring_container_name")
    
    dbutils.widgets.text("scoring_input_blob_folder", "","")
    dbutils.widgets.get("scoring_input_blob_folder")
    scoring_input_blob_folder=  getArgument("scoring_input_blob_folder")
    
    
    dbutils.widgets.text("file_type", "","")
    dbutils.widgets.get("file_type")
    file_type = getArgument("file_type")
    
    dbutils.widgets.text("file_to_score_name", "","")
    dbutils.widgets.get("file_to_score_name")
    file_to_score_name=  getArgument("file_to_score_name")
    ```

1. W drugiej komórce pobieramy wpisy tajne z Key Vault i przypisujemy je do zmiennych:

    ```python 
    cognitive_service_subscription_key = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveserviceSubscriptionKey")
    cognitive_service_endpoint = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveServiceEndpoint")
    
    training_storage_account_name = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountName")
    storage_account_sas_key= dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountSasKey")  
    
    ScoredFile = file_to_score_name+ "_output.json"
    training_storage_account_url="https://"+training_storage_account_name+".blob.core.windows.net/"+training_container_name+storage_account_sas_key 
    ```

### <a name="write-a-training-notebook"></a>Napisz Notes szkoleniowy

Po ukończeniu pracy notesu **ustawień** możemy utworzyć Notes do uczenia modelu. Jak wspomniano powyżej, użyjemy plików przechowywanych w folderze na koncie Azure Data Lake Storage Gen2 (**training_blob_root_folder**). Nazwa folderu została przeniesiona jako zmienna. Każdy zestaw typów formularzy będzie znajdować się w tym samym folderze. Podczas pętli w tabeli parametrów będziemy szkolić model przy użyciu wszystkich typów formularzy. 

1. Utwórz Notes o nazwie **TrainFormRecognizer**. 
1. W pierwszej komórce Uruchom Notes **ustawień** :

    ```python
    %run "./Settings"
    ```

1. W następnej komórce Przypisz zmienne z pliku ustawień i dynamicznie Przeszkol model dla każdego typu formularza, stosując kod w [przewodniku szybki start](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-train-extract.md#get-training-results%20)dla usługi REST.

    ```python
    import json
    import time
    from requests import get, post
    
    post_url = cognitive_service_endpoint + r"/formrecognizer/v2.0/custom/models"
    source = training_storage_account_url
    prefix= training_blob_root_folder
    
    includeSubFolders=True
    useLabelFile=False
    headers = {
        # Request headers.
        'Content-Type': file_type,
        'Ocp-Apim-Subscription-Key': cognitive_service_subscription_key,
    }
    body =     {
        "source": source
        ,"sourceFilter": {
            "prefix": prefix,
            "includeSubFolders": includeSubFolders
       },
    }
    if model_id=="-1": # If you don't already have a model you want to retrain. In this case, we create a model and use it to extract the key/value pairs.
      try:
          resp = post(url = post_url, json = body, headers = headers)
          if resp.status_code != 201:
              print("POST model failed (%s):\n%s" % (resp.status_code, json.dumps(resp.json())))
              quit()
          print("POST model succeeded:\n%s" % resp.headers)
          get_url = resp.headers["location"]
          model_id=get_url[get_url.index('models/')+len('models/'):]     
          
      except Exception as e:
          print("POST model failed:\n%s" % str(e))
          quit()
    else :# If you already have a model you want to retrain, we reuse it and (re)train with the new form types.  
      try:
        get_url =post_url+r"/"+model_id
          
      except Exception as e:
          print("POST model failed:\n%s" % str(e))
          quit()
    ```

1. Ostatnim krokiem w procesie szkolenia jest uzyskanie wyniku szkolenia w formacie JSON:

    ```python
    n_tries = 10
    n_try = 0
    wait_sec = 5
    max_wait_sec = 5
    while n_try < n_tries:
        try:
            resp = get(url = get_url, headers = headers)
            resp_json = resp.json()
            print (resp.status_code)
            if resp.status_code != 200:
                print("GET model failed (%s):\n%s" % (resp.status_code, json.dumps(resp_json)))
                n_try += 1
                quit()
            model_status = resp_json["modelInfo"]["status"]
            print (model_status)
            if model_status == "ready":
                print("Training succeeded:\n%s" % json.dumps(resp_json))
                n_try += 1
                quit()
            if model_status == "invalid":
                print("Training failed. Model is invalid:\n%s" % json.dumps(resp_json))
                n_try += 1
                quit()
            # Training still running. Wait and retry.
            time.sleep(wait_sec)
            n_try += 1
            wait_sec = min(2*wait_sec, max_wait_sec)     
            print (n_try)
        except Exception as e:
            msg = "GET model failed:\n%s" % str(e)
            print(msg)
            quit()
    print("Train operation did not complete within the allocated time.")
    ```

## <a name="extract-form-data-by-using-a-notebook"></a>Wyodrębnij dane formularza przy użyciu notesu

### <a name="mount-the-azure-data-lake-storage"></a>Zainstaluj Magazyn Azure Data Lake

Następnym krokiem jest ocena różnorodnych formularzy przy użyciu przeszkolonego modelu. Zainstalujemy konto Azure Data Lake Storage w kostkach i zapoznaj się z instalowaniem w procesie pozyskiwania.

Podobnie jak w przypadku etapu szkoleniowego, będziemy używać Azure Data Factory do wywołania wyodrębniania par klucz/wartość z formularzy. Będziemy przepętlać formularze w folderach, które są określone w tabeli parametrów.

1. Utwórz Notes, aby zainstalować konto magazynu w kostkach danych. Zadzwoń do **MountDataLake** IT. 
1. Musisz najpierw wywołać Notes **ustawień** :

    ```python
    %run "./Settings"
    ```

1. W drugiej komórce Zdefiniuj zmienne dla poufnych parametrów, które będziemy pobierać z naszych Key Vault wpisów tajnych:

    ```python
    cognitive_service_subscription_key = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveserviceSubscriptionKey")
    cognitive_service_endpoint = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "CognitiveServiceEndpoint")
    
    scoring_storage_account_name = dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountName")
    scoring_storage_account_sas_key= dbutils.secrets.get(scope = "FormRecognizer_SecretScope", key = "StorageAccountSasKey")  
    
    scoring_mount_point = "/mnt/"+scoring_container_name 
    scoring_source_str = "wasbs://{container}@{storage_acct}.blob.core.windows.net/".format(container=scoring_container_name, storage_acct=scoring_storage_account_name) 
    scoring_conf_key = "fs.azure.sas.{container}.{storage_acct}.blob.core.windows.net".format(container=scoring_container_name, storage_acct=scoring_storage_account_name)
    
    ```

1. Spróbuj odinstalować konto magazynu na wypadek, gdyby zostało wcześniej zainstalowane:

    ```python
    try:
      dbutils.fs.unmount(scoring_mount_point) # Use this to unmount as needed
    except:
      print("{} already unmounted".format(scoring_mount_point))
    
    ```

1. Zainstaluj konto magazynu:


    ```python
    try: 
      dbutils.fs.mount( 
        source = scoring_source_str, 
        mount_point = scoring_mount_point, 
        extra_configs = {scoring_conf_key: scoring_storage_account_sas_key} 
      ) 
    except Exception as e: 
      print("ERROR: {} already mounted. Run previous cells to unmount first".format(scoring_mount_point))
    
    ```

    > [!NOTE]
    > Zainstalowano tylko konto magazynu szkoleń. W takim przypadku pliki szkoleniowe i pliki, z których chcą wyodrębnić pary klucz/wartość, znajdują się na tym samym koncie magazynu. Jeśli konta magazynu oceniania i uczenia są różne, należy zainstalować oba konta magazynu. 

### <a name="write-the-scoring-notebook"></a>Napisz Notes oceniania

Teraz możemy utworzyć Notes oceniania. Wykonamy coś podobnego do tego, co będziemy w notesie szkoleniowym: będziemy używać plików przechowywanych w folderach na koncie Azure Data Lake Storage, które właśnie zainstalowano. Nazwa folderu jest przenoszona jako zmienna. Będziemy przepętlać wszystkie formularze w określonym folderze i wyodrębniać z nich pary klucz/wartość. 

1. Utwórz Notes i Wywołaj go **ScoreFormRecognizer**. 
1. Uruchom **Ustawienia** i notesy **MountDataLake** :

    ```python
    %run "./Settings"
    %run "./MountDataLake"
    ```

1. Dodaj ten kod, który wywołuje interfejs API [analizy](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm) :

    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post 
    
    
    #prefix= TrainingBlobFolder
    post_url = cognitive_service_endpoint + "/formrecognizer/v2.0/custom/models/%s/analyze" % model_id
    source = r"/dbfs/mnt/"+scoring_container_name+"/"+scoring_input_blob_folder+"/"+file_to_score_name
    output = r"/dbfs/mnt/"+scoring_container_name+"/scoringforms/ExtractionResult/"+os.path.splitext(os.path.basename(source))[0]+"_output.json"
    
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers.
        'Content-Type': file_type,
        'Ocp-Apim-Subscription-Key': cognitive_service_subscription_key,
    }
    
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```

1. W następnej komórce uzyskasz wyniki wyodrębniania par klucz/wartość. Ta komórka spowoduje wyjście z wyniku. Chcemy, aby wynik w formacie JSON mógł zostać przetworzony dalej w Azure SQL Database lub Azure Cosmos DB. Dlatego zapiszemy wynik do pliku JSON. Nazwa pliku wyjściowego będzie nazwą pliku wynikowego połączonego z "_output.json". Plik zostanie zapisany w tym samym folderze co plik źródłowy.

    ```python
    n_tries = 10
    n_try = 0
    wait_sec = 5
    max_wait_sec = 5
    while n_try < n_tries:
       try:
           resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": cognitive_service_subscription_key})
           resp_json = resp.json()
           if resp.status_code != 200:
               print("GET analyze results failed:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           status = resp_json["status"]
           if status == "succeeded":
               print("Analysis succeeded:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           if status == "failed":
               print("Analysis failed:\n%s" % json.dumps(resp_json))
               n_try += 1
               quit()
           # Analysis still running. Wait and retry.
           time.sleep(wait_sec)
           n_try += 1
           wait_sec = min(2*wait_sec, max_wait_sec)     
       except Exception as e:
           msg = "GET analyze results failed:\n%s" % str(e)
           print(msg)
           n_try += 1
           print("Analyze operation did not complete within the allocated time.")
           quit()
    
    ```
1. Wykonaj procedurę pisania pliku w nowej komórce:

    ```python
    import requests
    file = open(output, "w")
    file.write(str(resp_json))
    file.close()
    ```

## <a name="automate-training-and-scoring-by-using-azure-data-factory"></a>Automatyzacja szkoleń i oceniania przy użyciu Azure Data Factory

Jedyną pozostałą czynnością jest skonfigurowanie usługi Azure Data Factory w celu zautomatyzowania procesów szkolenia i oceniania. Najpierw wykonaj kroki opisane w sekcji [Tworzenie fabryki danych](../../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory). Po utworzeniu zasobu Azure Data Factory należy utworzyć trzy potoki: jeden do szkolenia i dwa do oceny. (Wyjaśnimy później).

### <a name="training-pipeline"></a>Potok szkoleń

Pierwsze działanie w potoku szkolenia jest wyszukiwaniem i zwróceniem wartości z tabeli parametryzacja w bazie danych SQL Azure. Wszystkie zbiory danych szkoleniowych będą znajdować się na tym samym koncie magazynu i kontenerze (ale potencjalnie w różnych folderach). Dlatego zachowamy domyślną wartość atrybutu **pierwszy wiersz tylko** w ustawieniach działania wyszukiwania. Dla każdego typu formularza, aby szkolić model, nauczymy model przy użyciu wszystkich plików w **training_blob_root_folder**.

:::image type="content" source="./media/tutorial-bulk-processing/training-pipeline.png" alt-text="Zrzut ekranu pokazujący potok szkoleniowy w Data Factory.":::

Procedura składowana przyjmuje dwa parametry: **model_id** i **form_batch_group_id**. Kod do zwrócenia identyfikatora modelu z notesu datakosteks to `dbutils.notebook.exit(model_id)` . Kod do odczytu kodu w działaniu procedury składowanej w Data Factory to `@activity('GetParametersFromMetadaTable').output.firstRow.form_batch_group_id` .

### <a name="scoring-pipelines"></a>Potoki oceniania

Aby wyodrębnić pary klucz/wartość, przeskanujemy wszystkie foldery w tabeli parametryzacja. Dla każdego folderu wyodrębnimy pary klucz/wartość wszystkich plików w nim. Azure Data Factory nie obsługuje obecnie zagnieżdżonych pętli ForEach. Zamiast tego utworzymy dwa potoki. Pierwszy potok wykona wyszukiwanie z tabeli parametryzacja i przekaże listę folderów jako parametr do drugiego potoku.

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-1a.png" alt-text="Zrzut ekranu, który pokazuje pierwszy potok oceniania w Data Factory.":::

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-1b.png" alt-text="Zrzut ekranu pokazujący szczegóły pierwszego potoku oceniania w Data Factory.":::

Drugi potok użyje działania getmeta, aby uzyskać listę plików w folderze i przekazać go jako parametr do notesu datakostki oceniania.

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-2a.png" alt-text="Zrzut ekranu pokazujący drugi potok oceniania w Data Factory.":::

:::image type="content" source="./media/tutorial-bulk-processing/scoring-pipeline-2b.png" alt-text="Zrzut ekranu pokazujący szczegóły drugiego potoku oceniania w Data Factory.":::

### <a name="specify-a-degree-of-parallelism"></a>Określ stopień równoległości

W przypadku potoków szkoleń i oceniania można określić stopień równoległości, aby można było jednocześnie przetwarzać wiele formularzy.

Aby ustawić stopień równoległości potoku Azure Data Factory:

1. Wybierz działanie **foreach** .
1. Wyczyść pole **sekwencyjne** .
1. Ustaw stopień równoległości w polu **Liczba partii** . Zalecamy maksymalną liczbę partii wynoszącą 15 na potrzeby oceny.

:::image type="content" source="./media/tutorial-bulk-processing/parallelism.png" alt-text="Zrzut ekranu pokazujący konfigurację równoległości działania oceniania w Azure Data Factory.":::

## <a name="how-to-use-the-pipeline"></a>Jak korzystać z potoku

Masz teraz zautomatyzowany potok umożliwiający cyfrowe zaległości formularzy i uruchamianie niektórych analiz na ich podstawie. Po dodaniu nowych form znanego typu do istniejącego folderu magazynu, wystarczy ponownie uruchomić potoki oceniania. Będą oni aktualizować wszystkie pliki wyjściowe, w tym pliki wyjściowe dla nowych formularzy. 

W przypadku dodania nowych formularzy nowego typu należy również przekazać zestaw danych szkoleniowych do odpowiedniego kontenera. Następnie dodasz nowy wiersz w tabeli parametryzacja, wprowadzając lokalizacje nowych dokumentów i ich zestawu danych szkoleniowych. Wprowadź wartość-1 dla **model_ID** , aby wskazać, że nowy model musi być szkolony dla formularzy. Następnie uruchom potok szkoleń w Azure Data Factory. Potok zostanie odczytany z tabeli, nauczyć się modelu i zastąpi Identyfikator modelu w tabeli. Następnie można wywołać potoki oceniania, aby rozpocząć zapisywanie plików wyjściowych.

## <a name="next-steps"></a>Następne kroki

W tym samouczku pokazano, jak skonfigurować potoki Azure Data Factory, aby wyzwolić szkolenia i uruchamiać modele aparatów rozpoznawania formularzy oraz cyfrowych fragmentów plików. Następnie zapoznaj się z interfejsem API aparatu rozpoznawania formularzy, aby zobaczyć, co jeszcze można zrobić z nim.

* [Interfejs API REST aparatu rozpoznawania formularzy](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeBusinessCardAsync)
