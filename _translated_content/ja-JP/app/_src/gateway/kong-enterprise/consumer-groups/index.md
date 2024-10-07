---
title: "コンシューマグループ"
badge: "enterprise"
---
コンシューマグループを使用すると、APIエコシステム内のコンシューマ（ユーザーまたはアプリケーション）を組織化して分類できます。コンシューマをグループ化することで、個別に管理する必要がなくなり、スケーラブルで効率的な方法で構成を管理できます。

{% if_version gte:3.4.x %}

コンシューマグループでは、プラグインを特定のコンシューマグループに絞り込むことができ、個々のコンシューマグループごとに新しいプラグインインスタンスが作成されるので、設定やカスタマイズがより柔軟で便利になります。

ユースケース
------

* 権限の管理: コンシューマグループを使用して、さまざまなレベルの権限を持つさまざまなユーザーを定義できます。たとえば、通常のユーザー、プレミアムユーザー、および管理者ごとに個別のコンシューマグループを作成できます。

* ロールの管理：組織内には、異なる方法でAPIを使用するさまざまな部門やチームが存在する場合があります。これらのさまざまなロールのコンシューマグループを作成することで、APIの使用エクスペリエンスをカスタマイズできます。たとえば、組織では、マーケティングチーム、開発チーム、サポートチームごとに個別のコンシューマグループを持つことができます。

* リソースクォータと流量制限：コンシューマグループを使用して、さまざまなコンシューマにリソースクォータとレート制限を適用できます。たとえば、サブスクリプションプランに基づいて、さまざまなコンシューマグループに異なるレート制限を適用できます。

* プラグイン構成のカスタマイズ：プラグインのスコープを定義済みグループに限定できるため、異なるコンシューマグループは、要件に基づいて異なるプラグイン構成を持つことができます。たとえば、あるグループでは追加のリクエスト変換が必要になる一方で、別のグループではまったく必要のない場合があります。

{:.note}
> 
> **注** : コンシューマグループ プラグインのスコーピングは {{site.base_gateway}} バージョン 3\.4 で追加された機能です。異なるバージョンが混在する {{site.base_gateway}} クラスタ（3\.4 コントロールプレーンで 3\.3 以上のデータプレーン）の実行は、コンシューマグループ スコープのプラグインを使用している場合、サポートされません。

プラグインの絞り込み
----------

次のプラグインをコンシューマグループにスコープできます。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)
* [Request Transformer Advanced](/hub/kong-inc/request-transformer-advanced/)
* [Response transformer Advanced](/hub/kong-inc/request-transformer-advanced/)
* [Request Transformer](/hub/kong-inc/request-transformer)
* [Response Transformer](/hub/kong-inc/response-transformer)

{:.note}
> 
> **注** : コンシューマグループ プラグインのスコーピングは {{site.base_gateway}} バージョン 3\.4 で追加された機能です。異なるバージョンが混在する {{site.base_gateway}} クラスタ（3\.4 コントロールプレーンで 3\.3 以上のデータプレーン）の実行は、コンシューマグループ スコープのプラグインを使用している場合、サポートされません。

{% endif_version %}

{% if_version lte:3.3.x %}
コンシューマグループを使用すると、必要な数の流量制限ティアを定義して、コンシューマのサブセットに適用できるため、コンシューマを個別に管理する必要がありません。

たとえば、次の3つのコンシューマグループを定義できます。

* 毎分1000リクエストの「ゴールドティア」
* 毎秒 10 リクエストの"シルバーティア"
* 毎秒6リクエストの「ブロンズティア」

`consumer_groups` エンドポイントは、[Rate Limiting Advancedプラグイン](/hub/kong-inc/rate-limiting-advanced/)と連携します。

コンシューマグループに属していないコンシューマは、デフォルトでRate Limiting Advancedプラグインの設定に従うため、一部のユーザーの階層グループを定義し、グループに属さないコンシューマに対してはデフォルトの動作を設定できます。

コンシューマグループを使用して流量制限するには、以下を行う必要があります。

* コンシューマグループを1つ以上作成する
* コンシューマを作成
* コンシューマをグループに割り当てる
* `enforce_consumer_groups` と`consumer_groups` パラメーターを使用してRate Limiting Advancedプラグインを設定し、プラグインが受け入れるコンシューマグループのリストを設定します。
* 各コンシューマグループにレート制限を設定し、プラグインの 構成を上書きします。

すべてのリクエストについては、
[コンシューマグループのリファレンス](/gateway/{{page.release}}/admin-api/consumer-groups/reference)を参照してください。

