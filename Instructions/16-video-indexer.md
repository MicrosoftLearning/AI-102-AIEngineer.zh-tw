---
lab:
  title: 使用影片分析器分析影片
  module: Module 8 - Getting Started with Computer Vision
ms.openlocfilehash: 50223cdfeb0a22933858d595d9329f8b8dcd873d
ms.sourcegitcommit: e20d9099aaecdefa62a763dae24833b97e3d9f6d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2022
ms.locfileid: "147052851"
---
# <a name="analyze-video-with-video-analyzer"></a>使用影片分析器分析影片

現今大部分建立和取用的資料都是視訊格式。 **適用於媒體的影片分析器** 是 AI 支援的服務，可用來編製影片的索引並從中擷取深入解析。

> **注意**：從 2022 年 6 月 21 日開始，傳回個人識別資訊的認知服務功能僅限已獲授與[有限存取權](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客戶使用。 若未獲得有限存取權核准，則無法針對此實驗室使用影片分析器辨識人物和名人。 如需 Microsoft 所做之變更和原因的詳細資訊，請參閱[臉部辨識的負責任 AI 投資和保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (英文)。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在進行此實驗室的環境，請在 Visual Studio Code 中予以開啟；否則，依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git:Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (不限資料夾)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來進行組建和偵錯，請選取 [現在不要]。

## <a name="upload-a-video-to-video-analyzer"></a>將影片上傳至影片分析器

首先，您必須登入影片分析器入口網站並上傳影片。

> **秘訣**：如果 [影片分析器] 頁面在託管的實驗室環境中載入速度緩慢，請使用本機安裝的瀏覽器。 您可以切換回到裝載的 VM，以便進行後續的工作。

1. 在您的瀏覽器中，開啟影片分析器入口網站 (位於 `https://www.videoindexer.ai`)。
2. 如果您有現有的影片分析器帳戶，請登入。 否則，註冊免費帳戶，並使用您的 Microsoft 帳戶 (或任何其他有效的帳戶類型) 進行登入。 如果登入時遇到困難，請嘗試開啟私人瀏覽器工作階段。
3. 在影片分析器中，選取 [上傳] 選項。 然後選取用以 **輸入檔案 URL** 的選項，然後輸入 `https://aka.ms/responsible-ai-video`。 將預設名稱變更為 **負責任 AI**、檢閱預設設定、選取核取方塊來驗證 Microsoft 臉部辨識原則的合規性，以及上傳檔案。
4. 上傳檔案之後，請稍候幾分鐘，影片分析器會自動為其編製索引。

> **注意**：在此練習中，我們會使用此影片來探索影片分析器功能；但是當您完成練習後，您應該花一些時間完整觀看影片，因為其中包含實用資訊和指引，以便負責地開發已啟用 AI 的應用程式！ 

## <a name="review-video-insights"></a>檢閱影片深入解析

索引編製程序會從影片中擷取深入解析，您可在入口網站中檢視。

1. 在影片分析器入口網站中，影片編製索引後，請加以選取進行檢視。 您會看到窗格中顯示從影片擷取的深入解析，該窗格旁邊則有影片播放程式。

![具有影片播放程式的影片分析器和深入解析窗格](./images/video-indexer-insights.png)

2. 視訊播放時，選取 [時間軸] 索引標籤以檢視視訊音訊的文字記錄。

![具有影片播放程式的影片分析器和顯示影片文字記錄的時間軸窗格。](./images/video-indexer-transcript.png)

3. 在入口網站的右上方，選取 [檢視] 符號 (看起來類似 &#128455;)，然後在深入解析清單中，除了 [文字記錄] 之外，選取 [OCR] 和 [說話者]。

![已選取具有 [文字記錄]、[OCR] 和 [說話者] 的影片分析器檢視功能表](./images/video-indexer-view-menu.png)

4. 觀察 [時間軸] 窗格現在包含：
    - 音訊旁白的文字記錄。
    - 影片中可見的文字。
    - 影片中出現之說話者的指示。 有些知名人物會自動依名稱辨識，其他人物則是以數字表示 (例如「說話者 #1」)。
5. 切換回到 [深入解析] 窗格，然後檢視該處顯示的深入解析。 其中包括：
    - 影片中出現的個別人物。
    - 影片中討論的主題。
    - 影片中出現之物件的標籤。
    - 具名實體，例如出現在影片中的人物和品牌。
    - 主要場景。
6. 在 [深入解析] 窗格顯示的情況下，再次選取 [檢視] 符號，然後在深入解析清單中，將 [關鍵字] 和 [情感] 新增至窗格。

    找到的深入解析可協助您判斷影片中的主要主題。 例如，此影片的 **主題** 顯示其顯然關於技術、社會責任和道德。

## <a name="search-for-insights"></a>搜尋深入解析

您可以使用影片分析器來搜尋影片以取得深入解析。

1. 在 [深入解析] 窗格的 [搜尋] 方塊，輸入「蜜蜂」。 您可能需要在 [深入解析] 窗格中向下捲動，才能查看所有類型的深入解析結果。
2. 觀察已找到一個相符的「標籤」，而其在影片中的位置於下方指出。
3. 選取指出蜜蜂存在其中的區段開頭，並在該時間點檢視影片 (您可能需要暫停影片並仔細選取 -蜜蜂只會短暫出現！)
4. 清除 [搜尋] 方塊以顯示影片的所有深入解析。

![蜜蜂的影片分析器搜尋結果](./images/video-indexer-search.png)

## <a name="use-video-analyzer-widgets"></a>使用影片分析器小工具

影片分析器入口網站是管理影片檢索專案的實用介面。 不過，您可能想要讓影片及其深入解析可供無法存取您的影片分析器帳戶的人員使用。 影片分析器會提供一些小工具，讓您為此目的內嵌於網頁中。

1. 在 Visual Studio Code 的 **16-video-indexer** 資料夾中，開啟 **analyze-video.html**。 這是您將新增影片分析器 [播放程式] 和 [深入解析] 小工具的基本 HTML 頁面。 請注意標頭中 **vb.widgets.mediator.js** 指令碼的參考 - 此指令碼可讓頁面上的多個影片分析器小工具彼此互動。
2. 在影片分析器入口網站中，返回 [媒體檔案] 頁面，然後開啟您的 [負責任 AI] 影片。
3. 在影片播放程式之下，選取 [&lt;/&gt; 內嵌] 可檢視 HTML iframe 程式碼以內嵌小工具。
4. 在 [共用和內嵌] 對話方塊中，選取 [播放程式] 小工具、將視訊大小設定為 560 x 315，然後將內嵌程式碼複製到剪貼簿。
5. 在 Visual Studio Code 中，將所複製的程式碼貼在 **analyze-video.html** 檔案中的 **&lt;-- Player widget goes here -- &gt;** 註解之下。
6. 回到 [共用和內嵌] 對話方塊中，選取 [深入解析] 小工具，然後將內嵌程式碼複製到剪貼簿。 接著關閉 [共用和內嵌] 對話方塊、切換回到 Visual Studio Code，然後將所複製的程式碼貼在 **&lt;-- Insights widget goes here -- &gt;** 註解之下。
7. 儲存檔案。 然後在 [總管] 窗格中，以滑鼠右鍵按一下 **analyze-video.html** 並選取 [在檔案總管中顯示]。
8. 在檔案總管中，在您的瀏覽器中開啟 **analyze-video.html** 以查看網頁。
9. 透過小工具進行試驗，使用 [深入解析] 小工具來搜尋深入解析，並跳到影片中的深入解析。

![網頁中的影片分析器小工具](./images/video-indexer-widgets.png)

## <a name="use-the-video-analyzer-rest-api"></a>使用影片分析器 API

影片分析器提供 REST API，可供您上傳和管理您帳戶中的影片。

### <a name="get-your-api-details"></a>取得 API 詳細資料

若要使用影片分析器 API，您需要一些資訊來驗證要求：

1. 在影片分析器入口網站中，展開功能表 (≡) 並選取 [帳戶設定] 頁面。
2. 請注意此頁面上的 [帳戶識別碼] - 您稍後會用到。
3. 開啟新的瀏覽器索引標籤，移至影片分析器開發人員入口網站 (位於 `https://api-portal.videoindexer.ai`)，然後使用影片分析器帳戶的認證進行登入。
4. 在 [設定檔] 頁面上，檢視與您的設定檔相關聯的 [訂用帳戶]。
5. 在具有您訂用帳戶的頁面上，觀察每個訂用帳戶都已獲得兩個金鑰 (主要和次要)。 然後針對任何金鑰選取 [顯示] 加以查看。 您很快就需要此金鑰。

### <a name="use-the-rest-api"></a>使用 REST API

現在您已擁有帳戶識別碼和 API 金鑰，即可使用 REST API 來處理帳戶中的影片。 在此程序中，您將使用 PowerShell 指令碼進行 REST 呼叫；但相同的準則適用於 HTTP 公用程式，例如 cURL 或 Postman，或任何能夠透過 HTTP 傳送和接收 JSON 的程式開發語言。

所有與影片分析器 REST API 的互動都會遵循相同的模式：

- 標頭中具有 API 金鑰之 **AccessToken** 方法的初始要求用於取得存取權杖。
- 後續要求會在呼叫 REST 方法來處理影片時，使用存取權杖進行驗證。

1. 在 Visual Studio Code 的 **16-video-indexer** 資料夾中，開啟 **get-videos.ps1**。
2. 在 PowerShell 指令碼中，將 **YOUR_ACCOUNT_ID** 和 **YOUR_API_KEY** 預留位置取代為您先前識別的帳戶識別碼和 API 金鑰值。
3. 觀察免費帳戶的 *位置* 是「試用版」。 如果您已建立不受限制的影片分析器帳戶 (具有相關聯的 Azure 資源)，即可將此變更為 Azure 資源的佈建位置 (例如 "eastus")。
4. 檢閱指令碼中的程式碼，請注意叫用兩個 REST 方法：一個用以取得存取權杖，另一個用以列出您帳戶中的影片。
5. 儲存您的變更，然後在指令碼窗格的右上方，使用 **&#9655;** 按鈕來執行指令碼。
6. 檢視來自 REST 服務的 JSON 回應，其中應該包含您先前編製索引之 **負責任 AI** 影片的詳細資料。

## <a name="more-information"></a>詳細資訊

人物和名人辨識仍可供使用，但需遵循[負責任 AI 標準](https://aka.ms/aah91ff)，並受有限存取權原則所限制。 這些功能包括臉部識別和名人辨識。 如需深入了解及申請存取權，請參閱[認知服務的有限存取權](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access) (英文)。

如需 **影片分析器** 的詳細資訊，請參閱 [影片分析器文件](https://docs.microsoft.com/azure/azure-video-analyzer/video-analyzer-for-media-docs/)。
