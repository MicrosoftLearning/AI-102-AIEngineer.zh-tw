---
lab:
  title: 建立交談語言理解用戶端應用程式 (預覽)
ms.openlocfilehash: 486a6716a189156739e9107824e561f7bb8fa95c
ms.sourcegitcommit: 2c92ea1491cac72a4765755cf2bc6d8b5f657a46
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/09/2022
ms.locfileid: "145195544"
---
# <a name="create-a-language-service-client-application"></a>建立語言服務用戶端應用程式

適用於語言的 Azure 認知服務的交談語言理解功能可讓您定義交談語言理解模型，而用戶端應用程式可使用該模型來解譯使用者的自然語言輸入、預測使用者 *意圖* (他們想要達成的目的)，以及識別任何應套用意圖的 *實體*。 您可以建立用戶端應用程式，直接透過 REST 介面或經由使用語言特有軟體開發工具組 (SDK) 來取用交談語言理解模型。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室使用的環境，請在 Visual Studio Code 中予以開啟；否則，依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。

2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。

3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="create-language-service-resources"></a>建立語言服務資源

如果您的 Azure 訂用帳戶中已經有語言服務資源，即可在此練習中使用。 否則，遵循這些指示來加以建立。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

2. 選取 [+ 建立資源] 按鈕，搜尋「語言服務」，然後使用下列設定建立 **語言服務** 資源：

    - **預設功能**：全部
    - **自訂功能**：無
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*westus2*
    - **名稱**：*輸入唯一名稱*
    - **定價層**：*S*
    - **法律條款**：*選取核取方塊進行確認*
    - **負責任 AI 注意事項**：*選取核取方塊進行確認*

3. 等候資源建立。 您可以導覽至在其中建立資源的資源群組來檢視資源。

## <a name="import-train-and-publish-a-conversational-language-understanding-model"></a>匯入、定型及發佈交談語言理解模型

如果您已經有上一個實驗室或練習中的 **Clock** 專案，即可在此練習中使用。 否則，遵循這些指示來加以建立。

1. 在新的瀏覽器索引標籤中，開啟 Language Studio - 預覽入口網站 (位於 `https://language.cognitive.azure.com`)。

2. 使用與您 Azure 訂用帳戶建立關聯的 Microsoft 帳戶登入。 若這是您第一次登入語言服務入口網站，您可能需要授與應用程式一些權限以存取您的詳細帳戶資料。 然後選取您剛才建立的 Azure 訂用帳戶和製作資源，以完成「歡迎使用」步驟。

3. 開啟 [交談語言理解] 頁面。

