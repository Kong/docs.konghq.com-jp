---
title: "概要"
badge: "enterprise"
content_type: "explanation"
---

{{site.ee_product_name}}は、最も高速で、最も採用されているAPIゲートウェイである{{site.base_gateway}}を拡張する、スケーラブルで安全かつ柔軟なAPIマネジメントソリューションです。Enterpriseプラグイン、高度なセキュリティ機能、GUI、24時間年中無休のサポートを追加します。ハイブリッドクラウドとマルチクラウドのアーキテクチャにわたるアプリケーション間の接続を管理、保護、監視することで、クラウドへの移行を加速し、より迅速な拡張と開発者の生産性向上を支援する唯一のソリューションです。

Enterpriseプラグイン
---------------


{{site.ee_product_name}} は、すぐに利用可能の 400 を超える Enterprise プラグインとコミュニティプラグインへのアクセスを提供します。
これには、[Rate Limiting Advanced プラグイン](/hub/kong-inc/rate-limiting-advanced/)などの OSS プラグインの特別バージョンが含まれ、コンシューマグループの使用やデータベース固有のストラテジなどの追加機能を備えています。また、[OpenID Connect](/hub/kong-inc/openid-connect/) を使用した認証といった Enterprise 特別機能も提供しており、これにより ID プロバイダー（IdP）の統合を標準化できます。


{{site.ee_product_name}}は、gRPC と REST、WebSocketをネイティブにサポートし、Apollo GraphQLサーバーおよび Apache Kafkaサービスと統合します。これらのプラグインを活用すると、次のような高度な接続機能とソリューションを{{site.base_gateway}}に提供できます。

