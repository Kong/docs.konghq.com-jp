---
title: "Kong AIメトリクスの公開とグラフ化"
minimum_version: "3.8.x"
badge: "enterprise"
---
このガイドでは、AIメトリクスを収集してPrometheusに送信し、Grafanaダッシュボードを設定する方法を説明します。

概要
---

KongのAI Gateway（AI Proxy）は、AI Proxyプラグインの設定に従ってLLMベースのサービスを呼び出します。
LLMプロバイダーのレスポンスを集計して、AI Proxyによって使用されるトークンの数をカウントできます。
モデルでインプットコストとアウトプットコストを定義している場合は、コスト集計を計算することもできます。
メトリクスの詳細には、リクエストが{{site.base_gateway}}によってキャッシュされていたかどうかも表示されるため、LLMプロバイダーへの問い合わせにかかるコストが削減され、パフォーマンスが向上します。

Kong AI Gatewayは、Kongおよび[Prometheus](https://prometheus.io/docs/introduction/overview/)説明形式でプロキシされた上流サービスに関連するメトリクスを公開します。
これはPrometheus サーバーによってスクレイピングできます。

このプラグインによって追跡されるメトリクスは、`http://localhost:<port id="sl-md0000000">/metrics`エンドポイントのAdmin APIとStatus APIの両方で利用できます。これらのAPIへのURLは、インスタンス固有であることにご留意ください。詳細については、 [メトリックへのアクセス](#accessing-the-metrics)を参照してください。

Kong [Prometheus プラグイン](/hub/kong-inc/prometheus/)は、ノードレベルでメトリクスを記録し、公開します。Prometheusサーバーでサービスディスカバリメカニズムを使用してすべてのKongノードを検出し、各ノードの構成済み`/metrics`エンドポイントからデータを消費する必要があります。

Grafana ダッシュボード
---------------

プラグインがエクスポートするAIメトリクスは、[ドロップイン
ダッシュボード](https://grafana.com/grafana/dashboards/21162-kong-cx-ai/)を使用してGrafanaでグラフ化できます。

![AI Grafanaダッシュボード](/assets/images/products/gateway/vitals/grafana-ai-dashboard.png)

利用可能なメトリクス
----------
{% if_version lte: 3.7.x %}
* **AIリクエスト** : LLMプロバイダーに送られたAIリクエスト。 これらは、プロバイダー、モデル、キャッシュ、データベース名（キャッシュされている場合）、およびワークスペースごとに使用できます。
* **AI Cost** ：LLMプロバイダーが請求するAIコスト。これらは、プロバイダー、モデル、キャッシュ、データベース名（キャッシュされている場合）、およびワークスペースごとに使用できます。
* **AIトークン** ：LLMプロバイダがカウントするAIトークン。これらは、プロバイダ、モデル、キャッシュ、データベース名（キャッシュされている場合）、トークンの種類、ワークスペースごとに使用できます。

{% endif_version %}

{% if_version gte: 3.8.x %}
以下のAI LLM指標はすべてプロバイダ、モデル、キャッシュごとに利用可能です。 データベース名 (キャッシュされている場合)、埋め込みプロバイダー (キャッシュされている場合)、埋め込みモデル (キャッシュされている場合)、およびワークスペース。

ai_llm_metrics が true の場合:

AI Requests: LLM プロバイダーに送信された AI リクエスト。
AIコスト: LLMプロバイダーが請求するAIコスト。
AI トークン: LLMプロバイダーによってカウントされるAIトークン。 これらは、以前にリストされているオプションに加えて、トークンタイプごとにも使用できます。
AI LLM Latency: LLM providersによるレスポンスを返すまでの時間。
AI Cache Fetch Latency: キャッシュから応答を返すまでの時間。
AI Cache Embedding Latency: キャッシュ中に埋め込みを生成するのにかかる時間
{% endif_version %}

AI メトリクスは、メトリクスのカーディナリティが高くなり、 パフォーマンスの問題を引き起こす可能性があるため、デフォルトでは無効になっています。 これを有効にするには、Prometheus プラグインの設定で ai_metrics を true に設定します。

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

# HELP ai_requests_total AI requests total per ai_provider in Kong
# TYPE ai_requests_total counter
ai_requests_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1"} 100
# HELP ai_cost_total AI requests cost per ai_provider/cache in Kong
# TYPE ai_cost_total counter
ai_cost_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1"} 50
# HELP ai_tokens_total AI tokens total per ai_provider/cache in Kong
# TYPE ai_tokens_total counter
ai_tokens_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",token_type="input",workspace="workspace1"} 1000
ai_tokens_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",token_type="output",workspace="workspace1"} 2000
```

{:.note}
> 
> **注：** キャッシュプラグインを使用しない場合は、`cache_status`、`vector_db`、`embeddings_provider`、`embeddings_model`の各値は空になります。

メトリクスへのアクセス
-----------

ほとんどの構成では、Kong Admin APIはファイアウォールの背後にあるか、認証を
要求するように設定する必要があります。Prometheusの`/metrics`エンドポイントへの
アクセスを許可する選択肢がいくつかあります。

1. [ステータスAPI](/gateway/latest/reference/configuration/#status_listen)が
   有効になっている場合は、 `/metrics`エンドポイントを使用できます。
   これが推奨される方法です。

2. `/metrics`エンドポイントはAdmin APIでも利用可能で、Status APIが有効になっていない場合に利用できます。Admin APIで[RBAC](/gateway/api/admin-ee/latest/#/rbac/get-rbac-users/)が有効になっている場合、このエンドポイントは利用できないことに注意してください（Prometheus はトークンを渡すためのKey\-Auth をサポートしていません）。

