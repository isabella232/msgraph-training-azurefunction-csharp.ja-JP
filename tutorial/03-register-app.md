---
ms.openlocfilehash: 0c9c6703a54e50f576a171a4a161305c9b9ae6ff
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822727"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2a44d-101">この手順では、Azure Active Directory 管理センターを使用して、次の3つの新しい Azure AD アプリケーションを作成します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-101">In this exercise you will create three new Azure AD applications using the Azure Active Directory admin center:</span></span>

- <span data-ttu-id="2a44d-102">シングルページアプリケーションのアプリ登録。これにより、ユーザーはサインインし、トークンを取得して、アプリケーションが Azure 関数を呼び出せるようになります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-102">An app registration for the single-page application so that it can sign in users and get tokens allowing the application to call the Azure Function.</span></span>
- <span data-ttu-id="2a44d-103">Microsoft Graph を呼び出せるようにするトークンについて、 [オンプレフロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) を使用して SPA によって送信されたトークンを交換できるようにする Azure 関数のアプリ登録。</span><span class="sxs-lookup"><span data-stu-id="2a44d-103">An app registration for the Azure Function that allows it to use the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to exchange the token sent by the SPA for a token that will allow it to call Microsoft Graph.</span></span>
- <span data-ttu-id="2a44d-104">Azure 関数 webhook のアプリ登録。これにより、 [クライアント資格情報フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) を使用して、ユーザーなしで Microsoft Graph を呼び出せるようになります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-104">An app registration for the Azure Function webhook that allows it to use the [client credential flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) to call Microsoft Graph without a user.</span></span>

> [!NOTE]
> <span data-ttu-id="2a44d-105">この例では、送信側フローとクライアント資格情報フローの両方を実装しているため、3つのアプリ登録が必要です。</span><span class="sxs-lookup"><span data-stu-id="2a44d-105">This example requires three app registrations because it is implementing both the on-behalf-of flow and the client credential flow.</span></span> <span data-ttu-id="2a44d-106">Azure 関数がこれらのフローのいずれかのみを使用する場合は、そのフローに対応するアプリの登録のみを作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-106">If your Azure Function only uses one of these flows, you would only need to create the app registrations that correspond to that flow.</span></span>

