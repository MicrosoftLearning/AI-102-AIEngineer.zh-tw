---
lab:
  title: 建置問題解答解決方案
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 42b3b6e6955188d3d442a4f587ead6afe0f389c6
ms.sourcegitcommit: a879eae298aaf2ab6415fc4fdb67f52f592d907a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2022
ms.locfileid: "146902478"
---
# <a name="create-a-question-answering-solution"></a>建置問題解答解決方案

最常見的交談案例之一，是透過常見問題 (FAQ) 知識庫提供支援。 許多組織都會將常見問題發佈為文件或網頁，這對一組小型問答很適合，但搜尋大型文件可能困難且耗時。

**語言** 服務包含一項 *問題解答* 功能，讓您能建立可使用自然語言輸入查詢的問題和答案配對知識庫，而且 Bot 最常使用該服務來查閱使用者所提交問題的答案資源。

> **注意**：語言服務中的問題回答功能是較新版本的 QnA Maker 服務，該服務仍可做為個別服務使用。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未將 **AI-102-AIEngineer** 程式碼存放庫複製至您目前使用實驗室的環境，請遵循下列步驟來執行這項操作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git:Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (不限資料夾)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來進行組建和偵錯，請選取 [現在不要]。

## <a name="create-a-language-resource"></a>建立語言資源

若要建立並裝載知識庫以回答問題，您需要 Azure 訂用帳戶中的 **語言服務** 資源。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 選取 [+ 建立資源] 按鈕，搜尋 [語言服務]，然後建立 **語言服務** 資源。
3. 按一下 [自訂問題解答] 區塊上的 [選取]。 然後按一下 [繼續建立您的資源]。 您必須輸入下列設定：
    
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*選擇任何可用的位置*
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S
    - **Azure 搜尋位置**\*：*在全球區域中，選擇和您的語言資源相同的位置*。
    - **Azure 搜尋定價層**：免費 (F) (*若此階層不可用，則選取 [基本 (B)]* )
    - **法律條款**：_同意_ 
    - **負責任 AI 注意事項**：_同意_
    
    \*自訂問題解答會使用 Azure 搜尋服務來編制問題與解答知識庫的索引和查詢。

4. 等候部署完成，然後檢視部署的詳細資料。

## <a name="create-a-question-answering-project"></a>建立問題解答專案

