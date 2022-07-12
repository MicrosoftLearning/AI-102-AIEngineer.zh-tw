---
lab:
  title: 使用電腦視覺分析影像
  module: Module 8 - Getting Started with Computer Vision
ms.openlocfilehash: f2ee18ff682d53e9fd554749ed2b9cbaa9b03611
ms.sourcegitcommit: 7191e53bc33cda92e710d957dde4478ee2496660
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2022
ms.locfileid: "147041663"
---
# <a name="analyze-images-with-computer-vision"></a>使用電腦視覺分析影像

電腦視覺是一種人工智慧功能，可讓軟體系統藉由分析影像來解譯視覺輸入。 在 Microsoft Azure 中，「電腦視覺」認知服務會提供常見電腦視覺工作的預先建置模型，包括分析影像以建議標題和標籤、偵測常見物件、地標、名人、品牌，以及是否包含成人內容。 您也可以使用電腦視覺服務來分析影像色彩和格式，並藉此產生「智慧裁剪」縮圖影像。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未將 **AI-102-AIEngineer** 程式碼存放庫複製至您目前使用實驗室的環境，請遵循下列步驟來執行這項操作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git:Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (不限資料夾)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]。

## <a name="provision-a-cognitive-services-resource"></a>佈建認知服務資源

如果您的訂用帳戶中還沒有 **認知服務** 資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 按一下 [&#65291;建立資源] 按鈕，搜尋 *認知服務*，並使用下列設定建立 **認知服務** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署詳細資料。
5. 部署資源後，移至該資源並檢視其 [金鑰和端點] 頁面。 在下一個程序中，您需要此頁面中的端點和其中一個金鑰。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>準備使用電腦視覺 SDK

在此練習中，您將完成部分實作的用戶端應用程式，其會使用電腦視覺 SDK 來分析影像。

> **注意**：您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **15-computer-vision** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **image-analysis** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝電腦視覺 SDK 套件：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. 檢視 **image-analysis** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其所包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。
4. 請注意，**image-analysis** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：image-analysis.py

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
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```
    
## <a name="view-the-images-you-will-analyze"></a>檢視您將分析的影像

在此練習中，您將使用電腦視覺服務來分析多張影像。

1. 在 Visual Studio Code 中，展開 **image-analysis** 資料夾及其包含的 **images** 資料夾。
2. 接著依序選取每個影像檔案以在 Visual Studio Code 中檢視。

## <a name="analyze-an-image-to-suggest-a-caption"></a>分析影像以建議標題

您現在已準備好使用 SDK 來呼叫電腦視覺服務，並分析影像。

1. 在用戶端應用程式的程式碼檔案 (**Program.cs** 或 **image-analysis.py**) 中，請注意 **Main** 函式中已經提供可載入組態設定的程式碼。 然後尋找 **Authenticate Computer Vision client** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以建立及驗證電腦視覺用戶端物件：

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

2. 在 **Main** 函式中您剛才新增的程式碼之下，請注意程式碼會指定影像檔的路徑，然後將影像路徑傳遞至其他兩個函式 (**AnalyzeImage** 和 **GetThumbnail**)。 這些函式尚未完全實作。

3. 在 **AnalyzeImage** 函式的 **Specify features to be retrieved** 註解之下，新增下列程式碼：

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. 在 **AnalyzeImage** 函式的註解 **Get image analysis** 底下新增下列程式碼 (包括指出您稍後將新增更多程式碼的位置註解)：

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. 儲存變更並返回 **image-analysis** 資料夾的整合式終端機，然後輸入下列命令來執行包含引數 **images/street.jpg** 程式：

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 觀察輸出，其中應該包含 **street.jpg** 影像的建議標題。
7. 再次執行程式，這次使用引數 **images/building.jpg** 來查看為 **building.jpg** 影像產生的標題。
8. 重複上一個步驟以產生 **images/person.jpg** 檔案的標題。

## <a name="get-suggested-tags-for-an-image"></a>取得影像的建議標籤

有時候，識別提供影像內容線索的相關 *標籤* 可能很有用。

1. 在 **AnalyzeImage** 函式的 **Get image tags** 註解之下，新增下列程式碼：

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔案執行一次程式，觀察除了影像標題之外，是否也有顯示建議標籤的清單。

## <a name="get-image-categories"></a>取得影像類別

電腦視覺服務可以建議影像的 *類別*，而且可以在每個類別中識別著名地標。

1. 在 **AnalyzeImage** 函式的 **Get image tags** \(取得影像標籤\) 註解下，新增下列程式碼：

**C#**

```C
// Get image categories
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

```

**Python**

```Python
# Get image categories
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

```
    
2. 儲存您的變更並針對 **images** 資料夾中的每個影像檔案執行一次程式，觀察除了影像標題和標籤之外，是否也有顯示建議的類別清單，以及任何可辨識的地標清單 (特別是在 **building.jpg** 影像中)。

## <a name="get-brands-in-an-image"></a>取得影像中的品牌

某些品牌可從標誌的視覺上辨識，即使未顯示品牌的名稱也可以。 電腦視覺服務經過訓練，可識別數千個知名品牌。

1. 在 **AnalyzeImage** 函式的 **Get brands in the image** 註解之下，新增下列程式碼：

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. 儲存您的變更，並針對 **images** 資料夾中的每一個影像檔執行一次程式，查看是否有辨識出任何品牌 (特別是 **person.jpg** 影像)。

## <a name="detect-and-locate-objects-in-an-image"></a>偵測並找出影像中的物件

*物件偵測* 是一種特別形式的電腦視覺，用於識別影像內的個別物件，並用週框方塊指示其所在位置。

1. 在 **AnalyzeImage** 函式的 **Get objects in the image** 註解之下，新增下列程式碼：

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔案執行一次程式，並觀察是否偵測到任何物件。 在每次執行後，檢視在與程式碼檔案相同的資料夾中產生的 **objects.jpg** 檔案，以查看標註的物件。

## <a name="get-moderation-ratings-for-an-image"></a>取得影像的仲裁評等

某些影像可能不適合所有受眾，因此您可能需要套用一些仲裁來識別具有成人或暴力性質的影像。

1. 在 **AnalyzeImage** 函式的 **Get moderation ratings** 註解之下，新增下列程式碼：

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔案執行一次程式，並觀察每個影像的評等。

> **注意**：在上述工作中，您使用是單一方法來分析影像，然後以累加方式新增程式碼來剖析並顯示結果。 SDK 也提供個別方法來建議標題、識別標籤、偵測物件等等，這表示您可以使用最適當的方法來只傳回所需的資訊，減少需要傳回的資料承載大小。 如需詳細資訊，請參閱 [.NET SDK 文件](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet)或 [Python SDK 文件](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python)。

## <a name="generate-a-thumbnail-image"></a>產生縮圖影像

在某些情況下，您可能需要建立稱為 *縮圖* 的較小影像版本，裁剪影像以在新尺寸影像中包含主要視覺主體。

1. 在您的程式碼檔案中，尋找 **GetThumbnail** 函式，並在註解 **Generate a thumbnail** 底下新增下列程式碼：

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔執行一次程式，開啟與每個影像程式碼檔案相同資料夾中產生的 **thumbnail.jpg** 檔案。

## <a name="more-information"></a>詳細資訊

在此練習中，您已探索電腦視覺服務的一些影像分析和操作功能。 此服務也包含讀取文字、偵測臉部和其他電腦視覺工作的功能。

有關使用 **電腦視覺** 服務的更多資訊，請參閱 [電腦視覺文件](https://docs.microsoft.com/azure/cognitive-services/computer-vision/)。
