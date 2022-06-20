---
lab:
  title: 從表單擷取資料
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 540fdc49b9efcf335d43cdd7a6db405c255cd058
ms.sourcegitcommit: de1f38bbe53ec209b42cd89516813773e2f3479b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/17/2022
ms.locfileid: "145195462"
---
# <a name="extract-data-from-forms"></a>從表單擷取資料 

假設一家公司目前要求員工手動購買訂單表，並將資料輸入資料庫中。 公司想要您利用 AI 服務來改善資料輸入流程。 您決定組建機器學習模型，以讀取表單並產生結構化資料，可用來自動更新資料庫。

**Azure 表格辨識器** 是一種認知服務，使用者能夠組建自動化資料處理軟體。 此軟體可以使用光學字元辨識 (OCR)，從表單文件中擷取文字、索引鍵/值組及資料表。 Azure 表格辨識器具有預先建立的模型，可辨識發票、收據及名片。 該服務也提供定型自訂模型的功能。 在此練習中，我們將著重於組建自訂模型。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未完成此步驟，您必須複製本課程的程式碼存放庫：

1. 啟動 Visual Studio Code。
2. 開啟調色盤 (SHIFT+CTRL+P) 並執行 **Git：複製**  命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟該資料夾。
4. 等候其他檔案完成安裝，以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]。

## <a name="create-a-form-recognizer-resource"></a>建立表單辨識器資源

若要使用 Azure 表格辨識器服務，您需要在您的 Azure 訂用帳戶中建立表格辨識器資源或認知服務資源。 您將使用 Azure 入口網站來建立資源。

1.  開啟位於 `https://portal.azure.com` 的 Azure 入口網站，並使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

