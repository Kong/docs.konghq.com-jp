---
title: "プロキシキャッシュ"
content-type: "tutorial"
book: "get-started"
chapter: 4
---
Kong がパフォーマンスを実現する方法の 1 つはキャッシュです。
[Proxy Cache プラグイン](/hub/kong-inc/proxy-cache/)は、構成可能なレスポンスコード、コンテンツタイプ、およびリクエストメソッドに基づいてレスポンスをキャッシュすることで、パフォーマンスを向上させます。キャッシュを有効にすると、{{site.base_gateway}} がキャッシュされた結果を使用してアップストリームサービスに代わって応答するため、アップストリームサービスが反復的なリクエストによって停止することはありません。キャッシングは、特定の {{site.base_gateway}} オブジェクトに対して有効にしたり、すべてのリクエストに対してグローバルに有効にしたりすることもできます。

### キャッシュの有効期間（TTL）

TTL はキャッシュされたコンテンツの更新レートを管理します。これは、クライアントが古いコンテンツを提供しないようにするために重要です。TTLが30秒の場合、30秒以上経過したコンテンツは期限切れとみなされ、後続のリクエストで更新されます。TTL構成は、アップストリームサービスが提供するコンテンツの種類に基づいて異なる方法で設定する必要があります。

* 稀にしか更新されない静的データはTTLが長くなります。

* 動的データでは、古いデータの処理を回避するために短期間のTTLを使用する必要があります。


