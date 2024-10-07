---
title: "デフォルトポート"
---
デフォルトでは、{{site.base_gateway}}は以下のポートをリッスンします。

| ポート                                                              | プロトコル | 説明                                                        | ゲートウェイのティア  |
|:-----------------------------------------------------------------|:------|:----------------------------------------------------------|:------------|
| [`:8000`](/gateway/{{page.release}}/reference/configuration/#proxy_listen)  | HTTP  | **コンシューマ** からの HTTP トラフィックを受け取り、アップストリーム **サービス** に転送します。 | すべてのティアとモード |
| [`:8443`](/gateway/{{page.release}}/reference/configuration/#proxy_listen)  | HTTPS | **コンシューマ** からの HTTPトラフィックを受け取り、アップストリーム **サービス** に転送します。  | すべてのティアとモード |
| [`:8001`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri) | HTTP  | Admin API。HTTP 経由でコマンド ラインからの呼び出しをリッスンします。                | すべてのティアとモード |
| [`:8444`](/gateway/{{page.release}}/reference/configuration/#admin_api_uri) | HTTPS | Admin API。HTTPS経由でコマンドラインからの呼び出しをリッスンします。                 | すべてのティアとモード |

{% if_version lte:3.3.x %}
| [`:8002`](/gateway/{{page.release}}/reference/configuration/#admin_gui_listen) | HTTP     | Kong Manager \(GUI\)。HTTPトラフィックをリッスンします。| {{site.base_gateway}}フリーモード |
| [`:8445`](/gateway/{{page.release}}/reference/configuration/#admin_gui_listen) | HTTPS    | Kong Manager \(GUI\).HTTPSトラフィックをリッスンします。| {{site.base_gateway}} フリーモード |
{% endif_version %}

{% if_version gte:3.4.x %}
| [`:8002`](/gateway/{{page.release}}/reference/configuration/#admin_gui_listen) | HTTP     | Kong Manager \(GUI\)。HTTPトラフィックをリッスンします。| すべてのティアとモード |
| [`:8445`](/gateway/{{page.release}}/reference/configuration/#admin_gui_listen) | HTTPS | Kong Manager（GUI）。HTTPSトラフィックをリッスンします。|すべてのティアとモード |
{% endif_version %}

| [`:8005`](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/setup/) | TCP | ハイブリッドモードのみ。コントロールプレーン（CP）はデータプレーン（DP）からのトラフィックをリッスンします。| すべてのティアとモード |
| [`:8006`](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/setup/) | TCP | ハイブリッドモードのみ。 コントロールプレーン（CP）は、データプレーン（DP）からの Vitals テレメトリデータをリッスンします。| すべてのティアとモード |

{% if_version gte:3.6.x %}
| [`:8007`](/gateway/{{page.release}}/reference/configuration/#status_listen) | HTTP | ステータスリスナー。HTTP経由で監視クライアントからの呼び出しをリッスンします。|すべてのティアとモード |
{% endif_version %}

{% if_version lte:3.4.x %}
| [`:8003`](/gateway/{{page.release}}/reference/configuration/#portal_gui_listen) | HTTP     | Dev Portal。Dev Portalが **有効** になっていると仮定して、HTTPトラフィックを監視します。| {{site.ee_product_name}}ティア |
| [`:8446`](/gateway/{{page.release}}/reference/configuration/#portal_gui_listen) | HTTPS     | Dev Portal。Dev Portalが **有効** になっていると仮定して、HTTPSトラフィックを監視します。| {{site.ee_product_name}}ティア |
| [`:8004`](/gateway/{{page.release}}/reference/configuration/#portal_api_listen) | HTTP    | HTTP経由のDev Portal `/files`トラフィック（Dev Portalが **有効** になっている場合）。| {{site.ee_product_name}}ティア |
| [`:8447`](/gateway/{{page.release}}/reference/configuration/#portal_api_listen) | HTTPS    | HTTPS経由のDev Portal `/files`トラフィック（Dev Portal が **有効** になっている場合）。| {{site.ee_product_name}} ティア |
{% endif_version %}

{:.note}
> 
> **注意：** {{site.base_gateway}}無料モードとEnterpriseティアは、オープンソースのゲートウェイ
> パッケージでは利用できません。

