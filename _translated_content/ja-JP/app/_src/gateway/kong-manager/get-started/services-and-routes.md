---
title: "サービスとルート"
---
このチュートリアルでは、Kong Managerでサービスとルートを作成する方法について説明します。

Admin API を使用する場合は、 [{{site.base_gateway}} スタートガイド](/gateway/latest/get-started/configure-services-and-routes/)を確認してください。

前提条件
----

Kong Manager が[有効になっている](/gateway/{{page.release}}/kong-manager/enable/) {{site.base_gateway}} インスタンスが必要です。

サービスを追加する
---------

このチュートリアルでは、httpbin API を指すサービスを作成します。Httpbin は、リクエストをリクエスト元にレスポンスとして返す「エコー」タイプの公開ウェブサイトです。

Kong Manager の \[ワークスペース\] タブで以下を行います。

1. **デフォルト** のワークスペースを開きます。

   この例では、デフォルトのワークスペースを使用していますが、新しいワークスペースを作成するか、既存のワークスペースを使用できます。
2. **サービス** セクションで、 **新規サービス** をクリックします。

3. **サービスを作成** ダイアログで、名前`example_service`とURL`http://httpbin.org`を入力します。

4. **作成** をクリックします。

サービスが作成され、ページは自動的に
`example_service` 概要ページにリダイレクトされます。

ルートの追加
------

APIゲートウェイ経由でサービスをアクセス可能にするには、サービスへのルートを追加する必要があります。

1. `example_service`の概要ページで、サブメニューから **ルート** を開き、
   **新規ルート** をクリックします。

2. **ルートの作成** ページでは、 **サービス** フィールドにサービス名とID番号が自動入力されます。このフィールドは必須です。

   サービスフィールドが自動的に入力されない場合は、左側のナビゲーションペインで
   **サービス** クリックします。サービスを探して、
   IDフィールドの横にあるクリップボードアイコンをクリックして、 **ルートの作成**
   ページに戻り、 **サービスフィールド** に貼り付けます。
3. ルートの名前と、ホスト、メソッド、パスのうち、少なくとも1つのフィールドを入力します。この例では、以下を使用します。

   1. **Name** には、 `mocking`と入力します。
   2. **パス** で、 **パスを追加** をクリックして `/mock` と入力します。

4. **作成** をクリックします。

Kongは自動的に`example_service`概要ページにリダイレクトします。新しいルートが「ルート」セクションの下に表示されます。

ルートがリクエストをサービスに転送していることを確認
--------------------------

デフォルトでは、{{site.base_gateway}} はポート `8000` でプロキシリクエストを処理します。

ウェブブラウザで、`http://localhost:8000/mock/anything`に移動します。

次の手順
----

次に、Kong Managerを通じて[サービスに流量制限を適用する方法](/gateway/{{page.release}}/kong-manager/get-started/rate-limiting/)について学習します。

