---
title: "Dev PortalでBasic Authを有効にします"
badge: "enterprise"
---
Kong Dev Portalは、HTTPプロトコルのBasic Authenticationスキームを使用して完全にまたは部分的に認証できます。リクエストは、`Basic`という単語の後にbase64でエンコードされた`username:password`文字列が続く認証ヘッダー付きで送信されます。

Dev PortalのBasic Authenticationは以下のいずれかの方法を使用して有効にできます。

* [Kong Manager](#enable-basic-auth-using-kong-manager)
* [コマンドライン](#enable-basic-auth-using-the-command-line)
* [Kong構成ファイル](#enable-basic-auth-using-kongconf)

**警告：** 

* Dev Portalで認証を有効にするには、Sessionプラグインを使用する必要があります。これが適切に設定されていない場合、開発者はログインできません。詳細については、
  [Dev Portalでのセッション](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/sessions)を参照してください。

* Dev Portal認証が有効になっていると、コンテンツファイルはロールが適用されるまで認証されません。例外は`settings.txt`と`dashboard.txt`で、`*`の役割で始まります。詳細については、
  [開発者の役割とコンテンツの権限](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/developer-permissions)をご覧ください。

ポータルSession構成を有効にする
-------------------

Kong設定ファイルで、 `portal_session_conf` プロパティを設定します。

    portal_session_conf={ "cookie_name":"portal_session","secret":"<change_this id="sl-md0000000">","storage":"kong"}

テスト中にHTTPを使用する場合は、構成に`"cookie_secure": false`を含めます。

    portal_session_conf={ "cookie_name":"portal_session","secret":"<change_this id="sl-md0000000">","storage":"kong","cookie_secure":false}

{% if_version lte:3.1.x %}
または、`portal_api_url` と `portal_gui_host` に異なるサブドメインがある場合は、`cookie_domain` および `cookie_samesite` プロパティを次のように設定します。

    portal_session_conf={ "cookie_name":"portal_session","secret":"<change_this id="sl-md0000000">","storage":"kong","cookie_secure":false,"cookie_domain":"<.your_subdomain.com>","cookie_samesite":"off"  }

{% endif_version %}
、{% if_version gte:3.2.x %}
または、`portal_api_url` と `portal_gui_host` に異なるサブドメインがある場合は、`cookie_domain`および `cookie_same_site` プロパティを次のように設定します。

    portal_session_conf={ "cookie_name":"portal_session","secret":"<change_this id="sl-md0000000">","storage":"kong","cookie_secure":false,"cookie_domain":"<.your_subdomain.com>","cookie_same_site":"Lax"  }

{% endif_version %}

### 関連項目

* ポータルセッション構成の詳細については、[セッション](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/sessions#portal-session-conf)を参照してください。

* Dev Portalセッション内のドメインとクッキーの詳細については、[ドメイン](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/sessions#domains)を参照して
  ください。

Kong Manager を使用して Basic Auth を有効化
----------------------------------

1. Dev Portal の **設定** ページに移動します。
2. **認証** タブで **認証プラグイン** を探します。
3. **Basic Authentication** を選択します。
4. **変更を保存** をクリックします。

![認証プラグイン](/assets/images/products/gateway/dev-portal/portal-settings-auth-plugin.png)

コマンドラインを使用したBasic Authの有効化
--------------------------

Dev Portalの認証プロパティを直接パッチするには、次のコマンドを実行します。

```bash
curl -X PATCH http://localhost:8001/workspaces/<workspace_name id="sl-md0000000"> \
  --data "config.portal_auth=basic-auth"
```

次を使用してBasic Authを有効化：`kong.conf`
--------------------------------

Kongでは、デフォルトの認証プラグインを
`portal_auth`プロパティを持つKong構成ファイル内に設定することができます。

`kong.conf` ファイルのプロパティを以下のように設定します。

    portal_auth="basic-auth"

これにより、すべてのDev Portalは、初期化時にデフォルトでBasic Authenticationを使用するように設定されます。

