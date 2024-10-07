---
title: "Kong Managerのセッション"
badge: "enterprise"
---
ユーザーが自分の認証情報を使ってKong Managerにログインすると、セッションプラグインはセッションCookieを作成します。このCookieは後続のすべてのリクエストに使用され、ユーザーの認証に有効です。セッションの有効期間は限られており、構成可能な間隔で更新されるため、セッション終了後に攻撃者が古いCookieを取得して使用することを防ぐことができます。

Session 設定はデフォルトでセキュアになっており、
Admin API と Kong Manager に HTTP または異なるドメインを使用する場合は[変更が必要](#session-security)な場合があります
。クッキーは暗号化されているので、たとえ攻撃者が古いものを入手しようとしても
、彼らにメリットはありません。暗号化された
セッションデータは Kong またはクッキー自体に保存されます。

Kong Manager のセションプラグインを設定
--------------------------

セッション認証を有効にするには、次の項目を設定します。

{% if_version lte:3.1.x %}

    enforce_rbac = on
    admin_gui_auth = <set to desired auth type id="sl-md0000000">
    admin_gui_session_conf = {
        "secret":"<set_secret id="sl-md0000000">",
        "cookie_name":"<set_cookie_name id="sl-md0000000">",
        "storage":"<set_storage id="sl-md0000000">",
        "cookie_lifetime":<number_of_seconds_to_live id="sl-md0000000">,
        "cookie_secure":<set_depending_on_protocol id="sl-md0000000">
        "cookie_samesite":"<set_depending_on_domain id="sl-md0000000">"
    }

{% endif_version %}
{% if_version gte:3.2.x %}

    enforce_rbac = on
    admin_gui_auth = <set to desired auth type id="sl-md0000000">
    admin_gui_session_conf = {
        "secret":"<set_secret id="sl-md0000000">",
        "cookie_name":"<set_cookie_name id="sl-md0000000">",
        "storage":"<set_storage id="sl-md0000000">",
        "rolling_timeout":<number_of_seconds_until_renewal id="sl-md0000000">,
        "cookie_secure":<set_depending_on_protocol id="sl-md0000000">
        "cookie_same_site":"<set_depending_on_domain id="sl-md0000000">"
    }

{% endif_version %}

{% if_version lte:3.1.x %}

|        属性         |                                                                        説明                                                                        |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| `cookie_name`     | Cookie の名前。 <br>例：`"cookie_name":"kong_cookie"`                                                                                                  |
| `secret`          | キー付き HMAC の生成で使用されるシークレット。Sessionプラグインのデフォルトはランダムな文字列ですが、`secret`は *必ず* Kong Manager使用時に手動で設定する必要があり、Kong ワーカー/ノードで同一である必要があります。                 |
| `storage`         | セッションデータが保存される場所。<br>デフォルト値は`cookie`です。データベースへのアクセスが必要になるため、`kong`に設定するとより安全になる可能性があります。                                                         |
| `cookie_lifetime` | セッションが開いたまま維持される時間（秒単位）。<br>デフォルト値は`3600`です。                                                                                                     |
| `cookie_secure`   | Secure ディレクティブを適用して、HTTPSプロトコルを介した暗号化されたリクエストでのみクッキーをサーバーに送信できるようにします。例外については、[Session セキュリティを](#session-security)参照してください。<br> デフォルト値は`true`です。 |
| `cookie_samesite` | クロスサイト・リクエストでCookieを送信するかどうか、またその送信方法を決定します。例外については、[Sessionのセキュリティ](#session-security)を参照してください。デフォルト値は`strict`です。                              |

{% endif_version %}

{% if_version gte:3.2.x %}

|         属性         |                                                                        説明                                                                        |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| `cookie_name`      | Cookie の名前。 <br>例：`"cookie_name":"kong_cookie"`                                                                                                  |
| `secret`           | キー付き HMAC の生成で使用されるシークレット。Sessionプラグインのデフォルトはランダムな文字列ですが、`secret`は *必ず* Kong Manager使用時に手動で設定する必要があり、Kong ワーカー/ノードで同一である必要があります。                 |
| `storage`          | セッションデータが保存される場所。<br>デフォルト値は`cookie`です。データベースへのアクセスが必要になるため、`kong`に設定するとより安全になる可能性があります。                                                         |
| `idling_timeout`   | セッション・クッキーのアイドル時間（秒）。 <br>デフォルト値は 900 です。                                                                                                        |
| `rolling_timeout`  | セッションの更新が必要になるまでのセッション使用可能時間を秒単位で指定します。<br>デフォルト値は3600です。                                                                                        |
| `cookie_secure`    | Secure ディレクティブを適用して、HTTPSプロトコルを介した暗号化されたリクエストでのみクッキーをサーバーに送信できるようにします。例外については、[Session セキュリティを](#session-security)参照してください。<br> デフォルト値は`true`です。 |
| `cookie_same_site` | クロスサイト・リクエストでCookieを送信するかどうか、またその送信方法を決定します。例外については、[Sessionのセキュリティ](#session-security)を参照してください。デフォルト値は`strict`です。                              |

{% endif_version %}

{:.important}
> 
> **重要:** 以下のプロパティは Kong Manager で使用するため、デフォルトから変更 **しない** でください。

* `logout_methods`
* `logout_query_arg`
* `logout_post_arg`

各設定プロパティの詳細な説明については、[Session プラグインのドキュメント](/hub/kong-inc/session)を参照してください。

Sessionのセキュリティ
--------------

Session構成はデフォルトでセキュアであるため、クッキーは[Secure, HttpOnly](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Secure_and_HttpOnly_cookies)と[SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies)ディレクトリを使用します。

以下のプロパティは、使用中のプロトコルとドメインに応じて変更する必要があります。

* HTTPSの代わりにHTTPを使用する場合：`"cookie_secure": false`
{% if_version lte:3.1.x -%}
* Admin APIとKong Managerに異なるドメインを使用する場合: `"cookie_samesite": "off"`
{% endif_version -%}
{% if_version gte:3.2.x -%}
* Admin APIとKong Managerに異なるドメインを使用する場合: `"cookie_same_site": "Lax"` {% endif_version %}

{:.important}
> 
> **重要:** `"storage": "cookie"`（デフォルト）が使用されている場合、ユーザーがログアウトしてもセッションは無効になりません。その場合、cookieはクライアント側で削除されます。セッションデータが`"storage": "kong"`セットでサーバー側に保存されている場合のみ、セッションはアクティブに無効になります。

構成例
---

HTTPSを使用し、Kong ManagerとAdmin APIを同じドメインからホストする場合、
Basic Authには次の構成を使用できます。

    enforce_rbac = on
    admin_gui_auth = basic-auth
    admin_gui_session_conf = {
        "cookie_name":"$4m04$",
        "secret":"change-this-secret",
        "storage":"kong"
    }

テストでは、HTTPを使用する場合は、代わりに次の構成を使用できます。

    enforce_rbac = on
    admin_gui_auth = basic-auth
    admin_gui_session_conf = {
        "cookie_name":"04tm34l",
        "secret":"change-this-secret",
        "storage":"kong",
        "cookie_secure":false
    }

