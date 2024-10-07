RBAC を有効にするには、 {{site.base_gateway}}最初にインストールして移行を実行したときに使用した初期の KONG\_PASSWORD が必要になります。これはスーパー管理者のデフォルトのパスワードでもあり、RBAC がオンになると必要になります。

{% navtabs %}
{% navtab UNIX-based system or Windows %}

1. `kong.conf`ファイルの以下の構成設定を変更します。`/etc/kong/kong.conf`ファイルに移動します。

   ```sh
   cd /etc/kong/
   ```

2. フォールバックできる作業コピーがあることを確認するために、 `kong.conf.default`ファイルをコピーします。

   ```sh
   cp kong.conf.default kong.conf
   ```

3. 次に、 `kong.conf`で次の設定を編集します。

       echo >> “enforce_rbac = on” >> /etc/kong/kong.conf
       echo >> “admin_gui_auth = basic-auth” >> /etc/kong.conf
       echo >> “admin_gui_session_conf = {"secret":"secret","storage":"kong","cookie_secure":false}”

   これによりRBACがオンになり、{{site.base_gateway}}にベーシック認証（ユーザー名/パスワード）を使用するよう指示し、セッションCookieの作成方法をSessionプラグインに指示します。

   クッキーは有効期限が切れるまで、ユーザーを認証するための後続のすべてのリクエストに使用されます。セッションの有効期間は限られており、構成可能な間隔で更新されるため、セッション終了後に攻撃者が古いクッキーを取得して使用することを防ぐことができます。
4. {{site.base_gateway}}を再起動し、以下のように新しい設定ファイルを指定します。

       kong restart -c /etc/kong/kong.conf

{% endnavtab %}
{% navtab Docker %}

Dockerがインストールされている場合は、次のコマンドを実行して必要な環境変数を設定し、ゲートウェイ構成を再読み込みします。

{:.note}
> 
> **注：** `{KONG-CONTAINER-ID}`をコンテナのIDに確実に置き換えてください。

```bash
echo "KONG_ENFORCE_RBAC=on
KONG_ADMIN_GUI_AUTH=basic-auth
KONG_ADMIN_GUI_SESSION_CONF='{\"secret\":\"secret\",\"storage\":\"kong\",\"cookie_secure\":false}'
kong reload exit" | docker exec -i {KONG_CONTAINER_ID} /bin/sh
```

これによりRBACがオンになり、{{site.base_gateway}}にベーシック認証（ユーザー名/パスワード）を使用するよう指示し、セッションCookieの作成方法をSessionプラグインに指示します。

クッキーは有効期限が切れるまで、ユーザーを認証するための後続のすべてのリクエストに使用されます。セッションの有効期間は限られており、設定可能な間隔で更新されるため、セッション終了後に攻撃者が古いクッキーを取得して使用することを防ぐことができます。

{% endnavtab %}
{% endnavtabs %}

{% if_version lte:2.8.x %}
このガイド以外では、インストールに応じてこれらの設定を異なる方法で変更する必要がある可能性があります。これらの設定の詳細については、[Kong ManagerのBasic Auth](/gateway/latest/kong-manager/auth/basic/)を参照してください。
{% endif_version %}

{% if_version gte:3.0.x %}
このガイド以外では、インストールに応じてこれらの設定を異なる方法で変更する必要がある可能性があります。これらの設定の詳細については、[Kong ManagerのBasic Auth](/gateway/{{page.release}}/kong-manager/auth/basic/)を参照してください。
{% endif_version %}

