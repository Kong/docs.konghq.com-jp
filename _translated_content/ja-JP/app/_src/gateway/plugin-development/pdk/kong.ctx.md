---
##
#  WARNING: this file was auto-generated by a script.
#  DO NOT edit this file directly. Instead, send a pull request to change
#  https://github.com/Kong/kong/tree/master/kong/pdk
#  or its associated files
##

title: "kong.ctx"
pdk: true
toc: true
source_url: "https://github.com/Kong/kong/tree/master/kong/pdk/ctx.lua"
---
現在のリクエストのコンテキストデータ。

kong.ctx.shared
---------------

現在のリクエストと同じ有効期間を持つテーブル。このテーブルは、すべてのプラグイン間で共有されます。これは、特定のリクエスト内の複数のプラグイン間でデータを共有するために使用できます。

このテーブルはリクエストのコンテキスト内でのみ関連し、Lua モジュールのトップレベルのチャンクからはアクセスできません。代わりに、リクエストフェーズ内でのみアクセス可能であり、プラグインインターフェイスの `rewrite`、`access`、`header_filter`、`response`、`body_filter`、`log`、および `preread` の各フェーズで表されます。これらの関数（およびその呼び出し先）でこのテーブルにアクセスすることは問題ありません。

プラグインによってこのテーブルに挿入された値は、他のすべてのプラグインから参照できます。このテーブル内の値を操作するときは、名前の競合によってデータが上書きされる可能性があるため、注意してください。

**フェーズ** 

* rewrite, access, header\_filter, response, body\_filter, log, preread

**使用方法** 

```lua
-- Two plugins A and B, and if plugin A has a higher priority than B's
-- (it executes before B):

-- plugin A handler.lua
function plugin_a_handler:access(conf)
  kong.ctx.shared.foo = "hello world"

  kong.ctx.shared.tab = {
    bar = "baz"
  }
end

-- plugin B handler.lua
function plugin_b_handler:access(conf)
  kong.log(kong.ctx.shared.foo) -- "hello world"
  kong.log(kong.ctx.shared.tab.bar) -- "baz"
end
```

kong.ctx.plugin
---------------

現在のリクエストと同じ有効期間を持つテーブル。`kong.ctx.shared`とは異なり、このテーブルはプラグイン間で共有 **されません** 。代わりに、現在のプラグインインスタンスに対してのみ表示されます。たとえば、Rate Limitingプラグインの複数のインスタンスが異なるサービスで構成されている場合、各インスタンスはリクエストごとに独自のテーブルを持ちます。

このテーブルは名前空間が設定されているため、プラグインで使用するには`kong.ctx.shared`
も安全です。名前空間があると、名前が競合して複数のプラグインが知らぬ間に相互のデータを上書きする可能性がなくなります。

このテーブルはリクエストのコンテキスト内でのみ関連し、Lua モジュールのトップレベルのチャンクからはアクセスできません。代わりに、リクエストフェーズ内でのみアクセス可能であり、プラグインインターフェイスの `rewrite`、`access`、`header_filter`、`body_filter`、`log`、および `preread` の各フェーズで表されます。これらの関数（およびその呼び出し先）でこのテーブルにアクセスすることは問題ありません。

プラグインによってこのテーブルに挿入された値は、このプラグインの
インスタンスの成功フェーズでのみ表示されます。

**フェーズ** 

* rewrite, access, header\_filter, response, body\_filter, log, preread

**使用方法** 

```lua
-- plugin handler.lua

-- For example, if a plugin wants to
-- save some value for post-processing during the `log` phase:

function plugin_handler:access(conf)
  kong.ctx.plugin.val_1 = "hello"
  kong.ctx.plugin.val_2 = "world"
end

function plugin_handler:log(conf)
  local value = kong.ctx.plugin.val_1 .. " " .. kong.ctx.plugin.val_2

  kong.log(value) -- "hello world"
end
```

