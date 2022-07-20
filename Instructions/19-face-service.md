---
lab:
  title: 偵測並分析臉部
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: 4f7f366d7f2fb221d4fcb89fdb3a4b21c88b915f
ms.sourcegitcommit: 3de3c160e5b2eae51308cacbd0bcd3dbdbe337a5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2022
ms.locfileid: "147375788"
---
# <a name="detect-and-analyze-faces"></a>偵測並分析臉部

偵測並分析人臉的能力，是核心 AI 功能。 在此練習中，您將探索兩個 Azure 認知服務，其可用來處理影像中的臉部：**電腦視覺** 服務和 **臉部** 服務。

> **注意**：從 2022 年 6 月 21 日開始，傳回個人識別資訊的認知服務功能僅限已獲授與 [有限存取權](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客戶使用。 此外，無法再使用推斷情緒狀態的功能。 這些限制可能會影響此實驗室練習。 我們正在解決此問題，但同時，您在遵循下列步驟時可能會遇到一些錯誤；對此我們表示抱歉。 如需 Microsoft 所做之變更和原因的詳細資訊，請參閱[臉部辨識的負責任 AI 投資和保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (英文)。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未完成此步驟，您必須複製本課程的程式碼存放庫：

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

在此練習中，您將完成部分實作的用戶端應用程式，其會使用電腦視覺 SDK 來分析影像中的臉部。

> **注意**：您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **19-face** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **computer-vision** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝電腦視覺 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. 檢視 **computer-vision** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

4. 開啟組態檔並更新其所包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。

5. 請注意，**computer-vision** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：detect-faces.py

6. 開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用電腦視覺 SDK 所需的命名空間：

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

## <a name="view-the-image-you-will-analyze"></a>檢視您將分析的影像

在此練習中，您將使用電腦視覺服務來分析人物的影像。

1. 在 Visual Studio Code 中，展開 **computer-vision** 資料夾及其包含的 **images** 資料夾。
2. 選取 **people.jpg** 影像加以檢視。

## <a name="detect-faces-in-an-image"></a>偵測影像中的人臉

您現在已準備好使用 SDK 來呼叫電腦視覺服務，並偵測影像中的臉部。

1. 在用戶端應用程式的程式碼檔案 (**Program.cs** 或 **detect-faces.py**) 中，請注意 **Main** 函式中已經提供可載入組態設定的程式碼。 然後尋找 **Authenticate Computer Vision client** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以建立及驗證電腦視覺用戶端物件：

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

2. 在 **Main** 函式中您剛才新增的程式碼之下，請注意程式碼會指定影像檔的路徑，然後將影像路徑傳遞至名為 **AnalyzeFaces** 的函式。 此函式尚未完全實作。

3. 在 **AnalyzeFaces** 函式的 **Specify features to be retrieved (faces)** 註解之下，新增下列程式碼：

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. 在 **AnalyzeFaces** 函式的 **Get image analysis** 註解之下，新增下列程式碼：

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person at approximately {r.Left}, {r.Top}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person at approximately {}, {}'.format(r.left, r.top)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. 儲存變更並返回 **computer-vision** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. 觀察輸出，其應該指出偵測到的臉部數目。
7. 檢視在與程式碼檔案相同的資料夾中產生的 **detected_faces.jpg** 檔案，以查看標註的臉部。 在此情況下，您的程式碼已使用臉部的特性來標示方塊左上角的位置，而周框方塊座標會在每個臉部周圍繪製矩形。

## <a name="prepare-to-use-the-face-sdk"></a>準備使用臉部 SDK

雖然 **電腦視覺** 服務提供基本臉部偵測 (以及許多其他影像分析功能)，但 **臉部** 服務提供更完整的臉部分析和辨識功能。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **19-face** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **face-api** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝臉部 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. 檢視 **face-api** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

4. 開啟組態檔並更新其所包含的組態值，以反映認知服務資源的 **端點** 和驗證 **金鑰**。 儲存您的變更。

5. 請注意，**face-api** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：analyze-faces.py

6. 開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用電腦視覺 SDK 所需的命名空間：

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. 在 **Main** 函式中，請注意已經提供可載入組態設定的程式碼。 然後尋找 **Authenticate Face client** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以建立及驗證 **FaceClient** 物件：

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. 在 **Main** 函式中您剛才新增的程式碼之下，請注意程式碼會顯示功能表，可讓您在程式碼中呼叫函式，以探索臉部服務的功能。 您將在本練習的其餘部分實作這些函式。

## <a name="detect-and-analyze-faces"></a>偵測並分析臉部

臉部服務的最基本功能之一就是偵測影像中的臉部，並判斷其特性，例如頭部姿勢、模糊、是否戴眼鏡等等。

1. 在應用程式的程式碼檔案中，在 **Main** 函式中，檢查使用者選取功能表選項 **1** 時所執行的程式碼。 此程式碼會呼叫 **DetectFaces** 函式，並將路徑傳遞至影像檔。
2. 在程式碼檔案中尋找 **DetectFaces** 函式，並在 **Specify facial features to be retrieved** 註解之下，新增下列程式碼：

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. 在 **DetectFaces** 函式中您剛才新增的程式碼之下，尋找 **Get faces** 註解並新增下列程式碼：

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. 檢查您新增至 **DetectFaces** 函式的程式碼。 會分析影像檔案並偵測其包含的任何臉部，包括年齡、情緒及配戴眼鏡等特性。 每個臉部的詳細資料都會顯示，包括指派給每個臉部的唯一臉部識別碼；而臉部的位置會使用週框方塊在影像上指出來。
5. 儲存變更並返回 **face-api** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    *C# 輸出可能會顯示警告，有關現在使用 **await** 運算子之非同步函式。您可以忽略這些警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 出現提示時，請輸入 **1** 並觀察輸出，其中應該包含偵測到的每個臉部的識別碼和特性。
7. 檢視在與程式碼檔案相同的資料夾中產生的 **detected_faces.jpg** 檔案，以查看標註的臉部。

## <a name="more-information"></a>詳細資訊

**臉部** 服務中還有幾個額外功能可供使用，但需遵循 [負責任 AI 標準](https://aka.ms/aah91ff)，並受有限存取權原則所限制。 這些功能包括識別、驗證及建立臉部辨識模型。 如需深入了解及申請存取權，請參閱[認知服務的有限存取權](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access) (英文)。

如需有關使用 **電腦視覺** 服務進行臉部偵測的詳細資訊，請參閱 [電腦視覺文件](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

若要深入了解 **臉部** 服務，請參閱 [臉部文件](https://docs.microsoft.com/azure/cognitive-services/face/)。
