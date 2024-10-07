---
title: "Kong Gatewayの監査ログ"
badge: "enterprise"
---

{{site.base_gateway}} は、Admin API を通じて詳細なログ機能を提供します。これにより、クラスタ管理者は、クラスタ構成に加えられた変更をその存続期間を通じて詳細に追跡できるため、コンプライアンスに取り組むことができ、フォレンジック調査中に貴重なデータポイントを提供できます。生成された監査ログ証跡は[ワークスペース](/gateway/api/admin-ee/latest/#/Workspaces)および[RBAC](/gateway/api/admin-ee/latest/#/rbac)に対応しているため、Kongのオペレータはクラスタ内で発生した変更を詳細かつ広範に調べることができます。

監査ログを有効にする
----------

監査ログはデフォルトで無効になっています。{{site.base_gateway}}で`kong.conf`構成を使用して構成します。

```bash
audit_log = on # audit logging is enabled
audit_log = off # audit logging is disabled
```

または環境変数で以下を設定します。

```bash
export KONG_AUDIT_LOG=on
export KONG_AUDIT_LOG=off
```

他の Kong 構成と同様に、変更は`kong reload`または`kong restart`で有効になります。

監査をリクエスト
--------

### 監査ログの生成と表示

監査ログには、KongのAdmin APIによって処理された各HTTPリクエストの詳細が表示されます。監査ログデータはKongのデータベースに書き込まれます。その結果、
リクエスト監査ログは、Admin API（ダイレクトベースクエリ経由のほか）から入手できます。

たとえば、`/status`エンドポイントへのAdmin APIクエリを考えてみましょう。

```sh
curl -i -X GET http://localhost:8001/status
```

次のレスポンスを受け取ります。

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Tue, 13 Nov 2018 17:32:47 GMT
Server: kong/ kong/{{page.versions.ee}}-enterprise-edition
Transfer-Encoding: chunked
X-Kong-Admin-Request-ID: ZuUfPfnxNn7D2OTU6Xi4zCnQkavzMUNM

{
    "database": {
        "reachable": true
    },
    
    "memory": {
        "lua_shared_dicts": {
            ...
        },
         "workers_lua_vms": [
            ...
         ]
    }

    "server": {
        "connections_accepted": 1,
        "connections_active": 1,
        "connections_handled": 1,
        "connections_reading": 0,
        "connections_waiting": 0,
        "connections_writing": 1,
        "total_requests": 1
    }
}
```

上記のAdmin APIとのやり取りにより、関連するエントリが監査ログテーブルに生成されます。
Admin APIを使用して監査ログをクエリすると、前のやり取りの詳細が返されます。

```sh
curl -i -X GET http://localhost:8001/audit/requests
```

{% if_version lte:3.1.x %}

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Tue, 13 Nov 2018 17:35:24 GMT
Server: kong/{{page.versions.ee}}-enterprise-edition
Transfer-Encoding: chunked
X-Kong-Admin-Request-ID: VXgMG1Y3rZKbjrzVYlSdLNPw8asVwhET

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

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Tue, 13 Nov 2018 17:35:24 GMT
Server: kong/{{page.versions.ee}}-enterprise-edition
Transfer-Encoding: chunked
X-Kong-Admin-Request-ID: VXgMG1Y3rZKbjrzVYlSdLNPw8asVwhET

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

`request_id` フィールドの値に注意してください。これは、
最初のトランザクションで受信した`X-Kong-Admin-Request-ID`応答ヘッダーと紐づいています。
これにより、Kongクラスタ内でクライアントのリクエストと監査ログレコードを密接に関連付けることができます
。

すべての監査ログエントリはKongのAdmin API経由で利用可能になるため、監査ログエントリを複製および検査するために、既存のログウェアハウス、SIEMソリューション、またはその他のリモートサービスに転送することは可能です。

### ワークスペースとRBAC

監査ログエントリは、リクエストされたワークスペースとRABCユーザー（存在する場合）を認識して書き込まれます。
RBACが強制されると、RBACユーザーのUUIDは監査ログエントリの`rbac_user_id`フィールドに書き込まれ、ユーザー名は`rbac_user_name`フィールドに書き込まれます。

{% if_version lte:3.1.x %}

```json
{
    "data": [
        {
            "client_ip": "127.0.0.1",
            "method": "GET",
            "path": "/status",
            "payload": null,
            "rbac_user_id": "2e959b45-0053-41cc-9c2c-5458d0964331",
            "request_id": "QUtUa3RMbRLxomqcL68ilOjjl68h56xr",
            "request_timestamp": 1581617463,
            "signature": null,
            "status": 200,
            "ttl": 2591995,
            "workspace": "0da4afe7-44ad-4e81-a953-5d2923ce68ae"
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
            "rbac_user_id": "2e959b45-0053-41cc-9c2c-5458d0964331",
            "rbac_user_name": "admin",
            "request_id": "QUtUa3RMbRLxomqcL68ilOjjl68h56xr",
            "request_source": "kong-manager",
            "request_timestamp": 1581617463,
            "signature": null,
            "status": 200,
            "ttl": 2591995,
            "workspace": "0da4afe7-44ad-4e81-a953-5d2923ce68ae"
        }
    ],
    "total": 1
}
```

{% endif_version %}

`workspace` フィールドが存在することに注意してください。これは、リクエストが関連付けられているワークスペースの UUID です。

{% if_version gte:3.2.x %}

### Kong Manager認証

Kong Managerでのログインとログアウトイベントは、`path`、`method`、`request_source`の監査ログフィールドから追跡できます。

たとえば、次の監査ログエントリを確認します。

```json
{
    "data": [
        {
            "client_ip": "127.0.0.1",
            "method": "GET",
            "path": "/auth",
            "payload": null,
            "rbac_user_id": "2e959b45-0053-41cc-9c2c-5458d0964331",
            "rbac_user_name": "admin",
            "request_id": "QUtUa3RMbRLxomqcL68ilOjjl68h56xr",
            "request_source": "kong-manager",
            "request_timestamp": 1581617463,
            "signature": null,
            "status": 200,
            "ttl": 2591995,
            "workspace": "0da4afe7-44ad-4e81-a953-5d2923ce68ae"
        }
    ],
    "total": 1
}
```

`request_source`フィールドは、Kong Managerでアクションが発生したことを示します。

`method`フィールドと`path`フィールドは、ログインまたはログアウトのいずれかのイベントに対応します。

* ログインイベント: `"method": "GET"`、`"path": "/auth"`
* ログアウトイベント: `"method": "DELETE"`、`"path": "/auth?session_logout=true"`

{% endif_version %}

### 監査ログ生成の制限

一部のAdmin APIリクエスト（`/status`エンドポイントへのヘルスチェックのリクエストなど）での監査ログの生成や、所定のワークスペースなどの特定のパスプレフィックスに対するリクエストは無視して構わない場合があります。

`audit_log_ignore_methods`と`audit_log_ignore_paths`構成オプションを使用します。

```bash
audit_log_ignore_methods = GET,OPTIONS
# do not generate an audit log entry for GET or OPTIONS HTTP requests
audit_log_ignore_paths = /foo,/status,^/services,/routes$,/one/.+/two,/upstreams/
# do not generate an audit log entry for requests that match the above regular expressions
```

`audit_log_ignore_paths`の値は、Perl互換の正規表現で照合されます。

たとえば、`audit_log_ignore_paths = /foo,/status,^/services,/routes$,/one/.+/two,/upstreams/` の場合、
次のリクエストパスでは、データベースに監査ログエントリは生成されません。

* `/status`
* `/status/`
* `/foo`
* `/foo/`
* `/services`
* `/services/example/`
* `/one/services/two`
* `/one/test/two`
* `/routes`
* `/plugins/routes`
* `/one/routes/two`
* `/upstreams/`
* `bad400request`

次のリクエストパスによりデータベースに監査ログエントリが生成されます。

* `/example/services`
* `/routes/plugins`
* `/one/two`
* `/routes/`
* `/upstreams`

データベースの監査
---------

### 監査ログの生成と表示

Admin APIリクエストデータに加えて、クラスタデータベースへの
すべての挿入、更新、および削除のエントリ用に、Kongは詳細な監査ログを生成することができます。
データベース更新の監査ログは、Admin APIリクエスト固有のIDにも関連付けられています。コンシューマを作成するための次のリクエストを検討してください。

```sh
curl -i -X POST http://localhost:8001/consumers username=bob
```

レスポンス:

```sh
HTTP/1.1 201 Created
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Tue, 13 Nov 2018 17:50:18 GMT
Server: kong/{{page.versions.ee}}-enterprise-edition
Transfer-Encoding: chunked
X-Kong-Admin-Request-ID: 59fpTWlpUtHJ0qnAWBzQRHRDv7i5DwK2

{
    "created_at": 1542131418000,
    "id": "16787ed7-d805-434a-9cec-5e5a3e5c9e4f",
    "type": 0,
    "username": "bob"
}

```

前述のように、リクエストの詳細を含むリクエスト監査ログが生成されます。
リクエストボディが記録されるときに`payload`フィールドが存在することに注意してください。

```sh
curl -i -X GET http://localhost:8001/audit/requests
```

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Tue, 13 Nov 2018 17:52:41 GMT
Server: kong/{{page.versions.ee}}-dev-enterprise-edition
Transfer-Encoding: chunked
X-Kong-Admin-Request-ID: SpPaxLTkDNndzKaYiWuZl3xrxDUIiGRR

{
    "data": [
        {
            "client_ip": "127.0.0.1",
            "method": "POST",
            "path": "/consumers",
            "payload": "{\"username\": \"bob\"}",
            "request_id": "59fpTWlpUtHJ0qnAWBzQRHRDv7i5DwK2",
            "request_timestamp": 1581617463,
            "signature": null,
            "status": 201,
            "ttl": 2591995,
            "workspace": "fd51ce6e-59c0-4b6b-b991-aa708a9ff4d2"
        }
    ],
    "total": 1
}
```

さらに、監査ログは、データベースエンティティの作成を追跡するために生成されます。

```sh
curl -i -X GET http://localhost:8001/audit/objects
```

レスポンス:

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Tue, 13 Nov 2018 17:53:27 GMT
Server: kong/{{page.versions.ee}}-dev-enterprise-edition
Transfer-Encoding: chunked
X-Kong-Admin-Request-ID: ZKra3QT0d3eJKl96jOUXYueLumo0ck8c

{
    "data": [
        {
            "dao_name": "consumers",
            "entity": "{\"created_at\":1542131418000,\"id\":\"16787ed7-d805-434a-9cec-5e5a3e5c9e4f\",\"username\":\"bob\",\"type\":0}",
            "entity_key": "16787ed7-d805-434a-9cec-5e5a3e5c9e4f",
            "expire": 1544723418009,
            "id": "7ebabee7-2b09-445d-bc1f-2092c4ddc4be",
            "operation": "create",
            "request_id": "59fpTWlpUtHJ0qnAWBzQRHRDv7i5DwK2",
            "request_timestamp": 1581617463,
        },
  ],
  "total": 1
}
```

オブジェクト監査エントリには、エンティティ本体自体、そのデータベース主キー、実行された操作の種類（`create`、`update`、または `delete`）など、更新されたエンティティに関する情報が含まれます。関連する`request_id`フィールドにも注意してください。

### 監査ログ生成の制限

リクエスト監査ログ同様、一部のデータベーステーブルの監査ログでは生成ステップをスキップした方がよい場合があります。
`audit_log_ignore_tables` Kong構成オプションからスキップする構成に変更できます。

    audit_log_ignore_tables = consumers
    # do not generate database audit logs for changes to the consumers table

監査ログの保持
-------

監査ログレコードは、[`audit_log_record_ttl`](/gateway/{{page.release}}/reference/configuration/#audit_log_record_ttl)Kong 構成プロパティで定義された期間、データベースに保持されます。`audit_log_record_ttl` 秒より古いデータベース内のレコードは自動的に削除されます。

{% if_version lte:3.3.x %}
PostgreSQLとCassandraはレコードの削除を異なる方法で処理します。

* Cassandraデータベースでは、Cassandra TTLメカニズムを通じてレコードは自動的に削除されます。 
* PostgreSQL データベースでは、レコードはデータベース挿入時に実行されるストアド プロシージャによって削除されます。{% endif_version %}

{% if_version gte:3.4.x %}
PostgreSQLは、レコードデータベースへの挿入時に実行されるストアドプロシージャ経由でレコードを消去します。{% endif_version %}
そのため、新規レコードが有効期限のタイムスタンプに従って監査テーブルに挿入されていない場合、監査リクエストのレコードは構成済みのTTLよりも長くデータベースに存在する可能性があります。

デジタル署名
------

否認防止を実現するために、監査ログはプライベートRSAキーで署名できます。有効にすると、各監査ログエントリの語順で並べ替えられた表現が定義された秘密鍵で署名されます。署名はそのレコード自体の追加フィールドに保存されます。公開鍵は別の場所に保存し、後でレコードの署名を検証するために使用できます。

### ログ署名の設定

1. `openssl` ツールを使用して秘密鍵を生成します。

   ```sh
   openssl genrsa -out private.pem 2048
   ```

2. 将来の監査検証のために公開鍵を抽出します。

   ```sh
   openssl rsa -in private.pem -outform PEM -pubout -out public.pem
   ```

3. 監査ログレコードに署名するように Kong を構成します。

       audit_log_signing_key = /path/to/private.pem

4. 監査ログエントリにフィールド`signature`が含まれるようになりました。

   {% if_version lte:3.1.x %}

   ```json
   {
       "client_ip": "127.0.0.1",
       "method": "GET",
       "path": "/status",
       "payload": null,
       "request_id": "Ka2GeB13RkRIbMwBHw0xqe2EEfY0uZG0",
       "request_timestamp": 1581617463,
       "signature": "l2LWYaRIHfXglFa5ehFc2j9ijfERazxisKVtJnYa+QUz2ckcytxfOLuA4VKEWHgY7cCLdn5C7uRJzE6es5V2SoOV59NOpskkr5lTt9kzao64UEw5UNOdeZYZKwyhG9Ge7IsxTK6haW0iG3a9dHqlKlwvnHZTbFM8TUV/umg8sJ1QJ/5ivXecbyHYtD5luKAI6oEgIdZPtQexRkwxlzvfR8lzeC/dDc2slSrjWRbBxNFlgfRKhDdVzVzgu8pEucgKggu67PKLkJ+bQEkxX1+Yg3czIpJyC3t6cgoggb0UNtBq1uUpswe0wdueKh6G5Gzz6XrmOjlv7zSz4gtVyEHZgg==",
       "status": 200,
       "ttl": 2591995,
       "workspace": "fd51ce6e-59c0-4b6b-b991-aa708a9ff4d2"
   }
   ```

   {% endif_version %}

   {% if_version gte:3.2.x %}

   ```json
   {
       "client_ip": "127.0.0.1",
       "method": "GET",
       "path": "/status",
       "payload": null,
       "rbac_user_id": "2e959b45-0053-41cc-9c2c-5458d0964331",
       "rbac_user_name": null,
       "request_id": "Ka2GeB13RkRIbMwBHw0xqe2EEfY0uZG0",
       "request_source": null,
       "request_timestamp": 1581617463,
       "signature": "l2LWYaRIHfXglFa5ehFc2j9ijfERazxisKVtJnYa+QUz2ckcytxfOLuA4VKEWHgY7cCLdn5C7uRJzE6es5V2SoOV59NOpskkr5lTt9kzao64UEw5UNOdeZYZKwyhG9Ge7IsxTK6haW0iG3a9dHqlKlwvnHZTbFM8TUV/umg8sJ1QJ/5ivXecbyHYtD5luKAI6oEgIdZPtQexRkwxlzvfR8lzeC/dDc2slSrjWRbBxNFlgfRKhDdVzVzgu8pEucgKggu67PKLkJ+bQEkxX1+Yg3czIpJyC3t6cgoggb0UNtBq1uUpswe0wdueKh6G5Gzz6XrmOjlv7zSz4gtVyEHZgg==",
       "status": 200,
       "ttl": 2591995,
       "workspace": "fd51ce6e-59c0-4b6b-b991-aa708a9ff4d2"
   }
   ```

   {% endif_version %}

### 署名の検証

レコード署名を検証するには、 `openssl`ユーティリティまたはRSA デジタル署名を検証できる
ツールその他の暗号化ユーティリティを使用します。

署名は256ビットのSHAダイジェストを使用して生成されます。次の例は、
上記の監査ログ記録の検証方法を示しています。

1. まず、Base64 エンコーディングを削除した後、レコードの署名をディスクに保存します。

   ```bash
   cat <<eof | base64 -d id="sl-md0000000" > record_signature
   > l2LWYaRIHfXglFa5ehFc2j9ijfERazxisKVtJnYa+QUz2ckcytxfOLuA4VKEWHgY7cCLdn5C7uRJzE6es5V2SoOV59NOpskkr5lTt9kzao64UEw5UNOdeZYZKwyhG9Ge7IsxTK6haW0iG3a9dHqlKlwvnHZTbFM8TUV/umg8sJ1QJ/5ivXecbyHYtD5luKAI6oEgIdZPtQexRkwxlzvfR8lzeC/dDc2slSrjWRbBxNFlgfRKhDdVzVzgu8pEucgKggu67PKLkJ+bQEkxX1+Yg3czIpJyC3t6cgoggb0UNtBq1uUpswe0wdueKh6G5Gzz6XrmOjlv7zSz4gtVyEHZgg==
   > EOF
   ```

2. 次に、監査レコードは署名生成に使用される基準形式に変換される必要があります。この変換では、レコードを検証可能な文字列形式にシリアル化する必要があります。この形式は、`signature`、`ttl`、または`expire`フィールドが *なく* 、語彙的にソートされ、各監査ログレコード部分でパイプ区切り文字列が使用されます。以下はLuaに記述された基準の実装です。

   ```lua
   local cjson = require "cjson"
   local pl_sort = require "pl.tablex".sort
   
   local function serialize(data)
   local p = {}
   
   data.signature = nil
   data.expire = nil
   data.ttl = nil
   
   for k, v in pl_sort(data) do
       if type(v) == "table" then
       p[#p + 1] = serialize(v)
       elseif v ~= cjson.null then
       p[#p + 1] = v
       end
   end
   
   return p
   end
   
   table.concat(serialize(data), "|")
   ```

   たとえば、上記の監査記録の正規の形式は次のとおりです。

       cat canonical_record.txt
       127.0.0.1|1544724298663|GET|/status|Ka2GeB13RkRIbMwBHw0xqe2EEfY0uZG0|1542132298664|200|fd51ce6e-59c0-4b6b-b991-aa708a9ff4d2

   {:.important}
   > 
   > ディスク上の正規レコードファイルの内容が、所定の正規レコード形式と厳密に一致することを確認します。
   > 末尾の改行`\n`など、追加バイトがある場合、次のステップで検証に失敗する原因になります。
3. これら 2 つの要素が配置されると、署名を検証できます。

   ```bash
   openssl dgst -sha256 -verify public.pem -signature record_signature canonical_record.txt
   Verified OK
   ```

その他の情報
------

* {{site.base_gateway}} `kong.conf`オプションの場合、 [データと管理監査](/gateway/{{page.release}}/reference/configuration/#data--admin-audit-section)セクションの 構成プロパティリファレンスを参照してください。
* `/audit` APIリファレンスについては、[監査ログ](/gateway/api/admin-ee/latest/#/audit-logs)を参照してください。

