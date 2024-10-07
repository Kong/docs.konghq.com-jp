---
title: "Kong Gatewayのアップグレード"
content_type: "reference"
purpose: "このガイドでは、{{site.base_gateway}} のアップグレードパスについてご紹介します。アップグレードの準備にお役立ていただけます。"
---
このガイドを使用して、{{site.base_gateway}}アップグレードを準備し、使用する{{site.base_gateway}}アップグレードパスを決定します。

このガイドでは、利用可能な4つのアップグレードストラテジについて説明し、各{{site.base_gateway}}デプロイメントモードに最適なストラテジを推奨します。
さらに、アップグレードプロセスで重要な役割を果たすいくつかの基本的な要素をリストアップし、データのバックアップと復元の方法について説明します。

このガイドでは、{{site.base_gateway}}のコンテキスト内で次の用語を使用します。

* **アップグレード** ：{{site.base_gateway}}の古いバージョンから新しいバージョンに切り替える全体的なプロセスです。
* **移行** ：データストアのデータを新しい環境に移行します。 たとえば、2\.8\.xのデータを古いPostgreSQLインスタンスから3\.4\.xの新しいインスタンスに移動するプロセスは、データベース移行と呼ばれます。

{:.note}
> 
> **注** ：{{site.ee_product_name}} 2\.8\.xおよび3\.4\.xの長期的
> サポート（LTS）バージョンのアップグレードについては、[LTSアップグレードガイドを](/gateway/{{page.release}}/upgrade/lts-upgrade/)参照してください。

アップグレードの概要
----------

{{site.base_gateway}}アップグレードには、アップグレードの準備とアップグレードの適用という2つのフェーズの作業が必要です。

**準備フェーズ** 

