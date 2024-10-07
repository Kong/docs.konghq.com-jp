---
title: "systemdによるKongゲートウェイの制御"
---
このドキュメントんいは、DebianのsystemdとRPMベースのパッケージで{{site.base_gateway}}を統合するための手順が記載されています。

{{site.base_gateway}}でサポートされている GNU/Linux ディストリビューションの一部には、
systemdをデフォルトのinitシステムとして採用していない可能性があります
\(例: CentOS 6 および RHEL 6\)。次の説明では
{{site.base_gateway}} は、すでにシステムでサポートされる
GNU/Linuxディストリビューションに
[インストールされ、設定済み](/gateway/{{page.release}}/install)
であると仮定されています。

{{site.base_gateway}}を操作するためのsystemdコマンド
------------------------

### {{site.base_gateway}}の起動

```bash
# For {{site.base_gateway}}
sudo systemctl start kong-enterprise-edition

# For {{site.ce_product_name}}
sudo systemctl start kong
```

### 停止{{site.base_gateway}}

```bash
# For {{site.base_gateway}}
sudo systemctl stop kong-enterprise-edition

# For {{site.ce_product_name}}
sudo systemctl stop kong
```

### システム起動時に{{site.base_gateway}}起動する

**システム起動時に{{site.base_gateway}}の自動開始を有効にします** 

```bash
# For {{site.base_gateway}}
sudo systemctl enable kong-enterprise-edition

# For {{site.ce_product_name}}
sudo systemctl enable kong
```

**システム起動時に{{site.base_gateway}}を自動的に開始しないようにします** 

```bash
# For {{site.base_gateway}}
sudo systemctl disable kong-enterprise-edition

# For {{site.ce_product_name}}
sudo systemctl disable kong
```

### Restart {{site.base_gateway}}

```bash
# For {{site.base_gateway}}
sudo systemctl restart kong-enterprise-edition

# For {{site.ce_product_name}}
sudo systemctl restart kong
```

### Query {{site.base_gateway}} status

```bash
# For {{site.base_gateway}}
sudo systemctl status kong-enterprise-edition

# For {{site.ce_product_name}}
sudo systemctl status kong
```

{{site.base_gateway}}ユニットファイルをカスタマイズ
---------------------

公式のsystemdサービスは
{{site.base_gateway}}用は`/lib/systemd/system/kong-enterprise-edition.service`に、
{{site.ce_product_name}}の場合は`/lib/systemd/system/kong.service`にあります。

公式のKong systemdサービスでは、`user`または`group`を`root`として設定しませんが、システムファイルの実行には設定が必要です。`user`と`group`の両方が、`root`に設定されている以下の例を参照してください。

    [Unit]
    Description=Kong Gateway
    Documentation=https://docs.konghq.com/gateway/
    After=syslog.target network.target remote-fs.target nss-lookup.target
    [Service]
    User=root
    Group=root
    ExecStartPre=/usr/local/bin/kong prepare -p /usr/local/kong -c /home/ec2-user/kong.conf
    ExecStart=/usr/local/openresty/nginx/sbin/nginx -p /usr/local/kong -c nginx.conf
    ExecReload=/usr/local/bin/kong prepare -p /usr/local/kong
    ExecReload=/usr/local/openresty/nginx/sbin/nginx -p /usr/local/kong -c nginx.conf -s reload
    ExecStop=/bin/kill -s QUIT $MAINPID
    PrivateTmp=true
    # All environment variables prefixed with KONG_ and capitalized will override
    # the settings specified in the /etc/kong/kong.conf.default file.
    #
    # For example:
    #   log_level = debug in the .conf file -> KONG_LOG_LEVEL=debug env var.
    Environment=KONG_NGINX_DAEMON=off
    # You can control this limit through /etc/security/limits.conf
    LimitNOFILE=infinity
    [Install]
    WantedBy=multi-user.target

カスタマイズが必要なシナリオ（たとえば、Kong の構成やサービスファイルの動作の変更）では、Kong の再インストールまたはアップグレード時の競合を避けるために、
{{site.base_gateway}} の場合は `/etc/systemd/system/kong-enterprise-edition.service`、
{{site.ce_product_name}} の場合は `/etc/systemd/system/kong.service` で別のサービスを作成することをお勧めします。

