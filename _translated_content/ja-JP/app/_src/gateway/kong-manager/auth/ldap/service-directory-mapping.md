---
title: "LDAP サービスディレクトリグループを Kong ロールにマッピングする"
badge: "enterprise"
---
サービスディレクトリマッピングにより、組織は{{site.base_gateway}}での認証と承認にLDAPディレクトリを使用できるようになります。

必要な構成で{{site.base_gateway}}を開始した後、ユーザー名がLDAPディレクトリ内のユーザー名と一致する新しい管理者を作成できます。その後、これらのユーザーはKong Managerへの招待を受け入れ、LDAP認証情報でログインできるようになります。

Kongでのサービスディレクトリマッピングの仕組みは以下のとおりです。

* ロールは、Admin APIまたはKong Managerを使用して{{site.base_gateway}}に作成されます。
* グループが作成され、ロールがグループに関連付けられます。
* ユーザーがKong Managerにログインすると、所属するグループに基づいて権限が付与されます。

たとえば、サービスディレクトリでユーザーのグループが変更されると、次にKong Managerにログインしたときに、 {{site.base_gateway}}でKong管理者アカウントに関連付けられたロールも変更されます。このマッピングにより、サービスディレクトリが記録システムになるため、{{site.base_gateway}}で手動でアクセスを管理する手間が省けます。

{% if_version gte:3.6.x %}

LDAPサービスディレクトリマッピングを使用すると、管理者に割り当てられたロールはサービスディレクトリによって管理されます。
管理者ロールの割当や割当解除を手動で行った場合、変更内容は次回のログインで上書きされます。

{% endif_version %}

前提条件
----

* {{site.base_gateway}}がインストールおよび構成済み
* Kong Managerのアクセス
* ローカルの LDAP ディレクトリがある

LDAP認証を有効化
----------

LDAPディレクトリを認証と承認に使用するようにサービスディレクトリマッピングを設定します。

1. {{site.base_gateway}}を起動します。シェルから、次のように入力します。

       kong start [-c /path/to/kong/conf]

