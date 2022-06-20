---
lab:
  title: 培養適用於 Azure 認知搜尋的自訂技能
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 1321115b5c06fb6791f24f9f89311524be14accc
ms.sourcegitcommit: 883a607fbe52f48a9425c38a997f776ecd0130af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/01/2022
ms.locfileid: "145195479"
---
# <a name="create-a-custom-skill-for-azure-cognitive-search"></a>培養適用於 Azure 認知搜尋的自訂技能

Azure 認知搜尋使用認知技能的擴充管線，從文件擷取 AI 產生的欄位，並將其包含在搜尋索引中。 有一組完整的內建技能可供您使用，但如果您有這些技能無法滿足的特定需求，您可以建立自訂技能。

在本練習中，您將建立自訂技能，將文件中各個字組的頻率製成表格，以產生前五個最常使用字組的清單，並將其新增至 Margie 旅遊 (虛構的旅遊機構) 的搜尋解決方案中。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室使用的環境，請在 Visual Studio Code 中予以開啟；否則，請依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟調色盤 (SHIFT+CTRL+P) 並執行 **Git：複製**  命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]。

## <a name="create-azure-resources"></a>建立 Azure 資源

> **注意**：如果您先前已完成 **[建立 Azure 認知搜尋解決方案](22-azure-search.md)** 練習，而且這些 Azure 資源仍在您的訂用帳戶中，您可以略過本節，然後從 **建立搜尋解決方案** 一節開始。 否則，請遵循下列步驟來佈建必要的 Azure 資源。

1. 在網頁瀏覽器上開啟位於 `https://portal.azure.com` 的 Azure 入口網站，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 檢視您訂用帳戶中的 **資源群組**。
3. 如果您使用受限制的訂用帳戶提供資源群組，請選取 [資源群組] 以檢視其屬性。 否則，請用您選擇的名稱建立新的資源群組，接著在建立後前往該群組。
4. 在資源群組的 [概觀] 頁面上，記下 [訂用帳戶識別碼] 和 [位置]。 在後續步驟中，您將需要這些值以及資源群組的名稱。
5. 在 Visual Studio Code 中，展開 [23-custom-search-skill] 資料夾，然後選取 [setup.cmd]。 您將使用此批次指令碼來執行所需 Azure 命令列介面 (CLI) 命令，以建立所需的 Azure 資源。
6. 以滑鼠右鍵按一下 [23-custom-search-skill] 資料夾，然後選取 [在整合式終端中開啟]。
7. 在終端窗格中，輸入下列命令來建立您 Azure 訂用帳戶的已驗證連線。

    ```
    az login --output none
    ```

8. 出現系統提示時，登入您的 Azure 訂用帳戶。 然後返回 Visual Studio Code，並等候登入流程完成。
9. 執行下列命令以列出 Azure 位置。

    ```
    az account list-locations -o table
    ```

10. 在輸出中，尋找對應資源群組位置的 **Name** 值 (例如，*美國東部* 的對應名稱為 *eastus*)。
11. 在 **setup.cmd** 指令碼中，將 **subscription_id**、**resource_group** 及 **location** 變數宣告修改為您的訂用帳戶識別碼、資源群組名及位置名稱的適當值。 然後儲存您的變更。
12. 在 **23-custom-search-skill** 資料夾的終端中，輸入下列命令以執行指令碼：

    ```
    setup
    ```

    > **注意**：搜尋 CLI 模組處於預覽狀態，而且可能會卡在 *- Running ..* 是最困難的操作。 如果此狀況持續超過 2 分鐘，請按 CTRL+C 取消長時間執行的作業，然後在系統詢問您是否要終止指令碼時，選取 [N]。 該作業之後應該會順利完成。
    >
    > 如果指令碼失敗，請確定您已使用正確的變數名稱儲存指令碼，然後再試一次。

13. 當指令碼完成時，請檢閱其顯示的輸出，並記下有關 您 Azure 資源的下列資訊 (您稍後會需要用到這些值)：
    - 儲存體帳戶名稱
    - 儲存體連接字串
    - 認知服務帳戶
    - 認知服務金鑰
    - 搜尋服務端點
    - 搜尋服務系統管理金鑰
    - 搜尋服務查詢金鑰

14. 在 Azure 入口網站中，重新整理資源群組，並驗證其中包含 Azure 儲存體帳戶、Azure 認知服務資源及 Azure 認知搜尋資源。

## <a name="create-a-search-solution"></a>建立搜尋解決方案

現在您已擁有必要的 Azure 資源，您可以建立包含下列元件的搜尋解決方案：

- 參考 Azure 儲存體容器中文件的 **資料來源**。
- 定義技能的擴充管線，以從文件中擷取 AI 產生欄位的 **技能**。
- 定義可搜尋文件資料列的 **索引**。
- 從資料來源擷取文件、套用技能，並填入索引的 **索引子**。

在本練習中，您將使用 Azure 認知搜尋 REST 介面來提交 JSON 要求，以建立這些元件。