{{site.base_gateway}}は、[RFC\-7234セクション5\.2](https://tools.ietf.org/html/rfc7234)に従って、キャッシュ制御された操作を行います。TTL 設定の詳細については、仕様とProxy Cache
プラグイン[パラメータリファレンス](/hub/kong-inc/proxy-cache/#parameters)を参照してください。

キャッシュの有効化
---------

次のチュートリアルでは、{{site.base_gateway}}のさまざまな側面にわたるプロキシキャッシングの管理について説明します。

### 前提条件

この章は、「 *Get Started with Kong* 」シリーズの一部です。最適な効果を得るために、シリーズを最初からご覧になることをお勧めします。

まずは、導入として [Kong の入手](/gateway/latest/get-started/)から始めましょう。ここには、ローカル {{site.base_gateway}} を実行するための前提条件と手順の一覧が含まれています。

ガイドの手順2である[サービスとルート](/gateway/latest/get-started/services-and-routes/)
には、このシリーズで使われている模擬サービスのインストール方法が記載されています。

これらの手順をまだ完了していない場合は、続行する前に完了してください。

### グローバルプロキシキャッシュ

プラグインをグローバルにインストールすると、{{site.base_gateway}} への *すべての* プロキシリクエストがキャッシュされる可能性があります。

1. **プロキシキャッシュを有効にする** 

   Proxy Cache プラグインはデフォルトで {{site.base_gateway}} にインストールされ、Admin API のプラグインオブジェクトに `POST` リクエストを送信することで有効にできます。

   ```sh
   curl -i -X POST http://localhost:8001/plugins \
     --data "name=proxy-cache" \
     --data "config.request_method=GET" \
     --data "config.response_code=200" \
     --data "config.content_type=application/json" \
     --data "config.cache_ttl=30" \
     --data "config.strategy=memory"
   ```

   構成が成功した場合は、 `201` 応答コードを受け取ります。

   このAdmin APIリクエストは、`200`のレスポンスコードと`application/json`に *等しい* *レスポンス* `Content-Type`ヘッダーを持つすべての`GET`リクエストに対してProxy Cacheプラグインを設定しました。`cache_ttl`は、30 秒後に値をフラッシュするようにプラグインに指示しました。

   最後のオプション`config.strategy=memory`は、キャッシュされたレスポンスのバッキングデータストアを指定します。`strategy`に
   関する詳しい情報は、Proxy Cacheプラグイン用の
   [パラメータリファレンス](/hub/kong-inc/proxy-cache/)に記載されています。
2. **検証** 

   `GET` リクエストを送信して返されたヘッダーを調べることで、Proxy Cache プラグインが機能していることを確認できます。このガイドのステップ 2 の[サービスとルート](/gateway/latest/get-started/services-and-routes/)では、
   プロキシのキャッシングの動作を確認するのに役立つ `/mock` ルートとサービスを設定します。

   まず、 `/mock`ルートに最初のリクエストを行います。Proxy Cacheプラグインは、`X-Cache`というプレフィックスが付いたステータス情報ヘッダーを返すため、`grep`を使用してその情報をフィルタリングします。

       curl -i -s -XGET http://localhost:8000/mock/anything | grep X-Cache

   最初のリクエストでは、レスポンスがキャッシュされていないはずで、ヘッダーには`X-Cache-Status: Miss`と
   表されます。

       X-Cache-Key: c9e1d4c8e5fd8209a5969eb3b0e85bc6
       X-Cache-Status: Miss

   最初のリクエストから30秒以内に、コマンドを繰り返して同じリクエストを送信し、
   ヘッダーはキャッシュ `Hit` を示します。

       X-Cache-Key: c9e1d4c8e5fd8209a5969eb3b0e85bc6
       X-Cache-Status: Hit

   `X-Cache-Status`ヘッダは以下のキャッシュ結果を返します。

   |   状態   |                                              説明                                               |
   |--------|-----------------------------------------------------------------------------------------------|
   | Miss   | リクエストはキャッシュでは処理できましたが、リソースのエントリがキャッシュに見つからず、リクエストは上流に転送されました。                                 |
   | Hit    | リクエストはキャッシュをもとに正常に処理されました。                                                                    |
   | リフレッシュ | リソースはキャッシュ内に見つかりましたが、キャッシュコントロールの動作またはハードコードされた `cache_ttl` のしきい値に達したため、リクエストを満たすことができませんでした。 |
   | Bypass | プラグイン構成に基づいてキャッシュからリクエストを満たすことができませんでした。                                                      |

### エンティティレベルのプロキシキャッシング

{% navtabs %}
{% navtab Service-level %}

Proxy Cacheプラグインは特定のサービスに対して有効にできます。リクエストは上記と同じですが、サービスURLに送信されます。

```sh
curl -X POST http://localhost:8001/services/example_service/plugins \
   --data "name=proxy-cache" \
   --data "config.request_method=GET" \
   --data "config.response_code=200" \
   --data "config.content_type=application/json" \
   --data "config.cache_ttl=30" \
   --data "config.strategy=memory"
```

{% endnavtab %}
{% navtab Route-level %}

Proxy Cacheプラグインは特定のルートに対して有効にできます。リクエストは上記と同じですが、リクエストはルートURLに送信されます。

```sh
curl -X POST http://localhost:8001/routes/example_route/plugins \
   --data "name=proxy-cache" \
   --data "config.request_method=GET" \
   --data "config.response_code=200" \
   --data "config.content_type=application/json" \
   --data "config.cache_ttl=30" \
   --data "config.strategy=memory"
```

{% endnavtab %}
{% navtab Consumer-level %}

{{site.base_gateway}}では、[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)とはサービスのユーザーを定義する抽象的概念です。コンシューマレベルのプロキシのキャッシュを使用して、コンシューマごとに応答をキャッシュできます。

1. コンシューマを作成：

   ```sh
   curl -X POST http://localhost:8001/consumers/ \
   --data username=sasha
   ```

2. コンシューマのキャッシュを有効にします。

   ```sh
   curl -X POST http://localhost:8001/consumers/sasha/plugins \
      --data "name=proxy-cache" \
      --data "config.request_method=GET" \
      --data "config.response_code=200" \
      --data "config.content_type=application/json" \
      --data "config.cache_ttl=30" \
      --data "config.strategy=memory"
   ```

{% endnavtab %}
{% endnavtabs %}

キャッシュされたエンティティを管理
-----------------

Proxy Cache プラグインは、キャッシュされたエンティティを管理するための管理エンドポイントをサポートします。管理者は、キャッシュされたエンティティを表示および削除したり、Admin API にリクエストを送信してキャッシュ全体を消去したりできます。

キャッシュされたエンティティを取得するには、キャッシュされた既知の値`X-Cache-Key`を使用して、Admin API `/proxy-cache`エンドポイントにリクエストを送信します。このリクエストはTTLの有効期限が切れる前に提出する必要があり、そうでない場合は、キャッシュされたエンティティは消去されます。

たとえば、上記の応答ヘッダーを使用して、`c9e1d4c8e5fd8209a5969eb3b0e85bc6`の`X-Cache-Key`の値ををAdmin APIへ渡します。

```sh
curl -i http://localhost:8001/proxy-cache/c9e1d4c8e5fd8209a5969eb3b0e85bc6
```

`200 OK`を含む応答には、キャッシュされたエンティティの完全な詳細が含まれます。

Proxy Cache別Admin APIエンドポイントの完全なリストについては、「[Proxy Cacheプラグインドキュメント](/hub/kong-inc/proxy-cache/#admin-api)」を参照してください。

