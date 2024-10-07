---
title: "監視の概要"
content-type: "reference"
---
API ゲートウェイは、アプリケーションを外部から隔離し、アップストリームサービスに対して重要な経路を保護します。API ゲートウェイシステムの状態を理解することは、信頼性の高い API ベースのシステムを提供するために重要です。

利用可能な監視および警告システムは多数あり、 {{site.base_gateway}}は複数の解決策と統合されています。

* [Prometheus](https://prometheus.io/)は、オープンソースのシステム監視およびアラートツールキットです。Prometheus は、多次元の時系列データモデルとクエリ言語を提供します。[Prometheus による監視](/gateway/latest/production/monitoring/prometheus/)ガイドでは、{{site.base_gateway}} [Prometheus](/hub/kong-inc/prometheus/)プラグインをインストールして構成する方法を説明します。
* [Datadog](https://www.datadoghq.com/)は、人気の高いクラウドベースのインフラおよびアプリケーション監視サービスです。 {{site.base_gateway}}をDatadogと統合する情報の詳細については、 [Datadogでメトリクスを収集するガイド](/gateway/latest/production/monitoring/datadog/)を参照してください。追加の分析情報を得るために、[Datadogプラグイン](/hub/kong-inc/datadog/){{site.base_gateway}}と統合することもできます。
* [StatsD](https://github.com/statsd/statsd)は、UDPまたはTCPでアプリケーションメトリクスをリッスンする軽量のネットワークデーモンで、集計された値を1つ以上のバックエンドサービスに送信します。{{site.base_gateway}}は[StatsDプラグイン](/hub/kong-inc/statsd/)でStatsDを直接サポートします。[StatsDによるモニタリング](/gateway/latest/production/monitoring/statsd/)で、StatsDを有効にするための詳細なガイドをご確認いただけます。
* 組み込みの[ノードレディネスエンドポイント](/gateway/latest/production/monitoring/healthcheck-probes/)を使用して、{{site.base_gateway}}がリクエストを受け入れる準備ができたか監視します。

監視と密接に関連するのがトレーシングです。APIゲートウェイの計測に関する詳細については、{{site.base_gateway}}[トレーシングの参考資料](/gateway/latest/production/tracing/)を参照してください。

ヘルスチェックとは？
----------

ヘルスチェックは、{{site.base_gateway}} ノードの状態を監視するためにインフラストラクチャコンポーネント（つまり、ロードバランサー）によって実行されるアクティビティです。これは、 {{site.base_gateway}}ノードが動作可能であり、受信要求を処理する準備ができているかどうかを判断するのに役立ちます。ヘルスチェック \(「プローブ」とも呼ばれます\) の詳細については、[準備状況チェック](/gateway/latest/production/monitoring/healthcheck-probes/)ガイドを参照してください。

ベストプラクティス
---------

すべての環境は独特であり、それぞれに状況が異なるため、以下の一般的な推奨事項に従うことをおすすめします。

* [`status_listen`](/gateway/latest/reference/configuration/#status_listen)構成パラメータを有効にします。
* 常に{{site.base_gateway}}のヘルスチェックを実行し、Datadog、Grafana、AppDynamicsなどの監視ダッシュボードでヘルスを追跡します。
* [readiness probe](/gateway/latest/production/monitoring/healthcheck-probes/)を使用するには、{{site.base_gateway}}のすぐ前にあるロードバランサ―またはその他のコンポーネントを構成します。
* Kubernetesの場合は、 {{site.base_gateway}}の[livenessプローブとreadinessプローブの](/gateway/latest/production/monitoring/healthcheck-probes/)両方を構成して、ロードバランサーが正しい[Kubernetesエンドポイント](https://kubernetes.io/docs/concepts/services-networking/service/#endpoints)を使用してトラフィックを{{site.base_gateway}}ポッドに転送するようにします。 {{site.base_gateway}} readinessエンドポイントは、{{site.base_gateway}}が最初の構成を読み込み、必要なすべてのデータ構造を構築するのに常に短時間でトラフィックを正常にプロキシできるようになるため、起動直後に`200 OK`で応答することを期待しないでください。
* インシデントが発生した場合に備えて積極的に対応できるように、ヘルスチェックのレスポンスに基づいてアラートを設定します。
* `kong health`CLIコマンドを使用して{{site.base_gateway}}の全体的なヘルスを検証しないでください。このコマンドはKongプロセスが実行されていることのみを確認し、構成の機能や有効性は確認しません。
* クラスタのすべてのノードが監視されていることを確認します。たとえば、クラスタ内のデータプレーンノードを1つだけチェックしても、同じクラスタのその他のデータプレーンノードの健全性について信頼性のあるインサイトを得ることはできません。

