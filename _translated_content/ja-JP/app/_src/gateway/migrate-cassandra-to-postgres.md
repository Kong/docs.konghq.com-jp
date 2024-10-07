---
title: "カサンドラから PostgreSQL への移行ガイドライン"
---
このガイドでは、{{site.base_gateway}}をCassandra DBベースの[従来](/gateway/{{page.release}}/production/deployment-topologies/traditional/)のデプロイメントからPostgreSQLベースの[ハイブリッド](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/)デプロイメントに移行する手順を説明します。

このガイドでは、ブルーグリーン移行アプローチを採用しています。このアプローチでは、DNS スイッチングにより、トラフィックをブルー環境からグリーン環境に切り替えることができます。これにより、問題が検出された場合に、より速くロールバックできるようになります。

Kongでは、宣言型コンフィギュレーション管理ツールの[decK](/deck/)を推奨しています。

移行手順：

1. PostgreSQLを使用してターゲット{{site.base_gateway}}ハイブリッド環境を構築します。これは空白のキャンバスとして機能します。
2. decK を使用して、従来の Cassandraベースのデプロイメントから（ `dump`）構成をエクスポートします。
3. decKを使用して、PostgreSQLを活用する新しいハイブリッド型デプロイメントに構成をインポート（`sync`）します。
4. Admin APIまたはKong Managerを使用して、基本認証情報とローカルRBACユーザーを作成します。
5. {{site.base_gateway}}構成がグリーン環境に移行されたら、カナリア経由でトラフィックをグリーン環境に徐々にリダイレクトします。これにより、迅速なロールバックメカニズムが確立され、リリースのリスクが軽減されます。

考慮事項
----

このドキュメントには大まかなガイドラインが記載されていますが、実際の移行手順は次の要因によって異なる場合があります。

* Kongデプロイ環境の複雑さ
* Kong plugins
* カスタムプラグイン
* ゲートウェイへの外部タッチポイント（外部IdPを備えたOIDCなど）
* Number of configuration entities, for example services and routes
* {{site.base_gateway}}の古いバージョンを実行している場合は、データベースの移行前に{{site.base_gateway}}バージョンへのアップグレードを実行することをお勧めします。これにより、アップグレード手順の移行部分が軽減されます。
* Cassandra を使用している場合は、従来のデプロイメントを使用している可能性があります。この機会に {{site.base_gateway}} の[デプロイメントトポロジーのオプション](/gateway/{{page.release}}/production/deployment-topologies/)を確認し、可能であればハイブリッドモードのデプロイメントに変換することをお勧めします。

次の図は、ハイブリッドモードデプロイメントのアーキテクチャを示しています。つまり、{{site.base_gateway}} コントロールプレーン（CP）とデータプレーン（DP）が分割されていることを意味します。従来のモードでデプロイされた {{site.base_gateway}} インスタンスでも、同じデータベース移行アプローチを使用できます。

{% mermaid %}
flowchart LR
A[api.customer.com]
B[(Cassandra)]
C(<img src="/assets/images/logos/KogoBlue.svg" style="max-height:20px" class="no-image-expand"/> {{site.base_gateway}} VM)
D(<img src="/assets/images/logos/KogoBlue.svg" style="max-height:20px" class="no-image-expand"/> {{site.base_gateway}} DP)
E(<img src="/assets/images/logos/KogoBlue.svg" style="max-height:20px" class="no-image-expand"/> {{site.base_gateway}} CP)
F[(Postgres)]

H[[decK]]

A --> C & D

subgraph id1 ["`**On premise**`"]
C --> B
end

subgraph id2 ["`**Kubernetes**`"]
D --> E --> F
end

C --export config via yaml--> H --sync config to CP--> E

style id1 stroke-dasharray:3,rx:10,ry:10
style id2 stroke-dasharray:3,rx:10,ry:10

{% endmermaid %}

前提条件
----

* {{site.base_gateway}}ブルー環境 \(Cassandra を使用\) とグリーン環境 \(PostgreSQL を使用\) は、同じゲートウェイバージョンを実行しています。
* 次の例では、ブルーは従来モードでデプロイされ、グリーンはハイブリッドモードでデプロイされます。従来のデプロイメントに移行したい場合でも、手順に従うことはできますが、コントロールプレーンとデータプレーンのタスクを別々に実行する必要はありません。

Migration approach
------------------

次の手順は、非本番環境でテストする必要があります。ターゲット状態のギャップは、本番環境で実行する前に特定して修正する必要があります。

