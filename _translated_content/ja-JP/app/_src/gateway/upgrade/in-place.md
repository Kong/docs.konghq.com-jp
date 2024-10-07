---
title: "インプレースアップグレード"
content_type: "how-to"
purpose: "Kong Gatewayのインプレースアップグレード方法をご紹介します"
---
インプレースアップグレード方式は、主に従来モードのデプロイメントとハイブリッドモードのコントロールプレーンに使用される{{site.base_gateway}}のアップグレードオプションです。
インプレースアップグレードでは、既存のデータベースを再利用します。

デュアルクラスターアップグレードと比較して、インプレースアップグレードはより少ないリソースの使用で済みますが、ビジネスにダウンタイムが生じます。

このガイドでは、古いバージョンをクラスターX、新しいバージョンをクラスターYと呼びます。このアップグレード方式では、クラスターXを停止し、新しいクラスターYがデータベースを指すように設定する必要があります。

{% mermaid %}
flowchart TD
    DB[(Database)]
    CPX(Current {{site.base_gateway}} X \n #40;inactive#41;)
    Admin(No Admin API \n write operations)
    CPY(New {{site.base_gateway}} Y)
    API(API requests)

    CPX -.X.-> DB
    API --> CPY
    CPY --kong migrations up \n kong migrations finish--> DB
    Admin -.X.- CPX & CPY

    style API stroke:none
    style CPX stroke-dasharray:3
    style Admin fill:none,stroke:none,color:#d44324
    linkStyle 0,3,4 stroke:#d44324,color:#d44324
{% endmermaid %} 
> 
> *図1: この図は、現在のクラスターXが新しいクラスターYに直接置き換えられるインプレースアップグレードの流れを示しています。*
> *データベースは新しいクラスターYで再利用され、すべてのノードが移行されると現在のクラスターXは停止されます。アップグレード中は、Admin APIの書き込み操作は実行できません。* 

アップグレード処理中にクラスターXが停止するため、業務でのダウンタイムが発生します。
[アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations)を事前に慎重に確認する必要があります。

{:.important} 
> 
> **重要** : {{site.base_gateway}}のデプロイ状況が次の場合を除いて、この方式は推奨されません。
> 非常にリソースが制限された環境、またはデュアルクラスターアップグレードのために新しいリソースを適時に取得することができない場合
> <br><br>
> 現在のクラスターXは、新しいクラスターYに置き換えることができます。
> ただし、この方式は、新しいクラスターYを別のマシンにデプロイすることを妨げるものではありません。

前提条件
----

* [一般アップグレードガイド](/gateway/{{page.release}}/upgrade/)を確認してアップグレードの準備を行い、オプションを確認すること。
* 従来モードのデプロイメントをしている、またはハイブリッドモードのデプロイメントでコントロールプレーン \(CP\) をアップグレードする必要があること。
* リソースの制限により、[デュアルクラスターのアップグレード](/gateway/{{page.release}}/upgrade/dual-cluster/)を実行できないこと。

インプレース方式を使用したアップグレード
--------------------

{:.note} 
> 
> 次の手順は、ガイドラインとしてご利用ください。
> これらの手順通りに実行できるかどうかは、環境によって異なります。

1. {{site.base_gateway}}のあらゆる設定更新を停止します \(例: Admin API呼び出し\)。これは、クラスターXとクラスターY間のデータの一貫性を保証するために重要です。

2. [バックアップガイド](/gateway/{{page.release}}/upgrade/backup-and-restore/)に従って、現在のクラスターXのデータをバックアップします。

3. [アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations/)の説明に従って、アップグレードに影響を与える可能性のある要因を評価します。
   `kong.conf`と{{site.base_gateway}}の両方の構成データのカスタマイズを検討する必要がある場合があります。

4. リリース間で発生したすべての変更を評価します。

   * [下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)
   * [全変更履歴](/gateway/changelog/)

5. 古いクラスターXの{{site.base_gateway}}ノードを停止しますが、データベースは実行したままにします。
   これにより、アップグレードが完了するまでダウンタイムが発生します。

6. [{{site.base_gateway}}のインストールオプション](/gateway/{{page.release}}/install/)の説明に従って、バージョンYを実行して新しいクラスターをインストールし、クラスターXの既存のデータベースに向けます。

   現在のクラスターXと同じ容量のリソースを新しいクラスターYに割り当てます。
7. `kong migrations up`を実行してデータベースを新しいバージョンに移行します。

8. 完了したら、`kong migrations finish`を実行します。

9. 新しいクラスターYを起動します。

10. すべてのプロキシメトリクスを頻繁に監視します。

11. 問題が発生した場合は、[アップグレードをロールバックしてください](/gateway/{{page.release}}/upgrade/backup-and-restore/#restore-gateway-entities)。
    アプリケーションレベルの方法よりも、データベースレベルの復元方法を優先します。

12. 問題がなくなったら、古いクラスターXを廃止してアップグレードを完了してください。

これで{{site.base_gateway}}への書き込み更新を実行できるようになりましたが、しばらくはメトリクスを監視し続けることをお勧めします。

