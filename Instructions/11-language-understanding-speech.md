---
lab:
  title: 使用語音和 Language Understanding 服務
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 31b88612bce38780f828859f8e2b166dbe8df34d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195532"
---
# <a name="use-the-speech-and-language-understanding-services"></a>使用語音和 Language Understanding 服務

您可以將語音服務整合 Language Understanding 服務以建立應用程式，透過智慧功能從語音輸入判斷使用者意圖。

> **注意**：使用麥克風可以達到最佳的練習效果。 有些裝載的虛擬環境或許能夠從本機麥克風擷取音訊，但如果這不可行 (或您完全沒有麥克風)，也可使用所提供的音訊檔案進行語音輸入。 請仔細遵循指示，因為您必須根據您使用的是麥克風或音訊檔案來選擇不同的選項。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在進行此實驗室的環境，請在 Visual Studio Code 中予以開啟；否則，依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (不限資料夾)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="create-language-understanding-resources"></a>建立 Language Understanding 資源

如果您在 Azure 訂用帳戶中已經有 Language Understanding 的製作和預測資源，則可在此練習中使用。 否則，請遵循這些指示來建立這些資源。

1. 開啟位於 `https://portal.azure.com` 的 Azure 入口網站，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 選取 [&#65291;建立資源] 按鈕、搜尋「language understanding」，並使用下列設定建立 **Language Understanding** 資源：
    - **建立選項**：兩者
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **名稱**：*輸入唯一名稱*
    - **製作位置**：選取您慣用的位置
    - **製作定價層**：F0
    - **預測位置**：*選擇與您的製作位置<u>相同的位置</u>*
    - **預測定價層**：F0 (*如果 F0 無法使用，則選擇 S0*)

3. 等待資源建立，並注意兩種已建置的兩種語言理解資源；一種是製作資源，另一種是預測資源。 您可以導覽到建立這兩種資源的資源群組以進行檢視。

## <a name="prepare-a-language-understanding-app"></a>準備 Language Understanding 應用程式

如果您已經有上一個練習中的 **Clock** (時鐘) 應用程式，請在位於 `https://www.luis.ai` 的 Language Understanding 入口網站中開啟。 否則，請遵循這些指示來加以建立。

1. 在新的瀏覽器索引標籤中開啟 Language Understanding 入口網站 (`https://www.luis.ai`)。
2. 使用與您 Azure 訂用帳戶建立關聯的 Microsoft 帳戶登入。 若這是您第一次登入語言理解入口網站，您可能需要授與應用程式一些權限以存取您的詳細帳戶資料。 然後選取您剛才建立的 Azure 訂用帳戶和製作資源，以完成「歡迎使用」步驟。
3. 開啟 [交談應用程式] 頁面，在 [&#65291;新增應用程式] 旁邊，檢視下拉式清單，然後選取 [匯入為 LU]。
瀏覽至專案資料夾中的 **11-luis-speech** 子資料夾 (其中包含此練習的實驗室檔案)，然後選取 **Clock.lu**。 然後為時鐘應用程式指定唯一的名稱。
4. 若顯示帶有建立有效語言理解應用程式的提示窗格，請將其關閉。

## <a name="train-and-publish-the-app-with-speech-priming"></a>使用 *語音預備* 訓練和發佈應用程式

1. 如果尚未訓練，在 Language Understanding 入口網站頂端，請選取 [訓練] 以訓練應用程式。
2. 在 Language Understanding 入口網站的右上方，選取 [發佈]。 然後選取 [生產位置] 並變更設定以啟用 **語音預備** (這會改善語音辨識的效能)。
3. 發佈完成之後，在 Language Understanding 入口網站頂端，選取 [管理]。
4. 在 [設定] 頁面上，記下 [應用程式識別碼]。 用戶端應用程式需要此資訊才能使用您的應用程式。
5. 在 [Azure 資源] 頁面的 [預測資源] 之下，如果沒有列出預測資源，請在您的 Azure 訂用帳戶中新增預測資源。
6. 請注意預測資源的 [主要金鑰]、[次要金鑰] 和 [位置] (<u>不是</u>端點！)。 語音 SDK 用戶端應用程式需要此位置和其中一個金鑰，才能連線到預測資源並進行驗證。

## <a name="configure-a-client-application-for-language-understanding"></a>設定 Language Understanding 的用戶端應用程式

在此練習中，您將建立可接受語音輸入的用戶端應用程式，並使用 Language Understanding 應用程式來預測使用者的意圖。

> **注意**：在此練習中，您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在後續步驟中，請執行適合於您偏好語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **11-luis-speech** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 檢視 **speaking-clock-client** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以納入 Language Understanding 應用程式的 [應用程式識別碼]，以及 [位置] (<u>不是</u>完整端點，例如 *eastus*) 及其預測資源的其中一個 [金鑰] (從 Language Understanding 入口網站中您應用程式的 [管理] 頁面)。

## <a name="install-sdk-packages"></a>安裝 SDK 套件

若要將語音 SDK 與 Language Understanding 服務搭配使用，您需要安裝適用於所使用程式設計語言的語音 SDK 套件。

1. 在 Visual Studio 中，以滑鼠右鍵按一下 **speaking-clock-client** 資料夾，然後開啟整合式終端。 然後針對您偏好的語言執行適當的命令，以安裝 Language Understanding SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. 此外，如果您的系統<u>沒有</u>可運作的麥克風，則必須使用音訊檔案來提供應用程式的語音輸入。 在此情況下，請使用下列命令來安裝額外的套件，讓程式可以播放音訊檔案；如果您想要使用麥克風，則可以略過此步驟：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. 請注意，**speaking-clock-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：speaking-clock-client.py

4. 開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言專屬的程式碼，以匯入您使用語音 SDK 所需的命名空間：

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. 此外，如果您的系統<u>沒有</u>可運作的麥克風，請在現有的命名空間匯入下，新增下列程式碼以匯入您將用來播放音訊檔案的程式庫：

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## <a name="create-an-intentrecognizer"></a>建立 *IntentRecognizer*

**IntentRecognizer** 類別會提供用戶端物件，可讓您使用語音輸入取得 Language Understanding 預測。

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入應用程式識別碼、預測區域和金鑰的程式碼。 然後尋找 **Configure speech service and get intent recognizer** 註解，並根據您要使用麥克風或音訊檔案進行語音輸入，新增下列程式碼：

    ### <a name="if-you-have-a-working-microphone"></a>**如果您有運作中的麥克風：**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### <a name="if-you-need-to-use-an-audio-file"></a>**如果您需要使用音訊檔案：**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## <a name="get-a-predicted-intent-from-spoken-input"></a>從語音輸入取得預測意圖

現在您已準備好實作程式碼，使用語音 SDK 從語音輸入取得預測意圖。

1. 在 **Main** 函式中，在您剛才新增的程式碼正下方尋找註解 **Get the model from the AppID and add the intents we want to use**，並新增下列程式碼，以根據其應用程式識別碼取得您的 Language Understanding 模型，並指定我們想要辨識器識別的意圖。

    **C#**

    ```C#
    // Get the model from the AppID and add the intents we want to use
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *請注意，您可以為每個意圖指定字串型的識別碼*

    **Python**

    ```Python
    # Get the model from the AppID and add the intents we want to use
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. 在註解 **Process speech input** 下，新增下列程式碼，此片段會使用辨識器，以語音輸入非同步呼叫 Language Understanding 服務，並擷取回應。 如果回應包含預測意圖，則會顯示口語查詢、預測意圖和完整 JSON 回應。 否則，程式碼會根據傳回的原因來處理回應。

**C#**

```C
// Process speech input
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

您到目前為止新增的程式碼會識別 *意圖*，但某些意圖可以參考 *實體*，因此您必須新增程式碼，以從服務傳回的 JSON 擷取實體資訊。

3. 在您剛新增的程式碼中，尋找註解 **Get the first entity (if any)** ，並在其下方新增下列程式碼：

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
您的程式碼現在會使用 Language Understanding 應用程式來預測意圖，以及輸入表達中偵測到的任何實體。 用戶端應用程式現在必須使用該預測來判斷和執行適當的動作。

4. 在您剛才新增的程式碼下方，尋找 **Apply the appropriate action** 註解，並新增下列程式碼，以檢查應用程式所支援的意圖 (**GetTime**、**GetDate** 和 **GetDay**)，並判斷是否已偵測到任何相關的實體，然後再呼叫現有的函式來產生適當的回應。

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## <a name="run-the-client-application"></a>執行用戶端應用程式

1. 儲存變更並返回 **speaking-clock-client** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. 如果使用麥克風，請大聲說出語句以測試應用程式。 例如，可嘗試下列語句 (每句都要重新執行程式)：

    *What's the time? (現在時間？)*
    
    *What time is it? (現在幾點？)*

    *What day is it? (今天是什麼日子？)*

    *What is the time in London? (倫敦現在幾點？)*

    *What's the date? (今天是幾月幾號？)*

    *What date is Sunday? (星期天是幾月幾號？)*

> **注意**：應用程式中的邏輯刻意簡單，而且有一些限制，但應該可以測試 Language Understanding 模型使用語音 SDK 從語音輸入預測意圖的能力。 由於辨識 **MM/DD/YYYY** 格式的日期尚有困難，您可能無法辨識特定日期實體的 *GetDay* 意圖！

## <a name="more-information"></a>詳細資訊

若要深入了解語音和 Language Understanding 整合，請參閱[語音文件](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition)。
