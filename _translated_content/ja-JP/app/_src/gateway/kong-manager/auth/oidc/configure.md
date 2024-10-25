---
title: "Kong ManagerのOIDCを有効にする"
badge: "enterprise"
---

{{site.base_gateway}} は、[OpenID Connect プラグイン](/hub/kong-inc/openid-connect/)を使用して、Kong Manager 管理者の認証を組織の OpenID Connect Identity プロバイダにバインドする機能を提供します。

{:.note}
> 
> **注** :以下の設定を使用することで、OpenID Connect認証は
> Kong Managerで有効になります。Admin APIまたはKong Manager経由でOpenID Connectプラグインを手動で有効にする必要はありません。

OIDC を使用して RBAC を設定
-------------------

以下は、Google を IdP として使用し、
デフォルトの URL `http://127.0.0.1:8002` から Kong Manager を扱う例です。

{% if_version lte:3.4.x %}
`admin_gui_auth_config`値は有効なJSONである必要があります。以下は構成の例です。

    enforce_rbac = on
    admin_gui_auth=openid-connect
    admin_gui_session_conf = { "secret":"set-your-string-here" }
    admin_gui_auth_conf={                                      \
      "issuer": "https://accounts.google.com/",                \
      "client_id": ["<enter_your_client_id id="sl-md0000000">"],                 \
      "client_secret": ["<enter_your_client_secret_here id="sl-md0000000">"],    \
      "consumer_by": ["username","custom_id"],                 \
      "ssl_verify": false,                                     \
      "admin_claim": ["email"],                                \
      "leeway": 60,                                            \
      "redirect_uri": ["http://localhost:8002/default"],       \
      "login_redirect_uri": ["http://localhost:8002/default"], \
      "logout_methods": ["GET", "DELETE"],                     \
      "logout_query_arg": "logout",                            \
      "logout_redirect_uri": ["http://localhost:8002/default"], \
      "scopes": ["openid","profile","email","offline_access"], \
      "auth_methods": ["authorization_code"]                   \
    }

{% endif_version %}
{% if_version eq:3.5.x %}

`admin_gui_auth_config`値は有効なJSONである必要があります。以下は構成の例です。

    enforce_rbac = on
    admin_gui_auth=openid-connect
    admin_gui_session_conf = { "secret":"set-your-string-here" }
    admin_gui_auth_conf={                                      \
      "issuer": "https://accounts.google.com/",                \
      "client_id": ["<enter_your_client_id id="sl-md0000000">"],                 \
      "client_secret": ["<enter_your_client_secret_here id="sl-md0000000">"],    \
      "admin_claim": ["email"],                                \
      "redirect_uri": ["http://localhost:8002/default"],       \
    }

OpenID Connect を使用して Kong Manager を認証する際は、IdP が`authorization_code`グラントタイプをサポートしており、関連付けられているクライアントに対して有効になっていることを確認してください。

次に、`openid-connect`の `admin_gui_auth_conf` の構成パラメータを示します。

| パラメータ | データタイプ | デフォルト値 | 注記 |
|-------|-------|-------|-------|
| `issuer`<br> *必須* | 文字列 | `"input issuer"` | URL を表す文字列 |
| `client_id`<br> *必須* | 配列 | `["input client id"]` | プラグインが IDプロバイダーの認証済みエンドポイントを呼び出すときに使用するクライアントID。|
| `client_secret`<br> *必須* | 配列 | `["input client secret"]` | クライアントシークレット。|
| `redirect_uri` <br> *必須* | 配列 | `["input http://ip:8002/default"]` | 認証エンドポイントとトークンエンドポイントに渡されるリダイレクトURI。|
| `authenticated_groups_claim` <br> *必須* | 配列 | `["groups"]` | 認証されたグループを含むクレーム。|
| `admin_claim`<br> *必須* | 配列 | `["email"]` | フィールドをユーザー名として取得します。|
| `admin_auto_create`<br> *任意* | ブール値 | `true` | このパラメータは、管理者の自動作成を有効にするために使用されます。|
| `ssl_verify` <br> *任意* | ブール値 | `false` | IDプロバイダーのサーバー証明書を確認します。|

