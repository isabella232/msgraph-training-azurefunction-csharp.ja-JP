---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822709"
---
<!-- markdownlint-disable MD002 MD041 -->

この演習では、azure Function [app に発行](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)するための準備として、azure 関数のサンプルに必要な変更点について説明します。

## <a name="update-code"></a>コードを更新する

構成はユーザーシークレットストアから読み取られます。これは開発コンピューターにのみ適用されます。 Azure に発行する前に、構成を保存する場所を変更し、それに応じて **Startup.cs** でコードを更新する必要があります。

アプリケーションシークレットは、 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)などのセキュリティで保護されたストレージに格納する必要があります。

## <a name="update-cors-setting-for-azure-function"></a>Azure 関数の CORS の設定を更新する

この例では、テストアプリケーションが関数を呼び出せるようにするために **local.settings.js** で CORS を構成しました。 公開された関数を構成して、それを呼び出す SPA アプリを許可する必要があります。

## <a name="update-app-registrations"></a>アプリの登録を更新する

`knownClientApplications` **Graph azure 関数** アプリ登録のマニフェストのプロパティは、azure 関数を呼び出すアプリのアプリケーション id で更新する必要があります。

## <a name="recreate-existing-subscriptions"></a>既存のサブスクリプションを再作成する

ローカルマシンまたは ngrok の webhook URL を使用して作成されたすべてのサブスクリプションは、Azure 関数の運用 URL を使用して再作成する必要があり `Notify` ます。
