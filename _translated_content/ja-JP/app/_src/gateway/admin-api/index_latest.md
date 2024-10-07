---
title: "Kong Gateway Admin API"
---

{{site.base_gateway}}には、管理用の **内部** RESTful Admin APIが付属しています。
Admin APIへのリクエストはクラスタ内のどのノードにも送信でき、Kongは
すべてのノード間で構成の一貫性を維持します。

* `8001`は、Admin APIがリッスンするデフォルトのポートです。
* `8444`は、Admin API への HTTPS トラフィックのデフォルトポートです。

このAPIは内部使用向けに設計されており、Kongを完全に制御可能にします。そのためKong環境を設定する際には、このAPIが不当に公開されることがないよう、注意してください。セキュリティ対策に関する詳細については、「[Admin APIの保護](/gateway/{{page.release}}/production/running-kong/secure-admin-api/)」を参照してください。

ドキュメント
------

Kong Admin APIはOpenAPI形式で文書化されています。

|                          仕様                          |                                                                                                                                               Insomnia のリンク                                                                                                                                                |
|------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Enterprise API](/gateway/api/admin-ee/latest/){:target="_blank"} | <a href="https://insomnia.rest/run/?label=Kong%20Gateway%20Enterprise%203.4&uri=https%3A%2F%2Fraw.githubusercontent.com%2FKong%2Fdocs.konghq.com%2Fmain%2Fapi-specs%2FGateway-EE%2F3.4%2Fkong-ee-3.4.json" target="_blank"><img src="https://insomnia.rest/images/run.svg" alt="Run in Insomnia"></a>      |
| [オープンソース API](/gateway/api/admin-oss/latest/) {:target="_blank"}  | <a href="https://insomnia.rest/run/?label=Kong%20Gateway%20Open%20Source%203.4&uri=https%3A%2F%2Fraw.githubusercontent.com%2FKong%2Fdocs.konghq.com%2Fmain%2Fapi-specs%2FGateway-OSS%2F3.4%2Fkong-oss-3.4.json" target="_blank"><img src="https://insomnia.rest/images/run.svg" alt="Run in Insomnia"></a> |

個々のエンティティのドキュメントについては、次のリンクを参照してください。

{% navtabs %}
{% navtab Enterprise endpoints %}

