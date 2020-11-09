---
ms.openlocfilehash: 003d7f382a0c950d31bb84176feb1e83fbd5cf19
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822697"
---
<!-- markdownlint-disable MD002 MD041 -->

このチュートリアルでは、Microsoft Graph を呼び出す HTTP トリガ関数を実装する簡単な Azure 関数を作成します。 これらの関数では、次のシナリオについて説明します。

- 送信 [フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 認証を使用して、ユーザーの受信トレイにアクセスするための API を実装します。
- [クライアント資格情報の付与フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)認証を使用して、ユーザーの受信トレイで通知をサブスクライブおよびサブスクライブ解除する API を実装します。
- クライアント資格情報の付与フローを使用して Microsoft Graph および access データから [変更通知](https://docs.microsoft.com/graph/webhooks) を受信するための webhook を実装します。

また、単純な JavaScript シングルページアプリケーション (SPA) を作成して、Azure 関数で実装されている Api を呼び出すこともできます。

## <a name="create-azure-functions-project"></a>Azure 関数プロジェクトを作成する

1. プロジェクトを作成するディレクトリで、コマンドラインインターフェイス (CLI) を開きます。 次のコマンドを実行します。

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. CLI の現在のディレクトリを **Graphtutorial** ディレクトリに変更し、次のコマンドを実行してプロジェクトに3つの関数を作成します。

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. **local.settings.js** を開き、次をファイルに追加します。このファイルには、 `http://localhost:8080` テストアプリケーションの URL である CORS を許可します。

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. プロジェクトをローカルで実行するには、次のコマンドを実行します。

    ```Shell
    func start
    ```

1. すべてが動作している場合は、次の出力が表示されます。

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. ブラウザーを開き、出力に表示されている関数の Url を参照して、関数が正しく動作していることを確認します。 ブラウザーに次のメッセージが表示されます `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` 。

## <a name="create-single-page-application"></a>単一ページアプリケーションを作成する

1. プロジェクトを作成するディレクトリで、CLI を開きます。 HTML ファイルと JavaScript ファイルを格納する **Testclient** という名前のディレクトリを作成します。

1. **Testclient** ディレクトリに **index.html** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    これにより、ナビゲーションバーを含む、アプリの基本的なレイアウトが定義されます。 また、次の項目も追加されます。

    - [ブートストラップ](https://getbootstrap.com/) とそれをサポートする JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [JavaScript の Microsoft 認証ライブラリ (MSAL.js) 2.0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > ページには、favicon () が含まれてい `<link rel="shortcut icon" href="g-raph.png">` ます。 この行を削除することも、 [GitHub](https://github.com/microsoftgraph/g-raph)から **g-raph.png** ファイルをダウンロードすることもできます。

1. **Testclient** ディレクトリに **スタイル .css** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="css" source="../demo/TestClient/style.css":::

1. **Testclient** ディレクトリに **ui.js** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    このコードでは、JavaScript を使用して、選択したビューに基づいて現在のページをレンダリングします。

### <a name="test-the-single-page-application"></a>単一ページアプリケーションをテストする

> [!NOTE]
> このセクションでは、 [dotnet](https://github.com/natemcmaster/dotnet-serve) を使用して、開発用コンピューターで簡単なテスト HTTP サーバーを実行する方法について説明します。 この特定のツールを使用する必要はありません。 必要なテストサーバーを使用して、 **Testclient** ディレクトリにサービスを提供できます。

1. **Dotnet** をインストールするには、CLI で次のコマンドを実行します。

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. CLI の現在のディレクトリを **Testclient** ディレクトリに変更し、次のコマンドを実行して HTTP サーバーを開始します。

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. ブラウザーを開き、`http://localhost:8080` に移動します。 ページは表示されますが、現在機能しているボタンはありません。

## <a name="add-nuget-packages"></a>NuGet パッケージを追加する

に進む前に、後で使用する追加の NuGet パッケージをインストールします。

- Azure 関数プロジェクトでの依存関係の挿入を有効にするための[拡張機能](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions)。
- [ Uration をMicrosoft.Extensions.Configします。](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) [.Net 開発者シークレットストア](https://docs.microsoft.com/aspnet/core/security/app-secrets)からアプリケーション構成を読み取るための usersecrets。
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/): Microsoft Graph を呼び出すためのものです。
- トークンを認証および管理するための[id](https://www.nuget.org/packages/Microsoft.Identity.Client/) 。
- トークンの検証のための OpenID configuration を取得するための、Id を使用して[接続し](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect)ます。
- Web API に送信されたトークンを検証するための[システムのモデル](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt)です。

1. CLI の現在のディレクトリを **Graphtutorial** ディレクトリに変更し、次のコマンドを実行します。

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
