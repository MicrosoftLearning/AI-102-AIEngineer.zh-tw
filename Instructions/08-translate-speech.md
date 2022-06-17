---
lab:
  title: 翻譯語音
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 45c8d0d31bee5901247b22917d8dbad8c587a0bd
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2022
ms.locfileid: "145195518"
---
# <a name="translate-speech"></a>翻譯語音

**語音** 服務包含 **語音翻譯** API，可讓您用來翻譯口語。 例如，假設您想要開發一個翻譯工具應用程式，人們可以在不會說當地語言的地方旅遊時使用。 他們能夠說出「車站在哪裡？」之類的片語？ 或以自己的語言說出「我需要找藥局」，然後將其翻譯成當地語言。

**注意**：此練習會要求您使用具有喇叭/耳機的電腦。 為了獲得最佳體驗，也需要麥克風。 有些裝載的虛擬環境可能能夠從本機麥克風擷取音訊，但如果這不可行 (或您完全沒有麥克風)，您可使用所提供的音訊檔案進行語音輸入。 請仔細遵循指示，因為您必須根據您使用的是麥克風或音訊檔案來選擇不同的選項。

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
2. 按一下 [&#65291;建立資源] 按鈕，搜尋 *認知服務*，然後使用以下設定建立 **認知服務** 資源：
    - **訂用帳戶**：Azure 訂閱
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：選擇任一可用區域
    - **名稱**：*輸入唯一名稱*
    - **定價層**：標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署詳細資料。
5. 部署資源後，移至該資源並檢視其 [金鑰和端點] 頁面。 在下一個程序中，您需要其中一個金鑰，以及從此頁面佈建服務的位置。

## <a name="prepare-to-use-the-speech-translation-service"></a>準備使用語音翻譯服務

在此練習中，您將完成部分實作的用戶端應用程式，其使用語音 SDK 來辨識、翻譯及合成語音。

> **注意**：您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **08-speech-translation** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **translator** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝語音 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.19.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.19.0
    ```

3. 檢視 **translator** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以納入認知服務資源的 **金鑰**，以及其部署所在的 **位置**。 儲存您的變更。
4. 請注意，**translator** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：translator.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用語音 SDK 所需的命名空間：

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. 在 **Main** 函式中，請注意已經提供可從組態檔載入認知服務金鑰和區域的程式碼。 您必須使用這些變數來為認知服務資源建立 **SpeechTranslationConfig**，您會將其用於翻譯口語輸入。 在 **Configure translation** 註解之下新增下列程式碼：

    **C#**
    
    ```C#
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. 您將使用 **SpeechTranslationConfig** 將語音翻譯成文字，但您也會使用 **SpeechConfig** 將翻譯合成為語音。 在 **Configure speech** 註解之下新增下列程式碼：

    **C#**
    
    ```C#
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. 儲存變更並返回 **translator** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. 如果您使用 C#，則可在非同步方法中忽略任何有關使用 **await** 運算子的警告，我們稍後會修正該問題。 程式碼應會顯示一則訊息，指出其已準備好從 en-US 翻譯。 按下 ENTER 可結束程式。

## <a name="implement-speech-translation"></a>實作語音翻譯

既然您在認知服務資源中有語音服務的 **SpeechTranslationConfig**，即可使用 **語音翻譯** API 來辨識和翻譯語音。

### <a name="if-you-have-a-working-microphone"></a>如果您有運作中的麥克風

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **Translate** 函式來翻譯口語輸入。
2. 在 **Translate** 函式的 **Translate speech** 註解之下，新增下列程式碼來建立 **TranslationRecognizer** 用戶端，以便用於辨識和翻譯使用預設系統麥克風進行輸入的語音。

    **C#**
    
    ```C#
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **注意**：應用程式中的程式碼會將輸入翻譯成單一呼叫中的所有三種語言。 只會顯示特定語言的翻譯，但您可以在結果的 **translations** 集合中指定目的語言程式碼，以擷取任何翻譯。

3. 現在跳到下面「執行程式」一節。

### <a name="alternatively-use-audio-input-from-a-file"></a>或者，使用來自檔案的音訊輸入

1. 在終端機視窗中，輸入下列命令來安裝可用於播放音訊檔案的程式庫：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

2. 在程式的程式碼檔案中，於現有命名空間匯入之下，新增下列程式碼以匯入您剛才安裝的程式庫：

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

3. 在程式的 **Main** 函式中，請注意程式碼會使用 **Translate** 函式來翻譯口語輸入。 然後在 **Translate** 函式的 **翻譯語音** 註解之下，新增下列程式碼來建立 **TranslationRecognizer** 用戶端，以便用於辨識和翻譯檔案中的語音。

    **C#**
    
    ```C#
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **注意**：應用程式中的程式碼會將輸入翻譯成單一呼叫中的所有三種語言。 只會顯示特定語言的翻譯，但您可以在結果的 **translations** 集合中指定目的語言程式碼，以擷取任何翻譯。

### <a name="run-the-program"></a>執行程式

1. 儲存變更並返回 **translator** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

2. 出現提示時，輸入有效的語言代碼 (*fr*、*es* 或 *hi*)，然後若使用麥克風，請清楚說出「車站在哪裡？」 或您可能在出國旅遊時使用的一些其他片語。 程式應該轉譯您的口語輸入，並將其翻譯成您指定的語言 (法文、西班牙文或印度文)。 重複此程序，並嘗試應用程式支援的每種語言。 完成時，請按 ENTER 可結束程式。

    > **注意**：TranslationRecognizer 提供約 5 秒的說話時間。 如果其偵測不到口語輸入，則會產生「沒有相符項目」結果。
    >
    > 由於字元編碼問題，翻譯成印度文不一定會正確顯示在主控台視窗中。

## <a name="synthesize-the-translation-to-speech"></a>將翻譯合成為語音

到目前為止，您的應用程式會將口語輸入轉譯為文字；如果您需要在旅行時向某人尋求協助，可能就已足夠。 不過，最好以合適的語音朗讀翻譯。

1. 在 **Translate** 函式的 **合成翻譯** 註解之下，新增下列程式碼，以使用 **SpeechSynthesizer** 用戶端透過預設喇叭將翻譯合成為語音：

    **C#**
    
    ```C#
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. 儲存變更並返回 **translator** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. 出現提示時，輸入有效的語言代碼 (*fr*、*es* 或 *hi*)，然後清楚對麥克風說話，並說出您在國外旅遊時可能使用的片語。 程式應該轉譯您的口語輸入，並使用口語翻譯進行回應。 重複此程序，並嘗試應用程式支援的每種語言。 完成時，請按 ENTER 可結束程式。

    > **注意** *在此範例中，您已使用 **SpeechTranslationConfig** 將語音翻譯成文字，然後使用 **SpeechConfig** 將翻譯合成為語音。事實上，您可使用 **SpeechTranslationConfig** 直接合成翻譯，但這只有在翻譯成單一語言時有作用，而結果是通常儲存為檔案而非直接傳送到喇叭的音訊串流。*

## <a name="more-information"></a>詳細資訊

如需使用 **語音翻譯** API 的詳細資訊，請參閱[語音翻譯文件](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation)。
