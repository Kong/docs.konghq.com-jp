---
title: "OktaとOIDCを使用して外部ポータルアプリケーション認証を設定する"
badge: "enterprise"
---
これらの手順は、Kong OIDCおよびPortal Application Registrationプラグインで使用するために、OktaをサードパーティのIDプロバイダーとして設定するのに役立ちます。

認可サーバーを定義し、Okta \{\#auth\-server\-cclaim\} でカスタムクレームを作成します
---------------------------------------------------------------

次の手順に従って、Oktaで、すべての認証タイプに対して認証サーバーを設定します。

1. [開発者向け Okta サイト](https://developer.okta.com/)にサインインします。

2. **セキュリティ > API > 認証サーバー** をクリックします。

   `default`という名前の認証サーバーがすでに設定されていることに注意してください。
   この例では、デフォルトの認証サーバーを使用しています。カスタム認証サーバーは、
   必要に応じていくつでも作成できます。詳細
   については、
   [Okta開発者向けドキュメント](https://developer.okta.com/docs/guides/customize-authz-server/overview/)を参照してください。
3. **デフォルト** をクリックすると、デフォルト認証サーバーの詳細が表示されます。認証サーバーとKongの関連付けに使用するため、`Issuer` URLをメモしておきます。

4. **クレーム** タブをクリックします。

5. **クレームの追加** をクリックします。`application_id`というカスタムクレームを追加して、正常に認証されたアプリケーションの`id`をアクセストークンに添付します。

   1. **名前** フィールドに `application_id` を入力します。
   2. `Include in token type`の選択が **アクセストークン** であることを確認します。
   3. **値** フィールドに`app.clientId`を入力します。
   4. **作成** をクリックします。

   カスタムクレームを作成したので、Application Registration プラグインを使用して `client_id` をサービスに関連付けることができます。まず、Kong Manager でサービスの作成から始めます。
6. サービスとルートを作成し、そのサービス上でOIDCプラグインをインスタンス化します。
   ほとんどのオプションでデフォルトの使用を許可できます。

   1. `Config.Issuer`フィールドにIDプロバイダーから入手した承認サーバーの発行者URLを入力します。

      ![Okta発行者URLを使用したOIDC](/assets/images/products/gateway/dev-portal/oidc-issuer-url.png)
   2. `Config.Consumer Claim`フィールドに、アプリケーションIDを含むOktaペイロードのフィールドの名前を入力します。
      これは通常`sub`です。

   **ヒント：** Oktaの検証ドキュメントには、デフォルトでサポートされている検証タイプがすべて含まれているわけではないため、`config.verify_parameters` オプションが無効になっていることを確認してください。

   ![OktaでOIDCの構成検証パラメーターをクリアする](/assets/images/products/gateway/dev-portal/oidc-clear-verify-params-app-reg.png)

   コア構成は次のようになります。

   ```json
   {
     "issuer": "<auth_server_issuer_url id="sl-md0000000">",
     "verify_credentials": false,
     "consumer_claim": "sub",
   }

   ```

7. サービス上で Portal Application Registration プラグインも構成します。[アプリケーション登録](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration#config-app-reg-plugin)をご覧ください。

Oktaにアプリケーションを登録
----------------

次の手順に従って、Oktaにアプリケーションを登録し、Kong Dev PortalのアプリケーションとOktaアプリケーションを関連付けます。

1. [開発者向け Okta サイト](https://developer.okta.com/)にサインインします。

2. **アプリケーション** > **アプリケーション** の順にクリックします。

3. 実装する認証フローに応じて、
   Okta アプリケーションは次のように異なります。

   * **新しいアプリ統合を作成** ：アプリケーションの種類を求められたら、`API Services`を選択します。 **新しいAPI Servicesアプリ統合** モーダルで、アプリ統合名を入力します。

   後ほど、[プロキシで認証をする](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/3rd-party-oauth#cc-flow)際に、`client_id` と `client_secret` が必要になります。
   * **Implicit Grant** ：アプリケーションの種類を求められたら、 `Single-Page App`、`Native`、または`Web`を選択します。`Allowed grant types`で`Implicit`が選択されていることを確認してください。アプリケーションのルーティングに応じて、 `Login redirect URIs` 、 `Logout redirect URIs` 、 `Initiate login URI`フィールドに正しい値を入力します。認可コードフローが可能な場合、Implicit Grantフローは推奨されません。

   * **認証コード** :アプリケーションのタイプを入力するように求められる場合は `Single-Page App`、`Native`、
     `Web` を選択してください。`Allowed grant types`に`Authorization Code`が選択されていることを確認してください。アプリケーションのルーティングに応じて、 `Login redirect URIs` 、 `Logout redirect URIs` 、 `Initiate login URI`フィールドに正しい値を入力します。

KongアプリケーションとIDプロバイダーアプリケーションとの関連付け
-----------------------------------

Oktaでアプリケーションが構成されました。次は、Kong Dev Portal内の対応するアプリケーションとOktaアプリケーションを関連付ける必要があります。

{:.important}
> 
> **注：** 開発者ごとに独自のアプリケーションをOktaとKongの両方に用意しておきます。  
> 各Oktaアプリケーションに付属する固有の`client_id`はKongの該当アプリケーションにマッピングします。
> このIDは基本的に、IDプロバイダーのアプリケーションをポータルアプリケーションにマッピングします。

この例では、クライアント認証情報がOAuthフローとして選択されていることを前提としています。

1. まだアカウントを作成していない場合は、Kong Dev Portalでアカウントを作成します。

2. ログインしたら、`My Apps`をクリックします。

3. アプリケーションページで `+ New Application` をクリックします。

4. **名前** と **説明** のフィールドに入力します。対応する Okta（またはその他の ID プロバイダー）アプリケーションの `client_id` を **リファレンス Id** フィールドに貼り付けます。

   ![Kongは参照ID付きのアプリケーションを作成します](/assets/images/products/gateway/dev-portal/create-app-ref-id.png)

アプリケーションが作成されたので、開発者はサポートされている中から推奨されている
[サードパーティのOAuthフロー](/gateway/{{page.release}}/kong-enterprise/dev-portal/authentication/3rd-party-oauth)を使用しているエンドポイントで認証できます。

