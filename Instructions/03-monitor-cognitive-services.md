---
lab:
  title: 監視認知服務
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: e0e0042421a4f7150fc3b95cef80887c81a78f3a
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195466"
---
# <a name="monitor-cognitive-services"></a>監視認知服務

Azure 認知服務可以是整體應用程式基礎結構的關鍵部分。 務必能夠監視活動，並收到可能需要留意之問題的警示。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室使用的環境，請在 Visual Studio Code 中予以開啟；否則，依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

如果您的訂用帳戶中還沒有 **認知服務** 資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋「認知服務」，然後使用以下設定建立 **認知服務** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取任何必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署詳細資料。
5. 部署資源後，移至該資源並檢視其 [金鑰和端點] 頁面。 記下端點 URI - 您稍後會用到。

## <a name="configure-an-alert"></a>設定警示

讓我們藉由定義警示規則來開始監視，以便偵測認知服務資源中的活動。

1. 在Azure 入口網站中，移至您的認知服務資源，並檢視其 [警示] 頁面 (位於 [監視] 區段中)。
2. 選取 [+ 新增警示規則]
3. 在 [建立警示規則] 頁面的 [範圍] 之下，確認您的認知服務資源已列出。
4. 在 [條件] 之下，按一下 [新增條件]，然後檢視右側顯示的 [選取訊號] 窗格，您可以在其中選取要監視的訊號類型。
5. 在 [訊號類型] 清單中，選取 [活動記錄]，然後在經過篩選的清單中，選取 [列出金鑰]。
6. 檢閱過去 6 小時內的活動。
7. 選取 [動作] 索引標籤。請注意，您可以指定 *動作群組*。 這可讓您在引發警示時設定自動化動作，例如傳送電子郵件通知。 在此練習中，我們不會這麼做；但是在生產環境中這麼做可能很有用。
8. 在 [詳細資料] 索引標籤中，將 [警示規則名稱] 設定為 [金鑰清單警示]。
9. 選取 [檢閱 + 建立]。 
10. 檢閱警示的設定。 選取 [建立]，然後等候警示規則建立。
11. 在 Visual Studio Code 中，以滑鼠右鍵按一下 **03-monitor** 資料夾，然後開啟整合式終端。 然後輸入下列命令，以使用 Azure CLI 登入您的 Azure 訂用帳戶。

    ```
    az login
    ```

    網頁瀏覽器索引標籤將會開啟，並提示您登入 Azure。 這麼做，然後關閉瀏覽器索引標籤並返回 Visual Studio Code。

    > **秘訣**：如果您有多個訂用帳戶，您必須確定您在包含認知服務資源的訂用帳戶中工作。  使用下列命令來判斷您目前的訂用帳戶。
    >
    > ```
    > az account show
    > ```
    >
    > 如果您需要變更訂用帳戶，請執行此命令，將 *&lt;subscriptionName&gt;* 變更為正確的訂用帳戶名稱。
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. 現在，您可使用下列命令來取得認知服務金鑰清單、將 *&lt;resourceName&gt;* 取代為您的認知服務資源名稱，並將 *&lt;resourceGroup&gt;* 取代為您在其中建立的資源群組名稱。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

此命令會傳回認知服務資源的金鑰清單。

11. 切換回包含 Azure 入口網站的瀏覽器，然後重新整理您的 [警示] 頁面。 您應會看到列在資料表中的 **Sev 4** 警示 (如果沒有，請等候多達五分鐘，然後再次重新整理)。
12. 選取警示以查看其詳細資料。

## <a name="visualize-a-metric"></a>將計量視覺化

除了定義警示，您還可以檢視認知服務資源的計量，以監視其使用率。

1. 在 Azure 入口網站中，於認知服務資源的頁面中，選取 [計量] (位於 [監視] 區段中)。
2. 如果沒有現有的圖表，請選取 [+ 新增圖表]。 然後在 [計量] 清單中，檢閱您可以視覺化的可能計量，然後選取 [呼叫總數]。
3. 在 [彙總] 清單中，選取 [計數]。  這可讓您監視對您認知服務資源的呼叫總數，有助於判斷在一段時間內使用的服務數量。
4. 若要對認知服務產生一些要求，您將使用 **curl** - 適用於 HTTP 要求的命令列工具。 在 Visual Studio Code 的 **03-monitor** 資料夾中，開啟 **rest-test.cmd** 並編輯其中包含的 **curl** 命令 (如下所示)，將 *&lt;yourEndpoint&gt;* 和 *&lt;yourKey&gt;* 取代為您的端點 URI 和 **Key1** 金鑰，以在認知服務資源中使用文字分析 API。

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.1/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. 儲存您的變更，然後在 **03-monitor** 資料夾的整合式終端機中執行下列命令：

    ```
    rest-test
    ```

此命令會傳回 JSON 文件，其中包含在輸入資料中偵測到的語言相關資訊 (應該是英文)。

6. 重新執行 **rest-test** 命令多次，以產生一些呼叫活動 (您可以使用 **^** 金鑰來逐一查看先前的命令)。
7. 返回 Azure 入口網站中的 [計量] 頁面，然後重新整理 [呼叫總數] 計數圖表。 使用 *curl* 進行的呼叫可能需要幾分鐘的時間才會反映在圖表中- 請持續重新整理圖表，直到更新成包含這些呼叫為止。

## <a name="more-information"></a>詳細資訊

監視認知服務的其中一個選項是使用「診斷記錄」。 啟用後，診斷記錄會擷取有關認知服務資源使用量的豐富資訊，並可成為實用的監視和偵錯工具。 在設定診斷記錄之後，可能需要一小時以上的時間才能產生任何資訊，這就是我們在此練習中尚未探索其原因；但您可以在[認知服務文件](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging)中深入了解。
