---
title: Konnect概要
subtitle: SaaS APIプラットフォーム
breadcrumb: Overview
---

## {{site.konnect_short_name}}の紹介


{{site.konnect_short_name}}はクラウドネイティブ時代のためにゼロから設計されたAPIライフサイクル
APIライフサイクル管理プラットフォームです。
サービスとして提供されます。このプラットフォームにより、最新のアプリケーションを
より良く、より速く、より安全に構築できます。コントロールプレーンは
データプレーンは、お好みのネットワーク環境でご自身でホストするか、クラウドでKongに管理させるかを選択できます。これらはすべて、{{site.base_gateway}}によって実現されています。- Kongの
軽量、高速、柔軟なAPIゲートウェイです。

<p align="center">
  <img src="/assets/images/products/konnect/dashboard/konnect-dashboard.png" alt="Konnect's dashboard screenshot" />
</p>

{{site.konnect_short_name}}は、マルチクラウドAPI管理の簡素化を支援します：


* クラウド、オンプレミス、Kubernetes、仮想マシンなど、あらゆる環境で API とマイクロサービスをデプロイおよび管理するためのコントロールプレーンを提供します。

* 強力なエンタープライズおよびコミュニティ・プラグインを使用して、認証、APIセキュリティ、およびトラフィック制御ポリシーをすべてのサービスに一貫して瞬時に適用します。

* すべてのサービスをリアルタイムで一元管理。各サービスやルートのエラー率やレイテンシーなどのゴールデンシグナルを監視し、API製品に関する深い洞察を得ることができます。

* ユニバーサルAPIコントロール・プレーンがどの[地理的地域](/konnect/geo)で操作されるかをコントロールします。現在、AU、EU、USのジオがサポートされています。これにより、エンドユーザーと同様の地域で{{site.konnect_short_name}}を運用することができ、データプライバシーと規制コンプライアンスを確保することができます。



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

[{{site.service_catalog_name}}](/konnect/service-catalog)は内部APIを発見可能にします、
内部開発チームのために再利用可能にします。{{site.service_catalog_name}}を使用して、すべてのサービスを監視し、単一のソースを作成することができます。
すべてのサービスを監視し
を作成することができます。{{site.service_catalog_name}}を活用することで、アプリケーション開発者は、{{site.service_catalog_name}}を利用することができます、
アプリケーション開発者
を活用することで、アプリケーション開発者は既存のサービスを検索、発見、利用できるようになります。
また、組織のアプリケーション全体でより一貫したエンドユーザーエクスペリエンスを実現できます。
組織のアプリケーション全体で、より一貫したエンドユーザーエクスペリエンスを実現します。

[カタログサービスの開始&rarr;](/konnect/service-catalog)


### ゲートウェイマネージャー

[ゲートウェイ・マネージャー](/konnect/gateway-manager/) は瞬時に
ホストされた{{site.base_gateway}}コントロール・プレーンを準備します。
[マネージド](/konnect/gateway-manager/dedicated-cloud-gateways/)データプレーンを安全にアタッチします。

Gateway Managerを通じて、OpenID Connect、Open Policy Agent、Mutual TLSなど、すぐに使えるエンタープライズとコミュニティのプラグインでAPIのセキュリティを向上させます。

[ゲートウェイ・マネージャーについてもっと知る &rarr;](/konnect/gateway-manager/)

[{{site.konnect_short_name}}互換プラグインをチェック &rarr;](/hub)

### メッシュマネージャ 

{{site.konnect_product_name}}の[Mesh Manager](https://cloud.konghq.com/mesh-manager)では、{{site.konnect_short_name}}プラットフォームを使って{{site.mesh_product_name}}サービスメッシュを作成、管理、表示することができます。Mesh Managerは、複数のゾーンにわたってメッシュデプロイメントを実行できる1つまたは複数のグローバルコントロールプレーンを持ちたい組織に最適です。Kubernetesゾーンとユニバーサルゾーンを混在して実行できます。メッシュデプロイメント環境には、ワークロードが異なるリージョン、異なるクラウド、または異なるデータセンターで実行される、マルチテナント用の複数の分離されたメッシュを含めることができます。

### APIプロダクト

APIプロダクトを使用すると、APIプロダクトを介して複数のサービスをバンドルして管理できます。API 製品を使用して、サービスのバージョン管理や、API 製品のドキュメントのアップロードを行うことができます。これにより、APIプロダクトをDev Portalに公開することで、サービスを製品化する準備ができます。
[APIプロダクトでサービスの製品化を開始する &rarr;](/konnect/api-products)

### 開発者ポータル

[開発ポータル](/konnect/dev-portal/)は、開発者のオンボーディングを合理化します。
API製品カタログから公開API製品を発見、登録、利用するためのセルフサービス開発者体験を提供します。
このカスタマイズ可能なエクスペリエンスは、独自のブランディングに合わせて使用することができます。
このカスタマイズ可能なエクスペリエンスは、貴社独自のブランディングに合わせて使用することができ、貴社サービスのドキュメントとインタラクティブなAPI仕様を強調表示します。
アプリケーション登録を有効にして、さまざまな認証プロバイダを使用してAPIを自動的に保護します。
 様々な認証プロバイダでAPIを自動的に保護します。

[Dev Portal の詳細はこちら &rarr;](/konnect/dev-portal)

### Analytics

[Analytics](/konnect/analytics/)を使用して、サービスやルート、アプリケーションの使用状況やヘルス・モニタリング・データに対する深い洞察
サービス、ルート、アプリケーションの使用状況や健全性監視データに対する深い洞察を得ることができます。カスタムレポートやコンテキストレポートで
カスタムレポートやコンテキストダッシュボードでAPI製品の健全性を把握できます。
さらに、ネイティブのモニタリングとアナリティクス機能を次のように強化できます。
{{site.base_gateway}}プラグインで、ネイティブのモニタリングと分析機能を強化することができます。
DatadogやPrometheusのようなサードパーティのアナリティクス・プロバイダーへのストリーミング・モニタリング・メトリクスを可能にします。

{{site.konnect_short_name}}アナリティクス。Analyticsは、すべての新しい組織に事前構築されたカスタムレポートを提供します。これらのレポートには、APIの成功を監視しながら追跡するための重要な主要業績評価指標（KPI）の一般的な例が含まれています。ユーザーは自由に変更することができ、独自の分析レポートを開始するためのベースとして使用することができます。

[アナリティクスについてもっと知る &rarr;](/konnect/analytics)

### Teams

あなたの環境を安全に管理するために、{{site.konnect_short_name}} は以下の機能を提供します。
[Teams](/konnect/org-management/teams-and-roles/) を使って権限を管理できます。
標準的なロールのセットには{{site.konnect_short_name}}の定義済みチームを使用できます、
を使用することもできますし、任意のロールでカスタムチームを作成することもできます。ユーザを招待し、これらのチームに追加して、ユーザ
アクセスを管理します。また、既存のIDプロバイダのグループを{{site.konnect_short_name}}チームにマップすることもできます。

[チームとユーザー管理の詳細はこちら &rarr;](/konnect/org-management/teams-and-roles)