1. 以下で、プラットフォームのバージョンとアップグレード先の{{site.kong_gateway}}バージョン間のバージョン互換性を確認します。 
   * [OSバージョン](/gateway/{{page.release}}/support-policy/#supported-versions)
   * [KubernetesバージョンとHelmの前提条件](/kubernetes-ingress-controller/latest/support-policy/)
   {% if_version gte:3.2.x -%}
   * [データベースのバージョン](/gateway/{{page.release}}/support/third-party/)
   * [依存関係のバージョン](/gateway/{{page.release}}/support/third-party/) {% endif_version %}

2. 開始するリリースとアップグレードするリリースに基づいて、[アップグレードパス](#preparation-review-upgrade-paths)を決定します。
3. データベースまたは宣言型構成ファイルを[バックアップ](#preparation-choose-a-backup-strategy)します。
4. お使いのデプロイメントトポロジーに基づいて、適切な[アップグレードストラテジ](#preparation-choose-an-upgrade-strategy-based-on-deployment-mode)を選択します。
5. アップグレードするバージョンの[下位互換性のない変更](#preparation-review-breaking-changes-and-changelogs)を確認します。
6. 残りの[アップグレードの考慮事項](#preparation-upgrade-considerations)があれば確認します。
7. 運用前環境で移行をテストします。

**アップグレードの実行** 

アップグレードの実際の実行形態は、{{site.base_gateway}}でのデプロイメントのタイプによって異なります。
アップグレードプロセスのこの段階では、準備フェーズで決定したストラテジを使用します。

1. [選択したアップグレードストラテジをdev上で実行します](#perform-the-upgrade)。
2. devからprodに移行します。
3. スモークテスト。
4. アップグレードを終了するか、ロールバックして再試行します。

それでは、アップグレードパスを決定することから始めて、準備に移りましょう。

準備：アップグレードパスの確認
---------------


{{site.base_gateway}}は、製品のバージョン管理において、メジャーバージョン、マイナーバージョン、パッチバージョンを区別する構造化されたアプローチを採用しています。

2\.xから3\.xへのアップグレードは、 **メジャー** アップグレードです。
{{site.base_gateway}} 3\.0\.xが移行をサポートする最も低いバージョンは、2\.1\.xからの移行です。


{{site.base_gateway}}では、1\.xから3\.xへの直接アップグレードをサポートしていません。
1\.xを使用している場合は、まずは2\.1\.0にアップグレードしてください。その後、3\.0\.xにアップグレードしてください。

最新バージョンに直接アップグレードすることもできますが、
2\.xシリーズおよび3\.xシリーズ間における下位互換性のない変更
（直近のバージョンと以前のバージョンの両方）および
[オープンソース（OSS）](/gateway/{{page.release}}/breaking-changes/)と
[Enterprise](/gateway/changelog/)ゲートウェイの変更ログを確認してください。{{site.base_gateway}}以降は
オープンソースの基盤上に構築されているため、OSSにおける下位互換性のない変更はすべての{{site.base_gateway}}パッケージに影響します。

アップグレードパスには広範な条件が適用され、デプロイメントモード、カスタムプラグイン、技術的機能、ハードウェア容量、SLAなどに応じて異なるため、すべてのユーザーに適用される万能のものはありません。アップグレードの際は、アクションを講じる前に、プロセスについて当社のエンジニアと網羅的かつ慎重に話し合ってください。

スムーズなアップグレードパスの維持に役立つため、{{site.base_gateway}}リリースを常に最新の状態に保つことをお勧めします。
バージョンのギャップが小さいほど、アップグレードプロセスは簡単になります。

### 保証されたアップグレードパス

デフォルトでは、{{site.base_gateway}}には隣接するバージョン間の移行テストがあるため、次のアップグレードパスが正式に保証されます。

1. 同じメジャーバージョンとマイナーバージョンのパッチリリース間。

2. 同じメジャーバージョンの隣接するマイナーリリース間。

3. LTS（Long Term Support）バージョン間。

   
    {{site.base_gateway}}はLTSバージョンを維持し、隣接するLTSバージョン間のアップグレードを保証します。
   2\.xシリーズの現在のLTSは2\.8であり、3\.xシリーズの現在のLTSは3\.4です。
   2\.8と3\.4のLTSバージョン間でアップグレードする場合は、
   [LTSアップグレードガイド](/gateway/{{page.release}}/upgrade/lts-upgrade/)を参照してください。

次の表は、現在使用している{{site.base_gateway}}バージョンに応じて、{{page.release}}するさまざまなアップグレードパスシナリオを示しています。

{% include_cached /md/gateway/upgrade-paths.md release=page.release %}

準備：バックアップストラテジを選択する
-------------------

{% include_cached /md/gateway/upgrade-backup.md release=page.release %}

準備：デプロイメントモードに基づいてアップグレードストラテジを選択する
-----------------------------------

独自のアップグレード手順を定義することもできますが、このセクションで紹介されている方法のいずれかを使用することをお勧めします。
カスタムのアップグレード要件には、適切に調整されたアップグレード戦略が必要になる場合があります。
たとえば、少数のカスタマーオブジェクトのみを新しいバージョンに送りたい場合は、
トラフィックインターセプトをサポートするCanaryプラグインとロードバランサーを使用します。

どちらのアップグレードストラテジを選択する場合でも、アップグレードのプロセス中は、Admin API操作とデータベースの更新が許可されないため、
{{site.base_gateway}}の管理ダウンタイムを考慮する必要があります。

デプロイメントタイプに基づいて、次のいずれかのアップグレードストラテジをお勧めします。
各オプションの説明を注意深く読み、最適なアップグレードストラテジを選択してください。

* [従来モード](#traditional-mode)または[ハイブリッドモードのコントロールプレーン](#control-planes)：

  * [デュアルクラスターのアップグレード](/gateway/{{page.release}}/upgrade/dual-cluster/)
  * [インプレースアップグレード](/gateway/{{page.release}}/upgrade/in-place/)
  * [ブルーグリーンアップグレード](/gateway/{{page.release}}/upgrade/blue-green/)（非推奨）

* [DBレスモード](#db-less-mode)または[ハイブリッドモードのデータプレーン](#data-planes)：

  * [ローリングアップグレード](/gateway/{{page.release}}/upgrade/rolling-upgrade/)

意思決定プロセスがどのように機能するかを詳しく説明したフローチャートを以下に示します。

{% include_cached md/gateway/upgrade-flow.md %}

各アップグレードストラテジガイドの詳細とリンクについては、次のセクションを参照してください。

### 従来モード

{% include_cached /md/gateway/traditional-upgrade.md %}

{:.important}
> 
> **重要** : [ブルーグリーンアップグレードストラテジ](/gateway/{{page.release}}/upgrade/blue-green/)はオプションですが、
> お勧めしません。このストラテジを使用したアップグレードに対するKongからのサポートは限られています。
> {{site.base_gateway}}バージョンの数、アップグレードストラテジ、採用された機能、デプロイメントモードを考慮してすべての組み合わせを網羅する必要があるため、すべての移行テストを完全に網羅することはほぼ不可能です。
> どうしてもこのストラテジを採用したい場合は、パッチバージョン間のアップグレードにのみ使用してください。

### DBレスモード

{% include_cached /md/gateway/db-less-upgrade.md release=page.release %}

### ハイブリッドモード

{% include_cached /md/gateway/hybrid-upgrade.md release=page.release %}

#### 3\.1\.0\.0または3\.1\.1\.1からのアップグレード

ハイブリッドモードで{{site.base_gateway}}をデプロイし、使用しているバージョンが3\.1\.0\.0または3\.1\.1\.1\.の場合は、特別なケースがあります。
Kongは、CPとDPの間のWebSocketプロトコルとして、3\.1\.0\.0ではレガシー版を削除し新しいものに置き換えましたが、3\.1\.1\.2ではレガシー版を再び追加しました。
そのため、上記の特別なケースでバージョンをアップグレードする場合は、まず、3\.1\.1\.2にアップグレードする必要があります。

さらに、新しいWebSocketプロトコルは3\.2\.0\.0以降完全に削除されました。
バージョンの内訳については、次の表を参照してください。

|          
{{site.base_gateway}}バージョン          | 従来のWebSocket（JSON） | 新しいWebSocket（RPC） |
|-------------------------------|--------------------|-------------------|
| 3\.0\.0\.0                 | Y                  | Y                 |
| 3\.1\.0\.0および3\.1\.1\.1 | N                  | Y                 |
| 3\.1\.1\.2                 | Y                  | Y                 |
| 3\.2\.0\.0                 | Y                  | N                 |

準備：下位互換性のない変更と変更ログを確認する
-----------------------

アップグレード先のリリースの[下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)と[変更ログ](/gateway/changelog/)
を確認します。
下位互換性のない変更の指示に従って、準備または調整を行います。

準備: アップグレードに関する考慮事項
-------------------

デプロイメントモードに関係なく、アップグレードに影響を与える可能性のある普遍的な要因がいくつかあります。

ターゲットデプロイメントモードのストラテジを選択しても、アップグレードをすぐに開始できるとは限りません。
また、次の要素も考慮する必要があります。

* アップグレードプロセス中は、データベースに変更を加えることはできません。
  アップグレードが完了するまで、次の行為は控えてください。

  * [Admin API](/gateway/{{page.release}}/admin-api)経由でデータベースに書き込まないでください。
  * データベースを直接操作しないでください。
  * [Kong Manager](/gateway/{{page.release}}/kong-manager/)、[decK](/deck/)、または[kong構成CLI](/gateway/{{page.release}}/reference/cli/#kong-config)を通じて 構成を更新しないでください。 

* 新しいバージョンYと既存のプラットフォーム間の互換性を確認します。
  要因には、以下が含まれますが、これらに限定されません。

  * [OSバージョン](/gateway/{{page.release}}/support-policy/#supported-versions)
  * [KubernetesバージョンとHelmの前提条件](/kubernetes-ingress-controller/latest/support-policy/)
  * [ハードウェアリソース](/gateway/{{page.release}}/production/sizing-guidelines/)
  {% if_version gte:3.2.x -%}
  * [データベースのバージョン](/gateway/{{page.release}}/support/third-party/)
  * [依存関係のバージョン](/gateway/{{page.release}}/support/third-party/) {% endif_version %}

* 現在のバージョンXから始まるすべての[変更ログ](/gateway/changelog/)を注意深く確認してください。
  変更ログはターゲットバージョンYまであります。

  * 特にスキーマの変更や機能の削除など、潜在的な競合がないか確認します。

  * 新しいクラスタを設定するときは、`kong.conf` を直接アップグレードするか、変更履歴に基づき環境変数を使用してアップグレードします。

    まれに、マイナーバージョンアップグレードで `kong.conf` に下位互換性のない変更が行われることもあります。

    たとえば、パラメータ`pg_ssl_version`は、2\.8\.2\.4ではデフォルトで`tlsv1`になります。しかし、3\.2\.2\.1では、`tlsv1`は有効な引数ではありません。
    この設定に依存していた場合は、新しいオプションの1つに合わせて環境を調整する必要があります。

* カスタムプラグインがある場合は、コードを変更ログと照合して、新しいバージョンYを使用してカスタムプラグインをテストしてください。

* `nginx-kong.conf`や`nginx-kong-stream.conf`などのNginxテンプレートを変更した場合は、新しいバージョンYのテンプレートにも同様の変更を加えます。
  詳細なカスタマイズガイドについては、[Nginxディレクティブ](/gateway/{{page.release}}/reference/nginx-directives/)を参照してください。

* {{site.ee_product_name}}を使用している場合は、新しいゲートウェイクラスタに必ず[Enterpriseライセンスを適用](/gateway/{{page.release}}/licenses/deploy/)してください。

* 必ず[バックアップ](/gateway/{{page.release}}/upgrade/backup-and-restore/)を取ってください。
  {% if_version gte:3.4.x -%}

* Cassandra DBのサポートは、3\.4\.0\.0をもって{{site.base_gateway}}から除外されました。
  [CassandraからPostgreSQLへの移行ガイドライン](/gateway/{{page.release}}/migrate-cassandra-to-postgres/)に従って、PostgreSQLに移行してください。
  {% endif_version %}

アップグレードの実行
----------

すべてを確認してストラテジを選択したら、選択したストラテジを使用して
{{site.base_gateway}}のアップグレードに進みます。

* [デュアルクラスターのアップグレード](/gateway/{{page.release}}/upgrade/dual-cluster/)
* [インプレースアップグレード](/gateway/{{page.release}}/upgrade/in-place/)
* [ブルーグリーンアップグレード](/gateway/{{page.release}}/upgrade/blue-green/)
* [ローリングアップグレード](/gateway/{{page.release}}/upgrade/rolling-upgrade/)

