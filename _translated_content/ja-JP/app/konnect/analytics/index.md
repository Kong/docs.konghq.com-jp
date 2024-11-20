---
title: Konnect Advanced Analytics
---


## {{site.konnect_short_name}}とは何ですか？アドバンスド・アナリティクスとは？

{{site.konnect_short_name}}。Advanced Analyticsは、APIの健全性、パフォーマンス、使用状況に関する深い洞察を提供する、リアルタイムで高度にコンテキスト化された分析プラットフォームです。企業がAPI戦略を最適化し、運用効率を向上させるために設計されており、{{site.konnect_short_name}}内のプレミアムサービスとして提供されています。

![Summary dashboard](/assets/images/products/konnect/analytics/konnect-api-usage-summary.png){:.image-border}

Konnect Advanced Analyticsは提供します：

* **一元化された可視性**： すべてのAPI、サービス、データプレーンについて、APIランドスケープ全体にわたる包括的で一元化された洞察を提供します。
* **Contextual API analytics**： {{site.konnect_short_name}}。Advanced Analyticsは、特定のルート、関係するコンシューマ、アクセスされたサービスなど、各APIリクエストに関する情報を提供します。
* **民主化されたデータ洞察**： {{site.konnect_short_name}}は、ビジネスチームとプラットフォームチームの両方が、特定の要件に基づいてあらゆるサービス、ルート、または消費者のレポートを生成できるようにします。
* **洞察への最短時間**： アプリケーションチームとプラットフォームチームに、各サービスの重要な API メトリクスを 1 秒未満で提供し、解決までの時間を短縮します。
* **所有コストの削減**： Advanced Analyticsは、構築、保守、サードパーティ製品との統合の必要性を排除したターンキー分析ソリューションです。


## Advanced Analyticsの能力

{{site.konnect_short_name}} アドバンスト・アナリティクス Advanced Analytics {% konnect_icon analytics %}は、APIとデータプレーンノードのパフォーマンスと動作を追跡するのに役立つさまざまなツールを提供します。これらのツールを使用して、主要な統計情報にアクセスし、バイタルを監視し、リアルタイムで異常を突き止めることができます。

The primary goal of {{site.konnect_short_name}} Advanced Analytics is to make important API usage and performance data easily accessible for both business users and platform and application teams. The interface is designed to be user-friendly and intuitive, ensuring that users can quickly find the information they need and derive valuable insights effortlessly.

You can use the following table to help you determine which analytics tools are best for your use case:

| あなたが望むのは... | そうしたいなら... |
| -------------- | ----------- |
| APIの使用状況やパフォーマンスを一目で把握し、組織全体のインサイトを得ることができます。| [Summary dashboard](/konnect/analytics/dashboard/) |
| API使用状況データを閲覧して、主要なパフォーマンスと健全性の統計にアクセスできます。わずか数回のクリックで、保存したAPI使用状況データを可視化、スライス、ダイスします。 | [Explorer](/konnect/analytics/explorer/#api-usage-reporting) |
| チームや部署を超えて洞察を伝え、重要なKPIを共有する。 | [Custom reports](/konnect/analytics/custom-reports/) |
| ユーザーの行動を理解し、問題の発生箇所を特定する。APIへの個々のリクエストに関する詳細な情報を見ることができます。 | [Requests](/konnect/analytics/api-requests/)  |
| 個々の制御プレーンのアナリティクス・コストを管理する| [Data ingestion](/konnect/analytics/#data-ingestion) |
| LLMの使用方法、トークン、コストを理解する | [Explorer](/konnect/analytics/explorer/#llm-usage-reporting) | 




### チャートと制限

{{site.konnect_short_name}}チャートは、要素の上にカーソルを置いて追加の詳細を表示したり、チャートで項目を選択してデータをフィルタリングしたりといったインタラクティブな機能を提供します。 {{site.konnect_short_name}}は、アクティビティグラフまたは[カスタムレポート](/konnect/analytics/use-cases/)に表示されるエンティティの数を50に制限します。 この制限を超えると、影響を受けるグラフやレポートの上部に警告アイコンが表示されます。

### データの取り込み

データの取り込みは、**コントロールプレーン・ダッシュボード**から管理され、アナリティクスのトグルを使用します。


この機能は、データインサイトの管理に柔軟性をもたらし、コスト管理に役立ちます。2つの状態があります： 


* **オン**： アナリティクスを有効にすると、基本データと詳細データの両方が収集されます。これにより、APIトラフィックに関する貴重な洞察を得ることができ、詳細なカスタムレポートを作成することができます。
* **オフ**： アナリティクスを無効にすると、高度なデータの収集が停止します。ただし、基本データは引き続き収集されます。これにより、本質的な洞察力を維持し、{{site.konnect_short_name}}のコンテキスト機能を機能させることができます。


アナリティクスが**off**に設定されている場合、このコントロールプレーンの新しいアナリティクスデータは[カスタムレポート](/konnect/analytics/use-cases/)や[APIリクエスト](/konnect/analytics/api-requests/)には表示されませんが、ゲートウェイマネージャー内の基本的なAPI使用情報にはアクセスできます。

アナリティクスをトグルできるのは、データプレーンが接続されているエンティティのみです。コントロールプレーングループを使用している場合、トグルはグループにのみ効果があり、メンバーCPには効果がありません。

{:.note}
> **移行時間**： 制御プレーンのトグルを変更した場合、設定が有効になるまで数分かかることがあります。この間、データプレーンのアナリティクスに部分的なデータが表示されることがあります。

### チーム・パーミッション

{{site.konnect_short_name}}ユーザーを特定の定義済みAnalyticsチームに割り当てることができます。これにより、特定のユーザに{{site.konnect_short_name}}インスタンスのAnalyticsエリアのみを表示または管理するアクセス権を与えることができます。Analytics AdminチームとAnalytics Viewerチームの詳細については、[Teams Reference](/konnect/org-management/teams-and-roles/teams-reference/) を参照してください。

## 詳細情報

{{site.konnect_short_name}}がテレメトリー・データをどのように扱うか、また切断された場合に何が起こるかについては、[ネットワーク通信FAQ](/konnect/network-resiliency/)を参照してください。
