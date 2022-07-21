---
lab:
  title: 管理認知服務安全性
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 9c8de44265ffa0846b6860fd7d416bb3be547ed9
ms.sourcegitcommit: 5ffc20f6a590fe643c2b695b8dc04589411be36e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/31/2022
ms.locfileid: "145951179"
---
# <a name="manage-cognitive-services-security"></a>管理認知服務安全性

安全性是任何應用程式的重要考量事項，而身為開發人員，您應該確保資源的存取 (例如認知服務) 僅限制於需要的人員。

認知服務的存取通常是透過驗證金鑰來控制，這會在您一開始建立認知服務資源時產生。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室中使用的環境，請在 Visual Studio Code 中予以開啟；否則，依照下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git: Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (不限資料夾)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

如果您的訂用帳戶中還沒有 **認知服務** 資源，則必須加以佈建。

1. 開啟位於 `https://portal.azure.com` 的 Azure 入口網站，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋 *認知服務*，並使用下列設定建立 **認知服務** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署的詳細資料。

## <a name="manage-authentication-keys"></a>管理驗證金鑰

當您建立認知服務資源時，會產生兩個驗證金鑰。 您可以在 Azure 入口網站中，或使用 Azure 命令列介面 (CLI) 來管理這些金鑰。

1. 在 Azure 入口網站中，移至您的認知服務資源，並檢視其 [金鑰和端點] 頁面。 此頁面包含您需要連線到資源的資訊，並從您開發的應用程式加以使用。 具體來說：
    - 用戶端應用程式可以傳送要求的 HTTP *端點*。
    - 兩個可用於驗證的 *金鑰* (用戶端應用程式可以使用任一金鑰。 常見的做法是將一個金鑰用於開發，而將另一個金鑰用於實際執行環境。 開發人員完成其工作後，您可以輕鬆重新產生開發金鑰，以防止繼續存取)。
    - 裝載資源的 *位置*。 這對於某些 API (但非所有) 的要求而言是必要的。
2. 在 Visual Studio Code 中，以滑鼠右鍵按一下 **02-cognitive-security** 資料夾，然後開啟整合式終端機。 然後輸入下列命令，以使用 Azure CLI 登入您的 Azure 訂用帳戶。

    ```
    az login
    ```

    將會開啟網頁瀏覽器索引標籤，並提示您登入 Azure。 如此操作後，接著關閉瀏覽器索引標籤並返回 Visual Studio Code。

    > **秘訣**：如果您有多個訂用帳戶，您必須確定您在包含認知服務資源的訂用帳戶中工作。  使用此命令來判斷您目前的訂用帳戶 - 其唯一識別碼是 JSON 中傳回的 **識別碼** 值。

    > **警告**：如果您收到 `az login` 的憑證驗證失敗，請嘗試等候幾分鐘後再試一次。
    >
    > ```
    > az account show
    > ```
    >
    > 如果您必須變更訂用帳戶，請執行此命令，將 *&lt;Your_Subscription_Id&gt;* 變更為正確的訂用帳戶識別碼。
    >
    > ```
    > az account set --subscription <Your_Subscription_Id>
    > ```
    >
    > 或者，您可以在每個 Azure CLI 命令的後方，明確將訂用帳戶識別碼指定為 *--subscription* 參數。

3. 現在，您可使用下列命令來取得認知服務金鑰清單、將 *&lt;resourceName&gt;* 取代為您的認知服務資源名稱，並將 *&lt;resourceGroup&gt;* 取代為您在其中建立的資源群組名稱。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

此命令會傳回認知服務資源的金鑰清單 - 有兩個金鑰，名為 **key1** 和 **key2**。

