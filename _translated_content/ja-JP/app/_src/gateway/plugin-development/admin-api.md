---
title: "Admin APIの拡張"
book: "plugin_dev"
chapter: 9
---
{:.note}
> 
> **注:** 
> 
> * この章ではユーザーに[Lapis](http://leafo.net/lapis/)に関する知識がある程度あることを前提としています。 
> * Admin API拡張機能は、ストリームプラグインではなく、HTTPプラグイン用です。 

Kongは[Admin API](/gateway/{{page.release}}/admin-api/)と呼ばれるRESTインターフェースを使用して構成できます。プラグインは、独自のエンドポイントを追加することで拡張し、カスタムエンティティやその他の個別の管理ニーズに対応できます。典型的な例としては、APIキーの作成、取得、削除（通常「CRUD操作」と呼ばれるもの）です。

Admin API は [Lapis](http://leafo.net/lapis/) アプリケーションであり、Kong の抽象化レベルによりエンドポイントの追加が容易になります。

モジュール
-----

    kong.plugins.<plugin_name id="sl-md0000000">.api

Admin APIにエンドポイントを追加する
----------------------

Kongは、エンドポイントが次の名前のモジュールで定義されている場合、それを検出してロードします。

    "kong.plugins.<plugin_name id="sl-md0000000">.api"

このモジュールは、次の構造を持つ 1 つ以上のエントリを含むテーブルを返すようにバインドされています。

```lua
{
  ["<path id="sl-md0000000">"] = {
     schema = <schema id="sl-md0000000">,
     methods = {
       before = function(self) ... end,
       on_error = function(self) ... end,
       GET = function(self) ... end,
       PUT = function(self) ... end,
       ...
     }
  },
  ...
}
```

この場合、以下の通りです。

* `<path id="sl-md0000000">` は、`/users` のようなルートを表す文字列である必要があります（詳細については、[Lapis routes&URL パターン](http://leafo.net/lapis/reference/actions.html#routes--url-patterns) を参照してください）。パスには、`/users/:users/new` のような補間パラメータを含めることができます。
* `<schema id="sl-md0000000">` は、スキーマ定義です。`kong.db.<entity id="sl-md0000000">.schema` を通じて、コアエンティティおよびカスタムプラグインエンティティのスキーマが利用可能です。スキーマは、特定のフィールドをその型に応じて解析するために使用されます。例えば、フィールドが整数としてマークされている場合、関数に渡される際には整数として解析されます（デフォルトでは、フォームフィールドはすべて文字列です）。
* `methods`サブテーブルには、文字列でインデックス付けされた関数が含まれています。 
  * `before`キーはオプションで、機能を保持することができます。存在する場合、他の関数が呼び出される前に、 `path`にヒットするすべてのリクエストで関数が実行されます。
  * 1 つ以上の関数を、 `GET`や`PUT`などのHTTPメソッド名でインデックス付けできます。これらの機能は、適切なHTTPメソッドと`path`が一致すると実行されます。`path`に`before`関数が存在する場合は、最初に実行されます。`before`関数は`kong.response.exit`を使用して早期に終了し、「通常」のhttpメソッド関数を効果的にキャンセルできることに注意してください。
  * `on_error`キーはオプションで、機能を保持することができます。存在する場合、他の関数（`before` または「http メソッド」のいずれか）からのコードがエラーをスローしたときに、関数が実行されます。存在しない場合、Kongはデフォルトのエラーハンドラーを使用してエラーを返します。

以下に例を示します。

```lua
local endpoints = require "kong.api.endpoints"

local credentials_schema = kong.db.keyauth_credentials.schema
local consumers_schema = kong.db.consumers.schema

return {
  ["/consumers/:consumers/key-auth"] = {
    schema = credentials_schema,
    methods = {
      GET = endpoints.get_collection_endpoint(
              credentials_schema, consumers_schema, "consumer"),

      POST = endpoints.post_collection_endpoint(
              credentials_schema, consumers_schema, "consumer"),
    },
  },
}
```

このコードは`/consumers/:consumers/key-auth`に2つのAdmin APIエンドポイントを作成して、（`GET`）を取得し、特定のコンシューマに関連付けられた（`POST`）認証情報を作成します。この例では、これらの関数は`kong.api.endpoints`ライブラリで提供されます。

現在、`endpoints`モジュールには、Kongで最もよく使用されるCRUD操作向けのデフォルト実装が含まれています。
このモジュールは、挿入、取得、更新または削除操作のヘルプツールを提供し、必要なDAO操作を実行し、適切なHTTPステータスコードで返答します。
また、サービスの名前やIDなどのパス、またはコンシューマのユーザー名やIDからパラメータを取得する機能も提供します。

`endpoints`で提供される関数が不十分な場合は、代わりに通常のLua関数を使用できます。そこから、以下を使用できます。

* `endpoints`モジュールによって提供されるいくつかの機能。
* [PDK](/gateway/{{page.release}}/plugin-development/pdk/)が提供するすべての機能
* [Lapisリクエストオブジェクト](http://leafo.net/lapis/reference/actions.html#request-object)である`self`パラメータ
* そしてもちろん、必要に応じて任意の Lua モジュールを`require`することもできます。このルートを選択する場合は、OpenRestyと互換性があることを確認してください。

```lua
local endpoints = require "kong.api.endpoints"

local credentials_schema = kong.db.keyauth_credentials.schema
local consumers_schema = kong.db.consumers.schema

return {
  ["/consumers/:consumers/key-auth/:keyauth_credentials"] = {
    schema = credentials_schema,
    methods = {
      before = function(self, db, helpers)
        local consumer, _, err_t = endpoints.select_entity(self, db, consumers_schema)
        if err_t then
          return endpoints.handle_error(err_t)
        end
        if not consumer then
          return kong.response.exit(404, { message = "Not found" })
        end

        self.consumer = consumer

        if self.req.method ~= "PUT" then
          local cred, _, err_t = endpoints.select_entity(self, db, credentials_schema)
          if err_t then
            return endpoints.handle_error(err_t)
          end

          if not cred or cred.consumer.id ~= consumer.id then
            return kong.response.exit(404, { message = "Not found" })
          end
          self.keyauth_credential = cred
          self.params.keyauth_credentials = cred.id
        end
      end,
      GET  = endpoints.get_entity_endpoint(credentials_schema),
      PUT  = function(self, db, helpers)
        self.args.post.consumer = { id = self.consumer.id }
        return endpoints.put_entity_endpoint(credentials_schema)(self, db, helpers)
      end,
    },
  },
}
```

先ほどの例では、`/consumers/:consumers/key-auth/:keyauth_credentials`のパスには
3つの関数があります。

* `before`関数は、`endpoints`が提供するいくつかのユーティリティ（`endpoints.handle_error`）とPDK関数（`kong.response.exit`）を使用するカスタムLua関数です。また、使用される後続の関数の`self.consumer`を生成します。
* `GET` 関数は、完全に `endpoints` を使用して構築されています。これは、`before` が予め `self.consumer` のようなものを「準備」しているために可能になっています。
* `PUT` 関数は、`put_entity_endpoint` が提供する `endpoints` 関数を呼び出す前に、`self.args.post.consumer` を設定します。

