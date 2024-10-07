---
title: "Kong Gateway Enterprise バージョンのサポート"
breadcrumb: "ディストリビューション"
---
Kongは、製品のバージョン管理に構造化されたアプローチを採用しています。製品は`{MAJOR}.{MINOR}.{PATCH}.{ENTERPRISE_PATCH}`のパターンに従っています。`ENTERPRISE_PATCH`セグメントは、{{site.ce_product_name}}に基づいて{{site.ee_product_name}}のサブパッチを識別します。

{:.note}
> 
> **注** : このポリシーは{{site.base_gateway}}と{{site.ee_product_name}}に **のみ** 適用されます。{{site.konnect_product_name}}バージョンのサポートポリシーについては、[{{site.konnect_product_name}}ポリシー](/konnect/compatibility/#kong-gateway-version-compatibility)を確認してください。

バージョン管理
-------

このサポートドキュメントでは、次のことを行います。

**リリースのバージョン管理** 

* **メジャーバージョン** とは、小数点の左端の数字（X.y.z.a）で識別されるバージョンを意味します。たとえば、2\.1\.3\.0はメジャーバージョン2を示し、1\.3\.0\.4はメジャーバージョン1を示します。

* **マイナーバージョン** とは、左端の小数点2桁の間の数字（x.Y.z.a）の変更によって識別されるバージョンを意味します。たとえば、2\.1\.3\.0はマイナーバージョン1を示し、1\.3\.0\.4はマイナーバージョン3を示します。

**パッチ** 

* **コミュニティエディションパッチとは** 、右端の小数点（x.y.Z.a）の左側の数字が変化することで識別されるパッチのことです。たとえば、コミュニティ/OSSバージョン2\.8\.3は、リリースバージョン2\.8、パッチ番号3を示し、3\.3\.0はリリースバージョン3\.0、パッチ番号0を示します。

* **Enterpriseパッチバージョン** とは、小数点の右端の数字（x.y.z.A）の変更によって識別されるバージョンを意味します。たとえば、2\.8\.3\.0は、パッチバージョン3\.0のEnterpriseリリースバージョン2\.8を示し、3\.3\.2\.1は、パッチ番号2\.1のEnterpriseリリースバージョン3\.3を示します。

Kongは、新たなメジャーバージョンをリリースすることで、主要な機能と重大な変更を導入します。メジャーバージョンのリリースはまれであり、メジャーバージョンのリリースは、通常、業界の大きなトレンドの変化、アーキテクチャの大幅な変更、社内の製品の大幅な革新に対応するものです。メジャーバージョンの定期的なリリースサイクルはありません。

Kongは、約12週間ごとに新たなマイナーバージョンをリリースすることを目指しています。マイナーバージョンには、機能およびバグの修正が含まれています。マイナーバージョンは通常、そのメジャーバージョンシーケンス内で後方互換性があります¹。すべてのマイナーバージョンは、リリース日から1年間サポートされます。これは、サポートされている各マイナーバージョンに適用されるパッチのリリースにより行われます。お客様は、インストールしたバージョンに適したパッチを適用して、インストールを最新の状態に保つことをお勧めします。Kongによってリリースされるすべてのパッチはロールアップパッチです（たとえば、リリースバージョン3\.3のパッチ1\.5には、パッチ1\.4、1\.3、1\.2、1\.1のすべての修正が含まれています）。

{:.note}
> 
> ¹ **注：** バージョン管理モデルには例外があります。
> バックポートのおかげで、パッチバージョンを含め、どのバージョンレベルでも新機能や下位互換性のない変更が可能です。
> 問題を回避するために、新しいバージョンへ自動アップグレードしないでください。
> また、デプロイメントを手動でアップグレードする前に、関連するすべての[変更ログエントリ](/gateway/changelog/)を確認してください。

### 長期サポート

Kongは、特定のマイナーバージョンを **長期サポート（LTS）** バージョンとして指定する場合があります。Kongは、特定のディストリビューションのLTSバージョンに対して、ディストリビューションのライフサイクル期間中、またはLTSバージョンのリリースから3年間のいずれか早い方の期間、テクニカルサポートを提供します。LTSバージョンは、メジャーバージョンシーケンス内で下位互換性があります。LTSバージョンではすべてのセキュリティ修正が適用されます。さらに、LTSバージョンでは、Kongの裁量により、特定の非セキュリティパッチが適用される場合があります。常に、少なくとも1つはアクティブなLTS {{site.ee_product_name}}バージョンが存在します。

### 提供終了サポート

製品のサポート期間が終了した後、Kongは、最長12か月の追加の提供終了期間まで、完全にサポートされているバージョンの{{site.ee_product_name}}にお客様がアップグレードできるように、限定的なサポートを提供します。Kongは、この提供終了期間の対象となるソフトウェアに対してはパッチを提供しません。この期間中にパッチを必要とする問題が発生した場合、アクティブサポートの対象となる新しい{{site.ee_product_name}}バージョンにアップグレードする必要があります。

{% include_cached /md/support-policy.md %}

サポートされているバージョン
--------------

Kongは、次のバージョンの {{site.ee_product_name}}をサポートしています。

{% navtabs %}
{% if_version gte: 3.8.x %}
{% navtab 3.8 %}
{% include_cached gateway-support.html version="3.8" data=site.data.tables.support.gateway.versions.38 eol="TBA" %}
{% endnavtab %}
{% endif_version %}
{% if_version gte: 3.7.x %}
{% navtab 3.7 %}
{% include_cached gateway-support.html version="3.7" data=site.data.tables.support.gateway.versions.37 eol="May 2025" %}
{% endnavtab %}
{% endif_version %}
{% if_version gte: 3.6.x %}
{% navtab 3.6 %}
{% include_cached gateway-support.html version="3.6" data=site.data.tables.support.gateway.versions.36 eol="Feb 2025" %}
{% endnavtab %}
{% endif_version %}
{% navtab 3.5 %}
{% include_cached gateway-support.html version="3.5" data=site.data.tables.support.gateway.versions.35 eol="Nov 2024" %}
{% endnavtab %}
{% navtab 3.4 LTS %}
{% include_cached gateway-support.html version="3.4" data=site.data.tables.support.gateway.versions.34 eol="August 2026" %}
{% endnavtab %}
{% navtab 2.8 LTS %}
{% include_cached gateway-support.html version="2.8 LTS" data=site.data.tables.support.gateway.versions.28  eol="March 2025" %}
{% endnavtab %}
{% endnavtabs %}

### マーケットプレイス


{{site.ee_product_name}} は、次のマーケットプレイスから購入できます。

* Azure Marketplace
* AWS Marketplace
* Google Cloud Marketplace

最新バージョンは保証されません。

### サポートされているパブリッククラウド導入プラットフォーム


{{site.ee_product_name}}は、次のパブリッククラウドデプロイメントプラットフォームをサポートしています。

* AWS EKS
* AWS EKS Fargate
* AWS ECS
* AWS ECS Fargate
* Azure AKS
* Azure Container Instances
* Azure Container Apps
* Google Cloud GKE
* Google Cloud GKE Autopilot
* Google Cloud Run

旧バージョン
------

これらのバージョンのフルサポートは終了しました。

|    バージョン    |     リリース日      |    フルサポート終了    |  提供終了サポートの終了   |
|:-----------:|:--------------:|:--------------:|:--------------:|
| 3\.3\.x.x | 2023\-05\-19 | 2024\-05\-19 | 2025\-05\-19 |
| 3\.2\.x.x | 2023\-02\-28 | 2024\-02\-28 | 2025\-02\-28 |
| 3\.1\.x.x | 2022\-12\-06 | 2023\-12\-06 | 2024\-12\-06 |
| 3\.0\.x.x | 2022\-09\-09 | 2023\-09\-09 | 2024\-09\-09 |
| 2\.7\.x.x | 2021\-12\-16 | 2023\-02\-24 | 2024\-08\-24 |
| 2\.6\.x.x | 2021\-10\-14 | 2023\-02\-24 | 2024\-08\-24 |
| 2\.5\.x.x | 2021\-08\-03 | 2022\-08\-24 | 2023\-08\-24 |
| 2\.4\.x.x | 2021\-05\-18 | 2022\-08\-24 | 2023\-08\-24 |
| 2\.3\.x.x | 2021\-02\-11 | 2022\-08\-24 | 2023\-08\-24 |
| 2\.2\.x.x | 2020\-11\-17 | 2022\-08\-24 | 2023\-08\-24 |
| 2\.1\.x.x | 2020\-08\-25 | 2022\-08\-24 | 2023\-08\-24 |
| 1\.5\.x.x | 2020\-04\-10 | 2021\-04\-09 | 2022\-04\-09 |
| 1\.3\.x.x | 2019\-11\-05 | 2020\-11\-04 | 2021\-11\-04 |
|   0\.36    | 2019\-08\-05 | 2020\-08\-04 | 2021\-08\-04 |
|   0\.35    | 2019\-05\-16 | 2020\-05\-15 | 2020\-11\-15 |
|   0\.34    | 2018\-11\-19 | 2019\-11\-18 | 2020\-11\-18 |
|   0\.33    | 2018\-07\-11 | 2019\-06\-10 | 2020\-06\-10 |
|   0\.32    | 2018\-05\-22 | 2019\-05\-21 | 2020\-05\-21 |
|   0\.31    | 2018\-03\-13 | 2019\-03\-12 | 2020\-03\-12 |
|   0\.30    | 2018\-01\-22 | 2019\-01\-21 | 2020\-01\-21 |

{:.note}
> 
> **注** : このポリシーは{{site.base_gateway}}と{{site.ee_product_name}}に **のみ** 適用されます。{{site.konnect_product_name}}バージョンのサポートポリシーについては、[{{site.konnect_product_name}}ポリシー](/konnect/compatibility/#kong-gateway-version-compatibility)を確認してください。

関連項目
----

* [{{site.mesh_product_name}} のバージョンサポートポリシー](/mesh/latest/support-policy/)
* [{{site.kic_product_name}} のバージョンサポートポリシー](/kubernetes-ingress-controller/latest/support-policy/)

