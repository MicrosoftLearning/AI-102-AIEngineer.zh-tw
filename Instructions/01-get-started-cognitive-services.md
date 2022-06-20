---
lab:
  title: 開始使用認知服務
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: a05256a78dee051041320aa3556a43add5596ce9
ms.sourcegitcommit: 5ffc20f6a590fe643c2b695b8dc04589411be36e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/31/2022
ms.locfileid: "145951187"
---
# <a name="get-started-with-cognitive-services"></a>開始使用認知服務

在此練習中，您將在 Azure 訂用帳戶中建立 **認知服務** 資源並從用戶端應用程式使用，藉此開始使用認知服務。 練習的目的並非取得任何特定服務的專業知識，而是要熟悉以開發人員身分佈建和使用認知服務的一般模式。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未將 **AI-102-AIEngineer** 程式碼存放庫複製到您正此實驗室使用的環境，請遵循下列步驟來執行這項操作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git:Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

Azure 認知服務是雲端式服務，可封裝您可併入應用程式的人工智慧功能。 您可以佈建特定 API 的個別認知服務資源 (例如 **語言** 或 **電腦視覺**)，也可以佈建一般 **認知服務** 資源，透過單一端點和金鑰來存取多個認知服務 API。 在此情況下，您將使用單一 **認知服務** 資源。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋「認知服務」，然後使用以下設定建立 **認知服務** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署的詳細資料。
5. 移至資源並檢視其 [金鑰和端點] 頁面。 此頁面包含您需要連線到資源的資訊，並從您開發的應用程式加以使用。 具體來說：
    - 用戶端應用程式可以傳送要求的 HTTP 端點。
    - 兩個可用於驗證的金鑰 (用戶端應用程式可以使用任一金鑰來驗證)。
    - 裝載資源的 *位置*。 這對於某些 API (但非所有) 的要求而言是必要的。

## <a name="use-a-rest-interface"></a>使用 REST 介面

認知服務 API 是以 REST 為基礎，因此您可透過 HTTP 提交 JSON 要求以取用。 在此範例中，您將探索使用 **語言** REST API 來執行語言偵測的主控台應用程式；但認知服務資源所支援的所有 API 的基本準則都相同。

> **注意**：在此練習中，您可以選擇從 **C#** 或 **Python** 使用 REST API。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **01-getting-started** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 檢視 **rest-client** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。
3. 請注意，**rest-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：rest-client.py

    開啟程式碼檔案並檢閱其所包含的程式碼，並注意下列詳細資料：
    - 已匯入各種命名空間來啟用 HTTP 通訊
    - **Main** 函式中的程式碼會擷取認知服務資源的端點和金鑰 - 這些將用於將 REST 要求傳送至文字分析服務。
    - 程式會接受使用者輸入，並使用 **GetLanguage** 函式為認知服務端點呼叫文字分析語言偵測 REST API，以偵測所輸入的文字語言。
    - 傳送至 API 的要求是由包含輸入資料的 JSON 物件所組成 - 在此案例中為 **文件** 物件的集合，而每個物件都有 **識別碼** 和 **文字**。
    - 服務的金鑰包含在要求標頭中，以便驗證用戶端應用程式。
    - 服務的回應是 JSON 物件，用戶端應用程式可以剖析此物件。
4. 以滑鼠右鍵按一下 **rest-client** 資料夾，然後開啟整合式終端機。 然後輸入下列語言特有命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

5. 出現提示時，輸入一些文字並檢閱服務偵測到的語言，這已在 JSON 回應中傳回。 例如，嘗試輸入 "Hello"、"Bonjour" 和 "Gracias"。
6. 當您完成應用程式的測試後，輸入 "quit" 可停止程式。

## <a name="use-an-sdk"></a>使用 SDK

您可以直接撰寫取用認知服務 REST API 的程式碼，但有許多熱門程式開發語言的軟體開發工具組 (SDK)，包括 Microsoft C#、Python 和 Node.js。 使用 SDK 可以大幅簡化取用認知服務的應用程式開發。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，於 **01-getting-started** 資料夾中，根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **sdk-client** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝文字分析 SDK 套件：

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    ```

3. 檢視 **sdk-client** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。
    
4. 請注意，**sdk-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：sdk-client.py

    開啟程式碼檔案並檢閱其所包含的程式碼，並注意下列詳細資料：
    - 已匯入您安裝的 SDK 命名空間
    - **Main** 函式中的程式碼會擷取認知服務資源的端點和金鑰 - 這些將搭配 SDK 使用以建立文字分析服務的用戶端。
    - **GetLanguage** 函式會使用 SDK 來建立服務的用戶端，然後使用此用戶端來偵測所輸入文字的語言。
5. 傳回 **sdk-client** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. 出現提示時，輸入一些文字並檢閱服務偵測到的語言。 例如，嘗試輸入 "Goodbye"、"Au revoir" 和 "Hasta la vista"。
7. 當您完成應用程式的測試後，輸入 "quit" 可停止程式。

> **注意**：在這個簡單的主控台應用程式中，可能無法辨識需要 Unicode 字元集的某些語言。

## <a name="more-information"></a>詳細資訊

如需 Azure 認知服務的詳細資訊，請參閱[認知服務文件](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services)。
