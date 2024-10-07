---
title: "Kong Managerでのロードバランシング"
---
このチュートリアルでは、Kong Managerでターゲット間のロードバランシングを設定する手順を説明します。

Admin APIの使用手順については、 [{{site.base_gateway}}スタートガイド](/gateway/latest/get-started/load-balancing/)をご覧ください。

前提条件
----

Kong Manager が[有効になっている](/gateway/{{page.release}}/kong-manager/enable/) {{site.base_gateway}} インスタンスが必要です。

アップストリームとターゲットの設定
-----------------

このチュートリアルでは、 `example_upstream`という名前のアップストリームを作成し、それに2つのターゲットを追加します。

Kong Managerの **ワークスペース** タブから以下を行います。

1. **デフォルトの** ワークスペースを開きます。
2. メニューから **アップストリーム** を開き、 **新規アップストリーム** をクリックします。
3. この例では、 **名前** フィールドに`example_upstream`と入力し、 **作成** をクリックします。
4. 新しいアップストリームをクリックして詳細ページを開きます。
5. サブメニューから **ターゲット** を開き、 **新規ターゲット** をクリックします。
6. ターゲットフィールドに値`httpbin.org:80`を設定し、 **作成** をクリックします。
7. 今度は `httpbun.com:80` のターゲットを作成します。
8. **サービス** ページを開きます。
9. `example_service`を開き、 **\[編集\]** をクリックします。
10. **ホスト** フィールドを`example_upstream`に変更し、 **更新** をクリックします。

これで、`httpbin.org`と`httpbun.com`の2つのターゲットを持つアップストリームと、そのアップストリームを指すサービスが作成されました。

アップストリームサービスを検証
---------------

{{site.base_gateway}}が2つのターゲット間でトラフィックをロードバランシングしていることをテストする方法は以下のとおりです。

1. アップストリームを設定したら、ウェブブラウザまたはシェルを使ってルート`http://localhost:8000/mock`にアクセスして、アップストリームが機能していることを確認します。

2. ページを数回更新します。サイトは`httpbin`から`httpbun`に切り替わるはずです。

次の手順
----

次に、Kong Managerで他に何ができるかについてのガイドを参照してください。

* [Kong Managerの認証を設定する](/gateway/{{page.release}}/kong-manager/auth/)
* [ロールベースのアクセス制御（RBAC）を使用してワークスペースとチームを管理する](/gateway/{{page.release}}/kong-manager/auth/workspaces-and-teams/)
* [カスタムワークスペースを作成する](/gateway/{{page.release}}/kong-manager/workspaces/)

