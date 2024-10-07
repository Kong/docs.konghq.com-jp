---
title: "Kong Manager"
---
Kong Managerは、{{site.base_gateway}}のグラフィカルユーザーインターフェース（GUI）です。内部的にはKong Admin APIを使用して{{site.base_gateway}}を管理および制御します。実行する{{site.base_gateway}}のエディションに応じて、オープンソースまたはエンタープライズの2つのオプションがあります。

Kong Manager Enterprise（または無料モード）エディションと OSS エディションでアクセスできる一部の機能の比較を次に示します。

| 機能 | Kong Manager Enterprise | Kong Manager OSS |
|\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|
| すべてのワークスペースを1か所で管理 | ✅ | ❌ |
| ルートとサービスの生成と管理 | ✅ | ✅ |
| プラグインの有効化または無効化 | ✅ | ✅ |
| 証明書の管理 | ✅ | ✅ |
| サービス、プラグイン、コンシューマ、およびその他すべてのものを、必要とするとおりにグループ化する | ✅ | ❌ |
| チームの管理 | ✅ | ❌ |
{% if_version lte:3.4.x -%}
| {{site.base_gateway}}およびDev Portal向けのユーザーとロールの管理 | ✅ | ❌ |
| Dev Portalを構成する：外見をカスタマイズし、開発者およびアプリケーションを管理し、Dev Portalのレイアウト、スペック、ドキュメントを編集する | ✅ | ❌ |
| パフォーマンスの監視：直感的でカスタマイズ可能なダッシュボードを使用して、クラスタレベル、ワークスペースレベルさらにはオブジェクトレベルでのヘルスの監視 | ✅ | ❌ |
{% endif_version -%}
{% if_version gte:3.5.x -%}
| {{site.base_gateway}}向けのユーザーとロールの管理 | ✅ | ❌ |
{% endif_version -%}
{% if_version gte:3.2.x -%}
| キーセットとキーを一元的に管理し、簡単にアクセスできる | ✅ | ✅ |
{% endif_version %}
| Vaultの管理 | ✅ | ✅ |

{:.note}
> 
> **Note** : If you are running Kong in [traditional mode](/gateway/{{page.release}}/production/deployment-topologies/traditional/), increased traffic could lead to potential performance issues for the Kong proxy.
> Server\-side sorting and filtering large quantities of entities can also cause increased CPU usage in both {{site.base_gateway}} and its database.