3. 在 [&#65291;建立新專案] 旁邊，檢視下拉式清單並選取 [匯入]。 按一下 [選擇檔案]，然後瀏覽至專案資料夾中的 **10b-clu-client-(preview)** 子資料夾 (其中包含本練習的實驗室檔案)。 選取 **Clock.json**，按一下 [開啟]，然後按一下 [完成]。

4. 若顯示帶有建立有效語言服務應用程式的提示窗格，請將其關閉。

5. 在 Language Studio 入口網站的左側，選取 [定型模型] 以定型應用程式。 按一下 [開始定型作業]，將模型命名為 **Clock**，並確定評估已啟用定型。 可能需要幾分鐘的時間才能完成定型。

    > **注意**：因為模型名稱 **Clock** 會在 clock-client 程式碼中進行硬式編碼 (稍後在實驗室中使用)，所以完全如所述拼出名稱 (大小寫相符)。 

6. 在 Language Studio 入口網站的左側，選取 [部署模型]，並使用 [新增部署] 來建立名為 **Production** 的 Clock 模型部署。

    > **注意**：因為部署名稱 **production** 會在 clock-client 程式碼中進行硬式編碼 (稍後在實驗室中使用)，所以完全如所述拼出名稱 (大小寫相符)。 

7. 部署完成後，選取 **production** 部署，然後按一下 [取得預測 URL]。 用戶端應用程式需要端點 URL 才能使用已部署的模型。

8. 在 Language Studio 入口網站的左側，選取 [專案設定] 並記下[主要金鑰]。 用戶端應用程式需要主要金鑰才能使用已部署的模型。

9. 用戶端應用程式需要來自預測 URL 和語言服務金鑰的資訊，才能連線到已部署的模型並進行驗證。

## <a name="prepare-to-use-the-language-service-sdk"></a>準備使用語言服務 SDK

在此練習中，您將完成部分實作的用戶端應用程式，該應用程式會使用 Clock 模型 (已發佈的交談語言理解模型) 來預測使用者輸入的意圖，並適當地回應。

> **注意**：您可選擇適用於 **.NET** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **10b-clu-client-(preview)** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。

2. 以滑鼠右鍵按一下 **clock-client** 資料夾，然後選取 [在整合式終端機中開啟]。 然後針對您的語言喜好設定執行適當的命令，以安裝交談語言服務 SDK 套件：

    **C#**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-language-conversations --pre
    python -m pip install python-dotenv
    python -m pip install python-dateutil
    ```

3. 檢視 **clock-client** 資料夾的內容，並注意其中包含組態設定的檔案：

    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以納入您語言資源的 [端點 URL] 和 [主要金鑰]。 您可以在 Azure 入口網站或 Language Studio 中找到必要的值，如下所示：

    - Azure 入口網站：開啟您的語言資源。 在 [資源管理] 下，選取 [金鑰與端點]。 將 [KEY 1] 和 [端點] 值複製到您的組態設定檔案。
    - Language Studio：開啟 **Clock** 專案。 您可以在 [部署模型] 頁面的 [取得預測 URL] 之下找到語言服務端點，以及在 [專案設定] 頁面上找到 [主要金鑰]。 預測 URL 的語言服務端點部分會以 **.cognitiveservices.azure.com/** 結尾。 例如：`https://ai102-langserv.cognitiveservices.azure.com/`。

4. 請注意，**clock-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：clock-client.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用語言服務 SDK 所需的命名空間：

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    from azure.ai.language.conversations.models import ConversationAnalysisOptions
    ```

## <a name="get-a-prediction-from-the-conversational-language-model"></a>從交談語言模型取得預測

現在您已準備好實作使用 SDK 的程式碼，以從交談語言模型取得預測。

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入預測端點和金鑰的程式碼。 然後尋找 **建立語言服務模型的用戶端** 註解，並新增下列程式碼來為您的語言服務應用程式建立預測用戶端：

    **C#**

    ```C#
    // Create a client for the Language service model
    Uri lsEndpoint = new Uri(predictionEndpoint);
    AzureKeyCredential lsCredential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient lsClient = new ConversationAnalysisClient(lsEndpoint, lsCredential);
    ```

    **Python**

    ```Python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

2. 請注意，**Main** 函式中的程式碼會提示使用者輸入，直到使用者輸入 "quit" 為止。 在此迴圈中，尋找 **呼叫語言服務模型以取得意圖和實體** 註解，並新增下列程式碼：

    **C#**

    ```C#
    // Call the Language service model to get intent and entities
    var lsProject = "Clock";
    var lsSlot = "production";

    ConversationsProject conversationsProject = new ConversationsProject(lsProject, lsSlot);
    Response<AnalyzeConversationResult> response = await lsClient.AnalyzeConversationAsync(userText, conversationsProject);

    ConversationPrediction conversationPrediction = response.Value.Prediction as ConversationPrediction;

    Console.WriteLine(JsonConvert.SerializeObject(conversationPrediction, Formatting.Indented));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].Confidence > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }

    var entities = conversationPrediction.Entities;
    ```

    **Python**

    ```Python
    # Call the Language service model to get intent and entities
    convInput = ConversationAnalysisOptions(
        query = userText
        )
    
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        result = client.analyze_conversations(
            convInput, 
            project_name=cls_project, 
            deployment_name=deployment_slot
            )

    # list the prediction results
    top_intent = result.prediction.top_intent
    entities = result.prediction.entities

    print("view top intent:")
    print("\ttop intent: {}".format(result.prediction.top_intent))
    print("\tcategory: {}".format(result.prediction.intents[0].category))
    print("\tconfidence score: {}\n".format(result.prediction.intents[0].confidence_score))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity.category))
        print("\ttext: {}".format(entity.text))
        print("\tconfidence score: {}".format(entity.confidence_score))

    print("query: {}".format(result.query))
    ```

    語言服務應用程式的呼叫會傳回預測/結果，其中包含最高 (最可能) 意圖以及在輸入表達中偵測到的任何實體。 用戶端應用程式現在必須使用該預測來判斷和執行適當的動作。

3. 尋找 **套用適合的動作** 註解，並新增下列程式碼，以檢查應用程式所支援的意圖 (**GetTime**、**GetDate** 和 **GetDay**)，並判斷是否已偵測到任何相關的實體，然後再呼叫現有的函式來產生適當的回應。

    **C#**

    ```C#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a location entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Location")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        location = entity.Text;
                    }
                    
                }

            }

            // Get the time for the specified location
            var getTimeTask = Task.Run(() => GetTime(location));
            string timeResponse = await getTimeTask;
            Console.WriteLine(timeResponse);
            break;

        case "GetDay":
            var date = DateTime.Today.ToShortDateString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Date entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Date")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        date = entity.Text;
                    }
                }
            }
            // Get the day for the specified date
            var getDayTask = Task.Run(() => GetDay(date));
            string dayResponse = await getDayTask;
            Console.WriteLine(dayResponse);
            break;

        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Weekday entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Weekday")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        day = entity.Text;
                    }
                }
                
            }
            // Get the date for the specified day
            var getDateTask = Task.Run(() => GetDate(day));
            string dateResponse = await getDateTask;
            Console.WriteLine(dateResponse);
            break;

        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**

    ```Python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity.category:
                    # ML entities are strings, get the first one
                    location = entity.text
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity.category:
                    # Regex entities are strings, get the first one
                    date_string = entity.text
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity.category:
                # List entities are lists
                    day = entity.text
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

4. 儲存變更並返回 **clock-client** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python clock-client.py
    ```

5. 出現提示時，請輸入表達來測試應用程式。 例如，嘗試：

    *您好*
    
    *What time is it? (現在幾點？)*

    *What's the time in London? (倫敦現在幾點？)*

    *What's the date? (今天是幾月幾號？)*

    *What date is Sunday? (星期天是幾月幾號？)*

    *What day is it? (今天是什麼日子？)*

    *What day is 01/01/2025? (今天是 2025 年 1 月 1 日嗎？)*

> **注意**：應用程式中的邏輯刻意簡單，而且有一些限制。 例如，取得時間時，只支援一組受限制的城市並忽略日光節約時間。 目標在於查看使用語言服務的典型模式範例，而您的應用程式在其中必須：
>
>   1. 連線至預測端點。
>   2. 提交表達以取得預測。
>   3. 實作邏輯以適當地回應預測的意圖和實體。

6. 完成測試後，請輸入 *quit*。

## <a name="more-information"></a>詳細資訊

若要深入了解如何建立語言服務用戶端，請參閱[開發人員文件](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)