{% endif_version %}
{% if_version gte:3.6.x %}

{:.important}
> 
> **重要** ：以前のバージョンの構成を使用している場合は、[移行ガイド](/gateway/{{page.release}}/kong-manager/auth/oidc/migrate/)を使って構成を確認および更新してください。

`admin_gui_auth_config`値は有効なJSONである必要があります。以下は構成の例です。

    enforce_rbac = on
    admin_gui_auth=openid-connect # specify the plugin
    admin_gui_auth_conf={                                                                                         \
      "issuer": "https://dev-xxxx.okta.com/oauth2/default",                                                       \
      "client_id": ["<enter_your_client_id id="sl-md0000000">"],                                                                    \
      "client_secret": ["<enter_your_client_secret_here id="sl-md0000000">"],                                                       \
      "redirect_uri": ["http://localhost:8001/auth"],                                                             \
      "scopes": ["openid","email","offline_access"], # "email" is for the admin_claim, may vary in different IdPs \
      "login_redirect_uri": ["http://localhost:8002"],                                                            \
      "logout_redirect_uri": ["http://localhost:8002"],                                                           \
      "admin_claim": "email",                                                                                     \
      "authenticated_groups_claim": ["groups"],                                                                   \
    }

OpenID Connect を使用してKong Managerを認証する際は、IdPが`authorization_code`グラントタイプをサポートしており、関連付けられているクライアントに対して有効になっていることを確認してください。

OpenID ConnectでKong Managerを認証する際、 `admin_gui_auth_conf`が
OIDC プラグイン構成に使用されます。共通パラメータの他に、Kong Manager で OIDC を使用する場合、重要かつ特定のパラメータがいくつかあります。

|                  パラメータ                   | データタイプ |                 デフォルト値                  |                                                    ノート                                                     |
|------------------------------------------|--------|-----------------------------------------|------------------------------------------------------------------------------------------------------------|
| `issuer`<br> *必須*                        | 文字列    | --                                  | IdP（IDプロバイダー）に関するメタデータを解決するベースURL。例：`"https://dev-xxxx.okta.com/oauth2/default"`                           |
| `client_id`<br> *必須*                     | Array  | --                                  | プラグインがIdPとの通信中に使用するクライアントID。                                                                               |
| `client_secret`<br> *必須*                 | Array  | --                                  | クライアントシークレット。                                                                                              |
| `redirect_uri`<br> *必須*                  | Array  | --                                  | IdPでの認証後にリダイレクトするURI。通常、Admin APIの`/auth`エンドポイントを指します。例：`"http://localhost:8001/auth"`                     |
| `login_redirect_uri`<br> *必須*            | Array  | --                                  | Admin APIでの認証後にリダイレクトするURI。Kong Managerのエンドポイントを指す必要があります。例：`"http://localhost:8002"`                      |
| `logout_redirect_uri`<br> *必須*           | Array  | --                                  | IdP からログアウトした後にリダイレクトする URI。Kong Manager のエンドポイントを指す必要があります。以下に例を示します。`"http://localhost:8002"`            |
| `admin_auto_create`<br> *オプション*          | ブール    | `true`                                  | このパラメータは、管理者の自動作成を有効にするために使用されます。                                                                          |
| `admin_claim`<br> *オプション*                | 文字列    | `"email"`                               | 管理者のユーザー名を検索するときに使用するクレーム。                                                                                 |
| `authenticated_groups_claim`<br> *オプション* | Array  | `["groups"]`                            | 認証されたグループを検索するときに使用するクレーム。                                                                                 |
| `scopes`<br> *オプション*                     | Array  | `["openid", "email", "offline_access"]` | IdP での認証中に使用するスコープ。`"openid"` と `"offline_access"` を含める必要があります。`admin_claim` が指定するクレームに必要なスコープも含める必要があります。 |
| `ssl_verify`<br> *オプション*                 | ブール    | `false`                                 | IDプロバイダーサーバー証明書を確認します。                                                                                     |

