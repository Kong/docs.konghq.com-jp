---
title: "テストの記述"
book: "plugin_dev"
chapter: 10
---
プラグインに真剣に取り組んでいるなら、プラグイン用のテストを書くことを希望しているかもしれません。Luaのユニットテストは簡単で、[さまざまなテストフレームワーク](http://lua-users.org/wiki/UnitTesting)が利用可能です。ただし、統合テストの記述も行いたい場合は、Kongにお任せください。お手伝いいたします。

統合テストの記述
--------

Kongが推奨するテストフレームは、[resty\-cli](https://github.com/lunarmodules/busted)解析を使用した[busted](https://github.com/openresty/resty-cli)実行ですが、別のものを自由に使用できます。Kongリポジトリでは、busted実行可能ファイルは`bin/busted`にあります。

Kongはテストスイート（`spec.helpers`）のLuaから開始および停止するためのヘルパーを提供します。このヘルパーは、テストを実行する前にデータストアにフィクスチャを挿入したり、フィクスチャを削除したり、その他のさまざまなヘルパーを提供する方法も提供します。

独自のリポジトリにプラグインを作成する場合は、Kongテストフレームワークが
リリースされるまで、次のファイルをコピーする必要があります。

* `bin/busted`: resty\-cli インタープリタで実行されている busted 実行ファイル
* `spec/helpers.lua`: Kongのバストを開始/停止するヘルパー関数
* `spec/kong_tests.conf`: ヘルパーモジュールを使用してテスト Kong インスタンスを実行するための構成ファイル

`spec.helpers`モジュールが`LUA_PATH`で利用可能であると仮定すると、
busted では次の Lua コードを使用して Kong を起動および停止できます。

```lua
local helpers = require "spec.helpers"

for _, strategy in helpers.each_strategy() do
  describe("my plugin", function()

    local bp = helpers.get_db_utils(strategy)

    setup(function()
      local service = bp.services:insert {
        name = "test-service",
        host = "httpbin.org"
      }

      bp.routes:insert({
        hosts = { "test.com" },
        service = { id = service.id }
      })

      -- start Kong with your testing Kong configuration (defined in "spec.helpers")
      assert(helpers.start_kong( { plugins = "bundled,my-plugin" }))

      admin_client = helpers.admin_client()
    end)

    teardown(function()
      if admin_client then
        admin_client:close()
      end

      helpers.stop_kong()
    end)

    before_each(function()
      proxy_client = helpers.proxy_client()
    end)

    after_each(function()
      if proxy_client then
        proxy_client:close()
      end
    end)

    describe("thing", function()
      it("should do thing", function()
        -- send requests through Kong
        local res = proxy_client:get("/get", {
          headers = {
            ["Host"] = "test.com"
          }
        })

        local body = assert.res_status(200, res)

        -- body is a string containing the response
      end)
    end)
  end)
end
```

テスト用Kong構成ファイルでは、Kongはそのプロキシがポート
9000（HTTP）、9443（HTTPS）、およびポート9001の
Admin APIをリッスンする状態で実行されます。

実際の例については、[Key\-Authプラグインの仕様](https://github.com/Kong/kong/tree/master/spec/03-plugins/09-key-auth)を参照してください。

