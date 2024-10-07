---
title: "アプリケーション登録向けサードパーティ OAuth2 サポート"
badge: "enterprise"
---
サードパーティOAuth2サポートを使用すると、開発者は任意の[サポートされるIDプロバイダー](#idps)を選択して、アプリケーション認証情報管理を一元化できます。外部IdP機能を使用するには、`portal_app_auth`構成オプションを`kong.conf.default`構成ファイルの`external-oauth2`に設定します。詳細については、[認証プロバイダーストラテジ](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/auth-provider-strategy)の設定を参照してください。

Kong [OIDC](/hub/kong-inc/openid-connect/)と
[Portal Application Registration](/hub/kong-inc/application-registration/)
プラグインはサービス上で相互に連携して使用されます。

* OIDCプラグインはOAuth2ハンドシェイクのあらゆる側面を処理します。
  コンシューマを検索する
  [カスタムクレーム](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/okta-config#auth-server-cclaim)
  （`custom_id`はIDプロバイダーの`client_id`クレームと一致します）。

* Application Registrationプラグインは、マップされたコンシューマの確認を担当し、コンシューマがルートにアクセスするために正しいACL（アクセスコントロールリスト）の権限を持っていることを確実にします。

サポートされているIDプロバイダー \{\#idps\}
-------------------------------

Kong OIDCプラグインは、多くのIDプロバイダーを最初からサポートしています。
次のプロバイダーは、Kong OIDCプラグインと連携して使用される現在のバージョンのKong
Portal Application Registrationプラグインでテスト済みです。

* [Okta](https://developer.okta.com/)。[Oktaセットアップの例](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/okta-config/)を参照してください。
* [Azure](https://azure.microsoft.com/) [Azure セットアップの例](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/azure-oidc-config/)を参照してください。
* [Ping Identity](https://www.pingidentity.com/)。

### リソース

サービスでの認証方法は、その基盤となるOAuth2実装によって異なります。詳細については、実装したIDプロバイダとOAuthフローに関する以下のドキュメントを参照してください。

* Okta

  * [認証コードフロー](https://developer.okta.com/docs/guides/implement-auth-code/overview/)
  * [認証コードフロー（PKCE）](https://developer.okta.com/docs/guides/implement-auth-code-pkce/overview/)
  * [クライアント認証情報フロー](https://developer.okta.com/docs/guides/implement-client-creds/overview/)
  * [Implicit Grant Flow](https://developer.okta.com/docs/guides/implement-implicit/overview/)

* Azure

  * [認証コードフロー](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow)
  * [クライアント認証情報フロー](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
  * [Implicit Grant Flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-implicit-grant-flow)

* Ping Identity

  * [Oauth2開発者ガイド](https://www.pingidentity.com/developer/en/resources/oauth-2-0-developers-guide.html)

サポートされているOAuthフロー
-----------------

* クライアント認証情報（[RFC 6742 セクション 4\.4](https://tools.ietf.org/html/rfc6749#section-4.4)）
* 認証コード（[RFC 6742 セクション4\.1](https://tools.ietf.org/html/rfc6749#section-4.1)）
* 暗黙的な許可（[RFC 6742 セクション 4\.2](https://tools.ietf.org/html/rfc6749#section-4.2)）
* パスワードグラント（[RFC 6742セクション4\.3](https://tools.ietf.org/html/rfc6749#section-4.3)）

パスワードグラントと[黙示的なグラントフロー](https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead)は使用できますが、推奨されません。承認コードとクライアント認証情報フローより安全性が低いためです。

### クライアント認証情報フロー \{\#cc\-flow\}

OIDC プラグインは、クライアント認証情報を使用した認証を非常に簡単に有効化します。
このフローはサーバー側で使用され、マシン間通信を
セキュアに使用する必要があります。クライアント認証情報フローでは、
アプリケーションの`client_secret`を保存および送信することを承認する当事者が必要です。

このフローでは、OIDCと適用されたアプリケーション登録プラグインを使用して、開発者がサービスに
対してリクエストを行います。このリクエストは、Basic Auth認証ヘッダーとして`client_id`および
`client_secret`を含まなければなりません。

`Authorization: Basic client_id:client_secret`

`client_id:client_secret`はbase64でエンコードされている必要があります。

次のシーケンス図は、OIDCプラグインと Application Registration プラグインを使ったクライアント認証情報フローを示しています。画像をクリックすると拡大表示されます。

![クライアント認証情報フロー](/assets/images/products/gateway/dev-portal/dp-appreg-3rdparty-ccflow.png)

| 手順 | 説明                                                                                                                                         |
|:---|:-------------------------------------------------------------------------------------------------------------------------------------------|
| a  | 開発者は、Okta アプリケーションの `client_id` と `client_secret` をルートに送信します。OIDC プラグインは、このリクエストを Okta 認証サーバーのエンドポイントにプロキシします。                             |
| b  | Oktaは`client_id`と`client_secret`を読み取り、アクセストークンを生成します。認証サーバーは、Oktaアプリケーションの`client_id`とのキーと値のペアであるカスタムクレーム`application_id`を挿入するように設定されています。 |
| c  | OktaはアクセストークンをKongに返します。                                                                                                                   |
| d  | OIDCプラグインは結果のアクセストークンを読み取り、 `application_id`カスタムクレームを介してリクエストをアプリケーションに関連付けます。                                                             |
| e  | 解決されたアプリケーションにPortal Application Registrationプラグイン経由でサービスを消費する権限がある場合、Kongはアップストリームにリクエストを転送します。                                           |

### 認証コードフロー \{\#ac\-flow\}

OIDC プラグインの制限により、1 つのプラグインインスタンスでは
複数のソース（アプリケーション）にプロビジョニングされる動的な `client_id's` を処理できません。
この問題の回避策として、[`show_issuer`](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration#app-reg-params) が 
Application Registration プラグインで有効になっている場合、IdP 発行者 URL が 
Dev Portal のアプリケーションページで開発者に公開されています。開発者は発行者 URL に直接アクセスして
アクセストークンをプロビジョニングできます。アクセストークンを取得したら、プロキシに対してリクエストする
ことができます。

1. IdPに対してアクセストークンを直接保護するようにアプリケーションを設定します。
   Oktaでの認証コードフローの実装の詳細については、
   [Okta開発者ガイド](https://developer.okta.com/docs/guides/implement-auth-code/overview/)を参照してください。

2. 最初のアクセストークンのハンドシェイクが完了したら、そのアクセストークンを[ベアラートークン](https://tools.ietf.org/html/rfc6750#section-2.1)として使用して、Kongサービスに対して後続のリクエストを行います。最初のリクエストが成功すると、OIDCプラグインはクライアントとのセッションを確立するため、リクエストのたびにアクセストークンを継続的に渡す必要がなくなります。

次のシーケンス図は、OIDCプラグインとApplication Registrationプラグインを通じた認証コードフローを示しています。画像をクリックすると拡大表示されます。

![認証コードフロー](/assets/images/products/gateway/dev-portal/dp-appreg-3rdparty-authcodeflow.png)

| 手順 | 説明                                                                                                                                                            |
|:---|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| a  | 開発者はターゲットサービスの `issuer_id` をコピーします。これは、Dev Portal アプリケーションビューのサービス詳細（Service Details）ページで公開できます。開発者は、このエンドポイントにユーザーの認証とアクセストークンの取得をリクエストするようにアプリケーションを設定できます。 |
| b  | Oktaはユーザーをログインページにリダイレクトします。                                                                                                                                  |
| c  | ユーザーはシングルサインオン（SSO）情報を入力します。                                                                                                                                  |
| d  | ユーザーは、Oktaユーザー名とパスワードを含むSSOフォームを送信します。                                                                                                                        |
| e  | ログインに成功すると、アプリケーションには、以降のすべてのリクエストの呼び出しに使用するためのアクセストークンが与えられます。                                                                                               |
| f  | ユーザーは、保護されたサービスとルートにリクエストを送信します。                                                                                                                              |
| g  | OIDCプラグインは、アクセストークンを受け入れ、イントロスペクションを実行して、必要に応じてOkta認証サーバーに問い合わせます。アクセストークンが検証されると、このプラグインはカスタムクレームを照合し、`custom_id`を介して関連するアプリケーションコンシューマを見つけます。               |
| h  | リクエストは Application Registration プラグインに渡され、コンシューマに適切な ACL（アクセス制御リスト）権限があるかどうかがチェックされます。                                                                        |
| i  | リクエストはアップストリームにプロキシされます。                                                                                                                                      |

### Implicit Grant Flow

認可コードフローが可能な場合、Implicit Grantフローは推奨されません。

1. IdPに対してアクセストークンを直接保護するようにアプリケーションを設定します。
   OktaでのImplicit Grantフローの実装の詳細については、
   [Okta開発者ガイド](https://developer.okta.com/docs/guides/implement-implicit/use-flow/)を参照してください。

2. アクセストークンのハンドシェイクが完了したら、そのアクセストークンをベアラートークンとして使用して、Kong
   サービスに対して後続のリクエストを行います。最初のリクエストに成功すると、OIDCプラグインはクライアントとのセッションを確立します。そのため、アクセストークンは継続的に渡される必要はありません。

