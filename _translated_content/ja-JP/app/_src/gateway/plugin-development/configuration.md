---
title: "プラグインの設定"
book: "plugin_dev"
chapter: 5
---
ほとんどの場合、プラグインがユーザーの
あらゆるニーズに応答するよう設定できるのは理にかなっています。プラグインの設定は
データストアに保存され、プラグインが実行されるときに Kongがそれを取得し、
[handler.lua]({{page.book.chapters.custom-logic}})メソッドに渡すことができます。

構成は、 **スキーマと呼ばれるKongのLuaテーブルで構成されています** 。これには、ユーザーが[Admin APIを通じて](/gateway/{{page.release}}/admin-api)プラグインを有効にするときに設定するキー/値のプロパティが含まれています。Kongは、プラグインに対するユーザーの設定を検証する方法を提供します。

ユーザーが[Admin API](/gateway/{{page.release}}/admin-api)にリクエストを発行し、
特定のサービス、ルート、コンシューマスキーマでプラグインを有効化または更新するときに、プラグインの構成が
スキーマに対して検証されます。

たとえば、ユーザーが次のリクエストを実行します。

```bash
curl -X POST http://localhost:8001/services/<service-name-or-id id="sl-md0000000">/plugins \
  -d "name=my-custom-plugin" \
  -d "config.foo=bar"
```

`config`オブジェクトのすべてのプロパティが、スキーマに従って有効であれば、APIは`201 Created`を返し、プラグインは構成とともにデータベースに保存されます。

```lua
{
  foo = "bar"
}
```

構成が有効でない場合、Admin APIは`400 Bad Request`と適当なエラーメッセージを返します。

モジュール
-----

    kong.plugins.<plugin_name id="sl-md0000000">.schema

schema.luaの仕様書
--------------

このモジュールは、プラグインが後でユーザーによって構成される方法を定義するプロパティとLuaテーブルを返すためのものです。使用可能なプロパティは以下のとおりです。

|     プロパティ名      |  Luaのタイプ   |            説明            |
|-----------------|------------|--------------------------|
| `name`          | `string`   | プラグインの名前、たとえば`key-auth`。 |
| `fields`        | `table`    | フィールド定義の配列。              |
| `entity_checks` | `function` | エンティティレベルの条件付き検証チェックの配列。 |

すべてのプラグインは、次のようなデフォルトフィールドを継承します。

|    フィールド名    |  Luaのタイプ  |                説明                |
|--------------|-----------|----------------------------------|
| `id`         | `string`  | 自動生成されたプラグインID。                  |
| `name`       | `string`  | プラグインの名前、たとえば`key-auth`。         |
| `created_at` | `number`  | プラグイン設定の作成時間（エポックからの経過秒数）。       |
| `route`      | `table`   | プラグインがバインドされているルート \(存在する場合\)。 |
| `service`    | `table`   | プラグインがバインドされるサービス \(該当する場合\)。  |
| `consumer`   | `table`   | 可能であれば、プラグインがバインドされるコンシューマ。      |
| `protocols`  | `table`   | プラグインは指定されたプロトコルで実行されます。         |
| `enabled`    | `boolean` | プラグインが有効になっているかどうか。              |
| `tags`       | `table`   | プラグインのタグ。                        |

ほとんどの場合、これらのほとんどを無視してデフォルトを使用できます。または、プラグインを有効にする際にユーザーが値を指定できるようにすることもできます。

以下に、潜在的な`schema.lua`ファイルの例を示します（いくつかのオーバーライドが適用されています）。

```lua
local typedefs = require "kong.db.schema.typedefs"


return {
  name = "<plugin-name id="sl-md0000000">",
  fields = {
    {
      -- this plugin will only be applied to Services or Routes
      consumer = typedefs.no_consumer
    },
    {
      -- this plugin will only run within Nginx HTTP module
      protocols = typedefs.protocols_http
    },
    {
      config = {
        type = "record",
        fields = {
          -- Describe your plugin's configuration's schema here.        
        },
      },
    },
  },
  entity_checks = {
    -- Describe your plugin's entity validation rules
  },
}
```

構成スキーマの説明
---------

`schema.lua`ファイルの`config.fields`プロパティには、プラグインの構成スキーマが記述されます。
それぞれプラグインの有効な構成プロパティであるフィールドについて、該当プロパティのルールの説明を含む定義が柔軟な配列形式で記述されます
以下に例を示します。

```lua
{
  name = "<plugin-name id="sl-md0000000">",
  fields = {
    config = {
      type = "record",
      fields = {
        {
          some_string = {
            type = "string",
            required = false,
          },
        },
        {
          some_boolean = {
            type = "boolean",
            default = false,
          },
        },
        {
          some_array = {
            type = "array",
            elements = {
              type = "string",
              one_of = {
                "GET",
                "POST",
                "PUT",
                "DELETE",
              },
            },
          },
        },
      },
    },
  },
}
```

以下は、プロパティに対して受け入れられる一般的な（すべてではありません）ルールのリストです（例については、上記のフィールドテーブルを参照してください）。

|    ルール     |              説明              |
|------------|------------------------------|
| `type`     | プロパティのタイプ。                   |
| `required` | プロパティが必須かどうか                 |
| `default`  | プロパティの値が指定されていない場合のデフォルト値    |
| `elements` | `array`要素または`set`要素のフィールド定義。 |
| `keys`     | `map`キーのフィールド定義です。           |
| `values`   | `map`値のフィールド定義。              |
| `fields`   | `record`フィールドのフィールド定義。       |

