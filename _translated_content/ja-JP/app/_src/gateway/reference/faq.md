---
title: "FAQ"
---
お探しのものがこのページで見つからない場合は、[team\-docs@konghq.com](mailto:team-docs@konghq.com)までご連絡ください。

### 起動時にKongがクラッシュする（MDB\_CORRUPTED）

**Q：** Kongを起動すると、次のエラーが表示されます。

    2022/04/11 12:01:07 [crit] 32790#0: *7 [lua] init.lua:648: init_worker(): worker initialization error: failed to create and open LMDB database: MDB_CORRUPTED: Located page was wrong type; this node must be restarted, context: init_worker_by_lua*

**A：** ローカルの構成キャッシュが破損しています。LMDBキャッシュ（`<prefix id="sl-md0000000">/dbless.lmdb`にありますが、デフォルトでは`/usr/local/kong/dbless.lmdb`にあります）を削除し、{{site.base_gateway}}を再起動します。その結果、破損したキャッシュは削除されるため、{{site.base_gateway}}ノードはコントロールプレーン（CP）から構成を強制的に再読み込みします。

*** ** * ** ***

