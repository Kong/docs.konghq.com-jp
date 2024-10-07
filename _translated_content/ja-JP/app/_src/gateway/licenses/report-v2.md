---
title: "ライセンス使用状況の監視"
badge: "enterprise"
---
**ライセンスレポート** モジュールを使用して、ライセンスの使用状況やデプロイ情報など、{{site.base_gateway}} データベースを基盤としたデプロイに関する情報を取得します。この情報を Kong と共有して、製品の使用状況と全体的なデプロイのパフォーマンスのヘルスチェック分析を実行し、ニーズに最適なライセンスとデプロイプランをもとに組織が最適化されていることを確認します。

ライセンスレポートモジュールの仕組み：

* ライセンスレポートモジュールは、以下に定義されているように、エンドポイントにリクエストを送信することで、使用状況とデプロイメントデータを含むレポートを手動で生成します。
* このレポートをKongの担当者と共有して、デプロイメントの分析を行ってください。

ライセンスレポートモジュールで **できない** こと

* ライセンス レポート モジュールは、自動的にレポートをしたり、Kong サーバーにデータを送信したりしません。
* ライセンスレポートモジュールは、エンドポイントにリクエストを送信した後のレスポンスで返されるデータ以外のデータを追跡または生成しません。

{:.important}
> 
> **重要:** ライセンスレポート機能は、DBレスのデプロイメントでは使用できません。

ライセンスレポートの生成
------------

ライセンスレポートモジュールを実行し、出力情報をKongの担当者と共有してデプロイメント分析を行います。

**前提条件** ：ライセンスレポートを作成するには、管理者権限が必要です。

ライセンスレポートを生成するには、HTTPクライアントから次の手順を実行します。

{% navtabs %}
{% navtab JSON response %}

JSON応答では、Kongノードエンドポイントの`/license/report`にHTTPリクエストを送信します。たとえば、次のcURLコマンドを使用します。

```bash
curl {ADMIN_API_URL}/license/report
```

次の例のようなJSON応答が返されます。

```json
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: http://localhost:8002
Connection: keep-alive
Content-Length: 814
Content-Type: application/json; charset=utf-8
Date: Mon, 06 Dec 2021 12:04:28 GMT
Server: kong/{{page.versions.ee}}-enterprise-edition
Vary: Origin
X-Kong-Admin-Request-ID: R1jmopI6fjkOLdOuPJVLEmGh4sCLMpSY
{
    "plugins_count": {
        "tiers": {
            "enterprise": {
                "kafka-upstream": 2,
                "rate-limiting-advanced": 1
            },
            "free": {
                "cors": 1
            },
            "custom": {
            }
        },
        "unique_route_lambdas": 0,
        "unique_route_kafkas": 1
    },
    "routes_count": 5,
    "timestamp": 1697013865,
    "license": {
        "license_expiration_date": "2017-7-20",
        "license_key": "KONGLICENSEKEY_NOTVALIDFORREAL_USAGE"
    },
    "checksum": "2935ed72e118fbaca9c09b9a6065f08a8ea40851",
    "counters": {
        "buckets": [
            {
                "bucket": "2021-09",
                "request_count": 30
            },
            {
                "bucket": "2020-10",
                "request_count": 42
            },
            {
                "bucket": "2021-11",
                "request_count": 296
            },
            {
                "bucket": "2021-12",
                "request_count": 58
            },
            {
                "bucket": "UNKNOWN",
                "request_count": 50
            }
        ],
        "total_requests": 476
    },
    "services_count": 27,
    "kong_version": "{{page.versions.ee}}-enterprise-edition",
    "db_version": "postgres 15.2",
    "system_info": {
        "cores": 6,
        "uname": "Linux x86_64",
        "hostname": "akongnode"
    },
    "deployment_info": {
        "type": "traditional"
    },
    "workspaces_count": 1
}
```

{% endnavtab %}
{% navtab TAR file %}

TARファイルに次のcURLコマンドを入力してKong Admin APIを呼び出します。

