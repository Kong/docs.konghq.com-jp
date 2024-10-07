---
title: "ネットワークとファイアウォール"
content_type: "reference"
---
このセクションは、Kongで推奨されるネットワークとファイアウォールの設定について説明します。

ポート
---

Kongは、さまざまな目的で複数の接続を使用します。

* プロキシ
* Admin API

### プロキシ

プロキシポートは、Kong が着信トラフィックを受信する場所です。次のデフォルトを持つ 2 つのポートがあります。

* `8000`HTTPトラフィックのプロキシ、および
* `8443`HTTPSトラフィックのプロキシ用

HTTP/HTTPSプロキシlistenオプションの詳細については、[proxy\_listen](/gateway/{{page.release}}/reference/configuration/#proxy_listen)参照してください。本番環境では、
HTTPおよびHTTPSのリッスンポートを`80` と `443`に変更します。

KongはTCP/TLSストリームをプロキシすることもできます。ストリームプロキシはデフォルトで無効になっています。ストリームプロキシリッスンオプションの追加の詳細と、これを有効にする方法（HTTP/HTTPSトラフィック以外のものをプロキシする予定の場合）については、[stream\_listen](/gateway/{{page.release}}/reference/configuration/#stream_listen)をご覧ください。

一般的に、プロキシポートは、クライアントが使用できるようにする必要がある **唯一のポート** です。

### Admin API

これは、Kongが管理APIを公開しているポートです。したがって、本番環境では、このポートを不正アクセスから守るために
ファイアウォールで保護する必要があります。

* `8001`HTTPでKongを操作するために使用できるKongの **Admin API** を提供します。[admin\_listen](/gateway/{{page.release}}/reference/configuration/#admin_listen)を参照してください。

{% include_cached /md/admin-listen.md desc='short' release=page.release %}

* `8444` 同じKong **Admin API** を提供していますが、HTTPSを使用しています。[admin\_listen](/gateway/{{page.release}}/reference/configuration/#admin_listen)と`ssl`サフィックスを参照してください。

ファイアウォール
--------

推奨されるファイアウォール設定を次に示します。

* Kongの背後にあるアップストリームサービスは、 [proxy\_listen](/gateway/{{page.release}}/reference/configuration/#proxy_listen)インターフェース/ポート値を介して利用できます。 アップストリームサービスに付与するアクセスレベルに応じて、これらの値を設定します。
* Admin APIを（[admin\_listen](/gateway/{{page.release}}/reference/configuration/#admin_listen)経由で）公開インターフェースにバインドする場合は、 **保護** して、 信頼されたクライアントのみがAdmin APIにアクセスできるようにします。[「Admin APIの保護」](/gateway/{{page.release}}/production/running-kong/secure-admin-api)も参照してください。
* プロキシには、構成したすべてのHTTP/HTTPSおよびTCP/TLSストリームリスナー用のルールを追加する必要があります。たとえば、Kong でポート `4242` のトラフィックを管理する場合、ファイアウォールで当該ポートのトラフィックを許可する必要があります。

#### 透過プロキシ

`transparent` listen オプションは、[proxy\_listen](/gateway/{{page.release}}/reference/configuration/#proxy_listen) および [stream\_listen](/gateway/{{page.release}}/reference/configuration/#stream_listen) 構成に適用できることに注意してください。`iptables`（Linux）や `pf`（macOS/BSD）などのパケットフィルタリングを使用する場合、またはハードウェアルータ/スイッチを使用する場合は、TCPパケットの事前ルーティングまたはリダイレクトルールを指定して元の宛先アドレスとポートを変更することができます。例えば、宛先アドレスが `10.0.0.1` 、宛先ポートが `80` の HTTP リクエストを、`127.0.0.1` のポート `8000` にリダイレクトできます。これを機能させるには、`transparent` listen オプションを Kong プロキシ `proxy_listen=8000 transparent` に追加する必要があります（Linuxの場合）。これにより、Kong が実際にリクエストを直接受信していなくても、Kong はリクエストの元の宛先（`10.0.0.1:80`）を確認できます。この情報により、Kong はリクエストを正しくルーティングできます。`transparent` listen オプションは Linux でのみ使用してください。macOS/BSD では、`transparent` listen オプションなしで透過プロキシが許可されます。Linux では、Kong を `root` ユーザーとして起動するか、実行可能ファイルに必要な機能を設定する必要があります。