| 手順 |      名前       |                              説明                              |
|----|---------------|--------------------------------------------------------------|
| 1  | プラットフォーム構築    | ブルー環境と同じバージョンを使用して、{{site.base_gateway}}でグリーン環境を構築します                       |
| 2  | データベースのセットアップ | 新しいPostgres DBを構築し、グリーン環境に接続します                              |
| 3  | 回帰テスト         | セットアップを新しいシステムに展開し、回帰テストを実行します。テストが完了したら、デプロイメントをクリーンアップします。 |

準備
---

| 手順 |        名前        |                                                説明                                                 |
|----|------------------|---------------------------------------------------------------------------------------------------|
| 1  | Change embargo   | Place a change embargo on the old environment preventing new deployments or configuration changes |
| 2  | ブルー環境のバックアップ     | ブルー環境の Cassandra データベースのバックアップを作成し、冗長ストレージに配置します。                                                 |
| 3  | 監視のセットアップ        | Create dashboards to track the health of the new environment                                      |
| 4  | Go/No Goチェックポイント | 移行実行の Go/No Go 決定ポイント                                                                             |

Execution
---------

このフェーズの目標は、ブルー環境で{{site.base_gateway}}構成を再現し、ライブトラフィックの切り替え前に回帰テストとスモークテストを実行することです。

| 手順 |        名前        |                                                                                                                                                                                                                                      説明                                                                                                                                                                                                                                       |
|----|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1  | 構成ダンプ            | \- Execute the `deck gateway dump` command against the blue environment for each workspace to build a YAML representation of the Kong estate<br>\- Execute `deck gateway dump --rbac-resources-only` against the blue environment for each workspace to build a YAML representation of any RBAC resources created<br>\- If using the Kong Dev Portal, run the `portal fetch` command via the Portal CLI to build a representation of the Dev Portal assets for a workspace |
| 2  | 構成の同期            | \- decK をグリーン環境で動作するように変更し、 `deck gateway sync`を実行して構成をグリーン環境にプッシュします。<br> \- ポータル CLI をグリーン環境に対して動作するように変更し、 `portal deploy`を実行して、Dev Portal 構成を新しい環境にプッシュします。                                                                                                                                                                                                                                                                                                             |
| 3  | 回帰テスト            | PostgreSQLを基盤としたグリーン環境に対して回帰テストを実行します                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 4  | Go/No Goチェックポイント | すべてのテストに合格したら、トラフィックの切り替えを実施します。                                                                                                                                                                                                                                                                                                                                                                                                                                              |

トラフィックのカットオーバー
--------------

このフェーズの目的は、問題が検出された場合の迅速なフォールバックメカニズムを使用して、制御された方法でトラフィックを移行することです。リスク許容度に応じて、カナリア分割を増やす前に待機時間を増減できます。

| 手順 | 名前                                                  | 説明|
|\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-|
| 1    | トラフィックの1%の新しいグリーン環境へのカナリアデプロイメント      | トラフィックの1%を新しいグリーン環境にカナリアデプロイし、トラフィックヘルスを監視します。 |
| 2    | トラフィックの5%の新しいグリーン環境へのカナリアデプロイメント      | 分割トラフィックを5%に引き上げ、監視します。 |
| 3    | トラフィックの10%の新しいグリーン環境へのカナリアデプロイメント     | 分割トラフィックを10%に引き上げ、監視します。 |
| 4    | トラフィックの50%の新しいグリーン環境へのカナリアデプロイメント     | 分割トラフィックを50%に日陰、監視します。|
| 5    | トラフィックの100%の新しいグリーン環境への切り替え    | 新しいグリーン環境を指すように完全なDNSの切り替えを実行します。|
| 6    | 旧プラットフォームの撤去 | 信頼性が高くなる24時間後に、旧プラットフォームを撤去します。 |

次の手順
----

* If using the Basic Authentication plugin, the passwords are hashed and encrypted in the database. A decK dump doesn't export these credentials, so it is impossible to get the passwords out of the database in the cleartext. You must migrate these credentials using the Admin Api or Kong Manager.

* Admin users on the {{site.base_gateway}} control plane are not propagated using decK. Migrate them using the Kong Admin API.

移行後、ベーシック認証情報はdecKで管理できます。decKでシークレットを管理する方法については、以下のトピックを参照してください。

* [ゲートウェイシークレット管理GCP](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/gcp-sm/)
* [decKによるシークレット管理](/deck/latest/guides/vaults/#configure-a-secret-vault)

