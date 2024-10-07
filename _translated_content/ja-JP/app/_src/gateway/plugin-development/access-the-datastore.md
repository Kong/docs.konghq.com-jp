---
title: "データストアへのアクセス"
book: "plugin_dev"
chapter: 6
---
Kongは、「DAO」と呼ばれるクラスを通じてモデルレイヤーとやり取りします。この
章では、データストアとのやり取りで使用できるAPIについて詳しく説明します。

{% if_version lte:3.3.x %}
Kongは[PostgreSQL

{{site.data.kong_latest.dependencies.postgres}}](http://www.postgresql.org/)と[Cassandra

{{site.data.kong_latest.dependencies.cassandra}}](http://cassandra.apache.org/)の2つのプライマリデータストアをサポートしています。

{% include_cached /md/enterprise/cassandra-deprecation.md length='short' release=page.release %}
{% endif_version %}

{% if_version gte:3.4.x %}
Kong はデータストアとして [PostgreSQL 
{{site.data.kong_latest.dependencies.postgres}}](http://www.postgresql.org/) をサポートしています。

{% endif_version %}

kong.db
-------

Kongのすべてのエンティティは、次のように表されます。

* エンティティがデータストア内のどのテーブルに関連するか、 外部キー、null以外の制約など、そのフィールドの制約を記述するスキーマ。 このスキーマは[プラグイン設定]({{page.book.chapters.plugin-configuration}})の章で記述されるテーブルです。
* 現在使用中のデータベースにマッピングする`DAO`クラスのインスタンス。 このクラスのメソッドはスキーマを消費し、 そのタイプのエンティティを挿入、更新、選択、削除するためのメソッドを公開します。

Kong のコアエンティティは、サービス、ルート、コンシューマ、プラグインです。
これらはすべて、`kong.db` グローバルシングルトンを通してデータアクセスオブジェクト（DAO）としてアクセスできます。

```lua
-- Core DAOs
local services  = kong.db.services
local routes    = kong.db.routes
local consumers = kong.db.consumers
local plugins   = kong.db.plugins
```

Kong のコアエンティティとプラグインのカスタムエンティティの両方が `kong.db.*` から利用可能です。

DAO Lua API
-----------

DAOクラスは、一般的にKongのエンティティにマッピングされる、データストアの特定のテーブルで実行される操作を担当します。サポートされているすべての基礎となるデータベースは同じインターフェースに準拠するため、すべてがDAOと互換性を持つようになります。

たとえば、サービスとプラグインの挿入は次のように簡単です。

```lua
local inserted_service, err = kong.db.services:insert({
  name = "httpbin",
  url  = "http://httpbin.org",
})

local inserted_plugin, err = kong.db.plugins:insert({
  name    = "key-auth",
  service = inserted_service,
})
```

プラグインで使用されている DAO の実際の例については、
[Key\-Auth プラグインのソースコード](https://github.com/Kong/kong/blob/master/kong/plugins/key-auth/handler.lua)を参照してください。

