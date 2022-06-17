---
lab:
  title: 辨識及合成語音
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 4f65f068cab76299f838153d32a2d4e8979a1253
ms.sourcegitcommit: 3374bf6b03869daf624f3916bc34510fcbe580e0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2022
ms.locfileid: "145195465"
---
# <a name="recognize-and-synthesize-speech"></a>辨識及合成語音

**語音** 服務是 Azure 認知服務，可提供語音相關功能，包括：

- 語音轉換文字 API，可讓您實作語音辨識 (將聽得見的口語字詞轉換成文字)。
- 文字轉換語音 API，可讓您實作語音合成 (將文字轉換成聽得見的語音)。

在此練習中，您將使用這兩個 API 來實作語音時鐘應用程式。

**注意**：此練習會要求您使用具有喇叭/耳機的電腦。 為了獲得最佳體驗，也需要麥克風。 有些裝載的虛擬環境可能能夠從本機麥克風擷取音訊，但如果這不可行 (或您完全沒有麥克風)，您可使用所提供的音訊檔案進行語音輸入。 請仔細遵循指示，因為您必須根據您使用的是麥克風或音訊檔案來選擇不同的選項。

## <a name="clone-the-repository-for-this-course"></a>複製本課程的存放庫

如果您尚未將 **AI-102-AIEngineer** 程式碼存放庫複製到您正此實驗室使用的環境，請遵循下列步驟來執行這項操作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：Clone** 命令，將 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    **注意**：如果系統提示您新增必要的資產來建置和偵錯，請選取 [現在不要]。

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

## <a name="prepare-to-use-the-speech-service"></a>準備使用語音服務

在此練習中，您將完成部分實作的用戶端應用程式，其使用語音 SDK 來辨識及合成語音。

**注意**：您可以選擇適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器] 窗格中，瀏覽至 **07-speech** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **speaking-clock** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝語音 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.19.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.19.0
    ```

3. 檢視 **speaking-clock** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#** ：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其中包含的組態值，以納入認知服務資源的 **金鑰**，以及其部署所在的 **位置**。 儲存您的變更。
4. 請注意，**speaking-clock** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#** ：Program.cs
    - **Python**：speaking-clock.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用語音 SDK 所需的命名空間：

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. 在 **Main** 函式中，請注意已經提供可從組態檔載入認知服務金鑰和區域的程式碼。 您必須使用這些變數來為您的認知服務資源建立 **SpeechConfig**。 在 **Configure speech service** 註解之下新增下列程式碼：

    **C#**
    
    ```C#
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```
    
    **Python**
    
    ```Python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

6. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

7. 如果您使用 C#，則可在非同步方法中忽略任何有關使用 **await** 運算子的警告，我們稍後會修正該問題。 程式碼應該顯示應用程式將使用的語音服務資源區域。

## <a name="recognize-speech"></a>辨識語音

既然您在認知服務資源中有語音服務的 **SpeechConfig**，即可使用 **語音轉換文字** API 來辨識語音並將其轉譯成文字。

### <a name="if-you-have-a-working-microphone"></a>如果您有運作中的麥克風

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **TranscribeCommand** 函式來接受口語輸入。
2. 在 **TranscribeCommand** 函式的 **設定語音辨識** 註解之下，新增適當的程式碼來建立 **SpeechRecognizer** 用戶端，以便使用預設系統麥克風來辨識和轉譯語音：

    **C#**
    
    ```C#
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```
    
    **Python**
    
    ```Python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

3. 現在跳到下面 **新增程式碼以處理轉譯的命令** 一節。

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

3. 在 **Main** 函式中，請注意程式碼會使用 **TranscribeCommand** 函式來接受口語輸入。 然後在 **TranscribeCommand** 函式的 **設定語音辨識** 註解之下，新增適當的程式碼來建立 **SpeechRecognizer** 用戶端，以便用於辨識音訊檔案中的語音並加以轉譯：

    **C#**

    ```C#
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech recognition
    audioFile = 'time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

### <a name="add-code-to-process-the-transcribed-command"></a>新增程式碼以處理轉譯的命令

1. 在 **TranscribeCommand** 函式的 **處理語音輸入** 註解之下，新增下列程式碼以接聽口語輸入，小心不要取代可傳回命令之函式尾端的程式碼：

    **C#**
    
    ```C#
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**
    
    ```Python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

2. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. 如果使用麥克風，請清楚說話並說出「現在幾點？」。 程式應該轉譯口語輸入，並根據程式碼執行所在電腦的當地時間顯示時間 (這可能不是您所在地點的正確時間)。

    SpeechRecognizer 提供約 5 秒的說話時間。 如果其偵測不到口語輸入，則會產生「沒有相符項目」結果。

    如果 SpeechRecognizer 發生錯誤，則會產生「已取消」的結果。 然後，應用程式中的程式碼會顯示錯誤訊息。 最可能的原因是組態檔中的金鑰或區域不正確。

## <a name="synthesize-speech"></a>合成語音

您的語音時鐘應用程式可接受口語輸入，但實際上不會說話！ 讓我們新增程式碼來合成語音，藉此修正該問題。

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **TellTime** 函式來告知使用者目前的時間。
2. 在 **TellTime** 函式的 **設定語音合成** 註解之下，新增下列程式碼來建立 **SpeechSynthesizer** 用戶端，以便用於產生語音輸出：

    **C#**
    
    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```
    
    > **注意**：*預設音訊設定會使用預設系統音訊裝置進行輸出，您就不需要明確提供 **AudioConfig**。如果您需要將音訊輸出重新導向至檔案，您可使用 **AudioConfig** 搭配檔案路徑來這麼做。*

3. 在 **TellTime** 函式的 **合成口語輸出** 註解之下，新增下列程式碼以產生口語輸出，小心不要取代可列印回應之函式尾端的程式碼：

    **C#**
    
    ```C#
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

4. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

5. 出現提示時，清楚對麥克風說話並說出「現在幾點？」。 程式應該說話，告知您時間。

## <a name="use-a-different-voice"></a>使用不同的語音

您的語音時鐘應用程式會使用預設語音，您可加以變更。 語音服務支援各種 *標準* 語音，以及更像人類的 *神經* 語音。 您也可以建立 *自訂* 語音。

> **注意**：如需神經和標準語音的清單，請參閱語音服務文件中的[語言和語音支援](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech)。

1. 在 **TellTime** 函式的 **設定語音合成** 註解之下，如下所示修改程式碼來指定替代語音，再建立 **SpeechSynthesizer** 用戶端：

   **C#**

    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

2. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. 出現提示時，清楚對麥克風說話並說出「現在幾點？」。 程式應該以指定的語音說話，告知您時間。

## <a name="use-speech-synthesis-markup-language"></a>使用語音合成標記語言

語音合成標記語言 (SSML) 可讓您自訂使用 XML 格式合成語音的方式。

1. 在 **TellTime** 函式中，以下列程式碼取代 **Synthesize spoken output** 註解之下的所有目前程式碼 (保留 **Print the response** 註解之下的程式碼)：

   **C#**

    ```C#
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**
    
    ```Python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. 出現提示時，清楚對麥克風說話並說出「現在幾點？」。 程式應該以 SSML 指定的語音說話 (覆寫 SpeechConfig 中指定的語音)，告知您時間，然後在暫停之後告知您是結束此實驗室的時候了！

## <a name="more-information"></a>詳細資訊

如需使用 **語音轉換文字** 和 **文字轉換語音** API 的詳細資訊，請參閱[語音轉換文字文件](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-to-text)和[文字轉換語音文件](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-text-to-speech)。