コンシューマグループの設定
-------------

1. `Gold`という名前のコンシューマグループを作成します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumer_groups \
   --data name=Gold
   ```

   レスポンス: 

   ```json
   {
       "created_at": 1638915521,
       "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4",
       "name": "Gold",
       "tags": null
   }
   ```

2. コンシューマ`Amal`を作成します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumers \
   --data username=Amal
   ```

   レスポンス: 

   ```json
   {
       "created_at": 1638915577,
       "custom_id": null,
       "id": "8089a0e6-1d31-4e00-bf51-5b902899b4cb",
       "tags": null,
       "type": 0,
       "username": "Amal",
       "username_lower": "amal"
   }
   ```

3. `Amal`を `Gold`コンシューマグループに追加します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumer_groups/Gold/consumers \
   --data consumer=Amal
   ```

   レスポンス: 

   ```json
   {
     "consumer_group": {
     "created_at": 1638915521,
     "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4",
     "name": "Gold",
     "tags": null
     },
     "consumers": [
       {
           "created_at": 1638915577,
           "id": "8089a0e6-1d31-4e00-bf51-5b902899b4cb",
           "type": 0,
           "username": "Amal",
           "username_lower": "amal"
       }
     ]
   }
   ```

コンシューマグループのRate Limiting Advanced構成を設定する
----------------------------------------

1. [流量制限高度プラグイン](/hub/kong-inc/rate-limiting-advanced/)を有効にし、流量制限を30秒につき（`config.window_size`）5件のリクエスト（`config.limit`）に制限します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/plugins/  \
   --data name=rate-limiting-advanced \
   --data config.limit=5 \
   --data config.window_size=30 \
   --data config.window_type=sliding \
   --data config.retry_after_jitter_max=0 \
   --data config.enforce_consumer_groups=true \
   --data config.consumer_groups=Gold
   ```

   コンシューマグループには以下のパラメータが必要です。
   * `config.enforce_consumer_groups=true`：このプラグインのコンシューマグループを有効にします。
   * `config.consumer_groups=Gold`：このプラグインが上書きを許可するグループのリストを指定します。

   {:.note}
   > 
   > **注：** この例では、プラグインをグローバルに設定しているので、これは
    {{site.base_gateway}}インスタンス内のすべてのエンティティ（サービス、ルート、コンシューマ）に適用されます。また、よりきめ細かな制御を行うために[特定のサービスやルート](/hub/kong-inc/rate-limiting-advanced/)にも適用できます。
2. 設定したプラグインは、クラスタ内のすべてのコンシューマに適用されます。`Gold` コンシューマグループに対してのみ流量制限構成を変更して、10 秒ごとに 10 件のリクエストに設定します。

   ```bash
   curl -i -X PUT http://{HOSTNAME}:8001/consumer_groups/Gold/overrides/plugins/rate-limiting-advanced \
   --data config.limit=10 \
   --data config.window_size=10 \
   --data config.retry_after_jitter_max=1
   ```

   レスポンス: 

   ```json
   {
     "config": {
         "limit": [
             10
         ],
         "retry_after_jitter_max": 1,
         "window_size": [
             10
         ]
     },
     "consumer_group": "Gold",
     "plugin": "rate-limiting-advanced"
   }
   ```

3. `Gold` コンシューマグループ オブジェクトを参照して動作していることを確認します。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumer_groups/Gold
   ```

   レスポンスの `plugins` オブジェクトと、Rate Limiting Advanced プラグインに指定したパラメーター
   に注目してください。

   ```json
   {
       "consumer_group": {
           "created_at": 1638915521,
           "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4",
           "name": "Gold",
           "tags": null
       },
       "consumers": [
           {
               "created_at": 1638915577,
               "id": "8089a0e6-1d31-4e00-bf51-5b902899b4cb",
               "type": 0,
               "username": "Amal",
               "username_lower": "amal"
           }
       ],
       "plugins": [
           {
               "config": {
                   "limit": [
                       10
                   ],
                   "retry_after_jitter_max": 1,
                   "window_size": [
                       10
                   ],
                   "window_type": "sliding"
               },
               "consumer_group": {
                   "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4"
               },
               "created_at": 1638916518,
               "id": "b7c426a2-6fcc-4bfd-9b7a-b66e8f1ad260",
               "name": "rate-limiting-advanced"
           }
       ]
   }
   ```

グループからコンシューマを削除する \- グループビュー
-----------------------------

グループからコンシューマを削除するには、`/consumers` または
`/consumer_groups`にアクセスします。次の手順では`/consumer_groups`を使用します。

1. コンシューマー名の`Gold`コンシューマグループを確認します。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumer_groups/Gold
   ```

   レスポンス: 

   ```json
   {
       "consumer_group": {
           "created_at": 1638915521,
           "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4",
           "name": "Gold",
           "tags": null
       },
       "consumers": [
           {
               "created_at": 1638915577,
               "id": "8089a0e6-1d31-4e00-bf51-5b902899b4cb",
               "type": 0,
               "username": "Amal",
               "username_lower": "amal"
           }
       ]
     }
   ```