2. 選取 [&#65291;建立資源] 按鈕、搜尋 [表格辨識器] ，並使用下列設定建立 **表格辨識器** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：F0

    > **注意**：若您的訂用帳戶中已經有 F0 表單辨識器服務，請為這一個選取 [S0] 。

3. 部署資源後，請移至該資源並檢視其 [金鑰和端點]  頁面。 稍後您將會需要此頁面的 **端點** 和其中一個 **金鑰**，來管理您程式碼的存取權。 

## <a name="gather-documents-for-training"></a>收集用於定型的文件

![發票的圖片。](../21-custom-form/sample-forms/Form_1.jpg)  

您將使用此存放庫中 **21-custom-form/sample-forms** 資料夾中的範例表單，其中包含定型和測試模型所需的所有檔案。

1. 在 Visual Studio Code 中的 **21-custom-form** 資料夾中，展開 **sample-forms** 資料夾。 請注意，資料夾中有以 **.json** 和 **.jpg** 結尾的檔案。

    您將使用 **.jpg** 檔案來定型您的模型。  

    **.json** 檔案已為您產生，並包含標籤資訊。 檔案會連同表單一起上傳至您的 Blob 儲存體容器。 

2. 返回位於 [https://portal.azure.com](https://portal.azure.com) 的 Azure 入口網站。

3. 檢視您先前建立 Azure 表格辨識器資源的 **資源群組**。

4. 在資源群組的 [概觀] 頁面上，記下 [訂用帳戶識別碼] 和 [位置]。 後續步驟中，您將需要這些值以及您的 **資源群組** 名稱。

![資源群組頁面的範例。](./images/resource_group_variables.png)

5. 在 [Visual Studio Code] 的 Explorer 窗格中，以滑鼠右鍵按一下 [21-custom-form] 資料夾，然後選取 [在整合式終端中開啟]。

6. 在終端窗格中，輸入下列命令來建立 Azure 訂用帳戶的已驗證連線。
    
```
az login --output none
```

7. 出現系統提示時，登入您的 Azure 訂用帳戶。 然後返回 Visual Studio Code，並等候登入流程完成。

8. 執行下列命令以列出 Azure 位置。

```
az account list-locations -o table
```

9. 在輸出中，尋找對應資源群組位置的 **Name** 值 (例如，對於 *美國東部* 的對應名稱是 *eastus*)。

    > **重要**：記錄 **Name** 值，並在步驟 12 中加以使用。

10. 在 Explorer 窗格的 [21-custom-form] 資料夾中，選取 [setup.cmd]。 您將使用此批次指令碼來執行 Azure 命令列介面 (CLI)，建立所需的其他 Azure 資源所需的命令。

11. 在 **setup.cmd** 指令碼中，檢閱 **rem** 命令。 這些批註概述指令碼將執行的程式。 該程式將會： 
    - 在 Azure 資源群組中建立儲存體帳戶
    - 將檔案從本地 _sampleforms_ 資料夾上傳至儲存體帳戶中名為 _sampleforms_ 的容器
    - 列印共用存取簽章 URI

12. 使用您部署表格辨識器資源之訂用帳戶、資源群組及位置名稱的適當值，修改 **subscription_id**、**resource_group** 以及 **位置** 變數宣告。 然後 **儲存** 您的變更。

    為練習保留 **expiry_date** 變數原狀。 此變數在產生共用存取簽章 (SAS) URI 時使用。 在實務上，您會想要為 SAS 設定適當的到期日期。 您可以在[此處](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works)深入了解 SAS。  

13. 在 **21-custom-form** 資料夾的終端中，輸入下列命令以執行指令碼：

```
setup
```

14. 當指令碼完成時，請檢閱顯示的輸出，並記下您 Azure 資源的 SAS URI。

> **重要**：在繼續之前，請將 SAS URI 貼到您稍後能夠再次擷取的地方 (例如，在 Visual Studio Code 的新文字檔中)。

15. 在 Azure 入口網站中，重新整理資源群組，並確認其中包含稍早才建立的 Azure 儲存體帳戶。 開啟儲存體帳戶，並在左側窗格中，選取[儲存體瀏覽器 (預覽)]。 然後在儲存體瀏覽器中，展開 [BLOB 容器] ，然後選取 [sampleforms] 容器，以確認檔案已從本機 **21-custom-form/sample-forms** 資料夾上傳。

## <a name="train-a-model-using-the-form-recognizer-sdk"></a>使用 Azure 表格辨識器 SDK 來定型模型

現在，您將使用 **.jpg** 和 **.json** 檔案來定型模型。

1. 在 Visual Studio Code 的 **21-custom-form/sample-forms** 資料夾中，開啟 **fields.json** 並檢閱其所包含的 JSON 文件。 此檔案定義您要定型模型以從表單中擷取的欄位。
2. 開啟 **Form_1.jpg.labels.json**，並檢閱其所包含的 JSON。 此檔案識別 **Form_1.jpg** 定型文件中具名欄位的位置和值。
3. 開啟 **Form_1.jpg.ocr.json**，然後檢閱其所包含的 JSON。 此檔案包含 **Form_1.jpg** 文字配置的 JSOn 標記法，包括表單中找到之所有文字區域的位置。

    *在本練習中為您提供了欄位資訊檔案。對於您自己的專案，您可以使用 [Azure 表格辨識器工作室](https://formrecognizer.appliedai.azure.com/studio)來建立這些檔案。當您使用該工具時，您的欄位資訊檔案會自動建立並儲存在您已連線的儲存體帳戶中。*

4. 在 Visual Studio Code 中的 **21-custom-formn** 資料夾中，根據您的語言喜好設定，展開 **C-Sharp** 或 **Python** 資料夾。
5. 以滑鼠右鍵按一下 **train-model** 資料夾，然後開啟整合式終端。

6. 針對您的語言喜好設定執行適當的命令，以安裝 Azure 表格辨識器套件：

**C#**

```
dotnet add package Azure.AI.FormRecognizer --version 3.0.0 
```

**Python**

```
pip install azure-ai-formrecognizer==3.0.0
```

7. 檢視 **train-model** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

8. 編輯組態檔，修改該設定以反映：
    - 您表格辨識器資源的 **端點**。
    - 您表格辨識器資源的 **金鑰**。
    - Blob 容器的 **SAS URI**。

9. 請注意，**train-model** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：train-model.py

    開啟程式碼檔案並檢閱其所包含的程式碼，並注意下列詳細資料：
    - 從您安裝的套件匯入命名空間
    - **Main** 函數會擷取組態設定，並使用金鑰和端點來建立已驗證的 **用戶端**。
    - 該程式碼會使用定型用戶端來定型模型，利用 blob 儲存體容器中的圖片，由您產生的 SAS URI 存取該容器。

10. 在 **train-model** 資料夾中，開啟定型應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：train-model.py

11. 傳回 **train-model** 資料夾的整合式終端，然後輸入下列命令來執行程式：

**C#**

```
dotnet run
```

**Python**

```
python train-model.py
```

12. 請等候程式結束，然後檢閱模型輸出。
13. 記下終端輸出中的模型識別碼。 您將在實驗室的下一個部分中需要使用該模型識別碼。 

## <a name="test-your-custom-form-recognizer-model"></a>測試您的自訂 Azure 表格辨識器模型 

1. 在 **21-custom-form** 資料夾中，於您慣用語言的子資料夾中，(**C-Sharp** 或 **Python**)，展開 [test-model] 資料夾。

2. 以滑鼠右鍵按一下 [test-model] 資料夾，然後選取 [開啟整合式終端]。

3. 在 **test-model** 資料夾的終端中，針對您的語言喜好設定執行適當的命令，以安裝表格辨識器套件：

**C#**

```
dotnet add package Azure.AI.FormRecognizer --version 3.0.0 
```

**Python**

```
pip install azure-ai-formrecognizer==3.0.0
```

*如果您先前已使用 pip 將套件安裝至 Python 環境，這不是絕對必要的；但確保其已安裝並沒有什麼壞處！*

4. 在 **test-model** 資料夾的同一個終端中，安裝 Tabulate 程式庫。 這將在資料表中提供您的輸出：

**C#**

```
Install-Package Tabulate.NET -Version 1.0.5
```

**Python**

```
pip install tabulate
```

5. 在 **test-model** 資料夾中，編輯組態檔 (**appsettings.json** 或 **.env**，具體取決於您的語言喜好）新增下列值：
    - 您的表格辨識器端點。
    - 您的表格辨識器金鑰。
    - 當您將模型定型時產生的模型識別碼 (您可以藉由將終端切換回 **train-model** 資料夾的 **Cmd** 主控台，來找到此識別碼)。 [儲存] 變更。

6. 在 **test-model** 資料夾中，開啟用戶端應用程式的程式碼檔案 (適用於 C# 的 *Program.cs*，適用於 Python 的 *test-model.py*)，並檢閱其所包含的程式碼，並注意下列詳細資料：
    - 從您安裝的套件匯入命名空間
    - **Main** 函數會擷取組態設定，並使用金鑰和端點來建立已驗證的 **用戶端**。
    - 然後，用戶端會用來從 **test1.jpg** 圖片中擷取表單欄位和值。
    

7. 傳回 **test-model** 資料夾的整合式終端，然後輸入下列命令來執行程式：

**C#**

```
dotnet run
```

**Python**

```
python test-model.py
```
    
8. 檢視輸出並觀察模型的輸出如何提供欄位名稱，例如 「CompanyPhoneNumber」和 「DatedAs」。   

## <a name="more-information"></a>詳細資訊

如需關於 Azure 表格辨識器服務的詳細資訊，請參閱[表格辨識器文件](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/)。
