---
title: "Kong Managerのネットワーク設定"
badge: "enterprise"
---
デフォルト構成
-------

{% if_version gte:3.8.x %}
デフォルトでは、Kong Managerは認証なしで起動し（ admin_gui_authを参照）、Admin APIがKong Managerを提供する同じホストのポート8001で利用可能であると仮定します。
{% endif_version %}
{% if_version lte: 3.7.x %}
デフォルトでは、Kong Manager は認証なしで起動し（
[`admin_gui_auth`](/gateway/{{page.release}}/reference/configuration/#admin_gui_auth)を参照）、Kong Manager を処理する同じホストのポート 8001（[デフォルトのポート](/gateway/{{page.release}}/production/networking/default-ports/)を参照）で Admin API が利用可能
であることを前提としています。
{% endif_version %}

カスタム構成
------

Kong Manager の一般的な構成シナリオをいくつか示します。

### 専用KongノードからのKong Managerの提供

Kong Managerが専用のKongノード上にある場合、
Admin API への外部呼び出しを行う必要があります。[`admin_api_uri`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri)を
Admin API の場所に設定します。

### 認証プラグインによる Kong Manager の保護

Kong Manager が認証プラグイン
で保護されており、専用ノード上に *ない* 場合、同じホスト上で Admin API
を呼び出します。デフォルトでは、Admin API はローカルホストのポート 8001 と 
8444 でリッスンします。必要に応じて [`admin_listen`](/gateway/{{page.release}}/reference/configuration/#admin_listen) を変更するか、
[`admin_api_uri`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri) を設定します。

{% include_cached /md/admin-listen.md desc='short' release=page.release %}

### Kong Managerの保護と専用ノードからの提供

Kong Managerが **保護され、専用ノードから提供される** 場合、
[`admin_api_uri`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri)をAdmin APIの場所に設定します。

### Admin APIへの接続

下表は、Kong ManagerのAdmin APIへの接続を構成する際に設定される（またはデフォルトで検証される）プロパティをまとめたものです。

| 認証が有効 |                                 ローカルAPI                                 |                                 リモートAPI                                  |                                                                                                                                                                 認証設定                                                                                                                                                                  |
|-------|-------------------------------------------------------------------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 有効    | [`admin_listen`](/gateway/{{page.release}}/reference/configuration/#admin_listen) | [`admin_api_uri`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri) | [`admin_gui_auth`](/gateway/{{page.release}}/reference/configuration/#admin_gui_auth)、 [`enforce_rbac`](/gateway/{{page.release}}/reference/configuration/#enforce_rbac)、 [`admin_gui_auth_conf`](/gateway/{{page.release}}/reference/configuration/#admin_gui_auth_conf)、 [`admin_gui_session_conf`](/gateway/{{page.release}}/reference/configuration/#admin_gui_session_conf) |
| いいえ   | [`admin_listen`](/gateway/{{page.release}}/reference/configuration/#admin_listen) | [`admin_api_uri`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri) | 対象外                                                                                                                                                                                                                                                                                                                                   |

### 認証を有効にする

認証を有効にするには、以下のプロパティを設定します。

* [`admin_gui_auth`](/gateway/{{page.release}}/reference/configuration/#admin_gui_auth)を目的のプラグインに設定
* [`admin_gui_auth_conf`](/gateway/{{page.release}}/reference/configuration/#admin_gui_auth_conf)（任意）
* [`admin_gui_session_conf`](/gateway/{{page.release}}/reference/configuration/#admin_gui_session_conf)を希望の構成に設定
* [`enforce_rbac`](/gateway/{{page.release}}/reference/configuration/#enforce_rbac)の設定を`on`

{:.important}
> 
> **重要:** Kong Manager認証が有効になっている場合、権限ルールを適用するにはRBACを有効にする必要があります。ただし、Kong Managerに
> ログインできる人は誰でもAdmin APIで利用可能な操作をすべて実行できます。

TLS証明書
------

デフォルトでは、Kong ManagerのURLがCAによって発行された証明書 *なしで* HTTPS経由でアクセスされた場合、最新のウェブブラウザが信頼しない自己署名証明書を受け取り、アプリケーションはAdmin API にアクセスできなくなります。

HTTPS経由でKong Managerを提供するには、信頼できる証明機関を使用してTLS証明書を発行します。
そして次の手順のために、結果の`.crt`ファイルと`.key`ファイルを準備します。

1. `.crt`および`.key`ファイルをKongノードの任意のディレクトリに移動します。

2. 証明書とキーの絶対パスを [`admin_gui_ssl_cert`](/gateway/{{page.release}}/reference/configuration/#admin_gui_ssl_cert) と [`admin_gui_ssl_cert_key`](/gateway/{{page.release}}/reference/configuration/#admin_gui_ssl_cert_key) に指定します。

    admin_gui_ssl_cert = /path/to/test.crt
    admin_gui_ssl_cert_key = /path/to/test.key

3. TLSを使用するには、`admin_gui_url`の前に`https`が付いていることを確認してください。例:

    admin_gui_url = https://test.com:8445

### [https://localhost](https://localhost)の使用

Kong Managerをローカルホストで提供する場合、HTTPをプロトコルとして使用する方が望ましい場合があります。RBACも使用している場合、`admin_gui_session_conf`に`cookie_secure=false`を設定します。`localhost`にHTTPを使用する理由は、`localhost`のTLS証明書を作成するには、より多くの労力と構成が必要であり、それを使用する理由がないかもしれないからです。TLSの適切なユースケースは、（1）ホスト間でデータが転送されている場合、または（2）[混合コンテンツ](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content)を含むアプリケーションをテストする場合（Kong Managerでは使用されません）。

外部CAは、誰も独自で`localhost`を所有せず、上位レベルドメイン（例：`.com`、`.org`）に根差しているわけでもないため証明書を提供することはできません。同様に、自己署名証明書は現代のブラウザでは信頼されません。その代わり、独自の証明書を発行できるプライベートCAを使用する必要があります。また、古い証明書によって、将来`localhost`へのアクセスを妨害されるのを回避するために、テスト後にSSLの状態が消去されているのを確実にしてください。

