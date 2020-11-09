---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822690"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5f8ff-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する Azure 関数を構築する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="5f8ff-102">完成したチュートリアルをダウンロードするだけで済む場合は、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)をダウンロードするか、クローンを作成できます。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="5f8ff-103">アプリ ID と secret を使用してアプリを構成する方法については、 **demo** フォルダーの README ファイルを参照してください。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="5f8ff-104">前提条件</span><span class="sxs-lookup"><span data-stu-id="5f8ff-104">Prerequisites</span></span>

<span data-ttu-id="5f8ff-105">このチュートリアルを開始する前に、開発用コンピューターに次のツールをインストールしておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="5f8ff-106">.NET Core SDK</span><span class="sxs-lookup"><span data-stu-id="5f8ff-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="5f8ff-107">Azure 関数コアツール</span><span class="sxs-lookup"><span data-stu-id="5f8ff-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="5f8ff-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="5f8ff-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="5f8ff-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="5f8ff-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="5f8ff-110">また、同じ組織内のグローバル管理者アカウントにアクセスできる、Microsoft の職場または学校アカウントを持っている必要があります。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="5f8ff-111">Microsoft アカウントを持っていない場合は、 [office 365 開発者プログラムにサインアップ](https://developer.microsoft.com/office/dev-program) して、無料の office 365 サブスクリプションを取得することができます。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="5f8ff-112">このチュートリアルは、上記のツールの次のバージョンで記述されています。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="5f8ff-113">このガイドの手順は、他のバージョンでは動作しますが、テストされていません。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="5f8ff-114">.NET Core SDK 3.1.301</span><span class="sxs-lookup"><span data-stu-id="5f8ff-114">.NET Core SDK 3.1.301</span></span>
> - <span data-ttu-id="5f8ff-115">Azure 関数コアツールの3.0.2630</span><span class="sxs-lookup"><span data-stu-id="5f8ff-115">Azure Functions Core Tools 3.0.2630</span></span>
> - <span data-ttu-id="5f8ff-116">Azure CLI 2.8.0</span><span class="sxs-lookup"><span data-stu-id="5f8ff-116">Azure CLI 2.8.0</span></span>
> - <span data-ttu-id="5f8ff-117">ngrok 2.3.35</span><span class="sxs-lookup"><span data-stu-id="5f8ff-117">ngrok 2.3.35</span></span>

## <a name="feedback"></a><span data-ttu-id="5f8ff-118">フィードバック</span><span class="sxs-lookup"><span data-stu-id="5f8ff-118">Feedback</span></span>

<span data-ttu-id="5f8ff-119">このチュートリアルに関するフィードバックは、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)に記入してください。</span><span class="sxs-lookup"><span data-stu-id="5f8ff-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
