---
title: "エンタープライズライセンスのデプロイ"
badge: "enterprise"
---
[エンタープライズ固有の機能](/gateway/{{page.release}}/licenses/)にアクセスするには、エンタープライズライセンスを {{site.base_gateway}} インストールにデプロイしてください。

{% include_cached /md/enterprise/deploy-license.md heading="##" release=page.release %}

トラブルシューティングとサポート
----------------

ライセンスの問題のトラブルシューティングについては、以下を参照してください。

* [ライセンスのトラブルシューティング](/gateway/{{page.release}}/licenses/#troubleshooting)

詳細と例については、以下を参照してください。

* [`/licenses` API リファレンス](/gateway/api/admin-ee/latest/#/licenses)
* [`/licenses` APIの例](/gateway/{{page.release}}/licenses/examples/)

`HTTP/1.1 200 OK` メッセージを受信しなかった場合、またはセットアップの完了にサポートが必要な場合は、Kong サポートに問い合わせるか、[サポートポータル](https://support.konghq.com/support/s/)にアクセスしてください。

