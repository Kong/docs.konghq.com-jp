---
title: "カスタムエンティティのキャッシュ"
book: "plugin_dev"
chapter: 8
---
プラグインは、各リクエストや応答に対してカスタムエンティティ（[前章]({{page.book.previous.url}})で説明）に頻繁にアクセスする必要がある可能性があります。
通常、一度ロードし、メモリに動的にキャッシュするとパフォーマンスは向上しますが、ロード量の増加でデータストアへの負荷が増加しないようにします。

リクエストごとにAPIキーを検証する必要があるAPI Key Authenticationプラグインを考えてみてください。
リクエストのたびに、カスタム認証情報オブジェクトがデータストアからロードされます。
クライアントがリクエストと一緒にAPIキーを提供すると、
通常は、データストアにクエリしてそのキーが存在するかどうかを確認し、それからユーザーを識別するために、
リクエストをブロックするか、コンシューマIDを取得します。これが
すべてのリクエストで発生すると、非常に非効率的です。

* データストアのクエリを実行すると、すべてのリクエストでレイテンシが発生し、リクエストの処理がおそくなります。
* データストアは読み込み量の増加の影響も受け、クラッシュまたは減速する恐れがあります。その影響はすべてのKongノードに波及します。 

データストアに対して毎回クエリを実行しないようにするには、ノード上でカスタムエンティティをメモリ内にキャッシュします。そうすることで、頻繁にエンティティを参照しても、初回以外はデータストア クエリを毎回トリガーすることなくメモリ内で実行されます。そして、データストアからクエリを実行するよりもはるかに高速で信頼性が高くなります（特に負荷が高い場合） 。

モジュール
-----

    kong.plugins.<plugin_name id="sl-md0000000">.daos

カスタムエンティティのキャッシュ
----------------

