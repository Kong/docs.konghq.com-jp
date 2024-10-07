---
title: "Azure AD と OIDC を使用して外部ポータルアプリケーション認証を設定する"
badge: "enterprise"
---
これらの手順は、Kong OIDC および Portal Application Registration プラグインで使用するために、Azure AD をサードパーティのアイデンティティプロバイダーとして設定するのに役立ちます。

前提条件
----

* `portal_app_auth`構成オプションは、OAuthプロバイダーとストラテジ（`kong-oauth2`または`external-oauth2`）向けに構成されています。 Portal Application Registrationプラグインについては、[認証プロバイダーストラテジの構成](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/auth-provider-strategy/)を参照してください。 

Azureでアプリケーションを作成する
-------------------

1. Azure内で **アプリ登録** サービスにアクセスして、新しいアプリケーションを登録してください。

2. **証明書&シークレット** で、クライアントシークレットを作成し、安全な場所に保存します。シークレットは一度だけ表示できます。

3. **マニフェスト** の下で、 `accessTokenAcceptedVersion=2`を更新します \(デフォルトは null\)。
   アプリケーションの JSON は、次の例のようになります。

Kongでサービスを作成
------------

```bash
curl -i -X PUT http://localhost:8001/services/httpbin-service-azure \
  --data 'url=https://httpbin.org/anything'
```

Kongでルートを作成
-----------

```bash
curl -i -X PUT http://localhost:8001/services/httpbin-service-azure/routes/httpbin-route-azure \
  --data 'paths=/httpbin-azure'
```

OIDC および Application Registration プラグインをサービスにマッピング
--------------------------------------------------

OpenID ConnectおよびApplication Registrationプラグインを **サービス** マッピングします。プラグインが適切に機能するには、サービスに適用する必要があります。

1. サービスの OIDC プラグインを構成します。

   ```bash
   curl -X POST http://localhost:8001/services/httpbin-service-azure/plugins \
     --data name=openid-connect \
     --data config.issuer="https://login.microsoftonline.com/<your_tenant_id id="sl-md0000000">/v2.0" \
     --data config.display_errors="true" \
     --data config.client_id="<your_client_id id="sl-md0000000">" \
     --data config.client_secret="<your_client_secret id="sl-md0000000">" \
     --data config.redirect_uri="https://example.com/api" \
     --data config.consumer_claim=aud \
     --data config.scopes="openid" \
     --data config.scopes="YOUR_CLIENT_ID/.default" \
     --data config.verify_parameters="false"
   ```

   詳細については、 [OIDC プラグイン](/hub/kong-inc/openid-connect/)を参照してください。
2. サービスのApplication Registrationプラグインを設定します。

   ```bash
   curl -X POST http://localhost:8001/services/httpbin-service-azure/plugins \
     --data "name=application-registration"  \
     --data "config.auto_approve=true" \
     --data "config.description=Uses consumer claim with various values (sub, aud, etc.) as registration id to support different flows and use cases." \
     --data "config.display_name=For Azure" \
     --data "config.show_issuer=true"
   ```

3. クライアント認証情報ワークフローを使用してAzureからアクセストークンを取得し、トークンを
   JSON Web Token \(JWT\) に変換します。プレースホルダーの値を以下の実際の値に置き換えます。
   `<your_tenant_id id="sl-md0000000">` 、 `<your_client_id id="sl-md0000000">` 、 `<your_client_secret id="sl-md0000000">` 、および
   `<admin-hostname id="sl-md0000000">` .

   ```bash
   curl -X POST https://login.microsoftonline.com/<your_tenant_id id="sl-md0000000">/oauth2/v2.0/token \
     --data scope="<your_client_id id="sl-md0000000">/.default" \
     --data grant_type="client_credentials" \
     --data client_id="<your_client_id id="sl-md0000000">" \
     --data client_secret="<your_client_secret id="sl-md0000000">" \
   ```

4. アクセストークンを JWTトークンに変換します。

   1. 前の手順で取得したアクセストークンを[JWT](https://jwt.io) に貼り付けます。

   2. 「 **Share JWT** 」をクリックして、[aud（audience）](https://tools.ietf.org/html/rfc7519#section-4.1.3)リクエストの値をクリップボードにコピーします。次の手順では、`aud`値を **参照ID** として使用します。

Kongでアプリケーションを作成
----------------

1. Dev Portalにログインして、新しいアプリケーションを作成します。

   1. **マイアプリ** メニュー \-> **新しいアプリケーション** を選択します。
   2. Azureアプリケーションの **名前** を入力します。
   3. JWTに生成された`aud`値を、 **参照ID** フィールドに貼り付けます。
   4. （任意） **説明** を入力します。

2. **作成** をクリックします。

3. アプリケーションを作成したら、必ずサービスを有効にしてください。アプリケーションダッシュボードの「サービス」セクションで、使用するサービスの **Activate** をクリックします。

   関連するApplication Registrationプラグインで[自動承認](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration##aa)を有効にしているため、管理者がリクエストを承認する必要はありません。

Azure アプリケーションで認証フローをテスト
------------------------

Azure AD 実装でクライアントの認証情報または認証コードフローをテストするには、
次の手順に従ってください。

1. トークンを取得します。

   ```bash
   curl -X POST "https://login.microsoftonline.com/<your_tenant_id id="sl-md0000000">/oauth2/v2.0/token" \
   --data scope="<your_client_id id="sl-md0000000">/.default" \
   --data grant_type="client_credentials" \
   --data client_id="<your_client_id id="sl-md0000000">" \
   --data client_secret="<your_client_secret id="sl-md0000000">"
   ```

2. 認可ヘッダー内のトークンを使用してデータを取得します。

   ```bash
   curl --header 'Authorization: bearer <token_from_above id="sl-md0000000">' '<admin-hostname id="sl-md0000000">:8000/httpbin-azure'
   ```

   `<token_from_above id="sl-md0000000">`を、前の手順で生成したbearerトークンに置き換えます。
3. 認証コードフローをテストします。ブラウザで、`http://localhost:8000/httpbin-azure`に移動します。

   Azureにログインすると、ブラウザに結果が表示されます。

トラブルシューティング
-----------

問題が発生した場合は、データプレーン（DP）のログを確認してください。
OpenID Connectプラグインで`display_errors=true`を有効にしたため
問題を正確に特定できるよう、より詳細なエラーメッセージが表示されます。

