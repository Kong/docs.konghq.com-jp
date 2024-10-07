---
title: "ワークスペースを使用した複数の Dev Portal の実行"
badge: "enterprise"
---
Kongは、[**ワークスペース**](/gateway/{{page.release}}/admin-api/workspaces/reference)を使用したDev Portalの複数のインスタンスの実行をサポートします。これにより、各ワークスペースは、Kongの1つのインスタンス内から個別のDev Portals（個別のファイル、設定、および承認を含む）を有効にして維持できます。

Kong Manager で複数の Dev Portal を管理
--------------------------------

Kongのインスタンス内のすべてのDev Portal のスナップショットは、Kong Managerの **Dev Portals** の上部ナビゲーションタブで表示できます。

このページでは、以下について説明します。

* 指定されたワークスペースの Dev Portal が有効か無効か
* Dev Portal が有効になっていない場合にセットアップするためのリンク
* 各 Dev Portal のホームページへのリンク
* Kong Manager 内の各 Dev Portal の個別概要ページへのリンク
* 各 Dev Portal が認証されているかどうか（各カードの右上隅にあるロックアイコンで示されます）

ワークスペースのデベロッパー ポータルを有効にする
-------------------------

**デフォルト** のワークスペースと同様に、追加のワークスペースが作成された場合、関連するDev Portalは手動で有効にされるまで`disabled`になります。

これを行うには、Kong Managerで **Dev Portals** のOverviewページにある **Set up Dev Portal** ボタンをクリックするか、サイドバーからワークスペースの **Dev Portal Settings** ページに直接移動して`Dev Portal Switch`を切り替えるか、次のcURLリクエストを送信します。

```bash
curl -X PATCH http://localhost:8001/workspaces/<workspace_name id="sl-md0000000"> \
 --data "config.portal=true"
```

初期化時に、Kong は Kong の構成ファイルで定義された[**デフォルト設定**](/gateway/{{page.release}}/reference/configuration/#dev-portal)を新しい Dev Portal に入力します。
> 
> ワークスペースでDev Portalを有効にできるのは、Kongの構成でDev Portal機能が有効になっている *場合のみ* です。

Dev Portal の URL 構造を定義
----------------------

各Dev PortalのURLは初期化時に自動的に設定され、次の4つのプロパティによって決定されます。

1. `portal_gui_protocol`プロパティ
2. `portal_gui_host`プロパティ
3. `portal_gui_use_subdomains` プロパティが有効か無効か
4. ワークスペースの`name`

サブドメインが無効になっているURLの例：`http://localhost:8003/example-workspace`

サブドメインが有効になっているURLの例：`http://example-workspace.localhost:8003`

最初の3つのプロパティはKongの構成ファイルによって制御され、Kong Managerを使用して編集することはできません。

デフォルト設定の上書き
-----------

初期化時に、Dev Portal は Kong の構成ファイルで定義された[**デフォルトのポータル設定**](/gateway/{{page.release}}/reference/configuration/#dev-portal)を使用して構成されます。

{:.note}
> 
> **注意：** Dev Portal機能が[{{site.base_gateway}}で有効](/gateway/{{page.release}}/kong-enterprise/dev-portal/enable/)になっている場合にのみワークスペースのDev Portalを有効にできます。

これらの設定は、Kong ManagerのDev Portalsの **Settings** タブで手動で上書きするか、設置に直接パッチを適用することで上書きできます。

ワークスペース ファイル
------------

ワークスペースのDev Portalを初期化すると、 **デフォルトの** Dev Portalファイルのコピーが作成され、新しいDev Portalに挿入されます。これにより、カスタマイズされたDev Portalのテーマを簡単に転送でき、 **デフォルトを** 「マスターテンプレート」として機能させることができます。ただしDev Portalは、最初に有効になった後は、 **デフォルトの** Dev Portalからの変更を同期しません。

開発者アクセス
-------

Dev Portal間でアクセスが同期されません。管理者または開発者が複数のデベロッパーポータルにアクセスしたい場合は、各Dev Portalに個別にサインアップする必要があります。