4. 若要測試認知服務，您可以使用 **curl** - HTTP 要求的命令列工具。 在 **02-cognitive-security** 資料夾中，開啟 **rest-test.cmd** 並編輯其中包含的 **curl** 命令 (如下所示)，將 *&lt;yourEndpoint&gt;* 和 *&lt;yourKey&gt;* 取代為您的端點 URI 和 **Key1** 金鑰，以在認知服務資源中使用文字分析 API。

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':[{'id':1,'text':'hello'}]}"
    ```

5. 儲存您的變更，然後在 **02-cognitive-security** 資料夾的整合式終端機中執行下列命令：

    ```
    rest-test
    ```

此命令會傳回 JSON 文件，其中包含在輸入資料中偵測到的語言相關資訊 (應為英文)。

6. 如果金鑰遭到入侵，或擁有金鑰的開發人員不再需要存取權，則您可以在入口網站或使用 Azure CLI 重新產生金鑰。 執行下列命令來重新產生 **key1** 金鑰 (針對資源，取代 *&lt;resourceName&gt;* 和 *&lt;resourceGroup&gt;* )。

    ```
    az cognitiveservices account keys regenerate --name <resourceName> --resource-group <resourceGroup> --key-name key1
    ```

將傳回認知服務資源的金鑰清單 - 請注意，**key1** 自上次擷取後已變更。

7. 使用舊金鑰重新執行 **rest-test** 命令 (您可以使用 **^** 金鑰來循環先前的命令)，並驗證其現在是否失敗。
8. 在 *rest-test.cmd* 中編輯 **curl** 命令，將金鑰取代為新的 **key1** 值，並儲存變更。 然後重新執行 **rest-test** 命令，並驗證其是否成功。

> **秘訣**：在此練習中，您已使用 Azure CLI 參數的完整名稱，例如 **--resource-group**。  您也可以使用較短的替代名稱 (例如 **-g**)，讓您的命令變得較不冗長 (但較難理解)。  [認知服務 CLI 命令參考](https://docs.microsoft.com/cli/azure/cognitiveservices?view=azure-cli-latest)會列出每個認知服務 CLI 命令的參數選項。

## <a name="secure-key-access-with-azure-key-vault"></a>使用 Azure KeyVault 保護金鑰存取

您可以使用金鑰進行驗證，來開發取用認知服務的應用程式。 不過，這表示應用程式程式碼必須能夠取得金鑰。 其中一個選項是將金鑰儲存在環境變數或部署應用程式的組態檔中，但此方法會使金鑰容易遭受未經授權的存取。 在 Azure 上開發應用程式時，更好的方法是將金鑰安全儲存在 Azure Key Vault 中，並透過 *受控識別* 提供金鑰的存取權 (亦即應用程式本身所使用的使用者帳戶)。

### <a name="create-a-key-vault-and-add-a-secret"></a>建立金鑰保存庫並新增秘密

您必須先為認知服務金鑰，建立金鑰保存庫並新增 *秘密*。

1. 記下認知服務資源的 **key1** 值 (或將其複製到剪貼簿)。
2. 在 Azure 入口網站的 [首頁] 頁面上，選取 [&#65291;建立資源] 按鈕、搜尋 *金鑰保存庫*，並使用下列設定建立 **金鑰保存庫** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*與您認知服務資源相同的資源群組*
    - **金鑰保存庫名稱**：*輸入唯一名稱*
    - **區域**：*與您認知服務資源相同的區域*
    - **定價層**：標準
3. 等候部署完成，然後移至您的金鑰保存庫資源。
4. 在左側瀏覽窗格中，選取 [設定] 區段中的 [秘密]。
5. 選取 [+ 產生/匯入]，並使用下列設定新增秘密：
    - **上傳選項**：手動
    - **名稱**：Cognitive-Services-Key *(請務必完全符合此名稱，因為稍後您將根據此名稱來執行程式碼擷取秘密)*
    - **值**：*您的 **key1** 認知服務金鑰*

### <a name="create-a-service-principal"></a>建立服務主體

若要存取金鑰保存庫中的秘密，您的應用程式必須使用具有秘密存取權的服務主體。 您將使用 Azure 命令列介面 (CLI) 來建立服務主體、尋找其物件識別碼，以及授與 Azure Valut 中祕密的存取權。

1. 返回至 Visual Studio Code，接著在 **02-cognitive-security** 資料夾的整合式終端機中執行下列 Azure CLI 命令，將 *&lt;spName&gt;* 取代為適當的應用程式識別名稱 (例如 *ai-app*)。 也請將 *&lt;subscriptionId&gt;* 和 *&lt;resourceGroup&gt;* 取代為您訂用帳戶識別碼的正確值，以及包含認知服務和金鑰保存庫資源的資源群組：

    > **秘訣**：如果您不確定訂用帳戶識別碼，請使用 **az account show** 命令來擷取您的訂用帳戶資訊 - 訂用帳戶識別碼是輸出中的 **id** 屬性。

    ```
    az ad sp create-for-rbac -n "api://<spName>" --role owner --scopes subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    ```

此命令的輸出包含新服務主體的相關資訊。 其內容應該會如下所示：

    ```
    {
        "appId": "abcd12345efghi67890jklmn",
        "displayName": "ai-app",
        "name": "http://ai-app",
        "password": "1a2b3c4d5e6f7g8h9i0j",
        "tenant": "1234abcd5678fghi90jklm"
    }
    ```

記下 **appId**、**password** 和 **tenant** 值 - 稍後將需要這些值 (如果您關閉此終端機，則將無法擷取密碼；因此，請務必立即記下這些值 - 您可以將輸出貼至 Visual Studio Code 中的新文字檔，以確保稍後您可以找到所需的值！)

2. 若要取得服務主體的 **物件識別碼**，請執行下列 Azure CLI 命令，以將 *&lt;appId&gt;* 取代為服務主體應用程式識別碼的值。

    ```
    az ad sp show --id <appId> --query objectId --out tsv
    ```

3. 若要為新的服務主體指派權限以存取金鑰保存庫中的秘密，請執行下列 Azure CLI 命令，將 *&lt;keyVaultName&gt;* 取代為您的 Azure Key Vault 資源名稱，並將 *&lt;objectId&gt;* 取代為您服務主體物件識別碼的值。

    ```
    az keyvault set-policy -n <keyVaultName> --object-id <objectId> --secret-permissions get list
    ```

### <a name="use-the-service-principal-in-an-application"></a>在應用程式中使用服務主體

現在您已準備好在應用程式中使用服務主體識別，因此其可以存取金鑰保存庫中的秘密認知服務金鑰，並用來連線至您的認知服務資源。

> **注意**：在此練習中，我們會將服務主體認證儲存在應用程式組態中，並使用來驗證應用程式程式碼中的 **ClientSecretCredential** 識別。 這適用於開發和測試，但在實際執行環境的應用程式中，系統管理員會將 *受控識別* 指派給應用程式，讓其使用服務主體識別來存取資源，而無須快取或儲存密碼。

1. 在 Visual Studio Code 中，展開 **02-cognitive-security** 資料夾，並取決於您的語言偏好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **keyvault-client** 資料夾，開啟整合式終端機。 然後，針對您的語言偏好設定，執行適當的命令來安裝您必須在認知服務資源中使用 Azure Key Vault 和文字分析 API 的套件：

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    dotnet add package Azure.Identity --version 1.5.0
    dotnet add package Azure.Security.KeyVault.Secrets --version 4.2.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    pip install azure-identity==1.5.0
    pip install azure-keyvault-secrets==4.2.0
    ```

3. 檢視 **keyvault-client** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其所包含的組態值，以反映下列設定：
    
    - 認知服務資源的 **endpoint**
    - **Azure Key Vault** 資源的名稱
    - 服務主體的 **tenant**
    - 服務主體的 **appId**
    - 服務主體的 **password**

     儲存您的變更。
4. 請注意，**keyvault-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：keyvault-client.py

    開啟程式碼檔案並檢閱其所包含的程式碼，並注意下列詳細資料：
    - 已匯入您安裝的 SDK 命名空間
    - **Main** 函式中的程式碼會擷取應用程式組態設定，然後使用服務主體認證來取得金鑰保存庫中的認知服務金鑰。
    - **GetLanguage** 函式會使用 SDK 來建立服務的用戶端，然後使用此用戶端來偵測所輸入文字的語言。
5. 傳回 **keyvault-client** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python keyvault-client.py
    ```

6. 出現提示時，輸入一些文字並檢閱服務偵測到的語言。 例如，嘗試輸入 "Hello"、"Bonjour" 和 "Gracias"。
7. 當您完成應用程式的測試後，輸入 "quit" 可停止程式。

## <a name="more-information"></a>詳細資訊

如需保護認知服務的詳細資訊，請參閱[認知服務安全性文件](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security)。