1. 在 Visual Studio Code 的 [23-custom-search-skill] 資料夾中，展開 [create-search] 資料夾，然後選取 [data_source.json]。 此檔案包含名為 **margies-custom-data** 資料來源的 JSON 定義。
2. 以 Azure 儲存體帳戶的連接字串取代 **YOUR_CONNECTION_STRING** 預留位置，結果應該如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *您可以在 Azure 入口網站的 [存取金鑰] 頁面上，找到您儲存體帳戶的連接字串。*

3. 儲存並關閉已更新的 JSON 檔案。
4. 在 [create-search] 資料夾中，開啟 [skillset.json]。 此檔案包含名為 **margies-custom-skillset** 技能的 JSON 定義。
5. 在技能定義頂端的 [cognitiveServices] 元素中，將 [YOUR_COGNITIVE_SERVICES_KEY] 預留位置取代為認知服務資源的其中一個金鑰。

    *您可以在 Azure 入口網站中認知服務資源的 [金鑰與端點] 頁面上找到金鑰。*

6. 儲存並關閉已更新的 JSON 檔案。
7. 在 [create-search] 資料夾中，開啟 [index.json]。 此檔案包含名為 **margies-custom-index** 索引的 JSON 定義。
8. 檢閱索引的 JSON，然後直接關閉該檔案而不做任何變更。
9. 在 [create-search] 資料夾中，開啟 [indexer.json]。 此檔案包含名為 **margies-custom-indexer** 索引子的 JSON 定義。
10. 檢閱索引子的 JSON，然後直接關閉該檔案而不做任何變更。
11. 在 [create-search] 資料夾中，開啟 [create-search.cmd]。 此批次指令碼會使用 cURL 公用程式，將 JSON 定義提交至您 Azure 認知搜尋資源的 REST 介面。
12. 將 **YOUR_SEARCH_URL** 和 **YOUR_ADMIN_KEY** 變數預留位置取代為您 Azure 認知搜尋資源的 **URL** 及其中一個 **系統管理員金鑰**。

    *您可以在 Azure 入口網站中 Azure 認知搜尋資源的 [概觀] 和 [金鑰] 頁面上找到這些值。*

13. 儲存已更新的批次檔案。
14. 以滑鼠右鍵按一下 [create-search] 資料夾，然後選取 [在整合式終端中開啟]。
15. 在 [create-search] 資料夾的終端窗格中，輸入下列命令以執行批次指令碼。

    ```
    create-search
    ```

16. 當指令碼完成時，在 Azure 入口網站的 Azure 認知搜尋資源頁面上，選取 [索引子] 頁面，並等候編製索引程序完成。

    *您可以選取 [重新整理] 以追蹤編制索引作業的進度。此操作可能需要一分鐘左右才能完成。*

## <a name="search-the-index"></a>搜尋索引

現在您已擁有索引，您可以加以搜尋。

1. 在 Azure 認知搜尋資源的窗格頂端，選取 [搜尋檔案總管] 。
2. 在搜尋檔案總管中，[查詢字串] 的方塊中，輸入下列查詢字串，然後選取 [搜尋]。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    此查詢會擷取由 *評論者* 撰寫，並提及 *倫敦* 所有文件的 **url**、**sentiment** 和 **keyphrases**，該評論有正面 **情緒** 標籤 (換句話說，提及倫敦的正面評論)

## <a name="create-an-azure-function-for-a-custom-skill"></a>為自訂技能建立 Azure 函數

搜尋解決方案包含一些內建認知技能，可透過文件的資訊來擴充索引，例如情緒分數以及上一個工作中所看到的關鍵字組清單。

您可以藉由建立自訂技能來進一步增強索引。 例如，識別每個文件中最常使用的字組可能很有用，但沒有內建技能提供這項功能。

若要實作字組計數功能作為自訂技能，您將以慣用的語言建立 Azure 函數。

