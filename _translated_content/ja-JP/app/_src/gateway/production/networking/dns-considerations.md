---
title: "Kong Gateway の DNS に関する考慮事項"
badge: "enterprise"
---
{% if_version lte:3.4.x %}

{{site.base_gateway}} は、適切に機能するために他のKongサービスと相互作用すべきウェブアプリケーションを提供します。Kong ManagerはAdmin APIと対話できなければならず、Dev PortalはPortal APIと対話できなければなりません。これらのアプリケーションはブラウザによって適用されるセキュリティ制限の対象となるため、Kongが正しく機能するには、ブラウザに適切な情報を送信する必要があります。
{% endif_version %}

{% if_version gte:3.5.x %}

{{site.base_gateway}}はKong Managerを提供し、Kong ManagerはAdmin APIと対話できる必要があります。このアプリケーションはブラウザによって適用されるセキュリティ制限の対象となるため、Kongが正しく機能するには、ブラウザに適切な情報を送信する必要があります。
{% endif_version %}

これらのセキュリティ制限は、アプリケーションの DNS ホスト名を使用して、アプリケーションのメタデータがセキュリティ制約を満たしているかどうかを評価します。そのため、その要件を満たすように DNS 構造を設計する必要があります。

クイックガイド
-------

このドキュメントを読んで、これらの要件が存在する理由と機能を理解することをお勧めします。簡単に言うと、環境は以下の2つの条件のいずれかを満たす必要があります。

* Kong ManagerとAdmin APIは、`/_adminapi/`などの未使用のパスにAdmin APIを配置することで、通常同じホスト名から処理されます。
* Kong ManagerとAdmin APIは、サフィックスを共有する別々のホスト名から提供されています （例：`api.admin.kong.example`と`manager.admin.kong.example`では`kong.example`）。 アドミンセッションの構成によって、共有サフィックスに`cookie_domain`が設定されます。 

{% if_version lte:3.4.x %}
ポータル API と Dev Portal にも同じことが当てはまります。
{% endif_version %}

最初のオプションは `kong.conf` の設定を簡素化しますが、HTTP
プロキシをアプリケーションの前に配置する必要があります
（HTTPパスベースのルーティングを使用しているため）。
これにはKongプロキシを使用できます。2番目のオプションには、kong.conf でさらに多くの設定が必要ですが、
アプリケーションをプロキシせずに使用できます。

CORS
----

### CORS を理解する

[オリジン間ソース共有、またはCORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)は、オリジン間でリクエストを
作成するための、すなわちページがリクエストを作成するときに同じスキーム、ホスト名、
ポートを共有しないURLに対しての、ウェブアプリケーションの一連のルールです。クロスオリジン
リクエストの作成時に、ブラウザは `Origin`リクエストヘッダーを送信し、サーバーは
一致する`Access-Control-Allow-Origin`（ACAO）ヘッダーで応答する必要があります。2つの
ヘッダーが一致しない場合、ブラウザは応答を破棄し、その応答のデータを
必要とする任意のアプリケーションコンポーネントは正しく
機能しません。

たとえば、次のリクエスト/レスポンスのペアに一致するCORSヘッダーがあり、成功するとします。

    GET / HTTP/1.1
    Host: example.com
    Origin: http://example.net
    
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: http://example.net

    GET / HTTP/1.1
    Host: example.com
    Origin: http://example.net
    
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: *

`*`は、任意の配信元が許可されていることを示します。

これらのリクエストには一致するCORSヘッダーのペアがないため、失敗します。

    GET / HTTP/1.1
    Host: example.com
    Origin: http://example.net
    
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: http://badbadcors.example

    GET / HTTP/1.1
    Host: example.com
    Origin: http://example.net
    
    HTTP/1.1 200 OK

CORSヘッダーが期待されるときにCORSヘッダーが見つからないと、失敗します。

### {{site.base_gateway}}の文脈におけるCORS

{% if_version lte:3.4.x %}
Kong ManagerとDev Portalは、JavaScriptを使用してそれぞれのAPIにリクエストを発行することで動作します。これらのリクエストは、お使いの環境によってはクロスオリジンになる可能性があり
環境に依存します。

Kong のサービスは、`kong.conf` のさまざまな場所のヒント設定に基づいて、送信する CORS ヘッダーを決定します。Admin API はその ACAO ヘッダー値を `admin_gui_url` から取得し、Portal API はそのヘッダー値を `portal_gui_protocol`、`portal_gui_host`、および `portal_gui_use_subdomains` 設定の情報から取得します。`portal_cors_origins` を使用して追加のポータル API オリジンをオプションで指定できます。{% endif_version %}

{% if_version gte:3.5.x %}
Kong Manager は、JavaScript を使用して Admin API にリクエストを発行することで動作します。
これらのリクエストは、お使いの環境によってはクロスオリジンになる可能性があり
環境に依存します。Admin API
は、ACAO ヘッダー値を`kong.conf`の`admin_gui_url`から取得します。
{% endif_version %}

