---
title: "OIDC 認証済みグループマッピング"
badge: "enterprise"
---
Kongの[OpenID Connectプラグイン](/hub/kong-inc/openid-connect/)（OIDC）を使用すると、IDプロバイダー（IdP）グループをKongロールにマッピングできます。この方法でKongにユーザーを追加すると、IdPのグループに基づいてユーザーがKongにアクセスできるようになります。

ID プロバイダ（IdP）グループをKongロールにマッピングすると、管理者アカウントが自動的に作成されます。ユーザー、グループ、ロールを個別に作成する必要はありません。その後、これらのユーザーはKong Managerへの参加招待を受け入れ、IdP認証情報でログインできるようになります。

{% if_version lte:2.7.x %}
{:.important}
> 
> **重要：** v2\.7\.xでは、以前のバージョンで必要とされた`consumer_claim`パラメータから `admin_claim`パラメータに
> 取って代わります。
> {% endif_version %}

{% if_version gte:3.6.x %}
{:.important}
> 
> **重要：** v3\.6\.xでは、`admin_gui_auth_conf`の下にあるパラメーターの多くが動作を変更しました。[変更を確認し](/gateway/{{page.release}}/kong-manager/auth/oidc/migrate/)、それに応じて構成を調整してください。
> {% endif_version %}

IdPで管理者のグループが変更されると、次にKong Managerにログインしたときに、Kong管理者アカウントの関連付けられたロールも{{site.base_gateway}}で変更されます。このマッピングにより、IdPが記録システムになるため、
{{site.base_gateway}}で主導でアクセスを管理する手間が省けます。

{% if_version gte:3.6.x %}
OIDCグループマッピングを使用すると、管理者に割り当てられたロールはIdPによって管理されます。管理者ロールの割当や割当解除を手動で行った場合、変更内容は次回のログインで上書きされます。{% endif_version %}

前提条件
----

