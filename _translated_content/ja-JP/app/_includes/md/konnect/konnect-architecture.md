<!-- Used in Konnect Architecture and Konnect Getting Started Overview-->
{{site.konnect_product_name}}プラットフォームは、すべてのサービス構成を管理するために、いくつかのホストされたコントロールプレーンオプションを提供します。以下のコントロールプレーンオプションを1つ以上使用できます：
* {{site.base_gateway}}
* {{site.kic_product_name}} 
* {{site.mesh_product_name}}

制御プレーンは、データプレーンノード（および{{site.mesh_product_name}}の場合はプロキシ）で構成されるデータプレーングループに、これらの設定を伝達する。個々のノードは、オンプレミス、クラウドホスト環境、または{{site.konnect_product_name}}の[専用クラウドゲートウェイ](/konnect/gateway-manager/dedicated-cloud-gateways/)による完全管理のいずれかで実行できます。各データプレーンノードはメモリ内にコンフィギュレーションを保持するため、展開モデル全体にわたって効率的で信頼性の高いサービス管理が保証されます。