`https://admin.kong.example/` の Kong Manager と `https://admin.kong.example/_api/` の Admin API にアクセスするなど、同じホスト名を使用して GUI と関連 API の両方にアクセスすることで、これらのリクエストがクロスオリジンにならないように環境を設定できます。このオプションでは、パスベースのルーティングを処理するために、Kong Manager と Admin API の両方の前にプロキシを配置する必要があります。この目的のために、Kong のプロキシを使用できます。GUI はドメインのルートで提供する必要があることに注意してください。API をルートに配置したり、GUI をパスの下に配置したりすることはできません。

### トラブルシューティング

CORSエラーは、ブラウザの開発者コンソールに特定の問題の説明とともに表示されます（例については、[Firefox](https://developer.mozilla.org/en-US/docs/Tools/Web_Console/Opening_the_Web_Console)と[Chrome](https://developers.google.com/web/tools/chrome-devtools/console/log)のドキュメントを参照してください）。ACAO/Originの不一致が最も一般的ですが、その他のエラー状態も表示される可能性があります。

たとえば、`admin_api_uri`を間違って入力した場合、次のようなメッセージが
表示されます。

    Access to XMLHttpRequest at 'https://admin.kong.example' from origin 'https://manager.kong.example' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: The 'Access-Control-Allow-Origin' header has a value 'https://typo.kong.example' that is not equal to the supplied origin.

これらのエラーは一般的に説明を必要としませんが、問題が明確でない場合は、ネットワーク開発者ツールを確認し、エラーのパスのリクエストを見つけて、その`Origin`リクエストヘッダーと`Access-Control-Allow-Origin`応答ヘッダー（完全に欠落している可能性があります）と比較します。

Cookie
------

### Cookieを理解する

[Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)とは、将来のリクエストで使用するためにブラウザによって保存される小さなデータです。サーバーには、Cookieを設定するために応答ヘッダーに[Set\-Cookieヘッダー](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)が含まれており、ブラウザは後続のリクエストを行うときに[Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)ヘッダーを含めます。

Cookiesは様々な目的で使用され、ブラウザが特定のユースケースに合わせてCookieを含むタイミングを調整するための設定が複数あります。特に注目すべきは、以下のディレクティブです。

* Cookie のスコープは、Cookie の `Domain` および `Path` のディレクティブによって定義されます。これらがない場合、Cookieはそれを作成したホスト名にのみ送信されます。`example.com` によって作成された Cookie は、`www.example.com` へのリクエストでは送信されません。`Domain` を指定すると、Cookie はそのホスト名とそのサブドメインに送信されるため、`Domain=example.com` を含む `example.com` からの Cookie は、`www.example.com` へのリクエストとともに送信 *されます* 。
* `Secure`ディレクティブ。Cookieが暗号化されていない（HTTPSでなくHTTP）接続経由で送信可能かどうかを決定します。`Secure` なCookieはHTTP経由で送信できず、HTTPSを使用するように設定される必要があります。
* `SameSite` ディレクティブ。クロスオリジンのリクエストでクッキーをいつ送信するかを制御します。クッキーにおけるクロスオリジンの概念は CORS とは異なり、クッキーはドメインのサフィックスと照合を行うことに注意してください。CORS の観点では `api.example.com` にリクエストを送る `example.com` はクロスオリジンですが、`Domain=example.com` を持つクッキーは `api.example.com` へのリクエストに対しては同一サイトと見なされます。`SameSite=Strict` クッキーは同一サイトのリクエストでのみ送信され、`Lax` は別のサイトからのリンクへ移動する場合に送信され、`None` はすべてのクロスオリジンのリクエストで送信されます。

### {{site.base_gateway}}の文脈におけるクッキー

{% if_version lte:3.4.x %}
Kong ManagerまたはDev Portalにログインした後、Kongはセッション情報をCookieに保存し、今後のリクエスト時にブラウザを認識します。これらのCookieは、[セッションプラグイン](/hub/kong-inc/session/)（`admin_gui_session_conf`経由）または[OpenID Connectプラグイン](/hub/kong-inc/openid-connect/)を使用して作成されます。{% endif_version %}
{% if_version gte:3.5.x %}
Kong Managerにログインすると、Kongはセッション情報をCookieに保存し、今後のリクエスト時にブラウザを認識します。これらのCookieはは、[セッションプラグイン](/hub/kong-inc/session/)（`admin_gui_session_conf`経由）または[OpenID Connectプラグイン](/hub/kong-inc/openid-connect/)を使用して作成されます。{% endif_version %}構成は、それぞれほぼ同じです。OpenID Connectプラグインにはセッションプラグインの埋め込みバージョンが含まれているため、Cookie処理コードは同じですが、OpenID Connectプラグイン設定（`admin_gui_auth_conf`）で直接構成されます。

{% if_version lte:3.4.x -%}

* `cookie_name`Cookie が使用される場所には影響しませんが、衝突を避けるために一意の値に設定する必要があります。一部の構成では、管理者CookieとポータルCookieの両方に同じ`cookie_domain`が使用され、両方に同じ名前を使用すると、Cookieが衝突して互いに上書きされてしまいます。{% endif_version -%}
* `cookie_domain` は、GUIとそのAPIが共有する共通のホスト名サフィックスと一致する必要があります。たとえば、Admin API と Kong Manager に `api.admin.kong.example` と `manager.admin.kong.example` を使用する場合、`cookie_domain` は `admin.kong.example` である必要があります。{% if_version lte:3.1.x -%}
* `cookie_samesite`は、通常 デフォルトの `strict`のままにしておきます。 
  * `none`は、このドキュメントの例に従ってDNSレコードと`cookie_domain`が設定されている場合は必要ありません。 
  * `off`が必要なのは、GUIとAPIが全く別のホスト名である場合のみです。たとえば、APIが`admin.kong.example`で、Kong Managerが`manager.example.com` など。この設定は`off`クロスサイトリクエストフォージェリ攻撃のベクトルを開くため、推奨されません。一部の開発環境やテスト環境では必要になるかもしれませんが、本番環境では使用しないでください。{% endif_version %}

{% if_version gte:3.2.x -%}

* `cookie_same_site`は、通常
  デフォルトの `Strict`のままにしておきます。

  * `None`は、このドキュメントの例に従ってDNSレコードと`cookie_domain`が設定されている場合は必要ありません。
  * `Lax`が必要なのは、GUIとAPIが全く別のホスト名である場合のみです。たとえば、APIが`admin.kong.example`で、Kong Managerが`manager.example.com` など。この設定は`Lax`クロスサイトリクエストフォージェリ攻撃のベクトルを開くため、推奨されません。一部の開発環境やテスト環境では必要になるかもしれませんが、本番環境では使用しないでください。{% endif_version -%}

* `cookie_secure` クッキーをセキュリティで保護されていない
  \(プレーンテキストの HTTP\) リクエストで送信できるかどうかを制御します。デフォルトでは`true`に設定されており、
  安全でない接続経由でのクッキーの送信を許可していません。この設定は
  デフォルトのまま残すべきですが、開発や
  HTTPS が使用されていないテスト環境では無効化されることもあります。

OpenID Connect は同じ設定を使用しますが、プレフィックスに`session_`が付きます。たとえば、`cookie_name`ではなく`session_cookie_name`です。

{% if_version lte:3.4.x %}
Dev Portalの構成はKong Managerの構成と大きく異なることはありませんが、Kong ManagerのPortal > Settingsセクションの「Session構成（JSON）」フィールドでワークスペースごとに構成されます。
{% endif_version %}

CORSと同様に、GUI と API の両方が同じホスト名を使用し、プロキシの背後にあり、API がそのホストの特定パスの配下に存在する場合、上記は必要ありません。

### トラブルシューティング

セッションクッキーの問題は、大半がクッキーが送信されなかった場合と、クッキーが設定されていない場合に発生します。ネットワーク（例：[Firefox](https://developer.mozilla.org/en-US/docs/Tools/Network_Monitor)と[Chrome](https://developers.google.com/web/tools/chrome-devtools#network)のドキュメントを参照）とアプリケーション/ストレージ（[Firefox](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector)と[Chrome](https://developers.google.com/web/tools/chrome-devtools#application)のドキュメントを参照）開発者ツールは、これらの問題の調査に役立ちます。

ネットワークツールで個々のリクエストを選択すると、そのリクエストと応答ヘッダーが
表示されます。認証リクエストが成功すると、`cookie_name`設定と
一致する名前のCookieを含む`Set-Cookie`レスポンスヘッダーが表示され、
後続のリクエストは`Cookie`リクエストヘッダーに同じCookieを含む
必要があります。

`Set-Cookie`が存在しない場合は、中間プロキシによって取り除かれているか、認証ハンドラでエラーが発生している可能性があります。
エラーの場合、通常、レスポンスステータスとボディに他の証拠があるはずであり、さらにKongのエラーログに追加情報がある可能性があります。

Cookie が設定されていても送信されない場合は、Cookie が削除されているか、Cookie を必要とするリクエストと一致しない可能性があります。アプリケーション/ストレージツールは、現在の Cookie とそのパラメータを表示します。これらを確認して、リクエストが Cookie を送信する基準（Cookie ドメインが Cookie を必要とするリクエストのサフィックスではない、または存在しないなど）を満たしていないかどうかを確認し、それに応じてセッション構成を調整します。

Cookieがアプリケーション/ストレージに存在 *しない* ものの、以前に`Set-Cookie`
で設定されていた場合は、それ以来削除されているか、期限切れになっている可能性があります。
`Set-Cookie`情報を見直して、Cookieに設定された有効期限と後続リクエストを確認し、他の応答で`Set-Cookie`
が発行された結果、（有効期限を過去の日付に設定することで）Cookieが削除されたかどうかを判断します。

このトラブルシューティング情報は、問題の原因をすぐに示すものではありませんが、Kongサポートが原因を特定するのに役立ちます。可能な場合はチケットを生成して問題を報告してください。

