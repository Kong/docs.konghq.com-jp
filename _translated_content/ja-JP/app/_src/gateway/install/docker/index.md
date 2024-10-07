---
title: "Docker に Kong Gateway をインストールする"
konnect_cta_card: true
---
このガイドでは、データベースの有無にかかわらず Docker 上で {{site.base_gateway}} を構成する手順について説明します。
このガイドで使用するデータベースは PostgreSQL です。

オープンソースの{{site.base_gateway}}イメージをDocker Composeで使用したい場合、Kongはオーケストレーションとスケーラビリティを組み込んだ[Docker Composeテンプレート](https://github.com/Kong/docker-kong/tree/master/compose)も提供しています。

一部の[古い{{site.base_gateway}}イメージ](https://support.konghq.com/support/s/article/Downloading-older-Kong-versions)は
一般に公開されていません。特定のパッチバージョンが必要で、[Kongの
パブリックDocker Hubページ](https://hub.docker.com/r/kong/kong-gateway)から見つけられない場合は、[Kongサポート](https://support.konghq.com/)に
お問い合わせください。

{{site.base_gateway}} ソフトウェアは、
[Kongソフトウェアライセンス契約](https://konghq.com/kongsoftwarelicense)で管理されています。

{{site.ce_product_name}} は、
[Apache 2\.0ライセンス](https://github.com/Kong/kong/blob/master/LICENSE)で
ライセンスされています。

前提条件
----

{:.note}
> 
> **注：**
> コントロールプレーン（CP）またはデータベースを管理せずに{{ site.base_gateway }}を実行したい場合は、Dockerクイックスタートスクリプトを使用すると5分以内に[Konnectを使い始めることができます](https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=docker-install)。

* 適切な Docker アクセスを備えた Docker 対応システム
* （Enterpriseのみ）Kongの`license.json`ファイル

{{site.base_gateway}}のインストールパスを次のとおり選択します。

* [データベースで](#install-kong-gateway-with-a-database)：データベースを使って Kongエンティティ構成を保存します。Kongの構成に、Admin APIまたは宣言型構成 ファイルを使用できます。
* [データベース不使用（DB レスモード）](#install-kong-gateway-in-db-less-mode): Kong 構成をノードのメモリ内に保存します。このモードでは、Admin API は読み取り専用であり、宣言的構成を使用して Kong を管理する必要があります。

どのオプションを使用するか分からない場合は、[データベース使用](#install-kong-gateway-with-a-database)から始めることをお勧めします。

データベースを使用した{{site.base_gateway}}のインストール
-----------------------

PostgreSQL データベースを使用して {{site.base_gateway}} コンテナを設定し、Kong の構成を保存します。

{% if_version lte:3.3.x %}
{% include_cached /md/enterprise/cassandra-deprecation.md length='short' release=page.release %}
{% endif_version %}

### データベースを準備

1. コンテナがお互いを検出して通信できるように、カスタムDockerネットワークを作成します。

   ```sh
   docker network create kong-net
   ```

   このネットワークには、任意の名前を付けることができます。このガイド全体を通して `kong-net` を例として使用します。
2. PostgreSQLコンテナを起動します。

   ```sh
   docker run -d --name kong-database \
    --network=kong-net \
    -p 5432:5432 \
    -e "POSTGRES_USER=kong" \
    -e "POSTGRES_DB=kong" \
    -e "POSTGRES_PASSWORD=kongpass" \
    postgres:13
   ```

   * `POSTGRES_USER`および`POSTGRES_DB`：これらの値を `kong`に設定します。これは {{site.base_gateway}}が想定するデフォルトの値です。
   * `POSTGRES_PASSWORD`: データベースのパスワードを任意の文字列に設定します。

   この例では、`kong-database`という名前のPostgresコンテナは `kong-net`ネットワーク上の任意のコンテナと通信します。
3. Kongデータベースを準備します。

{% capture migrations %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```sh
docker run --rm --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PASSWORD=test" \
kong/kong-gateway:{{page.versions.ee}} kong migrations bootstrap
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```sh
docker run --rm --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_PASSWORD=kongpass" \
kong:{{page.versions.ce}} kong migrations bootstrap
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}

{{ migrations | indent | replace: " </code>", "</code>" }}

    Where:
    * [`KONG_DATABASE`](/gateway/{{page.release}}/reference/configuration/#database):
     Specifies the type of database that Kong is using.
    * [`KONG_PG_HOST`](/gateway/{{page.release}}/reference/configuration/#postgres-settings):
    The name of the Postgres Docker container that is communicating over the
    `kong-net` network, from the previous step.
    * [`KONG_PG_PASSWORD`](/gateway/{{page.release}}/reference/configuration/#postgres-settings):
    The password that you set when bringing up the Postgres container in the
    previous step.
    * `KONG_PASSWORD` (Enterprise only): The default password for the admin
    super user for Kong Gateway.
    * `{IMAGE-NAME:TAG} kong migrations bootstrap`:
    In order, this is the Kong Gateway container name and tag, followed by the
    command to Kong to prepare the Postgres database.

### {{site.base_gateway}}の起動

{% include_cached /md/admin-listen.md desc='long' release=page.release %}

1. （任意）{{site.base_gateway}}のEnterpriseライセンスをお持ちの場合は、
   ライセンスキーを変数にエクスポートします。

   ライセンスデータが有効なJSONと見なされるためには、まっすぐの引用符が含まれている必要があります（`'`と`"`で、`’`や`“`ではありません）。

   {:.note}
   > 
   > **注:**
   > 以下のライセンスは一例です。以下の形式を使用する必要がありますが、内容は各自で提供してください。

   ```bash
   export KONG_LICENSE_DATA='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-20","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b","version":"1"}}'
   ```

<!-- -->
1. {{site.base_gateway}}を使用してコンテナを起動するには、次のコマンドを実行します。 {% capture start_container %} {% navtabs_ee codeblock %} {% navtab Kong Gateway %}

```sh
docker run -d --name kong-gateway \
 --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_USER=kong" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
 -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
 -e KONG_LICENSE_DATA \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 8001:8001 \
 -p 8444:8444 \
 -p 8002:8002 \
 -p 8445:8445 \
 -p 8003:8003 \
 -p 8004:8004 \
 kong/kong-gateway:{{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}
{% if_version lte:3.3.x %}

```sh
docker run -d --name kong-gateway \
 --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_USER=kong" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 127.0.0.1:8001:8001 \
 -p 127.0.0.1:8444:8444 \
 kong:{{page.versions.ce}}
```

{% endif_version %}
{% if_version gte:3.4.x %}

```sh
docker run -d --name kong-gateway \
 --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_USER=kong" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
 -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 127.0.0.1:8001:8001 \
 -p 127.0.0.1:8002:8002 \
 -p 127.0.0.1:8444:8444 \
 kong:{{page.versions.ce}}
```

{% endif_version %}
{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}

{{ start_container | indent | replace: " </code>", "</code>" }}

    Where:
    * `--name` and `--network`: The name of the container to create,
    and the Docker network it communicates on.
    * [`KONG_DATABASE`](/gateway/{{page.release}}/reference/configuration/#database):
    Specifies the type of database that Kong is using.
    * [`KONG_PG_HOST`](/gateway/{{page.release}}/reference/configuration/#postgres-settings):
    The name of the Postgres Docker container that is communicating over the
    `kong-net` network.
    * [`KONG_PG_USER` and `KONG_PG_PASSWORD`](/gateway/{{page.release}}/reference/configuration/#postgres-settings):
     The Postgres username and password. Kong Gateway needs the login information
     	to store configuration data in the `KONG_PG_HOST` database.
    * All [`_LOG`](/gateway/{{page.release}}/reference/configuration/#general-section)
    parameters: set filepaths for the logs to output to, or use the values in
    the example to  print messages and errors to `stdout` and `stderr`.
    * [`KONG_ADMIN_LISTEN`](/gateway/{{page.release}}/reference/configuration/#admin_listen):
    The port that the Kong Admin API listens on for requests.
    * [`KONG_ADMIN_GUI_URL`](/gateway/{{page.release}}/reference/configuration/#admin_gui_url): {% if_version lte:3.3.x %}(Not available in OSS) {% endif_version %}The URL for accessing Kong Manager, preceded by a protocol
    (for example, `http://`).
    * `KONG_LICENSE_DATA`: (Enterprise only) If you have a license file and have saved it
    as an environment variable, this parameter pulls the license from your environment.

1. インストールを確認します。

   Admin APIを使用して`/services`エンドポイントにアクセスします。

   ```sh
   curl -i -X GET --url http://localhost:8001/services
   ```

   `200`ステータスコードが表示されます。

{% if_version lte:3.3.x %}

1. \(OSSでは利用できません\)`KONG_ADMIN_GUI_URL` で指定された URL を使用し、Kong Managerにアクセスして実行されていることを確認します。

       http://localhost:8002

{% endif_version %}
{% if_version gte:3.4.x %}

1. `KONG_ADMIN_GUI_URL`で指定された
   URL を使用し、Kong Managerにアクセスして実行されていることを確認します。

       http://localhost:8002

{% endif_version %}

### {{site.base_gateway}}を使い始める

ここまででゲートウェイインスタンスが稼働しました。ここからは、Kongの一連の
[入門ガイド](/gateway/{{page.release}}/get-started/services-and-routes/)が
最初のサービスの設定と強化に役立ちます。

特にインストール直後には、以下を参照してください。

* [サービスとルートを作成する](/gateway/{{page.release}}/get-started/services-and-routes/)
* [プラグインを設定する](/gateway/{{page.release}}/get-started/rate-limiting/)
* [認証によるサービスの保護](/gateway/{{page.release}}/get-started/key-authentication/)

### コンテナのクリーンアップ

{{site.base_gateway}}テストが終了し、コンテナが不要になった場合は、次のコマンドを使用してクリーンアップできます。

    docker kill kong-gateway
    docker kill kong-database
    docker container rm kong-gateway
    docker container rm kong-database
    docker network rm kong-net

{{site.base_gateway}} を DB レスモードでインストールする
--------------------------

次のステップでは、[DBレスモード](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config)で {{site.base_gateway}}を開始する方法について説明します。

### Dockerネットワークを作成する

次のコマンドを実行します。

```bash
docker network create kong-net
```

このネットワークには、任意の名前を付けることができます。このガイド全体を通して `kong-net` を例として使用します。

このステップは、DBレスモードでKongを実行する際には厳密には必要ありませんが、将来的に他のもの（RedisクラスターでバックアップされたRate Limitingプラグインなど）を追加したい場合に備えておくとよいでしょう。

### 設定ファイルを準備する

1. `.yml`または`.json`形式で宣言型構成ファイルを準備します。

   構文とプロパティは
   [宣言型構成フォーマットガイド](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config/#the-declarative-configuration-format)で説明されています。このファイルに必要な
   任意のコアエンティティ（サービス、ルート、プラグイン、コンシューマなど）を追加します。

   たとえば、サービスとルートの簡単なファイルは以下のようになります。

   ```yaml
   _format_version: "3.0"
   _transform: true
   
   services:
   - host: httpbin.org
     name: example_service
     port: 80
     protocol: http
     routes:
     - name: example_route
       paths:
       - /mock
       strip_path: true
   ```

   このガイドでは、ファイルに`kong.yml`という名前を付けたと想定しています。
2. 宣言的設定をローカルに保存し、ファイルパスに注意してください。

### DBレスモードで{{site.base_gateway}}を起動します

{% include_cached /md/admin-listen.md desc='long' release=page.release %}

1. （任意）{{site.base_gateway}}のEnterpriseライセンスをお持ちの場合は、
   ライセンスキーを変数にエクスポートします。

   ライセンスデータが有効なJSONと見なされるためには、まっすぐの引用符が含まれている必要があります（`'`と`"`で、`’`や`“`ではありません）。

   {:.note}
   > 
   > **注:**
   > 以下のライセンスは一例です。以下の形式を使用する必要がありますが、内容は各自で提供してください。

   ```bash
   export KONG_LICENSE_DATA='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-20","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b","version":"1"}}'
   ```

2. `kong.yml` ファイルを作成したディレクトリで、次のコマンドを実行して {{site.base_gateway}} を含むコンテナを起動します。

{% capture start_container %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```sh
docker run -d --name kong-dbless \
 --network=kong-net \
 -v "$(pwd):/kong/declarative/" \
 -e "KONG_DATABASE=off" \
 -e "KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
 -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
 -e KONG_LICENSE_DATA \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 8001:8001 \
 -p 8444:8444 \
 -p 8002:8002 \
 -p 8445:8445 \
 -p 8003:8003 \
 -p 8004:8004 \
 kong/kong-gateway:{{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}
{% if_version lte:3.3.x %}

```sh
docker run -d --name kong-dbless \
 --network=kong-net \
 -v "$(pwd):/kong/declarative/" \
 -e "KONG_DATABASE=off" \
 -e "KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 127.0.0.1:8001:8001 \
 -p 127.0.0.1:8444:8444 \
 kong:{{page.versions.ce}}
```

{% endif_version %}
{% if_version gte:3.4.x %}

```sh
docker run -d --name kong-dbless \
 --network=kong-net \
 -v "$(pwd):/kong/declarative/" \
 -e "KONG_DATABASE=off" \
 -e "KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
 -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 127.0.0.1:8001:8001 \
 -p 127.0.0.1:8444:8444 \
 kong:{{page.versions.ce}}
```

{% endif_version %}
{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}

{{ start_container | indent | replace: " </code>", "</code>" }}

    Where:
    * `--name` and `--network`: The name of the container to create,
    and the Docker network it communicates on.
    * `-v $(pwd):/path/to/target/`: Mount the current directory on your
    local filesystem to a directory in the Docker container. This makes the
    `kong.yml` file visible from the Docker container.
    * [`KONG_DATABASE`](/gateway/{{page.release}}/reference/configuration/#database):
     Sets the database to `off` to tell Kong not to use any
    backing database for configuration storage.
    * [`KONG_DECLARATIVE_CONFIG`](/gateway/{{page.release}}/reference/configuration/#declarative_config):
    The path to a declarative configuration file inside the container.
    This path should match the target path that you're mapping with `-v`.
    * All [`_LOG`](/gateway/{{page.release}}/reference/configuration/#general-section)
    parameters: set filepaths for the logs to output to, or use the values in
    the example to  print messages and errors to `stdout` and `stderr`.
    * [`KONG_ADMIN_LISTEN`](/gateway/{{page.release}}/reference/configuration/#admin_listen):
    The port that the Kong Admin API listens on for requests.
    * [`KONG_ADMIN_GUI_URL`](/gateway/{{page.release}}/reference/configuration/#admin_gui_url): {% if_version lte:3.3.x %}(Not available in OSS) {% endif_version %}The URL for accessing Kong Manager, preceded by a protocol
    (for example, `http://`).
    * `KONG_LICENSE_DATA`: (Enterprise only) If you have a license file and have saved it
    as an environment variable, this parameter pulls the license from your environment.

1. {{site.base_gateway}} が実行されていることを確認します。

   ```sh
   curl -i http://localhost:8001
   ```

   エンドポイントをテストします。たとえば、サービスの一覧を取得します。

   ```sh
   curl -i http://localhost:8001/services
   ```

### {{site.base_gateway}}を使い始める

ここまででゲートウェイインスタンスが稼働しました。ここからは、Kongの一連の
[入門ガイド](/gateway/{{page.release}}/get-started/services-and-routes/)が
最初のサービスの設定と強化に役立ちます。

このガイドのサンプル`kong.yml`を使用した場合、サービスとルートはすでに構成されています。他にも以下をチェックしてみてください。

* [プラグインを設定する](/gateway/{{page.release}}/get-started/rate-limiting?tab=using-deck-yaml/)
* [認証によるサービスの保護](/gateway/{{page.release}}/get-started/key-authentication?tab=using-deck-yaml/)
* [ターゲット間でのトラフィックのロードバランシング](/gateway/{{page.release}}/get-started/load-balancing/?tab=using-deck-yaml/)

### コンテナのクリーンアップ

{{site.base_gateway}}テストが終了し、コンテナが不要になった場合は、次のコマンドを使用してクリーンアップできます。

    docker kill kong-dbless
    docker container rm kong-dbless
    docker network rm kong-net

Kongを読み取り専用モードで実行
-----------------

{{site.base_gateway}} 3\.2\.0から、コンテナを読み取り専用モードで実行できます。これを行うには、Kongがデータを書き込む必要がある場所にDockerボリュームをマウントします。デフォルトの構成では、`/tmp`と接頭辞パスへの書き込みアクセスが必要です。

{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```sh
docker run --read-only -d --name kong-dbless \
 --network=kong-net \
 -v "$(pwd)/declarative:/kong/declarative/" \
 -v "$(pwd)/tmp_volume:/tmp" \
 -v "$(pwd)/prefix_volume:/var/run/kong" \
 -e "KONG_PREFIX=/var/run/kong" \
 -e "KONG_DATABASE=off" \
 -e "KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
 -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
 -e KONG_LICENSE_DATA \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 8001:8001 \
 -p 8444:8444 \
 -p 8002:8002 \
 -p 8445:8445 \
 -p 8003:8003 \
 -p 8004:8004 \
 kong/kong-gateway:{{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```sh
docker run --read-only -d --name kong-dbless \
 --network=kong-net \
 -v "$(pwd)/declarative:/kong/declarative/" \
 -v "$(pwd)/tmp_volume:/tmp" \
 -v "$(pwd)/prefix_volume:/var/run/kong" \
 -e "KONG_PREFIX=/var/run/kong" \
 -e "KONG_DATABASE=off" \
 -e "KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 127.0.0.1:8001:8001 \
 -p 127.0.0.1:8444:8444 \
 kong:{{page.versions.ce}}
```

{% endnavtab %}
{% endnavtabs_ee %}

トラブルシューティング
-----------

ライセンスの問題のトラブルシューティングについては、以下を参照してください。

* [ライセンスのデプロイメントオプション](/gateway/{{page.release}}/licenses/deploy/)
* [`/licenses` API リファレンス](/gateway/{{page.release}}/admin-api/licenses/reference/)
* [`/licenses` APIの例](/gateway/{{page.release}}/licenses/examples/)

`200 OK`ステータスコードを受診しなかった場合、またはセットアップの完了にサポートが必要な場合は、サポート担当者に問い合わせるか、[サポートポータル](https://support.konghq.com/support/s/)にアクセスしてください。

