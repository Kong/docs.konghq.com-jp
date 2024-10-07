---
title: "Dev Portalでキー認証を有効にする"
badge: "enterprise"
---
Kong Dev Portalは、APIキーまたは **Key Authentication** を使用して、完全または部分的に認証できます。ユーザーは登録時に固有のキーを入力し、このキーを使用して
Dev Portalにログインします。

Dev PortalのKey Authenticationは、次の3つの方法で有効にできます。

* [Kong Manager](#enable-key-auth-via-kong-manager) 経由
* [コマンドライン](#enable-key-auth-via-the-command-line)経由
* [Kong構成ファイル](#enable-key-auth-via-the-kongconf)経由
> 
> **警告:** Dev Portalで認証を有効にするには、Sessionプラグインを使用する必要があります。これが適切に設定されていない場合、開発者はログインできません。詳細情報は[Dev Portalのセッション](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/sessions/)を参照してください。

ポータルSession構成の有効化
-----------------

    portal_session_conf={ "cookie_name": "portal_session", "secret": "CHANGE_THIS", "storage": "kong" }

テスト中にHTTPを使用する場合は、構成に`"cookie_secure": false`を含めます。

    portal_session_conf={ "cookie_name": "portal_session", "secret": "CHANGE_THIS", "storage": "kong", "cookie_secure": false }

Kong Manager経由のKey Authの有効化
---------------------------

1. Dev Portalの **設定** ページに移動します。
2. **認証** タブで **認証プラグイン** を探します。
3. ドロップダウンから **Key Authentication** を選択します。
4. **変更を保存** をクリックします。
> 
> **警告** Dev Portal認証が有効になっている場合、コンテンツファイルはロールが適用されるまで認証されません。`*`ロールで始まる`settings.txt`と`dashboard.txt`は、この例外です。詳細については、<a href="/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/developer-permissions">開発者のロールとコンテンツ権限</a>セクションを参照してください。

コマンドラインによるKey Auth の有効化
-----------------------

Dev Portal の認証プロパティを直接パッチするには、次のコマンドを実行します。

    curl -X PATCH http://localhost:8001/workspaces/<workspace NAME id="sl-md0000000"> \
      --data "config.portal_auth=key-auth"
> 
> **警告** Dev Portal認証が有効になっている場合、コンテンツファイルはロールが適用されるまで認証されません。`*`ロールで始まる`settings.txt`と`dashboard.txt`は、この例外です。詳細については、<a href="/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/developer-permissions">開発者のロールとコンテンツ権限</a>セクションを参照してください。

Kong.confによるKey Authの有効化
------------------------

Kongでは、 `portal_auth`プロパティを持つKong構成ファイルに
`default authentication plugin`を設定することができます。

`kong.conf` ファイルのプロパティを以下のように設定します。

    portal_auth="key-auth"

これにより、ワークスペースに関係なく、初期化時にすべてのDev Portalがデフォルトで
Key Authenticationを使用するように設定されます。

