---
title: "Datadogによるメトリクスの収集"
content_type: "how-to"
---
Datadogでは、{{site.base_gateway}}と[Prometheusプラグイン](/hub/kong-inc/prometheus/)を使用してメトリクスを収集できます。

前提条件
----

* [Datadog Agent v6以降](https://docs.datadoghq.com/agent/)
* [{{site.base_gateway}}でPrometheusプラグインを有効にする](/hub/kong-inc/prometheus/#example-config)

PrometheusプラグインをDatadogに接続
--------------------------

Datadog Agent 6を使用すると、DatadogをPrometheusエンドポイントに接続してメトリクスの収集を開始できます。

{{site.base_gateway}}の Prometheus プラグインを有効にした後、 [Datadog Agent openmetrics.d](https://docs.datadoghq.com/integrations/openmetrics/)を作成します。`/etc/datadog-agent/conf.d/openmetrics.d/conf.yaml`の構成。これは、エージェントに{{site.base_gateway}}からメトリックのスクレイピングを開始するように指示します。

以下は、すべての`kong_`接頭辞付きメトリクスをプルするための構成例です。

    instances:
      - prometheus_url: http://localhost:8001/metrics
        namespace: "kong"
        metrics:
          - kong_*

PrometheusとDatadog Agentを使用してメトリクスを収集する詳細については、[ホストからのPrometheusおよびOpenMetricsメトリクス収集](https://docs.datadoghq.com/integrations/guide/prometheus-host-collection/#pagetitle)を参照してください。

