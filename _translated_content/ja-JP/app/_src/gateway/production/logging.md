---
title: "ログリファレンス"
---
ログレベル
-----

ログレベルは [Kong の構成](/gateway/{{page.release}}/reference/configuration/#log_level)で設定されます。以下は、重大度の低いものから順に並べたログレベルです。`debug`、`info`、`notice`、`warn`、`error`、および `crit`。

* *`debug`：* プラグインの実行ループと個々のプラグインやその他のコンポーネントに関するデバッグ情報を提供します。これはデバッグ中にのみ使用してください。`debug`オプションを長時間オンのままにしておくと、ディスク領域が過剰に消費される可能性があります。
* *`info` / `notice` :* Kong では、これら 2 つのレベルに大きな違いはありません。通常の動作に関する情報を提供し、そのほとんどは無視できます。
* *`warn`：* トランザクションの中断には至らないものの、さらに調査が必要な異常な動作を記録するには、`warn`レベルを使用する必要があります。
* *`error`:* リクエストが破棄される原因となるエラー （HTTP 500 エラーを受信した場合など）のログを取得するために使用されます。このようなログの発生率を監視する必要があります。
* *`crit`：* このレベルは、Kongが危機的な状況で動作しており、正常に動作していないために複数のクライアントに影響がある場合に使用されます。Nginxでは`alert`レベルと`emerg`レベルも提供されていますが、現在Kongではこれらのレベルは使用されていないため、`crit`が最も重大度の高いログレベルになります。

`notice` は、デフォルトの推奨ログレベルです。ただし、ログが長すぎる場合は、`warn` などのより上位レベルに上げることができます。

Kongログからの特定要素の削除
----------------

GDPR のような個人データの保護に関する新しい規制により、ログ取得の習慣を変更する必要があるかもしれません。Kong を API ゲートウェイとして使用する場合、これを 1 か所で実行して、すべてのサービスに適用できます。このガイドでは、これを達成するための 1 つのアプローチを説明しますが、さまざまなニーズに応じて異なるアプローチが常に存在します。これらの変更は、NGINX アクセスログの出力に影響することに注意してください。これは Kong のログ取得プラグインには影響しません。

この例では、KongログからEメールアドレスのインスタンスをすべて削除するとします。メールアドレスは、たとえば`/servicename/v2/verify/alice@example.com`や`/v3/verify?alice@example.com`のように、さまざまな方法で渡される場合があります。これらがログに追加されないようにするには、カスタムNGINXテンプレートを使用する必要があります。

カスタム NGINX テンプレートの使用を開始するには、まずテンプレートのコピーを入手します。これは[設定プロパティのリファレンス](/gateway/{{page.release}}/reference/configuration/#custom-nginx-templates-embedding-kong)にあるか、以下からコピーできます

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

ログに何を配置するかを制御するために、テンプレートでNGINX mapモジュールを使用します。mapディレクティブの詳細については、[このガイド](http://nginx.org/en/docs/http/ngx_http_map_module.html)を参照してください。これにより、値が最初のパラメータで指定された1つ以上のsource変数の値に依存する、新しい変数が作成されます。形式は次のようになります：

    
    map $parameter_to_look_at $variable_name {
        pattern_to_look_for 0;
        second_pattern_to_look_for 0;
    
        default 1;
    }

この例では、 `$request_uri`に表示される特定の値に依存する`keeplog`という新しい変数をマッピングします。mapディレクティブを httpブロックの先頭に配置します。これは`include 'nginx-kong.conf';`より前である必要があります。この例では、次のような内容を追加します。

    map $request_uri $keeplog {
        ~.+\@.+\..+ 0;
        ~/servicename/v2/verify 0;
        ~/v3/verify 0;
    
        default 1;
    }

これらの行がそれぞれチルダで始まっていることに気付くでしょう。これは、行を評価する場合にNGINXにRegExを使用するように指示するものです。この例では、次の3つの点を確認します。

* 最初の行は正規表現を使用して、 `x@y.z`形式のメールアドレスを検索します
* 2行目は次を含むURIの任意の部分を検索します：`/servicename/v2/verify`
* 3行目は、次を含むURIの任意の部分を参照します：`/v3/verify`

これらはすべて`0`以外の値を持っているので、リクエストにこれらの要素のいずれかが含まれていても、ログには追加されません。

次に、ログに保持するログ形式を設定する必要があります。`log_format`モジュールを使用し、新しいログに`show_everything`という名前を割り当てます。ログの内容は必要に応じてカスタマイズできますが、この例ではすべてをKong標準に戻します。使用できるオプションの完全なリストを確認するには、[このガイド](https://nginx.org/en/docs/http/ngx_http_core_module.html#variables)を参照してください。

    log_format show_everything '$remote_addr - $remote_user [$time_local] '
        '$request_uri $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent"';

これで、カスタム NGINX テンプレートが使用できるようになりました。ここまでの手順に沿っていれば、ファイルは次のようになります。

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

最後に、新しく作成されたログ、`show_everything`を使用するようにKongに指示する必要があります。指示するには、Kong変数`proxy_access_log`を変更するために、`etc/kong/kong.conf`を編集するか、環境変数`KONG_PROXY_ACCESS_LOG`を使用します。次の要素が表示されるようにデフォルトの場所を修正することもお勧めします。

    proxy_access_log=logs/access.log show_everything if=$keeplog

すべての変更を有効にするプロセスの最後の手順は、Kong の再起動です。`kong restart` コマンドを使用して行うことができます。

これで、メールアドレスが含まれたリクエストは記録されなくなります。もちろん、条件に応じてこのロジックを使用し、ログから必要なものを削除することもできます。