他にもたくさんありますが、上記が一般的に使用されています。

フィールドバリデーターを追加することもできます。いくつか例を挙げます。

|        ルール         |                   説明                    |
|--------------------|-----------------------------------------|
| `between`          | 入力された数値が許容値の間にあることを確認します。               |
| `eq`               | 入力と許容値の等価性を確認します。                       |
| `ne`               | 入力された値が許可された値と一致していないことを確認します。          |
| `gt`               | 数値が与えられた値より大きいかどうかをチェックします。             |
| `len_eq`           | 入力文字列の長さが指定された値と等しいことを確認します。            |
| `len_min`          | 入力文字列の長さが指定された値より大きいことを確認します。           |
| `len_max`          | 入力文字列の長さが、指定された値以下であることを確認します。          |
| `match`            | 入力文字列が指定されたLuaパターンと一致するかどうかを確認します。      |
| `not_match`        | 入力文字列が指定されたLuaパターンと一致しないことを確認します。       |
| `match_all`        | 入力文字列が指定されたすべてのLuaパターンと一致するかどうかを確認します。  |
| `match_none`       | 入力文字列が指定されたLuaパターンのいずれかにも一致しないことを確認します。 |
| `match_any`        | 入力文字列が指定されたLuaパターンのいずれかと一致するかどうかを確認します。 |
| `starts_with`      | 入力文字列が指定された値で始まることを確認します。               |
| `one_of`           | 入力文字列が受け入れられる値の 1 つであることを確認します。         |
| `contains`         | 入力配列に指定した値が含まれているかどうかを調べます。             |
| `is_regex`         | 入力文字列が有効な正規表現パターンであるかどうかを確認します。         |
| `custom_validator` | Luaで記述されたカスタム検証関数。                      |

他にもバリデータがいくつかありますが、上記の表から、フィールドにバリデーションルールを指定する方法がよくわかります。

### 例

この`schema.lua`ファイルは[キー認証](/hub/kong-inc/key-auth/)プラグイン用です:

```lua
-- schema.lua
local typedefs = require "kong.db.schema.typedefs"


return {
  name = "key-auth",
  fields = {
    {
      consumer = typedefs.no_consumer
    },
    {
      protocols = typedefs.protocols_http
    },
    {
      config = {
        type = "record",
        fields = {
          {
            key_names = {
              type = "array",
              required = true,
              elements = typedefs.header_name,
              default = {
                "apikey",
              },
            },
          },
          {
            hide_credentials = {
              type = "boolean",
              default = false,
            },
          },
          {
            anonymous = {
              type = "string",
              uuid = true,
            },
          },
          {
            key_in_body = {
              type = "boolean",
              default = false,
            },
          },
          {
            run_on_preflight = {
              type = "boolean",
              default = true,
            },
          },
        },
      },
    },
  },
}
```

したがって、プラグインの`access()`関数を[handler.lua]({{page.book.chapters.custom-logic}})に実装し、ユーザーがプラグインをデフォルト値で有効にすると、以下にアクセスできるようになります。

```lua
-- handler.lua

local CustomHandler = {
  VERSION  = "1.0.0",
  PRIORITY = 10,
}

local kong = kong

function CustomHandler:access(config)

  kong.log.inspect(config.key_names)        -- { "apikey" }
  kong.log.inspect(config.hide_credentials) -- false
end


return CustomHandler
```

上記の例では、[プラグイン開発キット](/gateway/{{page.release}}/plugin-development/pdk)の
[kong.log.inspect](/gateway/{{page.release}}/plugin-development/pdk/kong.log/#kong_log_inspect)
関数を使用して、Kongのログに該当値を出力しています。

*** ** * ** ***

次のような複雑な例は、ロギングプラグインの最終形として使用できる可能性があります。

```lua
-- schema.lua
local typedefs = require "kong.db.schema.typedefs"


return {
  name = "my-custom-plugin",
  fields = {
    {
      config = {
        type = "record",
        fields = {
          {
            environment = {
              type = "string",
              required = true,
              one_of = {
                "production",
                "development",
              },
            },
          },
          {
            server = {
              type = "record",
              fields = {
                {
                  host = typedefs.host {
                    default = "example.com",
                  },
                },
                {
                  port = {
                    type = "number",
                    default = 80,
                    between = {
                      0,
                      65534
                    },
                  },
                },  
              },
            },
          },
        },
      },
    },
  },
}
```

このような設定により、ユーザーは次のようにプラグインに設定を POST できます。

```bash
curl -X POST http://localhost:8001/services/<service-name-or-id id="sl-md0000000">/plugins \
  -d "name=my-custom-plugin" \
  -d "config.environment=development" \
  -d "config.server.host=http://localhost"
```

そして、[handler.lua]({{page.book.chapters.custom-logic}})では以下が利用可能になります。

```lua
-- handler.lua

local CustomHandler = {
  VERSION  = "1.0.0",
  PRIORITY = 10,
}

local kong = kong

function CustomHandler:access(config)

  kong.log.inspect(config.environment) -- "development"
  kong.log.inspect(config.server.host) -- "http://localhost"
  kong.log.inspect(config.server.port) -- 80
end


return CustomHandler
```

また、[Key\-Authプラグインのソースコード](https://github.com/Kong/kong/blob/master/kong/plugins/key-auth/schema.lua)でスキーマの実際の例を見ることができます。

