---
title: "ゲートウェイログのカスタマイズ方法"
content_type: "how-to"
---
GDPRのような個人データの保護に関する規制により、{{site.base_gateway}}がログに記録する内容をカスタマイズする必要があるかもしれません。KongをAPIゲートウェイとして使用する場合、これを1か所で実行して、すべてのサービスに適用できます。このガイドでは、これを達成するための1つのアプローチを説明しますが、さまざまなニーズに応じて異なるアプローチが常に存在します。

これらの変更は、NGINXアクセスログの出力に影響します。これはKongのログ取得プラグインには影響しません。

この例では、Kong ログからメールアドレスのインスタンスを削除するとします。メールアドレスは、たとえば `/servicename/v2/verify/alice@example.com` や `/v3/verify?alice@example.com` のように、さまざまな方法で渡される場合があります。これらすべての形式がログに追加されないようにするには、カスタム NGINX テンプレートを使用する必要があります。

まず、{{site.base_gateway}}のNGINXテンプレートのコピーを作成します。このテンプレートは、[Nginxディレクティブリファレンス](/gateway/{{page.release}}/reference/nginx-directives/#custom-nginx-templates-and-embedding-kong-gateway/)で見つけるか、以下からコピーすることができます。

```nginx
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is mandatory
error_log logs/error.log ${{LOG_LEVEL}}; # can be set by kong.conf

events {
    use epoll; # custom setting
    multi_accept on;
}

http {
    # include default Kong Nginx config
    include 'nginx-kong.conf';

    # custom server
    server {
        listen 8888;
        server_name custom_server;

        location / {
          ... # etc
        }
    }
}
```

ログに含める内容を調整するには、このテンプレートで[NGINXマップモジュール](https://nginx.org/en/docs/http/ngx_http_map_module.html)を使用できます。これにより作成される新しい変数の値は、最初のパラメータに指定される1つ以上のsource変数の値に依存します。形式は次のようになります。

```nginx

map $parameter_to_look_at $variable_name {
    pattern_to_look_for 0;
    second_pattern_to_look_for 0;

    default 1;
}
```

この例では、 `$request_uri`に表示される値に依存する`keeplog`という新しい変数をマップします。map ディレクティブを`http`ブロックの先頭、 `include 'nginx-kong.conf';`の前に配置します。

```nginx
map $request_uri $keeplog {
    ~.+\@.+\..+ 0;
    ~/servicename/v2/verify 0;
    ~/v3/verify 0;

    default 1;
}
```

例の各行がチルダで始まっていることに注意してください。これは、行を評価するときに RegEx を使用するように NGINX に指示するものです。この例では、次の 3 つの点に注目してください。

* 最初の行は正規表現を使用して、`x@y.z` 形式のメールアドレスを検索します。
* 2行目は、URIの任意の部分を検索します。`/servicename/v2/verify`
* 3行目は、次を含むURIの任意の部分を参照します：`/v3/verify`

これらのパターンはすべて `0` 以外の値を持っているので、リクエストにこれらの要素のいずれかが含まれていても、ログには追加されません。

ここで、`log_format`モジュールを使用して、{{site.base_gateway}}がログに記録する対象のログ形式を設定します。

ログの内容は、ニーズに合わせてカスタマイズできます。この例では、新しいログに`show_everything`という名前を割り当て、すべてをKongのデフォルト標準に設定するだけです。
オプションの完全なリストを確認するには、[NGINXコアモジュール変数リファレンス](https://nginx.org/en/docs/http/ngx_http_core_module.html#variables)を参照してください。

```nginx
log_format show_everything '$remote_addr - $remote_user [$time_local] '
    '$request_uri $status $body_bytes_sent '
    '"$http_referer" "$http_user_agent"';
```

これで、カスタムNGINXテンプレートが使用できるようになりました。ここまで説明してきたとおりであれば、ファイルは次のようになります。

```nginx
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is mandatory
error_log stderr ${{LOG_LEVEL}}; # can be set by kong.conf



events {
    use epoll; # custom setting
    multi_accept on;
}

http {


    map $request_uri $keeplog {
        ~.+\@.+\..+ 0;
        ~/v1/invitation/ 0;
        ~/reset/v1/customer/password/token 0;
        ~/v2/verify 0;

        default 1;
    }
    log_format show_everything '$remote_addr - $remote_user [$time_local] '
        '$request_uri $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent"';

    include 'nginx-kong.conf';
}
```

最後に、新しく作成されたログ `show_everything` を使用するように {{site.base_gateway}} に指示する必要があります。これを行うには、`etc/kong/kong.conf` を編集するか、環境変数 `KONG_PROXY_ACCESS_LOG` を使用して Kong 変数 `proxy_access_log` を変更します。デフォルトの場所を次のように調整します。

    proxy_access_log=logs/access.log show_everything if=$keeplog

すべての変更を有効にするには、{{site.base_gateway}}を再起動します。`kong restart`コマンドを使用できます。

これで、メールアドレスを含むリクエストは記録されなくなります。

このロジックを使用して、条件に応じてログから必要なものを削除できます。

