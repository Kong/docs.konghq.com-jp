---
title: "Kong Gatewayのパフォーマンスベンチマークを確立する"
content_type: "reference"
description: "このドキュメントでは、ベンチマークを確立して Kong Gateway のパフォーマンスを最適化するための方法を総合的にご紹介します。"
---
{{site.base_gateway}}はすぐに使用できる状態で最適化されていますが、{{site.base_gateway}}の構成オプションを微調整することによってパフォーマンスが大幅に向上する場合があります。
{{site.base_gateway}}の初期ベンチマークを実行して`kong.conf`を最適化し、さらにいくつかのベンチマークテストを実施することによって、パフォーマンスのベースラインを確立できます。

このガイドでは、次について説明します。

* 初期 {{site.base_gateway}} パフォーマンス ベンチマークを確立する方法
* 追加のベンチマークを実行する前に{{site.base_gateway}}パフォーマンスを最適化する方法
* ベンチマーク用に`kong.conf`を構成する方法

前提条件
----

{% if_version eq:3.4.x %}
{{site.base_gateway}}3\.4\.2\.0 以降が必要です。{% endif_version %}

ベンチマークテストを実施する前に、テストベッドが正しく構成されていることを確認する必要があります。以下は、ベンチマークテストを開始する前のいくつかの一般的な推奨事項です。

* 多数の小さな{{site.base_gateway}}ノードでなく、4つまたは8つのNGINXワーカーを対応するCPUリソース割り当てと併用して、{{site.base_gateway}}のノード数を減らします。 
* {{site.base_gateway}}をDB\-lessまたはハイブリッドモードで実行します。これらのモードでは、{{site.base_gateway}}のプロキシノードはデータベースに接続されておらず、パフォーマンスに影響を及ぼす要素になる可能性があります。

ベースライン {{site.base_gateway}} パフォーマンス ベンチマークの実行
-------------------------------

