---
##
#  WARNING: this file was auto-generated by a script.
#  DO NOT edit this file directly. Instead, send a pull request to change
#  https://github.com/Kong/kong/tree/master/kong/pdk
#  or its associated files
##

title: "kong.router"
pdk: true
toc: true
source_url: "https://github.com/Kong/kong/tree/master/kong/pdk/router.lua"
---
ルーターモジュール。

リクエストのルーティングプロパティにアクセスするための関数のセット。

kong.router.get\_route\(\)
-----------------------------

現在の `route` エンティティを返します。リクエストはこの
ルートと照合されます。

**フェーズ** 

* アクセス、ヘッダーフィルター、レスポンス、ボディフィルター、ログ

**リターン** 

* `table`: `route`エンティティ。

**使用法** 

```lua
local route = kong.router.get_route()
local protocols = route.protocols
```

kong.router.get\_service\(\)
-------------------------------

現在の `service` エンティティを返します。リクエストはこの
アップストリームサービスを対象としています。

**フェーズ** 

* アクセス、ヘッダーフィルター、レスポンス、ボディフィルター、ログ

**リターン** 

* `table`: `service`エンティティ。

**使用法** 

```lua
if kong.router.get_service() then
  -- routed by route & service entities
else
  -- routed by a route without a service
end
```

