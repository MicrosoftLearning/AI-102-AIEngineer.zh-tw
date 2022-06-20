---
title: 線上託管說明
permalink: index.html
layout: home
ms.openlocfilehash: ba3d2a5154e5e9b08e750f7d9ced8b62048dd66d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2021
ms.locfileid: "145195494"
---
# <a name="content-directory"></a>內容目錄

此存放庫包含適用於 Microsoft 課程的實作教室練習 [AI-102 Designing and Implementing a Microsoft Azure AI Solution](https://docs.microsoft.com/learn/certifications/courses/ai-102t00) 和 [Microsoft Learn 等效自學型課程模組](https://aka.ms/AzureLearn_AIEngineer)。 此練習是為了配合學習材料並讓您使用其描述的技術進行練習而設計。

您將會需要 Microsoft Azure 訂用帳戶，以完成這些練習。 若您的講師沒有為您提供 Microsoft Azure 訂閱，您可以在 [https://azure.microsoft.com](https://azure.microsoft.com) 註冊免費試用版。

## <a name="labs"></a>實驗室

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| 模組 | 實驗室 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

