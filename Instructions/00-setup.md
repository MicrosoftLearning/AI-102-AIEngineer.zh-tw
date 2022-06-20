---
lab:
  title: 實驗室環境設定
  module: Setup
ms.openlocfilehash: 57363ce9fec94cb3a1a6ef7622a10bca48a6f055
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195520"
---
# <a name="lab-environment-setup"></a>實驗室環境設定

這些練習設計為在託管的實驗室環境中完成。 如果您想要在自己的電腦上完成這些練習，您可以藉由安裝下列軟體達成。 當您使用自己的環境時，可能會遇到未預料的對話方塊和行為。 由於可能發生的本機設定範圍十分廣泛，課程小組無法支援您在自己的環境中可能會遇到的問題。

> **注意**：下列指示適用於 Windows 10 電腦。 您也可以使用 Linux 或 MacOS。 您可能需要為您所選的 OS 調整實驗室指示。

### <a name="base-operating-system-windows-10"></a>基本作業系統 (Windows 10)

#### <a name="windows-10"></a>Windows 10

安裝 Windows 10 並套用所有更新。

#### <a name="edge"></a>Edge

安裝 [Edge (Chromium)](https://microsoft.com/edge)

### <a name="net-core-sdk"></a>.NET Core SDK

1. 從 https://dotnet.microsoft.com/download 下載並安裝 (下載 .NET Core SDK - 不只是執行階段)

### <a name="c-redistributable"></a>C++ 可轉散發套件

1. 從 https://aka.ms/vs/16/release/vc_redist.x64.exe 下載並安裝 Visual C++ 可轉散發套件 (x64)。

### <a name="nodejs"></a>Node.JS

1. 從 https://nodejs.org/en/download/ 下載最新的 LTS 版本 
2. 使用預設選項進行安裝

### <a name="python-and-required-packages"></a>Python (和必要套件)

1. 從 https://docs.conda.io/en/latest/miniconda.html 下載 3.8 版本 
2. 執行設定以開始安裝 - **重要事項**：選取該選項把 Miniconda 新增到 PATH 變數，並將 Miniconda 註冊為預設的 Python 環境。
3. 完成安裝後，開啟 Anaconda 提示字元，然後輸入下列命令來安裝套件： 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### <a name="azure-cli"></a>Azure CLI

1. 下載自 https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest 
2. 使用預設選項進行安裝

### <a name="git"></a>Git

1. 使用預設選項從 https://git-scm.com/download.html 下載，並安裝


### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code (和延伸模組)

1. 下載自 https://code.visualstudio.com/Download 
2. 使用預設選項進行安裝 
3. 完成安裝後，開啟 Visual Studio Code 並在 [擴充套件] 索引標籤 (CTRL+SHIFT+X) 上，搜尋並安裝 Microsoft 中的下列擴充套件：
    - Python
    - C#
    - Azure Functions
    - PowerShell


### <a name="bot-framework-emulator"></a>Bot Framework 模擬器

請遵循位於 https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md 的指示，來下載並安裝作業系統的最新穩定版本 Bot Framework Emulator。

### <a name="bot-framework-composer"></a>Bot Framework 編輯器

請從 https://docs.microsoft.com/en-us/composer/install-composer 安裝。