```bash
curl {ADMIN_API_URL}/license/report -o response.json && tar -cf report-$(date +"%Y_%m_%d_%I_%M_%p").tar response.json
```

ライセンスレポートファイルが生成され、 `*.tar`ファイルにアーカイブされます。

{% endnavtab %}
{% endnavtabs %}

レポート構成
------

|       フィールド        |                                                                                                                                                                                     説明                                                                                                                                                                                     |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `counters`         | リクエスト数をカウントします。<br><br>• `buckets`：指定された月に行われたリクエストの数をカウントします。<br><br>∙ `bucket`：リクエストが処理した年と月。`bucket`の値が`UNKNOWN`である場合は、リクエストは{{site.base_gateway}} 2\.7\.0\.1より前に処理されたことになります。<br>∙ `request_count`：指定された月と年に処理されたリクエスト数。<br><br>• `total_requests`：`buckets`で処理されたリクエスト数。つまり、`total_requests`は`buckets`の各項目の`request_count`を合計したものと等しくなります。                                        |
| `plugins_count`    | 使用中のプラグインの数をカウントします。<br><br> • `tiers`：ライセンスティアごとにカウントを分けます。∙ `free` : 使用中の空きプラグインの数。<br> ∙ `enterprise` : 使用中のエンタープライズプラグインの数。<br> ∙ `custom` : 使用中のカスタムプラグインの数。<br><br> • `unique_route_lambdas` : 使用中の`awk-lambda`プラグインの数。プラグインがサービスレスルートレベルで定義され、一意の関数名を持つ場合にのみカウントします。<br>• `unique_route_kafkas`：サービスレスルートレベルで定義された`kafka-upstream`プラグインのブローカー配列にリストされた一意のブローカーIPの数。 |
| `db_version`       | {{site.base_gateway}}が使用するデータストアのタイプとバージョン。                                                                                                                                                                                                                                                                                                                                               |
| `kong_version`     | {{site.base_gateway}}インスタンスのバージョン。                                                                                                                                                                                                                                                                                                                                                        |
| `license`          | 現在ライセンスを実行している{{site.base_gateway}}インスタンスに関する情報を表示します。<br><br>• `license_expiration_date`: 現在のライセンスの有効期限が切れる日付。ライセンスが存在しない場合は、フィールドには`2017-7-20`と表示されます。<br> • `license_key` : 現在のライセンスキー。ライセンスが存在しない場合は、フィールドには`UNLICENSED`と表示されます。                                                                                                                                                      |
| `rbac_users`       | RBAC を通して登録されたユーザーの数。                                                                                                                                                                                                                                                                                                                                                      |
| `services_count`   | {{site.base_gateway}}インスタンスで構成されるサービスの数。                                                                                                                                                                                                                                                                                                                                                  |
| `routes_count`     | {{site.base_gateway}} インスタンスで設定されたルートの数。                                                                                                                                                                                                                                                                                                                                                  |
| `system_info`      | {{site.base_gateway}}を実行しているシステムに関する情報を表示します。<br><br> • `cores` : ノード上のCPUコアの数<br>• `hostname` : 暗号化されたシステムホスト名<br>• `uname` : オペレーティングシステム\`                                                                                                                                                                                                                                             |
| `deployment_info`  | {{site.base_gateway}}を実行するデプロイメントに関する情報を表示します。<br><br> • `type`：デプロイメントモードのタイプ<br> • `connected_dp_count`：クラスタ全体のデータプレーンの数。デプロイメントがハイブリッドモードでない場合、フィールドは表示されません。                                                                                                                                                                                                                          |
| `timestamp`        | 現在の応答のタイムスタンプ。                                                                                                                                                                                                                                                                                                                                                             |
| `checksum`         | 現在のレポートのチェックサム。                                                                                                                                                                                                                                                                                                                                                            |
| `workspaces_count` | {{site.base_gateway}} インスタンスで構成されるワークスペースの数。                                                                                                                                                                                                                                                                                                                                              |

