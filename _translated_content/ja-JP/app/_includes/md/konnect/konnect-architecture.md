<!-- Used in Konnect Architecture and Konnect Getting Started Overview-->
{{site.konnect_product_name}}プラットフォームは、あらゆるサービス構成を管理するために、いくつかのコントロールプレーンオプションを提供します。以下のコントロールプレーンオプションを1つもしくは複数使用できます：
* {{site.base_gateway}}
* {{site.kic_product_name}} 
* {{site.mesh_product_name}}

コントロールプレーンは、データプレーンノード（および{{site.mesh_product_name}}の場合はプロキシ）で構成されるデータプレーングループに、これらの設定を連携します。個々のノードは、オンプレミス、クラウドホスト環境、または{{site.konnect_product_name}}の[Dedicated Cloud Gateways](/konnect/gateway-manager/dedicated-cloud-gateways/)によるフルマネージドのいずれかを選択できます。各データプレーンノードはメモリ内にコンフィギュレーションを保持するため、トポロジー全体にわたって効率的で信頼性の高いサービス管理が保証されます。