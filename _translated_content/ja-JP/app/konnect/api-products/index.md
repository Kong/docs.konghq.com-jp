---
title: API製品について
content_type: explanation
---

APIプロダクトは複数のサービスを束ねて管理する。各API製品は少なくとも1つのAPI製品バージョンで構成され、各API製品バージョンはGatewayサービスに接続されています。サービスを文書化し、API製品をDev Portalに公開して利用することができます。




## APIプロダクツダッシュボード

APIプロダクツダッシュボードは、APIプロダクツ、バージョン、およびドキュメントを管理するための場所です。このダッシュボードには、APIプロダクツダッシュボード内の任意のAPIプロダクツをクリックすることでアクセスできます。

以下は、APIプロダクツダッシュボードで実行できる主な操作です:

* APIプロダクツを設定する
* APIプロダクツをDev Portalに公開する
* APIプロダクツのバージョンを管理する
* トラフィック、エラー、レイテンシーデータを表示する


![{{site.konnect_short_name}} API Products](/assets/images/products/konnect/api-products/api-products-manage.png)


| 項目 | 説明 |
|-------|------------|
| **概要** | APIプロダクツの分析データ。この分析オプションは、[**Analytics tool**](/konnect/analytics/) を使用して設定できます。 |
| **プロダクトバージョン** | このセクションでは、APIプロダクツバージョンのステータスが表示されます。コンテキストメニューから、APIプロダクツバージョンを**削除**したり、**詳細を見る**ボタンを使用してそのバージョンのダッシュボードに移動したりできます。 |
| **ドキュメント** | APIプロダクツのMarkdownドキュメントを追加および編集したり、API仕様をアップロードしたり、各ドキュメントの公開ステータスを管理したりできます。 |

{:.note}
> **注意**: APIプロダクツは、現在選択している[地理的地域](/konnect/geo)でのみDev Portalに公開できます。別の地域のDev Portalに公開するには、{{site.konnect_product_name}}の左下のコントロールを使用して地域を切り替えてください。

### APIプロダクツバージョン

{{site.konnect_short_name}}のAPIプロダクツバージョンは、[コントロールプレーン](/konnect/gateway-manager/#control-planes)内のゲートウェイサービスにリンクされています。そのため、ゲートウェイサービスに関連付けられた設定やプラグインは、APIプロダクツバージョンにも関連付けられます。

APIプロダクツには複数のAPIプロダクツバージョンを持たせることができ、各バージョンはゲートウェイサービスにリンクできます。APIプロダクツは、異なるコントロールプレーンでAPIプロダクツバージョンをゲートウェイサービスにリンクすることで、複数の環境で利用可能にすることができます。また、APIプロダクツバージョンにAPI仕様を関連付けて、その仕様をDev Portalで利用可能にすることもできます。

一般的なユースケースは、環境の特化です。
例えば、`development`、`staging`、および`production`の3つのコントロールプレーンを持っている場合、APIプロダクツバージョンをそのコントロールプレーン内のゲートウェイサービスにリンクすることで、どの環境でAPIプロダクツバージョンを利用可能にするかを管理できます。例えば、`production`ではv1が稼働しており、`development`ではv2を作業中だとします。テストの準備が整ったら、まず`staging`でv2を作成し、その後、`production`でv1と並行してv2を作成する、といった流れです。


### 分析

APIプロダクツの**概要**に表示される分析ダッシュボードは、APIプロダクツの**トラフィック**、**エラー**、および**レイテンシ**に関するハイレベルの概要を示します。これらのレポートは、APIプロダクツへのトラフィックに基づいて自動的に生成されます。

詳細情報:

* [Analytics overview](/konnect/analytics/)

## ドキュメント

**[APIプロダクツダッシュボード](https://cloud.konghq.com/api-products/)** の**ドキュメント**セクションで、APIプロダクツのドキュメントを直接管理できます。ドキュメントをアップロードした後、{{site.konnect_short_name}}ダッシュボードからシームレスに編集できます。APIプロダクツが公開された後、ドキュメントはアクセス可能になります。

詳細情報:

* [Manage API product documentation](/konnect/api-products/service-documentation/)

### {{site.konnect_short_name}} マークダウンレンダラー

## 詳細情報

* [サービスのデプロイとテスト](/konnect/getting-started/add-api) - Gateway Managerを使用してサービスをデプロイおよびテストする方法を学びます。
* [サービスのプロダクト化](/konnect/getting-started/productize-service/) - APIプロダクツを使用してサービスをプロダクト化する方法を学びます。
