---
title: Konnect概要
subtitle: SaaS APIプラットフォーム
breadcrumb: Overview
---

## {{site.konnect_short_name}}の紹介


{{site.konnect_short_name}}はクラウドネイティブ時代のためにゼロから設計されたAPIライフサイクル管理プラットフォームであり、SaaSとして提供されます。このプラットフォームにより、最新のアプリケーションをより良く、より速く、より安全に構築できます。コントロールプレーンはKongによってクラウドで運営されます。データプレーンはお好みのネットワーク環境でご自身でホストするか、クラウドでKongに管理させるかを選択できます。これらはすべて、{{site.base_gateway}}によって実現されています。- Kongの
軽量、高速、柔軟なAPIゲートウェイです。

<p align="center">
  <img src="/assets/images/products/konnect/dashboard/konnect-dashboard.png" alt="Konnect's dashboard screenshot" />
</p>

{{site.konnect_short_name}}は、マルチクラウドAPI管理の簡素化を支援します：


* クラウド、オンプレミス、Kubernetes、仮想マシンなど、あらゆる環境で API とマイクロサービスをデプロイおよび管理するためのコントロールプレーンを提供します。

* 強力なエンタープライズおよびコミュニティ プラグインを使用して、認証、APIセキュリティ、およびトラフィック制御ポリシーをすべてのサービスに一貫して即座に適用できます。

* すべてのサービスをリアルタイムで一元管理。各サービスやルートのエラー率やレイテンシーなどのゴールデンシグナルを監視し、API製品に関する深い洞察を得ることができます。

* ユニバーサルAPIコントロールプレーンがどの[地理的地域](/konnect/geo)で操作されるかをコントロールします。現在、AU、EU、USの地域がサポートされています。これにより、エンドユーザーと同様の地域で{{site.konnect_short_name}}を運用することができ、データプライバシーと規制コンプライアンスを確保することができます。



<div class="docs-grid-install">

  <!-- TO DO: ADD KONNECT FEATURES TABLE
   <a href="#features" class="docs-grid-install-block no-description">
    <img class="install-icon no-image-expand" src="/assets/images/icons/documentation/icn-flag.svg" alt="">
    <div class="install-text">Features</div>
  </a> -->

  <a href="https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=konnect-getting-started" class="docs-grid-install-block no-description">
    <img class="install-icon no-image-expand" src="/assets/images/icons/documentation/icn-flag.svg" alt="">
    <div class="install-text">{{site.konnect_short_name}}にサインアップしてください。</div>
  </a>

  <a href="/konnect/getting-started/add-api" class="docs-grid-install-block no-description">
    <img class="install-icon no-image-expand" src="/assets/images/icons/documentation/icn-admin-api-color.svg" alt="">
    <div class="install-text">APIを{{site.konnect_short_name}}に追加する</div>
  </a>

  <a href="/konnect/getting-started/import" class="docs-grid-install-block no-description">
    <img class="install-icon no-image-expand" src="/assets/images/icons/documentation/icn-deployment-color.svg" alt="">
    <div class="install-text">{{site.base_gateway}}のエンティティを{{site.konnect_short_name}}にインポートする</div>
  </a>

  <a href="/hub/" class="docs-grid-install-block no-description">
    <img class="install-icon no-image-expand" src="/assets/images/icons/documentation/icn-api-plugins-color.svg" alt="">
    <div class="install-text">プラグイン</div>
  </a>
</div>
## {{site.konnect_short_name}} modules

### {{site.service_catalog_name}}

[{{site.service_catalog_name}}](/konnect/service-catalog)は、社内開発チームがAPIを見つけ、利用するプラクティスを可能にします。{{site.service_catalog_name}}を使用すると、すべてのサービスを観測しつつ、社内サービスの集約カタログを作成することができます。{{site.service_catalog_name}}を活用することによって、アプリケーション開発者は既存のサービスを探し、発見し、利用出来る - 市場への導入速度 (TTM) を加速し、同時にエンドユーザーにとって均一な品質のサービスとエクスペリエンスを提供する事ができます。

