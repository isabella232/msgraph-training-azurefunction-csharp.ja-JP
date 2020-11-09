---
ms.openlocfilehash: f55eecd4d157c945d0e95870d4e481653b26c9ec
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822660"
---
<!-- markdownlint-disable MD002 MD041 -->

この手順では、Azure 関数の実装を完了し `SetSubscription` `Notify` 、ユーザーの受信トレイの変更をサブスクライブおよびサブスクライブ解除するようにテストアプリケーションを更新します。

- `SetSubscription`関数は API として機能し、テストアプリがユーザーの受信トレイの変更に対する[サブスクリプション](https://docs.microsoft.com/graph/webhooks)を作成または削除できるようにします。
- この `Notify` 関数は、サブスクリプションによって生成された変更通知を受け取る webhook として機能します。

どちらの関数も、 [クライアント資格情報の付与フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) を使用して、Microsoft Graph を呼び出すためのアプリ専用トークンを取得します。 管理者は必要なアクセス許可スコープに対する管理者の同意を付与されているため、トークンを取得するためにユーザーの操作は必要ありません。

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Azure 関数プロジェクトにクライアント資格情報認証を追加する

このセクションでは、Microsoft Graph と互換性のあるアクセストークンを取得するために、Azure 関数プロジェクトにクライアント資格情報フローを実装します。

1. **Graphtutorial .csproj** を含むディレクトリで、CLI を開きます。

1. 次のコマンドを使用して、webhook アプリケーション ID と secret をシークレットストアに追加します。 を `YOUR_WEBHOOK_APP_ID_HERE` **Graph Azure 関数 Webhook** のアプリケーション ID に置き換えます。 `YOUR_WEBHOOK_APP_SECRET_HERE`を azure ポータルで作成したアプリケーションシークレットに置き換えます ( **Graph Azure 関数 Webhook** )。

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>クライアント資格情報認証プロバイダを作成する

1. **ClientCredentialsAuthProvider.cs** という名前の **/GraphTutorial/Authentication** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

**ClientCredentialsAuthProvider.cs** のコードについては、しばらくお待ちください。

- コンストラクターでは、パッケージから **ConfidentialClientApplication** を初期化し `Microsoft.Identity.Client` ます。 および関数を使用して、 `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` `.WithTenantId(tenantId)` ログイン対象ユーザーを、指定した Microsoft 365 組織のみに制限します。
- 関数では `GetAccessToken` 、を呼び出して、 `AcquireTokenForClient` アプリケーションのトークンを取得します。 クライアント資格情報のトークンフローは常に非対話的です。
- インターフェイスを実装 `Microsoft.Graph.IAuthenticationProvider` することで、このクラスをのコンストラクターに渡すことで、 `GraphServiceClient` 送信要求を認証できます。

## <a name="update-graphclientservice"></a>GraphClientService の更新

1. **GraphClientService.cs** を開き、次のプロパティをクラスに追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. 既存の `GetAppGraphClient` 関数を、以下の関数で置き換えます。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

## <a name="implement-notify-function"></a>Notify 関数を実装する

このセクションで `Notify` は、変更通知の通知 URL として使用される関数を実装します。

1. 「 **モデル** 」という名前の **graphtutorials** ディレクトリに新しいディレクトリを作成します。

1. **ResourceData.cs** という名前の **モデル** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. **ChangeNotification.cs** という名前の **モデル** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotification.cs" id="ChangeNotificationSnippet":::

1. **NotificationList.cs** という名前の **モデル** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. /GraphTutorial/Notify.cs を開き、内容全体を次のように置き換えます **。**

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

**Notify.cs** のコードについては、しばらくお待ちください。

- 関数は、 `Run` クエリパラメーターが存在するかどうかを確認し `validationToken` ます。 このパラメーターが存在する場合、要求は [検証要求](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)として処理され、それに応じて応答します。
- 要求が検証要求ではない場合、JSON ペイロードはに逆シリアル化され `NotificationList` ます。
- リストの各通知には、想定されるクライアント状態の値がチェックされ、処理されます。
- 通知をトリガーしたメッセージは、Microsoft Graph を使用して取得されます。

## <a name="implement-setsubscription-function"></a>SetSubscription 関数を実装します。

このセクションでは、SetSubscription 関数を実装します。 この関数は、ユーザーの受信トレイに対するサブスクリプションを作成または削除するために、テストアプリケーションによって呼び出される API として機能します。

1. **SetSubscriptionPayload.cs** という名前の **モデル** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. /GraphTutorial/SetSubscription.cs を開き、内容全体を次のように置き換えます **。**

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

**SetSubscription.cs** のコードについては、しばらくお待ちください。

- この `Run` 関数は、POST 要求で送信された JSON ペイロードを読み取って、要求の種類 (サブスクライブまたは登録解除)、サブスクライブするユーザー ID、およびサブスクライブを解除するサブスクリプション id を決定します。
- 要求が subscribe 要求の場合、Microsoft Graph SDK を使用して、指定されたユーザーの受信トレイに新しいサブスクリプションを作成します。 サブスクリプションは、メッセージが作成または更新されたときに通知を行います。 新しいサブスクリプションは、応答の JSON ペイロードで返されます。
- 要求が登録解除要求の場合、Microsoft Graph SDK を使用して指定されたサブスクリプションを削除します。

## <a name="call-setsubscription-from-the-test-app"></a>テストアプリからの SetSubscription の呼び出し

このセクションでは、テストアプリでサブスクリプションを作成および削除するための関数を実装します。

1. **/TestClient/azurefunctions.js** を開き、次の関数を追加します。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    このコードでは、 `SetSubscription` Azure 関数を呼び出して、新しいサブスクリプションをサブスクライブし、セッション内のサブスクリプションの配列に追加します。

1. **azurefunctions.js** に次の関数を追加します。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    このコードでは、 `SetSubscription` Azure 関数を呼び出して、セッション内のサブスクリプションの配列からサブスクリプションを削除します。

1. Ngrok を実行していない場合は、ngrok ( `ngrok http 7071` ) を実行して、HTTPS 転送 URL をコピーします。

1. 次のコマンドを実行して、ngrok URL をユーザーシークレットストアに追加します。

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Ngrok を再起動した場合は、このコマンドを繰り返して ngrok URL を更新する必要があります。

1. CLI の現在のディレクトリを **./graphチュートリアル** ディレクトリに変更し、次のコマンドを実行して Azure 関数をローカルで起動します。

    ```Shell
    func start
    ```

1. SPA を更新し、[ **サブスクリプション** ] ナビゲーション項目を選択します。 Exchange Online メールボックスを持つ、Microsoft 365 組織のユーザーのユーザー ID を入力します。 これは、ユーザー `id` (Microsoft Graph から) またはユーザーのいずれかになり `userPrincipalName` ます。 [ **購読** ] をクリックします。

1. ページが更新され、テーブルに新しいサブスクリプションが表示されます。

1. ユーザーに電子メールを送信します。 短時間で、関数を `Notify` 呼び出す必要があります。 これは、ngrok web インターフェイス ( `http://localhost:4040` ) または Azure 関数プロジェクトのデバッグ出力で確認できます。

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. テストアプリで、サブスクリプションのテーブル行の [ **削除** ] をクリックします。 ページが更新され、サブスクリプションがテーブルにない状態になります。
