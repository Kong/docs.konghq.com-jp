---
title: "アプリケーション登録の認証プロバイダ戦略"
badge: "enterprise"
---
{{site.base_gateway}}を使用するか、Application Registrationプラグインのあるレコードの外部システムを使用できます。

選択した認証ストラテジを使ってDev Portal Application Registrationプラグインを有効にするには、`kong.conf`で`portal_app_auth`構成オプションを設定する必要があります。

* `kong-oauth2`: デフォルト。{{site.base_gateway}} は記録システムです。Application
  Registration プラグインは[OAuth2](/hub/kong-inc/oauth2/)、または、
  [Key Authentication](/hub/kong-inc/key-auth/) プラグインと組み合わせて使用されます。

  {:.note}
  > 
  > **注** ：OAuth2プラグインは、従来のデプロイメントで
  > のみ使用できます。OAuth2プラグインにはすべてのゲートウェイインスタンスでデータベースが必要なので、
  > ハイブリッドモードまたはDBレスデプロイメントでは
  > 使用できません。
* `external-oauth2`：外部IdPは記録システムです。Portal Application Registrationプラグインは、[OpenID Connect（OIDC）](/hub/kong-inc/openid-connect/)プラグインと組み合わせて使用されます。`external-oauth2`オプションは、どのデプロイメントタイプでも使用できます。

  第三者認可ストラテジ（`external-oauth2`）は、{{site.base_gateway}}クラスタに
  あるすべてのワークスペース（Dev Portals）のすべてのアプリケーションに適用されます。

{{site.base_gateway}}認証ストラテジの使用
---------------

デフォルトの `kong-oauth2` 認証ストラテジで {{site.base_gateway}} を記録システムとして使用している場合は、次の手順でアプリ登録を設定してください。

1. サービスで[Application Registrationプラグイン](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration/)を有効にします。

2. Application Registration プラグインと同じサービスで、[OAuth2](/hub/kong-inc/oauth2/) プラグインまたは[Key Auth](/hub/kong-inc/key-auth/) プラグインのいずれかを構成します。

   OAuth2 プラグインはハイブリッドモードでは使用できません。

外部ポータル認証を設定 \{\#set\-external\-oauth2\}
--------------------------------------------

外部 IdP \( `external-oauth2` \) を使用している場合は、次の手順に従ってください。

1. [推奨ワークフロー](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/3rd-party-oauth#supported-oauth-flows)に目を通し、1つ選択します。

2. `kong.conf.default`を開き、`portal_app_auth`オプションを選択したストラテジに設定します。
   以下の構成例では、デフォルト（`kong-oauth2`）から外部IdP（`external-oauth2`）にストラテジを切り替えています。

       portal_app_auth = external-oauth2
                # Dev Portal application registration
                # auth provider and strategy. Must be set to configure
                # authentication in conjunction with the application_registration plugin.
                # Currently accepts kong-oauth2 or external-oauth2.

3. {{site.base_gateway}}インスタンスを再起動します。

       kong reload

4. サービスで[Application Registrationプラグイン](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration/)を有効にします。

5. Application Registration プラグインと同じサービスで[OIDC プラグイン](/hub/kong-inc/openid-connect/)を構成します。

6. IDプロバイダーをアプリケーションに設定し、{{site.base_gateway}}に
   アプリケーションを設定して、互いに関連付けてください。[Okta](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/okta-config/)
   または
   [Azure](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/azure-oidc-config/)セットアップの例を参照してください。