[前提条件](#prerequisites)の推奨事項を実装すると、ベンチマーク テストを開始できます。

1. [Request Terminationプラグイン](/hub/kong-inc/request-termination/)でルートを構成し、{{site.base_gateway}}のパフォーマンスを測定します。この場合、 {{site.base_gateway}}はリクエストに応答し、アップストリームサーバーにトラフィックを送信しません。
2. このテストを数回実行して、予期しないボトルネックを見つけます。{{site.base_gateway}}、ベンチマーク クライアント（k6 や Apache JMeter など）、またはその他のコンポーネントが予期しないボトルネックになる可能性があります。これらのボトルネックを解決するまでは、{{site.base_gateway}} のパフォーマンス向上は期待できません。このベースライン パフォーマンスが許容できる場合にのみ、次の手順に進んでください。
3. ベースラインを確立したら、プラグインなしでアップストリームサーバーにトラフィックを送信するルートを設定します。 これは、{{site.base_gateway}}のプロキシとアップストリームサーバーのパフォーマンスを測定します。
4. 続行する前に、予期せずボトルネックを引き起こしているコンポーネントがないことを確認してください。
5. データの信頼性を高めるために、ベンチマークを複数回実行します。 オブザベーション間の差が高くない（標準偏差が低い）ことを確認します。
6. ベンチマークの最初の 1 回または 2 回の反復で収集された統計を破棄します。システムが最適かつ安定したレベルで動作していることを確認するために、これを実行することをお勧めします。

追加の構成で {{site.base_gateway}} のベンチマークを実行するには、前の手順を完了する必要があります。追加のベンチマークを実行する前に、次のセクションの最適化に関する推奨事項をよく読み、必要に応じて設定を変更してください。

{{site.base_gateway}}のパフォーマンスを最適化
------------------

このセクションのサブセクションでは、追加のベンチマークテストで{{site.base_gateway}}パフォーマンスを向上させる
推奨事項について詳しく説明します。
各セクションを注意深く読み、構成ファイルに必要な調整を加えてください。

### 以下を確認してください`ulimit`

**アクション：** `ulimit`が`16384`より小さい場合は、値を大きくします。

**説明：** {{site.base_gateway}} はシステムから得られる出来る限り多くのリソースを使用できますが、
オペレーティングシステム（OS）は {{site.base_gateway}} がアップストリーム（または他の）サーバーと開くことができるコネクション数、またはクライアントから受け入れることができるコネクション数を制限します。{{site.base_gateway}}のオープン接続数はデフォルトで`ulimit`に設定されており、上限は16384です。
つまり、 `ulimit`が無制限であるか、16384より大きい値である場合、{{site.base_gateway}}は16384に制限されます。

{{site.base_gateway}}のコンテナまたはVMにシェルでアクセスして`ulimit -n`を実行すると、システムの`ulimit`を確認できます。
{{site.base_gateway}}がVM上のコンテナ内で実行されている場合は、コンテナにシェルでアクセスする必要があります。
`ulimit`の値が16384より小さければ、それを増加させます。
また、これらのシステムで接続ボトルネックがあるとパフォーマンスが最適でなくなるため、クライアントとアップストリームサーバーで、
適切な`ulimit`を確認して設定します。

### 接続の再利用を増やす

**アクション:** `upstream_keepalive_max_requests = 100000`と`nginx_http_keepalive_requests = 100000`を構成します。

**説明：** 10,000RPS以上の高スループットシナリオでは、TCPおよびTLS接続を設定するオーバーヘッドや不十分な接続により、ネットワーク帯域幅またはアップストリームサーバーの使用率が低下する可能性があります。接続の再利用を増やすには、 `upstream_keepalive_max_requests`と`nginx_http_keepalive_requests`を`100000`まで増やすか、最大の`500000`まで増やすことができます。

### 自動スケーリングの回避

**アクション：** {{site.base_gateway}}が拡大/縮小（水平）または拡大/縮小（垂直）されていないことを確認します。

**説明:** ベンチマークの実行中は、{{site.base_gateway}} がスケールイン/スケールアウト（水平）またはアップ/ダウン（垂直）になっていないことを確認してください。Kubernetesでは、これは通常、水平または垂直ポッドオートスケーラーを使用して行われます。オートスケーラーはベンチマークの統計に干渉し、不要なノイズを発生させます。

ベンチマーク中に自動スケーリングされないよう、ベンチマークをテストする前に{{site.base_gateway}}をスケールアウトしてください。{{site.base_gateway}} ノードの数を監視して、ベンチマーク中に新しいノードが生成されることを確認し、
既存のノードは置き換えられません。

### 複数のコアを効果的に使用する

**アクション:** ほとんどの VM セットアップでは、 `nginx_worker_processes`を`auto`に設定します。Kubernetesでは`nginx_worker_processes`を設定し
ワーカーノードの CPUより1つまたは2つ少なくします。

**説明:** `nginx_worker_processes` が正しく構成されていることを確認してください。

* ほとんどのVMセットアップでは、これを`auto`に設定します。これがデフォルト設定です。これにより、NGINXは1つのCPUコアに対して1つのワーカープロセスを生成するようになります。

* Kubernetes でこれを明示的に設定することをお勧めします。{{site.base_gateway}}
  のための CPU のリクエストと制限が {{site.base_gateway}} で設定されたワーカーの数と一致することを確認します。たとえば、`nginx_worker_processes=4` と設定すると、
  ポッドのスペックでは 4 つの CPU をリクエストする必要があります。

  Kubernetesワーカーノードとn個のCPUを併用して{{site.base_gateway}}ポッドを実行する場合、n\-2またはn\-1を
  {{site.base_gateway}}に割り当てて、ワーカープロセスカウントをこの数と等しく構成します。
  これにより、kubeletなどの設定済みのデーモンとKubernetesプロセスが{{site.base_gateway}}を持つリソースと競合しないように徹底できます。

  ワーカーが増えるたびにメモリ消費量が増加するため、{{site.base_gateway}}によってLinux Out\-of\-Memory Killerがトリガーされないことを確実にする必要があります。

### リソース競合

**アクション:** クライアント（Apache JMeterやk6など）、 {{site.base_gateway}} 、およびアップストリームサーバーが異なる
マシン \(VM またはベアメタル）を低レイテンシで同じローカルネットワーク上で実行されていることを確認してください。

**説明:** 

* クライアント（Apache JMeterやk6など）、{{site.base_gateway}}、アップストリームサーバーが異なるタイプのマシン（VMまたはベアメタル）で実行することを確認します。 これらがすべてKubernetesクラスタで実行されている場合は、これら3つのシステム向けポッドのスケジュールが専用ノードで組まれるようにします。 これらシステム間でリソースが競合（通常はCPUとネットワーク）する場合、どのシステムでも最適なパフォーマンスが得られない可能性があります。
* クライアント、{{site.base_gateway}}、およびアップストリームサーバーが同じローカルネットワーク上で低レイテンシーで動作していることを確認してください。クライアントと {{site.base_gateway}} 間、または {{site.base_gateway}} とアップストリームサーバー間のリクエストがインターネットを通過する場合、結果には不要なノイズが含まれることになります。

### アップストリームサーバーの限界

**アクション:** アップストリームサーバーが負荷の上限に達していないことを確認してください。

**説明：** アップストリームサーバーのCPUとメモリをチェックすることで、アップストリームサーバーが最大限に達していないことを確認できます。追加の{{site.base_gateway}}ノードをデプロイしても、スループットまたはエラーレートが変わらない場合は、アップストリームサーバーまたは{{site.base_gateway}}以外のシステムがボトルネックになっている可能性があります。

アップストリームサーバーが自動スケーリングされないようにする必要があります。

### クライアントの限界

**アクション：** クライアントはkeepalive接続を使用する必要があります。

**説明：** クライアント（k6やApache JMeterなど）が最大になることがあります。その調整には、クライアントを理解することが必要です。クライアントのCPU、スレッド、接続数を増やすと、
リソースの使用率とスループットが向上します。

また、クライアントはkeepalive接続を使用しなければなりません。たとえば、[k6](https://k6.io/docs/using-k6/k6-options/reference/#no-connection-reuse)
とApache JMeterの[HTTPClient4](https://hc.apache.org/httpcomponents-client-4.5.x/index.html)の実装は、デフォルトでどちらもkeepaliveが有効になります。これがテスト設定で適切に設定されていることを確認します。

### カスタムプラグイン

**アクション：** カスタムプラグインがパフォーマンスを妨げていないことを確実にしてください。

**説明：** カスタムプラグインはパフォーマンスに問題を引き起こすことがあります。
まず、カスタムプラグインがパフォーマンスの問題の原因であるかどうかを判断する必要があります。これを行うには、次の3つの構成バリエーションを測定します。

1. プラグインを有効にせずに{{site.base_gateway}}のパフォーマンスを測定します。これにより、 {{site.base_gateway}}のパフォーマンスのベースラインが提供されます。
2. 必要なバンドルプラグイン（製品に付属するプラグイン）を有効にしてから、{{site.base_gateway}}のパフォーマンスを測定します。
3. 次に、カスタムプラグイン（バンドルされたプラグインに加えて）を有効にして、{{site.base_gateway}} のパフォーマンスをもう一度測定します。

{{site.base_gateway}} のベースラインパフォーマンスが低い場合は、{{site.base_gateway}} の構成のチューニングが必要か、もしくは外部要因が影響している可能性があります。外部要因については、このガイドの他のセクションを参照してください。2 番目と 3 番目の手順のパフォーマンスに大きな差がある場合は、パフォーマンスの問題はカスタムプラグインが原因である可能性があります。

### クラウドプロバイダーのパフォーマンスの問題

**アクション：** バースト可能なインスタンスを使用していないか、帯域幅、単位時間あたりのTCP接続、PPSの制限に達していないことを確認してください。

**説明：** 以下ではAWSについて言及しますが、同じ推奨事項はほとんどのクラウドプロバイダーに当てはまります。

* AWSで、Tタイプのインスタンスのようなバースト可能なインスタンスを使用していないことを確認してください。 この場合、アプリケーションで使用できる CPU は変動するため、統計にノイズが発生します。 詳細については、[バースト可能なパフォーマンスインスタンス](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances.html)のAWSドキュメントを参照してください。
* 帯域幅制限、単位時間あたりのTCP接続数制限、またはパケット毎秒（PPS）制限に達していないことを確認してください。詳細については、[Amazon EC2 インスタンスのネットワーク帯域幅](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-network-bandwidth.html)に関する AWS ドキュメントを参照してください。

### ベンチマークテスト中の構成変更

**アクション:** ベンチマークテスト中に {{site.base_gateway}} 構成を変更しないでください。

**説明：** テスト中に構成を変更すると、{{site.base_gateway}}のテールレイテンシが急増する可能性があります。
そのため、構成変更後の{{site.base_gateway}}のパフォーマンスを測定する目的を除き、テスト中は構成を変更しないでください。

### 大規模なリクエストと応答本文

**アクション：** リクエストボディは8KB未満、レスポンス本文は32KB未満にしてください。

**説明：** ほとんどのベンチマーク設定は、通常、小さなHTTPボディとそれに対応するJSONまたはHTMLのレスポンスボディを含む
HTTPリクエストで構成されます。
リクエストボディが 8 KB 未満、レスポンス本文が 32 KB 未満の場合は小さいとみなされます。
リクエストまたはレスポンスボディが大きい場合、{{site.base_gateway}} はディスクを使用してリクエストとレスポンスをバッファリングします。
これは {{site.base_gateway}} のパフォーマンスに大きな影響を与えます。

### サードパーティシステムのボトルネック

**説明：** 多くの場合、{{site.base_gateway}} のボトルネックは、{{site.base_gateway}}が使用しているサードパーティシステムのボトルネックが原因です。次のセクションでは、一般的なサードパーティのボトルネックとその解決方法について説明します。

#### Redis

**アクション：** Redisを使用し、いずれかのプラグインが有効になっている場合、CPUがボトルネックになる可能性があります。CPUを追加して、Redisを垂直方向に拡張します。

**説明：** Redisを使用し、いずれかのプラグインが有効になっている場合は、Redisがボトルネックになっていないことを確認してください。通常、CPUはRedisのボトルネックとなるため、まずCPUの使用状況を確認してください。この場合は、追加のCPUを追加してRedisを垂直方向に拡張します。

{% if_version gte:3.8.x %}

#### DNS クライアント

**アクション:** 新しい DNS クライアントに移行します。

**説明** 新しいDNSクライアントは、以前のものよりもパフォーマンスが高いように設計されているため、移行はパフォーマンスを向上させます。

{% endif_version %}

#### DNS TTL

{% if_version lte:3.7.x %}
**アクション:** `dns_stale_ttl` を `300` または `86400` に増やします。
{% endif_version %}
{% if_version gte:3.8.x %}
**アクション:**`dns_stale_ttl` を `300` または `86400` に増やします。
{% endif_version %}

**説明：** {{site.base_gateway}}はリクエストの送信先を決定するためにDNSに依存しているため、DNSサーバーが{{site.base_gateway}}のボトルネックになる可能性があります。

In the case of Kubernetes, DNS TTLs are 5 seconds long and can cause problems.
{% if_version lte:3.7.x %}
`dns_stale_ttl`を`300`に、または最大`86400`をチケットとしてDNSを除外できます。
{% endif_version %}
{% if_version gte:3.8.x %}
`dns_stale_ttl` または `resolver_stale_ttl` を増やすことができます。 使用しているDNSクライアントに応じて、`300`または`86400`までのDNSを問題として除外します。
{% endif_version %}

DNSサーバーが根本原因である場合、CPUでボトルネックを発生させているポッドが`coredns`個あることがわかります。

### アクセスログのI/Oのブロック

**アクション:** 高スループットのベンチマークテストに対しては、`proxy_access_log` 設定パラメータを `off` に設定してアクセスログを無効にします。

**説明:** {{site.base_gateway}}と基礎となるNGINXは非ブロッキングネットワーク I/O 用にプログラムされており、ディスク I/O のブロックを可能な限り回避します。ただし、アクセスログはデフォルトで有効になっており、{{site.base_gateway}}ノードに電力を供給するディスクが何らかの理由で襲い場合、パフォーマンスの低下につながる可能性があります。`proxy_access_log`構成パラメータを`off`に設定して、高スループットベンチマークテストのアクセスログを無効にします。

### {{site.base_gateway}} の内部エラー

**アクション:** {{site.base_gateway}} の[エラーログ](/gateway/{{page.release}}/reference/configuration/#log_level)にエラーがないことを確認します。

**説明:** {{site.base_gateway}} のエラー ログで内部エラーを確認します。
内部エラーにより、 {{site.base_gateway}} 内の問題、またはトラフィックを中継するために {{site.base_gateway}} が依存しているサードパーティ システムの問題が明らかになる場合があります。

ベンチマークのための kong.conf の例
-----------------------

次の `kong.conf` ファイルの例には、前のセクションで推奨されたすべてのパラメータが含まれています。

{% navtabs %}
{% navtab kong.conf %}

`kong.conf` を直接編集して設定を適用する場合は、以下を使用します。

```bash
# For a Kubernetes setup, change nginx_worker_processes to a number matching the CPU limit. We recommend 4 or 8.
nginx_worker_processes=auto

upstream_keepalive_max_requests=100000
nginx_http_keepalive_requests=100000

proxy_access_log=off

dns_stale_ttl=3600
```

{% endnavtab %}
{% navtab Environment variable format %}

環境変数を使用して設定を適用する場合は、以下を使用します。

```bash
# For a Kubernetes setup, change nginx_worker_processes to a number matching the CPU limit. We recommend 4 or 8.
KONG_NGINX_WORKER_PROCESSES="auto"
KONG_UPSTREAM_KEEPALIVE_MAX_REQUESTS="100000"
KONG_NGINX_HTTP_KEEPALIVE_REQUESTS="100000"

KONG_PROXY_ACCESS_LOG="off"

KONG_DNS_STALE_TTL="3600"
```

{% endnavtab %}
{% navtab Helm chart values.yaml %}

Helm チャートを通して構成を適用する場合は、以下を使用します。

```yaml
# The value of 1 for nginx_worker_processes is a suggested value. 
# Change nginx_worker_processes to a number matching the CPU limit. We recommend 4 or 8.
# Allocate the same amount of CPU and appropriate memory to avoid OOM killer.
env:
  nginx_worker_processes: "1"
  upstream_keepalive_max_requests: "100000"
  nginx_http_keepalive_requests: "100000"
  proxy_access_log: "off"
  dns_stale_ttl: "3600"

resources:
  requests:
    cpu: 1
    memory: "2Gi"
```

{% endnavtab %}
{% endnavtabs %}

次の手順
----

{{site.base_gateway}}のパフォーマンスを最適化したので、追加のベンチマークを実行できます。
常に測定し、変更を加え、再度測定します。行き詰まったときに次のステップを理解したり、別のアプローチに戻ったりできるように、変更の記録を保存しておいてください。

{% if_version gte:3.6.x %}

詳細情報
----

* [パフォーマンステストのベンチマーク結果](/gateway/{{page.release}}/production/performance/performance-testing)：Kongの現在のバージョンのパフォーマンステストのベンチマーク結果を見て、Kongのテストスイートを使用した独自のパフォーマンステストを実行する方法を学びましょう。 {% endif_version %}

