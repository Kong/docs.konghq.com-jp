---
title: "Dev Portalのセッション"
---
{:.important}
> 
> **重要** ：Dev Portal認証に[OpenID Connect](/hub/kong-inc/openid-connect/)を使用する場合、ポータルSession構成は適用されません。以下の情報は、Dev Portalが`openid-connect`以外の`portal_auth`（たとえば、 `key-auth`または`basic-auth`）で構成されていることを前提としています。

Dev Portal で Session プラグインはどのように機能しますか？
---------------------------------------

ユーザーが認証情報を使用して Dev Portal にログインすると、セッションプラグインはセッション Cookie を作成します。この Cookie は後続のすべてのリクエストに使用され、ユーザーの認証に有効です。セッションの有効期間は限られており、構成可能な間隔で更新されるため、セッション終了後に攻撃者が古い Cookie を取得して使用することを防ぐことができます。

セッション構成はデフォルトで安全ですが、HTTPを使用する場合や、[`portal_api_url`](/gateway/{{page.release}}/reference/configuration/#portal_api_url)と[`portal_gui_host`](/gateway/{{page.release}}/reference/configuration/#portal_gui_host)に異なるドメインを使用する場合は、[変更が必要](#session-security)になることがあります。クッキーは暗号化されているので、たとえ攻撃者が古いものを入手しようとしても
、彼らにメリットはありません。暗号化された
セッションデータはKongまたはCookie自体に保存されます。

Dev Portal でセッションプラグインを使用するための設定
--------------------------------

セッション認証を有効にするには、次の項目を設定します。

{% if_version lte:3.1.x %}

    portal_auth = <set to desired auth type id="sl-md0000000">
    portal_session_conf = {
        "secret":"<set_secret id="sl-md0000000">",
        "cookie_name":"<set_cookie_name id="sl-md0000000">",
        "storage":"kong",
        "cookie_lifetime":<number_of_seconds_to_live id="sl-md0000000">,
        "cookie_secure":<set_depending_on_protocol id="sl-md0000000">,
        "cookie_domain":"<set_depending_on_domain id="sl-md0000000">",
        "cookie_samesite":"<set_depending_on_domain id="sl-md0000000">"
    }

{% endif_version %}

{% if_version gte:3.2.x %}

    portal_auth = <set to desired auth type id="sl-md0000000">
    portal_session_conf = {
        "secret":"<set_secret id="sl-md0000000">",
        "cookie_name":"<set_cookie_name id="sl-md0000000">",
        "storage":"kong",
        "rolling_timeout":<number_of_seconds_until_renewal id="sl-md0000000">,
        "cookie_secure":<set_depending_on_protocol id="sl-md0000000">,
        "cookie_domain":"<set_depending_on_domain id="sl-md0000000">",
        "cookie_samesite":"<set_depending_on_domain id="sl-md0000000">"
    }

{% endif_version %}

* `"cookie_name":"<set_cookie_name id="sl-md0000000">"`：cookieの名前 
  * 例：`"cookie_name":"portal_cookie"`

* `"secret":"<set_secret id="sl-md0000000">"`：キー付きHMAC生成に使用されるシークレット。 **Sessionプラグインの** デフォルトはランダムな文字列ですが、シークレットはKongワーカーまたはノード全体で同一でなければならないため、`secret`を使用するにはDev Portalで手動で設定する *必要があります* 。 
* `"storage":"kong"`: セッションデータが保存される場所です。Dev Portalで使用するには、この値を`kong`に *設定する必要があります* 。
{% if_version lte:3.1.x -%}
* `"cookie_lifetime":<number_of_seconds_to_live id="sl-md0000000">`: セッションが開いている期間 \(秒単位\)。デフォルトは 3600 です。
{% endif_version -%}
{% if_version gte:3.2.x -%}
* `"rolling_timeout":<number_of_seconds_until_renewal id="sl-md0000000">`セッションの更新が必要になるまでのセッション使用可能時間を秒単位で指定します。デフォルト値は3600です。
{% endif_version -%}
* `"cookie_secure":<set_depending_on_protocol id="sl-md0000000">`：デフォルトでは`true` です。例外については、[Sessionのセキュリティ](#session-security)を参照してください。
* `"cookie_domain":<set_depending_on_domain id="sl-md0000000">:`任意。例外については、[Session セキュリティ](#session-security)を参照してください。
{% if_version lte:3.1.x -%}
* `"cookie_samesite":"<set_depending_on_domain id="sl-md0000000">"`: デフォルトでは `"Strict"` 。例外については、[Session のセキュリティ](#session-security)を参照してください。{% endif_version -%}
{% if_version gte:3.2.x -%}
* `"cookie_same_site":"<set_depending_on_domain id="sl-md0000000">"`：デフォルトでは`"Strict"` です。例外については、[Sessionのセキュリティ](#session-security)を参照してください。{% endif_version %}

{:.important}
> 
> Dev Portal で使用する場合、次のプロパティをデフォルトから変更しないでください。
> 
> * `logout_methods`
> * `logout_query_arg`
> * `logout_post_arg`

各構成プロパティの詳細な説明については、[Session プラグインのドキュメント](/hub/kong-inc/session/)を参照してください。

Sessionのセキュリティ
--------------

Session構成はデフォルトでセキュアであるため、クッキーは[Secure, HttpOnly](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Secure_and_HttpOnly_cookies)と[SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies)ディレクトリを使用します。

使用中のプロトコルとドメインに応じて、次のプロパティを変更する必要があります。

* HTTPSの代わりに HTTP を使用する場合：`"cookie_secure": false`
* [`portal_api_url`](/gateway/{{page.release}}/reference/configuration/#portal_api_url)と[`portal_gui_host`](/gateway/{{page.release}}/reference/configuration/#portal_gui_host)に異なるサブドメインを使用する場合は、以下の[ドメイン](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/sessions/#domains)の例を参照してください。

{:.important}
> 
> **重要：** `"storage": "cookie"`（デフォルト）が使用されている場合、ユーザーがログアウトしてもセッションは無効になりません。その場合、cookieはクライアント側で削除されます。セッションデータが`"storage": "kong"`セットでサーバー側に保存されている場合のみ、セッションはアクティブに無効化されます。

構成例
---

### APIとGUIで同じドメインを持つHTTPS

HTTPSを使用しており、同じドメインのDev Portal APIとDev Portal GUIをホストしている場合、Basic Authには次の設定を使用できます。

    portal_auth = basic-auth
    portal_session_conf = {
        "cookie_name":"$4m04$"
        "secret":"change-this-secret"
        "storage":"kong"
        "cookie_secure":true
    }

テストでは、HTTP を使用する場合は、代わりに次の構成を使用できます。

    portal_auth = basic-auth
    portal_session_conf = {
        "cookie_name":"04tm34l"
        "secret":"change-this-secret"
        "storage":"kong"
        "cookie_secure":false
    }

### ドメイン

{% if_version lte:3.1.x %}
Dev Portal の `portal_gui_host` と Dev Portal の API である `portal_api_url` は、同じドメインまたはサブドメインを共有する必要があります。以下の例では、`portal.xyz.com` と `portalapi.xyz.com` のサブドメインを想定しています。
`"cookie_domain": ".xyz.com"` のようにサブドメインを設定し、`cookie_samesite` を `off` に設定してください。

    portal_auth = basic-auth
    portal_session_conf = {
        "storage":"kong"
        "cookie_name":"portal_session"
        "cookie_domain": ".xyz.com"
        "secret":"super-secret"
        "cookie_secure":false
        "cookie_lifetime":31557600,
        "cookie_samesite":"off"
    }

{% endif_version %}

{% if_version gte:3.2.x %}
Dev Portal の `portal_gui_host` と Dev Portal の API である `portal_api_url` は、同じドメインまたはサブドメインを共有する必要があります。以下の例では、`portal.xyz.com` と `portalapi.xyz.com` のサブドメインを想定しています。
`"cookie_domain": ".xyz.com"` のようにサブドメインを設定し、`cookie_same_site` を `Lax` に設定してください。

    portal_auth = basic-auth
    portal_session_conf = {
        "storage":"kong"
        "cookie_name":"portal_session"
        "cookie_domain": ".xyz.com"
        "secret":"super-secret"
        "cookie_secure":false
        "rolling_timeout":31557600,
        "cookie_same_site":"Lax"
    }

{% endif_version %}

