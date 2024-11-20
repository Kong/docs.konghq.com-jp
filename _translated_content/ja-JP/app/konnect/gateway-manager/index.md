---
title: Gateway Managerについて
---

[ゲートウェイマネージャ](https://cloud.konghq.com/gateway-manager)は{{site.konnect_saas}}機能モジュールです。およびデータプレーンノード(DP)のカタログ化、接続、ステータスの監視、およびコントロールプレーンのコンフィギュレーションを一元管理することができます。

Gateway Managerの概要ページには、組織が現在所有しているコントロールプレーンのリストが表示されます。ここから、コントロールプレーンを追加または削除したり、個々のコントロールプレーンに移動してデータプレーンノードとそのグローバル構成を管理したりできます。

![gateway manager dashboard](/assets/images/products/konnect/gateway-manager/konnect-control-plane-dashboard.png)


{{site.konnect_short_name}}がコントロールプレーンをホスティングしているため、データプレーンノードは設定データを保存するためのデータベースを必要としません。代わりに、設定は各ノード上にインメモリで保存され、数回のクリックでコントロールプレーン内のすべてのデータプレーンノードを簡単に更新できます。このモデルは、ノードがオンプレミスで管理されていても、クラウドホスト環境で管理されていても、[専用クラウドゲートウェイ](/konnect/gateway-manager/dedicated-cloud-gateways)または[サーバーレスゲートウェイ](/konnect/gateway-manager/serverless-gateways)のフルマネージドSaaSサービスを通じて管理されていても適用されます。


ゲートウェイマネージャと{{site.konnect_saas}}アプリケーション全体は、データプレーンノードを流れるデータにアクセスしたり可視化したりすることはなく、各ノードの状態と接続詳細以外のデータは保存しません。


## コントロールプレーン

{{site.konnect_short_name}}は、コントロールプレーンを介してデータプレーンの設定を管理する。

コントロールプレーンには3つのタイプがある：


* [**{{site.base_gateway}} コントロールプレーン**](#kong-gateway-control-planes): 
    同じコンフィグレーションと動作空間を共有する{{site.base_gateway}}データプレーンノードの集まり。各制御プレーンは独立してコンフィグレーションを管理します。

* [**Control plane group**](#control-plane-groups): 
    複数のコントロールプレーンの中央データプレーンノードを管理するコントロールプレーンの一種。メンバーであるコントロールプレーンからコンフィグレーションを収集し、集約されたコンフィグレーションをノードグループに適用します。
    
    これは、グループ内のチームが{{site.base_gateway}}データプレーンノードのクラスタを共有し、各チームがそれぞれ分離された設定を持つことを意味する。

* [**{{site.kic_product_name}}**](/konnect/gateway-manager/kic/)
    Kubernetesベースの{{site.base_gateway}}データプレーンノードの設定を監視する。


組織内のすべてのコントロールプレーンのリストは、[Gateway Manager overview](https://cloud.konghq.com/gateway-manager/)で確認できます。

各コントロール・プレーンへのアクセスは、エンティティ固有のパーミッションを使ってチームごとに設定できます。詳細については、[チームの管理](/konnect/org-management/teams-and-roles/) を参照してください。

### {{site.base_gateway}} コントロールプレーン

どの組織のどのリージョンも、デフォルトのコントロールプレーン1つで始まります。
site.konnect_short_name}}を使用すると、追加の{{site.base_gateway}}コントロールプレーンを設定することができます。
1つの{{site.konnect_short_name}}組織で複数のコントロールプレーンを使用して、データプレーンノードとその設定を任意のグループ分けで管理します。


複数のコントロールプレーンを使用する一般的なユースケースには、以下のようなものがある：


* **環境分離:** 開発環境、ステージング環境、本番環境など、目的に応じて環境を分割する。
* **リージョン分離:** 各制御プレーンをリージョンまたはリージョンのグループに割り当てる。各制御プレーンに対して、リージョン内のデータプレーンノードをスピンアップする。
* **チーム分離:** 各制御プレーンを異なるチームに割り当て、チームの目的に応じてリソースを共有する。
* **リソースのホスティングと管理の分離**： リソースホスティングと管理の分離**：セルフホスティングのデータプレーンノードを持つハイブリッドコントロールプレーンと、Kong管理ノードを持つ独立したDedicated Cloud Gatewayコントロールプレーンを実行します。
<!--vale off-->
{% mermaid %}
flowchart TD
A(Hybrid control plane)
B(Fully-managed \n control plane)
C(<img src="/assets/images/logos/KogoBlue.svg" style="max-height:20px" class="no-image-expand"/> Self-managed \n data plane nodes \n #40;locally-hosted#41;)
D(<img src="/assets/images/logos/KogoBlue.svg" style="max-height:20px" class="no-image-expand"/> Self-managed \n data plane nodes \n #40;hosted by you in cloud provider#41;)
E(<img src="/assets/images/logos/KogoBlue.svg" style="max-height:20px" class="no-image-expand"/> Fully-managed \n data plane nodes \n #40;hosted by Kong#41;)

subgraph id1 [Konnect]
A
B
end

A --proxy configuration---> C & D
B --proxy configuration---> E

style id1 stroke-dasharray:3,rx:10,ry:10
style A stroke:none,fill:#0E44A2,color:#fff
style B stroke:none,fill:#0E44A2,color:#fff

{% endmermaid %}
<!--vale on-->


#### 制御プレーンの構成

各制御プレーンについて、データプレーンノードをスピンアップし、次のように設定できる。
以下の{{site.base_gateway}}エンティティを構成します：
* ゲートウェイ・サービス
* ルート
* 消費者
* コンシューマーグループ
* プラグイン
* アップストリーム
* 証明書
* SNI
* 保管庫
* 鍵

複数のコントロールプレーンがある場合、エンティティのコンフィグレーションは、 それが作成されたコントロールプレーンにのみ適用される。コンシューマとその認証メカニズムは、他のコントロールプレーンには引き継がれない。


[{{site.base_gateway}} configuration in {{site.konnect_short_name}} &rarr;](/konnect/gateway-manager/configuration/)

### コントロール・プレーン・グループ

コントロール・プレーン・グループは、標準{{site.base_gateway}}コントロール・プレーンであるメンバーからのコンフィギュレーションを組み合わせた読み取り専用のコントロール・プレーンです。コントロールプレーングループのすべてのメンバーは、データプレーンノードの同じクラスタを共有します。


コントロールプレーングループの利点は以下の通りである：

* **共有インフラ、個別設定**： ユーザーまたは組織はインフラストラクチャを共有することができ、チームは個々の構成を管理するための独自の標準コントロールプレーンを持つことができます。
* **モジュラー・クラスター**： 標準的なコントロールプレーンをさまざまな方法で組み合わせて、さまざまな目的に応じた独自のコンフィギュレーションを作成できます。
* **クラウドのワークスペース**： 制御プレーングループは{{site.base_gateway}}ワークスペースと同様に機能しますが、クラウドコントロールプレーンの利点が追加されます。


コントロールプレーングループの詳細
* [Intro to control plane groups](/konnect/gateway-manager/control-plane-groups/)
* [Set up and manage control plane groups](/konnect/gateway-manager/control-plane-groups/how-to/)
* [Migrate configuration into a control plane group](/konnect/gateway-manager/control-plane-groups/migrate/)
* [Conflicts in control plane groups](/konnect/gateway-manager/control-plane-groups/conflicts/)

### コントロールプレーン・ダッシュボード

各制御プレーンについて、そのデータプレーンノードのトラフィック、エラー率、および{{site.base_gateway}}サービス分析を表示できます。
これにより、コントロールプレーンの使用量を確認できます。表示するアナリティクスの時間枠を選択することもできます。

### コントロールプレーンの削除

{:.warning}
> **Warning:** 制御プレーンの削除は不可逆的です。以下のことを確認してください。
コントロール・プレーンを削除したいこと、コントロール・プレーン内のすべてのエンティ ティとデータ・プレーン・ノードが説明されていることを確認してください。
ノードが説明されていることを確認してください。

コントロールプレーンを削除するには、ゲートウェイマネージャまたは {{site.konnect_short_name}}[Control Plane API](/konnect/api/control-planes/latest/)を使用します。

コントロールプレーンが削除されると、関連するすべてのエンティティも削除されます。
これには、ゲートウェイマネージャーでこのコントロールプレーンに設定されているすべてのエンティティが含まれます。
ベストプラクティスとして、必要なコンフィギュレーションが失われないように、コントロールプレーンを削除する前にコンフィギュレーションを[バックアップ](/konnect/gateway-manager/backup-restore/)してください。

コントロール・プレーンが削除されたときにまだアクティブであったデータ・プレーン・ノードは終了しませんが、孤児となります。新しいコントロールプレーンに接続されるか、手動でシャットダウンされるまで、最後に受信したコンフィギュレーションを使用してトラフィックの処理を継続する。

## データプレーンノード

データプレーンノードは1つの{{site.base_gateway}}インスタンスである。
データプレーンノードは制御プレーンのトラフィックを処理します。

データプレーンノードは以下の方法で配置できます：
* **Fully-managed**: 
    * 専用クラウドゲートウェイのデータプレーンノードは、お客様が選択したクラウドプロバイダーでKongが完全に管理します。お客様はゲートウェイインフラのサイズとロケーションを管理し、Kongは各インスタンスとクラスタ全体の管理を行います。Gateway ManagerのDedicated Cloud Gatewaysウィザードを使用して、クラウドプロバイダーの{{site.base_gateway}}データプレーンノードをプロビジョニングできます。

    * サーバーレスゲートウェイのデータプレーンノードも完全にKongによって管理されるが、インフラオプションはすべてユーザーから抽象化される。データプレーンノードの配置、サイズ、数を設定することはできない。

* **Self-managed**: データプレーンノードは、自社システムまたは外部のクラウドプロバイダーでホストされます。Gateway Manager のスクリプトを使用して、Linux、MacOS、または Windows を実行する Docker コンテナで {{site.base_gateway}} データプレーン ノードをプロビジョニングできます。


以下の表は、ユースケースに応じて、どのデータプレーン・ノード・ストラテジーを使用するかを決定するのに役立ちます：


| ユースケース | データプレーンノード戦略 | ソリューション |
| ------- | ----------- | ----------- |
| 組織にとってレイテンシを削減することが重要です。 | [Dedicated Cloud Gateways](/konnect/gateway-manager/dedicated-cloud-gateways/) | 複数のAWSおよびAzureリージョンに対応：<br><br>- AWSリージョン: シドニー、東京、シンガポール、フランクフルト、アイルランド、ロンドン、オハイオ、オレゴン。<br>- Azureリージョン: フランクフルト、アイルランド、UKサウス、バージニア、ワシントン。 |
| 組織が厳格なデータ保護およびプライバシー要件を持つ業界で運営されています。 | [Dedicated Cloud Gateways](/konnect/gateway-manager/dedicated-cloud-gateways/) | プライベートゲートウェイオプションを使用することで、Kongはプライベートネットワークロードバランサーをプロビジョニングし、UIでIPアドレスのみを公開します。 |
| 高可用性が必要で、データプレーンノードをアップグレードする際のダウンタイムをゼロにしたい。 | [Dedicated Cloud Gateways](/konnect/gateway-manager/dedicated-cloud-gateways/) | データプレーンノードをアップグレードしてもダウンタイムが発生しません。また、クラスタを事前にウォームアップさせることで、初回リクエストがインフラのスケールアップを待つ必要がありません。 |
| 複数のクラウドにインフラを持っています。 | [Dedicated Cloud Gateways](/konnect/gateway-manager/dedicated-cloud-gateways/) | Dedicated Cloud Gatewaysを使用することで、マルチクラウドソリューションを実行し、API運用を標準化して複雑性を軽減し、俊敏性を向上させることができます。 |
| 実験やサンドボックスユースケースのために非常に迅速なプロビジョニングが必要です。 | [Serverless Gateways](/konnect/gateway-manager/serverless-gateways/) | Serverless Gatewaysは、1分未満のプロビジョニング時間を提供し、迅速なイテレーションと開発ライフサイクルを可能にします。 |
| AWSまたはAzure以外のクラウドプロバイダーを使用しているか、組織ポリシーのためクラウドでのホスティングを望んでいません。 | [Self-managed](/konnect/gateway-manager/data-plane-nodes/) | AWS、Azure、Google Cloudでデータプレーンノードをセルフ管理できます。また、macOS、Windows、Linux（Docker）、またはKubernetesでセルフ管理のデータプレーンノードを展開することも可能です。 |


詳しくは[データプレーンノードのインストールオプション](/konnect/gateway-manager/data-plane-nodes/)を参照してください。


## プラグイン


{{site.konnect_short_name}}はプラグインを使って拡張できます。Kongは、{{site.konnect_short_name}}にバンドルされる標準Luaプラグインのセットを提供します。利用できるプラグインのセットはインストールによって異なります。


カスタムプラグインはKongコミュニティによって開発され、プラグイン作成者によってサポートおよびメンテナンスされます。これらのプラグインがKong Plugin Hubで公開されている場合、コミュニティまたはサードパーティプラグインと呼ばれます。


詳しくは[{{site.konnect_short_name}}プラグイン注文ドキュメント](/konnect/reference/plugins/)を参照してください。

