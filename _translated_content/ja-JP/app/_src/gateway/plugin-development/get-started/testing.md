---
title: "プラグインテストを追加"
book: "plugin_dev_getstarted"
chapter: 3
---
以下は、{{site.base_gateway}}カスタムプラグインのテスト環境を設定するためのガイドです。

前提条件
----

このページはカスタムプラグインを開発するための[「はじめに」](/gateway/{{page.release}}/plugin-development/get-started/index)
ガイドの第2章です。これらの手順はガイドの前の章を参照しており、同じ開発者ツールの
前提条件を必要とします。

段階的な手順
------

基本的なプラグインプロジェクトが作成されたので、そのテスト用自動化機能を構築できます。

### Pongoのインストール

[Pongo](https://github.com/Kong/kong-pongo)は、{{site.base_gateway}} のカスタムプラグインを検証して配布するのに役立つツールです。PongoはDockerを使用して{{site.base_gateway}}環境をブートストラップし、プラグインをすばやくロードし、自動テストを実行し、さまざまな{{site.base_gateway}}バージョンに対するプラグインの動作を手動で検証できるようにします。

次のスクリプトを使用すると、Pongoのインストールを自動化できます。代わりに[手動インストール手順](https://github.com/Kong/kong-pongo?tab=readme-ov-file#installation)に従うことも選択できます。

すでにPongoがインストールされている場合は、[次の手順](#initialize-the-plugin-environment)に進むか、インストールスクリプトを実行してPongoを最新バージョンに更新できます。

Pongo をインストールまたは更新するには、次のコマンドを実行します。

```sh
curl -Ls https://get.konghq.com/pongo | bash
```

このガイドの残りの部分が正しく機能するためには、システムパス内に `pongo` コマンドが必要です。上記のスクリプトと手動インストール手順の両方には、パス上に`pongo`を置くためのヒントが含まれています。

プロジェクトディレクトリ内でコマンドを実行して、`pongo`コマンドが`PATH`で使用できることを確認します。

```sh
pongo help
```

Pongo をインストールしたら、新しいプラグインのテスト環境をセットアップできます。

### テスト環境の初期化

Pongoはインストールされ利用可能なプラグインで
{{site.base_gateway}}をすばやく実行するツールを提供するため、プラグインの動作を検証できるようになります。

最初にプラグインを手動で検証しましょう。次に、このガイドの後続の手順で自動テストを追加します。

{:.note}
> 
> **注** ：{{site.base_gateway}}はさまざまな[デプロイメントトポロジー](/gateway/{{page.release}}/production/deployment-topologies)
> で実行されます。Pongoはデフォルトでは *従来モード* で{{site.base_gateway}}
> を実行します。このモードではデータベースを使用して、ルート、サービス、プラグインなどの構成済みエンティティを保存します。
> {{site.base_gateway}}とデータベースは別々のコンテナで実行されるため、データベースとは独立的にゲートウェイを循環させることができます。
> その結果、プラグインの論理的動作を迅速かつ反復的な形で検証しつつ、データベースでゲートウェイを独立状態に維持できます。

Pongoは、一部のデフォルトの構成ファイルを使用してプロジェクトディレクトリを初期化するオプションのコマンドを提供します。これを実行すると新しいプロジェクトを開始できます。

{:.important}
> 
> **重要：** これらのコマンドは、Pongoがプラグインコードを適切にパッケージ化して実行中の{{site.base_gateway}}に含めるために、`my-plugin`プロジェクトのルートディレクトリ内で実行する必要があります。

プロジェクトフォルダを初期化します。

```sh
pongo init
```

これで、{{site.base_gateway}}の依存関係コンテナを開始できます。
デフォルトでは、これには従来のモードで使用されるPostgresデータベースのみが含まれます。

依存関係を開始します。

```sh
pongo up
```

依存関係が正常に実行されたら、{{site.base_gateway}} コンテナを起動し、その中でシェルを開いてゲートウェイと通信することができます。Pongo はプリインストールされた CLI ツールを使用して {{site.base_gateway}} 
コンテナを実行し、テストの手助けをします。

ゲートウェイを起動し、次のようにシェルを開きます。

```sh
pongo shell
```

ターミナルは現在、{{site.base_gateway}}コンテナ *内* でシェルを実行しています。シェルプロンプトが変わり、ゲートウェイのバージョン、ホストプラグインのディレクトリ、およびコンテナ内の現在のパスが表示されるはずです。たとえば、プロンプトは次のようになります。

```sh
[Kong-3.6.1:my-plugin:/kong]$
```

Pongoは {{site.base_gateway}} のライフサイクルおよびデータベースに役立つエイリアスをいくつか提供しています。初回実行時には、データベースを初期化して
{{site.base_gateway}}を起動する必要があります。
Pongoは、この一般的なタスクを実行するために`kms`エイリアスを提供します。

データベースの移行を実行し、{{site.base_gateway}}を開始します。

```sh
kms
```

{{site.base_gateway}} が起動されたことを示す成功メッセージが表示されます。

```sh
...
64 migrations processed
64 executed
Database is up-to-date

Kong started
```

前述したように、Pongoはプラグインのテストに役立つ開発ツールをいくつかインストールします。
これで、`curl`を使用して[Admin API](/gateway/{{page.release}}/admin-api/)にクエリを実行し、`jq`
で応答をフィルタリングしてプラグインがインストールされることを確認できるようになりました。

```sh
curl -s localhost:8001 | \
  jq '.plugins.available_on_server."my-plugin"'
```

プラグインのテーブルの情報と一致する応答が表示されるはずです。

```json
{
  "priority": 1000,
  "version": "0.0.1"
}
```

テスト環境が初期化されたら、プラグインコードを手動で実行できるようになります。

### プラグインを手動でテスト

プラグインをインストールすると、{{site.base_gateway}}エンティティを構成してプラグインの動作を呼出して検証できるようになりました。

{:.note}
> 
> **注** ：以下のそれぞれのAdmin APIへの`POST`リクエストでは、エンティティが正常に作成されたことを示す`HTTP/1.1 201 Created`応答を{{site.base_gateway}}から受信するはずです。

引き続き{{site.base_gateway}}コンテナのシェル内で、以下の新規サービスを追加します。

```sh
curl -i -s -X POST http://localhost:8001/services \
    --data name=example_service \
    --data url='http://httpbin.org'
```

カスタムプラグインを`example_service`サービスに関連付けます。

```sh
curl -is -X POST http://localhost:8001/services/example_service/plugins \
    --data 'name=my-plugin'
```

`example_service`を介してリクエストを送信するための新しいルートを追加します。

```sh
curl -i -X POST http://localhost:8001/services/example_service/routes \
    --data 'paths[]=/mock' \
    --data name=example_route
```

プラグインの構成が完了したので、{{site.base_gateway}}が`example_service`を介してリクエストをプロキシすると呼び出されます。
アップストリームからの返信を転送する前に、プラグインは、応答ヘッダーのリストに`X-MyPlugin`ヘッダーを追加する必要があります。

リクエストをプロキシして動作をテストし、`curl`に`-i`フラグのある応答ヘッダーを表示するよう要求します。

```sh
curl -i http://localhost:8000/mock/anything
```

`curl`は`HTTP/1.1 200 OK`を報告し、ゲートウェイからのレスポンスヘッダーを表示する必要があります。
ヘッダーのセットに`X-MyPlugin: response`が表示され、プラグインのロジックが呼び出されたことが示されます。

以下に例を示します。

```sh
HTTP/1.1 200 OK
Content-Type: application/json
Connection: keep-alive
Content-Length: 529
Access-Control-Allow-Credentials: true
Date: Tue, 12 Mar 2024 14:44:22 GMT
Access-Control-Allow-Origin: *
Server: gunicorn/19.9.0
X-MyPlugin: response
X-Kong-Upstream-Latency: 97
X-Kong-Proxy-Latency: 1
Via: kong/3.6.1
X-Kong-Request-Id: 8ab8c32c4782536592994514b6dadf55
```

先に進む前に、以下で {{site.base_gateway}} シェルを終了します。

```sh
exit
```

すぐに使い始めるには、Pongoシェルを使用してプラグインを手動で検証すると効果的です。
本番環境では、自動化テストのデプロイが必要になる可能性が高いでしょう。
テスト駆動開発（TDD）方法論も必要になるかもしれません。
では、Pongoをこの目的にも活用する方法をみていきましょう。

### テストの記述

Pongoは[Busted](https://lunarmodules.github.io/busted/) Luaテストフレームワークを使用して、
テストを自動化して実行するのをサポートします。プラグイン内
プロジェクトの場合、テストファイルは`spec/<plugin-name id="sl-md0000000">`ディレクトリの下にあります。
このプロジェクトの場合、これは先ほど作成した`spec/my-plugin`フォルダーです。

以下は、プラグインの現在の動作を検証するテストのコードの一覧です。
このコードをコピーして、`spec/my-plugin/01-integration_spec.lua`にある新しいファイルに配置します。

{{site.base_gateway}}によって提供されているテストの設計とテストヘルパーの詳細については、コードのコメントを参照してください。

```lua
-- Helper functions provided by Kong Gateway, see https://github.com/Kong/kong/blob/master/spec/helpers.lua
local helpers = require "spec.helpers"

-- matches our plugin name defined in the plugins's schema.lua
local PLUGIN_NAME = "my-plugin"

-- Run the tests for each strategy. Strategies include "postgres" and "off"
--   which represent the deployment topologies for Kong Gateway
for _, strategy in helpers.all_strategies() do

  describe(PLUGIN_NAME .. ": [#" .. strategy .. "]", function()
    -- Will be initialized before_each nested test
    local client

    setup(function()

      -- A BluePrint gives us a helpful database wrapper to
      --    manage Kong Gateway entities directly.
      -- This function also truncates any existing data in an existing db.
      -- The custom plugin name is provided to this function so it mark as loaded
      local blue_print = helpers.get_db_utils(strategy, nil, { PLUGIN_NAME })

      -- Using the BluePrint to create a test route, automatically attaches it
      --    to the default "echo" service that will be created by the test framework
      local test_route = blue_print.routes:insert({
        paths = { "/mock" },
      })

      -- Add the custom plugin to the test route
      blue_print.plugins:insert {
        name = PLUGIN_NAME,
        route = { id = test_route.id },
      }

      -- start kong
      assert(helpers.start_kong({
        -- use the custom test template to create a local mock server
        nginx_conf = "spec/fixtures/custom_nginx.template",
        -- make sure our plugin gets loaded
        plugins = "bundled," .. PLUGIN_NAME,
      }))

    end)

    -- teardown runs after its parent describe block
    teardown(function()
      helpers.stop_kong(nil, true)
    end)

    -- before_each runs before each child describe
    before_each(function()
      client = helpers.proxy_client()
    end)

    -- after_each runs after each child describe
    after_each(function()
      if client then client:close() end
    end)

    -- a nested describe defines an actual test on the plugin behavior
    describe("The response", function()

      it("gets the expected header", function()

        -- invoke a test request
        local r = client:get("/mock/anything", {})

        -- validate that the request succeeded, response status 200
        assert.response(r).has.status(200)

        -- now validate and retrieve the expected response header 
        local header_value = assert.response(r).has.header("X-MyPlugin")

        -- validate the value of that header
        assert.equal("response", header_value)

      end)
    end)
  end)
end
```

このテストコードで、Pongoはテストの自動化をサポートします。

### テストの実行

Pongoは`pongo run`コマンドを使用して自動化されたテストを実行できます。これを実行すると、
Pongoは依存コンテナがすでに実行されているかどうかを判断し、もしそうなら
それを使用します。テストライブラリは、テストを実行する間に既存データの切り捨てを処理します。

テストランを実行します。

```sh
pongo run
```

次のような成功レポートが表示されます。

```sh
[pongo-INFO] auto-starting the test environment, use the 'pongo down' action to stop it
Kong version: 3.6.1

[==========] Running tests from scanned files.
[----------] Global test environment setup.
[----------] Running tests from /kong-plugin/spec/my-plugin/01-integration_spec.lua
[----------] Running tests from /kong-plugin/spec/my-plugin/01-integration_spec.lua
[ RUN      ] /kong-plugin/spec/my-plugin/01-integration_spec.lua:63: my-plugin: [#postgres] The response gets a 'X-MyPlugin' header
[       OK ] /kong-plugin/spec/my-plugin/01-integration_spec.lua:63: my-plugin: [#postgres] The response gets a 'X-MyPlugin' header (6.59 ms)
[ RUN      ] /kong-plugin/spec/my-plugin/01-integration_spec.lua:63: my-plugin: [#off] The response gets a 'X-MyPlugin' header
[       OK ] /kong-plugin/spec/my-plugin/01-integration_spec.lua:63: my-plugin: [#off] The response gets a 'X-MyPlugin' header (4.76 ms)
[----------] 2 tests from /kong-plugin/spec/my-plugin/01-integration_spec.lua (23022.12 ms total)

[----------] Global test environment teardown.
[==========] 2 tests from 1 test file ran. (23022.80 ms total)
[  PASSED  ] 2 tests.
```

Pongoは継続的統合（CI）システムの一環としても実行できます。詳細については、[リポジトリのドキュメント](https://github.com/Kong/kong-pongo?tab=readme-ov-file#setting-up-ci)
を参照してください。

次の手順
----

プロジェクトのセットアップと自動テストが完了したら、次の章では、プラグインに設定可能な値を追加する手順を説明します。