2. コンシューマのユーザー名またはID（この例では`Amal`）を使用して、グループからコンシューマを削除します。

   ```bash
   curl -i -X DELETE http://{HOSTNAME}:8001/consumer_groups/Gold/consumers/Amal
   ```

   成功すると、次のレスポンスを受け取ります。

       HTTP/1.1 204 No Content

3. 確認するには、コンシューマグループの構成をチェックします。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumer_groups/Gold
   ```

   コンシューマが割り当てられていない場合の応答。

   ```json
   {
       "consumer_group": {
           "created_at": 1638917780,
           "id": "be4bcfca-b1df-4fac-83cc-5cf6774bf48e",
           "name": "Gold",
           "tags": null
       }
   }
   ```

グループからコンシューマを削除 \- コンシューマビュー
-----------------------------

グループからコンシューマを削除するには、`/consumers` または
`/consumer_groups`にアクセスします。次の手順では`/consumers`を使用します。

1. コンシューマグループの名前ではなくコンシューマの名前がわかっている場合は、
   コンシューマからグループを調べることができます。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumers/Amal/consumer_groups
   ```

   レスポンス: 

   ```json
   {
       "consumer_groups": [
           {
               "created_at": 1638915521,
               "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4",
               "name": "Gold",
               "tags": null
           }
       ]
   }
   ```

2. グループのユーザー名または ID（この例では `Gold`）を使用して、グループからコンシューマを削除します。

   ```bash
   curl -i -X DELETE http://{HOSTNAME}:8001/consumers/Amal/consumer_groups/Gold
   ```

   成功すると、次のレスポンスを受け取ります。

       HTTP/1.1 204 No Content

3. 検証するには、コンシューマ オブジェクトの構成を確認します。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumer_groups/Gold
   ```

   コンシューマが割り当てられていない場合の応答。

   ```json
   {
       "consumer_group": {
           "created_at": 1638917780,
           "id": "be4bcfca-b1df-4fac-83cc-5cf6774bf48e",
           "name": "Gold",
           "tags": null
       }
   }
   ```

コンシューマグループ構成の削除
---------------

コンシューマグループ自体を削除せずに、コンシューマグループの構成をクリアすることもできます。

この方法では、グループ内のコンシューマは削除されず、コンシューマグループ内に残ります。

1. 次のリクエストを使用してコンシューマグループ構成を削除します。

   ```bash
   curl -i -X DELETE http://{HOSTNAME}:8001/consumer_groups/Gold/overrides/plugins/rate-limiting-advanced
   ```

   成功すると、次のレスポンスが表示されます。

       HTTP/1.1 204 No Content

2. 検証するには、コンシューマ オブジェクトの構成を確認します。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumer_groups/Gold
   ```

   `plugins` オブジェクトのないレスポンス:

   ```json
   {
       "consumer_group": {
           "created_at": 1638917780,
           "id": "be4bcfca-b1df-4fac-83cc-5cf6774bf48e",
           "name": "Gold",
           "tags": null
       }
   }
   ```

コンシューマグループを削除
-------------

コンシューマグループが不要になった場合は、削除できます。これにより、グループからすべてのコンシューマを削除し、グループ自体を削除します。グループのコンシューマは削除されません。

1. 次のリクエストを使用してコンシューマグループを削除します。

   ```bash
   curl -i -X DELETE http://{HOSTNAME}:8001/consumer_groups/Gold
   ```

   成功すると、次のレスポンスが表示されます。

       HTTP/1.1 204 No Content