若要在 Language 資源中建立問題回答的知識庫，您可以使用 Language Studio 入口網站來建立問題解答專案。 在此情況下，您將建立知識庫，其中包含 [Microsoft Learn](https://docs.microsoft.com/learn) 的問題和解答。

1. 在新的瀏覽器索引標籤中，開啟位於 `https://language.azure.com` 的 *Language Studio* 入口網站，然後使用與您 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 若系統提示您選擇語言資源，請選取下列設定：
    - **Azure 目錄**：包含您訂用帳戶的 Azure 目錄。
    - **Azure 訂用帳戶**：您的 Azure 訂用帳戶。
    - **語言資源**：您先前建立的語言資源。
3. 如果您<u>未</u>收到選擇語言資源的提示，可能是因為您的訂用帳戶中有多個語言資源；在此情況下：
    1. 在分頁頂端的列上，按一下 [設定 (&#9881;)] 按鈕。
    2. 在 [設定] 分頁上，檢視 [資源] 索引標籤。
    3. 選取您剛才建立的語言資源，然後按一下 [切換資源]。
    4. 在頁面頂端，按一下 [Language Studio] 以回到 Language Studio 首頁。
3. 在入口網站頂端的 [新建] 功能表中，選取 [自訂問題解答]。
4. 在 *[建立專案] 精靈的 [選擇語言設定] 頁面上，選取選項來設定此資源中所有專案的語言，然後選取 [英文] 作為語言。 然後按一下 [下一步]。
5. 在 [輸入基本資訊] 頁面上，輸入下列詳細資料，然後按 [下一步]：
    - **名字** LearnFAQ
    - **描述**：Microsoft Learn 的 FAQ
    - **未傳回任何答案時的預設答案**：抱歉，我不能理解此問題
6. 在 [檢閱和完成] 頁面上，按一下 [建立專案]。

## <a name="add-a-sources-to-the-knowledge-base"></a>將來源新增至知識庫

您可以從頭開始建立知識庫，但從現有的常見問題頁面或文件匯入問題和解答，也是常見的作法。 在此情況下，您將從 Microsoft Learn 的現有常見問題網頁匯入資料，而且您也會匯入一些預先定義的「閒聊」問題和解答，以支援常見的對話交談。

1. 在問題解答專案的 [管理來源] 頁面上，於 [&#9547; L33新增來源] 清單中，選取 [URL]。 然後在 [新增 URL] 對話方塊中，按一下 [&#9547; 新增 url] 並新增下列 URL，然後按一下 [全部新增] 將其新增至知識庫：
    - **名稱**：`Learn FAQ Page`
    - **URL**：`https://docs.microsoft.com/en-us/learn/support/faq`
2. 在問題解答專案的 [管理來源] 頁面上，於 [&#9547; L33新增來源] 清單中，選取 [Chitchat]。 在 [新增閒聊] 對話方塊中，選取 [易記]，然後按一下 [新增閒聊]。

## <a name="edit-the-knowledge-base"></a>編輯知識庫

您的知識庫已填入來自 Microsoft Learn 常見問題的問答組，並補充一組交談 *閒聊* 問題和答案配對。 您可以藉由新增其他問答配對來擴充知識庫。

1. 在 Language Studio 的 **LearnFAQ** 專案中，選取 [編輯知識庫] 頁面，以查看現有的問題與答案配對 (如果顯示秘訣，請閱讀秘訣並按一下 [了解] 以關閉，或按一下 [略過所有])
2. 在知識庫中，選取 [&#65291;新增問題配對]。
3. 在 [問題] 方塊中輸入 `What is Microsoft certification?`，然後按 **Enter****。
4. 選取 [&#65291; 新增替代片語] 並輸入 `How can I demonstrate my Microsoft technology skills?`，然後按 **Enter**。
5. 在 [答案] 方塊中，輸入 `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.`然後按下 [提交] 以新增問題 (包括替代片語) 和答案到知識庫。

    在某些情況下，有需要讓使用者能夠追蹤答案，以建立 *多回合* 交談，以便使用者能反復精簡問題以取得所需的答案。

6. 在您為認證問題輸入的答案下，選取 [&#65291;新增後續提示]。
7. 在 [後續提示] 對話方塊中，輸入下列設定，然後按一下 [新增提示]：
    - **提示中向使用者顯示的文字**：`Learn more about certification`。
    - 選取 [建立新配對的連結]，然後輸入下列文字：`You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - **僅顯示內容流程**：已選取。 *此選項可確保答案只會在原始認證問題的後續問題內容中傳回。*

## <a name="train-and-test-the-knowledge-base"></a>定型和測試知識庫

既然您擁有知識庫，就可以在 Language Studio 中測試知識庫。

1. 按一下頁面右上方的 [儲存變更]。
2. 儲存變更之後，按一下 [測試] 以開啟測試窗格。
3. 在測試窗格中的頂端，*取消選取* 顯示簡短答案的選項。 然後在底部輸入訊息 `Hello`。 應該傳回適當的回應。
4. 在測試窗格底部輸入訊息 `What is Microsoft Learn?`。 應該從常見問題集中退回適當的回應。
5. 輸入訊息 `Thanks!` 應會傳回適當的閒聊回應。
6. 輸入訊息 `Tell me about certification`。 您建立的答案應該會連同後續提示連結一起傳回。
7. 選取 [深入瞭解認證] 後續連結。 應該會傳回具有認證頁面連結的後續答案。
8. 完成知識庫測試之後，關閉測試窗格。

## <a name="deploy-and-test-the-knowledge-base"></a>部署和測試知識庫

知識庫提供了後端服務，用戶端應用程式可以使用該後端服務來回答問題。 現在您已準備好發佈知識庫，並從用戶端存取其 REST 介面。

1. 在 Language Studio 的 **LearnFAQ** 專案中，選取 [部署知識庫] 頁面。
2. 按一下頁面頂端的 [部署]。 然後按一下 [部署] 以確認您想要部署知識庫。
3. 部署完成時，按一下 [取得預測 URL] 以檢視您知識庫的 REST 端點，並將其複製到剪貼簿 (但還不要關閉對話方塊)。
4. 在 Visual Studio Code 的 **12-qna** 資料夾中，開啟 **ask-question.cmd**。 此指令碼會使用 *Curl* 來呼叫問題解答端點的 REST 介面。
5. 在指令碼中，將 *YOUR_PREDICTION_ENDPOINT* 取代為您複製的預測端點 (確保其以引號括住)。
6. 返回瀏覽器，然後在 [取得預測 URL] 對話方塊中，記下範例要求包含 **Ocp-Apim-Subscription-Key** 參數的值，看起來會類似 *ab12c345de678fg9hijk01lmno2pqrs34*。 這是您資源的授權金鑰。 將其複製到剪貼簿，然後按一下 [了解] 以關閉對話方塊。
7. 返回 Visual Studio Code，然後在 **ask-question.cmd** 指令碼中，將 *YOUR_KEY* 取代為您複製的金鑰 (確保其以引號括住)。
8. 請注意，指令碼中的 Curl 命令會提交其值為 **學習路徑是什麼？** 的 **問題** 參數。
9. 確認整個指令碼看起來類似下列程式碼，然後儲存檔案。

    ```
    @echo off
    SETLOCAL ENABLEDELAYEDEXPANSION

    rem Set variables
    set prediction_url="https://some-name.cognitiveservices.azure.com/language/......."
    set key="ab12c345de678fg9hijk01lmno2pqrs34"

    curl -X POST !prediction_url! -H "Ocp-Apim-Subscription-Key: !key!" -H "Content-Type: application/json" -d "{'question': 'What is a learning Path?' }"
    ```

10. 在 Explorer 窗格中的 [Visual Studio Code]，以滑鼠右鍵按一下 **ask-question.cmd** 指令碼，然後選取 [在整合式終端中開啟]。
11. 在終端機窗格中，輸入命令 `ask-question.cmd` 以執行指令碼，並檢視服務所傳回的 JSON 回應，這應該包含問題 *學習路徑是什麼？* 的適當答案。

## <a name="create-a-bot-for-the-knowledge-base"></a>為知識庫建立 Bot

最常見的是，Bot 是用來從知識庫擷取答案的用戶端應用程式。

1. 返回瀏覽器中的 Language Studio，然後在 [部署知識庫] 頁面中，選取 [建立 Bot]。 這將在新的瀏覽器索引標籤中打開 Azure 入口網站，以便您可以在 Azure 訂閱中建立 Web 應用程式 Bot (若收到提示，請登入)。
2. 在 Azure 入口網站中，透過以下設定建立 Web 應用程式 Bot (大多數設定將會為您預先填入)：

    *如果遺漏某些值，請重新整理瀏覽器。*  

  - **Bot 控制代碼**：您的 Bot 之唯一名稱
  - **訂用帳戶**：Azure 訂閱
  - **資源群組**：*包含您語言資源的資源群組*
  - **位置**：*和您文字分析服務相同的位置*。
  - **定價層**：F0
  - **應用程式名稱**：*與自動附加的唯一識別碼和 **.azurewebsites.net** *Bot 控制碼* 相同
  - **SDK 語言**：*選擇 C# 或 Node.js*
  - **QnA 授權金鑰**:該金鑰應該自動設定為您的 QnA 知識庫之驗證金鑰
  - **App Service 方案/位置**：*若有適當的方案和位置存在，可自動設定為該方案/位置。若沒有，則建立新的方案*
  - **Application Insights**：關閉
  - **Microsoft App ID 和密碼**：自動建立 App ID 和密碼。
3. 等待系統建立 Bot。 然後按一下 [轉到資源] (或者在首頁上，按一下 [資源群組]，打開已經在其中建立了 Web 應用程式 Bot 的資源群組，然後按一下這個資源群組。)
4. 在 Bot 的刀鋒視窗中，檢視 [網路聊天中的測試] 頁面，並等到 Bot 顯示 **Hello and welcome!** 訊息！ (這可能需要幾秒鐘來初始化)。
5. 使用測試聊天介面，確保您的 Bot 按照預想回答知識庫中的問題。 例如，嘗試提交 `What is Microsoft certification?`。

## <a name="more-information"></a>詳細資訊

若要深入了解語言服務中的問題解答，請參閱[語言服務文件](https://docs.microsoft.com/en-us/azure/cognitive-services/language-service/question-answering/overview)。
