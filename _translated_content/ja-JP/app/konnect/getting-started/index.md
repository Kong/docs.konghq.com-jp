---
title: Konnectをはじめましょう
---

<table style="border:1px solid #e0e4ea;border-radius:6px">
  <tr style="background-color:#fff;border:none">
    <td rowspan="3" style="border-right:1px solid #e0e4ea;vertical-align:top;border-bottom:none;background-color:#F9FAFB">
        <br>
        <p style="font-size:16px;">{{site.konnect_product_name}}<img style="min-height:18px" src="/assets/images/logos/kong-konnect-logo.svg" alt="を始めるには" class="no-image-expand" /> </p>
    </td>
  </tr>
    <tr style="background-color:#fff;border:none">
    <td style="border-bottom:1px solid #e0e4ea;">
        <br>
        <p><b>APIのセットアップを手伝ってください</b></p>
        <p>{{site.konnect_product_name}}にサインアップすると、オンボーディングウィザードですぐに始めることができます。</p>
        <p><a href="https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs">Sign up today</a></p>
    </td>
</tr>
  <tr style="background-color:#fff;border:none">
    <td style="border-bottom:none">
        <p><b>始め方を知っている</b></p>
        <p><a href="/konnect/getting-started/add-api/"><i class="fas fa-plus"></i> APIの追加 &rarr;</a></p>
        <p><a href="/konnect/getting-started/migration/"><i class="fas fa-file-import"></i> {{site.base_gateway}} エンティティのインポート &rarr;</a></p>
    </td>
  </tr>
</table>

### {{site.konnect_short_name}}について

{{site.konnect_short_name}}は、APIライフサイクル管理プロセスを合理化し、スピード、セキュリティ、効率性を向上させた最新のアプリケーションを開発できるようにします。Kongによって管理され、クラウドでホストされるコントロールプレーンと、自己管理またはKongを通じて好みのネットワーク環境でデータプレーンを管理できる汎用性を独自に組み合わせています。このプラットフォームの中核となるのが{{site.base_gateway}}で、Kongの革新的かつ高性能で適応性の高いAPIゲートウェイです。
{{site.konnect_short_name}}は以下のようなユースケースに役立ちます：

* **デベロッパー:** {{site.konnect_short_name}}を{{site.base_gateway}}を管理する最も簡単な方法として使用してください。これはAPIを保護しセキュアにするための定型的なコードを書く時間を節約するのに役立ちます。
* **小規模チーム:** APIカタログ、外部開発者ポータル、API分析などの機能を使用して、APIを共有し収益化する。
* **大企業:** 地理的に分散された複数のコントロールプレーンを使用し、APIチームにスピードと柔軟性を提供しながら、中央チームにガバナンスを与えることで、連携したAPI管理を実現する。

### {{site.konnect_short_name}} アーキテクチャ

{% include_cached /md/konnect/konnect-architecture.md %}

スタートガイドでは、{{site.base_gateway}}データプレーンノードを使用して、サービスやルートなどのエンティティを作成する方法を紹介します。

![{{site.konnect_product_name}}](/assets/images/products/konnect/konnect-intro.png)

> Figure 1: {{site.konnect_short_name}}モジュールの構成図。Kongがホストする{{site.konnect_short_name}}環境は、{{site.konnect_short_name}}アプリケーション、{{site.konnect_short_name}}プラットフォーム、およびコントロールプレーンで構成されます。{{site.konnect_short_name}}プラットフォームと接続されている{{site.base_gateway}}、{{site.mesh_product_name}}、{{site.kic_product_name}}データプレーンノードは、お客様またはお客様のクラウドプロバイダ、または{{site.konnect_short_name}}によって管理されます。

### Dedicated Cloud Gateway

{{site.konnect_short_name}}は[Dedicated Cloud Gateway](/konnect/gateway-manager/dedicated-cloud-gateways/)を備えており、パフォーマンス、拡張性、可用性の高いグローバルAPI管理インフラを実現します。
これらのゲートウェイは提供する：
* ワンクリックでプロビジョニングできるマルチリージョン対応
* アップグレードのためのダウンタイムがない
* さまざまなAPIトラフィックのワークロードを処理するスケーラビリティ
* 安全な接続のためのプライベート・ネットワーク

Dedicated Cloud Gatewayを[オートパイロットモード](/konnect/gateway-manager/dedicated-cloud-gateways/#autopilot-mode)で実行すると、トラフィックと量に応じて自動的に調整し、APIゲートウェイリソースの管理を合理化できます。
このモードでは、インフラストラクチャがトラフィックの到着に応じて処理できるように準備され、手動で介入することなくリソースをスケーリングして一貫したパフォーマンスを維持します。

Dedicated Cloud Gatewayは[プライベート・ネットワーキング](/konnect/gateway-manager/dedicated-cloud-gateways/transit-gateways)もサポートします。 
[AWS Transit Gateway](https://aws.amazon.com/transit-gateway/)との統合により、組織のセキュリティ要件をサポートする方法でネットワークを接続することができ、{{site.konnect_short_name}}を利用したAPIインフラストラクチャが内部ネットワークやクラウドリソースと安全にやり取りできるようになります。


### {{site.konnect_short_name}}の{{site.base_gateway}}エンティティ。

各{{site.base_gateway}}データプレーンノードには以下のエンティティが含まれます：

* [**サービス:**](/gateway/latest/key-concepts/services/) サービスとは、外部の上流APIやマイクロサービスを表すエンティティです。例えば、データ変換マイクロサービス、課金APIなど。
* [**ルート:**](/gateway/latest/key-concepts/routes/) ルートは、リクエストがゲートウェイに到達した後、そのサービスにどのように送られるかを決定します。サービスはバックエンドAPIを表し、ルートはクライアントに公開されるものを定義します。1つのサービスは多くのルートを持つことができます。一つのサービスは多くのルートを持つことができる。ルートがマッチすると、ゲートウェイはリクエスト を関連するサービスにプロキシします。
* [**コンシューマ:**](/gateway/latest/kong-enterprise/consumer-groups/) コンシューマ オブジェクトは、サービスのユーザーを表すもので、認証に使われることが多いものです。サービスへのアクセスを分割する方法を提供し、サービスの機能を妨げることなく、そのアクセスを簡単に取り消すことができます。
* [**ロードバランサ:**](/gateway/latest/get-started/load-balancing/) ロードバランシングとは、APIリクエストのトラフィックを複数のアップストリーム サービスに分散させる方法です。ロードバランシングは、システム全体の応答性を向上させ、個々のリソースの過負荷を防ぐことで障害を減らします。
* [**アップストリーム ターゲット:**](/gateway/latest/key-concepts/upstreams/) アップストリームとは、{{site.base_gateway}}がリクエストを{{site.base_gateway}}に転送するAPI、アプリケーション、またはマイクロサービスを指します。{{site.base_gateway}},では、アップストリームオブジェクトは仮想ホスト名を表し、複数のサービス上で受信リクエストのヘルスチェック、サーキットブレーク、ロードバランスに使用できます。

{{site.konnect_short_name}}のUIまたはAPIを使用して、サービスのようなこれらのエンティティを作成すると、Kongは自動的に対応するデータプレーンノードにエンティティを作成します。UIまたはAPIを使用すると、Kongは対応するデータプレーン ノードにエンティティを自動的に作成します。

詳しくは[{{site.konnect_short_name}}の{{site.base_gateway}}設定](/konnect/gateway-manager/configuration/)を参照してください。