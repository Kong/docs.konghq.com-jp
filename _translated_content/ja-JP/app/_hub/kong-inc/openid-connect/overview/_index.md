---
nav_title: "概要"
---
OpenID Connect \([1\.0](http://openid.net/specs/openid-connect-core-1_0.html)\) プラグイン（OIDCとも呼ばれます）は、標準化された方法でサードパーティのアイデンティティプロバイダ（IdP） との統合を可能にします。このプラグインは、Kongを（プロキシされた）[OAuth 2\.0](https://tools.ietf.org/html/rfc6749)のリソースサーバー（RS）やクライアントとアップストリームサービス間のOpenID Connectリライングパーティ（RP）として実装するために使用できます。

このプラグインは、数種類の認証情報と権限をサポートしており、いくつかのOpenID Connectプロバイダでテストされています。
すべてを確認するには、[サポートリファレンス](/hub/kong-inc/openid-connect/support/)を参照してください。

OpenID Connectについて
------------------

### OpenID Connectは何をするものですか?

OpenID Connectは、 **アイデンティティプロバイダ \(IdP\)** との **フェデレーション** を形成する方法を提供します。アイデンティティプロバイダは、アカウント認証情報を保存するサードパーティです。
アイデンティティプロバイダがユーザーを認証すると、アプリケーションはそのプロバイダを信頼し、ユーザーへのアクセスを許可します。
IDプロバイダは、ユーザーの認証情報を保護し、信頼性を確保する責任を負うため、アプリケーションが単独でその責任を負う必要がなくなります。

OpenID Connectは、アイデンティティプロバイダに責任を委譲するだけでなく、ユーザーのローカルマシンに認証情報を保存せずにシングルサインオンを可能にします。

最後に、企業は1つの中央記録システムで多数のアプリケーションのアクセス制御を管理する必要が生じる場合があります。
たとえば、従業員がEメールアドレスとパスワードを使用してさまざまなアプリケーションにアクセスできるようにしたい場合などです。
また、アクセスを一元的に変更する必要（たとえば、従業員の退職や役職の変更など）が生じるかもしれません。
OpenID Connectは、さまざまなアプリケーションが同じサードパーティ ID プロバイダを通じてユーザーを認証する方法を提供することで、この問題に対処します。

### KongのOpenID Connectプラグインは何をするものですか?

開発者は、OpenID Connectによって認証を他の当事者にオフロードできるようになるのと同様に、Kongによってプロセス全体をアプリケーションから分離できるようになります。 *サービス内* でOpenID Connectのコードを手書きする必要はなく、Kongをサービスの前に配置し、Kongに認証を処理させることができます。
この分離により、開発者はアプリケーション内のビジネスロジックに集中できるようになります。
また、フロントドアでの認証を維持したまま、簡単にサービスを入れ替えることができ、 *新しい* サービスにも同じ認証を簡単に展開することができます。

Kongユーザーは、Key AuthやBasic Authなどの他の認証タイプよりもOpenID Connectを好むかもしれません。なぜなら、パスワードを保存するデータベースを管理する必要がないためです。代わりに、選択した信頼できるIDプロバイダにタスクを委任します。

OpenID Connectプラグインはさまざまなユースケースに適合し、JWT（JSON Web Token）や0Auth 2\.0など他のプラグインにも拡張できますが、最も一般的な使用例は、[認証コードフロー](/hub/kong-inc/openid-connect/how-to/authentication/authorization-code-flow/)です。

使用ガイドとデモ
--------

{% include_cached /md/plugins-hub/oidc-prod-note.md %}

フローやグラントを実施する前に、[重要な設定パラメータ](/hub/kong-inc/openid-connect/primary-config-params/)を確認することをお勧めします。

### 認証フローとグラント

[HTTPie](https://httpie.org/) を使用して例を実行します。読みやすくするために出力は省略されています。[httpbin.orgは](https://httpbin.org/)アップストリームサービスとして使用されます。

プラグインをテストするときはAdmin APIを使用すると便利ですが、宣言形式でも同様の設定をセットアップできます。

このプラグインが複数のグラント/フローで構成されている場合、認証情報の順序はハードコードされます。

1. [Session認証](/hub/kong-inc/openid-connect/how-to/authentication/session/)
2. [JWTアクセストークン認証](/hub/kong-inc/openid-connect/how-to/authentication/jwt-access-token/)
3. [Kong OAuthトークン認証](/hub/kong-inc/openid-connect/how-to/authentication/kong-oauth-token/)
4. [イントロスペクション認証](/hub/kong-inc/openid-connect/how-to/authentication/introspection/)
5. [ユーザー情報認証](/hub/kong-inc/openid-connect/how-to/authentication/user-info/)
6. [リフレッシュトークンの付与](/hub/kong-inc/openid-connect/how-to/authentication/refresh-token)
7. [パスワードの付与](/hub/kong-inc/openid-connect/how-to/authentication/password-grant/)
8. [クライアント認証情報の付与](/hub/kong-inc/openid-connect/how-to/authentication/client-credentials/)
9. [認証コードフロー](/hub/kong-inc/openid-connect/how-to/authentication/authorization-code-flow/)

認証情報が見つかると、プラグインはそれ以上の検索を停止します。複数のグラントが同じ認証情報を共有する場合があります。たとえば、パスワードとクライアント認証情報の付与は、どちらも`Authorization`ヘッダーを介した基本的なアクセス認証を使用できます。

### 承認

OpenID Connectプラグインには、粗粒度の認証を行うための機能がいくつかあります。

1. [クレームベースの承認](/hub/kong-inc/openid-connect/how-to/authorization/claims/)
2. [ACLプラグインの認証](/hub/kong-inc/openid-connect/how-to/authorization/acl/)
3. [コンシューマの承認](/hub/kong-inc/openid-connect/how-to/authorization/consumer/)

### Kong ManagerでのOIDC認証


{{site.base_gateway}}はOpenID Connectを使用してKong Managerを保護できます。
バックグラウンドでOpenID Connectプラグインを使用して、Kong Manager管理者の認証を組織の OpenID Connect Identityプロバイダにバインドする機能を提供します。

プラグインを直接設定する必要はありません。代わりに、 {{site.base_gateway}}が`kong.conf`の設定を通じてOIDCプラグインにアクセスします。

OIDCを使用してKong ManagerでRBACを設定するには、以下を参照してください。

* [Kong ManagerのOIDCを有効にする](/gateway/latest/kong-manager/auth/oidc/configure/)
* [OIDC認証済みグループマッピング](/gateway/latest/kong-manager/auth/oidc/mapping/)

{% if_version lte:3.4.x %}

### Dev PortalでのOIDC認証

OpenID Connectプラグインを使用することで、Kong Dev Portalは、Google、Okta、Microsoft Azure AD、CurityなどのサードパーティのIDプロバイダを使用して既存の認証設定に接続できるようになります。

OIDCは、Dev PortalファイルAPIリクエストにCookieを利用する`session`メソッドで使用する必要があります。

プラグインを直接設定する必要はありません。代わりに、 {{site.base_gateway}} `kong.conf`の設定または{{site.konnect_short_name}}からOIDCプラグインにアクセスします。


{{site.konnect_short_name}}:

* [Dev PortalのAzure IdPを構成する](/konnect/dev-portal/access-and-approval/azure/)
* [シングルサインオン](/konnect/dev-portal/customization/#single-sign-on)

自己管理型{{site.base_gateway}}：

* [Dev PortalでOpenID Connectを有効にする](/gateway/latest/kong-enterprise/dev-portal/authentication/oidc/)
* [Dev Portal向けCurityを使用したOIDC](/hub/kong-inc/openid-connect/how-to/third-party/curity/#kong-dev-portal-authentication)

#### アプリケーション登録

Dev Portalでのアプリケーション登録にOIDCを使用することもできます。

{{site.konnect_short_name}}でのアプリケーション登録：

* [動的クライアント登録のためのAuth0の構成](/konnect/dev-portal/applications/dynamic-client-registration/auth0/)
* [動的クライアント登録のためのCurityの構成](/konnect/dev-portal/applications/dynamic-client-registration/curity/)
* [動的クライアント登録のためのOktaの構成](/konnect/dev-portal/applications/dynamic-client-registration/okta/)

自己管理型{{site.base_gateway}}でのアプリケーション登録：

* [アプリケーション登録プラグイン](/hub/kong-inc/application-registration/)
* [アプリケーション登録のためのサードパーティOAuth2サポート](/gateway/latest/kong-enterprise/dev-portal/authentication/3rd-party-oauth/)
* [Azure ADとOIDCを使用した外部ポータルアプリケーション認証](/gateway/latest/kong-enterprise/dev-portal/authentication/azure-oidc-config/)
* [OktaとOIDCを使用した外部ポータルアプリケーション認証の設定](/gateway/latest/kong-enterprise/dev-portal/authentication/okta-config/) {% endif_version %}

### 詳細情報

* [サポートされている認証情報、グラント、テスト済みのIdP](/hub/kong-inc/openid-connect/support/)
* [OIDC構成を開始するための重要なパラメータ](/hub/kong-inc/openid-connect/primary-config-params/)
* [構成リファレンス](/hub/kong-inc/openid-connect/configuration/)
* [基本的な構成例](/hub/kong-inc/openid-connect/how-to/basic-example/)
* [OpenID ConnectプラグインAPI参照](/hub/kong-inc/openid-connect/api/)
* [ヘッダー](/hub/kong-inc/openid-connect/how-to/headers/)
* [ログアウト](/hub/kong-inc/openid-connect/how-to/logout/)
* [記録](/hub/kong-inc/openid-connect/how-to/records/)
* [デバッグ](/hub/kong-inc/openid-connect/how-to/debugging/)

