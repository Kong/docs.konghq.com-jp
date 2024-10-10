---
nav_title: "概要"
---
サービスまたはルートへのアクセスを制限するには、任意のACLグループを使用して許可または拒否のリストにコンシューマを追加します。サービスまたはルートがすでに有効になっている場合、このプラグインには[認証プラグイン](/hub/#authentication)（[Basic Authentication](/hub/kong-inc/basic-auth/)、[Key Authentication](/hub/kong-inc/key-auth/)、[OAuth 2\.0](/hub/kong-inc/oauth2/)、[OpenID Connect](/hub/kong-inc/openid-connect/)など）が必要です。

{% if_version gte:3.6.x %}
構成オプション[`include_consumer_groups`](/hub/kong-inc/acl/configuration/#include_consumer_groups)を`true`に設定すると、コンシューマグループの使用を有効にすることもできます。このオプションを使用すると、`allow`と`deny`フィールドを評価する場合に、{{site.base_gateway}}がACLグループとコンシューマグループの両方を考慮するようになります。{% endif_version %}

`allow`と`deny`の両方の構成でACLを構成することはできません。`allow`を持つACLは、構成済みのグループがリソースにアクセスでき、その他すべてが本質的に拒否されるポジティブセキュリティモデルを提供します。対照的に、 `deny`構成は、特定のグループがリソースへのアクセスを明示的に拒否され（他のすべてのグループは許可される）、ネガティブセキュリティモデルを提供します。

ACLプラグインの使用開始
-------------

* [構成リファレンス](/hub/kong-inc/acl/configuration/)
* [基本的な構成例](/hub/kong-inc/acl/how-to/basic-example/)
* [プラグインの使用方法](/hub/kong-inc/acl/how-to/)
* [ACMEプラグインAPI参照](/hub/kong-inc/acl/api/) {% if_version gte:3.6.x %}
* [コンシューマグループでのACLの使用](/hub/kong-inc/acl/how-to/consumer-groups/) {% endif_version %}

