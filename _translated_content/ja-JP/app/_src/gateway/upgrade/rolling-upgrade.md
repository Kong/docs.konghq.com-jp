---
title: "ローリングアップグレード"
content_type: "how-to"
purpose: "Kong Gatewayのローリングアップグレード方法をご紹介します"
---
ローリングアップグレード方式は、DB\-less型、ハイブリッド型で動作するデータプレーン（DP）、{{site.konnect_short_name}}向けに特別に設計された{{site.base_gateway}}のアップグレードオプションです。
この方式では、データベースを使用せず、互いに独立しているノードを対象としています。

このガイドでは、古いバージョンをクラスターX、新しいバージョンをクラスターYと呼びます。

ローリングアップグレードはバージョンXのノードを順次停止しながら、同時に新しいバージョンYのノードを追加していきます

{% mermaid %}
flowchart TD
    yml[fa:fa-file kong.yml]
    CPX(Current 
    Node X)
    CPX2(Current 
    Node X)
    CPX3(Current 
    Node X)
    CPY(New 
    Node Y)
    CPY2(New 
    Node Y)
    CPY3(New 
    Node Y)
    LB(Load balancer)
    API(API requests)

    API --> LB & LB & LB & LB
    LB -.-> CPX
    LB -.90%.-> CPX2
    LB -.-> CPX3
    LB --> CPY
    LB --10%--> CPY2
    LB --> CPY3
    CPX -.- yml
    CPX2 -.- yml
    CPX3 -.- yml
    CPY -.- yml
    CPY2 -.- yml
    CPY3 -.- yml

    style API stroke:none
    style CPX stroke-dasharray:3
    style CPX2 stroke-dasharray:3
    style CPX3 stroke-dasharray:3
    linkStyle 3,7,8,9,13,14,15 stroke:#b6d7a8
{% endmermaid %} 
> 
> *図1: この図は、データベースなしでローリング方式を使用した{{site.base_gateway}}のアップグレードの流れを示しています。*
> *新しいノードは徐々に展開され、`kong.yml`のファイルに指定され、トラフィックは徐々に新しいノードにルート変更されます。* 

前提条件
----

* [一般アップグレードガイド](/gateway/{{page.release}}/upgrade/)を確認してアップグレードの準備を行い、オプションを確認すること。
* DB\-lessデプロイメントをしているか、ハイブリッドモードのデプロイメントでデータプレーン \(DP\) または{{site.konnect_short_name}}のDPをアップグレードする必要があること。

ローリングアップグレード方式
--------------

{:.note} 
> 
> 次の手順は、ガイドラインとしてご利用ください。
> これらの手順通りに実行できるかどうかは、環境によって異なります。

1. あらゆる{{site.base_gateway}}構成の更新 \(例: `:8001/config`へのAdmin API 呼び出し\) を停止します。
   これは、クラスターXとクラスターY間のデータの一貫性を保証するために重要です。

2. 以下の[宣言型構成のバックアップ手順](/gateway/{{page.release}}/upgrade/backup-and-restore/#declarative-config-backup)に従って、現在のクラスターXのデータをバックアップします。

3. [アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations/)の説明に従って、アップグレードに影響を与える可能性のある要因を評価します。
   `kong.conf`と{{site.base_gateway}}の両方の構成データのカスタマイズを検討する必要がある場合があります。

4. リリース間で発生したすべての変更を評価します。

   * [下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)
   * [全変更履歴](/gateway/changelog/)

5. バージョンYの新しい{{site.base_gateway}}のクラスターをデプロイします。

   1. [{{site.base_gateway}}のインストールオプション](/gateway/{{page.release}}/install/)の説明に従って、バージョンYを実行する新しいクラスターをインストールします。

      現在のクラスターXと同じ容量のリソースを新しいクラスターYに割り当てます。
   2. バージョンYに対してステージングテストを実行し、すべてのユースケースで機能することを確認します。

      たとえば、Key Authenticationプラグインはリクエストを適切に認証出来るかを確認します。

      データプレーン（DP）ノードの場合は、コントロールプレーン（CP）ノードとの通信が成功することを確認します。

      結果が期待通りでない場合は、[アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations/)や[下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)にもう一度目を通してください。
   3. 継続的にYノードをインストールして起動します。

6. 古いクラスターXから新しいクラスターYにトラフィックを転送します。

   これは通常、デプロイメントのリスクプロファイルに応じて、徐々に段階的に実行されます。
   DNS、Nginx、Kubernetesのリリース構造など、トラフィック分割をサポートするロードバランサーならいずれもここで機能します。
7. 問題が発生した場合は、すべてのトラフィックをクラスターXに転送するようロールバックし、問題を調査して上記の手順を繰り返してください。

8. 問題がなくなったら、古いクラスターXを廃止してアップグレードを完了してください。

これで{{site.base_gateway}}への書き込み更新を実行できるようになりましたが、しばらくはメトリクスを監視し続けることをお勧めします。

