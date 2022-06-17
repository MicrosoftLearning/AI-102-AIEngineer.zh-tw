---
lab:
  title: 翻譯文字
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 4ca4394ce5d9456abeabbdded1ee219f271f284e
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195472"
---
# <a name="translate-text"></a>翻譯文字

**翻譯工具** 服務是一項認知服務，可讓您翻譯不同語言的文字。

例如，假設旅遊機構想要檢查已提交至公司網站的旅館評論，並將英文標準化為用於分析的語言。 他們可以使用翻譯工具服務來判斷用以撰寫每篇評論的語言，如果尚未使用英文，請將其從其撰寫的任何來源語言翻譯成英文。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室使用的環境，請在Visual Studio Code中予以開啟；否則，依照下列步驟立即進行複製。

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
5. 部署資源後，移至該資源並檢視其 [金鑰和端點] 頁面。 在下一個程序中，您需要其中一個金鑰，以及從此頁面佈建服務的位置。

## <a name="prepare-to-use-the-translator-service"></a>準備使用翻譯工具服務

在此練習中，您將完成部分實作的用戶端應用程式，其會使用翻譯工具 REST API 來翻譯旅館評論。

> **注意**：您可以選擇從 **C#** 或 **Python** 使用 API。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **06-translate-text** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 檢視 **text-translation** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以納入認知服務資源的驗證 **金鑰**，以及其部署所在的 **位置** (「不是」<u></u>端點) - 您應該從認知服務資源的 [金鑰和端點] 頁面複製這兩者。 儲存您的變更。
3. 請注意，**text-translation** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：text-translation.py

    開啟程式碼檔案並檢查其中包含的程式碼。

4. 在 **Main** 函式中，請注意已經提供可從組態檔載入認知服務金鑰和區域的程式碼。 翻譯服務的端點也會在您的程式碼中指定。
5. 以滑鼠右鍵按一下 **text-translation** 資料夾，開啟整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

6. 觀察輸出，因為程式碼應該執行且不會發生錯誤，並在 **reviews** 資料夾中顯示每個評論文字檔的內容。 應用程式目前不會使用翻譯工具服務。 我們將在下一個程序中修正該問題。

## <a name="detect-language"></a>偵測語言

翻譯工具服務可以自動偵測要翻譯的文字來源語言，但也可讓您明確偵測撰寫文字的語言。

1. 在您的程式碼檔案中，尋找 **GetLanguage** 函式，此函式目前會針對所有文字值傳回 "en"。
2. 在 **GetLanguage** 函式的 **Use the Translator detect function** 註解之下，新增下列程式碼以使用翻譯工具的 REST API 來偵測指定文字的語言，小心不要取代可傳回語言之函式尾端的程式碼：

**C#**

```C#
// Use the Translator detect function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/detect?api-version=3.0";
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get language
        JArray jsonResponse = JArray.Parse(responseContent);
        language = (string)jsonResponse[0]["language"]; 
    }
}
```

**Python**

```Python
# Use the Translator detect function
path = '/detect'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0'
}

headers = {
'Ocp-Apim-Subscription-Key': cog_key,
'Ocp-Apim-Subscription-Region': cog_region,
'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get language
language = response[0]["language"]
```

3. 儲存變更並返回 **text-translation** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. 觀察輸出，請注意這次已識別每篇評論的語言。

## <a name="translate-text"></a>翻譯文字

既然您的應用程式可判斷用以撰寫評論的語言，您即可使用翻譯工具服務將任何非英文評論翻譯成英文。

1. 在您的程式碼檔案中，尋找 **Translate** 函式，此函式目前會針對所有文字值傳回空字串。
2. 在 **Translate** 函式的 **Use the Translator translate function** 註解之下，新增下列程式碼以使用翻譯工具的 REST API 將指定的文字從其來源語言翻譯成英文，小心不要取代可傳回翻譯之函式尾端的程式碼：

**C#**

```C#
// Use the Translator translate function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get translation
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```Python
# Use the Translator translate function
path = '/translate'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get translation
translation = response[0]["translations"][0]["text"]
```

3. 儲存變更並返回 **text-translation** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. 觀察輸出，請注意非英文評論會翻譯成英文。

## <a name="more-information"></a>詳細資訊

如需有關使用 **翻譯工具** 服務的詳細資訊，請參閱[翻譯工具文件](https://docs.microsoft.com/azure/cognitive-services/translator/)。
