---
title: "Kong設定ファイル"
content-type: "how-to"
---

{{site.base_gateway}}にはデフォルトの構成ファイル、`kong.conf`が付属します。公式パッケージを使用して{{site.base_gateway}}をインストールした場合、このファイルは`/etc/kong/kong.conf.default`にあります。

{{site.base_gateway}}構成ファイルは、Kongインスタンスの個々のプロパティを構成するために使用できるYAMLファイルです。このガイドでは、 `kong.conf`ファイルを使用して{{site.base_gateway}}を構成する方法について説明します。

`kong.conf`で使用可能なすべてのパラメータについては、[構成パラメータリファレンス](/gateway/{{page.release}}/reference/configuration/)を参照してください。

{{site.base_gateway}}の構成
--------

{{site.base_gateway}} を構成するには、デフォルトの構成ファイルのコピーを作成します。

```bash
cp /etc/kong/kong.conf.default /etc/kong/kong.conf
```

このファイルには、構成プロパティとドキュメントが含まれています。

```bash
#upstream_keepalive_pool_size = 60  # Sets the default size of the upstream
                                    # keepalive connection pools.
                                    # Upstream keepalive connection pools
                                    # are segmented by the `dst ip/dst
                                    # port/SNI` attributes of a connection.
                                    # A value of `0` will disable upstream
                                    # keepalive connections by default, forcing
                                    # each upstream request to open a new
                                    # connection.
```

プロパティを構成するには、コメントを解除して値を変更します。

```bash
upstream_keepalive_pool_size = 40
```

ブール値は、`on`/`off`または`true`/`false`として指定できます。

```bash
#dns_no_sync = off               # If enabled, then upon a cache-miss every
                                 # request will trigger its own dns query.
                                 # When disabled multiple requests for the
                                 # same name/type will be synchronised to a
                                 # single query.
```

{:.note}
> 
> {{site.base_gateway}}は、`kong.conf`内のコメントアウトされた値に対してデフォルト設定を使用します。

構成を確認
-----

構成が使用可能であることを確認するには、 `check`コマンドを使用します。`check`コマンドは、現在設定されている[環境変数を](/gateway/{{page.release}}/production/environment-variables/)評価し、
設定が無効な場合はエラーを出力します。

```bash
kong check /etc/kong/kong.conf
```

設定が有効な場合、シェルは次のように出力します。

```bash
configuration at /etc/kong/kong.conf is valid
```

カスタムパスの設定
---------

デフォルトでは、{{site.base_gateway}}は2つのデフォルトロケーションで`kong.conf`を検索します。

    /etc/kong/kong.conf
    /etc/kong.conf

この動作は、自分のカスタムパスを指定することで無効にできます。CLIで `-c / --conf` 引数を使用する設定ファイル：

```bash
kong start --conf /path/to/kong.conf
```

### デバッグモード

[{{site.base_gateway}} CLI](/gateway/{{page.release}}/reference/cli/)をデバッグモードで使用して、シェルに設定プロパティを出力できます。

```bash
kong start -c /etc/kong.conf --vv

2016/08/11 14:53:36 [verbose] no config file found at /etc/kong.conf
2016/08/11 14:53:36 [verbose] no config file found at /etc/kong/kong.conf
2016/08/11 14:53:36 [debug] admin_listen = "0.0.0.0:8001"
2016/08/11 14:53:36 [debug] database = "postgres"
2016/08/11 14:53:36 [debug] log_level = "notice"
[...]
```

詳細情報
----

* [OpenResty への Kong の埋め込み](/gateway/{{page.release}}/production/kong-openresty/)
* [環境変数の設定](/gateway/{{page.release}}/production/environment-variables/)
* [Kong で API とウェブサイトを提供する方法](/gateway/{{page.release}}/production/website-api-serving/)
* [構成パラメータに関する参考資料](/gateway/{{page.release}}/reference/configuration/)

