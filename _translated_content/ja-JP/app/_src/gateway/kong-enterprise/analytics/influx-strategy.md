---
title: "InfluxDBのVitals"
badge: "enterprise"
---
Vitalsデータ用の時系列データベースを活用することで、Kongクラスタを支えるデータベースに追加の書き込み負荷をかけることなく、トラフィックが非常に高い{{site.ee_product_name}}クラスタ（数万または数十万のデータを処理する環境など）でリクエストとVitalsのパフォーマンスを向上させることができます。

データベースをバックエンドとしてKong Vitalsを使用する方法については、[Kong Vitals](/gateway/{{page.release}}/kong-enterprise/analytics/)を参照してください。

InfluxDBを使用したKong Vitalsの設定
---------------------------

### InfluxDB データベースの起動

本番環境向けのInfluxDBインストールは別の作業としてデプロイされる必要がありますが、概念実証のテストでは、ローカルInfluxDBインスタンスをDockerで実行することは可能です。

1. コンテナがお互いを検出して通信できるようにカスタムネットワークを作成します。

   ```bash
   docker network create kong-ee-net
   ```

2. Dockerを使用してローカルInfluxDBインスタンスを起動します。

   ```bash
   docker run -d -p 8086:8086 \
     --network=kong-ee-net \
     --name influxdb \
     -e INFLUXDB_DB=kong \
     influxdb:1.8.4-alpine
   ```

   {:.warning}
   > 
   > InfluxDB 2\.0は動作 **しない** ため、 **必ず** InfluxDB 1\.8\.4\-alpine を使用する必要があります。

   Vitals データを InfluxDB に書き込むには、`kong` データベースを作成する必要があります。これは `INFLUXDB_DB` 変数を使用して行われます。

### {{site.base_gateway}}のインストール

