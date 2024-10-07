---
title: "StatsDによるモニタリング"
content_type: "how-to"
---
[StatsD](https://github.com/statsd/statsd)は、ネットワークでアプリケーションが公開したテキストベースの統計データをリッスンしてパフォーマンスメトリクスを収集し、集計するネットワークデーモンです。

このガイドはテスト用の{{site.base_gateway}}とStatsDサービスのセットアップに役立ちます。
セットアップしたら、{{site.base_gateway}}へのサンプルリクエストを生成し、
収集された監視データを観測します。

### 前提条件

* [Docker](https://docs.docker.com/get-docker/)は、StatsDを実行するのに使用され、サービスをローカルでサポートします。
* [curl](https://curl.se/)は{{site.base_gateway}}にリクエストを送信するために使用されます。`curl`はほとんどのシステムにプリインストールされています。
* [Netcat](http://netcat.sourceforge.net/) はシステム `PATH` に `nc` としてインストールされます。`nc` は、StatsD 管理インターフェイスにリクエストを送信するために使用されます。`nc` は多くのシステムにプリインストールされています。

### StatsDモニタリングの構成

1. {{site.base_gateway}}のインストール。

   {:.note}
   > 
   > 既存の{{site.base_gateway}}インストールを使用する場合、この手順はオプションです。既存の
        {{site.base_gateway}}を使用する場合は、ネットワーク接続とインストールされている{{site.base_gateway}}サービスおよびルートを考慮してコマンドを変更する必要があります。

   ```sh
   curl -Ls https://get.konghq.com/quickstart | bash -s -- -m
   ```

   `-m`フラグは、スクリプトにこのガイドでサンプルメトリクスを生成するために使用するモックサービスをインストールするよう指示します。

   {{site.base_gateway}}の準備が完了すると、以下のメッセージが表示されます。

   ```text
   Kong Gateway Ready 
   ```

2. StatsD コンテナを実行して監視データを取得します。

   ```sh
   docker run -d --rm -p 8126:8126 \
     --name kong-quickstart-statsd --network=kong-quickstart-net \
     statsd/statsd:latest
   ```

3. [StatsD {{site.base_gateway}} プラグイン](/hub/kong-inc/statsd/)をインストールし、リッスンしている StatsD サービスのホスト名とポートを構成します。

   ```sh
   curl -X POST http://localhost:8001/plugins/ \
       --data "name=statsd"  \
       --data "config.host=kong-quickstart-statsd" \
       --data "config.port=8125"
   ```

   インストールされたプラグインの詳細を含む JSON 応答が返されます。
4. モックサービスへのサンプルトラフィックを生成します。これにより、StatsDプラグインから
   生成されたメトリクスを確認できます。次のコマンドは1分間に60の
   リクエストを生成します。新しいターミナルで以下を実行します。

   ```sh
   for _ in {1..60}; do {curl localhost:8000/mock/anything; sleep 1; } done
   ```

5. StatsD管理インターフェースをクエリして、{{site.base_gateway}}から収集されたメトリクスを確認します。

   ```sh
   echo "counters" | nc localhost 8126
   ```

   以下のようなレスポンスが表示されます。

   ```text
   {
     'statsd.bad_lines_seen': 0,
     'statsd.packets_received': 56,
     'statsd.metrics_received': 56,
     'kong.mock.request.count': 7,
     'kong.mock.request.status.200': 7,
     'kong.mock.request.status.total': 7
   }
   END
   ```

StatsDプラグインの使用方法と構成方法の詳細については、「[StatsDプラグイン](/hub/kong-inc/statsd/)のドキュメントを参照してください。

### クリーンアップ

StatsDと{{site.base_gateway}}の見極めが終わったら、次のコマンドを利用して、
このガイド内の実行したソフトウェアを停止および削除できます。

```sh
docker stop kong-quickstart-statsd
curl -Ls https://get.konghq.com/quickstart | bash -s -- -d
```

### 詳細情報

* [Prometheusで監視する方法](/gateway/{{page.release}}/production/monitoring/prometheus/)では、[Prometheusプラグイン](/hub/kong-inc/prometheus/)を使って{{site.base_gateway}}を監視するための[Prometheus](https://prometheus.io/docs/introduction/overview/)の使い方について紹介します。
* {{site.base_gateway}}のトレース機能については、[トレーシング APIリファレンス](/gateway/{{page.release}}/production/tracing/api/)を参照してください。

