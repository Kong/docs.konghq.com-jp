---
nav_title: "概要"
---
Kongとプロキシされたアップストリームサービスに関連するメトリクスを、Prometheusサーバーでスクレイピングできる[Prometheus](https://prometheus.io/docs/introduction/overview/)形式で公開します。

このプラグインによって追跡されるメトリクスは、`http://localhost:<port id="sl-md0000000">/metrics`エンドポイントのAdmin APIとStatus APIの両方で利用できます。これらのAPIのURLはインストールごとに固有のものになることに注意してください。[メトリクスへのアクセス](#accessing-the-metrics)を参照してください。

このプラグインは、ノードレベルでメトリクスを記録して公開します。Prometheusサーバーは、サービスディスカバリメカニズムを介してすべてのKongノードを検出し、
各ノードの構成された`/metrics`エンドポイントからデータを消費する必要があります。

Grafanaダッシュボード
--------------

プラグインによってエクスポートされたメトリクスは、ドロップインダッシュボード（[https://grafana.com/grafana/dashboards/7424\-kong\-official/](https://grafana.com/grafana/dashboards/7424-kong-official/)）を使用してGrafanaでグラフ化できます。

利用可能なメトリクス
----------

{% if_version lte:2.8.x %}

* **ステータスコード** ：アップストリームサービスから返されるHTTPステータスコード。 これらは、サービスごと、すべてのサービス、コンシューマごとのルートごとに利用できます。
* **レイテンシヒストグラム** ：Kongで測定されたレイテンシ（ミリ秒単位）。 
  * **リクエスト** ：Kongとアップストリームサービスがリクエストを処理するのにかかった合計時間。
  * **Kong** ：Kongがリクエストをルーティングし、構成済みのすべてのプラグインを実行するのにかかる時間。
  * **アップストリーム** ：アップストリームサービスがリクエストに応答するのにかかる時間。

* **帯域幅** ：Kongを流れる総帯域幅（イーグレス/イングレス）。このメトリクスは、サービスごと、またはすべてのサービスの合計として利用できます。 {% endif_version %}
* **DB到達可能性** ：値が0または1のゲージタイプで、KongノードからDBにアクセスできるかどうかを表します。
* **接続** ：アクティブ、読み取り、書き込み、受け入れられた接続数などのさまざまなNginx接続メトリック。{% if_version lte:2.8.x %}
* **ターゲットの健全性** ：特定のアップストリームに属するターゲットおよびそのサブシステム（`http`または`stream`）の健全性ステータス（`healthchecks_off`、 `healthy`、 `unhealthy`、または`dns_error`）。{% endif_version %}
* **データプレーンのステータス** ：データプレーンノードの最終確認タイムスタンプ、構成ハッシュ、構成同期ステータス、および証明書有効期限タイムスタンプが、コントロールプレーンにエクスポートされます。
* **Enterpriseライセンス情報** ： {{site.base_gateway}}ライセンスの有効期限、機能、ライセンス署名。これらのメトリクスは{{site.base_gateway}}でのみエクスポートされます。
* **DBエンティティ数** <span class="badge enterprise"></span> ：現在のデータベースエンティティの数を測定するゲージメトリック。
* **Nginxタイマーの数** ：実行中または保留中の状態のNginxの総数を測定するゲージメトリック。

{% if_version gte:3.0.x %}
次のメトリクスは、メトリクスのカーディナリティが高くなり、パフォーマンスの問題が発生する可能性があるため、デフォルトでは無効になっています。

`status_code_metrics` が true に設定されている場合。

* **ステータスコード** ：アップストリームサービスから返されるHTTPステータスコード。 これらは、サービスごと、すべてのサービス、コンシューマごとのルートごとに利用できます。

`latency_metrics` が true に設定されている場合。

* **レイテンシヒストグラム** ：Kongで測定されたレイテンシ（ミリ秒単位）。 
  * **リクエスト** ：Kongとアップストリームサービスがリクエストを処理するのにかかった合計時間。
  * **Kong** ：Kongがリクエストをルーティングし、構成済みのすべてのプラグインを実行するのにかかる時間。
  * **アップストリーム** ：アップストリームサービスがリクエストに応答するのにかかる時間。

`bandwidth_metrics` が true に設定されている場合。

* **帯域幅** ：Kongを流れる総帯域幅（イーグレス/イングレス）。このメトリクスは、サービスごと、またはすべてのサービスの合計として利用できます。 

`upstream_health_metrics` が true に設定されている場合。

* **ターゲットの健全性** ：特定のアップストリームに属するターゲットおよびそのサブシステム（`http`または`stream`）の健全性ステータス（`healthchecks_off`、 `healthy`、 `unhealthy`、または`dns_error`）。

{% endif_version %}

{% if_version gte:3.8.x %}
`ai_llm_metrics`を`true`に設定した場合。

* **AIリクエスト** ：LLMプロバイダに送られたAIリクエスト。 これらは、プロバイダ、モデル、キャッシュ、データベース名（キャッシュされている場合）、およびワークスペースごとに使用できます。
* **AIコスト** ：LLMプロバイダが請求するAIコスト。 これらは、プロバイダ、モデル、キャッシュ、データベース名（キャッシュされている場合）、およびワークスペースごとに使用できます。
* **AIトークン** ：LLMプロバイダがカウントするAIトークン。 これらは、プロバイダ、モデル、キャッシュ、データベース名（キャッシュされている場合）、トークンの種類、ワークスペースごとに使用できます。

詳細については、 [AIメトリクス](/gateway/latest/production/monitoring/ai-metrics/)を参照してください。
{% endif_version %}

`/metrics`エンドポイントから期待できる出力の例を次に示します。

```bash
curl -i http://localhost:8001/metrics
```

応答：

```sh
HTTP/1.1 200 OK
Server: openresty/1.15.8.3
Date: Tue, 7 Jun 2020 16:35:40 GMT
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Access-Control-Allow-Origin: *

{% if_version gte:3.0.x %}
# HELP kong_bandwidth_bytes Total bandwidth (ingress/egress) throughput in bytes
# TYPE kong_bandwidth_bytes counter
kong_bandwidth_bytes{service="google",route="google.route-1",direction="egress",consumer=""} 264
kong_bandwidth_bytes{service="google",route="google.route-1",direction="ingress",consumer=""} 93
# HELP kong_datastore_reachable Datastore reachable from Kong, 0 is unreachable
# TYPE kong_datastore_reachable gauge
kong_datastore_reachable 1
# HELP kong_http_requests_total HTTP status codes per consumer/service/route in Kong
# TYPE kong_http_requests_total counter
kong_http_requests_total{service="google",route="google.route-1",code="200",source="service",consumer=""} 1
# HELP kong_node_info Kong Node metadata information
# TYPE kong_node_info gauge
kong_node_info{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",version="3.0.0"} 1
# HELP kong_kong_latency_ms Latency added by Kong and enabled plugins for each service/route in Kong
# TYPE kong_kong_latency_ms histogram
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="5"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="7"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="10"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="15"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="20"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="30"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="50"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="75"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="100"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="200"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="500"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="750"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="1000"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="+Inf"} 1
kong_kong_latency_ms_count{service="google",route="google.route-1"} 1
kong_kong_latency_ms_sum{service="google",route="google.route-1"} 4
# HELP kong_memory_lua_shared_dict_bytes Allocated slabs in bytes in a shared_dict
# TYPE kong_memory_lua_shared_dict_bytes gauge
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_cluster_events",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache",kong_subsystem="http"} 823296
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache_miss",kong_subsystem="http"} 90112
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache",kong_subsystem="http"} 794624
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache_miss",kong_subsystem="http"} 86016
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_healthchecks",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_locks",kong_subsystem="http"} 61440
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_process_events",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_rate_limiting_counters",kong_subsystem="http"} 86016
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="prometheus_metrics",kong_subsystem="http"} 57344
# HELP kong_memory_lua_shared_dict_total_bytes Total capacity in bytes of a shared_dict
# TYPE kong_memory_lua_shared_dict_total_bytes gauge
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_cluster_events",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache",kong_subsystem="http"} 134217728
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache_miss",kong_subsystem="http"} 12582912
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache",kong_subsystem="http"} 134217728
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache_miss",kong_subsystem="http"} 12582912
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_healthchecks",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_locks",kong_subsystem="http"} 8388608
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_process_events",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_rate_limiting_counters",kong_subsystem="http"} 12582912
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="prometheus_metrics",kong_subsystem="http"} 5242880
# HELP kong_memory_workers_lua_vms_bytes Allocated bytes in worker Lua VM
# TYPE kong_memory_workers_lua_vms_bytes gauge
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21173",kong_subsystem="http"} 64329517
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21174",kong_subsystem="http"} 46314808
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21175",kong_subsystem="http"} 46681598
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21176",kong_subsystem="http"} 46637209
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21177",kong_subsystem="http"} 46234336
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21178",kong_subsystem="http"} 46180420
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21179",kong_subsystem="http"} 46161105
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21180",kong_subsystem="http"} 46366877
# HELP kong_nginx_connections_total Number of connections by subsystem
# TYPE kong_nginx_connections_total gauge
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="accepted"} 296
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="active"} 9
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="handled"} 296
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="reading"} 0
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="total"} 296
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="waiting"} 0
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="writing"} 9
# HELP kong_nginx_metric_errors_total Number of nginx-lua-prometheus errors
# TYPE kong_nginx_metric_errors_total counter
kong_nginx_metric_errors_total 0
# HELP kong_nginx_requests_total Total number of requests
# TYPE kong_nginx_requests_total gauge
kong_nginx_requests_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http"} 296
# HELP kong_nginx_timers Number of Nginx timers
# TYPE kong_nginx_timers gauge
kong_nginx_timers{state="pending"} 1
kong_nginx_timers{state="running"} 39
# HELP kong_request_latency_ms Total latency incurred during requests for each service/route in Kong
# TYPE kong_request_latency_ms histogram
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="25"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="50"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="80"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="100"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="250"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="400"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="700"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="1000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="2000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="5000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="10000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="30000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="60000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="+Inf"} 1
kong_request_latency_ms_count{service="google",route="google.route-1"} 1
kong_request_latency_ms_sum{service="google",route="google.route-1"} 6
# HELP kong_upstream_latency_ms Latency added by upstream response for each service/route in Kong
# TYPE kong_upstream_latency_ms histogram
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="25"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="50"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="80"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="100"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="250"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="400"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="700"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="1000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="2000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="5000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="10000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="30000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="60000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="+Inf"} 1
kong_upstream_latency_ms_count{service="google",route="google.route-1"} 1
kong_upstream_latency_ms_sum{service="google",route="google.route-1"} 2
{% endif_version %}

{% if_version lte:2.8.x %}
# HELP kong_bandwidth Total bandwidth in bytes consumed per service/route in Kong
# TYPE kong_bandwidth counter
kong_bandwidth{type="egress",service="google",route="google.route-1"} 1277
kong_bandwidth{type="ingress",service="google",route="google.route-1"} 254
{% if_version gte:2.8.x %}
# HELP kong_nginx_timers Number of nginx timers
# TYPE kong_nginx_timers gauge
kong_nginx_timers{state="running"} 3
kong_nginx_timers{state="pending"} 1
{% endif_version %}
# HELP kong_datastore_reachable Datastore reachable from Kong, 0 is unreachable
# TYPE kong_datastore_reachable gauge
kong_datastore_reachable 1
# HELP kong_http_consumer_status HTTP status codes for customer per service/route in Kong
# TYPE kong_http_consumer_status counter
kong_http_consumer_status{service="s1",route="s1.route-1",code="200",consumer="CONSUMER_USERNAME"} 3
# HELP kong_http_status HTTP status codes per service/route in Kong
# TYPE kong_http_status counter
kong_http_status{code="301",service="google",route="google.route-1"} 2
# HELP kong_latency Latency added by Kong in ms, total request time and upstream latency for each service in Kong
# TYPE kong_latency histogram
kong_latency_bucket{type="kong",service="google",route="google.route-1",le="00001.0"} 1
kong_latency_bucket{type="kong",service="google",route="google.route-1",le="00002.0"} 1
.
.
.
kong_latency_bucket{type="kong",service="google",route="google.route-1",le="+Inf"} 2
kong_latency_bucket{type="request",service="google",route="google.route-1",le="00300.0"} 1
kong_latency_bucket{type="request",service="google",route="google.route-1",le="00400.0"} 1
.
.
kong_latency_bucket{type="request",service="google",route="google.route-1",le="+Inf"} 2
kong_latency_bucket{type="upstream",service="google",route="google.route-1",le="00300.0"} 2
kong_latency_bucket{type="upstream",service="google",route="google.route-1",le="00400.0"} 2
.
.
kong_latency_bucket{type="upstream",service="google",route="google.route-1",le="+Inf"} 2
kong_latency_count{type="kong",service="google",route="google.route-1"} 2
kong_latency_count{type="request",service="google",route="google.route-1"} 2
kong_latency_count{type="upstream",service="google",route="google.route-1"} 2
kong_latency_sum{type="kong",service="google",route="google.route-1"} 2145
kong_latency_sum{type="request",service="google",route="google.route-1"} 2672
kong_latency_sum{type="upstream",service="google",route="google.route-1"} 527
# HELP kong_nginx_http_current_connections Number of HTTP connections
# TYPE kong_nginx_http_current_connections gauge
kong_nginx_http_current_connections{state="accepted"} 8
kong_nginx_http_current_connections{state="active"} 1
kong_nginx_http_current_connections{state="handled"} 8
kong_nginx_http_current_connections{state="reading"} 0
kong_nginx_http_current_connections{state="total"} 8
kong_nginx_http_current_connections{state="waiting"} 0
kong_nginx_http_current_connections{state="writing"} 1
# HELP kong_memory_lua_shared_dict_bytes Allocated slabs in bytes in a shared_dict
# TYPE kong_memory_lua_shared_dict_bytes gauge
kong_memory_lua_shared_dict_bytes{shared_dict="kong",kong_subsystem="http"} 40960
.
.
# HELP kong_memory_lua_shared_dict_total_bytes Total capacity in bytes of a shared_dict
# TYPE kong_memory_lua_shared_dict_total_bytes gauge
kong_memory_lua_shared_dict_total_bytes{shared_dict="kong",kong_subsystem="http"} 5242880
.
.
# HELP kong_memory_workers_lua_vms_bytes Allocated bytes in worker Lua VM
# TYPE kong_memory_workers_lua_vms_bytes gauge
kong_memory_workers_lua_vms_bytes{pid="7281",kong_subsystem="http"} 41124353
# HELP kong_data_plane_config_hash Config hash value of the data plane
# TYPE kong_data_plane_config_hash gauge
kong_data_plane_config_hash{node_id="d4e7584e-b2f2-415b-bb68-3b0936f1fde3",hostname="ubuntu-bionic",ip="127.0.0.1"} 1.7158931820287e+38
# HELP kong_data_plane_last_seen Last time data plane contacted control plane
# TYPE kong_data_plane_last_seen gauge
kong_data_plane_last_seen{node_id="d4e7584e-b2f2-415b-bb68-3b0936f1fde3",hostname="ubuntu-bionic",ip="127.0.0.1"} 1600190275
# HELP kong_data_plane_version_compatible Version compatible status of the data plane, 0 is incompatible
# TYPE kong_data_plane_version_compatible gauge
kong_data_plane_version_compatible{node_id="d4e7584e-b2f2-415b-bb68-3b0936f1fde3",hostname="ubuntu-bionic",ip="127.0.0.1",kong_version="2.4.1"} 1
# HELP kong_nginx_metric_errors_total Number of nginx-lua-prometheus errors
# TYPE kong_nginx_metric_errors_total counter
kong_nginx_metric_errors_total 0
# HELP kong_upstream_target_health Health status of targets of upstream. States = healthchecks_off|healthy|unhealthy|dns_error, value is 1 when state is populated.
kong_upstream_target_health{upstream="UPSTREAM_NAME",target="TARGET",address="IP:PORT",state="healthchecks_off",subsystem="http"} 0
kong_upstream_target_health{upstream="UPSTREAM_NAME",target="TARGET",address="IP:PORT",state="healthy",subsystem="http"} 1
kong_upstream_target_health{upstream="UPSTREAM_NAME",target="TARGET",address="IP:PORT",state="unhealthy",subsystem="http"} 0
kong_upstream_target_health{upstream="UPSTREAM_NAME",target="TARGET",address="IP:PORT",state="dns_error",subsystem="http"} 0
{% endif_version %}
```

{:.note}
> 
> **注：** アップストリームターゲットのヘルス情報は、サブシステムごとに1回エクスポートされます。両方のストリームとHTTPリスナーが有効になっている場合は、ターゲットの状態は2回表示されます。ヘルスメトリクスには、メトリクスが参照するサブシステムを示す`subsystem`ラベルがあります。

メトリクスへのアクセス
-----------

ほとんどの構成では、Kong Admin APIはファイアウォールの背後にあるか、認証を要求するように設定する必要があります。Prometheusの`/metrics`エンドポイントへのアクセスを許可する選択肢がいくつかあります。

1. [ステータスAPI](/gateway/latest/reference/configuration/#status_listen)が有効になっている場合は、 `/metrics`エンドポイントを使用できます。
   これが好ましい方法です。

2. `/metrics`エンドポイントはAdmin APIでも利用可能で、Status APIが有効になっていない場合に利用できます。Admin APIで[RBAC](/gateway/api/admin-ee/latest/#/rbac/get-rbac-users/)が有効になっている場合、このエンドポイントは利用できないことに注意してください（Prometheus はトークンを渡すためのKey\-Auth をサポートしていません）。

*** ** * ** ***

