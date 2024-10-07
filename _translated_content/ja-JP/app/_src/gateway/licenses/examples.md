---
title: "ライセンスの例"
badge: "enterprise"
---
{:.note}
> 
> **注：** `/licenses`エンドポイントは標準的なライセンス構成を上書きしません。

`/licenses` エンドポイントは、システムディレクトリに環境変数を使用したり、
プレーンテキストファイルを配置したりせずに {{site.base_gateway}} を
設定する方法を提供します。ハイブリッドモードのデプロイメントでは、Admin API
`/licenses` エンドポイントはクラスタ内のすべてのデータプレーンも設定するので、
設定プロセスがシンプルになります。

前提条件
----

従来のデプロイメント、コントロールプレーン（CP）、データプレーン（DP）からすべての標準ライセンス
構成を削除します。

環境変数またはプレーンテキストファイルを使用してライセンスをデプロイする場合、この構成は`/licenses`エンドポイントより優先され、Admin APIに変更を伝達 **しません** 。
ライセンスが別の方法で構成されている場合に`/licenses`エンドポイントを使用しようとすると、新しいライセンスは適用されません。

すべてのライセンスを一覧表示する
----------------

{:.note}
> 
> **注** ：Admin APIを使用してライセンスを一覧表示するには、最初にライセンスが同じ方法で追加されている必要があります。ライセンスがAdmin APIを使用して追加されなかった場合、応答は以下の`Response if there is no license`タブに表示されるものと同じになります。

次のリクエストを送信します。

```bash
curl -i -X GET http://localhost:8001/licenses
```

{% navtabs codeblock %}
{% navtab Response when license exists %}

```json
{
  "data": [
    {
      "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
      "created_at": 1500508800,
      "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
      "updated_at": 1500508800
    },
  ],
  "next": null,
}
```

{% endnavtab %}
{% navtab Response if there is no license %}

```json
{
  "data": [],
  "next": null
}
```

{% endnavtab %}
{% endnavtabs %}

ライセンスの一覧表示
----------

ライセンスのIDを使用して、次のリクエストを送信します。

```bash
curl -i -X GET http://localhost:8001/licenses/30b4edb7-0847-4f65-af90-efbed8b0161f
```

```json
{
  "created_at": 1500508800,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

ライセンスを追加する
----------

### 自動生成ID

ID を自動的に生成するには、 `POST`リクエストを`/licenses`に直接送信します。

```bash
curl -i -X POST http://localhost:8001/licenses \
  --data payload='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-20","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b","version":"1"}}'
```

応答：

```json
{
  "created_at": 1500508800,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

### 手動で提供された ID

カスタムIDを使用してライセンスを作成するには、`PUT`リクエストを`/licenses/<uuid id="sl-md0000000">`に送信します。

```bash
curl -i -X PUT http://localhost:8001/licenses/e8201120-4ee3-43ca-9e92-3fed08b1a15d \
  payload='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-20","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b","version":"1"}}'
```

応答：

```json
{
  "created_at": 1500508800,
  "id": "e8201120-4ee3-43ca-9e92-3fed08b1a15d",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

{:.note}
> 
> **注** : 指定された ID が存在する場合、リクエストは新しい `license` エンティティを作成する代わりに、指定された ID のライセンスの更新を実行します。

ライセンスの更新
--------

ライセンスを更新するには、既存のライセンス ID に `PATCH` リクエストを送信します。

```bash
curl -i -X PATCH http://localhost:8001/licenses/30b4edb7-0847-4f65-af90-efbed8b0161f \
  payload='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-21","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"24cc21223633044c15c300be19cacc26ccc5aca0dd9a12df8a7324a1970fe304bc07b8dcd7fb08d7b92e04169313377ae3b550ead653b951bc44cd2eb59f6beb","version":"1"}}'
```

応答：

```json
{
  "created_at": 1500595200,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-21\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"24cc21223633044c15c300be19cacc26ccc5aca0dd9a12df8a7324a1970fe304bc07b8dcd7fb08d7b92e04169313377ae3b550ead653b951bc44cd2eb59f6beb\",\"version\":\"1\"}}",
  "updated_at": 1500595200
}
```

ライセンスを更新した後は、必ず{{site.base_gateway}}ノードを[再起動](/gateway/{{page.release}}/reference/cli/#kong-restart)してください。

ライセンスレポートの生成
------------

レポートを生成するには、 `GET`リクエストを`/license/report`に直接送信します。

```bash
curl -i -X GET http://localhost:8001/license/report
```

{% navtabs codeblock %}
{% navtab Response when license exists %}

```json
{
    "counters": [
        {
            "bucket": "2021-12",
            "request_count": 0
        }
    ],
    "db_version": "postgres 9.6.19",
    "kong_version": "{{page.versions.ee}}",
    "license_key": "ASDASDASDASDASDASDASDASDASD_ASDASDA",
    "rbac_users": 0,
    "services_count": 0,
    "system_info": {
        "cores": 4,
        "hostname": "13b867agsa008",
        "uname": "Linux x86_64"
    },
    "workspaces_count": 1
}
```

{% endnavtab %}
{% navtab Response if there is no license %}

```json
{
    "counters": [
        {
            "bucket": "2021-12",
            "request_count": 0
        }
    ],
    "db_version": "postgres 9.6.19",
    "kong_version": "{{page.versions.ee}}",
    "license_key": "UNLICENSED",
    "rbac_users": 0,
    "services_count": 0,
    "system_info": {
        "cores": 4,
        "hostname": "13b867agsa008",
        "uname": "Linux x86_64"
    },
    "workspaces_count": 1
}
```

{% endnavtab %}
{% endnavtabs %}