* [OpenID Connect \(OIDC\)](/hub/kong-inc/openid-connect/)
* [Kafka を使用したイベント ゲートウェイ](/hub/kong-inc/kafka-upstream/)
* [GraphQL](/hub/kong-inc/graphql-proxy-cache-advanced/)
* [Mocking](/hub/kong-inc/mocking/)
* [高度なデータ変換](/hub/kong-inc/jq/)
* [OPA Policy駆動型トラフィック管理](/hub/kong-inc/opa/)
{% if_version lte:3.3.x -%}
* [API製品階層](/gateway/{{page.release}}/admin-api/consumer-groups/reference/)
{% endif_version -%}
{% if_version gte:3.4.x -%}
* [API プロダクトティア](/gateway/api/admin-ee/latest/#/consumer_groups/get-consumer_groups) {% endif_version %} [プラグインを使い始める →](/hub/)

{% if_version lte:3.4.x %}

Dev Portal
----------

Dev Portal は、従来の API カタログと同様に、すべての開発者が API を検索、アクセス、使用するための信頼できる唯一の情報源を提供します。
Dev Portal は、 {{site.base_gateway}}から公開されたサービスを検出、登録、および使用するためのセルフサービス開発者エクスペリエンスを提供することで、開発者のオンボーディングを効率化します。
このカスタマイズ可能なエクスペリエンスは、独自のブランドに合わせて使用でき、サービスのドキュメントとインタラクティブな API 仕様を強調表示します。
さらに、アプリケーション登録を有効にすることで、さまざまな認証プロバイダーを使用して API を保護できます。

[Dev Portal の詳細情報について →](/gateway/{{page.release}}/kong-enterprise/dev-portal/)

監視と分析
-----

Vitalsプラットフォームは、サービス、ルート、アプリケーションの使用状況データに関する詳細なインサイトを提供します。カスタムレポートとコンテキストダッシュボードを使用してAPI製品のヘルスを確認できるほか、[Datadog](/hub/kong-inc/datadog/)や[Prometheus](/hub/kong-inc/prometheus/)などのサードパーティ分析プロバイダーへの監視メトリクスのストリーミングを可能にする{{site.base_gateway}}プラグインを使用して、ネイティブの監視および分析機能を強化できます。

[Vitalsで監視をスタート →](/gateway/{{page.release}}/kong-enterprise/analytics/)

{% endif_version %}

ロールベースのアクセス制御 \(RBAC\)
------------------------


{{site.ee_product_name}}では、組み込みのロールベースのアクセス制御 \(RBAC\) を使用して、ユーザー、ロール、および権限を構成できます。RBAC を使用すると、開発者のオンボーディングを効率化し、 [Admin API](/gateway/api/admin-ee/latest/)または[Kong Manager](/gateway/{{page.release}}/kong-manager/auth/rbac/)を使用してきめ細かいセキュリティおよびトラフィック ポリシーを作成して適用できます。

[RBACでチームを管理する →](/gateway/{{page.release}}/kong-manager/auth/rbac)

シークレット管理
--------


{{site.ee_product_name}} は、次のバックエンドを使用してすぐに使用できるシークレット管理を提供します。

* [Amazon Web Services（AWS）](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/aws-sm/)
{% if_version gte:3.5.x -%}
* [Microsoft Azure](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/azure-key-vaults/)
{% endif_version -%}
* [Google Cloud Platform \(GCP\)](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/gcp-sm/)
* [HashiCorp Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/hashicorp-vault/)

シークレット管理を構成するには、{{site.base_gateway}} はバックエンドプロバイダーでキーを使用し、バックエンドプロバイダーで認証し、バックエンドを使用してアプリケーションシークレット、機密データ、パスワード、キー、証明書、トークン、およびその他の項目を一元管理および保存します。

[アプリケーションシークレットの保護→](/gateway/{{page.release}}/kong-enterprise/secrets-management/)

キーリングとデータ暗号化
------------

キーリングとデータ暗号化機能は、保存中の機密データフィールドを透過的かつ対称的に暗号化できるようにします。有効にすると、{{site.base_gateway}}はデータベースへの書き込み直前、またはデータベースからの読み取り直後にデータを暗号化・復号化します。機密フィールドを含むAdmin APIによって生成された応答データは引き続きプレーンテキストとして表示され、機密フィールドへのアクセスを必要とする{{site.base_gateway}}のランタイム要素（プラグインなど）は、追加の構成を必要とせずに透過的にアクセスします。


{{site.base_gateway}}を使用すると、コンシューマのシークレットなどの機密データフィールドを、暗号化された形式でデータベース内に保存できます。
これにより、{{site.base_gateway}}クラスタでの保存時の暗号化セキュリティ制御を実現します。

[キーリングとデータ暗号化の設定 →](/gateway/{{page.release}}/kong-enterprise/db-encryption/)

監査ログ
----


{{site.base_gateway}} は Admin API の詳細なログ取得を実現します。コンプライアンスへの取り組みのため、およびフォレンジック調査中に貴重なデータポイントを提供するために、クラスタ構成に対して行われた変更をその有効期間全体にわたって詳細に追跡できます。生成された監査ログ証跡は[ワークスペース](/gateway/api/admin-ee/latest/#/Workspaces)および [RBAC](/gateway/api/admin-ee/latest/) に対応しているため、{{site.base_gateway}} のオペレータはクラスタ内で発生した変更を詳細かつ広範に調べることができます。

[監査ログの使用を開始する →](/gateway/{{page.release}}/kong-enterprise/audit-log/)

FIPS のサポート
----------


{{site.ee_product_name}}は自己管理型のFIPS140\-2 ゲートウェイパッケージを備えており、厳格なコンプライアンスとセキュリティを考慮する規制の厳しい業界に最適です。
通常、米国連邦政府機関およびその請負業者と連携する場合には、この規格に準拠する必要があります。

[FIPSサポートの詳細 →](/gateway/{{page.release}}/kong-enterprise/fips-support/)

ワークスペース
-------

ワークスペースは、{{site.base_gateway}}エンティティをセグメント化またはグループ化する方法を提供します。ワークスペース内のエンティティは、他のワークスペース内のエンティティから分離されます。

{{site.ce_product_name}}は、1つのワークスペースに制限されています。{{site.ee_product_name}}を使用すると、複数のワークスペースを活用して、開発者がプロジェクト間を簡単に移行したり、異なるアップストリームに属するサービスとルートを分離したりできるようになります。

[ワークスペースの詳細 →](/gateway/{{page.release}}/kong-manager/workspaces/)

動的プラグイン順序付け
-----------

動的プラグイン順序付けでは、各プラグインの `ordering` フィールドを使用して、任意の {{site.base_gateway}} プラグインの優先順位を上書きできます。
これにより、`access`フェーズ中のプラグインの順序が決まり、
プラグイン間の *動的な* 依存関係を作成できます。

[動的プラグイン順序付けを開始します→](/gateway/{{page.release}}/kong-enterprise/plugin-ordering/)

イベントフック
-------

イベントフックは {{site.base_gateway}} からのアウトバウンドコールです。イベントフックを使用すると、{{site.base_gateway}} はターゲットのサービスやリソースと通信して、イベントがトリガーされたことをターゲットに知らせます。{{site.base_gateway}} でイベントがトリガーされると、そのイベントに関する情報を含む URL が呼び出されます。イベントフックは、管理インターフェースを使用してワーカーイベントを購読するための構成レイヤーを追加します。

{{site.base_gateway}}では、これらのコールバックは次のいずれかのハンドラを使用して定義できます。

* webhook
* webhook\-custom
* ログ
* lambda

Admin APIを通じてイベントフックを構成できます。

[イベントフックの詳細を確認します→](/gateway/api/admin-ee/latest/#/Event-hooks/)

コンシューマグループ
----------

コンシューマグループを使用すると、API エコシステム内のコンシューマ（ユーザーまたはアプリケーション）を組織化して分類できます。コンシューマをグループ化すると、コンシューマを個別に管理する必要がなくなり、構成を管理するためのスケーラブルで効率的なアプローチが実現できます。

例えば、コンシューマグループを使用して流量制限の階層を定義し、各コンシューマを個別に管理する代わりに、コンシューマのサブセットに適用することができます。

{% if_version lte:3.3.x %}

* [コンシューマグループを設定する →](/hub/kong-inc/rate-limiting-advanced/how-to/)
* [コンシューマグループ API リファレンス →](/gateway/{{page.release}}/admin-api/consumer-groups/reference/) {% endif_version %} {% if_version gte:3.4.x %}
* [コンシューマグループ APIの文書化 →](/gateway/api/admin-ee/latest/#/consumer_groups/get-consumer_groups)
* [コンシューマグループをサポートするプラグイン →](/hub/plugins/compatibility/#scopes) {% endif_version %}

{% if_version gte:3.2.x %}

コントロールプレーン（CP）が停止した場合に新しいデータプレーン（DP）をプロビジョニング
---------------------------------------------

バージョン3\.2以降、{{site.base_gateway}}を設定して、コントロールプレーン（CP）が停止した場合に新しいデータプレーン（DP）を構成するのをサポートできます。詳細については、「[コントロールプレーンの停止時に新しいデータプレーンを管理する方法」](/gateway/latest/kong-enterprise/cp-outage-handling/)ドキュメント[またはコントロールプレーン（CP）の停止管理に関するFAQ](/gateway/latest/kong-enterprise/cp-outage-handling-faq/)を参照してください。

{% endif_version %}

{% if_version gte:3.5.x %}

Dockerコンテナイメージの署名
-----------------

{{site.ee_product_name}} 3\.5\.0\.2以降では、Dockerコンテナイメージは署名されており、Docker Hubリポジトリに公開された署名の付いた`cosign`を使用して検証できます。詳細については、[署名されたKongイメージの署名の検証](/gateway/{{ page.release }}/kong-enterprise/signed-images/)ドキュメントをお読みください。
{% endif_version %}

{% if_version gte:3.6.x %}

Dockerコンテナイメージのビルドの来歴
---------------------

KongはDockerコンテナイメージのビルドの由来を生成します。これは、Docker Hubリポジトリに公開された来歴証明と`cosign` / `slsa-verifier`を使用して検証できます。詳細については、[署名済みのKongイメージのビルド来歴証明を検証](/gateway/{{ page.release }}/kong-enterprise/provenance-verification/)ドキュメントをご覧ください。

{% endif_version %}

その他の情報
------

Enterprise 専用プラグインの詳細については、 [プラグインの互換性](/hub/plugins/compatibility/)を参照してください。