Kong Managerにアクセスするには、{{site.ce_product_name}}のインストール後に次のURLに移動します：[http://localhost:8002](http://localhost:8002)。

Kong Managerインターフェース
--------------------

{% if_version lte:3.0.x %}
![Kong Managerインターフェース](/assets/images/products/gateway/km_workspace_3.0.png)
{% endif_version %}

{% if_version gte:3.1.x lte:3.4.x %}
![Kong Managerインターフェース](/assets/images/products/gateway/km_workspace_3.1.png)
{% endif_version %}

{% if_version gte:3.5.x %}
![Kong Managerインターフェース](/assets/images/products/gateway/km_workspace_3.5.png)
{% endif_version %}
> 
> *Figure 1: Kong Manager individual workspace dashboard* 

### トップメニュー

{% if_version gte:3.5.x %}

|      アイテム      |                        説明                         |
|----------------|---------------------------------------------------|
| **Workspaces** | クラスタ内のすべてのワークスペースのダッシュボード。                        |
| **チーム**        | RBAC を使用してチームの役割と権限を管理したり、グループを IdP にマッピングしたりします。 |

{% endif_version %}

{% if_version gte:3.1.x lte:3.4.x %}

| 番号 |      アイテム       |                                                                              説明                                                                               |
|----|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1  | **Workspaces**  | クラスタ内のすべてのワークスペースのダッシュボード。                                                                                                                                    |
| 2  | **Dev Portals** | Overview of all Dev Portals in the cluster. At a glance, see which workspaces have active Dev Portals and access their URLs, or set up a Dev Portal instance. |
| 3  | **Vitals**      | クラスタ全体の監視のためのダッシュボード。ダッシュボードを使用して次のことを行います。<br>• リクエストアクティビティを表示する <br>• プロキシとアップストリームのレイテンシーを経時的に追跡する <br>• データストアのキャッシュにいつアクセスされたか、アクセスが成功したかどうかを確認する       |
| 4  | **チーム**         | RBAC を使用してチームの役割と権限を管理したり、グループを IdP にマッピングしたりします。                                                                                                             |
| 5  | **アカウント設定**     | パスワードとRBACトークンを管理します。                                                                                                                                         |

{% endif_version %}

### サイドメニュー

{% if_version gte:3.5.x %}

|      アイテム       |               説明               |
|-----------------|--------------------------------|
| **ダッシュボード**     | ワークスペースに関する情報を表示します。           |
| **API Gateway** | 現在のワークスペースの{{site.base_gateway}}エンティティを管理します。 |
| **シークレット**      | ご使用の環境のデータ保管庫、キー、キーセットを管理します。  |

{% endif_version %}
{% if_version lte:3.4.x %}

| 番号 |      アイテム       |                                                                                                                                                                説明                                                                                                                                                                |
|----|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 6  | **ワークスペースの変更**  | ワークスペースをすばやく変更するためのショートカットです。                                                                                                                                                                                                                                                                                                    |
| 7  | **API Gateway** | 現在のワークスペースの{{site.base_gateway}}エンティティを管理します。                                                                                                                                                                                                                                                                                                   |
| 8  | **Dev Portal**  | ワークスペース固有のDev Portal構成。現在のワークスペースでDev Portalを有効にすると、メニューには追加の項目が表示されます。<br>• 設定: このワークスペースのDev Portalインスタンスの一般的な設定<br>• 外観: Dev Portalの色、フォント、ブランディングをカスタマイズします<br>• 開発者: 開発者のリクエストとアクセスを管理します<br>• アプリケーション: アプリケーションのリクエストとアクセスを管理します<br>• 権限: 役割とコンテンツの権限を管理します<br>• エディター: Dev Portalのファイルエディターにアクセスして、レイアウト、ドキュメント、仕様を構成します |
| 9  | **Vitals**      | ワークスペース内のすべてのサービスのアクセスコード別にリクエストを監視します。                                                                                                                                                                                                                                                                                          |

{% if_version gte:3.1.x %}
| 10 | **Vault** | お使いの環境の Vault シークレットを管理します。|
{% endif_version %}

{% if_version gte:3.2.x %}
| 11 | **キー** | キーセットとキーを一元的に保存し、簡単にアクセスできます。|
{% endif_version %}

{% endif_version %}

{% if_version lte:3.4.x %}

### ワークスペースのダッシュボード

{% if_version lte:3.0.x %}
番号 \| 項目 \| 説明
\-\-\-\-\-\-\-\|\-\-\-\-\-\-\|\-\-\-\-\-\-\-\-\-\-\-\-
10 \| **設定** \| ワークスペースのアバターを編集するか、ワークスペースを削除します。
11｜ **概要パネル** ｜ワークスペースの主な統計情報（サービス数、コンシューマ数、APIリクエスト数、ライセンス有効期限情報）の概要。
12｜ **総トラフィックグラフ** ｜選択した時間枠内のワークスペース内の総トラフィックをステータスコード別に表示します。
13 \| **時間枠セレクタ** \| トラフィックグラフの時間枠を、過去5分間から過去12時間まで選択します。{% endif_version %}

{% if_version gte:3.1.x %}
Number \| Item \| Description
\-\-\-\-\-\-\-\|\-\-\-\-\-\-\|\-\-\-\-\-\-\-\-\-\-\-\-
11 \| **Settings** \| Edit the workspace avatar or delete the workspace.
12 \| **Overview panel** \| Overview of key statistics for the workspace: number of services, consumers, and API requests, as well as the license validity information.
13 \| **Total traffic graph** \| Total traffic in the workspace by status code within a selected time frame.
14 \| **Time frame selector** \| Choose the time frame for the traffic graph, from the last 5 minutes to the last 12 hours.
{% endif_version %}
{% endif_version %}

Kong Manager OSSインターフェース
------------------------

Kong Manager Open Source（OSS）は、 {{site.ce_product_name}}のグラフィカルユーザーインターフェイス（GUI）です。
内部的には Kong Admin API を使用して{{site.ce_product_name}}を管理および制御します。

{:.note}
> 
> **注：** Kong Manager OSS は、 {{site.ce_product_name}}のオープンソースバージョンで使用するように設計されています。

![Kong Manager OSSインターフェース](/assets/images/products/gateway/km_oss.png)
> 
> *図2：Kong Manager OSSの概要* 

|                           アイテム                           |                                                  説明                                                   |
|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **概要**                                                   | {{site.ce_product_name}}に関する情報を含むダッシュボード。                                                                               |
| [**ゲートウェイサービス**](/gateway/{{page.release}}/key-concepts/services/) | {{site.ce_product_name}}に関連付けられているすべてのサービスの概要。このダッシュボードから、新しいサービスを追加したり、既存のサービスを管理したり、すべてのサービスを一目で確認したりできます。            |
| [**ルート**](/gateway/{{page.release}}/key-concepts/routes/)          | {{site.ce_product_name}}に関連付けられているすべてのルートの概要。このダッシュボードから、新しいルートを追加し、既存のルートを管理し、すべてのルートを一目で確認することができます。                  |
| **コンシューマ**                                               | {{site.ce_product_name}}に関連付けられているすべてのコンシューマの概要。このダッシュボードから、新しいコンシューマを追加し、既存のコンシューマを管理し、すべてのコンシューマを一目で確認することができます。      |
| [**プラグイン**](/gateway/{{page.release}}/key-concepts/plugins/)       | {{site.ce_product_name}}に関連付けられているすべてのプラグインの概要。このダッシュボードから、プラグインを有効または無効にすることができます。                                     |
| [**アップストリーム**](/gateway/{{page.release}}/key-concepts/upstreams/)  | {{site.ce_product_name}}に関連するすべてのアップストリームの概要。このダッシュボードから、新しいアップストリームを追加したり、既存のアップストリームを管理したり、すべてのアップストリームを一目で確認したりできます。 |
| **証明書**                                                  | 暗号化されたリクエストのSSL/TLS終了用の証明書を管理します。                                                                     |
| **CA 証明書**                                               | クライアントおよびサーバーの証明書の検証のためにCA証明書を管理します。                                                                  |
| **SNI**                                                  | SNI オブジェクトを管理して、1 つの証明書に対して複数のホスト名をマッピングします。                                                          |
| **Vaults**                                               | シークレットを一元化して、{{site.ce_product_name}}のセキュリティを管理します。                                                                     |
| **Keys**                                                 | キーオブジェクトを追加して、非対称キーを管理します。                                                                            |
| **Key Sets**                                             | キーセットオブジェクトを追加して、非対称キーコレクションを管理します。                                                                   |
