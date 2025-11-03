# API Gateway proxy と Node.js の統合

このサンプルアプリケーションは、API Gateway REST APIからのイベントを処理するLambda関数です。このAPIは、ウェブブラウザやその他のHTTPクライアントからアクセスできるパブリックエンドポイントを提供します。エンドポイントにリクエストを送信すると、APIはリクエストをシリアル化し、関数に送信します。関数はLambda APIを呼び出して使用率データを取得し、必要な形式でAPIに返します。

:warning: アプリケーションはインターネット経由でアクセス可能なパブリックAPIエンドポイントを作成します。テストが完了したら、クリーンアップスクリプトを実行してエンドポイントを削除してください。

![Architecture](/sample-apps/nodejs-apig/images/sample-nodejs-apig.png)

プロジェクト ソースには、関数コードとサポート リソースが含まれています。

- `function` - Node.js関数
- `template.yml` - アプリケーションを作成する AWS CloudFormation テンプレート
- `1-create-bucket.sh`, `2-deploy.sh`, etc. - AWS CLI を使用してアプリケーションをデプロイおよび管理するシェルスクリプト。

サンプル アプリケーションをデプロイするには、次の手順に従います。

# 要件
- [Node.js 18 と npm](https://nodejs.org/en/download/releases/)
- Bashシェル。LinuxとmacOSではデフォルトで含まれています。Windows 10では、[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10)をインストールすることで、Windows統合版のUbuntuとBashを入手できます。
- [The AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) v1.17 以降。

# セットアップ
このリポジトリをダウンロードまたはクローンします。

    $ git clone https://github.com/awsdocs/aws-lambda-developer-guide.git
    $ cd aws-lambda-developer-guide/sample-apps/nodejs-apig

サンプルアプリケーション用の新しいバケットを作成するには、`1-create-bucket.sh` を実行します。

    nodejs-apig$ ./1-create-bucket.sh
    make_bucket: lambda-artifacts-a5e491dbb5b22e0d

# デプロイ
アプリケーションをデプロイするには、`2-deploy.sh` を実行します。

    nodejs-apig$ ./2-deploy.sh
    added 16 packages from 18 contributors and audited 18 packages in 0.926s
    added 17 packages from 19 contributors and audited 19 packages in 0.916s
    Uploading to e678bc216e6a0d510d661ca9ae2fd941  2737254 / 2737254.0  (100.00%)
    Successfully packaged artifacts and wrote output template to file out.yml.
    Waiting for changeset to be created..
    Waiting for stack create/update to complete
    Successfully created/updated stack - nodejs-apig

このスクリプトは、AWS CloudFormation を使用して Lambda 関数と IAM ロールをデプロイします。リソースを含む AWS CloudFormation スタックが既に存在する場合、スクリプトはテンプレートまたは関数コードの変更に合わせてスタックを更新します。

# テスト
テスト イベント (`event.json`) を使用して関数を直接呼び出すには、`3-invoke.sh` を実行します。

    nodejs-apig$ ./3-invoke.sh
    {
        "StatusCode": 200,
        "ExecutedVersion": "$LATEST"
    }

スクリプトに関数を数回呼び出させたら、`CRTL+C` を押して終了します。

REST API を使って関数を呼び出すには、`4-get.sh` スクリプトを実行します。このスクリプトは cURL を使用して API エンドポイントに GET リクエストを送信します。

    nodejs-apig$ ./4-get.sh
    > GET /api/ HTTP/1.1
    > Host: mf2fxmplbj.execute-api.us-east-2.amazonaws.com
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Content-Type: application/json
    < Content-Length: 55
    < Connection: keep-alive
    < x-amzn-RequestId: cb863771-xmpl-47cb-869e-3433209223a8
    < X-Custom-Header: My value
    < X-Custom-Header: My other value
    < X-Amzn-Trace-Id: Root=1-5e67ea83-4826xmpl9be7bf422bf70049
    ...
    {
      "TotalCodeSize": 184440616,
      "FunctionCount": 39
    }

アプリケーションはAWS X-Rayを使用してリクエストをトレースします。[X-Rayコンソール](https://console.aws.amazon.com/xray/home#/service-map)を開いてサービスマップを表示します。以下のサービスマップは、関数が2つの方法で呼び出されたことを示しています。

![Service Map](/sample-apps/nodejs-apig/images/nodejs-apig-servicemap.png)

メイン関数グラフ内のノードを選択します。次に、**トレースを表示** を選択してトレースのリストを表示します。任意のトレースを選択すると、関数によって実行された処理の詳細を示すタイムラインが表示されます。

![Trace](/sample-apps/nodejs-apig/images/nodejs-apig-trace.png)

最後に、Lambda コンソールでアプリケーションを表示します。

*アプリケーションを表示するには*
1. Lambda コンソールで [applications page](https://console.aws.amazon.com/lambda/home#/applications) を開く
2.  **nodejs-apig**. を選択する

  ![Application](/sample-apps/nodejs-apig/images/nodejs-apig-application.png)

# クリーンアップ
アプリケーションを削除するには、 `5-cleanup.sh`. を実行します。

    nodejs-apig$ ./5-cleanup.sh

