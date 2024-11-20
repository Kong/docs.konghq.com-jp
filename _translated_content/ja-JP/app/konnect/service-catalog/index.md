---
title: Konnect サービスカタログ
subtitle: アーキテクチャ全体のあらゆるサービスを追跡
content-type: explanation
beta: true
---

{{site.konnect_saas}}の{{site.service_catalog_name}}は、組織で実行されているすべてのサービスの包括的なカタログを提供します。Gateway ManagerやMesh ManagerのようなKonnect内部アプリケーションと、GitHubやPagerDutyのような外部アプリケーションの両方と統合することで、{{site.service_catalog_name}}は組織の各サービスを360度見渡すことができます。サービスの所有者、上流と下流の依存関係、コード・リポジトリ、CI/CDパイプライン、APIゲートウェイによってフロントされているか、サービス・メッシュの一部であるかなどの情報が表示される。

<!-- vale off-->
{% mermaid %}
graph LR
  %% Define styles for icons and nodes
  classDef konnect stroke-dasharray:3,rx:10,ry:10
  classDef serviceCatalog fill:#d0e5f3,rx:10,ry:10
  %% Define the resources
  ExternalIntegration1(External Integration 1):::resource
  ExternalIntegration2(External Integration 2</div>):::resource
  GatewayManager(<div style="text-align:center;"><img src="/assets/images/logos/konglogo-gradient-secondary.svg" style="max-width:25px; display:block; margin:0 auto;" class="no-image-expand"/> Gateway Manager</div>):::resource

  %% Define the Konnect Service Catalog and its subgraphs
  subgraph KonnectServiceCatalog[Konnect Service Catalog]
    direction TB

    %% Define the Billing Service subgraph inside Konnect Service Catalog
    subgraph BillingService[Billing Service]
      direction LR
      KongGatewayService(Gateway service 1)
      ExternalResourceMapping1(External resource)
      ExternalResourceMapping2(External resource)
    end

    %% Define the Inventory Service Catalog Service subgraph inside Konnect Service Catalog
    subgraph InventoryService[Inventory Service]
      direction LR
      SomeOtherService(Gateway service 2)
      AnotherService(External resource)
      DifferentRepo(External resource)
    end

  end

  %% Connect the resources to the Konnect Service Catalog
  ExternalIntegration1 -- "External resource" --> KonnectServiceCatalog
  ExternalIntegration2 -- "External resource" --> KonnectServiceCatalog
  GatewayManager -- "Gateway services" --> KonnectServiceCatalog

  %% Style the subgraphs
  class KonnectServiceCatalog konnect
  class BillingService serviceCatalog
  class InventoryService serviceCatalog
{% endmermaid %}
<!-- vale on-->

> *Figure 1: This diagram shows how you can use both external integrations, like GitHub and PagerDuty, as well as internal integrations like Gateway Manager to pull resources into Service Catalog. You can then map those resources (like GitHub repositories, PagerDuty services, and Gateway services) to Service Catalog services.*

## {{site.service_catalog_name}} use cases

| あなたが望むのは... | そうしたいなら... |
| -------------- | ----------- |
| チームを{{site.service_catalog_name}}サービスにマッピングすることで、組織のリソース所有権を追跡できます。 | {{site.konnect_short_name}}に{{site.service_catalog_name}}サービスを新規作成](https://cloud.konghq.com/service-catalog/create-service)する際に、{{site.service_catalog_name}}サービスの所有者を追加します。|
| 組織内の未認識または未発見のAPIを含む、すべてのサービスを可視化します。 | [{{site.service_catalog_name}} 統合](https://cloud.konghq.com/service-catalog/integrations) |
| 主要な{{site.service_catalog_name}}サービスの健全性メトリクス、ドキュメント、およびAPI仕様を1つのリストに統合し、1つの場所から他のツールとやり取りできるようにします。 | [{{site.service_catalog_name}}ダッシュボード](https://cloud.konghq.com/service-catalog/) |

<!-- commenting this out until it's released:
| Govern how services are created and maintained across your company to adhere to security, compliance, and engineering best practices. | Scorecards |-->

## {{site.service_catalog_name}} 用語

| 用語 | 定義 |
| ---- | ---------- |
| インテグレーション | これらは{{site.konnect_short_name}}内部または外部のアプリケーションで、リソースを取り込むソースとして機能します。例えば、GitHub。|
| リソース | 有効化された統合から{{site.service_catalog_name}}が取り込むエンティティを示す包括的な用語。リソースには、インフラストラクチャー・コンポーネント（ゲートウェイ・サービス、メッシュ・サービス、データベース、キャッシュなど）から、外部アプリケーションやツール（コード・リポジトリ、CI/CDインフラストラクチャー、オンコール・システムなど）、ドキュメントの一部（API仕様など）まで、さまざまなものがある。リソースは1つ以上の{{site.service_catalog_name}}サービスにマップすることができます。|
| {{site.service_catalog_name}} service |  通常1つのチームが所有し、1つ以上のAPIを公開し、他の{{site.service_catalog_name}}サービスに（上流または下流として）依存する可能性があるソフトウェアの単位。サービスは、1つ以上のリソースのコレクションと考えることができます。|
| {{site.service_catalog_name}} |  組織で実行されているすべてのリソースと{{site.service_catalog_name}}サービスの総合カタログ。 |
| イベント | [イベント](/konnect/service-catalog/integrations/#events)は{{site.service_catalog_name}}によって記録される情報の捕捉単位です。ユーザー主導のアクション、設定変更、アラートなど、幅広いアクティビティが含まれる。|


## FAQs

<details><summary>サービスカタログのサービスはAPI製品とどのように対応するのか？そこでの関係は？</summary>

{% capture service_mapping %}
Service CatalogサービスはAPI製品に直接マッピングされません。むしろ、GatewayサービスをService Catalogサービスにリンクして、GatewayサービスをService CatalogのAPI製品バージョンにマッピングすることができます。
{% endcapture %}

{{ service_mapping | markdownify }}

</details>

<details><summary>サービスカタログで自分のサービスのヘルスとステータスを表示するにはどうすればよいですか？</summary>

{% capture service_health %}
サービスカタログの**サービス**に移動し、ヘルスとステータスを表示したいサービスカタログサービスをクリックします。
{% endcapture %}

{{ service_health | markdownify }}

</details>

<details><summary>Service Catalogサービスにマッピング可能なリソースを特定するにはどうすればよいですか？</summary>

{% capture reuse_resources %}
サービスカタログの**リソース**ページに移動して、利用可能なすべてのリソースのリストを表示します。これらのリソースは、**Integrations**ページにある、インストールされ認証された統合からService Catalogに取り込まれます。
{% endcapture %}

{{ reuse_resources | markdownify }}

</details>

<details><summary>サービスカタログに表示されるデータに不一致がある場合はどうすればよいですか？</summary>

{% capture discrepancies %}
Service Catalogの統合設定とデータソースに問題がないか確認する。接続されているすべてのツールが適切に設定され、データの同期が正しく機能していることを確認します。
{% endcapture %}

{{ discrepancies | markdownify }}

</details>

<details><summary>特定のService Catalogサービスやデータへのアクセス権を管理できますか？</summary>

{% capture service_access %}
はい、[teams](/konnect/org-management/teams-and-roles/manage/)と[roles](/konnect/org-management/teams-and-roles/roles-reference/)を設定することで、アクセス制御を設定し、サービスカタログの権限を管理できます。
{% endcapture %}

{{ service_access | markdownify }}

</details>

## More information
* [Service Catalog integrations](/konnect/service-catalog/integrations)
