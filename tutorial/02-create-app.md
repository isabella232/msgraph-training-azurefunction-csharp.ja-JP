---
ms.openlocfilehash: 003d7f382a0c950d31bb84176feb1e83fbd5cf19
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822697"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bd176-101">このチュートリアルでは、Microsoft Graph を呼び出す HTTP トリガ関数を実装する簡単な Azure 関数を作成します。</span><span class="sxs-lookup"><span data-stu-id="bd176-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="bd176-102">これらの関数では、次のシナリオについて説明します。</span><span class="sxs-lookup"><span data-stu-id="bd176-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="bd176-103">送信 [フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 認証を使用して、ユーザーの受信トレイにアクセスするための API を実装します。</span><span class="sxs-lookup"><span data-stu-id="bd176-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="bd176-104">[クライアント資格情報の付与フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)認証を使用して、ユーザーの受信トレイで通知をサブスクライブおよびサブスクライブ解除する API を実装します。</span><span class="sxs-lookup"><span data-stu-id="bd176-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="bd176-105">クライアント資格情報の付与フローを使用して Microsoft Graph および access データから [変更通知](https://docs.microsoft.com/graph/webhooks) を受信するための webhook を実装します。</span><span class="sxs-lookup"><span data-stu-id="bd176-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="bd176-106">また、単純な JavaScript シングルページアプリケーション (SPA) を作成して、Azure 関数で実装されている Api を呼び出すこともできます。</span><span class="sxs-lookup"><span data-stu-id="bd176-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="bd176-107">Azure 関数プロジェクトを作成する</span><span class="sxs-lookup"><span data-stu-id="bd176-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="bd176-108">プロジェクトを作成するディレクトリで、コマンドラインインターフェイス (CLI) を開きます。</span><span class="sxs-lookup"><span data-stu-id="bd176-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="bd176-109">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="bd176-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="bd176-110">CLI の現在のディレクトリを **Graphtutorial** ディレクトリに変更し、次のコマンドを実行してプロジェクトに3つの関数を作成します。</span><span class="sxs-lookup"><span data-stu-id="bd176-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="bd176-111">**local.settings.js** を開き、次をファイルに追加します。このファイルには、 `http://localhost:8080` テストアプリケーションの URL である CORS を許可します。</span><span class="sxs-lookup"><span data-stu-id="bd176-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="bd176-112">プロジェクトをローカルで実行するには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="bd176-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="bd176-113">すべてが動作している場合は、次の出力が表示されます。</span><span class="sxs-lookup"><span data-stu-id="bd176-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="bd176-114">ブラウザーを開き、出力に表示されている関数の Url を参照して、関数が正しく動作していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="bd176-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="bd176-115">ブラウザーに次のメッセージが表示されます `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` 。</span><span class="sxs-lookup"><span data-stu-id="bd176-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="bd176-116">単一ページアプリケーションを作成する</span><span class="sxs-lookup"><span data-stu-id="bd176-116">Create single-page application</span></span>

1. <span data-ttu-id="bd176-117">プロジェクトを作成するディレクトリで、CLI を開きます。</span><span class="sxs-lookup"><span data-stu-id="bd176-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="bd176-118">HTML ファイルと JavaScript ファイルを格納する **Testclient** という名前のディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="bd176-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="bd176-119">**Testclient** ディレクトリに **index.html** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bd176-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="bd176-120">これにより、ナビゲーションバーを含む、アプリの基本的なレイアウトが定義されます。</span><span class="sxs-lookup"><span data-stu-id="bd176-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="bd176-121">また、次の項目も追加されます。</span><span class="sxs-lookup"><span data-stu-id="bd176-121">It also adds the following:</span></span>

    - <span data-ttu-id="bd176-122">[ブートストラップ](https://getbootstrap.com/) とそれをサポートする JavaScript</span><span class="sxs-lookup"><span data-stu-id="bd176-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="bd176-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="bd176-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="bd176-124">JavaScript の Microsoft 認証ライブラリ (MSAL.js) 2.0</span><span class="sxs-lookup"><span data-stu-id="bd176-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="bd176-125">ページには、favicon () が含まれてい `<link rel="shortcut icon" href="g-raph.png">` ます。</span><span class="sxs-lookup"><span data-stu-id="bd176-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="bd176-126">この行を削除することも、 [GitHub](https://github.com/microsoftgraph/g-raph)から **g-raph.png** ファイルをダウンロードすることもできます。</span><span class="sxs-lookup"><span data-stu-id="bd176-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="bd176-127">**Testclient** ディレクトリに **スタイル .css** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bd176-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="bd176-128">**Testclient** ディレクトリに **ui.js** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bd176-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="bd176-129">このコードでは、JavaScript を使用して、選択したビューに基づいて現在のページをレンダリングします。</span><span class="sxs-lookup"><span data-stu-id="bd176-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="bd176-130">単一ページアプリケーションをテストする</span><span class="sxs-lookup"><span data-stu-id="bd176-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="bd176-131">このセクションでは、 [dotnet](https://github.com/natemcmaster/dotnet-serve) を使用して、開発用コンピューターで簡単なテスト HTTP サーバーを実行する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="bd176-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="bd176-132">この特定のツールを使用する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="bd176-132">Using this specific tool is not required.</span></span> <span data-ttu-id="bd176-133">必要なテストサーバーを使用して、 **Testclient** ディレクトリにサービスを提供できます。</span><span class="sxs-lookup"><span data-stu-id="bd176-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="bd176-134">**Dotnet** をインストールするには、CLI で次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="bd176-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="bd176-135">CLI の現在のディレクトリを **Testclient** ディレクトリに変更し、次のコマンドを実行して HTTP サーバーを開始します。</span><span class="sxs-lookup"><span data-stu-id="bd176-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="bd176-136">ブラウザーを開き、`http://localhost:8080` に移動します。</span><span class="sxs-lookup"><span data-stu-id="bd176-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="bd176-137">ページは表示されますが、現在機能しているボタンはありません。</span><span class="sxs-lookup"><span data-stu-id="bd176-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="bd176-138">NuGet パッケージを追加する</span><span class="sxs-lookup"><span data-stu-id="bd176-138">Add NuGet packages</span></span>

<span data-ttu-id="bd176-139">に進む前に、後で使用する追加の NuGet パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="bd176-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="bd176-140">Azure 関数プロジェクトでの依存関係の挿入を有効にするための[拡張機能](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions)。</span><span class="sxs-lookup"><span data-stu-id="bd176-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="bd176-141">[ Uration をMicrosoft.Extensions.Configします。](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) [.Net 開発者シークレットストア](https://docs.microsoft.com/aspnet/core/security/app-secrets)からアプリケーション構成を読み取るための usersecrets。</span><span class="sxs-lookup"><span data-stu-id="bd176-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="bd176-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/): Microsoft Graph を呼び出すためのものです。</span><span class="sxs-lookup"><span data-stu-id="bd176-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="bd176-143">トークンを認証および管理するための[id](https://www.nuget.org/packages/Microsoft.Identity.Client/) 。</span><span class="sxs-lookup"><span data-stu-id="bd176-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="bd176-144">トークンの検証のための OpenID configuration を取得するための、Id を使用して[接続し](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect)ます。</span><span class="sxs-lookup"><span data-stu-id="bd176-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="bd176-145">Web API に送信されたトークンを検証するための[システムのモデル](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt)です。</span><span class="sxs-lookup"><span data-stu-id="bd176-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="bd176-146">CLI の現在のディレクトリを **Graphtutorial** ディレクトリに変更し、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="bd176-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
