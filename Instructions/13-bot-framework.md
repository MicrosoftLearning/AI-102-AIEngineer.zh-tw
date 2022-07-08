---
lab:
  title: 使用 Bot Framework SDK 建立 Bot
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: ab51a1f11f4eee99838634d83b5d9261a8c20ecb
ms.sourcegitcommit: 480c5898009ea964025fdecb57900aefeeac81fd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2022
ms.locfileid: "147019881"
---
# <a name="create-a-bot-with-the-bot-framework-sdk"></a>使用 Bot Framework SDK 建立 Bot

*Bot* 是可以參與與人類使用者溝通對話的軟體代理程式。 Microsoft Bot Framework 提供完整的平台來組建可透過 Azure Bot Service 作為雲端服務的 Bot。

在此練習中，您將使用 Microsoft Bot Framework SDK 來建立及部署 Bot。

## <a name="before-you-start"></a>在您開始使用 Intune 之前

讓我們從準備用於 Bot 開發的環境開始著手。

### <a name="update-the-bot-framework-emulator"></a>更新 Bot Framework Emulator

您將會使用 Bot Framework SDK 來建立您的 Bot，並使用 Bot Framework Emulator 來測試 Bot。 Bot Framework Emulator 會定期更新，因此請確認您已安裝最新版本。

> **注意**：更新可能包含影響本練習中指示的使用者介面變更。

1. 啟動 **Bot Framework Emulator**，如果系統提示您安裝更新，請針對目前登入的使用者執行此動作。 如果未自動提示，請使用 [說明] 功能表上的 [檢查更新] 選項來檢查更新。
2. 安裝任何可用的更新之後，請關閉 Bot Framework Emulator，直到您稍後再次需要該 Bot Framework Emulator。

> **重要**：有時更新會下載失敗，正在調查中。 如果在嘗試更新幾分鐘內沒有進展，您可以停止下載並使用模擬器目前安裝的版本。

### <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未將 **AI-102-AIEngineer** 程式碼存放庫複製至您目前使用實驗室的環境，請遵循下列步驟來執行這項操作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git:Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (不限資料夾)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]。

## <a name="create-a-bot"></a>建立 Bot

您可以使用 Bot Framework SDK 根據範本建立 Bot，然後自訂程式碼以符合您的具體需求。

> **注意**：在此練習中，您可以選擇使用 **C#** 或 **Python**。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [Explorer] 窗格中，瀏覽至 [13-bot-framework] 資料夾，然後根據您的語言偏好設定展開 [C-Sharp] 或 [Python] 資料夾。
2. 以滑鼠右鍵按一下所選語言的資料夾，並開啟整合式終端。
3. 在終端中，執行下列命令來安裝您需要的 Bot 範本和套件：

**C#**

```C#
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```Python
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. 安裝範本和套件之後，請執行下列命令，根據 *EchoBot* 範本來建立 Bot：

**C#**

```C#
dotnet new echobot -n TimeBot
```

**Python**

```Python
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

如果您使用 Python，當收到來自 Cookiecutter 的提示時，請輸入下列詳細資料：
- **bot_name**：TimeBot
- **bot_description**：我們這個時代的 Bot
    
5. 在終端窗格中輸入下列命令，將目前目錄變更為 **TimeBot** 資料夾，列出已為您的 Bot 產生的程式碼檔案：

    ```Code
    cd TimeBot
    dir
    ```

## <a name="test-the-bot-in-the-bot-framework-emulator"></a>在 Bot Framework Emulator 中測試該 Bot

您已根據 *EchoBot* 範本建立 Bot。 現在，您可以在本機執行該 Bot，並使用 Bot Framework Emulator (應該安裝在您的系統) 對其進行測試。

1. 在終端窗格中，確定目前的目錄是包含 Bot 程式碼檔案的 **TimeBot** 資料夾，然後輸入下列命令來啟動在本機執行的 Bot。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
當 Bot 啟動時，請注意顯示其所執行的端點。 端點應類似 **http://localhost:3978** 。

2. 啟動 Bot Framework Emulator，並指定附加 **/api/messages** 路徑的端點以開啟您的 Bot，如下所示：

    `http://localhost:3978/api/messages`

3. 在 [即時聊天] 窗格中開啟對話之後，請等候 [Hello 和 welcome！] 訊息。
4. 輸入 *Hello* 之類的訊息，然後檢視來自 Bot 的回應，其應該會回應您輸入的訊息。
5. 關閉 Bot Framework Emulator 並返回 Visual Studio Code，然後在終端視窗中輸入 [CTRL+C] 以停止 Bot。

## <a name="modify-the-bot-code"></a>修改 Bot 程式碼

您已建立會回應使用者輸入的 Bot。 這並不特別有用，但可用於說明對話的基本流程。 與 Bot 的對話是由一連串的 *活動* 所組成，其中文字、圖形或使用者介面 *卡片* 可用於交換資訊。 Bot 會以問候語開始對話，這是使用者初始化 Bot 聊天工作階段時所觸發的 *對話更新* 活動結果。 然後對話是由使用者和 Bot 輪流傳送 *訊息* 的一連串進一步活動所組成。

1. 在 Visual Studio Code 中，開啟 Bot 的下列程式碼檔案：
    - **C#** ：TimeBot/Bots/EchoBot.cs
    - **Python**：TimeBot/bot.py

    請注意，此檔案中的程式碼是由 *活動處理常式* 函數所組成；其中一個用於 *成員新增* 對話更新活動，(當有人加入聊天工作階段)，另一個則用於 *訊息活動* (當收到訊息時)。 對話是以 *輪流* 的概念為基礎，每次輪流代表 Bot 接收、處理及回應活動的互動。 *輪流內容* 是用來追蹤目前輪流中正在處理活動的相關資訊。

2. 在程式碼檔案的頂端，新增下列命名空間匯入陳述式：

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. 修改 *Message* 活動的活動處理常式函數，以符合下列程式碼：

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. 儲存您的變更，然後在終端窗格中，確定目前的目錄是包含 Bot 程式碼檔案的 **TimeBot** 資料夾，然後輸入下列命令來啟動在本機執行的 Bot。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

和之前一樣，當 Bot 啟動時，請注意顯示其所執行的端點。

5. 啟動 Bot Framework Emulator，並指定附加 **/api/messages** 路徑的端點以開啟您的 Bot，如下所示：

    `http://localhost:3978/api/messages`

6. 在 [即時聊天] 窗格中開啟對話之後，請等候 [Hello 和 welcome！] 訊息。
7. 輸入 [Hello] 之類的訊息，並檢視 Bot 的回應，其回應應該是 *詢問我現在幾點*。
8. 輸入 *現在幾點？* ，並檢視回應。

    Bot 現在會回應「現在幾點？」查詢 透過顯示 Bot 執行所在的當地時間。 針對任何其他查詢，其會提示使用者詢問其「現在幾點」。 這是能力非常有限的 Bot，可透過與語言理解服務和其他自訂程式碼的整合來改善，但其可作為如何透過 Bot Framework SDK 來組建解決方案的工作範例，方法是擴充從範本所建立的 Bot。

9. 關閉 Bot Framework Emulator 並返回 Visual Studio Code，然後在終端視窗中輸入 [CTRL+C] 以停止 Bot。

## <a name="more-information"></a>詳細資訊

若要深入了解 Bot Framework，請檢視 [Bot Framework 文件](https://docs.microsoft.com/azure/bot-service/index-bf-sdk)。
