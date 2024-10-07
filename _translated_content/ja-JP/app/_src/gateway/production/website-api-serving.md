---
title: "Kong GatewayでウェブサイトとAPIを提供する"
content-type: "how-to"
---
{{site.base_gateway}}を使用してウェブサイトとAPIの両方を提供する方法
------------------------------

APIプロバイダーの一般的なユースケースは、{{site.base_gateway}}にウェブサイトとポート経由の
APIのどちらも提供できるようにすることです。`80`または本番環境の`443`
です。例えば、`https://example.net`（ウェブサイト）と
`https://example.net/api/v1`（API）です。

これを行うには、`nginx_kong.lua`を一列で呼び出すカスタムNginx構成テンプレートを使用して、Kongプロキシ`location`ブロックと並行してウェブサイトを処理する新しい`location`ブロックを追加します。

    # ---------------------
    # custom_nginx.template
    # ---------------------
    
    worker_processes ${{ "{{NGINX_WORKER_PROCESSES" }}}}; # can be set by kong.conf
    daemon ${{ "{{NGINX_DAEMON" }}}};                     # can be set by kong.conf
    
    pid pids/nginx.pid;                      # this setting is mandatory
    error_log logs/error.log ${{ "{{LOG_LEVEL" }}}}; # can be set by kong.conf
    events {}
    
    http {
      # here, we inline the contents of nginx_kong.lua
      charset UTF-8;
    
      # any contents until Kong's Proxy server block
      ...
    
      # Kong's Proxy server block
      server {
        server_name kong;
    
        # any contents until the location / block
        ...
    
        # here, we declare our custom location serving our website
        # (or API portal) which we can optimize for serving static assets
        location / {
          root /var/www/example.net;
          index index.htm index.html;
          ...
        }
    
        # Kong's Proxy location / has been changed to /api/v1
        location /api/v1 {
          set $upstream_host nil;
          set $upstream_scheme nil;
          set $upstream_uri nil;
    
          # Any remaining configuration for the Proxy location
          ...
        }
      }
    
      # Kong's Admin server block goes below
      # ...
    }

次に、Nginxを起動します。

`nginx -p /usr/local/openresty -c my_nginx.conf`

その他の情報
------

* [環境変数の設定](/gateway/latest/production/environment-variables/)
* [使用方法`kong.conf`](/gateway/latest/production/kong-conf/)

