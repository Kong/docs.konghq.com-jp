---
title: "以前の設定からの移行"
badge: "enterprise"
---
Gateway v 3\.6以降、Kong ManagerはOpenID Connectプラグインのセッション管理メカニズムを使用します。OIDCで認証する場合、`admin_gui_session_conf`は不要になりました。代わりに、セッション関連の構成パラメータは`admin_gui_auth_conf`（`session_secret`など）で設定されます。

`admin_gui_auth_conf`
のセッション関連のパラメータの一部のデフォルト値が `admin_gui_session_conf` のものと異なりまので、設定を確認することをお勧めします。

admin\_gui\_auth\_conf
-------------------------

以下の `admin_gui_auth_conf` の属性に対する変更の概要を参照し、詳細については個々のリンクに従ってください。

パラメータ \| 古い動作 \| 新しい動作
\-\-\-\|\-\-\-\|\-\-\-\|\-\-\-\|\-\-\-
[`scopes`](#scopes) \| 古いデフォルト：`["openid", "profile", "email"]` \| 新しいデフォルト：`["openid", "email", "offline_access"]`
[`admin_claim`](#admin_claim) \| 必須 \| オプション（デフォルト：`"email"`）
[`authenticated_groups_claim`](#authenticated_groups_claim) \| 必須 \| オプション（デフォルト：`["groups"]`）
[`redirect_uri`](#redirect_uri) \| Kong Managerを指すURLの配列を受け取る \| Admin APIの`/auth`エンドポイントを指すURLの配列を受け取る
[`login_redirect_uri`](#login_redirect_uri) \| オプション \| 必須
[`logout_redirect_uri`](#logout_redirect_uri) \| オプション \| 必須

次のパラメータは必須ではなく、内部で制御されるため、指定された値は無視されます。

* `auth_methods`
* `login_action`
* `login_methods`
* `login_tokens`
* `logout_methods`
* `logout_query_arg`
* `logout_revoke_access_token`
* `logout_revoke_refresh_token`
* `logout_revoke`
* `refresh_tokens`
* `upstream_access_token_header`
* `upstream_id_token_header`
* `upstream_user_info_header`（`search_user_info`が`true`の間）

### スコープ

Kong ManagerでOpenID Connectプラグインを使用する間、`scopes`のデフォルト値は指定されていない場合、`["openid", "email", "offline_access"]`になります。

* `openid`: OpenID Connectに必須です。
* `email`：ユーザーのメールアドレスを取得し、IDトークンに含めます。
* `offline_access`: アクセストークンとセッションを更新するための必須の更新トークン。

このパラメータは、必要に応じて変更できます。ただし、OpenID Connectプラグインを正常に動作させるため、`"openid"`と`"offline_access"`は、常に含めてください。また、`scopes`にこのパラメータで指定されたクレームに十分なスコープが含まれていることを確認してください（例：デフォルトのスコープの `"email"`）。

### admin\_claim

`admin_claim`は、オプションのパラメータになりました。設定されていない場合、デフォルトは `"email"`です。

このパラメータは、ID トークンから管理者のユーザー名を検索するときに使用されます。この設定を構成する際、`scopes` にこのパラメータで指定されたクレームに対する十分なスコープが含まれていることを確認してください。

### authenticated\_groups\_claim

`authenticated_groups_claim`は、オプションのパラメータになりました。設定されていない場合、デフォルトは `["groups"]`です。

このパラメータは、ID トークンから管理者の関連グループを検索するときに使用されます。

### redirect\_uri

`redirect_uri`は、Admin APIの認証ポイント`<admin_api id="sl-md0000000">/auth`を指すURLの配列として設定されるようになったはずです（例：`["http://localhost:8001/auth"]`）。これまで、 `redirect_uri`はKong Managerを指すURLのリストでした（例：`["http://localhost:8002"]` ）。

ユーザーは、IdP のクライアント/アプリケーション設定を更新して、 `<admin_api id="sl-md0000000">/auth`（たとえば、 `http://localhost:8001/auth`）がリダイレクト URI の許可リストに含まれることを確認することをお勧めします。

### login\_redirect\_uri

`login_redirect_uri`は、IdPで認証後に宛先を設定するための **必須** パラメータになりました。これは常にKong Managerを指すURLの配列である必要があります。
（例：`["http://localhost:8002"]` ）。

### logout\_redirect\_uri

`logout_redirect_uri`は、IdPからログアウトした後に宛先を構成するための **必須** のパラメータとなりました。これは常にKong Managerを指すURLの配列である必要があります。
\(例：`["http://localhost:8002"]` \)。

以前はKong Managerは、ユーザーがログアウトをリクエストしたときに、IdPから[RP開始ログアウト](https://openid.net/specs/openid-connect-rpinitiated-1_0.html#RPLogout)を実行しませんでした。ゲートウェイv3\.6以降、Kong Managerはユーザーログアウト時にRP開始ログアウトを実行します。

admin\_gui\_session\_conf
----------------------------

OpenID Connectプラグインにセッション管理メカニズムが組み込まれたため、`admin_gui_session_conf`
は、OIDC による認証では使用されなくなりました。過去にOIDCの`admin_gui_session_conf`経由でセッション管理を構成していた場合
設定も更新する必要があります。

さらに、一部のパラメータのデフォルト値が変更されました。
詳細については、以下を参照してください。

古いパラメータ名と場所 \| 新しいパラメータ名と場所 \| 古いデフォルト \| 新しいデフォルト
\-\-\-\|\-\-\-\|\-\-\-\|\-\-\-\|\-\-\-
[`admin_gui_session_conf.secret`](#secret) \| `admin_gui_auth_conf.session_secret` \| \-\- \| \-\- \|
[`admin_gui_session_conf.cookie_secure`](#cookie_secure) \| `admin_gui_auth_conf.session_cookie_secure`\| `true` \| `false` \|
[`admin_gui_session_conf.cookie_samesite`](#cookie_samesite) \| `admin_gui_auth_conf.session_cookie_same_site` \| `Strict` \| `Lax` \|

### シークレット

これを `admin_gui_auth_conf.session_secret`で設定する必要があります。

設定されていない場合、 {{site.base_gateway}}はランダムにシークレットを生成します。

### cookie\_secure

これを `admin_gui_auth_conf.session_cookie_secure`で設定する必要があります。

以前は、指定されていない場合は`cookie_secure` `true`に設定されていました。ただし、`admin_gui_auth_conf.session_cookie_secure`は、現在デフォルト値が`false`です。HTTPではなくHTTPSを使用している場合は、セキュリティ強化のためにこのオプションを有効にすることを推奨します。

### cookie\_samesite

これを `admin_gui_auth_conf.session_cookie_same_site`で設定する必要があります。

以前は、指定されていない場合は`cookie_samesite` `Strict`に設定されていました。ただし、`admin_gui_auth_conf.session_cookie_same_site`は、現在デフォルト値が`Lax`です。Admin APIとKong Managerに同じドメインを使用している場合は、セキュリティ強化のために、この設定を`Strict`にアップグレードすることを推奨します。

[Cookie の SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#samesitesamesite-value) の詳細を確認する。

