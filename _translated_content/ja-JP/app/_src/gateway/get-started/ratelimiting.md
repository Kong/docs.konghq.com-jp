---
title: "Rate Limitingの構成"
content-type: "tutorial"
---
流量制限とは何ですか？
-----------

流量制限は、アップストリームサービスに送信されるリクエストレートを制御します。DoS攻撃を防止し、Webスクレイピングやその他の過剰使用を制限するために使用できます。流量制限がないと、ユーザーは好きなだけリクエストを行うことができるため、トラフィックの急増につながり、アップストリームサービスの他のユーザーに影響を与える可能性があります。{{site.base_gateway}}では、流量制限により、アップストリームサービスへのリクエストを制限するパラメータを設定できます。

流量制限プラグイン
---------


{{site.base_gateway}}はKong の[流量制限プラグイン](/hub/kong-inc/rate-limiting/)を使用して流量制限を管理します。流量制限プラグインにはオープンソースバージョンとEnterpriseバージョンがあり、[Enterpriseバージョンではスライディングウィンドウアルゴリズムのサポート](https://en.wikipedia.org/wiki/Sliding_window_protocol)やRedisサポートなどの機能を利用できます。

流量制限プラグインは、各ユーザーがAPIを呼び出すことができる頻度を制限します。これにより、不注意または悪意のある乱用から保護されます。流量制限がないと、各ユーザーは好きなだけリクエストできるため、リクエストの「急増」につながり、他のコンシューマのリソースが枯渇する可能性があります。流量制限を有効にすると、1秒あたりのリクエスト数が固定数に制限されます。Kongは、流量制限プラグインのオープンソースバージョンとEnterpriseバージョンを提供しています。Enterpriseバージョンでは、APIがウィンドウの境界近くで過負荷になるのを防ぐためのスライディングウィンドウアルゴリズムをサポートし、パフォーマンスを向上させるためにRedisのサポートを追加しています。このガイドでは、プラグインのEnterpriseバージョンを使用します。高度な流量制限プラグインの詳細については、こちらをご覧ください。

Kong の[Rate Limitingプラグインを使用すると、](/hub/kong-inc/rate-limiting/)アップストリームサービスがAPIコンシューマから受信するリクエストの数や、各ユーザーがAPIを呼び出すことができる頻度を制限できます。

{:.note}
> 
> [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)<span class="badge enterprise"></span>プラグインは、APIがウィンドウの境界近くで過負荷になるのを防ぐためのスライディングウィンドウアルゴリズムをサポートし、パフォーマンスを向上させるためにRedisのサポートを追加しています。

Rate Limitingプラグインの構成
---------------------

プラグインを有効にするには、Admin APIの[プラグイン](/gateway/latest/admin-api/#add-plugin)オブジェクトに`POST`リクエストを送信します。

```sh
curl -i -X POST http://localhost:8001/plugins \
  --data name=rate-limiting \
  --data config.minute=5 \
  --data config.policy=local
```

このリクエストは、1分あたり5回を超えるリクエストに流量制限を適用するように、流量制限プラグインを設定します。

### 流量制限の検証

流量制限を構成したら、5分以内に6つのリクエストを作成して、正しく構成され機能していることを確認できます。流量制限を確認するには、CLIからAPIにリクエストを6回送信して、リクエストが流量制限されていることを確認します。[サービスとルートを構成](/gateway/latest/get-started/configure-services-and-routes/)ガイドを使用して{{site.base_gateway}}を構成した場合は、以下の例を使用します。それ以外の場合は、既存の値を独自の値に置き換えます。

{% navtabs %}
{% navtab Admin API %}

```sh
curl -i -X GET http://localhost:8000/mock/anything
```

{% endnavtab %}
{% navtab Web browser %}

または、ウェブブラウザから次の手順に従うこともできます。

1. `localhost:8000/mock`を入力し、ブラウザを6回更新します。6回目のリクエストの後、エラーメッセージが表示されます。
2. 少なくとも30秒待ってからもう一度試してください。 このサービスには、30秒以内に6回目のアクセスを試みるまでアクセスできます。

{% endnavtab %}
{% endnavtabs %}

6 回目のリクエストの後、429「API レート制限を超えました」エラーを受け取ります。

    {
    "message": "API rate limit exceeded"
    }

リクエストのたびに、`POST`リクエストで設定されたポリシーと照合するカウンターがトリガーされます。
値`config.policy = local`は、ポリシーとカウンターをメモリ内にローカルに保存するよう{{site.base_gateway}}に指示します。
`config.policy`で使用可能なオプションは次のとおりです。

* `local`\- カウンターはノード上のメモリ内にローカルに保存されます。
* `cluster`\- カウンターはKongデータストアに保存され、ノード間で共有されます。
* `redis`\- カウンターは Redis サーバーに保存され、ノード間で共有されます。

### ドメインごとの流量制限

Rate Limitingプラグインを有効にして、次の3つの異なるスコープに関連付けることができます。

* サービス \- サービスにリクエストを行う各コンシューマを制限します。

* ルート \- リクエストを行うすべてのコンシューマをサービスの特定のルートに制限します。

* コンシューマ \- `consumer_id`を使用して、サービスの任意のルートにリクエストを行う指定されたコンシューマのみを制限します。

### サービスレベルの流量制限

アップストリームアプリケーションへのリクエストを制限するために、特定のサービスに流量制限を設定できます。

```sh

curl -X POST http://localhost:8001/services/example_service/plugins \
--data "name=rate-limiting" \
--data config.minute=5 \
--data config.policy=local

```

### ルートレベルの流量制限

トラフィックを特定のルートに制限できます。

```sh

curl -X POST http://localhost:8001/routes/mock/plugins \
--data "name=rate-limiting" \
--data "config.minute=2" \
--data "config.hour=100"

```

[サービスとルートの構成](/gateway/latest/get-started/configure-services-and-routes/)ガイドに従わない場合、`mock`値を`route_id`に置き換えます。

### コンシューマレベルの流量制限

コンシューマレベルの流量制限は、特定のコンシューマに流量制限ルールを適用する場合に使用できます。コンシューマは、Admin API の[コンシューマオブジェクト](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を使用して作成されます。まだコンシューマを作成していないので、このタイプの流量制限を試すには、まずコンシューマを作成する必要があります。

```sh

curl -X POST http://localhost:8001/consumers/ \
  --data username=example_consumer

```

そして、`example_consumer`変数を使用して、 `plugins`オブジェクトへの別のリクエストを作成し、 `example_consumer`を`consumer.id`パラメータに渡します。

```sh

curl -X POST http://localhost:8001/plugins \
--data "name=rate-limiting" \
--data "consumer.id=example_consumer" \
--data "config.second=5" \
```

Rate Limiting Advanced プラグイン
----------------------------

Rate Limiting Advanced は以下を提供します:

* 追加の構成: `limit`、`window_size`、および`sync_rate`
* Redis Sentinel、Redisクラスタ、Redis SSLのサポート
* パフォーマンスの向上: Rate Limiting Advanced では、スループット パフォーマンスが向上し、精度も向上します。バックエンド ストレージと定期的に同期するように`sync_rate`を構成します。
* 選択できる制限アルゴリズムがさらに増えました。これらのアルゴリズムはより正確で、より具体的な構成が可能になります。当社のアルゴリズムの詳細については、「スケーラブルな流量制限アルゴリズムの設計方法」を参照してください。
* コンシューマグループのサポート：コンシューマのグループを選択するために異なる流量制限構成を適用します。

### 追加の構成

Rate Limiting Advancedプラグインの構成プロセスは、Rate Limitingプラグインの構成プロセスと似ています。Rate Limiting Advancedプラグインを構成するには、次のような`POST`リクエストを作成します。

```sh

curl -i -X POST http://localhost:8001/plugins \
--data name=rate-limiting-advanced \
--data config.limit=5 \
--data config.window_size=30

```

このリクエストは、`config.limit`および`config.window_size`フォームパラメータを利用します。

* config.limit: ウィンドウごとに許可されるリクエストの数。
* config.window\_size：リクエストを追跡するウィンドウサイズ（秒単位）。
* config.sync\_rate：カウンターデータを中央データストアに同期する頻度。 `-1`同期動作を完全に無視し、カウンターをノードメモリにのみ保存し、 `>0`その秒数でカウンターを同期します。

ここで「ウィンドウ」という言葉は、時間枠とその時間枠内で実行可能なリクエストを指します。作成したリクエストのうち、5つのリクエストは30秒以内に実行できます。6つ目のリクエストでサーバーは`429`応答コードを返します。このリクエストに複数の`window_size`と`config.limit`パラメータを追加して、流量制限ルールを細かく指定できます。

### 次の手順

このシリーズの次のチュートリアルでは、[プロキシキャッシュを構成する](/gateway/latest/get-started/configure-ratelimiting/)方法と理由を説明します。

