---
title: "Kongを入手"
content-type: "tutorial"
book: "get-started"
chapter: 1
---
{:.note}
> 
> **注記：**
> このクイックスタートはローカルコンピューター上で実行され、 {{ site.base_gateway }}の機能を検索します。{{ site.base_gateway }}を本番環境ですぐに使えるAPIプラットフォームの一部として実行したい場合は、[/installページから始めてください](/gateway/{{ page.release }}/install/)。

[{{site.base_gateway}}](/gateway/latest/)は軽量、高速、柔軟なクラウドネイティブAPIゲートウェイです。

{{site.base_gateway}}はサービスアプリケーションの前面に位置し、動的に制御、分析、
リクエストと応答のルーティングを行います。{{site.base_gateway}}は、柔軟でローコードのプラグインベースの
アプローチを使用して、APIトラフィックポリシーを実装します

このチュートリアルでは、ローカルインストールをセットアップして{{site.base_gateway}}を使い始める方法、および一般的なAPIマネジメントタスクをいくつか説明します。

このページでは、 {{site.base_gateway}}の実行と[Admin API](/gateway/latest/admin-api)による検証について説明します。完了したら、次のタスクを実行してチュートリアルを完了できます。

* [サービスとルートの理解と設定](/gateway/{{ page.release }}/get-started/services-and-routes)
* [アップストリームサービスを保護するための Rate Limiting の設定](/gateway/{{ page.release }}/get-started/rate-limiting)
* [プロキシキャッシュによるシステムパフォーマンスの向上](/gateway/{{ page.release }}/get-started/proxy-caching)
* [サービスの水平スケーリングのためのロードバランシング](/gateway/{{ page.release }}/get-started/load-balancing)
* [Key Authenticationによるサービスの保護](/gateway/{{ page.release }}/get-started/key-authentication)

### 前提条件

* [Dockerは](https://docs.docker.com/get-docker/)、 {{site.base_gateway}}とサポートデータベースをローカルで実行するために使用されます。
* [curl](https://curl.se/)は{{site.base_gateway}}にリクエストを送信するために使用されます。`curl`は多くのシステムにプリインストールされています。
* [jq](https://stedolan.github.io/jq/)は、コマンドラインでJSON応答を処理するために使用されます。このツールは便利ですが、この チュートリアルのタスクを完了させるために必要なものではありません。`jq`を使用せずに続行する場合は、コマンドを 変更して`jq`処理を削除します。

### Kong を入手

このチュートリアルでは、{{site.base_gateway}} とそのサポートデータベースをすばやく実行するための `quickstart` スクリプトが提供されています。
このスクリプトは、Docker を使用して {{site.base_gateway}} を実行し、[PostgreSQL](https://www.postgresql.org/) データベースをバックアップデータベースとして使用します。

1. 以下のように`quickstart`スクリプトを使用して{{site.base_gateway}}を実行します。

   ```sh
   curl -Ls https://get.konghq.com/quickstart | bash
   ```

   {:.note}
   > 
   > **注** : クイックスタートのスクリプトは、{{site.ee_product_name}} をフリーモードで実行します。環境変数を介してライセンスをスクリプトに渡すことで、ライセンス付きで Kong を実行できます。これに関する機能やその他の高度な使用方法については、[コードリポジトリのドキュメント](https://github.com/Kong/get.konghq.com)を参照してください。

   このスクリプトは、{{site.base_gateway}}およびサポートする PostgreSQLデータベースのDockerコンテナを実行します。
   このスクリプトは、これらのコンテナが通信するためのDockerネットワークも作成します。最後に、データベースは
   適切な[移行](/gateway/latest/reference/cli/#kong-migrations)手順で初期化され、
   {{site.base_gateway}}の準備が整うと、次のメッセージが表示されます。

   ```text
   Kong Gateway Ready 
   ```

2. {{site.base_gateway}} が実行されていることを確認します。

   
   {{site.base_gateway}} は、デフォルトポート `8001` で Admin API を提供します。Admin API は、{{site.base_gateway}} の状態の照会と制御の両方に使用できます。次のコマンドは Admin API を照会し、ヘッダーのみを取得します。

   ```sh
   curl --head localhost:8001
   ```

   {{site.base_gateway}}が正常に実行されている場合、次のような`200`HTTPコードで応答します。

   ```text
   HTTP/1.1 200 OK
   Date: Mon, 22 Aug 2022 19:25:49 GMT
   Content-Type: application/json
   Connection: keep-alive
   Access-Control-Allow-Origin: *
   Content-Length: 11063
   X-Kong-Admin-Latency: 6
   Server: kong/{{page.versions.ce}}
   ```

3. {{site.base_gateway}}構成を評価します。

   Admin APIのルートルートは、ネットワーク、セキュリティ、プラグインの情報といった
   
   {{site.base_gateway}}実行に関する重要な情報を提供します。完全な
   構成は、返される JSON ドキュメントの`.configuration`キーで提供されます。

   ```sh
   curl -s localhost:8001 | jq '.configuration'
   ```

   {{site.base_gateway}} の設定情報が記載された大きな JSON 応答が返されるはずです。
4. Kong Managerへのアクセス

   このガイドの残りではAdmin APIを使用した{{site.base_gateway}}の構成を示していますが、Kong Managerを使用してサービス、ルート、プラグインなどを管理することもできます。Kong Managerにアクセスするには、URL [http://localhost:8002](http://localhost:8002)にアクセスします。

   {% if_version gte:3.4.x %}
   {:.note}
   > 
   > **注:** {{site.ce_product_name}} をインストールすると、[Kong Manager Open Source](/gateway/{{page.release}}/kong-manager-oss/) が使用されます。その他すべての {{site.base_gateway}} インストールでは [Kong Manager Enterprise](/gateway/{{page.release}}/kong-manager/) を使用します。
   > {% endif_version %}

このチュートリアルのすべての手順には実行中の{{site.base_gateway}}が必要であるため、すべてを実行中のままこのチュートリアルの次の手順に進んでください。