* 認証サーバーとグループを割り当てられたユーザーを持つIdP
* [{{site.base_gateway}}がインストールおよび構成済み](/gateway/{{page.release}}/get-started/)
* Kong Manager が有効である
* RBAC有効
* （Kubernetesの）[Helm](https://helm.sh/docs/intro/install/)がインストールされている

OIDC認証マッピングの{{site.base_gateway}}への適用
----------------------

### 重要な価値を見直す

次の例では、`admin_claim` パラメータと `authenticated_groups_claim` パラメータを指定して、IdP から {{site.base_gateway}} にマッピングする管理者の値とロール名を識別し、`admin_auto_create_rbac_token_disabled` を指定して、Kong の管理者に対して RBAC トークンを作成するかどうかを指定します。

* `admin_claim`値は、Kong ManagerにマップするIdPユーザー名の値を指定します。ユーザーがIdPにログインするには、ユーザー名とパスワードが必要です。

* `authenticated_groups_claim`の値は、 {{site.base_gateway}}ロールを特定の {{site.base_gateway}}管理者割り当てるために
  どのIdPクレームを使用すべきか指定します。

  この値はIdPによって異なります。たとえば、Oktaはクレームを`groups`として構成し、別のIdPはクレームを`roles`として構成する場合があります。

  IdPでは、グループクレーム値は`<workspace_name id="sl-md0000000">:<role_name id="sl-md0000000">`形式に従う必要があります。

  たとえば、`"authenticated_groups_claim": ["groups"]`が指定され、IdPで`groups:["default:super-admin"]`が指定される場合、`admin_claim`で指定されるアドミンはデフォルトの{{site.base_gateway}}ワークスペースでスーパーアドミンロールに割り当てられます。

  マッピングが期待どおりに機能しない場合は、IdPが作成したJWTをデコードし、管理者IDトークンに、この例ではkey:valueペア`groups:["default:super-admin"]`、またはIdPに設定されている適切なクレーム名とクレーム値が含まれていることを確認します。
* `admin_auto_create_rbac_token_disabled`ブール値は、OpenID Connectを使用して管理者を自動的に作成するときに、RBACトークンの作成を有効または無効にします。デフォルトは`false`です。

  * 管理者の自動トークン作成を無効にするには、 `true`に設定します
  * `false` に設定すると、管理者向けトークンの自動生成が有効になります

* `admin_auto_create`ブール値はOpenID Connect を使用し管理者の自動作成を
  有効または無効にします。デフォルトは`true`です。

  * `true`に設定すると、アドミンの自動生成が有効になります
  * `false`に設定すると、管理者の自動生成が無効になります

### マッピングの設定

{% navtabs %}
{% navtab Kubernetes with Helm %}

1. OIDCプラグインの設定ファイルを作成し、
   `admin_gui_auth_conf`のように保存します。

   中括弧 \( `{}` \) で示されるすべてのフィールドに独自の値を入力します。

   ```json
   {                                      
       "issuer": "{YOUR_IDP_URL}",        
       "admin_claim": "email",
       "client_id": ["{CLIENT_ID}"],                 
       "client_secret": ["{CLIENT_SECRET}"],
       "authenticated_groups_claim": ["{CLAIM_NAME}"],
       "ssl_verify": false,
       "leeway": 60,
       "redirect_uri": ["{YOUR_REDIRECT_URI}"],
       "login_redirect_uri": ["{YOUR_LOGIN_REDIRECT_URI}"],
       "logout_methods": ["GET", "DELETE"],
       "logout_query_arg": "logout",
       "logout_redirect_uri": ["{YOUR_LOGOUT_REDIRECT_URI}"],
       "scopes": ["openid","profile","email","offline_access"],
       "auth_methods": ["authorization_code"],
       "admin_auto_create_rbac_token_disabled": false,
       "admin_auto_create": true
   }
   ```

   すべての OIDC パラメータの詳細説明については、[OpenID Connect パラメータに関する参考資料](/hub/kong-inc/openid-connect/#configuration-parameters)を参照してください。
2. 作成したばかりのファイルからシークレットを作成します。

   ```sh
   kubectl create secret generic kong-idp-conf --from-file=admin_gui_auth_conf -n kong
   ```

3. 次のパラメータを使用して、デプロイメントの `values.yml` ファイルの RBAC セクションを更新します。

   ```yaml
   rbac:
     enabled: true
     admin_gui_auth: openid-connect
     session_conf_secret: kong-session-conf   
     admin_gui_auth_conf_secret: kong-idp-conf
   ```

4. Helmを使用して、YAMLファイル名でデプロイをアップグレードします。

   ```sh
   helm upgrade --install kong-ee kong/kong -f ./myvalues.yaml -n kong
   ```

{% endnavtab %}
{% navtab Docker %}

Dockerがインストールされている場合は、次のコマンドを実行して必要な環境変数を設定し、{{site.base_gateway}}構成を再読み込みします。

中括弧 \( `{}` \) で示されるすべてのフィールドに独自の値を入力します。

```sh
echo "
  KONG_ENFORCE_RBAC=on \
  KONG_ADMIN_GUI_AUTH=openid-connect \
  KONG_ADMIN_GUI_AUTH_CONF='{
      \"issuer\": \"{YOUR_IDP_URL}\",
      \"admin_claim\": \"email\",
      \"client_id\": [\"<someid id="sl-md0000000">\"],
      \"client_secret\": [\"<somesecret id="sl-md0000000">\"],
      \"authenticated_groups_claim\": [\"{CLAIM_NAME}\"],,
      \"ssl_verify\": false,
      \"leeway\": 60,
      \"redirect_uri\": [\"{YOUR_REDIRECT_URI}\"],
      \"login_redirect_uri\": [\"{YOUR_LOGIN_REDIRECT_URI}\"],
      \"logout_methods\": [\"GET\", \"DELETE\"],
      \"logout_query_arg\": \"logout\",
      \"logout_redirect_uri\": [\"{YOUR_LOGOUT_REDIRECT_URI}\"],
      \"scopes\": [\"openid\",\"profile\",\"email\",\"offline_access\"],
      \"auth_methods\": [\"authorization_code\"],
      \"admin_auto_create_rbac_token_disabled\": false,
      \"admin_auto_create\": true
    }' kong reload exit" | docker exec -i {KONG_CONTAINER_ID} /bin/sh
```

`{KONG_CONTAINER_ID}` をコンテナの ID に置き換えます。

上記で使用されるすべてのパラメータ、および他の多数のカスタマイズオプションの詳細な説明については、[OpenID Connectパラメータに関する参考資料](/hub/kong-inc/openid-connect/#configuration-parameters)を参照してください。

{% endnavtab %}
{% navtab kong.conf %}

1. `kong.conf`ファイルに移動します。

2. RBACを有効にした状態で、`admin_gui_auth`プロパティと`admin_gui_auth_conf`プロパティをファイルに追加します。

   中括弧 \( `{}` \) で示されるすべてのフィールドに独自の値を入力します。

       enforce_rbac = on
       admin_gui_auth = openid-connect
       admin_gui_auth_conf = {                                      
           "issuer": "{YOUR_IDP_URL}",        
           "admin_claim": "email",
           "client_id": ["{CLIENT_ID}"],                 
           "client_secret": ["{CLIENT_SECRET}"],
           "authenticated_groups_claim": ["{CLAIM_NAME}"],
           "ssl_verify": false,
           "leeway": 60,
           "redirect_uri": ["{YOUR_REDIRECT_URI}"],
           "login_redirect_uri": ["{YOUR_LOGIN_REDIRECT_URI}"],
           "logout_methods": ["GET", "DELETE"],
           "logout_query_arg": "logout",
           "logout_redirect_uri": ["{YOUR_LOGOUT_REDIRECT_URI}"],
           "scopes": ["openid","profile","email","offline_access"],
           "auth_methods": ["authorization_code"],
           "admin_auto_create_rbac_token_disabled": false,
           "admin_auto_create": true
       }

   上記で使用されるすべてのパラメータ、および他の多数のカスタマイズオプションの詳細な説明については、[OpenID Connectパラメータに関する参考資料](/hub/kong-inc/openid-connect/#configuration-parameters)を参照してください。
3. {{site.base_gateway}}を再起動して、ファイルを適用します。

   ```sh
   kong restart -c /path/to/kong.conf
   ```

{% endnavtab %}
{% endnavtabs %}

