---
lab:
  title: 建立 Language Understanding 用戶端應用程式
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 36f75e47910707c959495140be43c4196649d471
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195471"
---
# <a name="create-a-language-understanding-client-application"></a>建立 Language Understanding 用戶端應用程式

Language Understanding 服務可讓您定義一個封裝語言模型的應用程式，而應用程式可使用該語言模型來解譯使用者自然語言輸入、預測使用者 *意圖* (他們想要達成的目的)，以及識別應套用意圖的任何 *實體*。 您可以建立用戶端應用程式，直接透過 REST 介面或經由使用語言特有軟體開發工具組 (SDK) 來取用 Language Understanding 應用程式。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室使用的環境，請在 Visual Studio Code 中予以開啟；否則，依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="create-language-understanding-resources"></a>建立 Language Understanding 資源

如果您在 Azure 訂用帳戶中已經有 Language Understanding 製作和預測資源，即可在此練習中使用。 否則，遵循這些指示來建立。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 選取 [&#65291;建立資源] 按鈕、搜尋 *語言理解*，並使用下列設定建立 **Language Understanding** 資源：
    - **建立選項**：兩者
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **名稱**：*輸入唯一名稱*
    - **製作位置**：選取您慣用的位置
    - **製作定價層**：F0
    - **預測位置***選擇與您的製作位置<u>相同的位置</u>*
    - 預測定價層：F0 (*如果 F0 無法使用，則選擇 S0*)

3. 等待資源建立，並注意兩種已建置的兩種語言理解資源；一種是製作資源，另一種是預測資源。 您可以透過導覽到建立其資源群組來檢視這兩種資源。

## <a name="import-train-and-publish-a-language-understanding-app"></a>匯入、定型及發佈 Language Understanding 應用程式

如果您已經有上一個練習中的 **Clock** 應用程式，即可在此練習中使用。 否則，遵循這些指示來加以建立。

1. 在新的瀏覽器索引標籤中開啟 Language Understanding 入口網站 (位於 `https://www.luis.ai`)。
2. 使用與您 Azure 訂用帳戶建立關聯的 Microsoft 帳戶登入。 若這是您第一次登入語言理解入口網站，您可能需要授與應用程式一些權限以存取您的詳細帳戶資料。 然後選取您剛才建立的 Azure 訂用帳戶和製作資源，以完成「歡迎使用」步驟。
3. 開啟 [交談應用程式] 頁面，在 [&#65291;新增應用程式] 旁邊，檢視下拉式清單，然後選取 [ 匯入為 LU]。
瀏覽至專案資料夾中的 **10-luis-client** 子資料夾 (其中包含此練習的實驗室檔案)，然後選取 **Clock.lu**。 然後為時鐘應用程式指定唯一的名稱。
4. 若顯示帶有建立有效語言理解應用程式的提示窗格，請將其關閉。
5. 在 Language Understanding 入口網站頂端，選取 [定型] 以定型您的應用程式。
6. 在 Language Understanding 入口網站的右上方，選取 [發佈] 並將應用程式發佈至 [生產位置]。
7. 發佈完成之後，在 Language Understanding 入口網站頂端，選取 [管理]。
8. 在 [設定] 頁面上，記下 [應用程式識別碼]。 用戶端應用程式需要此資訊才能使用您的應用程式。
9. 在 [Azure 資源] 頁面的 [預測資源] 之下，如果沒有列出預測資源，請在您的 Azure 訂用帳戶中新增預測資源。
10. 請注意預測資源的 [主要金鑰]、[次要金鑰] 和 [端點 URL]。 用戶端應用程式需要此端點和其中一個金鑰，才能連線到預測資源並進行驗證。

## <a name="prepare-to-use-the-language-understanding-sdk"></a>準備使用 Language Understanding SDK

在此練習中，您將完成部分實作的用戶端應用程式，該應用程式會使用時鐘Language Understanding應用程式來預測使用者輸入的意圖，並適當地回應。

> **注意**：您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **10-luis-client** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **clock-client** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝 Language Understanding SDK 套件：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

*除了 [執行階段] (預測) 套件之外，還有一個 [製作] 套件可用來撰寫程式碼，以建立和管理 Language Understanding 模型。*

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
*Python SDK 套件包含 **預測** 和 **製作** 的類別。*

3. 檢視 **clock-client** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以納入 Language Understanding 應用程式的 [應用程式識別碼]，以及 [端點 URL] 及其預測資源的其中一個 [金鑰] (從 Language Understanding 入口網站中您應用程式的 [管理] 頁面)。

4. 請注意，**clock-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：clock-client.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用 Language Understanding 預測 SDK 所需的命名空間：

**C#**

```C#
// Import namespaces
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# Import namespaces
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## <a name="get-a-prediction-from-the-language-understanding-app"></a>從 Language Understanding 應用程式取得預測

現在您已準備好實作使用 SDK 的程式碼，從 Language Understanding 應用程式取得預測。

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入應用程式識別碼、預測端點和金鑰的程式碼。 然後尋找 **建立 LU 應用程式的用戶端** 註解，並新增下列程式碼來為您的 Language Understanding 應用程式建立預測用戶端：

**C#**

```C#
// Create a client for the LU app
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# Create a client for the LU app
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. 請注意，**Main** 函式中的程式碼會提示使用者輸入，直到使用者輸入 "quit" 為止。 在此迴圈中，尋找 **呼叫 LU 應用程式以取得意圖和實體** 註解，並新增下列程式碼：

**C#**

```C#
// Call the LU app to get intent and entities
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# Call the LU app to get intent and entities
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

Language Understanding 應用程式的呼叫會傳回預測，其中包含最高 (最可能) 意圖以及在輸入表達中偵測到的任何實體。 用戶端應用程式現在必須使用該預測來判斷和執行適當的動作。

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
            if (entities.ContainsKey("Location"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML entities are strings, get the first one
                location = entityJson[0].ToString();
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
            if (entities.ContainsKey("Date"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // Regex entities are strings, get the first one
                date = entityJson[0].ToString();
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
            if (entities.ContainsKey("Weekday"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // List entities are lists
                day = entityJson[0][0].ToString();
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
        if 'Location' in entities:
            # ML entities are strings, get the first one
            location = entities['Location'][0]
    # Get the time for the specified location
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if len(entities) > 0:
        # Check for a Date entity
        if 'Date' in entities:
            # Regex entities are strings, get the first one
            date_string = entities['Date'][0]
    # Get the day for the specified date
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # Check for entities
    if len(entities) > 0:
        # Check for a Weekday entity
        if 'Weekday' in entities:
            # List entities are lists
            day = entities['Weekday'][0][0]
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

> **注意**：應用程式中的邏輯刻意簡單，而且有一些限制。 例如，取得時間時，只支援一組受限制的城市並忽略日光節約時間。 目標在於查看使用 Language Understanding 的典型模式範例，以便，而您的應用程式在其中必須：
>
>   1. 連線至預測端點。
>   2. 提交表達以取得預測。
>   3. 實作邏輯以適當地回應預測的意圖和實體。

6. 完成測試後，請輸入 *quit*。

## <a name="more-information"></a>詳細資訊

若要深入了解如何建立 Language Understanding 用戶端，請參閱[開發人員文件](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)
