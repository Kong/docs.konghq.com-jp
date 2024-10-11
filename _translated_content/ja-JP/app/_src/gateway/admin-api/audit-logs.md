---
title: "監査ログ"
badge: "enterprise"
---
Admin APIを介してリクエストおよびデータベース監査ログにアクセスできます。

使用例については、 [{{site.base_gateway}}の監査ログ](/gateway/{{page.release}}/kong-enterprise/audit-log/)を参照してください。

監査ログの一覧表示
---------

### リクエスト監査ログ一覧

**エンドポイント** 
<div class="endpoint get">/audit/requests</div> 

**レスポンス** 

    HTTP 200 OK

RBACを有効にせずに`/status`エンドポイントをチェックするために生成されたレスポンスの例：

{% if_version lte:3.1.x %}

```json
{
    "data": [
        {
            "client_ip": "127.0.0.1",
            "method": "GET",
            "path": "/status",
            "payload": null,
            "rbac_user_id": null,
            "removed_from_payload": null,
            "request_id": "OjOcUBvt6q6XJlX3dd6BSpy1uUkTyctC",
            "request_timestamp": 1676424547,
            "signature": null,
            "status": 200,
            "ttl": 2591997,
            "workspace": "1065b6d6-219f-4002-b3e9-334fc3eff46c"
        }
    ],
    "total": 1
}
```

{% endif_version %}

{% if_version gte:3.2.x %}

```json
{
    "data": [
        {
            "client_ip": "127.0.0.1",
            "method": "GET",
            "path": "/status",
            "payload": null,
            "rbac_user_id": null,
            "rbac_user_name": null,
            "removed_from_payload": null,
            "request_id": "OjOcUBvt6q6XJlX3dd6BSpy1uUkTyctC",
            "request_source": null,
            "request_timestamp": 1676424547,
            "signature": null,
            "status": 200,
            "ttl": 2591997,
            "workspace": "1065b6d6-219f-4002-b3e9-334fc3eff46c"
        }
    ],
    "total": 1
}
```

{% endif_version %}

### データベース監査ログの一覧表示

**エンドポイント** 
<div class="endpoint get">/audit/objects</div> 

**レスポンス** 

    HTTP 200 OK

コンシューマ作成ログエントリの応答の例:

```json
{
    "data": [
        {
            "dao_name": "consumers",
            "entity": "{\"created_at\":1542131418000,\"id\":\"16787ed7-d805-434a-9cec-5e5a3e5c9e4f\",\"username\":\"bob\",\"type\":0}",
            "entity_key": "16787ed7-d805-434a-9cec-5e5a3e5c9e4f",
            "expire": 1544723418009,
            "id": "7ebabee7-2b09-445d-bc1f-2092c4ddc4be",
            "operation": "create",
            "request_id": "59fpTWlpUtHJ0qnAWBzQRHRDv7i5DwK2"
        },
  ],
  "total": 1
}
```

