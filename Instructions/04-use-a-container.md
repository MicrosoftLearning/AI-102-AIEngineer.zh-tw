---
lab:
  title: 使用認知服務容器
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 244ab1ef3754e668d64996dece9711682651691d
ms.sourcegitcommit: 29a684646784fe4f7370343b6c005728a953770d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2022
ms.locfileid: "145195473"
---
# <a name="use-a-cognitive-services-container"></a>使用認知服務容器

使用 Azure 中裝載的認知服務可讓應用程式開發人員專注於自己的程式碼基礎結構，同時也受益於由 Microsoft 管理的可調整服務。 然而，在眾多情況下，組織需要更充分掌控其服務基礎結構，以及服務之間傳遞的資料。

許多認知服務 API 都可以進行封裝，並部署在 *容器* 中，讓組織能夠在自己的基礎結構中裝載認知服務；例如，在本地 Docker 伺服器、Azure 容器執行個體或 Azure Kubernetes Services 叢集中。 容器化認知服務需要與 Azure 型認知服務帳戶進行通訊，以支援計費；但應用程式資料不會傳遞至後端服務，而且組織對容器的部署組態擁有有更大的掌控權，可啟用用於驗證、可擴縮性及其他考量的自訂解決方案。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您已經將 **AI-102-AIEngineer** 程式碼存放庫複製到您正在此實驗室使用的環境，請在 Visual Studio Code 中予以開啟；否則，請遵循下列步驟立即進行複製。

1. 啟動 Visual Studio Code。
2. 開啟調色盤 (SHIFT+CTRL+P) 並執行 **Git：複製**  命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

如果您的訂用帳戶中還沒有 **認知服務** 資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋 [認知服務] ，然後使用以下設定建立 [認知服務] 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署詳細資料。
5. 部署資源後，移至該資源並檢視其 [金鑰與端點] 頁面。 在下一個程序中，您將會需要此頁面中的端點和其中一個金鑰。

## <a name="deploy-and-run-a-text-analytics-container"></a>部署及執行文字分析容器

許多常用的認知服務 API 在容器映像中是可用的。 如需完整清單，請參閱[認知服務文件](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#container-availability-in-azure-cognitive-services)。 在此練習中，您將使用適用於文字分析 *語言偵測* API 的容器映射；但所有可用映射的準則都相同。

1. 在 Azure 入口網站的 [首頁] 頁面上，選取 [&#65291;建立資源] 按鈕、搜尋 [容器執行個體] ，並使用下列設定建立 [容器執行個體] 資源：

    - **基本**：
        - **訂用帳戶**：Azure 訂閱
        - **資源群組**：*選擇包含您認知服務資源的資源群組*
        - **容器名稱**：*輸入唯一名稱*
        - **區域**：選擇任一可用區域
        - **映像來源**：其他登錄
        - **映像類型**：公開
        - **映像**：`mcr.microsoft.com/azure-cognitive-services/textanalytics/language:1.1.013570001-amd64`
        - **OS 類型**：Linux
        - **尺寸**：1 vcpu，4 GB 記憶體
    - **網路**：
        - **網路類型**：公開
        - **DNS 名稱標籤**：*為容器端點輸入唯一的名稱*
        - **連接埠**：*將 TCP 連接埠 從 80 變更為 **5000***
    - **進階**：
        - **重新啟動原則**：失敗時
        - **環境變數**：

            | 標示為安全 | Key | 值 |
            | -------------- | --- | ----- |
            | Yes | `ApiKey` | *認知服務資源的任一金鑰* |
            | Yes | `Billing` | *認知服務資源的端點 URI* |
            | 否 | `Eula` | `accept` |

        - **命令覆寫**：[]
    - **標籤**：
        - *請勿新增任何標籤*

2. 等候部署完成，然後移至部署的資源。
3. 在其 [概觀] 頁面上觀察容器執行個體資源的下列屬性：
    - **狀態**：這個應該在 *運行中*。
    - **IP 位址**：此為您可以用於存取您容器執行個體的公開 IP 位址。
    - **FQDN**：這是容器執行個體資源的 *完整功能變數名稱*，您可以使用其來存取容器執行個體，而不是 IP 位址。

    > **注意**：在此練習中，您已將適用於文字平移的認知服務容器映射部署至 Azure 容器執行個體 (ACI) 資源。 您可以使用類似的方法將其部署到您自己的電腦或網路上的 *[Docker](https://www.docker.com/products/docker-desktop)* 主機，方法是在單一行上執行下列命令，將語言偵測容器部署到本機 Docker 執行個體，並將 *&lt;yourEndpoint&gt;* 和 *&lt;yourKey&gt;* 取代為端點 URI，以及認知服務資源的其中一個金鑰。
    >
    > ```
    > docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 mcr.microsoft.com/azure-cognitive-services/textanalytics/language Eula=accept Billing=<yourEndpoint> ApiKey=<yourKey>
    > ```
    >
    > 該命令會在本機電腦上尋找映射，如果其找不到該映射，則會從 *mcr.microsoft.com* 映射登錄提取映射，並將其部署至 Docker 執行個體。 部署完成時，容器會啟動並接聽連接埠 5000 上的傳入要求。

## <a name="use-the-container"></a>使用該容器

1. 在 Visual Studio Code 的 **04-containers** 資料夾中，開啟 **rest-test.cmd** 並編輯其包含的 **curl** 命令 (如下所示)，並以容器的 IP 位址或 FQDN 取代 *&lt;your_ACI_IP_address_or_FQDN&gt;* 。

    ```
    curl -X POST "http://<your_ACI_IP_address_or_FQDN>:5000/text/analytics/v3.0/languages?" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'Hello world.'},{'id':2,'text':'Salut tout le monde.'}]}"
    ```

2. 儲存您對指令碼做出的變更。 請注意，您不需要指定認知服務端點或金鑰 - 要求是由容器化服務進行處理。 該容器接著會定期與 Azure 中的服務通訊，以報告計費使用量，但不會傳送要求資料。
3. 以滑鼠右鍵按一下 **04-containers** 資料夾，然後開啟整合式終端。 然後輸入下列命令以執行指令碼：

    ```
    rest-test
    ```

4. 驗證該命令會傳回 JSON 文件，其中包含在兩個輸入資料中偵測到的語言相關資訊 (應該是英文與法文)。

## <a name="clean-up"></a>清除

如果您已完成容器執行個體的實驗，您應該將其刪除。

1. 在 Azure 入口網站中，開啟您為此練習建立資源的資源群組。
2. 選取容器執行個體資源並加以刪除。

## <a name="more-information"></a>詳細資訊

如需容器化認知服務的詳細資訊，請參閱[認知服務容器文件](https://docs.microsoft.com/azure/cognitive-services/containers/)。
