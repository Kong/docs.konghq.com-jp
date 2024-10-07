---
title: "Admin APIの保護"
---

{{site.base_gateway}} の Admin API は、サービス、ルート、プラグイン、コンシューマ、および認証情報の管理と設定のための RESTful インターフェイスを提供します。この API を使うと Kong を完全にコントロールすることが可能なため、望ましくないアクセスからこの API を保護することが重要です。このドキュメントでは、Admin API を保護するためのいくつかの方法について説明します。

ネットワーク層のアクセス制限
--------------

### 最小限のリスニングフットプリント

0\.12\.0リリース以降のデフォルトでは、Kongはデフォルトの`admin_listen`値で指定されているローカルインターフェースからのリクエストのみを受け入れます。

    admin_listen = 127.0.0.1:8001

この値を変更する場合は、Admin APIを第三者に公開しないようにするため、リスニングフットプリントを最低限にしてください。
Kongクラスタ全体のセキュリティが深刻な危険にさらされる可能性があります。
たとえば、 **Kongをすべてのインターフェースにバインドしないようにするためには** 、
`0.0.0.0:8001`などの値を使用します。

### レイヤー 3/4 ネットワーク制御

Admin API をローカルホストのインターフェース以外で公開する必要がある場合、
ネットワークセキュリティのベストプラクティスに従い、ネットワーク層でのアクセスをできるだけ制限する
ことが求められます。Kong がプライベート
ネットワークインターフェースでリッスンしているが、特定の IP 範囲の一部からのみアクセスができる環境を考えてみてください。
このような場合、ホストベースのファイアウォール（例: iptables）は入力トラフィックの範囲を制限する
のに便利です。以下はその例です。

```bash
# assume that Kong is listening on the address defined below, as defined as a
# /24 CIDR block, and only a select few hosts in this range should have access

grep admin_listen /etc/kong/kong.conf
admin_listen 10.10.10.3:8001

# explicitly allow TCP packets on port 8001 from the Kong node itself
# this is not necessary if Admin API requests are not sent from the node
iptables -A INPUT -s 10.10.10.3 -m tcp -p tcp --dport 8001 -j ACCEPT

# explicitly allow TCP packets on port 8001 from the following addresses
iptables -A INPUT -s 10.10.10.4 -m tcp -p tcp --dport 8001 -j ACCEPT
iptables -A INPUT -s 10.10.10.5 -m tcp -p tcp --dport 8001 -j ACCEPT

# drop all TCP packets on port 8001 not in the above IP list
iptables -A INPUT -m tcp -p tcp --dport 8001 -j DROP

```

ネットワークデバイスレベルで適用される同様のACLなどの追加の制御は
推奨されますが、このドキュメントの範囲外です。

Kong APIループバック
--------------

Kong のルーティング設計により、Admin API 自体のプロキシとして機能することができます。このようにして、Kong 自体を使用して、Admin API へのきめ細かいアクセス制御を提供できます。このような環境では、`admin_listen` アドレスをサービスの `url` として定義する新しいサービスをブートストラップする必要があります。

たとえば、Kong`admin_listen`が`127.0.0.1:8001`であるため、localhostからのみ利用できるとします。ポート `8000`はプロキシトラフィックを提供しており、おそらく<content: myhost.dev:8000>経由で公開されています。`myhost.dev:8000`

Admin APIを、制御された方法でURL`:8000/admin-api`経由で公開したいと考えています。そのためには
`127.0.0.1`内からサービスとルートを作成します。

```bash
curl -X POST http://127.0.0.1:8001/services \
  --data name=admin-api \
  --data host=127.0.0.1 \
  --data port=8001

curl -X POST http://127.0.0.1:8001/services/admin-api/routes \
  --data paths[]=/admin-api
```

外部`127.0.0.1`のプロキシサーバーを介して透過的にAdmin APIにアクセスできるようになりました。

```bash
curl myhost.dev:8000/admin-api/services
{
   "data":[
      {
        "id": "653b21bd-4d81-4573-ba00-177cc0108dec",
        "created_at": 1422386534,
        "updated_at": 1422386534,
        "name": "admin-api",
        "retries": 5,
        "protocol": "http",
        "host": "127.0.0.1",
        "port": 8001,
        "path": "/admin-api",
        "connect_timeout": 60000,
        "write_timeout": 60000,
        "read_timeout": 60000
      }
   ],
   "total":1
}
```

