---
lab:
  title: 使用 Bot Framework Composer 建立 Bot
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: f25609df8d9abc29e691bd83d0470561c9e3e4b0
ms.sourcegitcommit: b934aa694b86756d8b297a384cc6b707f0536e57
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2021
ms.locfileid: "145195482"
---
# <a name="create-a-bot-with-bot-framework-composer"></a>使用 Bot Framework Composer 建立 Bot

Bot Framework Composer 是一個圖表化設計工具，可讓您快速且輕鬆地建立複雜的對話式 Bot，而不需要撰寫程式碼。 Composer 是一個開放原始碼工具，提供用來建立 Bot 的視覺化畫布。

## <a name="prepare-to-develop-a-bot"></a>準備開發 Bot

讓我們從準備開發 Bot 所需的服務和工具開始著手。

### <a name="get-an-openweather-api-key"></a>取得 OpenWeather API 金鑰

在此練習中，您將建立使用 OpenWeather 服務的 Bot，以便為使用者輸入的城市擷取其天氣狀況。 您將需要 API 金鑰，服務才能順利運作。

1. 在網頁瀏覽器中，移至位於 `https://openweathermap.org/price` 的 OpenWeather 網站。
2. 要求免費的 API 金鑰，並建立 OpenWeather 帳戶 (如果您尚未擁有帳戶的話)。
3. 註冊之後，請檢視 [API 金鑰] 頁面以查看您的 API 金鑰。

### <a name="update-bot-framework-composer"></a>更新 Bot Framework Composer

您將使用 Bot Framework Composer 來建立您的 Bot。 此工具會定期更新，因此請確認您已安裝最新版本。

> **注意**：更新可能包含影響本練習中指示的使用者介面變更。

1. 請啟動 **Bot Framework Composer**，如果系統未自動提示您安裝更新，請使用 [說明] 功能表上的 [檢查更新] 選項來檢查更新。
2. 如果有可用的更新，請選擇在關閉應用程式時對其進行安裝的選項。 然後關閉 Bot Framework Composer 並安裝目前登入使用者的更新，並在安裝完成後重新啟動 Bot Framework Composer。 安裝可能需要幾分鐘的時間。
3. 請確定 Bot Framework Composer 版本為 **2.0.0** 或更新版本。

## <a name="create-a-bot"></a>建立 Bot

現在您已準備好使用 Bot Framework Composer 來建立 Bot。

### <a name="create-a-bot-and-customize-the-welcome-dialog-flow"></a>建立 Bot，並自訂「歡迎」對話方塊流程

1. 若您尚未開啟 Bot Framework Composer，請加以啟動。
2. 在 **首頁** 畫面控制項上，選取 [新增] 。 然後建立新的空白 Bot；將其命名為 **WeatherBot**，並將 Bot 儲存在本機資料夾中。
3. 如果 **入門** 窗格為開啟狀態，請關閉該窗格，然後在左側導覽窗格中，選取 [問候] 以開啟 [製作畫布]，並顯示使用者一開始加入 Bot 交談時所呼叫的 *ConversationUpdate* 活動。 該活動是由動作流程所組成。
4. 在右側的屬性窗格中，選取右側屬性窗格頂端的 [Greeting]  字組，並將其變更為 [WelcomeUsers] ，以編輯 **Greeting** 的標題。
5. 在 [製作畫布] 中，選取 [傳送回應] 動作。 然後，在 [屬性] 窗格中，將預設文字從 *問候您的 Bot* 變更為 `Hi! I'm WeatherBot.`
6. 在 [製作畫布] 中，選取最後 **+** 一個符號 (其位於標示對話方塊流程 <u>結尾</u> 的圓形上方)，然後新增 [詢問問題] 動作以取得 **文字** 回應。

    新動作會在對話流程中建立兩個節點。 第一個節點定義提示 Bot 詢問使用者問題，而第二個節點代表將會從使用者收到的回應。 在屬性窗格中，這些節點具有對應的 [Bot 回應] 及 [使用者輸入] 索引標籤。

7. 在 [屬性] 窗格中的 [Bot 回應] 索引標籤上，新增包含文字 `What's your name?` 的回應。 然後，在 [使用者輸入] 索引標籤上，將 [屬性] 值設定為 `user.name`，以定義您可以在稍後在 Bot 對話中存取的變數。
8. 回到 [製作畫布]，選取 **+** 符號，其在您剛才新增的 **使用者輸入 (文字)** 動作底下，然後新增 [傳送回應] 動作。
9. 選取新增的 [傳送回應] 動作，然後在 [屬性] 窗格中，將文字值設定為 `Hello ${user.name}, nice to meet you!`。

    完整活動流程應如下所示：

    ![歡迎使用者並要求使用者名稱的對話方塊流程](./images/welcomeUsers.png)

