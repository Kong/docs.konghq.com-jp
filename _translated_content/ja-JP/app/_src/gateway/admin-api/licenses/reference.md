---
title: "ライセンスリファレンス"
badge: "enterprise"
licenses_attribute_id: "属性名 | 説明\n---:| ---\n  id  | **ライセンスの**一意のID。\n"
licenses_body: "属性名 | 説明\n---:| ---\n  payload  | JSON形式の **Kong Gatewayライセンス**。\n"
---
{{site.base_gateway}}ライセンス機能は、
[Admin API](/gateway/{{page.release}}/admin-api/)から構成できます。この機能を使用すると、
{{site.base_gateway}}クラスタで、従来のデプロイメントとハイブリッドモードのデプロイメントの両方を使用してライセンスを構成できます。ハイブリッドモードのデプロイメントでは、コントロールプレーン（CP）は`/licenses`エンドポイントを介して構成されたライセンスをクラスタのすべてのデータプレーン（DP）に送信します。データプレーン（DP）は最新の`updated_at`ライセンスを使用します。

ライセンスを一覧表示する
------------

**エンドポイント** 
<div class="endpoint get">/licenses/</div> 

**応答** 

    HTTP 200 OK

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

{{site.base_gateway}}によって保存されたライセンスがない場合、データ配列は空になります。

```json
{
  "data": [],
  "next": null
}
```

特定のライセンスを一覧表示する
---------------

**エンドポイント** 
<div class="endpoint get">/licenses/\{id\}</div> 


{{ page.licenses_attribute_id }}

**応答** 

    HTTP 200 OK

```json
{
  "created_at": 1500508800,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-21\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"24cc21223633044c15c300be19cacc26ccc5aca0dd9a12df8a7324a1970fe304bc07b8dcd7fb08d7b92e04169313377ae3b550ead653b951bc44cd2eb59f6beb\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

ライセンスの追加
--------

自動生成された UUID を使用してライセンスを作成する。

**エンドポイント** 
<div class="endpoint post">/licenses/</div> 

**リクエストボディ** 


{{ page.licenses_body }}

`POST`を使用する場合、リクエストペイロードに有効な{{site.base_gateway}}ライセンスが **含まれている** 場合、ライセンスが追加されます。

リクエストペイロードに有効なライセンスが含まれて **いない** 場合は、`400 BAD REQUEST` が返されます。

**応答** 

    HTTP 201 Created

```json
{
  "created_at": 1500508800,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

ライセンスを更新または追加する
---------------

**エンドポイント** 
<div class="endpoint put">/licenses/\{id\}</div> 


{{ page.licenses_attribute_id }}

`PUT` を使うとき、リクエストペイロードが
エンティティの主キー（ライセンスは `id`）を **含まない** 場合、
ライセンスが追加され指定されたIDに割り当てられます。

リクエストペイロードにエンティティのプライマリキー（ライセンスの場合は`id`）が **含まれている** と、ライセンスは指定されたペイロード属性に置き換えられます。ID が有効なUUIDでない場合は、 `400 BAD REQUEST`が返されます。ID を省略すると、`405 NOT ALLOWED`が返されます。

**リクエストボディ** 


{{ page.licenses_body }}

**応答** 

    HTTP 200 OK

```json
{
  "created_at": 1500508800,
  "id": "e8201120-4ee3-43ca-9e92-3fed08b1a15d",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

ライセンスの更新
--------

**エンドポイント** 
<div class="endpoint patch">/licenses/\{id\}</div> 


{{ page.licenses_attribute_id }}

`PATCH`を使用する場合、リクエストペイロードにエンティティのプライマリキー（ライセンスの場合は `id`）が含まれて **いる** と、ライセンスは指定されたペイロード属性に置き換えられます。

リクエストペイロードに、エンティティのプライマリキー（ライセンスの`id`）が含まれて **いない** 場合、`404 NOT FOUND`が返されるか、リクエストペイロードに無効なライセンスが含まれている場合は、`400 BAD REQUEST`が返されます。

**リクエストボディ** 


{{ page.licenses_body }}

**応答** 

    HTTP 200 OK

```json
{
  "created_at": 1500595200,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-21\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"24cc21223633044c15c300be19cacc26ccc5aca0dd9a12df8a7324a1970fe304bc07b8dcd7fb08d7b92e04169313377ae3b550ead653b951bc44cd2eb59f6beb\",\"version\":\"1\"}}",
  "updated_at": 1500595200
}
```

ライセンスの削除
--------

**エンドポイント** 
<div class="endpoint delete">/licenses/\{id\}</div> 


{{ page.licenses_attribute_id }}

**応答** 

    HTTP 204 No Content

レポートの生成
-------

月間使用状況データを収集するには、 {{site.base_gateway}}インスタンスに関するレポートを生成します。
<div class="endpoint get">/license/report</div> 

レポートで使用できるフィールド：

|       フィールド        |                                                                                        説明                                                                                        |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `counters`         | 指定された月に行われたリクエストの数をカウントします。 <br><br>• `bucket`: リクエストが処理された年と月。`bucket` の値が `UNKNOWN` の場合、リクエストは {{site.base_gateway}} 2\.7\.0\.1 より前に処理されました。<br>\- `request_count`: 指定された月と年に処理されたリクエスト数。 |
| `db_version`       | {{site.base_gateway}}が使用するデータストアのタイプとバージョン。                                                                                                                                                     |
| `kong_version`     | {{site.base_gateway}}インスタンスのバージョン。                                                                                                                                                              |
| `license_key`      | 現在のライセンスキーの暗号化された識別子。ライセンスが存在しない場合は、フィールドには `UNLICENSED` と表示されます。                                                                                                                |
| `rbac_users`       | RBAC を通して登録されたユーザーの数。                                                                                                                                                            |
| `services_count`   | {{site.base_gateway}}インスタンスで構成されるサービスの数。                                                                                                                                                        |
| `system_info`      | {{site.base_gateway}}の実行システムに関する情報を表示。<br><br> • `cores`：ノードのCPUコア数<br> • `hostname`：暗号化されたシステムのホスト名<br> • `uname`：オペレーティングシステム                                                                 |
| `workspaces_count` | {{site.base_gateway}} インスタンスで構成されるワークスペースの数。                                                                                                                                                    |

**応答** 

    HTTP 200 OK

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

{{site.base_gateway}}によってライセンスが保存されていない場合、レポートには`"license_key": "UNLICENSED"`が含まれます。

    HTTP 200 OK

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

