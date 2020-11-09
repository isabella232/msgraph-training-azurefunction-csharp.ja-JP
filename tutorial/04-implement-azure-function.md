---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822733"
---
<!-- markdownlint-disable MD002 MD041 -->

この手順では、Azure 関数の実装を終了 `GetMyNewestMessage` し、関数を呼び出すテストクライアントを更新します。

Azure 関数は、 [代わりのフロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)を使用します。 このフローのイベントの基本的な順序は次のとおりです。

- テストアプリケーションは、対話型認証フローを使用して、ユーザーがサインインして同意を与えることができるようにします。 Azure 関数にスコープが設定されているトークンを戻します。 このトークンには、Microsoft Graph のスコープが含まれ **ていません** 。
- テストアプリケーションは、Azure 関数を呼び出し、そのアクセストークンをヘッダーに送信し `Authorization` ます。
- Azure 関数はトークンを検証し、次に、そのトークンを Microsoft Graph スコープを含む2番目のアクセストークンに交換します。
- Azure 関数は、2番目のアクセストークンを使用して、ユーザーの代わりに Microsoft Graph を呼び出します。

> [!IMPORTANT]
> アプリケーション ID とシークレットをソースに格納しないようにするには、 [.Net シークレットマネージャー](https://docs.microsoft.com/aspnet/core/security/app-secrets) を使用してこれらの値を格納します。 シークレットマネージャーは開発のみを目的としており、運用アプリでは、信頼されたシークレットマネージャーを使用して機密情報を格納する必要があります。

## <a name="add-authentication-to-the-single-page-application"></a>認証を単一ページアプリケーションに追加する

最初に、SPA に認証を追加します。 これにより、アプリケーションは、Azure 関数を呼び出すためのアクセス許可を付与するアクセストークンを取得できるようになります。 これは SPA であるため、 [PKCE で認証コードフロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)を使用することになります。

1. **config.js** という名前の **testclient** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    `YOUR_TEST_APP_APP_ID_HERE`を azure ポータルで作成したアプリケーション ID に置き換えます。 **Graph Azure 関数テストアプリ** を使用します。 を `YOUR_TENANT_ID_HERE` Azure portal からコピーした **ディレクトリ (テナント) ID** 値に置き換えます。 を `YOUR_AZURE_FUNCTION_APP_ID_HERE` **Graph Azure 関数** のアプリケーション ID に置き換えます。

    > [!IMPORTANT]
    > Git などのソース管理を使用している場合は、アプリ Id とテナント ID が誤ってリークしないように、ソース管理から **config.js** ファイルを除外することをお勧めします。

1. **auth.js** という名前の **testclient** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    このコードの内容を検討してください。

    - `PublicClientApplication` **config.js** に格納されている値を使用してを初期化します。
    - `loginPopup`Azure 関数のアクセス許可のスコープを使用して、ユーザーのサインインに使用します。
    - ユーザーのユーザー名をセッションに格納します。

    > [!IMPORTANT]
    > アプリではを使用しているため `loginPopup` 、ブラウザーのポップアップブロックを、からのポップアップを許可するように変更する必要がある場合があり `http://localhost:8080` ます。

1. ページを更新し、サインインします。 ページはユーザー名で更新され、サインインが成功したことを示します。

> [!TIP]
> でアクセストークンを解析し、 [https://jwt.ms](https://jwt.ms) `aud` 要求が azure 関数のアプリ ID であること、および要求に、 `scp` Microsoft Graph ではなく azure 関数のアクセス許可のスコープが含まれていることを確認できます。

## <a name="add-authentication-to-the-azure-function"></a>Azure 関数に認証を追加する

このセクションでは、 `GetMyNewestMessage` Microsoft Graph と互換性のあるアクセストークンを取得するために、Azure 関数での代わりの流れを実装します。

1. **Graphtutorial .csproj** を含むディレクトリで CLI を開き、次のコマンドを実行して、.net 開発シークレットストアを初期化します。

    ```Shell
    dotnet user-secrets init
    ```

1. 次のコマンドを使用して、アプリケーション ID、シークレット、およびテナント ID をシークレットストアに追加します。 を `YOUR_API_FUNCTION_APP_ID_HERE` **Graph Azure 関数** のアプリケーション ID に置き換えます。 `YOUR_API_FUNCTION_APP_SECRET_HERE`を、 **Graph azure 関数** 用の azure portal で作成したアプリケーションシークレットに置き換えます。 を `YOUR_TENANT_ID_HERE` Azure portal からコピーした **ディレクトリ (テナント) ID** 値に置き換えます。

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>受信したベアラートークンを処理する

このセクションでは、SPA から Azure 関数に送信されたベアラートークンを検証して処理するクラスを実装します。

1. 「 **Authentication** 」という名前を持つ「 **graphtutorial** 」ディレクトリに新しいディレクトリを作成します。

1. **/GraphTutorial/Authentication** フォルダーに **TokenValidationResult.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. **/GraphTutorial/Authentication** フォルダーに **TokenValidation.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

このコードの内容を検討してください。

- ヘッダーにベアラートークンがあることを確認し `Authorization` ます。
- このメソッドは、Azure の発行された OpenID configuration から署名と発行元を確認します。
- 対象ユーザー ( `aud` クレーム) が Azure 関数のアプリケーション ID と一致することを確認します。
- トークンを解析し、MSAL アカウント ID を生成します。これは、トークンキャッシュを利用するために必要になります。

### <a name="create-an-on-behalf-of-authentication-provider"></a>代理認証プロバイダーを作成する

1. **OnBehalfOfAuthProvider.cs** という名前の **認証** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

**OnBehalfOfAuthProvider.cs** のコードについては、しばらくお待ちください。

- 関数では `GetAccessToken` 、まず、を使用してトークンキャッシュからユーザートークンを取得しようとし `AcquireTokenSilent` ます。 これが失敗した場合は、テストアプリによって Azure 関数に送信されたベアラートークンを使用して、ユーザーアサーションを生成します。 その後、そのユーザーアサーションを使用して、を使用してグラフ互換性のあるトークンを取得し `AcquireTokenOnBehalfOf` ます。
- インターフェイスを実装 `Microsoft.Graph.IAuthenticationProvider` することで、このクラスをのコンストラクターに渡すことで、 `GraphServiceClient` 送信要求を認証できます。

### <a name="implement-a-graph-client-service"></a>Graph クライアントサービスを実装する

このセクションでは、 [依存関係の挿入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)用に登録できるサービスを実装します。 このサービスは、認証済みのグラフクライアントを取得するために使用されます。

1. **Services** という名前の **graphtutorial** ディレクトリに新しいディレクトリを作成します。

1. **IGraphClientService.cs** という名前の **サービス** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. **GraphClientService.cs** という名前の **サービス** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. 次のプロパティをクラスに追加し `GraphClientService` ます。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. 次の関数をクラスに追加し `GraphClientService` ます。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. 関数のプレースホルダー実装を追加 `GetAppGraphClient` します。 これは、後のセクションで実装します。

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    この `GetUserGraphClient` 関数は、トークン検証の結果を取得し、 `GraphServiceClient` そのユーザーに対して認証を作成します。

1. **Startup.cs** という名前の **graphtutorial** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    このコードにより、Azure 関数での [依存関係の挿入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) が可能になり、オブジェクトとサービスが公開され `IConfiguration` `GraphClientService` ます。

### <a name="implement-getmynewestmessage-function"></a>GetMyNewestMessage 関数を実装します。

1. /GraphTutorial/GetMyNewestMessage.cs を開き、内容全体を次のように置き換えます **。**

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>GetMyNewestMessage.cs のコードを確認する

**GetMyNewestMessage.cs** のコードについては、しばらくお待ちください。

- コンストラクターでは、 `IConfiguration` 依存関係の挿入によって渡されたオブジェクトを保存します。
- 関数で `Run` は、次の処理が行われます。
  - 必要な構成値がオブジェクトに存在することを検証し `IConfiguration` ます。
  - ベアラートークンを検証し、 `401` トークンが無効な場合は状態コードを返します。
  - `GraphClientService`この要求を行ったユーザーの Graph クライアントをから取得します。
  - Microsoft Graph SDK を使用して、ユーザーの受信トレイから最新のメッセージを取得し、それを応答で JSON 本文として返します。

## <a name="call-the-azure-function-from-the-test-app"></a>テストアプリから Azure 関数を呼び出す

1. **auth.js** を開き、次の関数を追加して、アクセストークンを取得します。

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    このコードの内容を検討してください。

    - 最初に、ユーザーによる操作なしで、アクセストークンを暗黙的に取得しようとします。 ユーザーが既にサインインしている必要があるため、MSAL にはユーザーのキャッシュにトークンを含める必要があります。
    - ユーザーが対話する必要があることを示すエラーで失敗した場合は、トークンを対話的に取得しようとします。

1. **azurefunctions.js** という名前の **testclient** ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. CLI の現在のディレクトリを **./graphチュートリアル** ディレクトリに変更し、次のコマンドを実行して Azure 関数をローカルで起動します。

    ```Shell
    func start
    ```

1. まだ SPA が提供されていない場合は、2番目の CLI ウィンドウを開き、現在のディレクトリを **./TestClient** ディレクトリに変更します。 テストアプリケーションを実行するには、次のコマンドを実行します。

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. ブラウザーを開き、`http://localhost:8080` に移動します。 サインインして、[ **最新のメッセージ** ] ナビゲーションアイテムを選択します。 アプリは、ユーザーの受信トレイにある最新のメッセージに関する情報を表示します。