### <a name="test-the-bot"></a>測試 Bot

您的基本 Bot 已完成，現在讓我們對其進行測試。

1. 選取 Composer 右上角的 [啟動 Bot] ，然後等候 Bot 編譯並啟動。 這可能需要數分鐘的時間。

    - 如果系統顯示 Windows 防火牆訊息，請啟用所有網路的存取權。

2. 在 [本機 Bot 執行階段管理員] 窗格中，選取 [開啟網路聊天]。
3. 在 **WeatherBot** 網路聊天窗格中，短暫暫停之後，您會看到歡迎訊息及輸入您名字的提示。  輸入您的名字，然後按 [Enter] 鍵。
4. 該 Bot 應該以 **您好 *your_name*，很高興認識您！** 回應。
5. 關閉網路聊天面板。
6. 在 [Composer] 右上方，[&#8635;重新開機 Bot] 旁，按一下 **<u>=</u>** 以開啟 [本機 Bot 執行階段管理員] 窗格，並使用⏹圖示來停止 Bot。

## <a name="add-a-dialog-to-get-the-weather"></a>新增對話方塊以取得天氣

現在您已擁有可運作的 Bot，您可以藉由新增特定互動的對話方塊來擴充其功能。 在此情況下，您將新增對話方塊，當使用者提及「天氣」時就會觸發。

### <a name="add-a-dialog"></a>新增對話方塊

首先，您必須定義對話流程，其用來處理天氣相關問題。

1. 在 [Composer] 的瀏覽窗格中，將滑鼠停留在最上層節點 ([WeatherBot] )，然後在 **...** 功能表中選取 [+ 新增對話方塊]，如下所示：

    ![新增對話方塊功能表](./images/add-dialog.png)

    然後建立名為 **GetWeather** 的新對話方塊，其描述為 **取得所提供郵遞區號的目前天氣狀況**。
2. 在瀏覽窗格中，選取新 [GetWeather] 對話方塊的 [BeginDialog] 節點。 然後在 [製作畫布] 上，使用 **+** 符號來新增[文字] 回應的 [詢問問題] 動作。
3. 在 [屬性] 窗格中的 [Bot 回應] 索引標籤上，新增 `Enter your city.` 的回應
4. 在 [使用者輸入] 索引標籤上，將 [屬性] 欄位設定為 `dialog.city`，並將 [輸出格式] 欄位設定為運算式 `=trim(this.value)`，以移除使用者提供值周圍任何多餘的空格。

    到目前為止的活動流程應該如下所示：

    ![具有一個「傳送回應」動作的對話方塊流程](./images/getWeather-dialog-1.png)

    到目前為止，對話方塊會請求使用者輸入城市。 現在您必須實作邏輯來擷取輸入城市的天氣資訊。

6. 在 [製作畫布] 上，在城市輸入的 [使用者輸入] 動作底下，選取 **+** 符號以新增動作。
7. 從動作清單中，選取 [存取外部資源] ，然後 選取 [傳送 HTTP 要求]。
8. 設定 **HTTP 要求** 的屬性，如下所示，以 [OpenWeather API](https://openweathermap.org/price) 金鑰取代 **YOUR_API_KEY**：
    - **HTTP 方法**：GET
    - **Url**：`http://api.openweathermap.org/data/2.5/weather?units=metric&q=${dialog.city}&appid=YOUR_API_KEY`
    - **結果屬性**：`dialog.api_response`

    該結果可以包含來自 HTTP 回應的下列四個屬性之一：

    - **statusCode**。 透過 **dialog.api_response.statusCode** 存取。
    - **reasonPhrase**。 透過 **dialog.api_response.reasonPhrase** 存取。
    - **內容**。 透過 **dialog.api_response.content** 存取。
    - **標題**。 透過 **dialog.api_response.headers** 存取。

    此外，如果回應類型為 JSON，其將會是可透過 **dialog.api_response.content** 屬性取得的還原序列化物件。 如需 OpenWeather API 及其傳回響應的詳細資訊，請參閱 [OpenWeather API 文件](https://openweathermap.org/current)。

    現在您需要將邏輯新增至處理回應的對話方塊流程，這可能表示 HTTP 要求的成功或失敗。

9. 在製作畫布中您所建立的 [傳送 HTTP 要求] 動作底下，新增 [建立條件分支：if/else]  >  動作。 此動作會使用 **True** 和 **False** 路徑來定義對話流程中的分支。
10. 在分支動作的 [屬性] 中，設定 [條件] 欄位以寫入下列運算式：

    ```
    =dialog.api_response.statusCode == 200
    ```

11. 如果呼叫已成功，您必須將回應儲存在變數中。 在 [製作畫布] 的 [True] 分支中，新增 [管理屬性]  >  [設定屬性] 動作。 然後在 [屬性] 窗格中，新增下列屬性設定：

    | 屬性 | 值 |
    | -- | -- |
    | `dialog.weather` | `=dialog.api_response.content.weather[0].description` |
    | `dialog.temp` | `=round(dialog.api_response.content.main.temp)` |
    | `dialog.icon` | `=dialog.api_response.content.weather[0].icon` |

12. 仍在 [True] 分支中的 [設定屬性] 動作底下新增 [傳送回應] 動作，並將其文字設定為：

    ```
    The weather in ${dialog.city} is ${dialog.weather} and the temperature is ${dialog.temp}&deg;.
    ```

    ***備註**：此訊息會使用您在先前動作中設定的 **dialog.city**、**dialog.weather** 和 **dialog.temp** 屬性。 稍後，您也將會使用 **dialog.icon** 屬性。*

13. 您也需要考量來自非 200 的天氣服務回應，因此，請在 [False] 分支中，新增 [傳送回應] 動作，並將其文字設定為 `I got an error: ${dialog.api_response.content.message}.`

    對話流程現在應如下所示：

    ![具有 HTTP 回應結果分支的對話流程](./images/getWeather-dialog-2.png)

### <a name="add-a-trigger-for-the-dialog"></a>新增對話方塊的觸發程序

現在您需要某種方法來從現有的歡迎對話起始新的對話方塊。

1. 在瀏覽窗格中，選取包含 [WelcomeUsers] 的 [WeatherBot] 對話方塊 (這是在相同名稱的最上層 Bot 節點下)。

    ![已選取的 WeatherBot 工作流程](./images/select-workflow.png)

2. 在所選 [WeatherBot] 對話方塊的 [屬性窗格] 中，於 [語言理解] 區段中，將 [辨識器類型] 設定為 [規則運算式辨識器]。

    > 預設辨識器類型會使用語言理解服務，以自然語言理解模型來預測使用者的意圖。 我們使用規則運算式辨識器來簡化此練習。 在實際的應用程式中，您應該考慮使用語言理解來允許更複雜的意圖辨識。

3. 在 [WeatherBot] 對話方塊的 **...** 功能表中，選取 [新增觸發程序]。

    ![新增觸發程序功能表](./images/add-trigger.png)

    然後使用下列設定建立觸發程序：

    - **此觸發程序的類型為何？** ：已識別意圖
    - **此觸發程序的名稱為何？(RegEx)** ：`WeatherRequested`
    - **請輸入 RegEx 模式**：`weather`

    > 在 RegEx 模式文字輸入框中輸入的文字是簡單的規則運算式模式，其將會導致 Bot 在任何內送郵件中尋找 *天氣* 一字。  如果「天氣」這詞出現，訊息就會變成 **已辨識的意圖**，並起始觸發程序。

4. 現在已建立觸發程序，您必須為其設定動作。 在觸發程序的製作畫布中，選取您新的 [WeatherRequested]  觸發程序節點底下的 **+** 符號。 然後在動作清單中，選取 [對話方塊管理] ，然後選取 [開始新的對話]。
5. 選取 [開始新的對話方塊] 動作之後，在 [屬性] 窗格中，從 [對話方塊名稱] 下拉式清單中選取[GetWeather] 對話方塊，以啟動您稍早在辨識 **WeatherRequested** 觸發程式時定義的 **GetWeather** 對話方塊。

    **WeatherRequested** 活動流程應如下所示：

    ![RegEx 觸發程序會開始 GetWeather 對話方塊](./images/weather-regex.png)

6. 重新啟動 Bot 並開啟網路聊天窗格。然後重新啟動對話，並在輸入您的名稱之後，輸入 `What is the weather like?` 。 然後，當出現提示時，輸入城市，例如 `Seattle`。 Bot 會連絡該服務，並應回應小型天氣報告陳述式。
7. 完成測試後，請關閉網路聊天窗格，並停止 Bot。

## <a name="handle-interruptions"></a>處理中斷

設計良好的 Bot 應該能讓使用者變更對話流程，例如藉由取消要求進行變更。

1. 在 Bot Composer 的瀏覽窗格中，使用 [WeatherBot] 對話方塊的 **...** 功能表來新增新的觸發程序 (除了現有的 **WelcomeUsers** 和 **WeatherRequested** 觸發程序之外)。 新的觸發程序應具有下列設定：

    - **此觸發程序的類型為何？** ：已識別意圖
    - **此觸發程序的名稱為何？(RegEx)** ：`CancelRequest`
    - **請輸入 RegEx 模式**：`cancel`

    > 在 RegEx 模式文字輸入框中輸入的文字是簡單的規則運算式模式，其將會導致 Bot 在任何內送郵件中尋找 *取消* 一字。

2. 在觸發程序的製作畫布中，新增 [傳送回應] 動作，並將其文字回應設定為 `OK. Whenever you're ready, you can ask me about the weather.`
3. 在 [傳送回應] 動作底下，選取 [對話方塊管理] 和 [結束此對話方塊] ，以新增動作來結束對話方塊。

    **CancelRequest** 對話流程應如下所示：

    ![具有 [傳送回應] 和 [結束此對話方塊動作] 的 、CancelRequest 觸發程序](./images/cancel-dialog.png)

    既然您現在有觸發程序可回應使用者的取消要求，您必須允許對話流程中斷，讓使用者可能想要提出這類要求 - 例如在要求天氣資訊之後，提示輸入郵遞區號時。

4. 在瀏覽窗格中，選取 [GetWeather] 對話方塊下的 [BeginDialog]。
5. 選取 [提示輸入文字] 動作，要求使用者輸入其城市。
6. 在該動作的屬性中，展開 [其他] 索引標籤上的 [提示設定] ，並將 [允許中斷] 屬性設定為 [true]。
7. 重新啟動 Bot，並開啟網路聊天窗格。 重新啟動對話，然後在輸入您的名稱之後，輸入 `What is the weather like?`。 然後在出現提示時，輸入 `cancel`，並確認要求已取消。
8. 取消要求之後，請輸入 `What's the weather like?`，並注意適當的觸發程序會啟動 **GetWeather** 對話方塊的新執行個體，並再次提示您輸入城市。
9. 完成測試後，請關閉網路聊天窗格，並停止 Bot。

## <a name="enhance-the-user-experience"></a>提升使用者體驗

到目前為止，與天氣 Bot 一直是透過文字進行互動。  使用者輸入其文字意圖，而 Bot 會以文字回應。 雖然文字通常是合適的溝通方式，但您可以透過其他形式的使用者介面元素來提升體驗。  例如，您可以使用按鈕來起始建議的動作，或顯示 *卡片*，以視覺化方式呈現資訊。

### <a name="add-a-button"></a>新增按鈕

1. 在 Bot Framework Composer 瀏覽窗格的 [GetWeather] 動作下方，選取 [BeginDialog]。
2. 在製作畫布中，選取包含城市提示的 [文字動作提示]。
3. 在 [屬性] 窗格中，選取 [顯示程式碼] ，並以下列程式碼取代現有的程式碼。

```
[Activity    
    Text = Enter your city.
    SuggestedActions = Cancel
]
```

此活動將會提示使用者輸入其城市，但也會顯示 [取消] 按鈕。

### <a name="add-a-card"></a>新增卡片

1. 在 [GetWeather] 對話方塊中，在檢查 HTTP 天氣服務的回應之後的 [True] 路徑中，選取 [傳送回應動作] ，其會顯示天氣報告。
2. 在 [屬性] 窗格中，選取 [顯示程式碼] ，並以下列程式碼取代現有的程式碼。

```
[ThumbnailCard
    title = Weather for ${dialog.city}
    text = ${dialog.weather} (${dialog.temp}&deg;)
    image = http://openweathermap.org/img/w/${dialog.icon}.png
]
```

此範本將會使用與之前相同的變數作為天氣條件，但也會將標題新增至之後會顯示的卡片，以及天氣狀況的圖片。

### <a name="test-the-new-user-interface"></a>測試新使用者介面

1. 重新啟動 Bot，並開啟網路聊天窗格。 重新啟動對話，並在輸入您的名稱之後，輸入 `What is the weather like?`。 然後，在出現提示時，按一下 [取消] 按鈕以取消該要求。
2. 取消之後，輸入 `Tell me about the weather`，並在出現提示時，輸入城市，例如 `London`。 Bot 會連絡該服務，並應回應指出天氣狀況的卡片。
3. 當您完成測試後，請關閉模擬器，並停止 Bot。

## <a name="more-information"></a>詳細資訊

若要深入了解 Bot Framework Composer，請檢視 [Bot Framework Composer 文件](https://docs.microsoft.com/composer/introduction)。