カスタムエンティティを定義したら、[プラグイン開発キット](/gateway/{{page.release}}/plugin-development/pdk)によって提供される
[kong.cacheを](/gateway/{{page.release}}/plugin-development/pdk/#kong-cache)モジュールを使用して
メモリ内にキャッシュすることができます。

    local cache = kong.cache

キャッシュには次の2つのレベルがあります。

1. L1: Luaメモリ ャッシュ \- Nginxワーカープロセスにローカル。これは、任意のタイプのLua値を保持できます。
2. L2: 共有メモリキャッシュ（SHM）\- Nginx ノードに対してはローカルですが、 そのNginx ノードのすべてのワーカー間で共有されます。これはスカラー値しか保持できないので、Lua テーブルのようなより複雑な型の（逆）シリアル化が必要となります。

データがデータベースから取得されると、両方のキャッシュに保存されます。
同じワーカープロセスが再度データを要求すると、
Lua メモリキャッシュから以前にシリアル化されたデータを取得します。同じ Nginx ノード内の異なるワーカーがそのデータを要求した場合、
SHM でデータを見つけ、逆シリアル化し（そして独自の Lua メモリキャッシュに保存して）、返します。

このモジュールでは、次の機能が公開されています。

|                      関数名                      |                                                                                                                   説明                                                                                                                    |
|-----------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value, err = cache:get(key, opts?, cb, ...)` | キャッシュから値を取得します。キャッシュに値がない場合（miss）、保護モードで`cb`を呼び出します。`cb`は、キャッシュされる1つの値（1つのみ）を返す必要があります。エラーがスローされる *可能性があります* が、それらはKongによってキャッチされ、`ngx.ERR`レベルで適切にログに記録されます。この関数は負の結果（`nil`）をキャッシュ **します** 。そのため、エラーをチェックするときは、2番目の引数`err`に依存する必要があります。 |
| `ttl, err, value = cache:probe(key)`          | 値がキャッシュされているかどうかを確認します。キャッシュされている場合は、残りの TTL を返します。キャッシュされていない場合は、`nil` を返します。キャッシュされる値はネガティブキャッシュになることもあります。3 番目の戻り値は、キャッシュされる値自体です。                                                                                                   |
| `cache:invalidate_local(key)`                 | ノードのキャッシュから値を削除します。                                                                                                                                                                                                                     |
| `cache:invalidate(key)`                       | ノードのキャッシュから値を削除 **し** 、そのエビクションイベントをクラスタ内の他のすべてのノードに伝達します。                                                                                                                                                                              |
| `cache:purge()`                               | ノードのキャッシュから **すべての** 値を削除します。                                                                                                                                                                                                           |

認証プラグインの例に戻り、特定のAPIキーで認証情報を検索する場合、以下のような記述になります。

```lua
-- handler.lua

local CustomHandler = {
  VERSION  = "1.0.0",
  PRIORITY = 10,
}

local kong = kong

local function load_credential(key)
  local credential, err = kong.db.keyauth_credentials:select_by_key(key)
  if not credential then
    return nil, err
  end
  return credential
end


function CustomHandler:access(config)

  -- retrieve the apikey from the request querystring
  local key = kong.request.get_query_arg("apikey")

  local credential_cache_key = kong.db.keyauth_credentials:cache_key(key)

  -- We are using cache.get to first check if the apikey has been already
  -- stored into the in-memory cache. If it's not, then we lookup the datastore
  -- and return the credential object. Internally cache.get will save the value
  -- in-memory, and then return the credential.
  local credential, err = kong.cache:get(credential_cache_key, nil,
                                         load_credential, credential_cache_key)
  if err then
    kong.log.err(err)
    return kong.response.exit(500, {
      message = "Unexpected error"
    })
  end

  if not credential then
    -- no credentials in cache nor datastore
    return kong.response.exit(401, {
      message = "Invalid authentication credentials"
    })
  end

  -- set an upstream header if the credential exists and is valid
  kong.service.request.set_header("X-API-Key", credential.apikey)
end


return CustomHandler
```

上記の例では、リクエストの処理、モジュールのキャッシュ、または当社のプラグインから応答を生成するために、[プラグイン開発キット](/gateway/{{page.release}}/plugin-development/pdk)からさまざまなコンポーネントを使用していることに注意してください。

さて、上記の仕組みを導入すると、コンシューマがAPIキー使ってリクエストを作成すると、
キャッシュはウォーム状態とみなされ、その後のリクエストは
データベースクエリの結果になりません。

キャッシュは[キー認証プラグインハンドラ](https://github.com/Kong/kong/blob/master/kong/plugins/key-auth/handler.lua)の複数の場所で使用されます。
公式プラグインでのキャッシュの使用方法については、ファイルをご覧ください。

### ユーザー定義エンティティを更新または削除する

Admin API を使用して、キャッシュされたカスタムエンティティがデータストアで更新または削除されるたびに、データストア内のデータと Kong ノードのメモリにキャッシュされたデータの間に不整合が発生します。この不整合を回避するには、キャッシュされたエンティティをメモリ内ストアから削除し、Kong にデータストアのエンティティを再度リクエストさせる必要があります。このプロセスをキャッシュ無効化と呼びます。

エンティティのキャッシュ無効化
---------------

キャッシュされたエンティティがTTLに達するのを待つ代わりに、CRUD操作時に無効にするには、次のいくつかのステップに従う必要があります。
このプロセスはほとんどのエンティティで自動化できますが、より複雑な関係を持つ一部のエンティティを無効にするために、一部のCRUDイベントを手動でサブスクライブする必要が生じることがあります。

### 自動キャッシュ無効化

エンティティのスキーマの`cache_key`プロパティを信頼している場合、
キャッシュの無効化は追加設定なしでエンティティに提供されます。たとえば、次のスキーマでは
このようになります。

```lua
local typedefs = require "kong.db.schema.typedefs"


return {
  -- this plugin only results in one custom DAO, named `keyauth_credentials`:
  keyauth_credentials = {
    name                  = "keyauth_credentials", -- the actual table in the database
    endpoint_key          = "key",
    primary_key           = { "id" },
    cache_key             = { "key" },
    generate_admin_api    = true,
    admin_api_name        = "key-auths",
    admin_api_nested_name = "key-auth",    
    fields = {
      {
        -- a value to be inserted by the DAO itself
        -- (think of serial id and the uniqueness of such required here)
        id = typedefs.uuid,
      },
      {
        -- also interted by the DAO itself
        created_at = typedefs.auto_timestamp_s,
      },
      {
        -- a foreign key to a consumer's id
        consumer = {
          type      = "foreign",
          reference = "consumers",
          default   = ngx.null,
          on_delete = "cascade",
        },
      },
      {
        -- a unique API key
        key = {
          type      = "string",
          required  = false,
          unique    = true,
          auto      = true,
        },
      },
    },
  },
}
```

このAPIキーエンティティのキャッシュキーを`key`属性として宣言していることがわかります。ここで`key`を使用するのは、一意の制約が適用されているためです。したがって、`cache_key` に追加される属性は、2つのエンティティが同じキャッシュキーを生成することがないようにするため、一意の組み合わせである必要があります。

この値を追加すると、そのエンティティのDAOで次の関数を使用できるようになります。

```lua
cache_key = kong.db.<dao id="sl-md0000000">:cache_key(arg1, arg2, arg3, ...)
```

引数はスキーマの
`cache_key`プロパティで指定された属性でなければならず、指定された順に表示されます。この関数は
一意であることが保証された文字列値`cache_key`を計算します。

たとえば、APIキーの`cache_key`を生成する場合は次のようになります。

```lua
local cache_key = kong.db.keyauth_credentials:cache_key("abcd")
```

これによりAPIキー`"abcd"`の`cache_key`が生成され（クエリの引数の1つから取得）、キャッシュからキーを取得するのに使用できます（またはキャッチがミスの場合はデータベースから取得）。

```lua
local key       = kong.request.get_query_arg("apikey")
local cache_key = kong.db.keyauth_credentials:cache_key(key)

local credential, err = kong.cache:get(cache_key, nil, load_entity_key, apikey)
if err then
  kong.log.err(err)
  return kong.response.exit(500, { message = "Unexpected error" })
end

if not credential then
  return kong.response.exit(401, { message = "Invalid authentication credentials" })
end


-- do something with the credential
```

`cache_key`がこのように生成され、エンティティのスキーマで指定されている場合、
キャッシュの無効化は自動プロセスになります。
このAPIキーに影響を与えるすべてのCRUD操作は、影響を与える`cache_key`をKongに生成させ、クラスタ上の他のすべてのノードにそれをブロードキャストすることで、特定の値をキャッシュから退避させたうえで、次のリクエストでデータストアから新しい値を取得できるようにします。

親エンティティが CRUD 操作を受けているとき（例えば、スキーマの `consumer_id` 属性に従い、この API キーを所有するコンシューマ）、Kong は
親エンティティと子エンティティの両方に対してキャッシュを無効化するメカニズムを適用します。

**注** : Kongが提供するネガティブキャッシュに注意してください。上記の例では、特定のキーのデータストアにAPIキーがない場合、キャッシュモジュールはヒットしたかのようにミスを保存します。つまり、「Create」イベント（この特定のキーを使用してAPIキーを作成するイベント）はKongによっても伝播されるため、ミスを保存したすべてのノードがミスを削除し、新しく作成されたAPIキーをデータストアから適切に取得できます。

[Clustering Guide](/gateway/{{page.release}}/production/deployment-topologies/traditional/)を参照して、このような無効化イベントに対してクラスタが適切に構成されていることを確認してください。

### 手動によるキャッシュの無効化

場合によっては、エンティティスキーマの`cache_key`プロパティは十分に柔軟でない場合があり、手動でそのキャッシュを無効にする必要があります。これは、プラグインが従来の`foreign = "parent_entity:parent_attribute"`構文を介して別のエンティティとの関係を定義していないか、DAOの`cache_key`メソッドを使用していないか、何らかの形でキャッシングメカニズムを悪用しているのが原因です。

そのような場合は、Kongがリッスンしているものと同じ
無効化チャンネルに自分のサブスクライバーを手動でセットアップし、独自のカスタム無効化作業を行うことができます。

Kong内の無効化チャンネルでリッスンするには、プラグインの`init_worker`ハンドラーで
以下を実装してください。

```lua
function MyCustomHandler:init_worker()
  -- listen to all CRUD operations made on Consumers
  kong.worker_events.register(function(data)

  end, "crud", "consumers")

  -- or, listen to a specific CRUD operation only
  kong.worker_events.register(function(data)
    kong.log.inspect(data.operation)  -- "update"
    kong.log.inspect(data.old_entity) -- old entity table (only for "update")
    kong.log.inspect(data.entity)     -- new entity table
    kong.log.inspect(data.schema)     -- entity's schema
  end, "crud", "consumers:update")
end
```

上記のリスナーが目的のエンティティに配置されたら、
プラグインがキャッシュしたエンティティを手動で無効化できます。
例:

```lua
kong.worker_events.register(function(data)
  if data.operation == "delete" then
    local cache_key = data.entity.id
    kong.cache:invalidate("prefix:" .. cache_key)
  end
end, "crud", "consumers")
```

{% if_version gte:3.5.x %}
{:.note}
> 
> 多くの場合、プラグインを使用して`configure`を実装し、イベントを使用せずに問題/ニーズが解決するかどうかを確認するとよいでしょう。たとえば、Kongノードの役割（従来型、DBレス、データプレーン）によってイベントの動作が異なる場合があります。{% endif_version %}

Admin APIの拡張
------------

すでにご存知かも知れませんが、[Admin API](/gateway/{{page.release}}/admin-api/)はKongユーザーがAPIやプラグインを設定するためにKongと通信する場所です。これらはプラグイン用に実装したカスタムエンティティにも対応できる必要があります（たとえば、APIキーの作成および削除）。これは、Admin APIを拡張して実行します。次の章「[Admin APIの拡張]({{page.book.next.url}})」で詳細をご紹介します。

