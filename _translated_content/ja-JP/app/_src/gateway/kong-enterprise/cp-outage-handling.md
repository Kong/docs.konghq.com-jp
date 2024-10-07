---
title: "データプレーン（DP）の復元力を構成する方法"
badge: "enterprise"
content_type: "how-to"
toc: false
---
バージョン3\.2以降、{{site.base_gateway}}を設定して、コントロールプレーン（CP）が停止した場合に新しいデータプレーン（DP）を構成するのをサポートできます。この機能は、1 つ以上のバックアップノードを指定し、データストアへの読み取り/書き込みアクセスを許可することで機能します。このバックアップノードは、有効な{{site.base_gateway}}構成をデータストアに自動的にプッシュします。新しいノードが作成されたときにコントロールプレーン（CP）が停止した場合、データストアから最新の{{site.base_gateway}}構成を取得して自動的に構成し、リクエストのプロキシ処理を開始します。

このオプションは、より大きなメンテナンス負荷が必要となるため、厳格な可用性SLAを遵守する必要があるお客様にのみ推奨されます。

{% navtabs %}
{% navtab Amazon S3 %}

前提条件
----

* Amazon S3 バケット
* バケットの認証情報の読み取りと書き込み

構成
---

この設定では、バックアップノードを 1 つ指定する必要があります。
バックアップノードにはS3バケットへの読み取りと書き込みのアクセス権が必要で、プロビジョニングされたデータプレーンノードには同じS3バケットへの読み取りアクセス権が必要です。
バックアップノードは、コントロールプレーンから S3 バケットに{{site.base_gateway}} `kong.conf`構成ファイルの状態を伝達する役割を担います。

ノードは、`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_DEFAULT_REGION` などの環境変数を介してフォールバック構成で初期化されます。
IAM ロールに関連付ける場合、およびバックアップノードが AWS プラットフォーム上に存在しない場合は、追加の環境変数 `AWS_SESSION_TOKEN` が必要になることがあります。

{:.important}
> 
> トラフィックをプロキシするためにバックアップノードを使用することはお勧めしません。バックアップジョブはプロキシDPの攻撃対象領域を拡大し、P99遅延に大きく影響します。この方法でノードをデプロイする場合は、リスクを知っておく必要があります。
> {% if_version lte:3.5.x %}と、バックアップノードとして機能するDPは、バックアップ構成ではプロビジョニングできません。{% endif_version %} {% if_version gte:3.6.x %}とバックアップノードとして構成されている場合、バックアップ構成でプロビジョニングするには、DPが少なくとも`3.6.0.0`である必要があります。
> すべてのデプロイメントには単一のバックアップノードで十分ですが、追加のバックアップノードを設定することもできます。リーダー選出アルゴリズムは、指定されたバックアップノードのグループから1つのノードを選択してバックアップジョブを実行します。
> {% endif_version %}

環境変数で設定されるデータの詳細については、[AWS環境変数構成ドキュメント](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)を参照してください。

Docker Composeを使用すると、バックアップノードを構成できます。

```yaml
kong-exporter:
    image: 'kong/kong-gateway:latest'
    ports:
      - '8000:8000'
      - '8443:8443'
    environment:
      <<: *other-kong-envs
      AWS_REGION: 'us-east-2'
      AWS_ACCESS_KEY_ID: <access_key_write id="sl-md0000000">
      AWS_SECRET_ACCESS_KEY: <secret_access_key_write id="sl-md0000000">
      KONG_CLUSTER_FALLBACK_CONFIG_STORAGE: s3://test-bucket/test-prefix
      KONG_CLUSTER_FALLBACK_CONFIG_EXPORT: "on"

```

{% if_version lte:3.5.x %}
このノードは、新しい構成を受信したときにS3バケットに書き込みます。ファイル構造はバケット内に自動的に作成されるため、手動で作成しないでください。上記の例でノードバージョンが`3.2.0.0`の場合、キー名は`test-prefix/3.2.0.0/config.json`です。{% endif_version %}

{% if_version gte:3.6.x %}
次の段落で説明するオブジェクトキー名/プレフィックスはすべて、設定およびゲートウェイバージョンで指定されたプレフィックスでパラメータ化されます。とえば、ノードに `3.6.0.0` のバージョンがあるとします。

バックアップノードは、接頭辞 `test-prefix/3.6.0.0/election/` を付けてリーダー選挙を実行するための登録ファイルを作成します。何日も更新されない場合、この接頭辞の付いたオブジェクトを削除するライフサイクルルールを設定できます。

選択したノードは、新しい構成を受信したときに S3 バケットに書き込みます。ファイル構造はバケット内に自動的に作成されるため、手動で作成しないでください。キー名は `test-prefix/3.6.0.0/config.json` です。

{% endif_version %}

コントロールプレーンとデータプレーンの両方を構成して、構成をエクスポートできます。

