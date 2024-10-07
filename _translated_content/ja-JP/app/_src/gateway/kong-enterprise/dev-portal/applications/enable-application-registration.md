---
title: "アプリケーション登録の有効化"
badge: "enterprise"
---
アプリケーション登録により、Kong Dev Portalに登録された開発者は、Kong上のサービスに対してサポートされている認証プラグインを使用して認証を行うことができます。{{site.base_gateway}}または外部IDプロバイダー管理者は、Kong Managerを使用してサービスへのアクセスを選択的に許可できます。

前提条件
----

* Dev Portalは、同じワークスペースでサービスとして有効になります。
* サービスはHTTPSで作成され、有効になります。
* 認証はDev Portalで有効になっています。
* アプリケーション、サービス、および開発者に対する読み取りおよび書き込み権限を持つ管理者としてログインします。
* `portal_app_auth`設定オプションは OAuth プロバイダおよびストラテジ（ `kong-oauth2`デフォルトまたは`external-oauth2` ）用に設定されています。Portal Application Registrationプラグインについては、[認証プロバイダーストラテジの設定](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/auth-provider-strategy)を参照してください。
* サポートされているサードパーティIDプロバイダーをOIDCプラグインで使用する場合、認証プロバイダーが構成されます。
  * OktaをIDプロバイダーとして使用する手順の例については、[Oktaの例](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/okta-config/)を参照してください。
  * Azure ADをIDプロバイダーとして使用する手順の例については、 [Azureの例](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/azure-oidc-config/)を参照してください。

Kong Managerを使用してサービスへのアプリケーション登録を有効にする \{\#enable\-app\-reg\-plugin\}
----------------------------------------------------------------------------

サービスでアプリケーション登録を使用する場合は、Portal Application Registrationプラグインを有効にします。

Kong Managerで、アプリケーション登録を有効にするサービスにアクセスします。

1. ワークスペースの左側のナビゲーションペインで、 **API Gateway> Services** に移動します。

2. サービスページで、サービスを選択し、 **表示** をクリックします。

3. サービスページのプラグインペインで、 **プラグインを追加** をクリックします。

4. 新しいプラグインの追加ページの認証セクションで、
   **Portal Application Registration** プラグインを見つけ、 **有効化** をクリックします。

5. 構成設定を入力します。次のセクション
   [アプリケーション登録設定パラメータ](#application-registration-configuration-parameters)のパラメータを使って、
   フィールドを完成させてください。

   {:.important}
   > 
   > **重要：** 発行者のURL公開は、
   > サードパーティのIDプロバイダー向けに構成された
   > ワークフローである[認証コードフロー](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/3rd-party-oauth/#ac-flow)に不可欠です。
6. **作成** をクリックします。

### Application Registration 構成パラメータ \{\#app\-reg\-params\}

| フォームパラメータ      | 説明                                                                                                                                                                                                       |
|:---------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Service`      | このプラグイン構成の対象となるサービス。必須。                                                                                                                                                                                  |
| `Tags`         | グループ化およびフィルタリングのための、カンマで区切られた文字列のセット。任意。                                                                                                                                                                 |
| `Auto Approve` | 有効にすると、すべての新しいサービス契約リクエストが自動的に承認されます。そうでない場合、Dev Portal管理者はリクエストを手動で承認する必要があります。デフォルトは`false`です。                                                                                                         |
| `Description`  | Dev Portalのサービス情報に表示される説明。任意。                                                                                                                                                                            |
| `Display Name` | Dev Portalのサービス情報に表示される一意の名前。必須。                                                                                                                                                                         |
| `Show Issuer`  | 「サービスの詳細」ページに発行者URLが表示されます。デフォルト：`false`。 **重要：** **発行者のURLを公開することは** 、[サードパーティのIDプロバイダ向けに構成された認証コードフローワークフローに不可欠です](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/3rd-party-oauth/#ac-flow)。 |

次の手順
----

[認証ストラテジ](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/auth-provider-strategy/)
を選択し、適切なプラグイン（OAuth2、Key Authentication、またはOpenID Connect）を構成します。

