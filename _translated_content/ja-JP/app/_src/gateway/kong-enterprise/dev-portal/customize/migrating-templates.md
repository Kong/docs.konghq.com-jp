---
title: "ワークスペース間でのテンプレートの移行"
badge: "enterprise"
---
デフォルトのDev Portalのテンプレートスタイルは、他のワークスペースで作成した新しいデベロッパーポータルには自動的に適用されません。ただし、 [Portal CLI](/gateway/latest/kong-enterprise/dev-portal/cli/)を使用すると、数ステップでテンプレートを移動できます。

前提条件
----

* [Portal CLI](/gateway/latest/kong-enterprise/dev-portal/cli/) をインストールし、[設定](/gateway/latest/kong-enterprise/dev-portal/cli/#usage)を行うこと。

デフォルトのDev Portalからのカスタマイゼーションの適用
--------------------------------

あるDev Portalから別のものにカスタマイゼーションを適用するには、以下の手順を実行します。

1. Kong Managerから新しいワークスペースを作成し、新しいDev Portalを有効にします。このドキュメントの残りの部分では、新しいワークスペース名は`{WORKSPACE_NAME}`になります。

2. Kongポータルテンプレートリポジトリをクローンします。

   ```bash
   git clone https://github.com/Kong/kong-portal-templates
   ```

3. `kong-portal-templates` に移動します。

   ```bash
   cd kong-portal-templates
   ```

4. デフォルトのワークスペースを取得します。

   ```bash
   portal fetch default
   ```

5. デフォルトのワークスペースから別のワークスペースにテンプレートをコピーします。

   ```bash
   cp workspaces/default workspaces/{WORKSPACE_NAME}
   ```

6. テンプレートを新しいワークスペースにデプロイします。

   ```bash
   portal deploy {WORKSPACE_NAME}
   ```