2. コンシューマグループのリストをチェックして、 `Gold`グループが削除されたことを確認します。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumer_groups
   ```

   レスポンス: 

   ```json
   {
   "data": [],
   "next": null
   }
   ```

3. グループに参加していたコンシューマをチェックして、まだ存在していることを確認してください。

   ```bash
   curl -i -X GET http://{HOSTNAME}:8001/consumers/Amal
   ```

   `HTTP/1.1 200 OK`の応答は、そのコンシューマが存在することを意味します。

複数のコンシューマの管理
------------

大量の `/consumer_groups` 操作を一括で実行できます。

1. 前のセクションでグループ `Gold` を削除したと仮定して、`Speedsters` という名前の別のグループと一緒に再度作成します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumer_groups \
   --data name=Gold
   
   curl -i -X POST http://{HOSTNAME}:8001/consumer_groups \
   --data name=Speedsters
   ```

2. `BarryAllen` と `WallyWest` の2つのコンシューマを作成します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumers \
   --data username=BarryAllen
   
   curl -i -X POST http://{HOSTNAME}:8001/consumers \
   --data username=WallyWest
   ```

3. 両方のコンシューマを`Speedsters`グループに追加します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumer_groups/Speedsters/consumers \
   --data consumer=BarryAllen \
   --data consumer=WallyWest
   ```

   
    {{site.base_gateway}}は、コンシューマグループに割り当てる前に、提供されたコンシューマのリストを検証します。検証に合格するには、コンシューマが存在し、すでにグループに存在していない必要があります。

   いずれかのコンシューマが検証に失敗した場合、グループにはコンシューマは割り当てられません。

   すべてが成功した場合のレスポンスは下のとおりです。

   ```json
   {
       "consumer_group": {
           "created_at": 1639432267,
           "id": "a905151a-5767-40e8-804e-50eec4d0235b",
           "name": "Speedsters",
           "tags": null
       },
       "consumers": [
           {
               "created_at": 1639432286,
               "id": "ea904e1d-1f0d-4d5a-8391-cae60cb21d61",
               "type": 0,
               "username": "BarryAllen",
               "username_lower": "barryallen"
           },
           {
               "created_at": 1639432288,
               "id": "065d8249-6fe6-4d80-a0ae-f159caef7af0",
               "type": 0,
               "username": "WallyWest",
               "username_lower": "wallywest"
           }
       ]
   }
   ```

4. 1回のリクエストでグループからすべてのコンシューマを消去できます。これは、新しいユーザーのグループを循環させる必要がある場合に
   便利です。

   たとえば、`Speedsters` グループからすべてのコンシューマを削除します。

   ```bash
   curl -i -X DELETE http://{HOSTNAME}:8001/consumer_groups/Speedsters/consumers
   ```

   レスポンス: 

       HTTP/1.1 204 No Content

5. また、コンシューマを複数のグループに追加することもできます。

   * すべてのグループが Rate Limiting Advanced プラグインで許可されている場合は、最初のグループの設定だけが適用されます。
   * それ以外の場合は、Rate Limiting Advancedプラグインで指定されたグループがアクティブになります。

   `BarryAllen` を `Gold` と `Speedsters` の 2 つのグループに追加します。

   ```bash
   curl -i -X POST http://{HOSTNAME}:8001/consumers/BarryAllen/consumer_groups \
   --data group=Gold \
   --data group=Speedsters
   ```

   応答は次のようになります。

   ```json
   {
     "consumer": {
         "created_at": 1639436091,
         "custom_id": null,
         "id": "6098d577-6741-4cf8-9c86-e68057b8f970",
         "tags": null,
         "type": 0,
         "username": "BarryAllen",
         "username_lower": "barryallen"
     },
     "consumer_groups": [
         {
             "created_at": 1639432267,
             "id": "a905151a-5767-40e8-804e-50eec4d0235b",
             "name": "Gold",
             "tags": null
         },
         {
             "created_at": 1639436107,
             "id": "2fd2bdd6-690c-4e49-8296-31f55015496d",
             "name": "Speedsters",
             "tags": null
         }
     ]
   }
   ```

6. なお、コンシューマをすべてのグループから削除することもできます。

   ```bash
   curl -i -X DELETE http://{HOSTNAME}:8001/consumers/BarryAllen/consumer_groups
   ```

   レスポンス: 

       HTTP/1.1 204 No Content

{% endif_version %}

その他の情報
------

* [コンシューマグループの優先順位情報](/gateway/latest/key-concepts/plugins/#precedence)
* [APIの文書化](/gateway/api/admin-ee/3.3.0.x/#/consumer_groups/get-consumer_groups)
* [Rate Limiting Advanced プラグインを使用して複数の流量制限層を適用する](/hub/kong-inc/rate-limiting-advanced/how-to/)