すでに{{site.base_gateway}}インスタンスがある場合は、[ライセンスのデプロイ](#deploy-a-kong-gateway-license)に進みます。

まだ{{site.base_gateway}}をインストールしていない場合、Dockerインストールは
このガイドの目的に適しています。

#### {{site.base_gateway}} Dockerイメージをプルします \{\#pull\-image\}

1. 次のDockerイメージをプルします。

   ```bash
   docker pull kong/kong-gateway:{{page.versions.ee}}
   ```

   {:.important}
   > 
   > 一部の[古い{{site.base_gateway}}イメージ](https://support.konghq.com/support/s/article/Downloading-older-Kong-versions)は一般に公開されていません。
   > 特定のパッチバージョンが必要で、[KongのパブリックDocker Hubページ](https://hub.docker.com/r/kong/kong-gateway)から見つけられない場合は、[Kongサポート](https://support.konghq.com/)にお問い合わせください。

   これで、{{site.base_gateway}} イメージがローカルに作成されました。
2. 画像にタグを付けます。

   ```bash
   docker tag kong/kong-gateway:{{page.versions.ee}} kong-ee
   ```

#### データベースと {{site.base_gateway}} コンテナの起動

1. PostgreSQL コンテナを起動します。

   ```p
   docker run -d --name kong-ee-database \
     --network=kong-ee-net \
     -p 5432:5432 \
     -e "POSTGRES_USER=kong" \
     -e "POSTGRES_DB=kong" \
     -e "POSTGRES_PASSWORD=kong" \
     postgres:9.6
   ```

2. Kongデータベースを準備します。

   ```sh
   docker run --rm --network=kong-ee-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-ee-database" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_PASSWORD=kongpass" \
     kong-ee kong migrations bootstrap
   ```

3. Kong ManagerとInfluxDBを使用してゲートウェイを起動します。

{% include_cached /md/admin-listen.md desc='long' release=page.release %}

    ```sh
    docker run -d --name kong-ee --network=kong-ee-net \
      -e "KONG_DATABASE=postgres" \
      -e "KONG_PG_HOST=kong-ee-database" \
      -e "KONG_PG_PASSWORD=kong" \
      -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
      -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
      -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
      -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
      -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
      -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
      -e "KONG_VITALS_STRATEGY=influxdb" \
      -e "KONG_VITALS_TSDB_ADDRESS=influxdb:8086" \
      -p 8000:8000 \
      -p 8443:8443 \
      -p 8001:8001 \
      -p 8444:8444 \
      -p 8002:8002 \
      -p 8445:8445 \
      -p 8003:8003 \
      -p 8004:8004 \
      kong-ee
    ```
    
    {:.note}
    > **Note:** For `KONG_ADMIN_GUI_URL`, replace `localhost`
    with with the DNS name or IP of the Docker host. `KONG_ADMIN_GUI_URL`
    _should_ have a protocol, for example, `http://`.

### {{site.base_gateway}} ライセンスのデプロイ

{{site.base_gateway}}インスタンスにすでに{{site.ee_product_name}}ライセンスを添付している場合、
[InfluxDBでVitalsを設定する](#configure-vitals-with-influxdb)に進んでください。

{{site.base_gateway}}インスタンスに有効な
{{site.ee_product_name}}ライセンスが添付されていない限り、Kong Vitals機能にはアクセスできません。

{% include_cached /md/enterprise/deploy-license.md heading="####" release=page.release %}

### InfluxDB {{site.base_gateway}}を使用してVitalsを構成

{:.note}
> 
> **注:** [Dockerへの{{site.base_gateway}}のインストール](#install-kong-gateway)
> の構成を使用した場合は
> 、この手順を完了する必要はありません。

Kong Vitalsを有効にすることに加えて、InfluxDBをVitalsのバッキングストラテジとして使用するよう
{{site.base_gateway}}を構成する必要があります。InfluxDBのホストとポートも定義する必要があります。

```bash
echo "KONG_VITALS_STRATEGY=influxdb KONG_VITALS_TSDB_ADDRESS=influxdb:8086 kong reload exit" \
| docker exec -i kong-ee /bin/sh
```

{:.note}
> 
> **注** ：ハイブリッドモードでは、コントロールプレーンとすべてのデータプレーンの両方で[`vitals_strategy`](/gateway/{{page.release}}/reference/configuration/#vitals_strategy)と[`vitals_tsdb_address`](/gateway/{{page.release}}/reference/configuration/#vitals_tsdb_address)を構成します。

InfluxDB測定を使用したVitalsデータの理解
---------------------------

Kong Vitalsは、2つのInfluxDB測定でメトリクスを記録します。

1. `kong_request`: リクエストのレイテンシと HTTP のフィールド値、リクエストに関連するさまざまな Kong エンティティ（対象のルートやサービスなど）のタグが含まれています。
2. `kong_datastore_cache`：キャッシュのヒットとミスに関するポイントを含みます。

Docker で実行中の InfluxDB インスタンスの測定スキーマを表示するには以下のようにします。

1. InfluxDB Dockerコンテナでコマンドラインを開きます。

   ```sh
   docker exec -it influxdb /bin/sh
   ```

2. InfluxDB CLIにログインします。

   ```sh
   influx -precision rfc3339
   ```

3. 指定されたデータベースに関連するタグキーのリストを返す InfluxQL クエリを入力します。

   ```sql
   > SHOW TAG KEYS ON kong
   ```

   結果の例：

   ```sql
   name: kong_request
   tagKey
   ------
   consumer
   hostname
   route
   service
   status_f
   wid
   workspace
   
   name: kong_datastore_cache
   tagKey
   ------
   hostname
   wid
   ```

4. フィールドキーとフィールド値のデータ型を返すInfluxQLクエリを入力します。

   ```sh
   > SHOW FIELD KEYS ON kong
   ```

   結果の例：

   ```sql
   name: kong_request
   fieldKey	         fieldType
   --------	         ---------
   kong_latency       integer
   proxy_latency      integer
   request_latency    integer
   status             integer
   
   name: kong_datastore_cache
   fieldKey  fieldType
   --------  ---------
   
   hits      integer
   misses    integer
   ```

タグ `wid` は、同じタイミングで送信される重複したメトリクスを避けるために、ホストごとの一意のワーカー ID を区別するために使用されます。

上で示したように、 `kong_request`測定のシリーズカーディナリティは、Kongクラスタ構成のカーディナリティに基づいて変化します。Kongが処理するサービス/ルート/コンシューマ/ワークスペースの組み合わせの数が増加すると、Vitalsによって書き込まれるシリーズカーディナリティも増加します。

VitalsのInfluxDBノード/クラスターのサイズ設定
------------------------------

InfluxDBノード/クラスタの適切なサイジングに関しては、
[InfluxDBのサイジングガイドライン](https://docs.influxdata.com/influxdb/v1.8/guides/hardware_sizing/)
を参考にしてください。

{:.note}
> 
> **注：** Vitalsデータを読み込む場合のクエリ動作は、InfluxDBサイズ設定ガイドラインの定義に従って「中程度」の読み込みカテゴリに該当します。Vitals API応答を生成するために、いくつかの`GROUP BY`ステートメントと関数が使用され、数十万、数百万のデータポイントが存在する場合の実行では大量のCPUリソースが必要になります。

クエリの頻度と精度
---------

Kongは、Vitalsメトリックをバッファリングし、InfluxDBポイントをバッチで書き込むことで、InfluxDB のスループットを向上させ、Kong プロキシパスのオーバーヘッドを削減させます。各Kongワーカープロセスは、10秒ごと、または5000データポイントごと（いずれか早い方）にメトリックのバッファをフラッシュします。

メトリクスポイントはミリ秒（`u`）単位の精度で記述されます。[Vitals API](/api/vitals.yaml)に準拠するには、測定値は秒でグループ化されて呼び戻されます。

{:.note}
> 
> **注：** OpenResty APIには制限があるため、マイクロ秒の精度で値を書き込むには、リクエストごとに追加のシステムコールが必要です。

Kongデータベースの保持ポリシー管理
-------------------

Vitals InfluxDBデータポイントはKong を通じた保持ポリシーによりダウンサンプリングまたは管理されません。
Vitalsデータポイントの管理に必要なディスク容量とメモリを削減するため、InfluxDBオペレータは`kong`データベースの保持ポリシーを手動で管理することが推奨されます。

