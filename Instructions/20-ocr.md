---
lab:
  title: 讀取圖片中的文字
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 1199e4e4f44a98fc5f900fa1ad021384b56f0c2b
ms.sourcegitcommit: e242452e8125a2622093980048f1e2cacb8b893d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/25/2022
ms.locfileid: "145757487"
---
# <a name="read-text-in-images"></a>讀取圖片中的文字

光學字元辨識 (OCR) 是電腦視覺的子集，處理在圖片及文件中讀取文字。 **電腦視覺** 服務提供兩個 API 來讀取文字，您將在此練習中探索該 API。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未完成此步驟，您必須複製本課程的程式碼存放庫：

1. 啟動 Visual Studio Code。
2. 開啟調色盤 (SHIFT+CTRL+P) 並執行 **Git：複製** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟該資料夾。
4. 等候額外檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

如果您的訂用帳戶中還沒有 **認知服務** 資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，並使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋 [認知服務] ，並使用下列設定建立 [認知服務] 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊，並建立資源。
4. 等候部署完成，然後檢視部署詳細資料。
5. 部署資源後，請移至該資源並檢視其 [金鑰與端點] 頁面。 在下一個程序中，您將需要此頁面中的端點和其中一個金鑰。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>準備使用電腦視覺 SDK

在此練習中，您將完成已部分實作的用戶端應用程式，其使用電腦視覺 SDK 來讀取文字。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [Explorer] 窗格中，瀏覽至 [20-ocr] 資料夾，然後根據您的語言偏好設定展開 [C-Sharp] 或 [Python] 資料夾。
2. 以滑鼠右鍵按一下 [read-text] 資料夾，然後開啟整合式終端。 然後針對您的語言偏好設定執行適當的命令，以安裝電腦視覺 SDK 套件：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. 檢視 **read-text** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。
4. 請注意，**read-text** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：read-text.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用電腦視覺 SDK 所需的命名空間：

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. 在用戶端應用程式的程式碼檔案中，並在 **Main** 函數中，請注意已提供可載入組態設定的程式碼。 然後尋找 **Authenticate Computer Vision client** 註解。 然後，在此註解之下，新增下列語言特有程式碼，來建立及驗證電腦視覺用戶端物件：

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
    
## <a name="use-the-ocr-api"></a>使用 OCR API

**OCR** API 是一種光學字元辨識 API，已針對在 *.jpg*、 *.png*、 *.gif* 及 *.bmp* 格式圖片中讀取少量至中量列印文字進行最佳化。 其支援各種不同的語言，除了在圖片中讀取文字之外，還可以決定每個文字區域的方向，並傳回與圖片相關的文字旋轉角度相關資訊

1. 在應用程式的程式碼檔案中的 **Main** 函數中，檢查使用者選取功能表選項 **1** 時所執行的程式碼。 此程式碼會呼叫 **GetTextOcr** 函數，並將路徑傳遞至圖片檔。
2. 在 [讀取文字/圖片] 資料夾中，開啟 [Lincoln.jpg]，以檢視程式碼將處理的圖片。
3. 在程式碼檔案中尋找 [GetTextOcr] 函數，並在將訊息列印至主控台的現有程式碼之下，新增下列程式碼：

**C#**

```C#
// Use OCR API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // Show the position of the line of text
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // Read the words in the line of text
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // Save the image with the text locations highlighted
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# Use OCR API to read text in image
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# Prepare image for drawing
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# Process the text line by line
for region in ocr_results.regions:
    for line in region.lines:

        # Show the position of the line of text
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # Read the words in the line of text
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# Save the image with the text locations highlighted
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. 檢查您新增至 **GetTextOcr** 函數的程式碼。 其會從圖片檔偵測列印文字的區域，並針對每個區域擷取文字行，並醒目提示圖片上文字行的位置。 然後其會擷取每一行中的字組，並加以列印。
5. 儲存您的變更並返回 **read-text** 資料夾的整合式終端，然後輸入下列命令來執行程式：

**C#**

```
dotnet run
```

*C# 輸出可能會顯示警告，有關現在使用 **await** 運算子之非同步函式。您可以忽略這些警告。*

**Python**

```
python read-text.py
```

6. 出現提示時，輸入 **1** 並觀察輸出，其為從圖片中擷取的文字。
7. 檢視與程式碼檔案相同資料夾中產生的 **ocr_results.jpg** 檔案，以查看圖片中標註的文字行。

## <a name="use-the-read-api"></a>使用 Read API

**讀取** API 會使用比 OCR API 更新的文字辨識模型，並對於包含大量文字的較大圖片表現得更好。 其也支援從 *.pdf* 檔案擷取文字，而且可以辨識多種語言的印刷文字及手寫文字。

**讀取** API 會使用非同步作業模型，其中會提交啟動文字辨識的要求；和從要求傳回的作業識別碼，其後續可用來檢查進度和擷取結果。

1. 在應用程式的程式碼檔案中的 **Main** 函數中，檢查使用者選取功能表選項 **2** 時所執行的程式碼。 此程式碼會呼叫 **GetTextRead** 函數，並將路徑傳遞至 PDF 文件檔。
2. 在 [讀取文字/圖片] 資料夾中，以滑鼠右鍵按一下 [Rome.pdf]，然後選取 [在檔案總管中顯示]。 然後在檔案總管中，開啟 PDF 檔案以檢視檔案。
3. 在 Visual Studio Code 中的程式碼檔案中尋找 [GetTextRead] 函數，並在將訊息列印至主控台的現有程式碼之下，新增下列程式碼：

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfuly, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfuly, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. 檢查您新增至 **GetTextRead** 函數的程式碼。 其會提交讀取作業的要求，然後重複檢查狀態，直到作業完成為止。 如果成功，程式碼會逐一查看每個頁面，然後逐行來處理結果。
5. 儲存您的變更並返回 **read-text** 資料夾的整合式終端，然後輸入下列命令來執行程式：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. 出現提示時，請輸入 **2** 並觀察輸出，其為從文件中擷取的文字。

## <a name="read-handwritten-text"></a>讀取手寫文字

除了列印文字之外，**讀取** API 還可以擷取英文的手寫文字。

1. 在應用程式的程式碼檔案中，於 **Main** 函式中，檢查使用者選取功能表選項 **3** 時所執行的程式碼。 此程式碼會呼叫 **GetTextRead** 函數，並將路徑傳遞至圖片檔。
2. 在 [讀取文字/圖片] 資料夾中，開啟 [Note.jpg]，以檢視程式碼將處理的圖片。
3. 在 **read-text** 資料夾的整合式終端，然後輸入下列命令來執行程式：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. 出現提示時，請輸入 **3** 並觀察輸出，其為從文件中擷取的文字。

## <a name="more-information"></a>詳細資訊

如需使用 **電腦視覺** 服務來讀取文字的詳細資訊，請參閱[電腦視覺文件](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text)。
