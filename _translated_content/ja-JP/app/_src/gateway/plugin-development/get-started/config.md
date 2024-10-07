---
title: "プラグイン設定を追加"
book: "plugin_dev_getstarted"
chapter: 4
---
以下は、 {{site.base_gateway}}のカスタム プラグインに設定を追加するためのステップ バイ ステップ ガイドです。

前提条件
----

このページは[「はじめに」](/gateway/{{page.release}}/plugin-development/get-started/index)の第3章です。
カスタムプラグインを開発するためのガイド。 これらの手順はガイドの前の章を参照しており、同じ開発者ツールの前提条件を必要とします。

段階的な手順
------

テスト付きの基本的なプラグインプロジェクトができたので、次の手順でプラグインコードへの
設定機能を追加していきます。

### スキーマに構成フィールドを追加

`schema.lua`ファイルにいくつかの設定フィールドを追加して、再デプロイや再起動をせずにプラグインの動作を設定できるようにしましょう。


{{site.base_gateway}}の基本的な[型の定義](https://github.com/Kong/kong/blob/master/kong/db/schema/typedefs.lua)モジュールでは、APIゲートウェイプラグインに関連してよく使用される型について役立つ情報が得られます。

`schema.lua` ファイルの先頭に、以下のステートメントで {{site.base_gateway}}
`typedefs` モジュールを組み込みます。

```lua
local typedefs = require "kong.db.schema.typedefs"
```

以下のLuaスニペットに表示されている典型的な
[構成フィールド](/gateway/{{page.release}}/plugin-development/configuration/#describing-your-configuration-schema)
では、`response_header_name`という名前のフィールドを定義します。

ここでは、フィールドをnullにできない文字列と定義し、ヘッダー名に関するゲートウェイのルールに準拠する`header_name`型定義を使用しています。

```lua
 { response_header_name = typedefs.header_name {
     required = false,
     default = "X-MyPlugin" } },
```

この定義では、設定値が必須では *ない* こと、つまり、プラグインを設定する際にユーザーにとってオプションであることも示されています。また、ユーザーが値を指定しない場合に使用されるデフォルト値も指定します。

この定義を、先ほど定義した `fields` 配列内の `schema.lua` ファイルに配置します。

完全な `schema.lua` は次のようになります。

```lua
local typedefs = require "kong.db.schema.typedefs"

local PLUGIN_NAME = "my-plugin"

local schema = {
  name = PLUGIN_NAME,
  fields = {
    { config = {
        type = "record",
        fields = {
          { response_header_name = typedefs.header_name {
            required = false,
            default = "X-MyPlugin" } },
        },
      },
    },
  },
}

return schema
```

これで、プラグインは構成値を受け入れることができます。プラグインのロジックコードからそれを読み取る方法を見てみましょう。

### プラグインコードから構成値を読み取ります

現在ハードコードされている値ではなく、受信した `conf` パラメータから構成値を読み取るように、`handler.lua` ファイル `response` 関数を変更します。

関数は次のようになります。

```lua
function MyPluginHandler:response(conf)
    kong.response.set_header(conf.response_header_name, "response")
end
```

必要なのはこれだけです。プラグインは実行時に動的な構成が可能になりました。次は、
変更が期待どおり機能することを手動および自動テストで検証しましょう。

### 手動で構成を検証する

[前回のテストの章](/gateway/{{page.release}}/plugin-development/get-started/testing#3-manually-test-plugin)では、
Pongo を使用してシェルを実行し、プラグインの動作を手動で検証する方法を示しました。ここで同じプロセスを繰り返し、プラグインの新しい構成可能な動作を検証します。

ゲートウェイを起動し、シェルを開きます。

```sh
pongo shell
```

シェル内からデータベースの移行を実行し、{{site.base_gateway}} を起動します。

```sh
kms
```

テストサービスを追加します。

```sh
curl -i -s -X POST http://localhost:8001/services \
   --data name=example_service \
   --data url='http://httpbin.org'
```

プラグインインスタンスを作成しますが、今回は構成値を`POST`リクエストのデータとしてAdmin APIに指定します。

```sh
curl -is -X POST http://localhost:8001/services/example_service/plugins \
    --data 'name=my-plugin' --data 'config.response_header_name=X-CustomHeaderName'
```

テストリクエストを送信できるようにルートを追加します。

```sh
curl -i -X POST http://localhost:8001/services/example_service/routes \
   --data 'paths[]=/mock' \
   --data name=example_route
```

そして、テストルートを通してリクエストをプロキシします。

```sh
curl -i http://localhost:8000/mock/anything
```

今回は、設定したヘッダーがレスポンスに表示されるはずです。

```sh
...
[X-CustomHeaderName] = 'response'
...
```

`my-plugin`が構成可能になり、その動作を検証しました。次は、変更を自動的に検証する方法をどのように作成できるかを見ていきましょう。

### 自動構成テストを追加する

何も変更を加えずに、`pongo run`にすでに存在するテストを再実行できます。

```sh
pongo run
```

テストは引き続き成功を報告するはずです。

```sh
[  PASSED  ] 2 tests.
```

構成可能なフィールドの既定値がテストケースの値と一致するため、テストはまだ有効です。`response_header_name` フィールドの異なる値を検証する方法を見てみましょう。

`spec/01-integration_spec.lua`モジュール内の`setup`関数を変更して、 `my-plugin`が
データベースに追加されたものは、 `response_header_name`フィールドに異なる値で構成されています。

コードは次のとおりです。

```lua
-- Add the custom plugin to the test route
blue_print.plugins:insert {
  name = PLUGIN_NAME,
  route = { id = test_route.id },
  config = {
    response_header_name = "X-CustomHeaderName",
  },
}
```

Pongoを使用してテストを再実行します。

```sh
pongo run
```

すべてを適切に変更した場合、テストは失敗し、次のようなレポートが生成されます。

```sh
/kong-plugin/spec/my-plugin/01-integration_spec.lua:75: Expected header:
(string) 'X-MyPlugin'
But it was not found in:
(table: 0x484c9942b180) {
  [Connection] = 'keep-alive'
  [Content-Length] = '1016'
  [Content-Type] = 'application/json'
  [Date] = 'Mon, 18 Mar 2024 13:49:06 GMT'
  [Server] = 'mock-upstream/1.0.0'
  [Via] = 'kong/{{page.release}}'
  [X-CustomHeaderName] = 'response'
  [X-Kong-Proxy-Latency] = '1'
  [X-Kong-Request-Id] = '92dfcf56b12b0d29f54ed9295aed6356'
  [X-Kong-Upstream-Latency] = '2'
  [X-Powered-By] = 'mock_upstream' }
stack traceback:
        /kong-plugin/spec/my-plugin/01-integration_spec.lua:75: in function </kong-plugin/spec/my-plugin/01-integration_spec.lua:66>
[  FAILED  ] /kong-plugin/spec/my-plugin/01-integration_spec.lua:66: my-plugin: [#postgres] The response gets a 'X-MyPlugin' header (4.58 ms)
```

プラグイン構成を更新し、テストケースで予期されるヘッダー名を更新していないためテストは失敗しました。テストを修正するには、テストアサーションが構成済みのヘッダー名と一致するように修正します。

この行を変更します。

```lua
local header_value = assert.response(r).has.header("X-MyPlugin")
```

新しく設定されたヘッダー名を予想するには、以下を実行します。

```lua
local header_value = assert.response(r).has.header("X-CustomHeaderName")
```

テストを再実行すると、次のようになります。

```sh
pongo run
```

Pongoは、テストの実行が成功したことを報告する必要があります。

```sh
[  PASSED  ] 2 tests.
```

次の手順
----

これで、完全な開発環境とカスタムプラグインに機能を追加するためのフレームワークができました。次に、プラグインコード内から外部サービスに接続する方法を示し、その後、プラグインを{{site.base_gateway}}データプレーンにデプロイしてビジネスロジックを本番環境に配置する方法を示します。

