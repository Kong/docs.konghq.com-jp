---
title: "ロードバランシング"
content-type: "tutorial"
book: "get-started"
chapter: 6
---
ロードバランシングは、複数のアップストリームサービス間でのAPIリクエスト
トラフィックを分散するメソッドです。ロードバランシングにより、システム全体の応答性が向上し、
個々のリソースの過負荷を防ぐことで障害を減らします。

次の例では、2つの異なるサーバー、つまりアップストリームターゲットにデプロイされたアプリケーションを使用します。

{{site.base_gateway}}は両方のサーバー間でロードバランシングする必要があるため、いずれかのサーバーが利用できなくなった場合でも、
問題を自動的に検出し、すべてのトラフィックを稼働中のサーバーにルーティングします。

[アップストリーム](/gateway/latest/key-concepts/upstreams/)
とは、{{site.base_gateway}}の背後にあるサービスアプリケーションを指し、
ここにクライアントリクエストが転送されます。{{site.base_gateway}}では、アップストリームは仮想ホスト名を表し、
ヘルスチェック、サーキットブレーク、複数の[ターゲット](/gateway/api/admin-ee/latest/#/Targets/list-target-with-upstream/)バックエンドサービスにわたる受信リクエストのロードバランシングに使用できます。

このセクションでは、先ほど作成したサービス（`example_service`）を特定のホストではなくアップストリームを指すように再構成します。
この例では、アップストリームは2つのターゲット、`httpbin.org`と`httpbun.com`を指します。通常は、ターゲットは異なるホストシステム上で実行されている同じバックエンドサービスのインスタンスになります。

以下はセットアップを示す図です。

{% mermaid %}
flowchart LR
  A("`Route 
  (/mock)`")
  B("`Service
  (example_service)`")
  C(Load balancer)
  D(httpbin.org)
  E(httpbun.com)
  
  subgraph id1 ["`**KONG GATEWAY**`"]
    A --> B --> C
  end

  subgraph id2 ["`Targets (example_upstream)`"]
    C --> D & E
  end

  style id1 rx:10,ry:10
  style id2 stroke:none
{% endmermaid %}

ロードバランシングを有効にする
---------------

このセクションでは、 `example_upstream`という名前のアップストリームを作成し、それに2つのターゲットを追加します。

### 前提条件

この章は、「 *Get Started with Kong* 」シリーズの一部です。最適な効果を得るために、シリーズを最初からご覧になることをお勧めします。

まずは、導入として [Kong の入手](/gateway/latest/get-started/)から始めましょう。ここには、ローカル {{site.base_gateway}} を実行するための前提条件と手順の一覧が含まれています。

ガイドの手順 2 の[サービスとルート](/gateway/latest/get-started/services-and-routes/)には、このシリーズで使われているモックサービスのインストール方法が記載されています。

これらの手順をまだ完了していない場合は、続行する前に完了してください。

### ロードバランシングを有効にする手順

1. **アップストリームの作成** 

   Admin APIを使用して、`example_upstream`という名前のアップストリームを作成します。

   ```sh
   curl -X POST http://localhost:8001/upstreams \
     --data name=example_upstream
   ```

2. **アップストリームターゲットの作成** 

   `example_upstream`の 2 つのターゲットを作成します。各リクエストは新しいターゲットを作成し、
   バックエンドサービス接続エンドポイントを設定します。

   ```sh
   curl -X POST http://localhost:8001/upstreams/example_upstream/targets \
     --data target='httpbun.com:80'
   curl -X POST http://localhost:8001/upstreams/example_upstream/targets \
     --data target='httpbin.org:80'
   ```

3. **サービスの更新** 

   本ガイドの「[サービスとルート](/gateway/latest/get-started/services-and-routes/)」セクションで作成した`example_service`は、明示的なホスト、`http://httpbun.com`を指していました。今度はこのサービスをアップストリームを指すように変更します。

   ```sh
   curl -X PATCH http://localhost:8001/services/example_service \
     --data host='example_upstream'
   ```

   これで、`httpbin.org` と `httpbun.com` の 2 つのターゲットを持つアップストリームと、そのアップストリームを指すサービスができました。
4. **検証** 

   ウェブブラウザまたはCLIを使ってルート
   `http://localhost:8000/mock`にアクセスして、構成したアップストリームが機能していることを確認します。
   * **Webブラウザ** ：`http://localhost:8000/mock`にアクセスし、ページを数回更新して、サイトが`httpbin`から`httpbun`に変わるのを確認します。
   * **CLI** : コマンド `curl -s http://localhost:8000/mock/headers |grep -i -A1 '"host"'` を複数回実行します。`httpbin` と `httpbun` の間でのホスト名変更がわかります。

次の手順
----

Kong 入門ガイドは完了しましたが、[{{site.base_gateway}}](/gateway/latest/) を使用するとさらに多くのことが可能になります。
以下は、{{site.base_gateway}} の詳細な機能に関するガイドです。

* [{{site.base_gateway}} による監視](/gateway/{{ page.release }}/production/monitoring/)
* [RBACによる{{site.base_gateway}}保護](/gateway/{{ page.release }}/kong-manager/auth/rbac/enable/)<span class="badge enterprise"></span>
* [{{site.base_gateway}}でワークスペースとチームを管理する](/gateway/{{ page.release }}/kong-manager/auth/workspaces-and-teams/)<span class="badge enterprise"></span>