> **注意**：在本練習中，您將使用 Azure 入口網站中的程式碼編輯功能，建立簡單的 Node.JS 函數。 在生產解決方案中，您通常會使用 Visual Studio Code 等開發環境，以您慣用的語言建立函數應用程式 (例如 C#、Python、Node.JS 或 JAVA) 建立函數應用程式，並將其發佈至 Azure 作為 DevOps 流程的一部分。

1. 在 Azure 入口網站的 [首頁] 頁面上，使用下列設定建立新的 [函式應用程式] 資源：
    - **訂用帳戶**：您的訂用帳戶
    - **資源群組**：*與您 Azure 認知搜尋資源相同的資源群組*
    - **函數應用程式名稱**：*唯一名稱*
    - **發佈**：程式碼
    - **執行階段堆疊**：Node.js
    - **版本**：14 LTS
    - **區域**：*與您的 Azure 認知搜尋資源在同一區域*

2. 等候部署完成，然後移至已部署的函數應用程式資源。
3. 在函數應用程式的窗格中，於左側的窗格中選取 [函數] 索引標籤。然後建立一個包含以下設定的新函數：
    - **設定開發環境**」
        - **開發環境**：在入口網站中開發
    - **選取範本**」
        - **範本**：HTTP 觸發程序
    - **範本詳細資料**：
        - **新增函數**：wordcount
        - **授權等級**：函式
4. 等候建立 *wordcount* 函數。 然後在其頁面中，選取 [程式碼 + 測試] 索引標籤。
5. 以下列程式碼取代預設函數程式碼：

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. 儲存該函數，然後開啟 [測試/執行] 窗格。
7. 在 [測試/執行] 窗格中，將現有的 [Body] 取代為下列 JSON，以反映 Azure 認知搜尋技能所預期的結構描述，其中會提交一或多個文件的記錄進行處理：

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
8. 按一下 [執行] ，並檢視函數所傳回的 HTTP 回應內容。 這會反映取用技能時所預期 Azure 認知搜尋的結構描述，其中會傳回每個文件的回應。 在此情況下，回應包含每個文件中最多 10 個字詞，其出現頻率的遞減順序如下：

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

9. 關閉 [測試/執行] 窗格，然後在 [wordcount]  函數窗格中，按一下 [取得函數 URL] 。 然後將預設金鑰的 URL 複製到剪貼簿。 您將在下一個程序中需要使用該 URL。

## <a name="add-the-custom-skill-to-the-search-solution"></a>將自訂技能新增至搜尋解決方案

現在，您必須在搜尋解決方案技能中包含函數作為自訂技能，並將其所產生的結果對應至索引中的欄位。 

1. 在 Visual Studio Code 的 [23-custom-search-skill/update-search] 資料夾中，開啟 [update-skillset.json] 檔案。 這包含技能的 JSON 定義。
2. 檢閱技能定義。 其中包含與先前相同的技能，以及名為 **get-top-words** 的新 **WebApiSkill** 技能。
3. 編輯 **get-top-words** 技能定義，將 **uri** 值設定為 Azure 函數的 URL (您在上一個程式中複製到剪貼簿的程式)，取代 **YOUR-FUNCTION-APP-URL**。
4. 在技能定義頂端的 [cognitiveServices] 元素中，將 [YOUR_COGNITIVE_SERVICES_KEY] 預留位置取代為認知服務資源的其中一個金鑰。

    *您可以在 Azure 入口網站中認知服務資源的 [金鑰與端點] 頁面上找到金鑰。*

5. 儲存並關閉已更新的 JSON 檔案。
6. 在 [update-search] 資料夾中，開啟 [update-index.json]。 此檔案包含 **margies-custom-index** 索引的 JSON 定義，並在索引定義底部有一個名為 **top_words** 的額外欄位。
7. 檢閱索引的 JSON，然後直接關閉該檔案而不做任何變更。
8. 在 [update-search] 資料夾中，開啟 [update-index.json]。 此檔案包含 **margies-custom-indexer** 的 JSON 定義，以及 **top_words** 欄位的額外對應。
9. 檢閱索引子的 JSON，然後直接關閉該檔案而不做任何變更。
10. 在 [update-search] 資料夾中，開啟 [update-search.cmd]。 此批次指令碼會使用 cURL 公用程式，將已更新的 JSON 定義提交至您 Azure 認知搜尋資源的 REST 介面。
11. 將 **YOUR_SEARCH_URL** 和 **YOUR_ADMIN_KEY** 變數預留位置取代為您 Azure 認知搜尋資源的 **URL** 及其中一個 **系統管理員金鑰**。

    *您可以在 Azure 入口網站中 Azure 認知搜尋資源的 [概觀] 和 [金鑰] 頁面上找到這些值。*

12. 儲存已更新的批次檔案。
13. 以滑鼠右鍵按一下 [update-search] 資料夾，然後選取 [在整合式終端中開啟]。
14. 在 [update-search] 資料夾的終端窗格中，輸入下列命令以執行批次指令碼。

    ```
    update-search
    ```

15. 當指令碼完成時，在 Azure 入口網站的 Azure 認知搜尋資源頁面上，選取 [索引子] 頁面，並等候編製索引程序完成。

    *您可以選取 [重新整理] 以追蹤編制索引作業的進度。此操作可能需要一分鐘左右才能完成。*

## <a name="search-the-index"></a>搜尋索引

現在您已擁有索引，您可以加以搜尋。

1. 在 Azure 認知搜尋資源的窗格頂端，選取 [搜尋檔案總管] 。
2. 在搜尋檔案總管中，[查詢字串] 的方塊中，輸入下列查詢字串，然後選取 [搜尋]。

    ```
    search=Las Vegas&$select=url,top_words
    ```

    此查詢會擷取所有提及 **Las Vegas** 之文件的 **URL** 及 *top_words* 欄位。

## <a name="more-information"></a>詳細資訊

若要深入了解如何培養適用於 Azure 認知搜尋的自訂技能，請參閱 [Azure 認知搜尋文件](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)。
