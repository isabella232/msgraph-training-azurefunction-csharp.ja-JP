---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822690"
---
<!-- markdownlint-disable MD002 MD041 -->

このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する Azure 関数を構築する方法について説明します。

> [!TIP]
> 完成したチュートリアルをダウンロードするだけで済む場合は、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)をダウンロードするか、クローンを作成できます。 アプリ ID と secret を使用してアプリを構成する方法については、 **demo** フォルダーの README ファイルを参照してください。

## <a name="prerequisites"></a>前提条件

このチュートリアルを開始する前に、開発用コンピューターに次のツールをインストールしておく必要があります。

- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [Azure 関数コアツール](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

また、同じ組織内のグローバル管理者アカウントにアクセスできる、Microsoft の職場または学校アカウントを持っている必要があります。 Microsoft アカウントを持っていない場合は、 [office 365 開発者プログラムにサインアップ](https://developer.microsoft.com/office/dev-program) して、無料の office 365 サブスクリプションを取得することができます。

> [!NOTE]
> このチュートリアルは、上記のツールの次のバージョンで記述されています。 このガイドの手順は、他のバージョンでは動作しますが、テストされていません。
>
> - .NET Core SDK 3.1.301
> - Azure 関数コアツールの3.0.2630
> - Azure CLI 2.8.0
> - ngrok 2.3.35

## <a name="feedback"></a>フィードバック

このチュートリアルに関するフィードバックは、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)に記入してください。
