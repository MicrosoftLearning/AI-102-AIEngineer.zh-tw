---
lab:
  title: 分析文字
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 27cd22e81d4fe8fda27e6fc960d3b3aeb8debded
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195538"
---
# <a name="analyze-text"></a>分析文字

**語言** 服務是支援文字分析的認知服務，包括語言偵測、情感分析、關鍵片語擷取和實體辨識。

例如，假設旅遊機構想要處理已提交至公司網站的旅館評論。 他們可使用語言服務來判斷用以撰寫每篇評論的語言、評論的情感 (正面、中立或負面)、可能指出評論中所討論主要主題的關鍵片語，以及具名實體，例如評論中提及的地點、地標或人物。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未將 **AI-102-AIEngineer** 程式碼存放庫複製到您目前使用實驗室的環境，請遵循下列步驟來執行這項操作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

如果您的訂用帳戶中還沒有 **認知服務** 資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋 *認知服務*，然後使用以下設定建立 **認知服務** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署詳細資料。
5. 部署資源後，移至該資源並檢視其 [金鑰和端點] 頁面。 在下一個程序中，您需要此頁面中的端點和其中一個金鑰。

## <a name="prepare-to-use-the-language-sdk-for-text-analytics"></a>準備使用語言 SDK 進行文字分析

在此練習中，您將完成部分實作的用戶端應用程式，該應用程式會使用語言服務文字分析 SDK 來分析旅館評論。

> **注意**：您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **05-analyze-text** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **text-analysis** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝文字分析 SDK 套件：
    
    **C#**
    
    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```
    
    **Python**
    
    ```
    pip install azure-ai-textanalytics==5.1.0
    ```
    
3. 檢視 **text-analysis** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。

4. 請注意，**text-analysis** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：text-analysis.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**
    
    ```C#
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

5. 在 **Main** 函式中，請注意已經提供可從組態檔載入認知服務端點和金鑰的程式碼。 然後尋找 **Create client using endpoint and key** 註解，接著新增下列程式碼以建立文字分析 API 的用戶端：

    **C#**

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

6. 儲存變更並返回 **text-analysis** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

6. 觀察輸出，因為程式碼應該執行且不會發生錯誤，並在 **reviews** 資料夾中顯示每個評論文字檔的內容。 應用程式已成功為文字分析 API 建立用戶端，但不會使用。 我們將在下一個程序中修正該問題。

## <a name="detect-language"></a>偵測語言種類

既然您已為文字分析 API 建立用戶端，讓我們使用來偵測用以撰寫每篇評論的語言。

1. 在程式的 **Main** 函式中，尋找 **Get language** 註解。 然後，在此註解之下，新增用以在每個評論文件中偵測語言所需的程式碼：

    **C#**
    
    ```C
    // Get language
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**
    
    ```Python
    # Get language
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **注意**：*在此範例中，每篇評論都會進行個別分析，導致對每個檔案個別呼叫服務。替代方法是建立一組文件並在單一呼叫中將其傳遞給服務。在這兩種方法中，服務的回應都是由一組文件所組成；這就是為何在上述的 Python 程式碼中，已指定了回應 ([0]) 中第一份 (且唯一的) 文件的索引。*

6. 儲存變更並返回 **text-analysis** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

7. 觀察輸出，請注意這次已識別每篇評論的語言。

## <a name="evaluate-sentiment"></a>評估情感

*情感分析* 是常用的技術，可將文字分類為 *正面* 或 *負面*(或可能是 *中立* 或 *混合*)。 其通常用來分析社交媒體文章、產品評論，以及文字情感可能提供實用見解的其他項目。

1. 在程式的 **Main** 函式中，尋找 **取得情緒** 註解。 然後，在此註解之下，新增用以偵測每個評論文件情感所需的程式碼：

    **C#**
    
    ```C
    // Get sentiment
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**
    
    ```Python
    # Get sentiment
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

2. 儲存變更並返回 **text-analysis** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

3. 觀察輸出，請注意已偵測到評論的情感。

## <a name="identify-key-phrases"></a>識別關鍵片語

識別文字本文中的關鍵片語有助於判斷其所討論的主要主題。

1. 在程式的 **Main** 函式中，尋找 **取得關鍵片語** 註解。 然後，在此註解之下，新增用以在每個評論文件中偵測關鍵片語所需的程式碼：

    **C#**

    ```C
    // Get key phrases
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```
    
    **Python**
    
    ```Python
    # Get key phrases
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

2. 儲存變更並返回 **text-analysis** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 觀察輸出，請注意每個文件都包含關鍵片語，以提供評論內容的一些見解。

## <a name="extract-entities"></a>擷取實體

通常，文件或其他文字本文會提及人物、地點、時段或其他實體。 文字分析 API 可以在您文字中偵測多種實體類別 (和子類別)。

1. 在程式的 **Main** 函式中，尋找 **取得實體** 註解。 然後，在此註解之下，新增用以識別每篇評論中提及的實體所需的程式碼：

    **C#**
    
    ```C
    // Get entities
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get entities
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

2. 儲存變更並返回 **text-analysis** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 觀察輸出，請注意文字中偵測到的實體。

## <a name="extract-linked-entities"></a>擷取連結的實體

除了已分類的實體之外，文字分析 API 還可以偵測資料來源有已知連結的實體，例如 Wikipedia。

1. 在程式的 **Main** 函式中，尋找 **取得連結的實體** 註解。 然後，在此註解之下，新增用以識別每篇評論中提及的連結實體所需的程式碼：

    **C#**
    
    ```C
    // Get linked entities
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get linked entities
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

2. 儲存變更並返回 **text-analysis** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 觀察輸出，請注意已識別的連結實體。

## <a name="more-information"></a>詳細資訊

如需使用 **語言** 服務的詳細資訊，請參閱[文字分析文件](https://docs.microsoft.com/azure/cognitive-services/language-service/)。
