---
title: "Kong Managerでのテーマ編集"
badge: "enterprise"
---
Kong Dev Portal には、プリセット画像、背景色、フォント、ボタン スタイルなどのデフォルトのテーマが付属しています。これらの設定は、コードを編集する必要なく、Kong Manager 内からすばやく簡単に編集できます。

前提条件
----

* Kong Managerへのアクセス
* Dev Portalが有効かつ実行中

外観タブ
----

{:.note}
> 
> **注** : スタイル設定は、デフォルトの Dev Portal から他のワークスペース内の異なる Dev Portal に自動的に引き継がれることはありません。スタイル設定を複製するワークスペースに手動でテンプレートをコピーする方法については、[ワークスペース間でのテンプレートの移行](/gateway/{{page.release}}/kong-enterprise/dev-portal/customize/migrating-templates/)を参照してください。

**Kong Manager** の **ワークスペース** ダッシュボードで、左側のサイドバーの **Dev Portal** にある **外観** タブをクリックします。これにより、Dev Portalsテーマエディターが開きます。このページから、ヘッダーロゴ、背景色、フォントの色、ボタンのスタイルをカラーピッカーインターフェイスを使用して編集できます。

事前定義された変数は、次の要素を参照します。

![外観タブ - 変数](/assets/images/products/gateway/dev-portal/appearance-dev-portal.png)

要素の上にマウスを置くと、カラーピッカーと定義済みの色のリストが表示されます。これらの色は現在のテンプレートによって定義され、 [theme.conf.yaml](#editing-theme-files-with-the-editor)ファイルで編集できます。

エディタでテーマファイルを編集
---------------

Kong Manager内のDev Portalエディタは、デフォルトのDev Portalテーマファイルを公開します。テーマファイルはファイルリストの一番下の「 *テーマ* 」にあります。「 *外観* 」タブに公開される変数は、 `theme.conf.yaml`ファイルで編集できます。エディタでファイルを編集、プレビュー、保存する方法の詳細については、 「[エディタの使用](/gateway/{{page.release}}/kong-enterprise/dev-portal/using-the-editor/)」を参照してください。

