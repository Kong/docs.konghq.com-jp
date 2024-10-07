---
title: "ログリファレンス"
content_type: "reference"
---
ログレベル
-----

ログレベルは[Kongの構成](/gateway/{{page.release}}/reference/configuration/#log_level)で設定されます。重大度の低いものから並べると、`debug`、`info`、`notice`、`warn`、`error`、`crit`の順になります。

* *`debug`:* プラグインの実行ループと個々のプラグインやその他のコンポーネントに関するデバッグ情報を提供します。これはデバッグ中にのみ使用してください。`debug`オプションを長時間オンのままにしておくと、ディスク領域が過剰に消費される可能性があります。
* *`info`/`notice`：* Kongでは、これら2つのレベルに大きな違いはありません。通常の動作に関する情報を提供し、そのほとんどは無視して構いません。
* *`warn`：* トランザクションの中断には至らないものの、さらに調査が必要な異常な動作を記録するには、`warn`レベルを使用する必要があります。
* *`error`:* リクエストが破棄される原因となるエラー （HTTP 500 エラーを受信した場合など）のログを取得するために使用されます。このようなログの発生率を監視する必要があります。
* *`crit`：* このレベルは、Kongが危機的な状況で動作しており、正常に動作していないために複数のクライアントに影響がある場合に使用されます。Nginxには`alert`レベルと`emerg`レベルも用意されていますが、現在Kongではこれらのレベルは使用されていないため、`crit`が最も重大度の高いログレベルになります。

`notice` は、デフォルトの推奨ログレベルです。ただし、ログが長すぎる場合は、`warn` などのより上位レベルに上げることができます。

{% if_version gte:3.5.x %}

クライアントリクエスト識別子をログに記録する
----------------------

`X-Kong-Request-Id`ヘッダーには、各クライアント要求の一意の識別子が含まれます。これは、アップストリームとダウンストリームの両方でデフォルトで有効になっています。この一意の ID は、特定のリクエストを対応するエラーログと照合するのに役立ち、デバッグに役立ちます。{{site.base_gateway}}がエラーを返す場合、リクエスト ID も{{site.base_gateway}}によって生成されるレスポンス本文に含まれます。さらに、生成される{{site.base_gateway}}エラーログには、次の形式の同じ要求 ID が含まれます: `request_id: xxx` 。

デバッグヘッダーとデバッグ レスポンスヘッダーによって生成されるログ行には、リクエスト ID が含まれています。これを使用すると、ログビューアー UI でデバッグヘッダー出力を検索できます。これは、デバッグ出力が長すぎてレスポンスヘッダーに収まらない場合に特に便利です。

この機能は、 [`headers`](/gateway/latest/reference/configuration/#headers)および[`headers_upstream`](/gateway/latest/reference/configuration/#headers_upstream)構成オプションを使用して、アップストリームとダウンストリームに対してカスタマイズできます。{% endif_version %}

詳細情報
----

* [{{site.base_gateway}}ログからの要素の削除](/gateway/{{page.release}}/production/logging/customize-gateway-logs/)
{% if_version gte:3.1.x -%}
* [動的なログレベルの更新](/gateway/{{page.release}}/production/logging/update-log-level-dynamically/) {% endif_version %}

