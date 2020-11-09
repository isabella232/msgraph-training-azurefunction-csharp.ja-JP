---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822709"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6fb72-101">この演習では、azure Function [app に発行](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)するための準備として、azure 関数のサンプルに必要な変更点について説明します。</span><span class="sxs-lookup"><span data-stu-id="6fb72-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="6fb72-102">コードを更新する</span><span class="sxs-lookup"><span data-stu-id="6fb72-102">Update code</span></span>

<span data-ttu-id="6fb72-103">構成はユーザーシークレットストアから読み取られます。これは開発コンピューターにのみ適用されます。</span><span class="sxs-lookup"><span data-stu-id="6fb72-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="6fb72-104">Azure に発行する前に、構成を保存する場所を変更し、それに応じて **Startup.cs** でコードを更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6fb72-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Startup.cs** accordingly.</span></span>

<span data-ttu-id="6fb72-105">アプリケーションシークレットは、 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)などのセキュリティで保護されたストレージに格納する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6fb72-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="6fb72-106">Azure 関数の CORS の設定を更新する</span><span class="sxs-lookup"><span data-stu-id="6fb72-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="6fb72-107">この例では、テストアプリケーションが関数を呼び出せるようにするために **local.settings.js** で CORS を構成しました。</span><span class="sxs-lookup"><span data-stu-id="6fb72-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="6fb72-108">公開された関数を構成して、それを呼び出す SPA アプリを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6fb72-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="6fb72-109">アプリの登録を更新する</span><span class="sxs-lookup"><span data-stu-id="6fb72-109">Update app registrations</span></span>

<span data-ttu-id="6fb72-110">`knownClientApplications` **Graph azure 関数** アプリ登録のマニフェストのプロパティは、azure 関数を呼び出すアプリのアプリケーション id で更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6fb72-110">The  `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="6fb72-111">既存のサブスクリプションを再作成する</span><span class="sxs-lookup"><span data-stu-id="6fb72-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="6fb72-112">ローカルマシンまたは ngrok の webhook URL を使用して作成されたすべてのサブスクリプションは、Azure 関数の運用 URL を使用して再作成する必要があり `Notify` ます。</span><span class="sxs-lookup"><span data-stu-id="6fb72-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
