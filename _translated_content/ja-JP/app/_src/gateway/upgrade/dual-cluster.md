---
title: "デュアルクラスターのアップグレード"
content_type: "how-to"
purpose: "Kong Gatewayのデュアルクラスターをアップグレードする方法をご紹介します"
---
デュアルクラスターアップグレード方式は、主に従来モードのデプロイメントとハイブリッドモードのコントロールプレーンで使用される{{site.base_gateway}}のアップグレードオプションです。

このガイドでは、古いバージョンをクラスターX、新しいバージョンをクラスターYと呼びます。

デュアルクラスターアップグレードの場合、現在のバージョンXと一緒にバージョンYの新しいクラスターを展開できるため、アップグレード処理中に2つのクラスターが同時にリクエストを処理します。
2つのクラスター間のトラフィック比率を徐々に調整して、ビジネス指標に基づいてトラフィックを古いクラスターから新しいクラスターにに切り替えていきます。

{% mermaid %}
flowchart TD
    DBX[(Current
    database)]
    DBY[(New 
    database)]
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
    CPX -.-> DBX
    CPY --pg_restore--> DBY

    style API stroke:none
    style DBX stroke-dasharray:3
    style CPX stroke-dasharray:3
    style Admin fill:none,stroke:none,color:#d44324
    style Admin2 fill:none,stroke:none,color:#d44324
    linkStyle 4,7 stroke:#d44324,color:#d44324
    linkStyle 3,6,9 stroke:#b6d7a8
{% endmermaid %} 
> 
> *図1: この図は、デュアルクラスター方式を使用した{{site.base_gateway}}のアップグレードの流れを示しています。*
> *新しい{{site.base_gateway}}のクラスターYは、現在の{{site.base_gateway}}のクラスターXと一緒に展開されます。*
> *新しいデータベースは新しいデプロイメントに対応します。*
> *すべてのAPIトラフィックが移行されるまで、トラフィックは徐々に新しいデプロイメントに切り替わります。* 

このアップグレード方式は、利用可能なすべての方式の中で最も安全であり、アップグレード処理中に業務停止期間を予定する必要がありません。

この方法では、データベースに依存する自動生成されたランタイムメトリクスに制限があります。
アップグレード中に、一部のランタイムメトリクス \(リクエスト数など\) が2つのデータベースに別々に送信されます。
データベース間のメトリクスは同期されないため、アップグレード中はメトリクスが正確ではありません。

たとえば、Rate Limiting Advanced \(RLA\) プラグインがデータベース中のリクエストカウンタを保存するように設定されている場合、データベースXとデータベースY間のカウンタは同期されていません。
影響範囲は、プラグインの`window_size`パラメータとアップグレード処理の期間によって異なります。

同様に、PostgreSQLまたはカサンドラにバッファリングされたメトリクスが大量にある場合は、同じ制限がVitalsにも適用されます。

前提条件
----

* [一般アップグレードガイド](/gateway/{{page.release}}/upgrade/)を確認してアップグレードの準備を行い、オプションを確認すること。
* 従来型の展開をしている、またはハイブリッドモードのデプロイメントでコントロールプレーン \(CP\) をアップグレードする必要があること。
* 既存のクラスターと並行して追加の{{site.base_gateway}}のクラスターを一時的に実行するのに十分なリソースがあること。

デュアルクラスター方式を使用したアップグレード
-----------------------

{:.note} 
> 
> 次の手順は、ガイドラインとしてご利用ください。
> これらの手順通りに実行できるかどうかは、環境によって異なります。

1. {{site.base_gateway}}のあらゆる設定更新を停止します \(例: Admin API呼び出し\)。これは、クラスターXとクラスターY間のデータの一貫性を保証するために重要です。

   2つのクラスター間でデータの一貫性を保つために、[Admin API](/gateway/{{page.release}}/admin-api/)、[Kong Manager](/gateway/{{page.release}}/kong-manager/)、[decK](/deck/)を介して、または直接データベースの更新を介してあらゆる書き込み操作の実行は中止する必要があります。
2. [バックアップガイド](/gateway/{{page.release}}/upgrade/backup-and-restore/)に従って、現在のクラスターXのデータをバックアップします。

3. [アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations/)の説明に従って、アップグレードに影響を与える可能性のある要因を評価します。
   `kong.conf`と{{site.base_gateway}}の両方の構成データのカスタマイズを検討する必要がある場合があります。

4. リリース間で発生したすべての変更を評価します。

   * [下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)
   * [全変更履歴](/gateway/changelog/)

5. バージョンYの新しい{{site.base_gateway}}のクラスターを展開します。

   1. [{{site.base_gateway}}のインストールオプション](/gateway/{{page.release}}/install/)の説明に従って、バージョンYを実行する新しい{{site.base_gateway}}のクラスターをインストールします。

      現在のクラスターXと同じ容量のリソースを新しいクラスターYに割り当てます。
   2. 同じバージョンの新しいデータベースをインストールします。

   3. 新しいデータベースに[バックアップデータを復元します](/gateway/{{page.release}}/upgrade/backup-and-restore/#restore-gateway-entities)。

   4. 新しいデータベースを指すように新しいクラスターYを構成します。

   5. クラスターYを起動します。

   6. バージョンYに対してステージングテストを実行し、すべてのユースケースで機能することを確認します。

      たとえば、Key Authenticationプラグインはリクエストを適切に認証するかを確認します。

      結果が期待通りでない場合は、[アップグレードに際しての考慮事項](/gateway/{{page.release}}/upgrade/#preparation-upgrade-considerations/)や[下位互換性のない変更](/gateway/{{page.release}}/breaking-changes/)にもう一度目を通してください。

6. 古いクラスターXから新しいクラスターYにトラフィックを転送します。

   これは通常、デプロイメントのリスクプロファイルに応じて、徐々に段階的に実行されます。
   DNS、Nginx、Kubernetesのリリース構造など、トラフィック分割をサポートするロードバランサーならいずれもここで機能します。
7. すべてのプロキシメトリクスを頻繁に監視します。

8. 問題が発生した場合は、すべてのトラフィックをクラスターXに設定してロールバックし、問題を調査して上記の手順を繰り返してください。

9. 問題がなくなったら、古いクラスターXを廃止してアップグレードを完了してください。

これで{{site.base_gateway}}への書き込み更新を実行できるようになりましたが、しばらくはメトリクスを監視し続けることをお勧めします。