次の環境変数を使用してコントロールプレーン（CP）にアクセスできない場合、新しいデータプレーン（DP）の構成を通じてS3バケットから構成を読み込むことができます。

```yaml
kong-dp-importer:
    image: 'kong/kong-gateway:latest'
    ports:
      - '8000:8000'
      - '8443:8443'
    environment:
      <<: *other-kong-envs
      AWS_REGION: 'us-east-2'
      AWS_ACCESS_KEY_ID: <access_key_read id="sl-md0000000">
      AWS_SECRET_ACCESS_KEY: <secret_access_key_read id="sl-md0000000">
      KONG_CLUSTER_FALLBACK_CONFIG_STORAGE: s3://test-bucket/test-prefix
      KONG_CLUSTER_FALLBACK_CONFIG_IMPORT: "on"

```

{% endnavtab %}
{% navtab GCP Cloud Storage %}

前提条件
----

* GCP クラウドストレージバケット
* バケットの認証情報の読み取りと書き込み

構成
---

この設定では、バックアップノードを1つ指定する必要があります。
バックアップノードにはGCPクラウドストレージバケットへの読み取りと書き込みのアクセス権が必要で、プロビジョニングされたデータプレーンノードには同じGCPクラウドストレージバケットへの読み取りアクセス権が必要です。
このノードは、コントロールプレーンから GCP クラウドストレージバケットに {{site.base_gateway}} `kong.conf` 設定ファイルの状態を伝える役割を果たします。

認証情報は、環境変数`GCP_SERVICE_ACCOUNT`を介して渡されます。認証情報についての詳細は、[GCP認証情報のドキュメント](https://developers.google.com/workspace/guides/create-credentials)を参照してください。

バックアップノードはトラフィックのプロキシには使用しないでください。すべてのデプロイメントには単一のバックアップノードで十分です。

Docker Compose を使用すると、バックアップノードを構成できます。

```yaml
kong-dp-exporter:
    image: 'kong/kong-gateway:latest'
    ports:
      - '8000:8000'
      - '8443:8443'
    environment:
      <<: *other-kong-envs
      GCP_SERVICE_ACCOUNT: <gcp_json_string_write id="sl-md0000000">
      KONG_CLUSTER_FALLBACK_CONFIG_STORAGE: gcs://test-bucket/test-prefix
      KONG_CLUSTER_FALLBACK_CONFIG_EXPORT: "on"
```

このノードは、新しい構成の受信時にGCPバケットに書き込みむ役割を担います。
ファイル構造はバケット内に自動的に作成され、手動では作成しません。ノードバージョンが`3.2.0.0`の場合、上記例のキー名は`test-prefix/3.2.0.0/config.json`となります。

コントロールプレーンとデータプレーンの両方を構成して、構成をエクスポートできます。

次の環境変数を使用して、コントロールプレーンにアクセスできない場合にGCPクラウドストレージバケットから設定を読み込むように新しいデータプレーンを設定できます。

```yaml
  kong-dp-importer:
    image: 'kong/kong-gateway:latest'
    ports:
      - '8000:8000'
      - '8443:8443'
    environment:
      <<: *other-kong-envs
      GCP_SERVICE_ACCOUNT: <gcp_json_string_read id="sl-md0000000">
      KONG_CLUSTER_FALLBACK_CONFIG_STORAGE: gcs://test-bucket/test-prefix
      KONG_CLUSTER_FALLBACK_CONFIG_IMPORT: "on"
```

{% endnavtab %}

{% navtab S3 object storage%}
AWS S3に互換しないオブジェクトストレージを構成できます。構成プロセスはAWS S3プロセスと類似しますが、追加のパラメータ`AWS_CONFIG_STORAGE_ENDPOINT`をオブジェクトストレージプロバイダーのエンドポイントに設定する必要があります。

以下の例では、MinIOを使用してバックアップノードの構成を示しています。

```yaml
  kong-exporter:
    image: 'kong/kong-gateway:latest'
    ports:
      - '8000:8000'
      - '8443:8443'
    environment:
      <<: *other-kong-envs
      AWS_REGION: 'us-east-2'
      AWS_ACCESS_KEY_ID: <access_key_write id="sl-md0000000">
      AWS_SECRET_ACCESS_KEY: <secret_access_key_write id="sl-md0000000">
      KONG_CLUSTER_FALLBACK_CONFIG_EXPORT: "on"
      KONG_CLUSTER_FALLBACK_CONFIG_STORAGE: s3://test-bucket/test-prefix
      AWS_CONFIG_STORAGE_ENDPOINT: http://minio:9000/
```

{% endnavtab %}
{% endnavtabs %}

詳細情報
----

* [データプレーンのレジリエンスに関する FAQ](/gateway/latest/kong-enterprise/cp-outage-handling-faq/)
* [ハイブリッドモード](/gateway/latest/production/deployment-topologies/hybrid-mode/)

