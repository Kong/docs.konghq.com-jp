---
title: "Dev PortalでOpenID Connectを有効にします"
badge: "enterprise"
---
[OpenID Connect プラグイン](/hub/kong-inc/openid-connect/)（OIDC）では、Kong Dev Portal は Google、Okta、Microsoft Azure AD、[Curity](/hub/kong-inc/openid-connect/how-to/third-party/curity/#kong-dev-portal-authentication) などのサードパーティの *ID プロバイダー* （IdP）を使用して既存の認証設定に接続することができます。

[OIDC](/hub/kong-inc/openid-connect/)は、
Dev Portal File API リクエストに Cookie を使用する`session`メソッドで使用される必要があります。

さらに、OIDCを有効にするには構成オブジェクトが必要です。詳細については、このドキュメントの[サンプル構成オブジェクト](#/sample-configuration-object)セクションを参照してください。

{:.note}
> 
> **注** ：OIDC経由でのログイン時、Dev Portalは自動的に開発者アカウントを作成しません。
> `consumer_claim`設定パラメーターと一致する開発者アカウントを事前に作成および承認（自動承認が有効になっていない場合）する必要があります。
> <br><br>
> 登録フローでは、ユーザーIDPリダイレクトされたログインページでログイン情報を入力する必要があります。その後、ユーザーがDev Portalの
> 登録ページに移動すると、登録フォームにメールアドレスがあらかじめ入力されています。
> ユーザーはこの登録フォームでメールアドレスを変更することはできません。
> また、アカウント管理者によって設定された追加フィールドの入力を求められる場合があります。

Dev PortalのOIDCは、次のいずれかの方法で有効にできます。

* [ポータルSessionプラグインの構成](#portal-session-plugin-config)
* [サンプル設定オブジェクト](#sample-configuration-object)
* [Kong Managerを使用してOIDCを有効にする](#enable-oidc-using-kong-manager)
* [コマンドラインを使用して OIDC を有効にする](#enable-oidc-using-the-command-line)
* [kong.conf を使用して OIDC を有効にする](#enable-oidc-using-kongconf)

ポータルSessionプラグインの構成
-------------------

OpenID Connect を使用する場合、Sessionプラグイン構成は適用されません。

サンプル設定オブジェクト
------------

以下は、 *Google* をIDプロバイダーとして使用する場合の構成JSON Objectの例です。

    {
      "consumer_by": ["username","custom_id","id"],
      "leeway": 1000,
      "scopes": ["openid","profile","email","offline_access"],
      "logout_query_arg": "logout",
      "client_id": ["<client-id id="sl-md0000000">"],
      "login_action": "redirect",
      "logout_redirect_uri": ["http://localhost:8003/default"],
      "ssl_verify": false,
      "consumer_claim": ["email"],
      "forbidden_redirect_uri": ["http://localhost:8003/unauthorized"],
      "client_secret": ["<client_secret id="sl-md0000000">"],
      "issuer": "https://accounts.google.com/",
      "logout_methods": ["GET"],
      "login_redirect_uri": ["http://localhost:8003/default"],
      "login_redirect_mode": "query"
    }

上記のプレースホルダーは、実際の値に置き換えてください。

* `<client_id id="sl-md0000000">`\- IdPによって提供されるクライアントID
* `<client_secret id="sl-md0000000">`\- IdPによって提供されるクライアントシークレット
* `default`\- ワークスペース名

詳細については、[OpenID Connectプラグインのドキュメント](/hub/kong-inc/openid-connect/)を参照してください。

**重要：** `redirect_uri`はIdPで許可されたURIとして設定する必要があります。
構成オブジェクトで明示的に設定されていない場合、デフォルトのURIは
`http://localhost:8004/<workspace_name id="sl-md0000000">/auth`です。

`portal_gui_host`と`portal_api_url`がドメインを共有するように設定されているものの、サブドメインが異なる場合、OpenID Connectがセッションを正しく適用できるように、`redirect_uri`と`session_cookie_domain`が構成される必要があります。

以下に例を示します。

    {
      "consumer_by": ["username","custom_id","id"],
      "leeway": 1000,
      "scopes": ["openid","profile","email","offline_access"],
      "logout_query_arg": "logout",
      "client_id": ["<client_id id="sl-md0000000">"],
      "login_redirect_uri": ["https://example.portal.com/default"],
      "login_action": "redirect",
      "logout_redirect_uri": ["https://example.portal.com/default"],
      "ssl_verify": false,
      "consumer_claim": ["email"],
      "redirect_uri": ["https://exampleapi.portal.com/default/auth"],
      "session_cookie_domain": ".portal.com",
      "forbidden_redirect_uri": ["https://example.portal.com/unauthorized"],
      "client_secret": ["<CLIENT_SECRET"],
      "issuer": "https://accounts.google.com/",
      "logout_methods": ["GET"],
      "login_redirect_mode": "query"
    }


Kong Managerを使用してOIDCを有効にする
---------------------------

1. Dev Portal の **設定** ページに移動します。
2. **認証** タブで **認証プラグイン** を探します。
3. ドロップダウンから **OpenId Connect** を選択します。
4. **Auth Config（JSON）** フィールドのドロップダウンから **Custom** を選択します。
5. カスタマイズした[**構成JSONObject**](#/sample-configuration-object)を指定された テキストエリアに入力します。
6. **変更を保存** をクリックします。
> 
> **警告** Dev Portal認証が有効になっている場合、コンテンツファイルはロールが適用されるまで認証されません。`*`ロールが始まる`settings.txt`と`dashboard.txt`は、この例外です。詳細については、<a href="/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/developer-permissions">開発者のロールとコンテンツ権限</a>のセクションをご覧ください。

コマンドラインを使用して OIDC を有効にする
------------------------

Kong Admin APIを使用して、Dev Portal認証を設定できます。Dev Portalの認証プロパティを直接パッチするには、次のコマンドを実行します。

    curl -X PATCH http://localhost:8001/workspaces/<workspace NAME id="sl-md0000000"> \
      --data "config.portal_auth=openid-connect"
      "config.portal_auth_conf=<replace WITH JSON CONFIG OBJECT id="sl-md0000000">
> 
> **警告** Dev Portal認証が有効になっている場合、コンテンツファイルはロールが適用されるまで認証されません。`*`ロールが始まる`settings.txt`と`dashboard.txt`は、この例外です。詳細については、<a href="/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/developer-permissions">開発者のロールとコンテンツ権限</a>のセクションをご覧ください。

kong.conf を使用して OIDC を有効にする
---------------------------

Kongでは、 `portal_auth`プロパティを持つKong構成ファイルに
`default authentication plugin`を設定することができます。

`kong.conf` ファイルのプロパティを以下のように設定します。

    portal_auth="openid-connect"

次に、 `portal_auth_conf`プロパティを
カスタマイズされた[**構成 JSONObject**](#sample-configuration-object)に設定します。

これにより、ワークスペースに関係なく、初期化時にすべての Dev Portal がデフォルトで OIDC を使用するように設定されます。

