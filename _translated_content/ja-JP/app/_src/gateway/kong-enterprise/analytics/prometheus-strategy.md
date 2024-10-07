---
title: "PrometheusのVitals"
badge: "enterprise"
---
このドキュメントでは、Kong Vitals を新規または既存の Prometheus 時系列サーバーまたはクラスタと統合する方法について説明します。Vitals データ用の時系列データベースを活用することで、Kong クラスタを支えるデータベースに追加の書き込み負荷をかけることなく、非常に高いトラフィック {{site.base_gateway}} クラスタ（数万または数十万のデータを処理する環境など）でのリクエストおよび Vitals のパフォーマンスを向上させることができます。

バックエンドとしてのデータベースで Vitals を使用する場合は、[Kong Vitals](/gateway/{{page.release}}/kong-enterprise/analytics/) を参照してください。

ライフサイクルの概要
----------

Kong Vitalsは、中間データエクスポーターである[Prometheus StatsDエクスポーター](https://github.com/prometheus/statsd_exporter)を使用してPrometheusと統合します。
この統合によりKongは、Kongプロキシ自体の中でパフォーマンスを低下させることなく、
データポイントを集計してPrometheusが利用できるようにする外部プロセスに、Vitalsのメトリクスを効率的に
送信できるようになります。この設計では、KongはVitalsメトリクスを
[StatsDメトリクス](https://github.com/statsd/statsd/blob/master/docs/metric_types.md)としてStatsDエクスポーターに書き込みます。Prometheus
は、他のエンドポイントと同じように、このエクスポーターをスクレイピングします。次に、KongはPrometheusにクエリし、APIとKong Managerを介してVitalsデータを取得して表示します。Prometheusは、時系列データのためにをKongノードを直接スクレイピングすることはありません。単純化されたワークフローは、次のようになります。

![単一ノードのデータフローの例](/assets/images/products/gateway/vitals/prometheus-single.png)

Kongの機能をノードのクラスタ間で分離することは珍しいことではありません。たとえば、1つ以上のノードがプロキシトラフィックのみを処理し、別のノードがKong Admin APIとKong Managerの処理を担当する場合です。この場合、プロキシトラフィックを担当するノードはStatsDエクスポーターにデータを書き込み、Admin APIを担当するノードはPrometheusから読み取ります。

![複数ノードのデータフローの例](/assets/images/products/gateway/vitals/prometheus-read-write.png)

どちらの場合でも、StatsDエクスポータープロセスは、スタンドアロンのプロセス/コンテナ
、あるいはVM内のサイドカー/隣接プロセスとして実行できます。トラフィック量の多い環境では、StatsDエクスポータープロセス内のデータ集計によって
CPU使用率が大幅に増加する可能性があります。そのような場合には、
CPU使用率が飽和するのを避けるため、KongプロセスとStatsDプロセスを別々のハードウェア/VM/コンテナ環境で実行します。

Vitals の Prometheus 環境を設定する
---------------------------

### Prometheusのダウンロード

Prometheusの最新リリースは、[Prometheusダウンロードページ](https://prometheus.io/download/#prometheus)にあります。

Prometheus は、Kongを実行しているノードとは別のノードで実行する必要があります。すでにインフラでPrometheusを使用しているユーザーは、既存のPrometheusノードをVitalsストレージバックエンドとして使用できます。

このガイドでは、Prometheusがホスト名`prometheus-node`で実行されていると仮定します。ポート`9090`でリッスンするデフォルト設定を使用します。

### StatsDエクスポーターのダウンロードと構成

StatsDエクスポーターはDockerイメージとして配布されています。次のコマンドで
最新バージョンをプルします。

```sh
docker pull kong/statsd-exporter-advanced:0.3.1
```

バイナリには、パブリックプロジェクトではサポートされていない最小/最大ゲージやUnixドメインソケットのサポートなどの機能が含まれています。

{:.important}
> 
> このDockerイメージは、PrometheusコミュニティプロジェクトのKongフォークであり、下記でリンクされているマッピングルールは公開イメージでは動作しません。適切なものを使用していることを確認してください。
> 
> PrometheusコミュニティのHelmチャートを使ってエクスポーターをデプロイする場合は、以下のように`values`ファイルのイメージとタグをオーバーライドします。
> 
> ```yaml
> image:
>  repository: kong/statsd-exporter-advanced
>  tag: 0.3.1
> ```

StatsDエクスポーターは、StatsD UDP イベントをPrometheus メトリックに
変換するためのマッピングルールのセットを設定する必要があります。マッピングルールの
デフォルトセットは
[statsd.rules.yaml](/code-snippets/statsd.rules.yaml)からダウンロード可能です。
次に、StatsD エクスポーターを次のように開始します。

```bash
./statsd_exporter --statsd.mapping-config=statsd.rules.yaml \
                    --statsd.listen-unixgram=''
```

StatsDマッピングルールファイルは、Kongから送信されたメトリックと一致するように構成する必要があります。StatsDイベント名をカスタマイズする方法については、[KongのPrometheusストラテジでVitalsを有効にする](#enable-vitals-with-prometheus-strategy-in-kong)セクションを参照してください。

StatsDエクスポーターは、Kongとは別のノードで実行する（Kongとのリソースの競合を避けるため）ことも、Kongと同じホストで実行する（不要なネットワークオーバーヘッドを削減するため）こともできます。

このガイドでは、StatsDエクスポーターがホスト名`statsd-node`で実行され、
ポート`9125`のUDPトラフィックをリッスンするデフォルト設定を使用し、
Prometheus公開形式のメトリクスがポート`9102`で公開されていると仮定しています。

### StatsD エクスポーターをスクレイピングするための Prometheus 構成

StatsDエクスポーターをスクレイピングするようにPrometheusを構成するには、`scrape_configs`に`prometheus.yaml`形式で次のセクションを追加します。

```yaml
scrape_configs:
  - job_name: 'vitals_statsd_exporter'
    scrape_interval: "5s"
    static_configs:
      - targets: ['statsd-node:9102']
```

StatsD エクスポーターを実行する実際のホスト名で`statsd-node`を更新してください。

サービスディスカバリを使用している場合、Prometheusに複数のStatsDエクスポーターを構成した方が便利です。
詳細は、Prometheusドキュメントの
[`scape_configs`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Cscrape_config%3E)
セクションをお読みください。

デフォルトでは、Kong ManagerのVitalsグラフは凡例で設定されたターゲットアドレスを使用します。
これは、Prometheus メトリックラベルでは`instance`と名付けられています。`instance` が `IP:PORT` のサービスディスカバリ設定については、
ユーザーは `instance` のラベルを付け直し、
よりわかりやすいホスト名を凡例に表示できるものもあります。
そのためには、ユーザーは [`scape_configs`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Cscrape_config%3E) セクションを参照し、`instance` ラベルを対応するメタラベルに書き直します。

たとえば、Kubernetes環境では次の再ラベル付けルールを使用します。

```yaml
scrape_configs:
  - job_name: 'vitals_statsd_exporter'
    kubernetes_sd_configs:
      # your SD config to filter statsd exporter pods
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: 'instance'
```

### KongでPrometheusストラテジを使用してVitalsを有効にする

構成でこれを変更して、PrometheusでVitalsを有効にすることができます。

```bash
# via your Kong configuration file
vitals = on
vitals_strategy = prometheus
vitals_statsd_address = statsd-node:9125
vitals_tsdb_address = prometheus-node:9090
```

```bash
# or via environment variables
export KONG_VITALS=on
export KONG_VITALS_STRATEGY=prometheus
export KONG_VITALS_STATSD_ADDRESS=statsd-node:9125
export KONG_VITALS_TSDB_ADDRESS=prometheus-node:9090
```

{:.note}
> 
> **注** ：ハイブリッドモードでは、コントロールプレーンとすべてのデータプレーンの両方で[`vitals_strategy`](/gateway/{{page.release}}/reference/configuration/#vitals_strategy)と[`vitals_tsdb_address`](/gateway/{{page.release}}/reference/configuration/#vitals_tsdb_address)を構成します。

StatsDエクスポーターとPrometheusを実行する実際のホスト名で`statsd-node`と`prometheus-node`を更新してください。

他のKong構成と同様に、変更は `kong reload` または
`kong restart`。

Prometheusで `scrape_interval` をデフォルト値の `5` 秒以外に設定した場合、以下も更新する必要があります。

```bash
# via your Kong configuration file
vitals_prometheus_scrape_interval = new_value_in_seconds
```

```bash
# or via environment variables
export KONG_VITALS_PROMETHEUS_SCRAPE_INTERVAL=new_value_in_seconds
```

上記のオプションは、Prometheusをクエリするときに`interval`パラメータを構成します。値`new_value_in_seconds`は、Prometheusの`scrape_interval`構成と等しいかそれより大きくなければなりません。

また、Kongを構成して`kong`のデフォルト値から異なる接頭辞を使用してStatsDイベントを送信することもできます。statsd.rulesの接頭辞がKongの構成と同じであることを確実にしてください。

```bash
# via your Kong configuration file
vitals_statsd_prefix = kong-vitals
```

```bash
# or via environment variables
export KONG_VITALS_STATSD_PREFIX=kong-vitals
```

```yaml
# in statsd.rules
mappings:
# by API
- match: kong-vitals.api.*.request.count
  name: "kong_requests_proxy"
  labels:
    job: "kong_metrics"
# follows other metrics
# ...
```

### Grafanaダッシュボードのインストール

Grafanaを使用している場合は、Kong Vitals Prometheusのダッシュボードをインポートしてデータを視覚化できます。

Grafanaインストールで、サイドバーの **\+** ボタンをクリックし、 **インポート** を選択します。

**\[インポート\]** 画面で **\[Grafana.com ダッシュボード\]** フィールドを見つけて、「`11870`」と入力します。次に **\[ロード\]** をクリックします。オプションで、
[https://grafana.com/grafana/dashboards/11870](https://grafana.com/grafana/dashboards/11870)からJSONモデルをダウンロードして手動でインポートすることもできます。

次の画面で、 `statsd-exporter`をスクレイピングするように設定された **Prometheus** データソースを選択し、 **インポート** をクリックします。

チューニングと最適化
----------

### StatsD エクスポーターの UDP バッファ

同時リクエストの量が増加すると、処理されていないUDPイベントを格納するキューの長さも増加します。パケットのドロップを避けるためには、UDP読み取りバッファサイズを増やす必要があります。

StatsDエクスポータープロセスのUDP読み取りバッファを増やすには、
次の例を使用してバイナリを実行し、読み取りバッファを約3MBに設定します。

    ./statsd_exporter --statsd.mapping-config=statsd.rules.yaml \
                        --statsd.listen-unixgram='' \
                        --statsd.read-buffer=30000000

実行中のホストの UDP 読み取りバッファを増やすには、次の例で示す行を `/etc/sysctl.conf` に追加します.

    net.core.rmem_max = 60000000

そして、以下のようにルート権限で設定を適用してください。

    # sysctl -p

### Unixドメインソケットを使用したStatsDエクスポーター

StatsD エクスポーターを Kong と同じノードにデプロイし、エクスポーターをローカル Unix ドメインソケットでリッスンさせることで、ネットワークのオーバーヘッドをさらに削減できます。

```bash
./statsd_exporter --statsd.mapping-config=statsd.rules.yaml \
                    --statsd.read-buffer=30000000 \
                    --statsd.listen-unixgram='/tmp/statsd.sock'
```

デフォルトではソケットは権限`0755`で作成されるため、
StatsDエクスポーターはKongと同じユーザーを実行して、KongがソケットにUDPパケットを書き込めるように
しなければなりません。エクスポーターとKongが異なるユーザーとして実行できるように
するため、ソケットは`0777`権限と次のもので作成できます。

```bash
./statsd_exporter --statsd.mapping-config=statsd.rules.yaml \
                    --statsd.read-buffer=30000000 \
                    --statsd.listen-unixgram='/tmp/statsd.sock' \
                    --statsd.unixsocket-umask="777"
```

エクスポートされたメトリクス
--------------

上記の構成により、Prometheus StatsDエクスポーターは、[標準Vitals構成](/api/vitals.yaml)によって提供されたすべてのメトリクスを利用可能にします。

さらに、エクスポータープロセスは、[Golang 
Prometheus クライアントライブラリ](https://prometheus.io/docs/guides/go-application/)によって公開されるデフォルトのメトリクスへのアクセスを提供します。これらのメトリクス名は、
`go_`および`process_`接頭辞を含めます。これらのデータポイントは
Kong ではなく、StatsD エクスポータープロセス自体に特に関連しています。インフラチームにとってこのデータは有益かもしれませんが、
Kongの行動を監視するという点では価値が限られています。

Prometheus から Vitals メトリクスにアクセスする
---------------------------------

Prometheus で Kong Vitals のメトリクスにアクセスし、Grafana に表示
またはアラートルールを設定することができます。StatsD のマッピングルールの例では、すべてのメトリクスに
`exported_job=kong_vitals` のラベル付けがされます。上記の Prometheus スクレイピング設定で
、すべてのメトリクスに `job=vitals_statsd_exporter` というラベルも付けられます。