1. <span data-ttu-id="2a44d-107">ブラウザーを開き、 [Azure Active Directory 管理センター](https://aad.portal.azure.com) に移動して、Microsoft 365 テナントの組織管理者を使用してログインします。</span><span class="sxs-lookup"><span data-stu-id="2a44d-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using an Microsoft 365 tenant organization admin.</span></span>

1. <span data-ttu-id="2a44d-108">左側のナビゲーションで **[Azure Active Directory]** を選択し、それから **[管理]** で **[アプリの登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-108">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="2a44d-109">アプリの登録のスクリーンショット</span><span class="sxs-lookup"><span data-stu-id="2a44d-109">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

## <a name="register-an-app-for-the-single-page-application"></a><span data-ttu-id="2a44d-110">シングルページアプリケーション用のアプリを登録する</span><span class="sxs-lookup"><span data-stu-id="2a44d-110">Register an app for the single-page application</span></span>

1. <span data-ttu-id="2a44d-111">**[新規登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-111">Select **New registration**.</span></span> <span data-ttu-id="2a44d-112">**[アプリケーションを登録]** ページで、次のように値を設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="2a44d-113">`Graph Azure Function Test App` に **[名前]** を設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-113">Set **Name** to `Graph Azure Function Test App`.</span></span>
    - <span data-ttu-id="2a44d-114">**サポートされているアカウントの種類** は、 **この組織ディレクトリのアカウント** に対してのみ設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-114">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="2a44d-115">[ **リダイレクト URI** ] で、ドロップダウンを [ **シングルページアプリケーション (SPA)** ] に変更し、値をに設定し `http://localhost:8080` ます。</span><span class="sxs-lookup"><span data-stu-id="2a44d-115">Under **Redirect URI** , change the dropdown to **Single-page application (SPA)** and set the value to `http://localhost:8080`.</span></span>

    ![[アプリケーションを登録する] ページのスクリーンショット](./images/register-command-line-app.png)

1. <span data-ttu-id="2a44d-117">**[登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-117">Select **Register**.</span></span> <span data-ttu-id="2a44d-118">[ **Graph Azure 関数テストアプリ** ] ページで、 **アプリケーション (クライアント) id** と **ディレクトリ (テナント) id** の値をコピーして保存します。そのためには、後の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-118">On the **Graph Azure Function Test App** page, copy the values of the **Application (client) ID** and **Directory (tenant) ID** and save them, you will need them in the later steps.</span></span>

    ![新しいアプリ登録のアプリケーション ID のスクリーンショット](./images/aad-application-id.png)

## <a name="register-an-app-for-the-azure-function"></a><span data-ttu-id="2a44d-120">Azure 関数用のアプリを登録する</span><span class="sxs-lookup"><span data-stu-id="2a44d-120">Register an app for the Azure Function</span></span>

1. <span data-ttu-id="2a44d-121">[ **アプリの登録** ] に戻り、[ **新しい登録** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-121">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="2a44d-122">**[アプリケーションを登録]** ページで、次のように値を設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-122">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="2a44d-123">`Graph Azure Function` に **[名前]** を設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-123">Set **Name** to `Graph Azure Function`.</span></span>
    - <span data-ttu-id="2a44d-124">**サポートされているアカウントの種類** は、 **この組織ディレクトリのアカウント** に対してのみ設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-124">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="2a44d-125">**リダイレクト URI** を空白のままにします。</span><span class="sxs-lookup"><span data-stu-id="2a44d-125">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="2a44d-126">**[登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-126">Select **Register**.</span></span> <span data-ttu-id="2a44d-127">[ **Graph Azure 関数** ] ページで、 **アプリケーション (クライアント) ID** の値をコピーして保存します。そのためには、次の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-127">On the **Graph Azure Function** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="2a44d-128">**[管理]** で **[証明書とシークレット]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="2a44d-129">
            \*\*[新しいクライアント シークレット]** ボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-129">Select the **New client secret** button.</span></span> <span data-ttu-id="2a44d-130">**[説明]** に値を入力して、 **[有効期限]** のオプションのいずれかを選び、 **[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-130">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

    ![[クライアントシークレットの追加] ダイアログのスクリーンショット](./images/aad-new-client-secret.png)

1. <span data-ttu-id="2a44d-132">このページを離れる前に、クライアント シークレットの値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="2a44d-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="2a44d-133">次の手順で行います。</span><span class="sxs-lookup"><span data-stu-id="2a44d-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="2a44d-134">このクライアント シークレットは今後表示されないため、この段階で必ずコピーするようにしてください。</span><span class="sxs-lookup"><span data-stu-id="2a44d-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![新規追加されたクライアント シークレットのスクリーンショット](./images/aad-copy-client-secret.png)

1. <span data-ttu-id="2a44d-136">[ **管理** ] で、[ **API のアクセス許可** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-136">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="2a44d-137">[ **アクセス許可の追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-137">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="2a44d-138">[ **Microsoft Graph** ]、[委任された **アクセス許可** ] の順に選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-138">Select **Microsoft Graph** , then **Delegated Permissions**.</span></span> <span data-ttu-id="2a44d-139">メールを追加し **ます。読み取り** 、[ **アクセス許可の追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-139">Add **Mail.Read** and select **Add permissions**.</span></span>

    ![Azure 関数アプリの登録に対して構成されているアクセス許可のスクリーンショット](./images/web-api-configured-permissions.png)

1. <span data-ttu-id="2a44d-141">[ **Manage** ] の下にある [ **API を公開する** ] を選択し、[ **範囲の追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-141">Select **Expose an API** under **Manage** , then choose **Add a scope**.</span></span>

1. <span data-ttu-id="2a44d-142">既定の **アプリケーション ID URI** をそのまま使用し、[ **保存して続行** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-142">Accept the default **Application ID URI** and choose **Save and continue**.</span></span>

1. <span data-ttu-id="2a44d-143">[ **範囲の追加** ] フォームに次のように入力します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-143">Fill in the **Add a scope** form as follows:</span></span>

    - <span data-ttu-id="2a44d-144">**スコープ名:** メールを読み取ります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-144">**Scope name:** Mail.Read</span></span>
    - <span data-ttu-id="2a44d-145">**同意できるユーザー:** 管理者とユーザー</span><span class="sxs-lookup"><span data-stu-id="2a44d-145">**Who can consent?:** Admins and users</span></span>
    - <span data-ttu-id="2a44d-146">**管理者の同意の表示名:** すべてのユーザーの受信トレイの読み取り</span><span class="sxs-lookup"><span data-stu-id="2a44d-146">**Admin consent display name:** Read all users' inboxes</span></span>
    - <span data-ttu-id="2a44d-147">**管理者の同意の説明:** アプリですべてのユーザーの受信トレイを読み取ることができるようにします。</span><span class="sxs-lookup"><span data-stu-id="2a44d-147">**Admin consent description:** Allows the app to read all users' inboxes</span></span>
    - <span data-ttu-id="2a44d-148">**ユーザー同意の表示名:** 受信トレイの読み取り</span><span class="sxs-lookup"><span data-stu-id="2a44d-148">**User consent display name:** Read your inbox</span></span>
    - <span data-ttu-id="2a44d-149">**ユーザーの同意の説明:** アプリで受信トレイを読み取ることができるようにします。</span><span class="sxs-lookup"><span data-stu-id="2a44d-149">**User consent description:** Allows the app to read your inbox</span></span>
    - <span data-ttu-id="2a44d-150">**状態:** い</span><span class="sxs-lookup"><span data-stu-id="2a44d-150">**State:** Enabled</span></span>

1. <span data-ttu-id="2a44d-151">**[スコープの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-151">Select **Add scope**.</span></span>

1. <span data-ttu-id="2a44d-152">新しい範囲をコピーします。これは後の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-152">Copy the new scope, you'll need it in later steps.</span></span>

    ![Azure 関数アプリの登録で定義されているスコープのスクリーンショット](./images/web-api-defined-scopes.png)

1. <span data-ttu-id="2a44d-154">[ **管理** ] の下の [ **マニフェスト** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-154">Select **Manifest** under **Manage**.</span></span>

1. <span data-ttu-id="2a44d-155">マニフェスト内を見つけて、 `knownClientApplications` の現在の値を `[]` に置き換え `[TEST_APP_ID]` ます。ここで、 `TEST_APP_ID` は、 **Graph Azure 関数テストアプリ** の登録のアプリケーション ID です。</span><span class="sxs-lookup"><span data-stu-id="2a44d-155">Locate `knownClientApplications` in the manifest, and replace it's current value of `[]` with `[TEST_APP_ID]`, where `TEST_APP_ID` is the application ID of the **Graph Azure Function Test App** app registration.</span></span> <span data-ttu-id="2a44d-156">**[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-156">Select **Save**.</span></span>

> [!NOTE]
> <span data-ttu-id="2a44d-157">テストアプリケーションのアプリ ID を `knownClientApplications` Azure 関数のマニフェストのプロパティに追加すると、テストアプリケーションは、組み合わせられた [同意フロー](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent)を開始することができます。</span><span class="sxs-lookup"><span data-stu-id="2a44d-157">Adding the test application's app ID to the `knownClientApplications` property in the Azure Function's manifest allows the test application to trigger a [combined consent flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span></span> <span data-ttu-id="2a44d-158">これは、その代わりのフローが動作するために必要です。</span><span class="sxs-lookup"><span data-stu-id="2a44d-158">This is necessary for the on-behalf-of flow to work.</span></span>

## <a name="add-azure-function-scope-to-test-application-registration"></a><span data-ttu-id="2a44d-159">アプリケーション登録のテストに Azure 関数スコープを追加する</span><span class="sxs-lookup"><span data-stu-id="2a44d-159">Add Azure Function scope to test application registration</span></span>

1. <span data-ttu-id="2a44d-160">**Graph Azure 関数テストアプリ** の登録に戻り、[ **管理** ] の下にある [ **API アクセス許可** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-160">Return to the **Graph Azure Function Test App** registration, and select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="2a44d-161">[ **アクセス許可を追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-161">Select **Add a permission**.</span></span>

1. <span data-ttu-id="2a44d-162">[ **My api** ] を選択し、[ **さらに読み込む** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-162">Select **My APIs** , then select **Load more**.</span></span> <span data-ttu-id="2a44d-163">**Graph Azure 関数** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-163">Select **Graph Azure Function**.</span></span>

    ![[API アクセス許可の要求] ダイアログのスクリーンショット](./images/test-app-add-permissions.png)

1. <span data-ttu-id="2a44d-165">[ **メール] 読み取り** アクセス許可を選択し、[ **アクセス許可の追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-165">Select the **Mail.Read** permission, then select **Add permissions**.</span></span>

1. <span data-ttu-id="2a44d-166">[ **構成済みのアクセス許可** ] で、ユーザーを削除 **します。** アクセス許可の右側にある [ **.** ..] を選択して [ **削除] アクセス** 許可を選択することにより、 **Microsoft Graph** の下の [読み取り] アクセス許可を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-166">In the **Configured permissions** , remove the **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="2a44d-167">[ **はい、削除** ] を選択して確定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-167">Select **Yes, remove** to confirm.</span></span>

    ![テストアプリのアプリ登録に対して構成されているアクセス許可のスクリーンショット](./images/test-app-configured-permissions.png)

## <a name="register-an-app-for-the-azure-function-webhook"></a><span data-ttu-id="2a44d-169">Azure 関数 webhook のアプリを登録する</span><span class="sxs-lookup"><span data-stu-id="2a44d-169">Register an app for the Azure Function webhook</span></span>

1. <span data-ttu-id="2a44d-170">[ **アプリの登録** ] に戻り、[ **新しい登録** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-170">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="2a44d-171">**[アプリケーションを登録]** ページで、次のように値を設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-171">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="2a44d-172">`Graph Azure Function Webhook` に **[名前]** を設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-172">Set **Name** to `Graph Azure Function Webhook`.</span></span>
    - <span data-ttu-id="2a44d-173">**サポートされているアカウントの種類** は、 **この組織ディレクトリのアカウント** に対してのみ設定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-173">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="2a44d-174">**リダイレクト URI** を空白のままにします。</span><span class="sxs-lookup"><span data-stu-id="2a44d-174">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="2a44d-175">**[登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-175">Select **Register**.</span></span> <span data-ttu-id="2a44d-176">[ **Graph Azure 関数 webhook** ] ページで、 **アプリケーション (クライアント) ID** の値をコピーして保存します。次の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="2a44d-176">On the **Graph Azure Function webhook** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="2a44d-177">**[管理]** で **[証明書とシークレット]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-177">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="2a44d-178">
            \*\*[新しいクライアント シークレット]** ボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-178">Select the **New client secret** button.</span></span> <span data-ttu-id="2a44d-179">**[説明]** に値を入力して、 **[有効期限]** のオプションのいずれかを選び、 **[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-179">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

1. <span data-ttu-id="2a44d-180">クライアント シークレットの値をコピーしてから、このページから移動します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-180">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="2a44d-181">次の手順で行います。</span><span class="sxs-lookup"><span data-stu-id="2a44d-181">You will need it in the next step.</span></span>

1. <span data-ttu-id="2a44d-182">[ **管理** ] で、[ **API のアクセス許可** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-182">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="2a44d-183">[ **アクセス許可の追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-183">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="2a44d-184">[ **Microsoft Graph** ]、[ **アプリケーションのアクセス許可** ] の順に選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-184">Select **Microsoft Graph** , then **Application Permissions**.</span></span> <span data-ttu-id="2a44d-185">[ **ユーザー** の追加 **] を選び、[\*\*\*\*アクセス許可の追加** ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-185">Add **User.Read.All** and **Mail.Read** , then select **Add permissions**.</span></span>

1. <span data-ttu-id="2a44d-186">[ **構成済みのアクセス許可** ] で、委任されたユーザーを削除 **します。** アクセス許可の右側にある [ **.** ..] を選択し、[ **削除] アクセス** 許可を選択して、 **Microsoft Graph** の下にある読み取りアクセス許可</span><span class="sxs-lookup"><span data-stu-id="2a44d-186">In the **Configured permissions** , remove the delegated **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="2a44d-187">[ **はい、削除** ] を選択して確定します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-187">Select **Yes, remove** to confirm.</span></span>

1. <span data-ttu-id="2a44d-188">[ **管理者の同意を許可** する] ボタンを選択し、[ **はい** ] を選択して、構成されたアプリケーションのアクセス許可に対する管理者の同意を付与します。</span><span class="sxs-lookup"><span data-stu-id="2a44d-188">Select the **Grant admin consent for...** button, then select **Yes** to grant admin consent for the configured application permissions.</span></span> <span data-ttu-id="2a44d-189">[構成された **アクセス許可** ] テーブルの [ **状態** ] 列が [許可] に変更され **ます** 。</span><span class="sxs-lookup"><span data-stu-id="2a44d-189">The **Status** column in the **Configured permissions** table changes to **Granted for ...**.</span></span>

    ![管理者の同意が付与された webhook に対して構成されたアクセス許可のスクリーンショット](./images/webhook-configured-permissions.png)
