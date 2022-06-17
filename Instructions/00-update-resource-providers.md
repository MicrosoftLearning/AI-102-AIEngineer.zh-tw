---
lab:
  title: 啟用資源提供者
  module: Setup
ms.openlocfilehash: 2cd48d0d9f67b9bab3e64452bf6199e1f438492a
ms.sourcegitcommit: e06fbeaa3bc15e4dbe99f72216dfeeb27f58babd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/15/2022
ms.locfileid: "145195519"
---
# <a name="enable-resource-providers"></a>啟用資源提供者

您的 Azure 訂用帳戶中必須註冊一些資源提供者。 請遵循下列步驟，以確保其已註冊。

1. 使用與您的訂用帳戶相關聯的 Microsoft 認證來登入 Azure 入口網站 (位於 `https://portal.azure.com`)。
2. 在 [首頁] 頁面上，選取 [訂用帳戶] (或展開 **&#8801;** 功能表，選取 [所有服務]，然後在 [一般] 類別中選取 [訂用帳戶])。
3. 請選取您的 Azure 訂用帳戶 (如果您有多個訂用帳戶，請選取經由兌換您的 Azure Pass 所建立的訂用帳戶)。
4. 在訂用帳戶的刀鋒視窗中，於左側窗格的 [設定] 區段中，選取 [資源提供者]。
5. 在資源提供者清單中，確定已註冊下列提供者 (若未註冊，選取並按一下 [註冊])：
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
