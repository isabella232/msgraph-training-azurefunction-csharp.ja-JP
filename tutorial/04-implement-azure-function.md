---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822733"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ad479-101">この手順では、Azure 関数の実装を終了 `GetMyNewestMessage` し、関数を呼び出すテストクライアントを更新します。</span><span class="sxs-lookup"><span data-stu-id="ad479-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="ad479-102">Azure 関数は、 [代わりのフロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)を使用します。</span><span class="sxs-lookup"><span data-stu-id="ad479-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="ad479-103">このフローのイベントの基本的な順序は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="ad479-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="ad479-104">テストアプリケーションは、対話型認証フローを使用して、ユーザーがサインインして同意を与えることができるようにします。</span><span class="sxs-lookup"><span data-stu-id="ad479-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="ad479-105">Azure 関数にスコープが設定されているトークンを戻します。</span><span class="sxs-lookup"><span data-stu-id="ad479-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="ad479-106">このトークンには、Microsoft Graph のスコープが含まれ **ていません** 。</span><span class="sxs-lookup"><span data-stu-id="ad479-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="ad479-107">テストアプリケーションは、Azure 関数を呼び出し、そのアクセストークンをヘッダーに送信し `Authorization` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="ad479-108">Azure 関数はトークンを検証し、次に、そのトークンを Microsoft Graph スコープを含む2番目のアクセストークンに交換します。</span><span class="sxs-lookup"><span data-stu-id="ad479-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="ad479-109">Azure 関数は、2番目のアクセストークンを使用して、ユーザーの代わりに Microsoft Graph を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="ad479-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="ad479-110">アプリケーション ID とシークレットをソースに格納しないようにするには、 [.Net シークレットマネージャー](https://docs.microsoft.com/aspnet/core/security/app-secrets) を使用してこれらの値を格納します。</span><span class="sxs-lookup"><span data-stu-id="ad479-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="ad479-111">シークレットマネージャーは開発のみを目的としており、運用アプリでは、信頼されたシークレットマネージャーを使用して機密情報を格納する必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad479-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="ad479-112">認証を単一ページアプリケーションに追加する</span><span class="sxs-lookup"><span data-stu-id="ad479-112">Add authentication to the single page application</span></span>

<span data-ttu-id="ad479-113">最初に、SPA に認証を追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="ad479-114">これにより、アプリケーションは、Azure 関数を呼び出すためのアクセス許可を付与するアクセストークンを取得できるようになります。</span><span class="sxs-lookup"><span data-stu-id="ad479-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="ad479-115">これは SPA であるため、 [PKCE で認証コードフロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)を使用することになります。</span><span class="sxs-lookup"><span data-stu-id="ad479-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="ad479-116">**config.js** という名前の **testclient** ディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="ad479-117">`YOUR_TEST_APP_APP_ID_HERE`を azure ポータルで作成したアプリケーション ID に置き換えます。 **Graph Azure 関数テストアプリ** を使用します。</span><span class="sxs-lookup"><span data-stu-id="ad479-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="ad479-118">を `YOUR_TENANT_ID_HERE` Azure portal からコピーした **ディレクトリ (テナント) ID** 値に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ad479-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="ad479-119">を `YOUR_AZURE_FUNCTION_APP_ID_HERE` **Graph Azure 関数** のアプリケーション ID に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ad479-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="ad479-120">Git などのソース管理を使用している場合は、アプリ Id とテナント ID が誤ってリークしないように、ソース管理から **config.js** ファイルを除外することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="ad479-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="ad479-121">**auth.js** という名前の **testclient** ディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="ad479-122">このコードの内容を検討してください。</span><span class="sxs-lookup"><span data-stu-id="ad479-122">Consider what this code does.</span></span>

    - <span data-ttu-id="ad479-123">`PublicClientApplication` **config.js** に格納されている値を使用してを初期化します。</span><span class="sxs-lookup"><span data-stu-id="ad479-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="ad479-124">`loginPopup`Azure 関数のアクセス許可のスコープを使用して、ユーザーのサインインに使用します。</span><span class="sxs-lookup"><span data-stu-id="ad479-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="ad479-125">ユーザーのユーザー名をセッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="ad479-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="ad479-126">アプリではを使用しているため `loginPopup` 、ブラウザーのポップアップブロックを、からのポップアップを許可するように変更する必要がある場合があり `http://localhost:8080` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="ad479-127">ページを更新し、サインインします。</span><span class="sxs-lookup"><span data-stu-id="ad479-127">Refresh the page and sign in.</span></span> <span data-ttu-id="ad479-128">ページはユーザー名で更新され、サインインが成功したことを示します。</span><span class="sxs-lookup"><span data-stu-id="ad479-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

> [!TIP]
> <span data-ttu-id="ad479-129">でアクセストークンを解析し、 [https://jwt.ms](https://jwt.ms) `aud` 要求が azure 関数のアプリ ID であること、および要求に、 `scp` Microsoft Graph ではなく azure 関数のアクセス許可のスコープが含まれていることを確認できます。</span><span class="sxs-lookup"><span data-stu-id="ad479-129">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="ad479-130">Azure 関数に認証を追加する</span><span class="sxs-lookup"><span data-stu-id="ad479-130">Add authentication to the Azure Function</span></span>

<span data-ttu-id="ad479-131">このセクションでは、 `GetMyNewestMessage` Microsoft Graph と互換性のあるアクセストークンを取得するために、Azure 関数での代わりの流れを実装します。</span><span class="sxs-lookup"><span data-stu-id="ad479-131">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="ad479-132">**Graphtutorial .csproj** を含むディレクトリで CLI を開き、次のコマンドを実行して、.net 開発シークレットストアを初期化します。</span><span class="sxs-lookup"><span data-stu-id="ad479-132">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="ad479-133">次のコマンドを使用して、アプリケーション ID、シークレット、およびテナント ID をシークレットストアに追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-133">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="ad479-134">を `YOUR_API_FUNCTION_APP_ID_HERE` **Graph Azure 関数** のアプリケーション ID に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ad479-134">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="ad479-135">`YOUR_API_FUNCTION_APP_SECRET_HERE`を、 **Graph azure 関数** 用の azure portal で作成したアプリケーションシークレットに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ad479-135">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="ad479-136">を `YOUR_TENANT_ID_HERE` Azure portal からコピーした **ディレクトリ (テナント) ID** 値に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ad479-136">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="ad479-137">受信したベアラートークンを処理する</span><span class="sxs-lookup"><span data-stu-id="ad479-137">Process the incoming bearer token</span></span>

<span data-ttu-id="ad479-138">このセクションでは、SPA から Azure 関数に送信されたベアラートークンを検証して処理するクラスを実装します。</span><span class="sxs-lookup"><span data-stu-id="ad479-138">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="ad479-139">「 **Authentication** 」という名前を持つ「 **graphtutorial** 」ディレクトリに新しいディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="ad479-139">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="ad479-140">**/GraphTutorial/Authentication** フォルダーに **TokenValidationResult.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-140">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="ad479-141">**/GraphTutorial/Authentication** フォルダーに **TokenValidation.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-141">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="ad479-142">このコードの内容を検討してください。</span><span class="sxs-lookup"><span data-stu-id="ad479-142">Consider what this code does.</span></span>

- <span data-ttu-id="ad479-143">ヘッダーにベアラートークンがあることを確認し `Authorization` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-143">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="ad479-144">このメソッドは、Azure の発行された OpenID configuration から署名と発行元を確認します。</span><span class="sxs-lookup"><span data-stu-id="ad479-144">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="ad479-145">対象ユーザー ( `aud` クレーム) が Azure 関数のアプリケーション ID と一致することを確認します。</span><span class="sxs-lookup"><span data-stu-id="ad479-145">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="ad479-146">トークンを解析し、MSAL アカウント ID を生成します。これは、トークンキャッシュを利用するために必要になります。</span><span class="sxs-lookup"><span data-stu-id="ad479-146">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="ad479-147">代理認証プロバイダーを作成する</span><span class="sxs-lookup"><span data-stu-id="ad479-147">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="ad479-148">**OnBehalfOfAuthProvider.cs** という名前の **認証** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-148">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="ad479-149">**OnBehalfOfAuthProvider.cs** のコードについては、しばらくお待ちください。</span><span class="sxs-lookup"><span data-stu-id="ad479-149">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="ad479-150">関数では `GetAccessToken` 、まず、を使用してトークンキャッシュからユーザートークンを取得しようとし `AcquireTokenSilent` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-150">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="ad479-151">これが失敗した場合は、テストアプリによって Azure 関数に送信されたベアラートークンを使用して、ユーザーアサーションを生成します。</span><span class="sxs-lookup"><span data-stu-id="ad479-151">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="ad479-152">その後、そのユーザーアサーションを使用して、を使用してグラフ互換性のあるトークンを取得し `AcquireTokenOnBehalfOf` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-152">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="ad479-153">インターフェイスを実装 `Microsoft.Graph.IAuthenticationProvider` することで、このクラスをのコンストラクターに渡すことで、 `GraphServiceClient` 送信要求を認証できます。</span><span class="sxs-lookup"><span data-stu-id="ad479-153">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="ad479-154">Graph クライアントサービスを実装する</span><span class="sxs-lookup"><span data-stu-id="ad479-154">Implement a Graph client service</span></span>

<span data-ttu-id="ad479-155">このセクションでは、 [依存関係の挿入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)用に登録できるサービスを実装します。</span><span class="sxs-lookup"><span data-stu-id="ad479-155">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="ad479-156">このサービスは、認証済みのグラフクライアントを取得するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="ad479-156">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="ad479-157">**Services** という名前の **graphtutorial** ディレクトリに新しいディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="ad479-157">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="ad479-158">**IGraphClientService.cs** という名前の **サービス** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-158">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="ad479-159">**GraphClientService.cs** という名前の **サービス** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-159">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="ad479-160">次のプロパティをクラスに追加し `GraphClientService` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-160">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="ad479-161">次の関数をクラスに追加し `GraphClientService` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-161">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="ad479-162">関数のプレースホルダー実装を追加 `GetAppGraphClient` します。</span><span class="sxs-lookup"><span data-stu-id="ad479-162">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="ad479-163">これは、後のセクションで実装します。</span><span class="sxs-lookup"><span data-stu-id="ad479-163">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="ad479-164">この `GetUserGraphClient` 関数は、トークン検証の結果を取得し、 `GraphServiceClient` そのユーザーに対して認証を作成します。</span><span class="sxs-lookup"><span data-stu-id="ad479-164">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="ad479-165">**Startup.cs** という名前の **graphtutorial** ディレクトリに新しいファイルを作成し、そのファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-165">Create a new file in the **GraphTutorial** directory named **Startup.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    <span data-ttu-id="ad479-166">このコードにより、Azure 関数での [依存関係の挿入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) が可能になり、オブジェクトとサービスが公開され `IConfiguration` `GraphClientService` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-166">This code will enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `IConfiguration` object and the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="ad479-167">GetMyNewestMessage 関数を実装します。</span><span class="sxs-lookup"><span data-stu-id="ad479-167">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="ad479-168">/GraphTutorial/GetMyNewestMessage.cs を開き、内容全体を次のように置き換えます **。**</span><span class="sxs-lookup"><span data-stu-id="ad479-168">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="ad479-169">GetMyNewestMessage.cs のコードを確認する</span><span class="sxs-lookup"><span data-stu-id="ad479-169">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="ad479-170">**GetMyNewestMessage.cs** のコードについては、しばらくお待ちください。</span><span class="sxs-lookup"><span data-stu-id="ad479-170">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="ad479-171">コンストラクターでは、 `IConfiguration` 依存関係の挿入によって渡されたオブジェクトを保存します。</span><span class="sxs-lookup"><span data-stu-id="ad479-171">In the constructor, it saves the `IConfiguration` object passed in via dependency injection.</span></span>
- <span data-ttu-id="ad479-172">関数で `Run` は、次の処理が行われます。</span><span class="sxs-lookup"><span data-stu-id="ad479-172">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="ad479-173">必要な構成値がオブジェクトに存在することを検証し `IConfiguration` ます。</span><span class="sxs-lookup"><span data-stu-id="ad479-173">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="ad479-174">ベアラートークンを検証し、 `401` トークンが無効な場合は状態コードを返します。</span><span class="sxs-lookup"><span data-stu-id="ad479-174">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="ad479-175">`GraphClientService`この要求を行ったユーザーの Graph クライアントをから取得します。</span><span class="sxs-lookup"><span data-stu-id="ad479-175">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="ad479-176">Microsoft Graph SDK を使用して、ユーザーの受信トレイから最新のメッセージを取得し、それを応答で JSON 本文として返します。</span><span class="sxs-lookup"><span data-stu-id="ad479-176">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="ad479-177">テストアプリから Azure 関数を呼び出す</span><span class="sxs-lookup"><span data-stu-id="ad479-177">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="ad479-178">**auth.js** を開き、次の関数を追加して、アクセストークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="ad479-178">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="ad479-179">このコードの内容を検討してください。</span><span class="sxs-lookup"><span data-stu-id="ad479-179">Consider what this code does.</span></span>

    - <span data-ttu-id="ad479-180">最初に、ユーザーによる操作なしで、アクセストークンを暗黙的に取得しようとします。</span><span class="sxs-lookup"><span data-stu-id="ad479-180">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="ad479-181">ユーザーが既にサインインしている必要があるため、MSAL にはユーザーのキャッシュにトークンを含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad479-181">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="ad479-182">ユーザーが対話する必要があることを示すエラーで失敗した場合は、トークンを対話的に取得しようとします。</span><span class="sxs-lookup"><span data-stu-id="ad479-182">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

1. <span data-ttu-id="ad479-183">**azurefunctions.js** という名前の **testclient** ディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ad479-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="ad479-184">CLI の現在のディレクトリを **./graphチュートリアル** ディレクトリに変更し、次のコマンドを実行して Azure 関数をローカルで起動します。</span><span class="sxs-lookup"><span data-stu-id="ad479-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="ad479-185">まだ SPA が提供されていない場合は、2番目の CLI ウィンドウを開き、現在のディレクトリを **./TestClient** ディレクトリに変更します。</span><span class="sxs-lookup"><span data-stu-id="ad479-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="ad479-186">テストアプリケーションを実行するには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="ad479-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="ad479-187">ブラウザーを開き、`http://localhost:8080` に移動します。</span><span class="sxs-lookup"><span data-stu-id="ad479-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="ad479-188">サインインして、[ **最新のメッセージ** ] ナビゲーションアイテムを選択します。</span><span class="sxs-lookup"><span data-stu-id="ad479-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="ad479-189">アプリは、ユーザーの受信トレイにある最新のメッセージに関する情報を表示します。</span><span class="sxs-lookup"><span data-stu-id="ad479-189">The app displays information about the newest message in the user's inbox.</span></span>