2. LDAP Authenticationを有効にし、Kong ManagerのRBACを適用するには、次のプロパティで[`kong.conf`](/gateway/{{page.release}}/reference/configuration/)を通じてKongを構成します。

       enforce_rbac = on
       admin_gui_auth = ldap-auth-advanced
       admin_gui_session_conf = { "secret":"set-your-string-here" }

   Kong Managerはバックグラウンドでセッションプラグインも使用します。このプラグイン（`admin_gui_session_conf`で構成）にはシークレットが必要であり、デフォルトで安全に構成されています。
   * どのような状況においても、`secret` は手動で文字列に設定する必要があります。
   * HTTPSではなくHTTPを使用する場合は、`cookie_secure`を手動で`false`に設定する必要があります。{% if_version lte:3.1.x -%}
   * Admin APIとKong Managerに異なるドメインを使用する場合、`cookie_samesite`を`off`に設定する必要があります。
   {% endif_version -%}
   {% if_version gte:3.2.x -%}
   * Admin APIとKong Managerに異なるドメインを使用する場合、`cookie_same_site`を`Lax`に設定する必要があります。 {% endif_version %}

   これらのプロパティの詳細については、[Kong ManagerのSessionセキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照し、[構成例](/gateway/{{page.release}}/kong-manager/auth/sessions/#example-configurations)を確認してください。

LDAP認証の構成
---------

次のプロパティを使用してKong ManagerのLDAP認証を構成します。以下に定義する属性変数に注意してください。

    admin_gui_auth_conf = {
     "anonymous":"", \
     "attribute":"<enter_your_attribute_here id="sl-md0000000">", \
     "bind_dn":"<enter_your_bind_dn_here id="sl-md0000000">", \
     "base_dn":"<enter_your_base_dn_here id="sl-md0000000">", \
     "cache_ttl": 2, \
     "header_type":"Basic", \
     "keepalive":60000, \
     "ldap_host":"<enter_your_ldap_host_here id="sl-md0000000">", \
     "ldap_password":"<enter_your_ldap_password_here id="sl-md0000000">", \
     "ldap_port":389, \
     "start_tls":false, \
     "timeout":10000, \
     "verify_ldap_host":true, \
     "consumer_by":["username", "custom_id"], \
     "group_base_dn":"<enter_your_group_base_dn_here id="sl-md0000000">",
     "group_name_attribute":"<enter_your_group_name_attribute_here id="sl-md0000000">",
     "group_member_attribute":"<enter_your_group_member_attribute_here id="sl-md0000000">",
    }

|            属性            |                                                                                               説明                                                                                                |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `attribute`              | LDAPユーザーを識別するために使用される属性。<br>たとえば、LDAPユーザーをユーザー名で管理者にマッピングするには、 `uid`を設定します。                                                                                                                     |
| `bind_dn`                | LDAPバインドDN（識別名）。ユーザーのLDAP検索を実行するために使用されます。この`bind_dn`には、認証中のユーザーを検索する権限が必要です。<br>たとえば、`uid=einstein,ou=scientists,dc=ldap,dc=com`。                                                              |
| `base_dn`                | LDAPベースDN（識別名）。<br>`ou=scientists,dc=ldap,dc=com`はその一例です。                                                                                                                                       |
| `ldap_host`              | LDAPホストドメイン。 <br>たとえば、 `ec2-XX-XXX-XX-XXX.compute-1.amazonaws.com` 。                                                                                                                            |
| `ldap_port`              | デフォルトのLDAP ポートは389です。636はSSL LDAPおよびADに必要なポートです。`ldaps`が構成されている場合は、ポート636を使用する必要があります。より複雑なActive Directory（AD）環境では、ドメインコントローラーとポート389の代わりに、グローバルカタログのホストとポートの使用を検討してください。デフォルトではポート3268を使用します。 |
| `ldap_password`          | LDAP パスワード。 <br>他の構成プロパティと同様に、機密情報は構成ファイルに直接書き込むのではなく、環境変数として設定できます。                                                                                                                            |
| `group_base_dn`          | LDAPがグループの検索を開始するエントリの識別名を設定します。<br> デフォルトは`conf.base_dn`の値です。                                                                                                                                  |
| `group_name_attribute`   | グループの名前を保持する属性を設定します。通常、 `name` （Active Directoryの場合）または`cn` （OpenLDAP の場合）と呼ばれます。<br>デフォルトは`conf.attribute`からの値です。                                                                             |
| `group_member_attribute` | LDAPグループのメンバーを保持する属性を設定します。<br>デフォルトは `memberOf` です。                                                                                                                                            |

ロール権限を定義する
----------

Admin APIの[RBAC エンドポイント](/gateway/api/admin-ee/latest/#/rbac/put-rbac-roles-name_or_id)を使用するか、Kong Managerの[チームページ](/gateway/{{page.release}}/kong-manager/auth/rbac/add-user/)を使用して、 {{site.base_gateway}}で権限を持つロールを定義します。次のいずれかを使用して、どのKongロールがサービスディレクトリの各グループに対応するかを手動で定義する必要があります。

* Kong Managerのディレクトリマッピングセクション。 **Teams** > **Groups** タブにあります。
* Admin APIのディレクトリマッピングエンドポイントを使用します。


{{site.base_gateway}}はサービスディレクトリに書き込みません。たとえば、 {{site.base_gateway}}管理者はディレクトリ内にユーザーまたはグループを作成できません。ユーザーとグループを{{site.base_gateway}}にマッピングする前に、個別に作成する必要があります。

ユーザーと管理者のマッピング
--------------

サービス ディレクトリ ユーザーを Kong 管理者にマップするには、管理者のユーザー名を`admin_gui_auth_conf`で構成された属性に対応する **名前** の値にマップします。[Kong Manager](/gateway/{{page.release}}/kong-manager/auth/rbac/add-admin/)で管理者アカウントを作成するか、[Admin API](/gateway/api/admin-ee/latest/#/admins/post-admins)を使用します。

ブートストラップされたスーパーアドミンとディレクトリユーザーでペアを組む方法に関する手順については、[ディレクトリユーザーを最初のスーパーアドミンとして設定](#set-up-a-directory-user-as-the-first-super-admin)を参照してください。

既にロールが割り当てられている管理者がいる状態で、代わりにグループマッピングを使用する場合は、最初にすべてのロールを削除する必要があります。サービスディレクトリは、ユーザー権限の記録システムとして機能します。割り当てられたロールは、グループからマップされたロールに加えて、ユーザーの権限にも影響します。

グループロールの割り当て
------------

サービスディレクトリマッピングを使用すると、グループがロールにマッピングされます。ユーザーがログインすると、管理者のユーザー名で識別され、サービスディレクトリで一致するユーザー認証情報で認証されます。
その後、サービスディレクトリ内のグループは、組織が定義した関連する役割と自動的に対応付けられます。

### 例

1. Example Corpは、サービスディレクトリグループ`T1-Mgmt`を Kongのロールのスーパー管理者にマッピングします。
2. Example Corpは、 `example-user`という名前のサービスディレクトリユーザーを、同じ名前`example-user`のKong管理者アカウントにマッピングします。
3. ユーザー`example-user`は、LDAP ディレクトリ内のグループ`T1-Mgmt`に割り当てられています。

`example-user`がKong Managerに管理者としてログインすると、マッピングの結果として自動的にスーパー管理者ロールが与えられます。

Example Corpが`T1-Mgmt`への割り当てを削除して`example-user`の権限を取り消すことにした場合、ログインしようとするとスーパー管理者の役割が失われます。

ディレクトリユーザーを最初のスーパー管理者として設定
--------------------------

Kong は、ディレクトリユーザーを最初のスーパー管理者として設定することを推奨しています。

この例では、一意の識別子（UID）を使用して構成された属性と、スーパー管理者にしたいディレクトリユーザーが、`UID=example-user` という識別名（DN）エントリを持っていることを示しています。

```sh
curl -i -X PATCH https://localhost:8001/admins/kong_admin \
  --header 'Kong-Admin-Token: <rbac_token id="sl-md0000000">' \
  --header 'Content-Type: application/json' \
  --data '{"username":"example-user"}'
```

このユーザーはログインできますが、`example-user` に属するグループをロールにマップするまで、ユーザーはディレクトリを認証にのみ使用します。`example-user` が属するグループにスーパー管理者ロールをマップすると、`example-user` 管理者からスーパー管理者ロールを削除できます。選択するグループは、ディレクトリ内で「スーパー」である必要があります。そうでない場合、他の管理者が例えば「従業員」グループなどの一般的なグループでログインすると、彼らもスーパー管理者になってしまいます。

{:.important}
> 
> **重要** ：唯一のadminからスーパー管理者ロールを削除し、そのスーパー管理者ロールをadminが属するグループにまだマッピングしていない場合、Kong Managerにログインできません。

代替案:

* RBACをオフにしてKongを起動し、グループをスーパー管理者ロールにマッピングしてから、そのグループに属するユーザーに対応する管理者を作成します。そうすることで、スーパー管理者の権限が完全にディレクトリグループに関連付けられるのに対し、スーパー管理者のブートストラップでは、認証のためだけにディレクトリを使用します。

* RBACを適用する前に、一致するディレクトリユーザーのすべての管理者アカウントを作成し、既存のグループが適切なロールにマップされていることを確認します。

