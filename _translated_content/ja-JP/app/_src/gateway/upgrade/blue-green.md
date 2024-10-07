---
title: "ブルーグリーンアップグレード"
content_type: "how-to"
purpose: "Kong Gatewayのブルーグリーンアップグレード方法をご紹介します"
---
ブルーグリーンアップグレード方式は、主に従来モードのデプロイメントとハイブリッドモードのコントロールプレーンに使用される{{site.base_gateway}}のアップグレードオプションです。

このガイドでは、古いバージョンをクラスターX、新しいバージョンをクラスターYと呼びます。

ブルーグリーンアップグレードは、インプレースアップグレード方式から派生した方法です。
このアップグレード方式の利点は、`kong migrations up`が現在のクラスターXまたは新しいクラスターYのいずれかによるリクエストを処理できる状態にデータベースを残すという点です。
データベースとクラスターXの互換性は、`kong migrations finish`が実行された場合にのみ失われます。

これは、新しいデータベースをデプロイする必要がないという点で、デュアルクラスターアップグレードよりも高度な方式です。
次の図に示すように、現在のクラスターXから新しいクラスターYへのトラフィックの段階的な迂回は引き続きサポートされています。
さらに、ランタイムメトリクス \(たとえば、Rate Limiting Advancedプラグインカウンタ\) は同じデータベースに送信されます。
アップグレード処理中、両方のクラスターからメトリクスが継続的に収集されます。

{% mermaid %}
flowchart TD
    DB[(Database)]
    CPX(Current 
    {{site.base_gateway}} X)
    Admin(No admin 
    write operations)
    Admin2(No admin 
    write operations)
    CPY(New 
    {{site.base_gateway}} Y)
    LB(Load balancer)
    API(API requests)

    API --> LB & LB & LB & LB
    Admin2 -."X".- CPX
    LB -.90%.-> CPX
    LB --10%--> CPY
    Admin -."X".- CPY
    CPX -.-> DB
    CPY --"kong migrations up \n (NO kong migrations finish)"--> DB

    style API stroke:none
    style CPX stroke-dasharray:3
    style Admin fill:none,stroke:none,color:#d44324
    style Admin2 fill:none,stroke:none,color:#d44324
    linkStyle 4,7 stroke:#d44324,color:#d44324
    linkStyle 3,6,9 stroke:#b6d7a8
{% endmermaid %} 
> 
> *図1: この図は、ブルーグリーン方式を使用しての{{site.base_gateway}}のアップグレードの流れを示しています。*
> *新しい{{site.base_gateway}}のクラスターYは、現在の{{site.base_gateway}}のクラスターXと一緒に展開されます。どちらのクラスターも同じデータベースを使用します。*
> *すべてのAPIトラフィックが移行されるまで、トラフィックは徐々に新しいデプロイメントに切り替わります。* 

デュアルクラスターやインプレースアップグレードと比較して、ブルーグリーンアップグレードは追加のデータベースを必要としないため、消費するリソースは少なくて済み、ビジネスのダウンタイムもありません。

{:.important} 
> 
> **重要** : この方式を使用したアップグレードに対するKongからのサポートは限られています。
> ブルーグリーンアップグレードはサポートされていますが、{{site.base_gateway}}バージョンの数、アップグレード方式、採用された機能、デプロイメントモードを考慮してすべての組み合わせを網羅する必要があるため、すべての移行テストを完全に網羅することはほぼ不可能です。
> どうしてもこの方式を採用したい場合は、パッチバージョン間のアップグレードにのみ使用してください。
> <br><br>
> 従来型では、ブルーグリーンアップグレードは、2\.8\.2\.x以降で利用できます。
> 2\.8\.2\.xより前のバージョンの{{site.base_gateway}}をお持ちの場合は、3\.xシリーズへのアップグレードを開始する前に、少なくとも2\.8\.2\.0にアップグレードしてください。

前提条件
----

* [一般アップグレードガイド](/gateway/{{page.release}}/upgrade/)を確認してアップグレードの準備を行い、オプションを確認すること。
* 従来型の展開をしている、またはハイブリッドモードのデプロイメントでコントロールプレーン \(CP\) をアップグレードする必要があること。
* {{site.base_gateway}}の2\.8\.2\.x以降を使用していること。
* リソースの制限により、[デュアルクラスターのアップグレード](/gateway/{{page.release}}/upgrade/dual-cluster/)を実行できないこと。

ブルーグリーン方式でアップグレード
-----------------

次の手順では、`kong migrations finish`は新しいクラスターYが期待どおりに動作していることを確認後のアップグレードの最後にのみ実行されます。

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

5. バージョンYの新しい{{site.base_gateway}}のクラスターを展開します。

   1. [{{site.base_gateway}}のインストールオプション](/gateway/{{page.release}}/install/)の説明に従って、バージョンYを実行する新しい{{site.base_gateway}}のクラスターをインストールし、クラスターXの既存のデータベースに向けます。

      現在のクラスターXと同じ容量のリソースを新しいクラスターYに割り当てます。
   2. `kong migrations up`を実行してデータベースを新しいバージョンに移行します。

      
        {{site.base_gateway}}は、保留中の移行が存在することを示す警告ログエントリを出力します。
      これは想定どおりの流れです。
   3. 新しいクラスターYを起動します。

   4. バージョンYに対してステージングテストを実行し、すべてのユースケースで機能することを確認します。

      たとえば、Key Authenticationプラグインはリクエストを適切に認証するかを確認します。

      結果が期待通りでない場合は、[アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations/)や[下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)にもう一度目を通してください。

6. 古いクラスターXから新しいクラスターYにトラフィックを転送します。

   これは通常、デプロイメントのリスクプロファイルに応じて、徐々に段階的に実行されます。
   DNS、Nginx、Kubernetesのリリース構造など、トラフィック分割をサポートするロードバランサーならいずれもここで機能します。
7. すべてのプロキシメトリクスをアクティブに監視します。

8. 問題が発生した場合は、すべてのトラフィックをクラスターXに設定してロールバックし、問題を調査して上記の手順を繰り返してください。

9. `kong migrations finish`でデータベースの移行を完了します。

10. 問題がなくなったら、古いクラスターXを廃止してアップグレードを完了してください。

これで{{site.base_gateway}}への書き込み更新を実行できるようになりましたが、しばらくはメトリクスを監視し続けることをお勧めします。

{:.note} 
> 
> **注** : このアップグレード方式は、[ブルーグリーン \(カナリア\) デプロイメント](/gateway/{{page.release}}/production/canary/)とは異なります。
> この処理は上流サービスのアップグレードを目的とし、{{site.base_gateway}}のアップグレードとは関係ありません。