接頭辞が大文字の`KONG_`であるすべての環境変数は、`/etc/kong/kong.conf.default`ファイルで指定されている設定を上書きします。例：.confの`log_level = debug`ファイルは、`KONG_LOG_LEVEL=debug`環境変数に変換されます。

環境変数の代わりに構成ファイルを使用することも選択できます。この場合、`ExecStartPre` systemdディレクティブを変更して、`-c`引数で構成ファイルを指して`kong prepare`を実行します。たとえば、`/etc/kong/kong.conf`にカスタム構成ファイルがある場合は、`ExecStartPre`ディレクティブを次のように変更します。

    ExecStartPre=/usr/local/bin/kong prepare -p /usr/local/kong -c /etc/kong/kong.conf

`EnvironmentFile` systemdディレクティブを使用して非環境ファイルをリンクする場合、systemdパーサーは環境変数の割り当てのみを認識することに注意してください。たとえば、Kong のデフォルト設定ファイルの1つ（ `/etc/kong/kong.conf.default`と`/etc/kong.conf` ）がリンクされている場合、ファイル内の環境変数以外の割り当てによってsystemdエラーが発生する可能性があります。この場合、systemdはKongサービスの起動を許可しません。このため、デフォルト以外の`EnvironmentFile`を指定することをお勧めします。

    EnvironmentFile=/etc/kong/kong_env.conf

### syslogとjournaldへのロギング

この場合は、`/etc/systemd/system/kong-enterprise-edition.service` にあるカスタマイズされた systemd サービスファイルに、次の `Environment` systemd ディレクティブを追加することで対応できます。

    Environment=KONG_PROXY_ACCESS_LOG=syslog:server=unix:/dev/log
    Environment=KONG_PROXY_ERROR_LOG=syslog:server=unix:/dev/log
    Environment=KONG_ADMIN_ACCESS_LOG=syslog:server=unix:/dev/log
    Environment=KONG_ADMIN_ERROR_LOG=syslog:server=unix:/dev/log

journald ログを表示するには、次のようにします。

```bash
# For {{site.base_gateway}}
journalctl -u kong-enterprise-edition

# For {{site.ce_product_name}}
journalctl -u kong
```

以下の方法で、syslogログを表示します。

```bash
tail -F /var/log/syslog
```

### Nginx ディレクティブ インジェクション システムを使用して Kong の Nginx インスタンスをカスタマイズする

環境変数で[インジェクションシステム](/gateway/{{page.release}}/reference/configuration/#injecting-individual-nginx-directives)を使用するには、以下の`Environment` systemdディレクティブを`/etc/systemd/system/kong-enterprise-edition.service`（ {{site.base_gateway}}）または`/etc/systemd/system/kong.service`（ {{site.ce_product_name}}）のカスタマーサービスに追加します。スペースを含む環境変数を指定するには、systemdによって定義された引用符ルールに注意してください。

    Environment="KONG_NGINX_HTTP_OUTPUT_BUFFERS=4 64k"

### ––nginx\-confを使用してKongのNginxインスタンスをカスタマイズ

[`--nginx-conf`](/gateway/{{page.release}}/reference/configuration/#custom-nginx-templates) 引数を使用するには、`ExecStartPre` systemdディレクティブを変更して、`--nginx-conf` 引数を指定して `kong prepare` を実行します。たとえば、`/usr/local/kong/custom-nginx.template` にカスタムテンプレートがある場合は、`ExecStartPre` ディレクティブを次のように変更します。

    ExecStartPre=/usr/local/bin/kong prepare -p /usr/local/kong --nginx-conf /usr/local/kong/custom-nginx.template

### 注入されたNginxディレクティブを介して、ファイルを含むKongのNginxインスタンスをカスタマイズ

環境変数で[注入されたNginxディレクティブ経由でファイルを含める](/gateway/{{page.release}}/reference/configuration/#including-files-via-injected-nginx-directives)には、以下の`Environment` systemdディレクティブを`/etc/systemd/system/kong-enterprise-edition.service`（{{site.base_gateway}}）または`/etc/systemd/system/kong.service`（{{site.ce_product_name}}）にあるカスタムサービスに追加します。

    Environment=KONG_NGINX_HTTP_INCLUDE=/path/to/your/my-server.kong.conf

