---
title: "Prometheus によるモニタリング"
content_type: "how-to"
---
[Prometheus](https://prometheus.io/)は、システム監視とアラート用の人気のツールキットです。多次元の時系列データモデルと分散ストレージシステムを実装し、メトリクスデータをHTTP経由のプルモデルで収集します。


{{site.base_gateway}}は[Prometheusプラグイン](/hub/kong-inc/prometheus/)でPrometheusをサポートしており、
{{site.base_gateway}}パフォーマンスとプロキシされたアップストリームサービスメトリクスを`/metrics`エンドポイント上で公開します。

このガイドはテスト{{site.base_gateway}}および
Prometheusサービスのセットアップに役立ちます。次に、{{site.base_gateway}}へのサンプルリクエストを生成し、
収集された監視データを観測します。

### 前提条件

このガイドでは、以下のツールがローカルにインストールされていることを前提としています。

* [Docker](https://docs.docker.com/get-docker/)は、{{site.base_gateway}}、サポートデータベース、Prometheusをローカルで実行するために使用されます。
* [curl](https://curl.se/) は {{site.base_gateway}} にリクエストを送信するために使用されます。`curl` は大抵のシステムにプリインストールされています。

### Prometheus 監視を構成する

1. {{site.base_gateway}}のインストール。

   {:.note}
   > 
   > 既存の{{site.base_gateway}}インストールを使用する場合、この手順はオプションです。既存の
   > 
        {{site.base_gateway}}を使用する場合は、ネットワーク接続とインストールされている{{site.base_gateway}}サービスおよびルートを考慮してコマンドを変更する必要があります。

   ```sh
   curl -Ls https://get.konghq.com/quickstart | bash -s -- -m
   ```

   `-m`フラグは、スクリプトにこのガイドでサンプルメトリクスを生成するために使用するモックサービスをインストールするよう指示します。

   {{site.base_gateway}}の準備が完了すると、以下のメッセージが表示されます。

   ```text
   Kong Gateway Ready
   ```

2. Prometheus {{site.base_gateway}}プラグインをインストールします。

   ```sh
   curl -s -X POST http://localhost:8001/plugins/ \
    --data "name=prometheus" 
   ```

   インストールされたプラグインの詳細を含むJSON応答が返されます。
3. 現在のディレクトリに `prometheus.yml` という名前のPrometheus設定ファイルを作成し、次の値をコピーします。

   ```text
   scrape_configs:
     - job_name: 'kong'
       scrape_interval: 5s
       static_configs:
         - targets: ['kong-quickstart-gateway:8001']
   ```

   これらの設定の詳細については、Prometheusの[設定ドキュメント](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)を参照してください。
4. Prometheusサーバーを実行し、前の手順で作成した構成ファイルを渡します。Prometheusは
   {{site.base_gateway}}からのメトリクスデータの収集を開始します。

   ```sh
   docker run -d --name kong-quickstart-prometheus \
      --network=kong-quickstart-net -p 9090:9090 \
      -v $(PWD)/prometheus.yml:/etc/prometheus/prometheus.yml \
      prom/prometheus:latest
   ```

5. モックサービスへのサンプルトラフィックを生成します。これにより、StatsDプラグインから
   生成されたメトリクスを確認できます。次のコマンドは1分間に60の
   リクエストを生成します。新しいターミナルで以下を実行します。

   ```bash
   for _ in {1..60}; do {curl -s localhost:8000/mock/anything; sleep 1; } done
   ```

6. 以下のように、[Admin API](/gateway/{{page.release}}/admin-api/)上の `/metrics` エンドポイントをクエリすることで、{{site.base_gateway}}からメトリクスデータを直接表示できます。

   ```sh
   curl -s localhost:8001/metrics
   ```

   
   {{site.base_gateway}}は、デフォルトでシステム全体のパフォーマンスメトリクスを報告します。
   プラグインがインストールされ、トラフィックがプロキシされると、サービス、ルート、
   アップストリームのディメンションにわたる追加のメトリクスが記録されます。

   応答は次のスニペットのようになります。

   ```text
   # HELP kong_bandwidth Total bandwidth in bytes consumed per service/route in Kong
   # TYPE kong_bandwidth counter
   kong_bandwidth{service="mock",route="mock",type="egress"} 13579
   kong_bandwidth{service="mock",route="mock",type="ingress"} 540
   # HELP kong_datastore_reachable Datastore reachable from Kong, 0 is unreachable
   # TYPE kong_datastore_reachable gauge
   kong_datastore_reachable 1
   # HELP kong_http_status HTTP status codes per service/route in Kong
   # TYPE kong_http_status counter
   kong_http_status{service="mock",route="mock",code="200"} 6
   # HELP kong_latency Latency added by Kong, total request time and upstream latency for each service/route in Kong
   # TYPE kong_latency histogram
   kong_latency_bucket{service="mock",route="mock",type="kong",le="1"} 4
   kong_latency_bucket{service="mock",route="mock",type="kong",le="2"} 4
   ...
   ```

   利用可能なメトリクスと構成の詳細については、
   [Kong Prometheusプラグインのドキュメント](/hub/kong-inc/prometheus/)を参照してください。
7. Prometheusには、収集されるメトリクスデータのクエリ方法が複数あります。

   [Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)ビューアはブラウザで
   [http://localhost:9090/graph](http://localhost:9090/graph)を開いて表示できます。

   Prometheusの[HTTP API](https://prometheus.io/docs/prometheus/latest/querying/api/)を使用して直接クエリすることもできます。

   ```sh
   curl -s 'localhost:9090/api/v1/query?query=kong_http_status'
   ```

   Prometheusは、収集した時系列データの
   視覚化ツールとしての[Grafana](https://grafana.com/)を設定するための、[ドキュメントも提供します](https://prometheus.io/docs/visualization/grafana/)。

### クリーンアップ

Prometheusと{{site.base_gateway}}の実験が終わったら、次のコマンドを使用し、
このガイドで作成したサービスを停止および削除することができます。

```sh
docker stop kong-quickstart-prometheus
curl -Ls https://get.konghq.com/quickstart | bash -s -- -d
```

### その他の情報

* [StatsDで監視する方法](/gateway/{{page.release}}/production/monitoring/statsd/) [{{site.base_gateway}}プラグイン](/hub/kong-inc/statsd/)での監視に、[StatsD](https://github.com/statsd/statsd)を使用するためのガイドを提供します。 
* {{site.base_gateway}} のトレース機能については、[トレーシング API リファレンス](/gateway/{{page.release}}/production/tracing/api/)を参照してください。

