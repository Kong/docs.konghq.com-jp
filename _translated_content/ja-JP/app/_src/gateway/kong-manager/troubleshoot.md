---
title: "Kong Managerのトラブルシューティング"
content_type: "reference"
---
Kong Manager の URL が解決されません
---------------------------

**問題：** 

{{site.base_gateway}}をインストールし、実行していますが、Kong Managerにアクセスできません。おそらく、インストール中にポートが公開されなかったのが原因と考えられます。

**解決策：** 

新しいインスタンスをインストールし、インストール中にポート `8002` をマップします。
以下は、 [Docker をインストールする](/gateway/{{page.release}}/install/docker/?install=oss)場合です。

     -p 127.0.0.1:8002:8002

