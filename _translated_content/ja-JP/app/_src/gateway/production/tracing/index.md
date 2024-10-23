---
title: "トレースリファレンス"
content-type: "reference"
---
このリファレンスでは、{{site.base_gateway}} の OpenTelemetry トレース機能について説明します。
{% if_version gte:3.7.x %}

{:.important}
> 
> **重要** : OpenTelemetryトレースは、非推奨の Granular トレース機能に代わるものです。Granular トレースは 3\.7\.0\.0 以降の {{site.base_gateway}} から削除され、`tracing = on` などの構成は使用できなくなりました。その代替として、このページで説明する OpenTelemetryトレースを使用します。{% endif_version %}

コア計測
----

次の計測は、KongのトレーシングAPI上に構築された任意のプラグインで使用できます。[OpenTelemetryプラグイン](/hub/kong-inc/opentelemetry/)はその一例です。

{% if_version gte:3.2.x %}
Kongはトレース用のコア計測のセットを提供しており、これらは`tracing_instrumentations`構成で構成できます。
{% endif_version %}
{% if_version lte:3.1.x %}
Kongはトレース用のコア計測セットを提供しており、これらは`opentelemetry_tracing`構成で構成できます。{% endif_version %}

* `off`: 計測を有効にしません。
* `request`: リクエストレベルの計測のみを有効にします。
* `all`：すべてのインストルメンテーションを有効にします。
* `db_query`：データベースクエリをトレースします。
* `dns_query`: DNSクエリをトレースします。
* `router`：ルーターの再構築を含む、ルーターの実行をトレースします。
* `http_client`: OpenResty HTTPクライアントリクエストをトレースします。
* `balancer`: トレースバランサーの再試行。
* `plugin_rewrite`: リライトフェーズでプラグインのイテレータ実行をトレースします。
* `plugin_access`: アクセスフェーズでプラグインのイテレータ実行をトレースします。
* `plugin_header_filter`: `header_filter` でプラグインのイテレータ実行をトレースします。

伝搬
---

トレースAPI は、次のヘッダーの伝播をサポートしています。

* `w3c` \- [W3Cトレースコンテキスト](https://www.w3.org/TR/trace-context/)
* `b3`、 `b3-single` \- [Zipkin ヘッダー](https://github.com/openzipkin/b3-propagation)
* `jaeger`[\- イェーガーヘッダー](https://www.jaegertracing.io/docs/client-libraries/#propagation-format)
* `ot` \- [OpenTracingヘッダー](https://github.com/opentracing/specification/blob/master/rfc/trace_identifiers.md)

トレーシングAPIはヘッダーから伝播形式を検出し、適切な形式を使用してスパンコンテキストを伝播します。
適切な形式が見つからない場合は、ユーザーが指定できるデフォルトの形式に戻ります。

伝達 APIは、OpenTelemetry プラグインと Zipkin プラグインの両方で動作します。

ヘッダー
----

サポートされているすべてのトレースヘッダーについては、[kong.conf](/gateway/latest/reference/configuration/#headers)をご覧ください。

{% if_version gte:3.5.x %}

### X\-Kong\-Request\-Id header

T`X-Kong-Request-Id`ヘッダーには、クライアントのリクエストごとに一意の識別子が含まれています。 これはデフォルトで上流と下流の両方で有効になります。 この一意な ID は、特定のリクエストを対応するエラーログにマッチさせるのに役立ちます。 {{site.base_gateway}} が PDK kong.response.error を呼び出してエラーを返した場合、 {{site.base_gateway}} が生成したレスポンスボディにもリクエスト ID が含まれます。 さらに、生成された {{site.base_gateway}} エラーログには、request_id: `xxx` という形式のリクエストIDが含まれます。

デバッグヘッダとデバッグレスポンスヘッダによって生成されたログ行には、リクエスト ID が含まれます。 これを使用して、ログビューアの UI を使用してデバッグヘッダー出力を検索できます。 これは、デバッグ出力がレスポンスヘッダーに収まるには長すぎる場合に特に便利です。

デバッグヘッダーとデバッグレスポンスヘッダーによって生成されるログ行には、リクエストIDが含まれています。これを使用すると、ログビューアーUIでデバッグヘッダー出力を検索できます。これは、デバッグ出力が長すぎてレスポンスヘッダーに収まらない場合に特に便利です。{% endif_version %}