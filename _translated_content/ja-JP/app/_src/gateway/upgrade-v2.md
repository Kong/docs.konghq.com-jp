---
title: "Kong Gateway 3.x.x のアップグレード"
---
メジャー、マイナー、パッチへのアップグレード{{site.base_gateway}}
`kong migrations`コマンドを使用してリリースします。

コマンドを使用して、すべての{{site.base_gateway}}オープンソースエンティティを
{{site.base_gateway}}（Enterprise）へ移行することもできます。[{{site.ce_product_name}}から{{site.base_gateway}}への移行](/gateway/{{page.release}}/migrate-ce-to-ke/)を参照してください。

移行中に問題が発生した場合は、
[Kongサポート](https://support.konghq.com/support/s/)までお問い合わせください。

{{site.base_gateway}}リリースのアップグレードパス
-------------------

Kongは、製品のバージョン管理において、メジャーバージョン、マイナーバージョン、パッチバージョンを区別する構造化されたアプローチを採用しています。

{{page.release}}へのアップグレードは **マイナー** アップグレードです。

{:.important}
> 
> **重要** : 2\.8\.2以下のバージョンから3\.0\.xへのトラディショナルモードでのブルーグリーンマイグレーションはサポートされていません。
> 2\.8\.2 リリースには、ブルーグリーン移行のサポートが含まれています。 あなたが望むなら
> ダウンタイムなしで従来のモードへの移行を実行するには、
> 2\.8\.2 にアップグレードして[から、 {{page.release}}に移行します](#migrate-db)。

最新バージョンに直接アップグレードすることもできますが、
このドキュメントに記載されている2\.xシリーズおよび3\.xシリーズ間における下位互換性のない変更
（直近のバージョンと以前のバージョンの両方）および
[オープンソース（OSS）とEnterpriseゲートウェイの変更ログ](/gateway/changelog/)
を確認してください。{{site.base_gateway}}以降は
オープンソースの基盤上に構築されているため、OSSにおける下位互換性のない変更はすべての{{site.base_gateway}}パッケージに影響します。


{{site.base_gateway}}では、1\.xから{{page.release}}への直接アップグレードをサポートしていません。
1\.xを使用している場合は、まず2\.8\.2にアップグレードしてから3\.0\.xおよび3\.1\.xにアップグレードしてください。最後に、その状態で

{{page.release}}にアップグレードしてください。

どちらの場合でも、 {% if_version lte:3.2.x %}[アップグレードの考慮事項と重大な変更点](#upgrade-considerations-and-breaking-changes){% endif_version %}のバージョンの重大な変更点を確認できます。
次に、[データベース移行](#migrate-db)の手順に従います。

{{site.base_gateway}} {{page.release}}のアップグレードパス
-----------------------

次の表は、現在使用している{{site.base_gateway}}バージョンに応じて、{{page.release}}するさまざまなアップグレードパスシナリオを示しています。

{% if_version lte: 3.1.x %}

|  **現在のバージョン**   | **トポロジー** | **直接アップグレードの可否** |                                                                                        **アップグレードパス**                                                                                         |
|-----------------|-----------|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2\.x—2\.7\.x | 従来        | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)し（ブルーグリーンデプロイメントにのみ必要）、次に[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)します。その後、[3\.1\.xにアップグレード](#migrate-db)します。 |
| 2\.x—2\.7\.x | ハイブリッドモード | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)し、[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、その後、[3\.1\.xにアップグレード](#migrate-db)します。                          |
| 2\.x—2\.7\.x | DBレスモード   | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)してから、[3\.1\.x にアップグレードします](#migrate-db)。                                                                                                       |
| 2\.8\.x       | 従来        | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)してから、[3\.1\.x にアップグレードします](#migrate-db)。                                                                                                       |
| 2\.8\.x       | ハイブリッドモード | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)してから、その後、[3\.1\.1\.3にアップグレード](#migrate-db)します。                                                                                                |
| 2\.8\.x       | DBレスモード   | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)してから、[3\.1\.x にアップグレードします](#migrate-db)。                                                                                                       |
| 3\.0\.x       | 従来        | 可能               | [3\.1\.xにアップグレード](#migrate-db)します。                                                                                                                                                         |
| 3\.0\.x       | ハイブリッドモード | 可能               | [3\.1\.xにアップグレード](#migrate-db)します。                                                                                                                                                         |
| 3\.0\.x       | DBレスモード   | 可能               | [3\.1\.xにアップグレード](#migrate-db)します。                                                                                                                                                         |

{% endif_version %}

{% if_version eq: 3.2.x %}

|         **現在のバージョン**          | **トポロジー** | **直接アップグレードの可否** |                                                                                                                   **アップグレードパス**                                                                                                                   |
|-------------------------------|-----------|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2\.x—2\.7\.x               | 従来        | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)（ブルーグリーンデプロイメントにのみ必要）し、[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、[3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、その後、[3\.2\.xにアップグレード](#migrate-db)します。 |
| 2\.x—2\.7\.x               | ハイブリッドモード | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)し、[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、[3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、その後、[3\.2\.xにアップグレードします](#migrate-db)。                      |
| 2\.x—2\.7\.x               | DBレスモード   | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、[3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、その後、[3\.2\.xにアップグレードします](#migrate-db)。                                                                                                   |
| 2\.8\.x                     | 従来        | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 その後、 [3\.2\.x にアップグレードします](#migrate-db)。                                                                                                                                          |
| 2\.8\.x                     | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 その後、 [3\.2\.x にアップグレードします](#migrate-db)。                                                                                                                                          |
| 2\.8\.x                     | DBレスモード   | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 その後、 [3\.2\.x にアップグレードします](#migrate-db)。                                                                                                                                          |
| 3\.0\.x                     | 従来        | 不可               | [3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、その後、[3\.2\.xにアップグレード](#migrate-db)します。                                                                                                                                                 |
| 3\.0\.x                     | ハイブリッドモード | 不可               | [3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、その後、[3\.2\.xにアップグレード](#migrate-db)します。                                                                                                                                                 |
| 3\.0\.x                     | DBレスモード   | 不可               | [3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、その後、[3\.2\.xにアップグレード](#migrate-db)します。                                                                                                                                                 |
| 3\.1\.x                     | 従来        | 可能               | [3\.2\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                              |
| 3\.1\.0\.x\-3\.1\.1\.2 | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 その後、 [3\.2\.x にアップグレードします](#migrate-db)。                                                                                                                                          |
| 3\.1\.1\.3                 | ハイブリッドモード | 可能               | [3\.2\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                              |
| 3\.1\.x                     | DBレスモード   | 可能               | [3\.2\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                              |

{% endif_version %}

{% if_version eq: 3.3.x %}

|         **現在のバージョン**          | **トポロジー** | **直接アップグレードの可否** |                                                                                                                                    **アップグレードパス**                                                                                                                                    |
|-------------------------------|-----------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2\.x—2\.7\.x               | 従来        | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)（ブルーグリーンデプロイメントにのみ必要）し、[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、[3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレードします](#migrate-db)。 |
| 2\.x—2\.7\.x               | ハイブリッドモード | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)し、[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、[3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレードします](#migrate-db)。                      |
| 2\.x—2\.7\.x               | DBレスモード   | 不可               | [3\.0\.xにアップグレードし](/gateway/3.0.x/upgrade/)、 [3\.1\.x にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 その後、 [3\.3\.x にアップグレードします](#migrate-db)。                                                                                             |
| 2\.8\.x                     | 従来        | 不可               | [3\.1\.1\.3にアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレードします](#migrate-db)。                                                                                                                                             |
| 2\.8\.x                     | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレードします](#migrate-db)。                                                                                                                                             |
| 2\.8\.x                     | DBレスモード   | 不可               | [3\.1\.1\.3にアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレードします](#migrate-db)。                                                                                                                                             |
| 3\.0\.x                     | 従来        | 不可               | [3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレードし](#migrate-db)、その後、[3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                 |
| 3\.0\.x                     | ハイブリッドモード | 不可               | [3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレードし](#migrate-db)、その後、[3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                 |
| 3\.0\.x                     | DBレスモード   | 不可               | [3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレードし](#migrate-db)、その後、[3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                 |
| 3\.1\.x                     | 従来        | 不可               | [3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                          |
| 3\.1\.0\.x\-3\.1\.1\.2 | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレードします](#migrate-db)。                                                                                                                                             |
| 3\.1\.1\.3                 | ハイブリッドモード | 不可               | [3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                          |
| 3\.1\.x                     | DBレスモード   | 不可               | [3\.2\.xにアップグレード](#migrate-db)し、その後、[3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                          |
| 3\.2\.x                     | 従来        | 可能               | [3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                                                                |
| 3\.2\.x                     | ハイブリッドモード | 可能               | [3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                                                                |
| 3\.2\.x                     | DBレスモード   | 可能               | [3\.3\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                                                                |

{% endif_version %}

{% if_version eq: 3.4.x %}

|         **現在のバージョン**          | **トポロジー** | **直接アップグレードの可否** |                                                                                                                                                    **アップグレードパス**                                                                                                                                                     |
|-------------------------------|-----------|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2\.x—2\.7\.x               | 従来        | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)（ブルーグリーンデプロイメントにのみ必要）し、[3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)し、[3\.1\.xにアップグレード](/gateway/3.1.x/upgrade/#migrate-db)し、[3\.2\.xにアップグレード](#migrate-db)し、[3\.3\.xにアップグレード](#migrate-db)してから、[3\.4\.xにアップグレード](#migrate-db)します。 |
| 2\.x—2\.7\.x               | ハイブリッドモード | 不可               | [2\.8\.2\.x にアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)、 [3\.0\.x にアップグレード](/gateway/3.0.x/upgrade/)、 [3\.1\.x にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。             |
| 2\.x—2\.7\.x               | DBレスモード   | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)してから、[3\.1\.x にアップグレード](/gateway/3.1.x/upgrade/#migrate-db)してから、[3\.2\.xにアップグレード](#migrate-db)してから、[3\.3\.xにアップグレード](#migrate-db)してから、[3\.4\.xにアップグレード](#migrate-db)します。                                                                                         |
| 2\.8\.x                     | 従来        | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                       |
| 2\.8\.x                     | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                       |
| 2\.8\.x                     | DBレスモード   | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                       |
| 3\.0\.x                     | 従来        | 不可               | [3\.1\.xにアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                           |
| 3\.0\.x                     | ハイブリッドモード | 不可               | [3\.1\.xにアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                           |
| 3\.0\.x                     | DBレスモード   | 不可               | [3\.1\.xにアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                           |
| 3\.1\.x                     | 従来        | 不可               | [3\.2\.xにアップグレード](#migrate-db)し、[3\.3\.xにアップグレード](#migrate-db)してから、その後、[3\.4\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                      |
| 3\.1\.0\.x\-3\.1\.1\.2 | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 [3\.2\.xにアップグレードし](#migrate-db)、 [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                       |
| 3\.1\.1\.3                 | ハイブリッドモード | 不可               | [3\.2\.xにアップグレード](#migrate-db)し、[3\.3\.xにアップグレード](#migrate-db)してから、その後、[3\.4\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                      |
| 3\.1\.x                     | DBレスモード   | 不可               | [3\.2\.xにアップグレード](#migrate-db)し、[3\.3\.xにアップグレード](#migrate-db)してから、その後、[3\.4\.xにアップグレード](#migrate-db)します。                                                                                                                                                                                                      |
| 3\.2\.x                     | 従来        | 不可               | [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                                                                                                                        |
| 3\.2\.x                     | ハイブリッドモード | 不可               | [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                                                                                                                        |
| 3\.2\.x                     | DBレスモード   | 不可               | [3\.3\.xにアップグレードし](#migrate-db)、 その後、 [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                                                                                                                        |
| 3\.3\.x                     | 従来        | 可能               | [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                                                                                                                                                                |
| 3\.3\.x                     | ハイブリッドモード | 可能               | [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                                                                                                                                                                |
| 3\.3\.x                     | DBレスモード   | 可能               | [3\.4\.x にアップグレードします](#migrate-db)。                                                                                                                                                                                                                                                                                |

{% endif_version %}

{% if_version eq: 3.5.x %}

|         **現在のバージョン**          | **トポロジー** | **直接アップグレードの可否** |                                                                                        **アップグレードパス**                                                                                         |
|-------------------------------|-----------|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2\.x—2\.7\.x               | 従来        | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)し（ブルーグリーンデプロイメントにのみ必要）、3\.4\.xにアップグレードします。その後、3\.5\.xにアップグレードします。                                             |
| 2\.x—2\.7\.x               | ハイブリッドモード | 不可               | [2\.8\.2\.xにアップグレード](/gateway/2.8.x/install-and-run/upgrade-enterprise/)し、3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                    |
| 2\.x—2\.7\.x               | DBレスモード   | 不可               | [3\.0\.xにアップグレード](/gateway/3.0.x/upgrade/)してから、[3\.1\.xにアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、3\.2\.xにアップグレードし、3\.3\.xにアップグレードし、3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。 |
| 2\.8\.x                     | ハイブリッドモード | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 2\.8\.x                     | DBレスモード   | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.0\.x                     | 従来        | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.0\.x                     | ハイブリッドモード | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.0\.x                     | DBレスモード   | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.1\.x                     | 従来        | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.1\.0\.x\-3\.1\.1\.2 | ハイブリッドモード | 不可               | [3\.1\.1\.3にアップグレードし](/gateway/3.1.x/upgrade/#migrate-db)、 3\.2\.x にアップグレードし、 3\.3\.x にアップグレードし、 3\.4\.x にアップグレードし、 その後、3\.5\.x にアップグレードします。                                      |
| 3\.1\.1\.3                 | ハイブリッドモード | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.1\.x                     | DBレスモード   | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.2\.x                     | 従来        | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.2\.x                     | ハイブリッドモード | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.2\.x                     | DBレスモード   | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.3\.x                     | 従来        | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.3\.x                     | ハイブリッドモード | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.3\.x                     | DBレスモード   | 不可               | 3\.4\.xにアップグレードし、その後、3\.5\.xにアップグレードします。                                                                                                                                                 |
| 3\.4\.x                     | 従来        | 可能               | 3\.5\.xにアップグレードします。                                                                                                                                                                        |
| 3\.4\.x                     | ハイブリッドモード | 可能               | 3\.5\.xにアップグレードします。                                                                                                                                                                        |
| 3\.4\.x                     | DBレスモード   | 可能               | 3\.5\.xにアップグレードします。                                                                                                                                                                        |

{% endif_version %}

{% if_version lte:3.2.x %}

アップグレードに関する考慮事項と下位互換性のない変更
--------------------------

アップグレードする前に、現在のインストールに影響するこのバージョンおよび以前のバージョンの構成または下位互換性のない変更を確認してください。

デプロイ方法、使用している機能セット、カスタムプラグインなどに応じて、異なるアップグレード手順を採用する必要がある場合があります。

### プラグイン

プラグインの下位互換性のない変更については、お使いの{{site.base_gateway}}バージョンの[{{site.base_gateway}}変更履歴](/gateway/changelog/)を参照してください。

{% if_version gte:3.3.x %}

### プラグインキューイング

プラグインキューイングシステムは、{{site.base_gateway}} 3\.3\.xで改良されました。そのため、一部のプラグインパラメータが期待どおりに機能しなくなる可能性があります。次のプラグインでキューを使用する場合は、新しいパラメータを構成する必要があります。

* [HTTP Log](/hub/kong-inc/http-log/)
* [OpenTelemetry](/hub/kong-inc/opentelemetry/)
* [Datadog](/hub/kong-inc/datadog/)
* [StatsD](/hub/kong-inc/statsd/)
* [Zipkin](/hub/kong-inc/zipkin/)

プラグインキューイングの仕組みと設定できるプラグインキューイングのパラメータの詳細については、[プラグインキューイングについて](/gateway/{{page.release}}/kong-plugins/queue/)、および[プラグインキューイングのリファレンス](/gateway/{{page.release}}/kong-plugins/queue/reference/)を参照してください。

### 従来の互換ルーター

複数のパスを持つルートを優先順位が異なる複数の`atc`ルートに分割することで、`traditional_compat`ルーターモードと`traditional`モードの動作との互換性が向上しました。{{site.base_gateway}} 3\.0で新しいルーターが導入されて以来、`traditional_compat`モードでは異なる接頭辞パス長と正規表現がルートに混在している場合でも、各ルートには1つの優先順位のみが割り当てられるようになっています。これは、別々の優先順位値がルートの各パスに割り当てられるように、複数パスが`traditional`ルーターで処理される方法ではなく、動作が変更されたためです。

{% endif_version %}

{% if_version gte:3.3.x %}

### PostgreSQL 15導入後の{{site.base_gateway}}アップグレード

PostgreSQL 15は、以前のバージョンのPostgreSQLとは異なる権限をパブリックスキーマに適用します。これには、スキーマを変更するための正しい権限をKongユーザーにグラントするための追加の手順が必要です。

次の2つの方法のいずれかで、権限を付与できます。

* `ALTER DATABASE kong OWNER TO kong`を実行して、Kongのデータベース所有者をKongに割り当てます。
* Kongユーザーにパブリックスキーマを変更する権限を一時的に付与し、その後その権限を取り消します。このオプションはより制限が厳しく、以下の2つの部分からなるプロセスがあります。 
  1. ブートストラップ移行コマンドを実行する前に、 `GRANT CREATE ON SCHEMA public TO kong`を使用してスキーマを変更する権限を付与します。
  2. 移行が終わったら、`REVOKE CREATE ON SCHEMA public FROM kong`を実行してこの権限を削除してください。 {% endif_version %}

{% if_version gte:3.2.x %}

### PostgreSQL SSLバージョン更新

デフォルトのPostgreSQL SSLバージョンをTLS 1\.2に更新しました。

これにより、[`pg_ssl_version`](/gateway/latest/reference/configuration/#postgres-settings)（`kong.conf`を通じて設定）が変更されます。

* デフォルト値は`tlsv1_2`になりました。
* `pg_ssl_version` 以前はどのような文字列でも使用可能でしたが、このバージョンでは、`tlsv1_1` 、 `tlsv1_2` 、 `tlsv1_3` 、または`any` のいずれかの値が必要です。

これは、PostgreSQL 12\.x以降の設定`ssl_min_protocol_version`を反映します。そのパラメータの詳細については、[PostgreSQLドキュメント](https://postgresqlco.nf/doc/en/param/ssl_min_protocol_version/)を参照してください。

`kong.conf`のデフォルト設定を使用するには、PostgresサーバーがTLS 1\.2以降のバージョンをサポートしていることを確認するか、TLSのバージョンを自分で設定します。

`tlsv1_2`より前の TLS バージョンはすでに非推奨になっており、PostgreSQL 12\.x 以降では安全ではないと見なされます。

### Kong\-Debugヘッダーの変更

[`allow_debug_header`](/gateway/latest/reference/configuration/#allow_debug_header)構成プロパティを`kong.conf`に追加して、デバッグのために`Kong-Debug`ヘッダーを制約します。このオプションのデフォルトは`off`です。

これまで `Kong-Debug` ヘッダーを使用してデバッグ情報を提供していた場合は、`kong.conf` に `allow_debug_header: on` を設定して引き続き使用してください。

### JWTプラグイン

[JWTプラグイン](/hub/kong-inc/jwt/)は、JWTトークンの検索場所に異なるトークンがあるリクエストをすべて拒否するようになりました。

### Sessionライブラリのアップグレード

[`lua-resty-session`](https://github.com/bungle/lua-resty-session)ライブラリがv4\.0\.0にアップグレードされました。
このバージョンには、Sessionライブラリの完全なリライトが含まれています。

このアップグレードは、以下に影響します。

* [Sessionプラグイン](/hub/kong-inc/session/)
* [OpenID Connectプラグイン](/hub/kong-inc/openid-connect/)
* [SAMLプラグイン](/hub/kong-inc/saml/)
* Kong ManagerおよびDev Portalのセッションを含む、バックグラウンドでSessionまたはOpenID Connectプラグインを使用するすべてのセッション構成。

このバージョンにアップグレードすると、既存のセッションはすべて無効になります。
このバージョンでセッションが期待どおりに動作するには、すべてのノードで{{site.base_gateway}}3\.2\.xまたはそれ以降を実行する必要があります。
複数のデータプレーンが異なるバージョンを実行している場合、ユーザーが異なるDPにアクセスするたびに、同じエンドポイントでも、前のセッションは無効になります。

そのため、アップグレード中は、バージョンが混在するプロキシノードをできるだけ実行しないことをお勧めします。その間、無効なセッションによってエラーが発生し、部分的なダウンタイムが発生する可能性があります。

次のような動作が予想されます。

* **コントロールプレーンのアップグレード後** ：既存のKong ManagerとDev Portalのセッションは無効になり、すべてのユーザーが再ログインする必要があります。
* **データプレーンのアップグレード後** ：既存のプロキシセッションは無効になります。IdPが設定されている場合、ユーザーはIdPに再度ログインする必要があります。

#### Session構成パラメータの変更

セッションライブラリのアップグレードには、新しいパラメータ、変更されたパラメータ、および削除されたパラメータが含まれます。これらの機能は次のとおりです。

* `cookie_lifetime`を置き換える新しいパラメータ`idling_timeout`のデフォルト値は900です。 違う設定をしないかぎり、セッションはアイドリング状態が900秒（15分）続くと期限切れになります。
* 新しいパラメータ`absolute_timeout`のデフォルト値は86400です。 違う設定をしないかぎり、セッションは86400秒（24時間）後に期限切れになります。
* 名前が変更されたすべてのパラメータは、古い名前でも引き続き機能します。
* 削除されたパラメータは機能しなくなります。設定を壊すことはなく、セッションは機能し続けますが、構成には何も寄与しません。

既存のセッション構成は、古いパラメータで構成されたとおりに引き続き機能します。

すべてのCPノードとDPノードがアップグレードされるまで、 **パラメーターを新しいものに変更しないでください** 。

すべてのCPおよびDPノードを3\.2にアップグレードし、環境が安定していることを確認したら、
パラメータを新しい名前に変更したバージョンに更新し、予期しない動作を回避するために
セッション構成から削除されたパラメータを消去することをお勧めします。

#### Sessionプラグイン

次のパラメータと、それらが受け入れる値が変更されました。新しく受け入れられる値の詳細については、[Session プラグイン](/hub/kong-inc/session/)のドキュメントを参照してください。

|     古いパラメータ名      |      新しいパラメータ名      |
|-------------------|---------------------|
| `cookie_lifetime` | `rolling_timeout`   |
| `cookie_idletime` | `idling_timeout`    |
| `cookie_samesite` | `cookie_same_site`  |
| `cookie_httponly` | `cookie_http_only`  |
| `cookie_discard`  | `stale_ttl`         |
| `cookie_renew`    | 削除済み。置換パラメータはありません。 |

#### SAMLプラグイン

次のパラメータと、それらが受け入れる値が変更されました。新しく受け入れられる値の詳細については、[SAMLプラグイン](/hub/kong-inc/saml/)のドキュメントを参照してください。

|                古いパラメータ名                 |                新しいパラメータ名                 |
|-----------------------------------------|------------------------------------------|
| `session_cookie_lifetime`               | `session_rolling_timeout`                |
| `session_cookie_idletime`               | `session_idling_timeout`                 |
| `session_cookie_samesite`               | `session_cookie_same_site`               |
| `session_cookie_httponly`               | `session_cookie_http_only`               |
| `session_memcache_prefix`               | `session_memcached_prefix`               |
| `session_memcache_socket`               | `session_memcached_socket`               |
| `session_memcache_host`                 | `session_memcached_host`                 |
| `session_memcache_port`                 | `session_memcached_port`                 |
| `session_redis_cluster_maxredirections` | `session_redis_cluster_max_redirections` |
| `session_cookie_renew`                  | 削除済み。置換パラメータはありません。                      |
| `session_cookie_maxsize`                | 削除済み。置換パラメータはありません。                      |
| `session_strategy`                      | 削除済み。置換パラメータはありません。                      |
| `session_compressor`                    | 削除済み。置換パラメータはありません。                      |

#### OpenID Connectプラグイン

次のパラメータと、それらが受け入れる値が変更されました。新しく受け入れられる値の詳細については、 [OpenID Connect プラグイン](/hub/kong-inc/openid-connect/)のドキュメントを参照してください。

|                古いパラメータ名                 |                新しいパラメータ名                 |
|-----------------------------------------|------------------------------------------|
| `authorization_cookie_lifetime`         | `authorization_rolling_timeout`          |
| `authorization_cookie_samesite`         | `authorization_cookie_same_site`         |
| `authorization_cookie_httponly`         | `authorization_cookie_http_only`         |
| `session_cookie_lifetime`               | `session_rolling_timeout`                |
| `session_cookie_idletime`               | `session_idling_timeout`                 |
| `session_cookie_samesite`               | `session_cookie_same_site`               |
| `session_cookie_httponly`               | `session_cookie_http_only`               |
| `session_memcache_prefix`               | `session_memcached_prefix`               |
| `session_memcache_socket`               | `session_memcached_socket`               |
| `session_memcache_host`                 | `session_memcached_host`                 |
| `session_memcache_port`                 | `session_memcached_port`                 |
| `session_redis_cluster_maxredirections` | `session_redis_cluster_max_redirections` |
| `session_cookie_renew`                  | 削除済み。置換パラメータはありません。                      |
| `session_cookie_maxsize`                | 削除済み。置換パラメータはありません。                      |
| `session_strategy`                      | 削除済み。置換パラメータはありません。                      |
| `session_compressor`                    | 削除済み。置換パラメータはありません。                      |

{% endif_version %}

### KubernetesのKongに関する考慮事項

Helmチャートはアップグレード移行プロセスを自動化します。`helm upgrade` を実行すると、チャートは、`kong migrations up` を実行するための初期ジョブを生成し、更新されたバージョンで新しいKongポッドを生成します。これらのポッドの準備が整うと、トラフィックの処理が開始され、古いポッドは終了されます。これが完了すると、チャートは別のジョブを生成して `kong migrations finish` を実行します。

移行自体は自動化されていますが、チャートは、推奨されているアップグレードパスに従うことを自動的に保証するものではありません。複数のマイナー
{{site.base_gateway}}バージョンからさかのぼってアップグレードする場合は、推奨されるアップグレードパスを確認してください。

必須ではありませんが、ユーザーはチャートのバージョンと{{site.base_gateway}}バージョンを個別にアップグレードする必要があります。問題が発生した場合、これは問題がKubernetesリソースの変更に起因するものか、{{site.base_gateway}}の変更変更に起因するものかを明確にするのに役立ちます。

Kong for Kubernetesのバージョンアップグレードに関する具体的な考慮事項については、以下を参照してください。
[アップグレードの考慮事項](https://github.com/Kong/charts/blob/main/charts/kong/UPGRADE.md)

#### Kongデプロイメントは複数のリリースに分割されています

標準のチャートアップグレード自動化プロセスでは、{{site.base_gateway}}クラスタ内で{{site.base_gateway}}リリースが1つだけであると想定しており、`migrations up`と`migrations finish`の両方のジョブを実行します。

{{site.base_gateway}}デプロイメントを複数のHelmリリースに分割する場合（たとえば、プロキシ専用ノードや管理者専用ノードなど）、アップグレードの順番に応じて、どの移行ジョブを実行するかを設定する必要があります。

複数のリリースに分割されたクラスタを処理するには、次の操作を行う必要があります。

1. いずれかのリリースを次のようにアップグレードします。

   ```shell
   helm upgrade RELEASENAME -f values.yaml \
   --set migrations.preUpgrade=true \
   --set migrations.postUpgrade=false
   ```

2. 残りのリリースのうち、1つを除くすべてを次のようにアップグレードします。

   ```shell
   helm upgrade RELEASENAME -f values.yaml \
   --set migrations.preUpgrade=false \
   --set migrations.postUpgrade=false
   ```

3. 最終リリースを次のようにアップグレードします。

   ```shell
   helm upgrade RELEASENAME -f values.yaml \
   --set migrations.preUpgrade=false \
   --set migrations.postUpgrade=true
   ```

これにより、すべてのインスタンスが、`kong migrations finish`を実行する前に新しい{{site.base_gateway}}パッケージを使用していることが保証されます。

### ハイブリッドモードに関する考慮事項

{:.important}
> 
> **重要:** [現在ハイブリッドモードで実行している場合は](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/)、
> 最初にコントロールプレーンをアップグレードし、次にデータプレーンをアップグレードしてください。

* 現在、クラシック（従来モード）で以前のバージョンを実行している場合で、ハイブリッドモードの実行に切り替えたい場合は、移行を行った後、ハイブリッドモード [インストール手順](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/setup/) に従ってください。
* カスタムプラグイン（独自のプラグインまたは{{site.base_gateway}}に付属していないサードパーティ製プラグイン）は、ハイブリッドモードでコントロールプレーン（CP）とデータプレーン（DP）の両方にインストールする必要があります。プラグインは、最初にコントロールプレーン（CP）にインストールし、次にデータプレーン（DP）にインストールします。
* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)プラグインは、ハイブリッドモードでの `cluster`ストラテジをサポートしていません。代わりに、`redis`ストラテジを使用する必要があります。

### テンプレートの変更

2\.0\.x以降の{{site.base_gateway}}のマイナーバージョンとメジャーバージョンの間では、Nginx構成ファイルに変更があります。

3\.0\.xでは、非推奨の`Kong.serve_admin_api`エイリアスが削除されました。
カスタムNginxテンプレートをまだ使用している場合は、 `Kong.admin_content`に変更してください。

{% navtabs %}
{% navtab OSS %}
バージョン間のすべての構成変更を表示するには、[Kongリポジトリ](https://github.com/kong/kong)を複製し、構成テンプレート上で `git diff` を実行します。読みやすくするためには `-w` を使用します。

以前のバージョンと{{page.versions.ce}}の違いを確認する方法は次のとおりです。

    git clone https://github.com/kong/kong
    cd kong
    git diff -w 2.0.0 {{page.versions.ce}} kong/templates/nginx_kong*.lua

開始バージョン番号 \(例では 2\.0\.0\) を現在使用しているバージョン番号に調整します。

パッチファイルを生成するには、次のコマンドを使用します。

    git diff 2.0.0 {{page.versions.ce}} kong/templates/nginx_kong*.lua > kong_config_changes.diff

開始バージョン番号を現在使用しているバージョン番号（例では2\.0\.0）に調整します。

{% endnavtab %}
{% navtab Enterprise %}

{{site.base_gateway}}のデフォルトテンプレートは、{{site.base_gateway}}インスタンスを実行しているシステム上で、以下のコマンドを使用して見つけることができます：`find / -type d -name "templates" | grep kong`

アップグレードするときは、必ず古いクラスタと新しいクラスタの両方でこのコマンドを実行してください。
ファイルを比較して変更箇所を特定し、必要に応じて適用します。

{% endnavtab %}
{% endnavtabs %}

{% endif_version %}

一般的なアップグレードパス \{\#migrate\-db\}
-----------------------------------

このワークフローでの`kong migrations`の実行は取り消し不能であるため、変更を加える前にデータをバックアップすることをお勧めします。

データベース レベルで移行の失敗から回復できるように、データベース ダンプを実行することをお勧めします。

さらに、{{site.base_gateway}}は`kong config db_export`を使用したYAML形式でのデータエクスポートに対応しています。これは後で
`kong config db_import`でインポートし直すことができます。詳細については、
[kong config CLI](/gateway/latest/reference/cli/#kong-config)を参照してください。

### トラディショナルモード

1. データベースをクローンします。
2. アップグレードする{{site.base_gateway}}のバージョンをダウンロードして、クローンのデータストアを指すように構成します。 `kong migrations up`と`kong migrations finish`を実行します。
3. 新しい{{site.base_gateway}}バージョンのクラスタを起動します。
4. 今では古いものも新しいものも クラスターを同時に実行できるようになりました。 新しい{{site.base_gateway}}バージョン ノードのプロビジョニングを開始します。
5. 徐々に、古いノードから 新しいクラスタへトラフィックを移行します。トラフィックを監視し、移行がスムーズに行われているか 確認します。
6. トラフィックが新しいクラスタに完全に移行されると、古いノードを廃止します。

### ハイブリッドモード

{:.important}
> 
> Kong の設定 \( `kong.conf` \) を変更したり、Admin API を使用したりしないでください。アップグレードの途中です。これにより、データ プレーン ノード間の非互換性が発生する可能性があります。

クラスタのローリングアップデートを次の手順で実行します。

1. {{site.base_gateway}}のアップグレード先バージョンをダウンロードします。
2. 既存のコントロールプレーンを廃止します。この間、既存のデータ プレーン（DP）は、コントロールプレーン（CP）が有効 ではない状態でもプロキシトラフィックを処理し続けることができます。
3. 新しい{{site.base_gateway}}バージョンのコントロールプレーンが、古いコントロールプレーンと同じデータストアを指すように構成します。`kong migrations up`と`kong migrations finish`を実行します。
4. 新しいコントロールプレーンを開始します。
5. 新しいデータ プレーンを開始します。
6. 徐々に、古いデータプレーン（DP）から 新しいデータプレーン（DP）へトラフィックを移行します。トラフィックを監視し、移行がスムーズに行われているか 確認します。
7. トラフィックが新しいクラスタに完全に移行されると、古いデータプレーン（DP）を廃止します。

{{site.base_gateway}} 3\.x.xへのアップグレードとPrometheusの2\.x.xアラート保持
------------------------------------------------

{{site.base_gateway}} 2\.xx Prometheusアラートまたはダッシュボードを維持しつつ{{site.base_gateway}} 3\.x.xにアップグレードできます。これは、新しい{{site.base_gateway}} 3\.x.x Prometheusメトリクスで準拠するようにパッチを適用する容量がない場合に役立ちます。

{{site.base_gateway}} 3\.xxのPrometheusメトリクスを{{site.base_gateway}} 2\.xxの`kong.config`ファイル内にあるPrometheusメトリクスに変換します。

```yaml
- job_name: kong-3x-metrics-as-kong-2x
  scrape_interval: 20s
  scrape_timeout: 19s
  metrics_path: /metrics
  scheme: http
  metric_relabel_configs:
    - action: replace
      source_labels:
        - __name__
      regex: kong_http_requests_total
      target_label: __name__
      replacement: kong_http_status

    - action: replace
      source_labels:
        - __name__
      regex: kong_(.*)_latency_ms_(bucket|count|sum)
      target_label: type
      replacement: $1
    - action: replace
      source_labels:
        - __name__
      regex: kong_(.*)_latency_ms_(bucket|count|sum)
      target_label: __name__
      replacement: kong_latency_$2

    - action: replace
      source_labels:
        - __name__
        - direction
      regex: (kong_bandwidth_bytes);(egress|ingress)
      separator: ;
      target_label: type
      replacement: $2
    - action: replace
      source_labels:
        - __name__
      regex: kong_bandwidth_bytes
      target_label: __name__
      replacement: kong_bandwidth

    - action: replace
      source_labels:
        - __name__
      regex: kong_nginx_connections_total
      target_label: node_id
      replacement: ""
    - action: replace
      source_labels:
        - __name__
      regex: kong_nginx_connections_total
      target_label: subsystem
      replacement: ""
    - action: replace
      source_labels:
        - __name__
      regex: kong_nginx_connections_total
      target_label: __name__
      replacement: kong_nginx_http_current_connections
```