|[インフォメーションルート](/gateway/api/admin-ee/latest/#/Information/get-endpoints){:target="_blank"} |[正常ルート](/gateway/api/admin-ee/latest/#/Information/get-status){:target="_blank"} |[タグ](/gateway/api/admin-ee/latest/#/tags/get-tags){:target="_blank"} |
|[デバッグルート](/gateway/api/admin-ee/latest/#/debug/put-debug-cluster-control-planes-nodes-log-level-log_level){:target="_blank"} |[サービス](/gateway/api/admin-ee/latest/#/Services/list-service){:target="_blank"} |[ルート](/gateway/api/admin-ee/latest/#/Routes/list-route){:target="_blank"} |
|[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer){:target="_blank"} |[プラグイン](/gateway/api/admin-ee/latest/#/Plugins/list-plugins-with-consumer){:target="_blank"} |[証明書](/gateway/api/admin-ee/latest/#/Certificates/list-certificate){:target="_blank"} |
| [CA 証明書](/gateway/api/admin-ee/latest/#/CA%20Certificates/list-ca_certificate){:target="_blank"} | [SNI](/gateway/api/admin-ee/latest/#/SNIs/list-sni-with-certificate) {:target="_blank"} |[上流](/gateway/api/admin-ee/latest/#/Upstreams/list-upstream){:target="_blank"} |
|[ターゲット](/gateway/api/admin-ee/latest/#/Targets/list-target-with-upstream){:target="_blank"} |[金庫](/gateway/api/admin-ee/latest/#/Vaults/list-vault){:target="_blank"} |[キー](/gateway/api/admin-ee/latest/#/Keys/list-key){:target="_blank"} |
|[フィルターチェーン](/gateway/api/admin-ee/latest/#/filter-chains/get-filter-chains){:target="_blank"} |[ライセンス](/gateway/api/admin-ee/latest/#/licenses/get-licenses){:target="_blank"} |[ワークスペース](/gateway/api/admin-ee/latest/#/Workspaces/list-workspace){:target="_blank"} |
| [RBAC](/gateway/api/admin-ee/latest/#/rbac/get-rbac-users) {:target="_blank"} |[管理者](/gateway/api/admin-ee/latest/#/admins/get-admins){:target="_blank"} |[コンシューマグループ](/gateway/api/admin-ee/latest/#/consumer_groups/){:target="_blank"} |
|[イベントフック](/gateway/api/admin-ee/latest/#/Event-hooks/get-event-hooks){:target="_blank"} |[キーリングとデータ暗号化](/gateway/api/admin-ee/latest/#/Keyring/get-keyring){:target="_blank"} |[監査ログ](/gateway/api/admin-ee/latest/#/audit-logs/get-audit-requests){:target="_blank"} |
{% endnavtab %}
{% navtab OSS endpoints %}
|[インフォメーションルート](/gateway/api/admin-oss/latest/#/Information/get-endpoints){:target="_blank"} |[正常ルート](/gateway/api/admin-oss/latest/#/Information/get-status){:target="_blank"} |[タグ](/gateway/api/admin-oss/latest/#/tags/get-tags){:target="_blank"} |
|[デバッグルート](/gateway/api/admin-oss/latest/#/debug/put-debug-cluster-control-planes-nodes-log-level-log_level){:target="_blank"} |[サービス](/gateway/api/admin-oss/latest/#/Services/list-service){:target="_blank"} |[ルート](/gateway/api/admin-oss/latest/#/Routes/list-route){:target="_blank"} |
|[コンシューマ](/gateway/api/admin-oss/latest/#/Consumers/list-consumer){:target="_blank"} |[プラグイン](/gateway/api/admin-oss/latest/#/Plugins/list-plugins-with-consumer){:target="_blank"} |[証明書](/gateway/api/admin-oss/latest/#/Certificates/list-certificate){:target="_blank"} |
| [CA 証明書](/gateway/api/admin-oss/latest/#/CA%20Certificates/list-ca_certificate){:target="_blank"} | [SNI](/gateway/api/admin-oss/latest/#/SNIs/list-sni-with-certificate) {:target="_blank"} |[上流](/gateway/api/admin-oss/latest/#/Upstreams/list-upstream){:target="_blank"} |
|[ターゲット](/gateway/api/admin-oss/latest/#/Targets/list-target-with-upstream){:target="_blank"} |[Vault](/gateway/api/admin-oss/latest/#/Vaults/list-vault){:target="_blank"} |[キー](/gateway/api/admin-oss/latest/#/Keys/list-key){:target="_blank"} |
|[フィルターチェーン](/gateway/api/admin-oss/latest/#/filter-chains/get-filter-chains){:target="_blank"} | | |
{% endnavtab %}
{% endnavtabs %}

DB レスモード
--------

[DBレスモード](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config/)では、[{{site.base_gateway}}を宣言的に構成](/gateway/{{page.release}}/admin-api/declarative-configuration/)します。
各KongノードのAdmin APIは独立して機能し、特定のKongノードのメモリの状態を反映します。
これは、Kongノード間のデータベース連携がないためです。
したがって、Admin APIのほとんどは読み取り専用です。

{{site.base_gateway}}をDBレスモードで実行する場合、Admin APIは宣言型構成の処理に関連するタスクのみを実行できます。

* [Validating configurations against schemas](/gateway/api/admin-oss/latest/#/Information/post-schemas-entity-validate)
* [スキーマに対するプラグイン設定の検証](/gateway/api/admin-oss/latest/#/Information/post-schemas-plugins-validate)
* [宣言型構成の再読み込み](/gateway/{{page.release}}/admin-api/declarative-configuration/)

サポートされているコンテンツタイプ
-----------------

The Admin API accepts 3 content types on every endpoint:

### application/json

`application/json`コンテンツタイプは、複雑な本文（複雑なプラグイン構成など）に役立ちます。送信するデータのJSON表現を送信します。以下はその例です。

```json
{
    "config": {
        "limit": 10,
        "period": "seconds"
    }
}
```

`test-service`という名前のサービスにルートを追加する例を次に示します。

    curl -i -X POST http://localhost:8001/services/test-service/routes \
         -H "Content-Type: application/json" \
         -d '{"name": "test-route", "paths": [ "/path/one", "/path/two" ]}'

### application/x\-www\-form\-urlencoded

コンテンツタイプ「`application/x-www-form-urlencoded`」は、基本的なリクエストボディに便利なもので、
ほとんどの状況で使用できます。
ネストされた値を送信する場合、Kongはネストされたオブジェクトがドット付きのキーを使って参照されることを想定しています。以下に例を示します。

    config.limit=10&config.period=seconds

配列を指定するときは、値を順番に送るか、角括弧（括弧の中はオプションですが、
指定する場合は1インデックスで、連続している必要があります）
を使用してください。

Here's an example route added to a service named `test-service`:

```sh
curl -i -X POST http://localhost:8001/services/test-service/routes \
     -d "name=test-route" \
     -d "paths[1]=/path/one" \
     -d "paths[2]=/path/two"
```

次の2つの例は上記のものと同じですが、明示的ではありません：

```sh
curl -i -X POST http://localhost:8001/services/test-service/routes \
     -d "name=test-route" \
     -d "paths[]=/path/one" \
     -d "paths[]=/path/two"

curl -i -X POST http://localhost:8001/services/test-service/routes \
    -d "name=test-route" \
    -d "paths=/path/one" \
    -d "paths=/path/two"
```

### multipart/form\-data

`multipart/form-data`コンテンツタイプはURLエンコードに似ています。このコンテンツタイプでは、ドットキーを使用してネストされたオブジェクトを参照します。以下は、LuaファイルをプリファンクションKongプラグインに送信する例です。

```sh
curl -i -X POST http://localhost:8001/services/plugin-testing/plugins \
     -F "name=pre-function" \
     -F "config.access=@custom-auth.lua"
```

このコンテンツタイプの配列を指定する場合は、配列インデックスを指定する必要があります。たとえば、以下は`test-service`という名前のサービスに追加されたルートの例です。

```sh
curl -i -X POST http://localhost:8001/services/test-service/routes \
     -F "name=test-route" \
     -F "paths[1]=/path/one" \
     -F "paths[2]=/path/two"
```

HTTPステータス応答コード
--------------

HTTPレスポンスでは、以下のステータスコードが返されます。

| HTTPコード |   HTTPの説明    |                                                                                 注記                                                                                 |             Request method             |
|---------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|
| 200     | OK           | リクエストは成功しました。`200`リクエストの結果は、リクエストの種類によって異なります。<br> \- `GET`：リソースが取得され、メッセージ本文で送信されました。<br> \- `PUT`または`POST`：アクションの結果を説明するリソースがメッセージ本文で送信されます。<br> \- `PATCH`：？ | `GET`, `POST`, `PATCH`, `PUT`          |
| 201     | Created      | リクエストが成功し、新しいリソースが作成されました。                                                                                                                                         | `POST`                                 |
| 204     | No Content   | 送信する要求にコンテンツがありません。                                                                                                                                                | `DELETE`                               |
| 400     | Bad Request  | クライアントのエラーのため、サーバーはリクエストを送信できないか、送信しません。                                                                                                                           | `POST`、`PATCH`、`PUT`                   |
| 401     | Unauthorized | クライアントは認証されていません。                                                                                                                                                  | `GET`, `POST`, `DELETE`, `PATCH`,`PUT` |
| 404     | 見つかりません      | The server can't find the resource you requested. With an API, this can mean that the endpoint is valid but the resource doesn't exist.                            | `GET`、`PATCH`、`PUT`                    |
| 405     | 許可されていないメソッド | サーバーはリクエストメソッドを認識していますが、リソースではサポートされていません。                                                                                                                         | `PUT`                                  |
| 409     | Conflict     | リクエストがサーバーの現在の状態と競合しています。                                                                                                                                          | `POST`                                 |

ワークスペースでのAPIの使用
---------------
{:.badge .enterprise}

{% include_cached /md/gateway/admin-api-workspaces.md %}