[プラグインのドキュメント](/hub/kong-inc/openid-connect/configuration/)も参照し、
要件に応じて構成を変更します。

Kong ManagerをOpenID Connectで認証する場合、認証状態を持続するためにプラグイン内部のセッションメカニズムが使用されます。詳細については、接頭辞が`session_`のパラメータ向けのドキュメントを参照してください。

### セッションのセキュリティを強化するための推奨事項

* `session_secret` を設定することをお勧めします。指定されていない場合は、ランダムに生成されたシークレットが使用されます。
* `session_cookie_secure`（デフォルト値は`false`）HTTPではなくHTTPSを使用する場合に有効にすることをおすすめします。
* Admin APIおよびKong Managerで同じドメインを使用している場合、[`session_cookie_same_site`](/hub/kong-inc/openid-connect/configuration/#config-session_cookie_same_site)から`Strict`へのアップグレードを検討してください。

これらの概念の詳細については、[Kong ManagerのSessionセキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照してください。

{% endif_version %}

{% if_version lte:3.5.x %}

**セッションプラグイン** （`admin_gui_session_conf`で構成）にはシークレットが必要であり、デフォルトで安全に構成されています。

* どのような状況でも、 `secret`は手動で文字列に設定する必要があります。
* HTTPS ではなく HTTP を使用する場合は、`cookie_secure`を手動で`false`に設定する必要があります。
{% if_version lte:3.1.x -%}
* Admin APIとKong Managerに異なるドメインを使用する場合は、`cookie_samesite`を`off`に設定する必要があります。
{% endif_version -%}
{% if_version gte:3.2.x -%}
* Admin APIとKong Managerに異なるドメインを使用する場合は、`cookie_same_site`を`Lax`に設定する必要があります。 {% endif_version %}

これらのプロパティの詳細については、[Kong ManagerのSessionセキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照し、[構成例](/gateway/{{page.release}}/kong-manager/auth/sessions/#example-configurations)を確認してください。

{% endif_version %}

`<>` で囲まれたエントリを、IdP に有効な値に置き換えます。
たとえば、Google の認証情報はここにあります:
[https://console.cloud.google.com/projectselector/apis/credentials](https://console.cloud.google.com/projectselector/apis/credentials)

管理者の作成
------

IDプロバイダーがログイン成功時に返すメールとユーザー名が
一致する管理者を作成してください。

```bash
curl -i -X POST http://localhost:8001/admins \
  --data username="<admin_email id="sl-md0000000">" \
  --data email="<admin_email id="sl-md0000000">" \
  --header Kong-Admin-Token:<rbac_token id="sl-md0000000">
```

たとえば、あるユーザーのメールアドレスが`example_user@example.com`の場合:

```bash
curl -i -X POST http://localhost:8001/admins \
  --data username="example_user@example_com" \
  --data email="example_user@example.com" \
  --header Kong-Admin-Token:<rbac_token id="sl-md0000000">
```

{:.note}
> 
> **注：** リクエストに入力されるアドミンのメールアドレスは招待メールを確実に受信するために使用されます。一方ユーザー名はプラグインがIdPと併用する属性です。

ロールを管理者に割り当て
------------

新しいアドミンに少なくとも1つのロールを割り当てて、Kongエンティティにログインしてアクセスできるようにします。

```bash
curl -i -X POST http://localhost:8001/admins/<admin_email id="sl-md0000000">/roles \
  --data roles="<role-name id="sl-md0000000">" \
  --header Kong-Admin-Token:<rbac_token id="sl-md0000000">
```

たとえば、以下の方法で、 `example_user@example.com`にスーパー管理者ロールをグラントします。

```bash
curl -i -X POST http://localhost:8001/admins/example_user@example.com/roles \
  --data roles="super-admin" \
  --header Kong-Admin-Token:<rbac_token id="sl-md0000000">
```

