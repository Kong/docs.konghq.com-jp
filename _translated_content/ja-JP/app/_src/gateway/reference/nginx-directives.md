---
title: "Nginxディレクティブ"
content-type: "reference"
---
個々のNginxディレクティブの挿入
------------------

`kong.conf`内の`nginx_http_`で始まるエントリ、
`nginx_proxy_`または`nginx_admin_`はNginxディレクティブに変換されます。

* `nginx_http_`という接頭辞が付いたエントリは、
  `http`ブロックディレクティブ全体に挿入されます。

* `nginx_proxy_`という接頭辞が付いたエントリは、{{site.base_gateway}}のプロキシポートを処理する
  `server`ブロックディレクティブに挿入されます。

* `nginx_admin_`という接頭辞が付いたエントリは、{{site.base_gateway}}のAdmin APIポートを処理する
  `server`ブロックディレクティブに挿入されます。

サポートされているすべての名前空間については、[NGINX注入ディレクティブセクション](/gateway/latest/reference/configuration/#nginx-injected-directives-section)を参照してください。

たとえば、次の行を `kong.conf` ファイルに追加するとします。

    nginx_proxy_large_client_header_buffers=16 128k

{{site.base_gateway}}のNginx構成のプロキシ`server`ブロックに次のディレクティブを追加します。

    large_client_header_buffers 16 128k;

これらのディレクティブは、
[環境変数を](/gateway/latest/production/environment-variables/)使用します。たとえば、
次のように環境変数を宣言するとします。

```bash
export KONG_NGINX_HTTP_OUTPUT_BUFFERS="4 64k"
```

その結果、次のNginxディレクティブが`http`ブロックに追加されます。

    output_buffers 4 64k;

Nginxの構成ファイルの構造とブロック
ディレクティブについては、[Nginxリファレンス](https://nginx.org/en/docs/beginners_guide.html#conf_structure)を参照してください。

Nginxディレクティブの一覧については、[Nginxディレクティブインデックス](https://nginx.org/en/docs/dirindex.html)を参照してください。

注入されたNginxディレクティブを介してファイルを含める
-----------------------------

複雑な構成では、Nginx構成に新しい`server`ブロックを追加しなければならない場合があります。
Nginx設定ファイルを指す`include`ディレクティブをNginx構成に注入できます。

たとえば、`my-server.kong.conf`というファイルを次の内容で
作成します。

    # custom server
    server {
      listen 2112;
      location / {
        # ...more settings...
        return 200;
      }
    }

以下のエントリをお客様の`kong.conf`ファイルに追加することで、{{site.base_gateway}}ノードがこのポートに情報を提供するように設定できます。

    nginx_http_include = /path/to/your/my-server.kong.conf

以下のように、環境変数を使用することもできます。

```bash
export KONG_NGINX_HTTP_INCLUDE="/path/to/your/my-server.kong.conf"
```

{{site.base_gateway}}を起動すると、そのファイルの`server`セクションが
そのファイルに追加されます。これは、
通常の{{site.base_gateway}}ポートに加えて定義されたカスタムサーバーが応答することを意味します。

```bash
curl -I http://127.0.0.1:2112
HTTP/1.1 200 OK
...
```

`nginx_http_include`プロパティで相対パスを使用した場合、そのパスは `kong.conf` ファイルの `prefix` プロパティの値（または {{site.base_gateway}} を開始する際に接頭辞を上書きするために使用した場合は `kong start` の `-p` フラグの値）に対して相対的に解釈されます。

カスタムNginxテンプレートと埋め込み {{site.base_gateway}}
--------------------------

次の2つのケースでは、カスタムNginx構成テンプレートを直接使用できます。

* {{site.base_gateway}}のデフォルト
  のNginx構成を変更する必要があります。具体的には、 `kong.conf`で調整できない値については、Nginx 構成を生成するために {{site.base_gateway}} が使用するテンプレートを変更し、カスタマイズしたテンプレートを使用して {{site.base_gateway}} を起動することができます。

* すでに実行中のOpenRestyインスタンスに{{site.base_gateway}}を埋め込む必要があります。
  {{site.base_gateway}}の生成された構成を再利用して既存の構成に含めることができます

### カスタムNginxテンプレート


{{site.base_gateway}}は起動、再読み込み、再起動に`--nginx-conf`引数を使用します。この引数はNginx構成テンプレートで指定する必要があります。
かかるテンプレートで使用される[Penlight](http://stevedonovan.github.io/Penlight/api/index.html)[テンプレートエンジン](http://stevedonovan.github.io/Penlight/api/libraries/pl.template.html)は、{{site.base_gateway}}構成を使用してコンパイルされます。

[テンプレートエンジン](http://stevedonovan.github.io/Penlight/api/libraries/pl.template.html)では次のLua関数が使用できます。

* `pairs`, `ipairs`
* `tostring`
* `os.getenv`


{{site.base_gateway}}のデフォルトテンプレートは、{{site.base_gateway}}インスタンスを実行しているシステム上で、以下のコマンドを使用して見つけることができます：`find / -type d -name "templates" | grep kong`オープンソース{{site.base_gateway}}については、
[テンプレートディレクトリ](https://github.com/kong/kong/tree/master/kong/templates)を参照してください。

Nginx構成ファイルのテンプレートは
`nginx.lua`および`nginx_kong.lua` の2つに分かれています。前者は
最小限かつ後者を含み、 {{site.base_gateway}}の実行に必要なものがすべて含まれています。`kong start`が実行されると、Nginxを起動する直前にこれらの2つのファイルが接頭辞ディレクトリにコピーされ、次のようになります。

    /usr/local/kong
    ├── nginx-kong.conf
    └── nginx.conf

{{site.base_gateway}}で定義されているが`kong.conf`の{{site.base_gateway}}構成を介して調整できないグローバル設定を調整する必要がある場合は、`nginx_kong.lua`構成テンプレートをカスタムテンプレートファイル（この例では
`custom_nginx.template`と呼ばれます）の内容にインライン化できます。

    # ---------------------
    # custom_nginx.template
    # ---------------------
    
    worker_processes ${{ "{{NGINX_WORKER_PROCESSES" }}}}; # can be set by kong.conf
    daemon ${{ "{{NGINX_DAEMON" }}}};                     # can be set by kong.conf
    
    pid pids/nginx.pid;                      # this setting is mandatory
    error_log logs/error.log ${{ "{{LOG_LEVEL" }}}}; # can be set by kong.conf
    
    events {
        use epoll;          # a custom setting
        multi_accept on;
    }
    
    http {
    
      # contents of the nginx_kong.lua template follow:
    
      resolver ${{ "{{DNS_RESOLVER" }}}} ipv6=off;
      charset UTF-8;
      error_log logs/error.log ${{ "{{LOG_LEVEL" }}}};
      access_log logs/access.log;
    
      ... # etc
    }

その後、以下のように {{site.base_gateway}} を開始できます。

```bash
kong start -c kong.conf --nginx-conf custom_nginx.template
```

詳細情報
----

* [使用方法 `kong.conf`](/gateway/{{page.release}}/production/kong-conf/)
* [KongでAPIとウェブサイトを提供する方法](/gateway/{{page.release}}/production/website-api-serving/) {%- if_version gte:3.6.x %}
* [`ngx_brotli`モジュール経由でBrotli圧縮を有効にする](/gateway/{{page.release}}/production/performance/brotli/) {% endif_version %}

