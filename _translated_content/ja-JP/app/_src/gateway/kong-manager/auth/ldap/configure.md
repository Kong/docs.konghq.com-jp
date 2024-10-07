---
title: "Kong Manager の LDAP を有効にする"
badge: "enterprise"
---

{{site.base_gateway}}では、[LDAP Authentication Advancedプラグイン](/hub/kong-inc/ldap-auth-advanced)
を使用して、Kong Managerの
*アドミン* を会社のActive Directoryとバインドできます。

以下の設定を使用すると、LDAPプラグインを手動で適用する必要はありません。この設定だけでKong ManagerのLDAP認証が有効になります。

LDAPを有効にする
----------

Kongで、`kong.conf`構成ファイル内に、または環境変数を使用して次のプロパティが構成されるようにします。

    admin_gui_auth = ldap-auth-advanced
    enforce_rbac = on
    admin_gui_session_conf = { "secret":"set-your-string-here" }
    admin_gui_auth_conf = {                                       \
        "anonymous":"",                                           \
        "attribute":"<enter_your_attribute_here id="sl-md0000000">",                \
        "bind_dn":"<enter_your_bind_dn_here id="sl-md0000000">",                    \
        "base_dn":"<enter_your_base_dn_here id="sl-md0000000">",                    \
        "cache_ttl": 2,                                           \
        "consumer_by":["username", "custom_id"],                  \
        "header_type":"Basic",                                    \
        "keepalive":60000,                                        \
        "ldap_host":"<enter_your_ldap_host_here id="sl-md0000000">",                \
        "ldap_password":"<enter_your_ldap_password_here id="sl-md0000000">",        \
        "ldap_port":389,                                          \
        "start_tls":false,                                        \
        "ldaps":false,                                            \
        "timeout":10000,                                          \
        "verify_ldap_host":true                                   \
    }

|       属性        |                                                                                                       説明                                                                                                        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `attribute`     | LDAPユーザーを識別するために使用される属性。<br>たとえば、LDAPユーザーをユーザー名で管理者にマッピングするには、 `uid`を設定します。                                                                                                                                     |
| `bind_dn`       | LDAPバインドDN（識別名）。ユーザーのLDAP検索を実行するために使用されます。この`bind_dn`には、認証中のユーザーを検索する権限が必要です。<br>たとえば、`uid=einstein,ou=scientists,dc=ldap,dc=com`。                                                                              |
| `base_dn`       | LDAPベースDN（識別名）。<br>`ou=scientists,dc=ldap,dc=com`はその一例です。                                                                                                                                                       |
| `ldap_host`     | LDAPホストドメイン。 <br>たとえば、 `ec2-XX-XXX-XX-XXX.compute-1.amazonaws.com` 。                                                                                                                                            |
| `ldap_port`     | デフォルトの LDAP ポートは 389 です。636 は SSL LDAP および AD に必要なポートです。`ldaps` が設定されている場合は、ポート 636 を使用する必要があります。より複雑な Active Directory（AD）環境では、ドメインコントローラーとポート 389 の代わりに、グローバルカタログのホストとポートの使用を検討してください。デフォルトではポート 3268 を使用します。 |
| `ldap_password` | LDAPパスワード。 <br>他の構成プロパティと同様に、機密情報は構成ファイルに直接書き込むのではなく、環境変数として設定できます。                                                                                                                                             |
| `ldaps`         | `true` に設定して、LDAP サーバーに安全に接続するためのプロトコル（TLS に構成可能）である `ldaps` を使用します。`ldaps` 設定が有効になっている場合は、`start_tls` 設定が無効になっていることを確認してください。                                                                                  |

**セッションプラグイン** （`admin_gui_session_conf`で構成）にはシークレットが必要であり、デフォルトで安全に構成されています。

* どのような状況でも、`secret`は手動で文字列に設定する必要があります。
* HTTPS ではなく HTTP を使用する場合は、`cookie_secure` を手動で `false` に設定する必要があります。
{% if_version lte:3.1.x -%}
* Admin API と Kong Manager に異なるドメインを使用する場合は、 `cookie_samesite`を`off`に設定する必要があります。
{% endif_version -%}
{% if_version gte:3.2.x -%}
* Admin APIとKong Managerに異なるドメインを使用する場合は、`cookie_same_site`を`Lax`に設定する必要があります。 {% endif_version %}

これらのプロパティの詳細については、[Kong ManagerのSessionセキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照し、[構成例](/gateway/{{page.release}}/kong-manager/auth/sessions/#example-configurations)を確認してください。

Kongを希望の構成で起動した後、ユーザー名がADのものと一致する新しい *アドミン* を作成できます。
それらのユーザーは、Kong Manager
に参加するための招待状を受け入れ、LDAP認証情報でログインできるようになります。

### CLI での Service Directory マッピングの使用

{% include_cached /md/gateway/ldap-service-directory-mapping.md %}

