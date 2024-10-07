---
title: "フォワードプロキシを介したコントロールプレーンとデータプレーンの通信"
content_type: "reference"
---
コントロールプレーン（CP）とデータプレーン（DP）が、プロキシを通じて外部通信を実行するファイアウォールの異なる側で実行される場合、
{{site.base_gateway}}を構成してプロキシサーバーを認証し、トラフィックの通過を許可できます。


{{site.base_gateway}}は[HTTP CONNECT](https://httpwg.org/specs/rfc9110.html#CONNECT)プロキシのみをサポートします。

{:.important}
> 
> この機能は TLS相互認証（mTLS）の終了をサポートしていません。

フォワードプロキシ接続を設定する
----------------

[`kong.conf`](/gateway/{{page.release}}/production/kong-conf/)で次のパラメータを設定します。

    proxy_server = http(s)://<username id="sl-md0000000">:<password id="sl-md0000000">@<proxy-host id="sl-md0000000">:<proxy-port id="sl-md0000000">
    proxy_server_tls_verify = on/off
    cluster_use_proxy = on
    lua_ssl_trusted_certificate = system | <certificate id="sl-md0000000"> | <path-to-cert id="sl-md0000000">

* `proxy_server`: URL として定義されたプロキシサーバー。{{site.base_gateway}} は、プロキシを使用するように明示的に構成されているコンポーネントがある場合にのみ、このオプションを使用します。

* `proxy_server_tls_verify`: `proxy_server` が HTTPS を使用している場合、サーバー証明書の検証を切り替えます。HTTPS（デフォルト）を使用する場合は `on` に設定し、HTTPを使用する場合は `off` に設定します。

* `cluster_use_proxy`: HTTP CONNECTプロキシサポートをハイブリッドモードの接続に使用するように、クラスタに指示します。オンにすると、{{site.base_gateway}}は、`proxy_server`で定義されたURLを接続に使用します。

* `lua_ssl_trusted_certificate`（ *オプション* ）：HTTPSを使用する場合は、
  `lua_ssl_trusted_certificate`を使用してカスタム証明機関を指定します。[システムのデフォルトCA](/gateway/{{page.release}}/reference/configuration/#lua_ssl_trusted_certificate)を使用している場合は、この値を変更する必要はありません。

接続を有効にするには、{{site.base_gateway}}を再読み込みします。

    kong reload

