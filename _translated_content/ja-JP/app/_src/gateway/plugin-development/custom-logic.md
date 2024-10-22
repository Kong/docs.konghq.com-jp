---
title: "カスタムロジックの実装"
book: "plugin_dev"
chapter: 4
---
{:.note}
> 
> **注** ：この章は、[Lua](http://www.lua.org/)に精通していることを前提と
> しています。

{{site.base_gateway}}プラグインを使うと、{{site.base_gateway}}でプロキシされる
リクエスト/レスポンスまたはTCPストリーム接続のライフサイクルにおいて、
いくつかのエントリポイントでカスタムロジックを（Luaに）注入できます。そのためには、ファイル
`kong.plugins.<plugin_name id="sl-md0000000">.handler`は、1つ以上のあらかじめ決められた
名前の関数のテーブルを返さなければなりません。この関数はトラフィックを
処理するときに、様々なフェーズで{{site.base_gateway}}によって呼び出されます。

{% if_version gte: 3.4.x %}
最初に取るパラメータは常に`self`です。`init_worker`と`configure`を除く
すべての関数は、プラグイン構成を含むテーブルという第2のパラメータを
受け取ることができます。`configure`は特定のプラグインに関するすべての構成の配列を
受け取ります。
{% endif_version %}

{% if_version lte:3.3.x %}
最初に取るパラメータは常に`self`です。`init_worker`を除くすべての関数は、
プラグイン構成を含むテーブルという第2のパラメータを受け取ることができます。
{% endif_version %}

モジュール
-----

    kong.plugins.<plugin_name id="sl-md0000000">.handler

使用可能なコンテキスト
-----------

`handler.lua` ファイルで次のいずれかの関数を定義すると、{{site.base_gateway}} の実行ライフサイクルのさまざまなエントリポイントにカスタムロジックが実装されます。

* **[HTTPモジュール](https://github.com/openresty/lua-nginx-module)** *は、HTTP/HTTPSリクエスト用に記述されるプラグインで使用されます* {% if_version lte: 3.3.x %}

|                            機能名                             |     Kong Phase      |                                                                                        Nginx Directives                                                                                        |         リクエストプロトコル          |                                                                                                                  説明                                                                                                                  |
|------------------------------------------------------------|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `init_worker`                                              | `init_worker`       | [`init_worker_by_*`](https://github.com/openresty/lua-nginx-module#init_worker_by_lua_block)                                                                                                   | \*                         | すべてのNginxワーカープロセスの起動時に実行されます。                                                                                                                                                                                                        |
| `certificate`                                              | `certificate`       | [`ssl_certificate_by_*`](https://github.com/openresty/lua-nginx-module#ssl_certificate_by_lua_block)                                                                                           | `https`、`grpcs`、`wss`       | SSLハンドシェイクのSSL証明書提供フェーズ中に実行されます。                                                                                                                                                                                                     |
| `rewrite`                                                  | `rewrite`           | [`rewrite_by_*`](https://github.com/openresty/lua-nginx-module#rewrite_by_lua_block)                                                                                                           | \*                         | リライトフェーズハンドラとして、クライアントからリクエストを受信するたびに実行されます。<br>このフェーズでは、`Service`も`Consumer`も特定されていないため、このハンドラは、プラグインがグローバルプラグインとして構成されている場合にのみ実行されます。                                                                                              |
| `access`                                                   | `access`            | [`access_by_*`](https://github.com/openresty/lua-nginx-module#access_by_lua_block)                                                                                                             | `http(s)`、`grpc(s)`、`ws(s)` | クライアントからのすべてのリクエストに対して、アップストリームサービスにプロキシされる前に実行されます。                                                                                                                                                                                 |
| `response`                                                 | `response`          | [`header_filter_by_*`](https://github.com/openresty/lua-nginx-module#header_filter_by_lua_block), [`body_filter_by_*`](https://github.com/openresty/lua-nginx-module#body_filter_by_lua_block) | `http(s)`、`grpc(s)`         | `header_filter()` と `body_filter()`の両方を置き換えます。アップストリームサービスから応答全体が受信された後に、その一部をクライアントに送信する前に実行されます。                                                                                                                                   |
| `header_filter`                                            | `header_filter`     | [`header_filter_by_*`](https://github.com/openresty/lua-nginx-module#header_filter_by_lua_block)                                                                                               | `http(s)`、`grpc(s)`         | アップストリームサービスからすべての応答ヘッダーバイトが受信されたときに実行されます。                                                                                                                                                                                          |
| `body_filter`                                              | `body_filter`       | [`body_filter_by_*`](https://github.com/openresty/lua-nginx-module#body_filter_by_lua_block)                                                                                                   | `http(s)`、`grpc(s)`         | アップストリームサービスから受信した応答本文の各チャンクに対して実行されます。応答はクライアントにストリーミングされるので、バッファサイズを超えて、チャンクごとにストリーミングされる可能性があります。応答が大きい場合、この関数は複数回呼び出される可能性があります。詳細については、 [lua\-nginx\-module の](https://github.com/openresty/lua-nginx-module)ドキュメントを参照してください。 |
| `ws_handshake`                                             | `ws_handshake`      | [`access_by_*`](https://github.com/openresty/lua-nginx-module#access_by_lua_block)                                                                                                             | `ws(s)`                     | WebSocket ハンドシェイクが完了する直前に、WebSocket サービスへのすべてのリクエストに対して実行されます。                                                                                                                                                                       |
| `ws_client_frame` <span class="badge enterprise"></span>   | `ws_client_frame`   | [`content_by_*`](https://github.com/openresty/lua-nginx-module#content_by_lua_block)                                                                                                           | `ws(s)`                     | クライアントから受信したWebSocketメッセージごとに実行されます。                                                                                                                                                                                                 |
| `ws_upstream_frame` <span class="badge enterprise"></span> | `ws_upstream_frame` | [`content_by_*`](https://github.com/openresty/lua-nginx-module#content_by_lua_block)                                                                                                           | `ws(s)`                     | アップストリームサービスから受信したWebSocketメッセージごとに実行されます。                                                                                                                                                                                           |
| `log`                                                      | `log`               | [`log_by_*`](https://github.com/openresty/lua-nginx-module#log_by_lua_block)                                                                                                                   | `http(s)`、`grpc(s)`         | 最後のレスポンスバイトがクライアントに送信されたときに実行されます。                                                                                                                                                                                                   |
| `ws_close` <span class="badge enterprise"></span>          | `ws_close`          | [`log_by_*`](https://github.com/openresty/lua-nginx-module#log_by_lua_block)                                                                                                                   | `ws(s)`                     | WebSocket接続が終了した後に実行されます。                                                                                                                                                                                                            |

{:.note}
> 
> **注：** モジュールが`response`関数を実装している場合、[`kong.service.request.enable_buffering()`関数](/gateway/{{page.release}}/plugin-development/pdk/kong.service.request/#kongservicerequestenable_buffering)が呼び出されているため、{{site.base_gateway}}は自動的に「バッファリングされたプロキシ」モードをアクティブにします。現在のNginxの制限により、これはHTTP/2またはgRPCアップストリームでは機能しません。

予期しない動作変更を減らすため、プラグインが `response` と `header_filter` または `body_filter` の両方を実装している場合、{{site.base_gateway}} は起動しません。

* **[ストリームモジュール](https://github.com/openresty/stream-lua-nginx-module)** *は、TCPおよびUDPストリーム接続用に記述されたプラグインに使用されます* 

|      機能名      |  Kong Phase   |                                           Nginx Directives                                           |                説明                |
|---------------|---------------|------------------------------------------------------------------------------------------------------|----------------------------------|
| `init_worker` | `init_worker` | [`init_worker_by_*`](https://github.com/openresty/lua-nginx-module#init_worker_by_lua_block)         | すべてのNginxワーカープロセスの起動時に実行されます。    |
| `preread`     | `preread`     | [`preread_by_*`](https://github.com/openresty/stream-lua-nginx-module#preread_by_lua_block)          | 接続ごとに 1 回実行されます。                 |
| `log`         | `log`         | [`log_by_*`](https://github.com/openresty/lua-nginx-module#log_by_lua_block)                         | 接続が閉じられた後、接続ごとに 1 回実行されます。       |
| `certificate` | `certificate` | [`ssl_certificate_by_*`](https://github.com/openresty/lua-nginx-module#ssl_certificate_by_lua_block) | SSLハンドシェイクのSSL証明書提供フェーズ中に実行されます。 |

{% endif_version %}

{% if_version gte: 3.4.x %}
\| Function name       \| Kong Phase            \| Nginx Directives         \| Request Protocol              \| Description
\|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\|\-\-\-\-\-\-\-\-\-\-\-\-
\| `init_worker`       \| `init_worker`         \| [`init_worker_by_*`](https://github.com/openresty/lua-nginx-module#init_worker_by_lua_block)     \| \*                             \| Executed upon every Nginx worker process's startup.
\| `configure`         \| `init_worker`/`timer` \| [`init_worker_by_*`](https://github.com/openresty/lua-nginx-module#init_worker_by_lua_block)     \| \*                             \| Executed every time the Kong plugin iterator is rebuilt \(after changes to configure plugins\).
\| `certificate`       \| `certificate`         \| [`ssl_certificate_by_*`](https://github.com/openresty/lua-nginx-module#ssl_certificate_by_lua_block) \| `https`, `grpcs`, `wss`       \| Executed during the SSL certificate serving phase of the SSL handshake.
\| `rewrite`           \| `rewrite`             \| [`rewrite_by_*`](https://github.com/openresty/lua-nginx-module#rewrite_by_lua_block)         \| \*                             \| Executed for every request upon its reception from a client as a rewrite phase handler. <br> In this phase, neither the `Service` nor the `Consumer` have been identified, hence this handler will only be executed if the plugin was configured as a global plugin.
\| `access`            \| `access`              \| [`access_by_*`](https://github.com/openresty/lua-nginx-module#access_by_lua_block)          \| `http(s)`, `grpc(s)`, `ws(s)` \| Executed for every request from a client and before it is being proxied to the upstream service.
\| `response`          \| `response`            \| [`header_filter_by_*`](https://github.com/openresty/lua-nginx-module#header_filter_by_lua_block), [`body_filter_by_*`](https://github.com/openresty/lua-nginx-module#body_filter_by_lua_block) \| `http(s)`, `grpc(s)` \| Replaces both `header_filter()` and `body_filter()`. Executed after the whole response has been received from the upstream service, but before sending any part of it to the client.
\| `header_filter`     \| `header_filter`       \| [`header_filter_by_*`](https://github.com/openresty/lua-nginx-module#header_filter_by_lua_block)   \| `http(s)`, `grpc(s)`          \| Executed when all response headers bytes have been received from the upstream service.
\| `body_filter`       \| `body_filter`         \| [`body_filter_by_*`](https://github.com/openresty/lua-nginx-module#body_filter_by_lua_block)     \| `http(s)`, `grpc(s)`          \| Executed for each chunk of the response body received from the upstream service. Since the response is streamed back to the client, it can exceed the buffer size and be streamed chunk by chunk. This function can be called multiple times if the response is large. See the [lua\-nginx\-module](https://github.com/openresty/lua-nginx-module) documentation for more details.
\| `ws_handshake`      \| `ws_handshake`     \| [`access_by_*`](https://github.com/openresty/lua-nginx-module#access_by_lua_block)          \| `ws(s)`                       \| Executed for every request to a WebSocket service just before completing the WebSocket handshake.
\| `ws_client_frame` <span class="badge enterprise"></span> \| `ws_client_frame`  \| [`content_by_*`](https://github.com/openresty/lua-nginx-module#content_by_lua_block)         \| `ws(s)`                       \| Executed for each WebSocket message received from the client.
\| `ws_upstream_frame` <span class="badge enterprise"></span> \| `ws_upstream_frame`\| [`content_by_*`](https://github.com/openresty/lua-nginx-module#content_by_lua_block)         \| `ws(s)`                       \| Executed for each WebSocket message received from the upstream service.
\| `log`               \| `log`              \| [`log_by_*`](https://github.com/openresty/lua-nginx-module#log_by_lua_block)             \| `http(s)`, `grpc(s)`          \| Executed when the last response byte has been sent to the client.
\| `ws_close` <span class="badge enterprise"></span> \| `ws_close`         \| [`log_by_*`](https://github.com/openresty/lua-nginx-module#log_by_lua_block)             \| `ws(s)`                       \| Executed after the WebSocket connection has been terminated.

{:.note}
> 
> **注：** モジュールが`response`関数を実装している場合、[`kong.service.request.enable_buffering()`関数](/gateway/{{page.release}}/plugin-development/pdk/kong.service.request/#kongservicerequestenable_buffering)が呼び出されているため、{{site.base_gateway}}は自動的に「バッファリングされたプロキシ」モードをアクティブにします。現在のNginxの制限により、これはHTTP/2またはgRPCアップストリームでは機能しません。

予期しない動作変更を減らすため、プラグインが `response` と `header_filter` または `body_filter` の両方を実装している場合、{{site.base_gateway}} は起動しません。

* **[ストリームモジュール](https://github.com/openresty/stream-lua-nginx-module)** *は、TCPおよびUDPストリーム接続用に記述されたプラグインに使用されます* 

|      機能名      |      Kong Phase       |                                           Nginx Directives                                           |                      説明                       |
|---------------|-----------------------|------------------------------------------------------------------------------------------------------|-----------------------------------------------|
| `init_worker` | `init_worker`         | [`init_worker_by_*`](https://github.com/openresty/lua-nginx-module#init_worker_by_lua_block)         | すべてのNginxワーカープロセスの起動時に実行されます。                 |
| `configure`   | `init_worker`/`timer` | [`init_worker_by_*`](https://github.com/openresty/lua-nginx-module#init_worker_by_lua_block)         | Kongプラグインイテレータが再構築されるたびに（プラグイン構成の変更後に）実行されます。 |
| `preread`     | `preread`             | [`preread_by_*`](https://github.com/openresty/stream-lua-nginx-module#preread_by_lua_block)          | 接続ごとに 1 回実行されます。                              |
| `log`         | `log`                 | [`log_by_*`](https://github.com/openresty/lua-nginx-module#log_by_lua_block)                         | 接続が閉じられた後、接続ごとに 1 回実行されます。                    |
| `certificate` | `certificate`         | [`ssl_certificate_by_*`](https://github.com/openresty/lua-nginx-module#ssl_certificate_by_lua_block) | SSLハンドシェイクのSSL証明書提供フェーズ中に実行されます。              |

`init_worker` と `configure` を除くすべての関数は、{{site.base_gateway}} の呼び出し時に指定されたパラメータ（プラグインの構成）を受け取ります。このパラメータは Lua テーブルであり、プラグインのスキーマ（`schema.lua` モジュールに記述）に従ってユーザーが定義した値が含まれます。プラグインのスキーマについて、[次の章]({{page.book.next.url}})で詳しく説明します。`configure` は、特定のプラグインで有効になっているすべてのプラグイン構成の配列で呼び出されます（プラグインのアクティブな構成がない場合は、`nil` が渡されます）。`init_worker` と `configure` はリクエストまたはフレームの外側で実行されますが、残りのフェーズは受信したリクエストやフレームにバインドされます。

UDP ストリームには実際の接続がないことに注意してください。{{site.base_gateway}} は
同一の起点、宛先のホストとポートを持つ全パケットを単一接続
とみなします。パケットがない状態が設定可能な時間経過すると、接続は
閉じられたとみなされ、 `log`関数が実行されます。

{:.note}
> 
> `configure`ハンドラはKong 3\.5で追加され、3\.4 LTSにバックポートされました。現在、この新しいフェーズに対するフィードバックを求めており、その署名が将来変更される可能性がわずかにあります。{% endif_version %}

handler.luaの仕様書
---------------


{{site.base_gateway}} はリクエストを **フェーズ** で処理します。プラグインは、リクエストがプロキシされている間に各フェーズが実行されるときに、{{site.base_gateway}} によってアクティブ化されるコードです。

{% if_version gte:3.4.x %}
フェーズでできることは限定されています。たとえば、`init_worker` フェーズは `config` パラメータにアクセスできません。これは、kong が各ワーカーを初期化しているときにその情報が利用できないためです。一方、`configure` にはプラグインのすべてのアクティブな構成が渡されます（設定されていない場合は`nil`）。{% endif_version %}
{% if_version lte: 3.3.x %}
フェーズでできることは限定されています。たとえば、`init_worker` フェーズは `config` パラメータにアクセスできません。これは、kongが各ワーカーを初期化しているときにその情報が利用できないためです。{% endif_version %}
プラグインの `handler.lua` は、各フェーズで実行する必要がある関数を含むテーブルを返す必要があります。


{{site.base_gateway}} HTTP およびストリーム トラフィックを処理できます。フェーズには、
HTTPトラフィックを処理するときのみ実行されるもの、ストリームを処理するときに実行されるもの、
また、`init_worker`や`log`などのように、両種のトラフィックによって呼び出されるものがあります。

関数に加えて、プラグインは次の2つのフィールドを定義する必要があります。

* `VERSION`情報フィールドであり、 {{site.base_gateway}}によって直接使用されるものではありません。通常の場合、 プラグインのRockspecバージョンで定義されているバージョン（存在する場合）と一致します。
* `PRIORITY`各フェーズを実行する前にプラグインをソートするために使用されます。 優先度の高いプラグインが最初に実行されます。このフィールドの詳細については、下記の [プラグイン実行の順序](#plugins-execution-order)を参照してください。

次のサンプルファイル、`handler.lua`
は、HTTPとストリームトラフィックの両方で使用可能なすべてのフェーズでカスタム関数を定義します。このファイルには、フェーズが呼び出されるたびにログにメッセージを記述する以外の機能はありません。
注
プラグインはすべてのフェーズに機能を提供する必要はありません。

```lua
local CustomHandler = {
  VERSION  = "1.0.0",
  PRIORITY = 10,
}

function CustomHandler:init_worker()
  -- Implement logic for the init_worker phase here (http/stream)
  kong.log("init_worker")
end

{% if_version gte:3.4.x -%}
function CustomHandler:configure(configs)
  -- Implement logic for the configure phase here
  --(called whenever there is change to any of the plugins)
  kong.log("configure")
end
{%- endif_version %}

function CustomHandler:preread(config)
  -- Implement logic for the preread phase here (stream)
  kong.log("preread")
end

function CustomHandler:certificate(config)
  -- Implement logic for the certificate phase here (http/stream)
  kong.log("certificate")
end

function CustomHandler:rewrite(config)
  -- Implement logic for the rewrite phase here (http)
  kong.log("rewrite")
end

function CustomHandler:access(config)
  -- Implement logic for the access phase here (http)
  kong.log("access")
end

function CustomHandler:ws_handshake(config)
  -- Implement logic for the WebSocket handshake here
  kong.log("ws_handshake")
end

function CustomHandler:header_filter(config)
  -- Implement logic for the header_filter phase here (http)
  kong.log("header_filter")
end

function CustomHandler:ws_client_frame(config)
  -- Implement logic for WebSocket client messages here
  kong.log("ws_client_frame")
end

function CustomHandler:ws_upstream_frame(config)
  -- Implement logic for WebSocket upstream messages here
  kong.log("ws_upstream_frame")
end

function CustomHandler:body_filter(config)
  -- Implement logic for the body_filter phase here (http)
  kong.log("body_filter")
end

function CustomHandler:log(config)
  -- Implement logic for the log phase here (http/stream)
  kong.log("log")
end

function CustomHandler:ws_close(config)
  -- Implement logic for WebSocket post-connection here
  kong.log("ws_close")
end

-- return the created table, so that Kong can execute it
return CustomHandler
```

上記の例では、最初のパラメータとして`self`を取る関数に関して、Luaの`:`短縮構文を使用していることに注意してください。`access`関数の同等な非省略形バージョンは次のようになります。

```lua
function CustomHandler.access(self, config)
  -- Implement logic for the access phase here (http)
  kong.log("access")
end
```

プラグインのロジックをすべて`handler.lua`ファイル内で定義する必要はありません。
これは複数のLuaファイル（ *モジュール* とも呼ばれます）に分割できます。
`handler.lua`モジュールは`require`を使用して、プラグインに他のモジュールを含めることができます。

たとえば、次のプラグインは機能を3つのファイルに分割します。
`access.lua`と`body_filter.lua`は関数を返します。これらは`handler.lua`と
同じフォルダー内にあり、プラグインの構築に必要で、使用します。

```lua
-- handler.lua
local access = require "kong.plugins.my-custom-plugin.access"
local body_filter = require "kong.plugins.my-custom-plugin.body_filter"

local CustomHandler = {
  VERSION  = "1.0.0",
  PRIORITY = 10
}

CustomHandler.access = access
CustomHandler.body_filter = body_filter

return CustomHandler
```

```lua
-- access.lua
return function(self, config)
  kong.log("access phase")
end
```

```lua
-- body_filter.lua
return function(self, config)
  kong.log("body_filter phase")
end
```

実際のハンドラーコードの例については、[Key\-Authプラグインのソースコード](https://github.com/Kong/kong/blob/master/kong/plugins/key-auth/handler.lua)
を参照してください。

### BasePlugin モジュールからの移行

`BasePlugin`モジュールは非推奨となり、
{{site.base_gateway}} から削除されました。このモジュールを使用する古いプラグインがある場合は、次のセクションを置き換えます。

```lua
--  DEPRECATED --
local BasePlugin = require "kong.plugins.base_plugin"
local CustomHandler = BasePlugin:extend()
CustomHandler.VERSION  = "1.0.0"
CustomHandler.PRIORITY = 10
```

現在の同等のもの：

```lua
local CustomHandler = {
  VERSION  = "1.0.0",
  PRIORITY = 10,
}
```

`:new()`メソッドを追加したり、任意の`CustomHandler.super.XXX:(self)`
メソッドを呼び出したりする必要はありません。

WebSocketプラグイン開発
----------------

{:.badge .enterprise}

### ハンドラ関数

`ws`または`wss`プロトコルを使用するサービスへのハンドラ関数のリクエストは、通常のhttpリクエストとは異なるパスをプロキシ経由で経由します。したがって、それらのプラグインを開発する際に考慮しなければならない動作にはいくつかの違いがあります。

次のハンドラはWebSocketサービスでは実行 *されません* 。

* `access`
* `response`
* `header_filter`
* `body_filter`
* `log`

以下のハンドラは WebSocket サービスに *固有の* ハンドラです。

* `ws_handshake`
* `ws_client_frame`
* `ws_upstream_frame`
* `ws_close`

以下のハンドラは、WebSocket サービス *と* 非 WebSocket サービスの両方で実行されます。

* `init_worker` {% if_version gte:3.4.x -%}
* `configure` {% endif_version -%}
* `certificate`（TLS/SSLリクエストのみ）
* `rewrite`

これらの違いがあっても、WebSocketサービスと非WebSocketサービスの両方をサポートするプラグインを開発することは可能です。以下に例を示します。

```lua
-- handler.lua
--
-- I am a plugin that implements both WebSocket and non-WebSocket handlers.
--
-- I can be enabled for ws/wss services, http/https/grpc/grpcs services, or
-- even as global plugin.
local MultiProtoHandler = {
  VERSION = "0.1.0",
  PRIORITY = 1000,
}

function MultiProtoHandler:access()
  kong.ctx.plugin.request_type = "non-WebSocket"
end

function MultiProtoHandler:ws_handshake()
  kong.ctx.plugin.request_type = "WebSocket"
end


function MultiProtoHandler:log()
  kong.log("finishing ", kong.ctx.plugin.request_type, " request")
end

-- the `ws_close` handler for this plugin does not implement any WebSocket-specific
-- business logic, so it can simply be aliased to the `log` handler
MultiProtoHandler.ws_close = MultiProtoHandler.log

return MultiProtoHandler
```

上記のように、 `log`ハンドラーと`ws_close`ハンドラーは互いに並行しています。多くの場合、
追加コードを書かずに一方を他方に
エイリアス化することができます。この点に関しては`access`と`ws_handshake`ハンドラも非常に
よく似ています。注目すべき違いは、それぞれの文脈においてどのPDK機能が利用可能か、
または利用できないかという点です。たとえば、 `kong.request.get_body()` PDK関数は
`access`ハンドラでは基本的にこの種のリクエストのと互換性がないため
使用しないでください。

### 非WebSocketサービスへのWebSocketリクエスト

WebSocketトラフィックは、http/httpsサービス経由でプロキシされる場合には非WebSocketリクエストとして扱われます。
そのため、httpハンドラ（`access`、`header_filter`など）が実行され、WebSocketハンドラ（`ws_handshake`、`ws_close`など）は実行 *されません* 。

プラグイン開発キット
----------

これらのフェーズで実装されるロジックは、リクエスト/応答オブジェクトやコアコンポーネントと相互作用しなければならない可能性が高いです（例：キャッシュやデータベースにアクセスするなど）。{{site.base_gateway}}は、このような[プラグイン開発キット](/gateway/{{page.release}}/plugin-development/pdk)（または「PDK」）を提供します。
それは、プラグインが様々なゲートウェイ操作を実行するために使用できるLua関数および変数のセットと、{{site.base_gateway}} の将来のリリースとの互換性が保証されているためです。

{{site.base_gateway}}とやりとりする必要があるロジックを実装しようとしている場合
（例えば、リクエストヘッダーの取得、プラグインからのレスポンスの生成、
いくつかのエラーまたはデバッグ情報のロギングなど）については、[プラグイン開発キットリファレンス](/gateway/{{page.release}}/plugin-development/pdk)にご相談ください。

プラグインの実行順序
----------

一部のプラグインは、一部の操作を実行するために他のプラグインの実行に依存する場合があります。たとえば、コンシューマの ID に依存するプラグインは、認証プラグインの **後** に実行する必要があります。これを考慮し、{{site.base_gateway}} はプラグインの実行間の **優先順位** を定義して、順序が尊重されるようにします。

プラグインの優先順位は、返されたハンドラテーブルで番号を受け入れるプロパティを
経由して、設定することができます。

```lua
CustomHandler.PRIORITY = 10
```

優先度が高ければ高いほど、他のプラグインのフェーズに対して、お使いのプラグインのフェーズがより早く実行されます
（ `:access()`、`:log()` など）。

### Kong プラグイン

{{site.base_gateway}} にバンドルされているすべてのプラグインには静的優先順位があります。
これは、`ordering` オプションを使用して動的に調整できます。詳細については、[プラグインの動的順序付け](/gateway/{{page.release}}/kong-enterprise/plugin-ordering/)を参照してください。

{% navtabs %}
{% navtab OSS %}

以下のリストには、オープンソース
{{site.base_gateway}}にバンドルされているすべての
プラグインが含まれています。

{:.note}
> 
> **注：** Correlation IDプラグインの優先順位は、実行しているのがオープン
> ソースモードか無料モードかによって変わります。
> 無料モードは{{site.ee_product_name}}パッケージを使用します。
> このプラグインの正しい優先順位を確認するには、 **エンタープライズ** タブに切り替えてください。

バンドルされたプラグインの現在の実行順序は次のとおりです。

{% plugins_priority_table oss %}

{% endnavtab %}
{% navtab Enterprise %}
次のリストには、{{site.ee_product_name}}
サブスクリプションにバンドルされているすべてのプラグインが含まれています。この優先順位は、無料モードで実行されているプラグイン（{{site.ee_product_name}}
パッケージを使用）にも適用されます。

バンドルされたプラグインの現在の実行順序は次のとおりです。

{% plugins_priority_table enterprise %}

{% endnavtab %}
{% endnavtabs %}