[カタログサービスの開始&rarr;](/konnect/service-catalog)


### Gateway Manager

[Gateway Manager](/konnect/gateway-manager/) は即時に{{site.base_gateway}}コントロールプレーンを提供し、安全に自己運用もしくは[マネージド](/konnect/gateway-manager/dedicated-cloud-gateways/)データプレーンを接続することが出来ます。

Gateway Managerを通じて、OpenID Connect、Open Policy Agent、Mutual TLSなど、すぐに使えるエンタープライズとコミュニティのプラグインでAPIのセキュリティを向上させることができます。

[Gateway Managerについてもっと知る &rarr;](/konnect/gateway-manager/)

[{{site.konnect_short_name}}互換プラグインをチェック &rarr;](/hub)

### Mesh Manager 

{{site.konnect_product_name}}の[Mesh Manager](https://cloud.konghq.com/mesh-manager)では、{{site.konnect_short_name}}プラットフォームを使って{{site.mesh_product_name}}サービスメッシュを作成、管理、観測することができます。Mesh Managerは、複数のゾーンにわたってメッシュのデプロイを実行できるグローバルコントロールプレーンを提供します。KubernetesゾーンとVM等その他（ユニバーサル）ゾーンを混在して実行できます。メッシュデプロイメント環境には、ワークロードが異なるリージョン、異なるクラウド、または異なるデータセンターで実行される、マルチテナント用の複数の分離されたメッシュを含めることができます。

### API　Product

API Productを使用すると、API Productを介して複数のサービスをバンドルして管理できます。API Productを使用して、サービスのバージョン管理やドキュメントの公開を行うことができます。これにより、API ProductをDev Portalに公開することで、サービスを製品化する準備ができます。
[API Productでサービスの製品化を開始する &rarr;](/konnect/api-products)

### Dev Portal

[Dev Portal](/konnect/dev-portal/)は、開発者のAPI Productへのアクセスを合理化します。API Productのカタログから公開API製品を発見、登録、利用するためのセルフサービス開発者体験を提供します。この体験はカスタマイズ可能であり、独自のブランディングに合わせて使用したり、貴社独自のブランディングに合わせて使用することができます。貴社サービスのドキュメントとAPI仕様を親しみやすく、かつ有益なものとする事ができます。利用するAPIはアプリケーションとしてグループ化でき、さまざまな認証方法によってAPIを自動的に保護します。

[Dev Portalの詳細はこちら &rarr;](/konnect/dev-portal)

### Analytics

[Analytics](/konnect/analytics/)を使用して、サービスやルート、アプリケーションの使用状況や健全性監視データに対する深い洞察を得ることができます。カスタムレポートやコンテキストダッシュボードでAPI製品の健全性を把握できます。さらにKong Gatewayでは、DatadogやPrometheusといった外部観測基盤に、プラグインを利用してメトリクスを送ることができます。

{{site.konnect_short_name}} Analyticsはすぐに利用できるカスタムレポートの機能を提供します。これらのレポートには、APIの成功を監視しながら追跡するための重要なKPIの一般的な例が含まれています。ユーザーは自由に変更することができ、独自の分析レポートを開始するためのベースとして使用することができます。

[Analyticsについてもっと知る &rarr;](/konnect/analytics)

### Teams

あなたの環境を安全に管理するために、{{site.konnect_short_name}} は[Teams](/konnect/org-management/teams-and-roles/) と呼ばれる権限管理の仕組みを提供しています。標準的なロールのセットには{{site.konnect_short_name}}の定義済みチームを使用できますし、任意のロールでカスタムチームを作成することもできます。ユーザを招待し、これらのチームに追加して、ユーザ
アクセスを管理します。また、既存のIDプロバイダのグループを{{site.konnect_short_name}}チームにマッピングすることもできます。

[Teamsとユーザー管理の詳細はこちら &rarr;](/konnect/org-management/teams-and-roles)