ここからは、他のKong APIに通常行うのと同じように、必要なKong固有のセキュリティコントロール（[Basic Authentication](/hub/kong-inc/basic-auth)または[Key Authentication](/hub/kong-inc/key-auth)、[IP制限](/hub/kong-inc/ip-restriction)、[アクセス制御リスト](/hub/kong-inc/acl)など）を適用するだけです。

{{site.ee_product_name}}のホストにDockerを使用している場合は、次のような宣言型構成を使用して同様のタスクを実行できます。

```yaml
_format_version: "3.0"

services:
- name: admin-api
  url: http://127.0.0.1:8001
  routes:
    - paths:
      - /admin-api
  plugins:
  - name: key-auth

consumers:
- username: admin
  keyauth_credentials:
  - key: secret
```

この構成では、Admin APIは`/admin-api`を通じて利用可能になりますが、`?apikey=secret`クエリパラメータを伴うリクエストに対してのみ
利用可能です。

上記のファイルが`$(pwd)/kong.yml`に保存されていると仮定すると、DBレスの{{site.ee_product_name}}は次のように起動してそれを使用できます。

```bash
docker run -d --name kong-ee \
    -e "KONG_DATABASE=off" \
    -e "KONG_DECLARATIVE_CONFIG=/home/kong/kong.yml"
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    -v $(pwd):/home/kong
    kong-ee
```

PostgreSQLデータベースの場合、初期化手順は次のようになります。

```bash
# Start PostgreSQL on a Docker container
# Notice that PG_PASSWORD needs to be set
docker run --name kong-ee-database \
    -p 5432:5432 \
    -e "POSTGRES_USER=kong" \
    -e "POSTGRES_DB=kong" \
    -e "POSTGRES_PASSWORD=$PG_PASSWORD" \
    -d postgres:9.6

# Run Kong migrations to initialize the database
docker run --rm \
    --link kong-ee-database:kong-ee-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-ee-database" \
    -e "KONG_PG_PASSWORD=$PG_PASSWORD" \
    kong-ee kong migrations bootstrap

# Load the configuration file which enables the Admin API loopback
# Notice that it is assumed that kong.yml is located in $(pwd)/kong.yml
docker run --rm \
    --link kong-ee-database:kong-ee-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-ee-database" \
    -e "KONG_PG_PASSWORD=$PG_PASSWORD" \
    -v $(pwd):/home/kong \
    kong-ee kong config db_import /home/kong/kong.yml

# Start Kong
docker run -d --name kong \
    --link kong-ee-database:kong-ee-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-ee-database" \
    -e "KONG_PG_PASSWORD=$PG_PASSWORD" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    kong-ee
```

どちらの場合も、Kongが起動して実行されると、Admin APIは使用可能になりますが保護されます。

```bash
curl myhost.dev:8000/admin-api/services
=> HTTP/1.1 401 Unauthorized

curl myhost.dev:8000/admin-api/services?apikey=secret
=> HTTP/1.1 200 OK
{
    "data": [
        {
            "ca_certificates": null,
            "client_certificate": null,
            "connect_timeout": 60000,
        ...
      }
    ]
}
```

カスタムNginx構成
-----------

KongはHTTPデーモンとしてNginxと密接に結合されており、カスタムNginx構成の環境に
統合することができます。このように、複雑なセキュリティ/アクセス制御要件のあるユースケースでは、Nginx/OpenResty力をフルに活用して、必要なAdmin APIを格納するサーバー/ロケーションブロックを構築できます。これにより、カスタム/複雑なセキュリティ制御を構築できるOpenResty環境を提供できるのに加え、
このような環境では、ネイティブのNginx認証、認証メカニズム、ACLモジュールなどを
活用できるようになります。

カスタム Nginx 構成への Kong の統合の詳細については、[カスタム Nginx 構成と Kong の埋め込み](/gateway/{{page.release}}/reference/nginx-directives)に関するページを参照してください。

ロールベースのアクセス制御
-------------
{:.badge .enterprise}


{{site.base_gateway}} ユーザーは、[ロールベースのアクセスコントロール](/gateway/{{page.release}}/production/access-control/enable-rbac/)を構成して、Admin API へのアクセスを保護できます。RBAC を使用すると、ユーザーのロールと権限のモデルに基づいて、リソースアクセスをきめ細かく制御できます。ユーザーは 1 つ以上のロールに割り当てられ、各ロールは、特定のリソースへのアクセスを許可または拒否する 1 つ以上の権限を持ちます。このようにして、複雑でケース固有の使用を許可するようにスケーリングしながら、特定の Admin API リソースに対するきめ細かい制御を適用できます。

