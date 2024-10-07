---
title: "ライセンス使用状況の監視"
badge: "enterprise"
---
**ライセンスレポート** モジュールを使用して、ライセンスの使用状況やデプロイ情報など、{{site.base_gateway}}データベースベースのデプロイメントに関する情報を入手してください。この情報をKongと共有して、製品の使用状況と全体的なデプロイメントパフォーマンスのヘルスチェック分析を実行し、組織がニーズに最適なライセンスとデプロイメントプランで最適化されているかどうかを確認します。

ライセンスレポートモジュールの仕組み：

* ライセンスレポートモジュールは、以下に定義されているように、エンドポイントにリクエストを送信することで使用状況とデプロイメントデータを含むレポートを手動で生成します。
* このレポートをKongの担当者と共有して、デプロイメントの分析を行ってください。

ライセンスレポートモジュールで **できない** こと

* ライセンス レポート モジュールは、自動的にレポートをしたり、Kong サーバーにデータを送信したりしません。
* ライセンスレポートモジュールは、リクエストをエンドポイントに送信した後の応答で返されるデータ以外のデータを追跡または生成しません。

{:.important}
> 
> **重要:** ライセンス レポート機能は、DB レスのデプロイメントでは使用できません。

ライセンスレポートの生成
------------

ライセンスレポートモジュールを実行し、出力情報をKongの担当者と共有してデプロイメント分析を行います。

**前提条件** ：ライセンスレポートを生成するには、管理者権限が必要です。

ライセンス レポートを生成するには、HTTP クライアントから次の手順を実行します。

{% navtabs %}
{% navtab JSON response %}

JSON のレスポンスの場合は、Kong ノード エンドポイント
`/license/report` にHTTPリクエストを送信します。たとえば、次の cURL コマンドを使用します。

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
   "counters": [
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
   "db_version": "postgres 9.6.24",
   "kong_version": "{{page.versions.ee}}-enterprise-edition",
   "license_key": "KONGLICENSEKEY_NOTVALIDFORREAL_USAGE",
   "rbac_users": 0,
   "services_count": 27,
   "system_info": {
      "cores": 6,
      "hostname": "akongnode",
      "uname": "Linux x86_64"
   },
   "workspaces_count":1
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

レポートの構成
-------

|       フィールド        |                                                                                        説明                                                                                        |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `counters`         | 指定された月に行われたリクエストの数をカウントします。 <br><br>• `bucket`: リクエストが処理された年と月。`bucket` の値が `UNKNOWN` の場合、リクエストは {{site.base_gateway}} 2\.7\.0\.1 より前に処理されました。<br>\- `request_count`: 指定された月と年に処理されたリクエスト数。 |
| `db_version`       | {{site.base_gateway}}が使用するデータストアのタイプとバージョン。                                                                                                                                                     |
| `kong_version`     | {{site.base_gateway}}インスタンスのバージョン。                                                                                                                                                              |
| `license_key`      | 現在のライセンスキーの暗号化された識別子。ライセンスが存在しない場合は、フィールドには`UNLICENSED`と表示されます。                                                                                                                  |
| `rbac_users`       | RBAC を通じて登録されたユーザーの数。                                                                                                                                                            |
| `services_count`   | {{site.base_gateway}}インスタンスで設定されたサービスの数。                                                                                                                                                        |
| `system_info`      | {{site.base_gateway}}を実行しているシステムに関する情報を表示します。<br><br> • `cores` : ノード上のCPUコアの数<br>• `hostname` : 暗号化されたシステムホスト名<br>• `uname` : オペレーティングシステム                                                     |
| `workspaces_count` | {{site.base_gateway}}インスタンスで構成されるワークスペースの数。                                                                                                                                                     |

