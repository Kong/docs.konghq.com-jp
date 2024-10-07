---
title: "Kong ManagerのBasic Authを有効にする"
badge: "enterprise"
content_type: "how-to"
---
Kong Managerインスタンスで[ベーシック認証](/hub/kong-inc/basic-auth/)を有効にします。

前提条件
----

[スーパー管理者権限](/gateway/{{page.release}}/kong-manager/auth/super-admin/)があるか、
`/admins`と`/rbac`の読み取りと書き込みアクセス権を持つユーザーがいる

ベーシック認証を設定
----------

1. [`kong.conf`](/gateway/{{page.release}}/reference/configuration/)で、次のプロパティを構成します。

       enforce_rbac = on
       admin_gui_auth = basic-auth
       admin_gui_session_conf = { "secret":"set-your-string-here" }

   これによりRBACが有効になり、`basic-auth`が認証方法として設定され、セッションシークレットが作成されます。

   Kong Managerはバックグラウンドでセッションプラグインを使用します。
   このプラグイン（`admin_gui_session_conf`で構成）にはシークレットが必要であり、デフォルトで安全に構成されています。
   * どのような状況でも、`secret`は手動で文字列に設定する必要があります。
   * HTTPSの代わりにHTTPを使用する場合は、`cookie_secure`を手動で`false`に設定する必要があります。
   {% if_version lte:3.1.x -%}
   * Admin API と Kong Manager に異なるドメインを使用する場合は、`cookie_samesite` を `off` に設定する必要があります。
   {% endif_version -%}
   {% if_version gte:3.2.x -%}
   * Admin APIとKong Managerに異なるドメインを使用する場合、`cookie_same_site`を`Lax`に設定する必要があります。 {% endif_version %}

   これらのプロパティの詳細については、[Kong Manager の Session セキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照し、[構成例](/gateway/{{page.release}}/kong-manager/auth/sessions/#example-configurations)を確認してください。
2. Kongを起動または再読み込みし、`kong.conf`ファイルを指定します。

       kong start [-c /path/to/kong/conf]

3. 次のいずれかのオプションを選択します。

   * データベース移行でスーパー管理者を作成した場合は、
     ユーザー名`kong_admin`と[環境変数](/gateway/{{page.release}}/production/access-control/start-securely/)で設定したパスワードでKong Managerにログインします。

   * Kong Managerの **チーム** タブからスーパー管理者を作成した場合、
     [スーパー管理者を作成する方法](/gateway/{{page.release}}/kong-manager/auth/super-admin/)に記載されているように、
     メール招待を受け取った後に作成した認証情報でログインします。

