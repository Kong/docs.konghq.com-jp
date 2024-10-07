---
##
#  WARNING: this file was auto-generated by a script.
#  DO NOT edit this file directly. Instead, send a pull request to change
#  https://github.com/Kong/kong/tree/master/kong/pdk
#  or its associated files
##

title: "kong.response"
pdk: true
toc: true
source_url: "https://github.com/Kong/kong/tree/master/kong/pdk/response.lua"
---
クライアント応答モジュール。

ダウンストリームの応答モジュールには、クライアント（ダウンストリーム）に
送り返される応答を生成し操作するための一連の機能が含まれています。応答は
Kongによって生成される（たとえば、リクエストを拒否する認証プラグイン）か、
サービスの応答本文からプロキシされて返されます。

`kong.service.response`とは異なり、このモジュールではクライアントに返送する前に応答を変更できます。

kong.response.get\_status\(\)
--------------------------------

ダウンストリームレスポンスに現在設定されているHTTPステータスコードを（Luaの番号として）返します。

リクエストがプロキシされた場合（`kong.response.get_source()` のように）、戻り値はサービスからのレスポンスです（`kong.service.response.get_status()` と同じ）。

リクエストがプロキシ *されず* 、応答がKong自体によって生成された場合（つまり`kong.response.exit()`経由）の場合、戻り値はそのまま返されます。

**フェーズ** 

* header\_filter, response, body\_filter, log, admin\_api

**戻り値** 

* `number`：ステータス。HTTPステータスコードは現在ダウンストリームの応答に設定されています。 

**使用方法** 

```lua
kong.response.get_status() -- 200
```

kong.response.get\_header\(name\)
------------------------------------

指定されたレスポンスヘッダーの値を、クライアントが
受け取った際に見ることが出来る形で返します。

この関数によって返されるヘッダーのリストは、プロキシされたサービスからのレスポンスヘッダー *および* Kong によって追加されたヘッダー（`kong.response.add_header()` など）の両方で構成されます。

戻り値は`string`か、ヘッダーのある`name`が応答に見つからない場合は`nil`になります。同じ名前のヘッダーがリクエストに複数回存在する場合、この関数はこのヘッダーの最初に発生した値を返します。

**フェーズ** 

* header\_filter, response, body\_filter, log, admin\_api

**パラメータ** 

* **name** \(`string`\): ヘッダーの名前。

ヘッダー名は大文字と小文字を区別せず、ダッシュ（`-`）は
アンダースコア（`_`）と書くことができます。たとえば、ヘッダー `X-Custom-Header` は `x_custom_header` としても
取得できます。

**戻り値** 

* `string|nil`: ヘッダーの値。

**使用方法** 

```lua
-- Given a response with the following headers:
-- X-Custom-Header: bla
-- X-Another: foo bar
-- X-Another: baz

kong.response.get_header("x-custom-header") -- "bla"
kong.response.get_header("X-Another")       -- "foo bar"
kong.response.get_header("X-None")          -- nil
```

kong.response.get\_headers\(\[max\_headers\]\)
--------------------------------------------------

レスポンスヘッダーを保持する Lua テーブルを返します。キーはヘッダー名です。
値はヘッダー値を含む文字列か、ヘッダーが複数回送信された場合は文字列の配列になります。このテーブルのヘッダー名は大文字と小文字を区別せず、小文字に正規化され、ダッシュ（`-`）はアンダースコア（`_`）と書くことができます。たとえば、ヘッダー `X-Custom-Header` は `x_custom_header` としても取得できます。

応答には、最初はヘッダーがありません。ヘッダーは、プラグインがヘッダーを生成してプロキシを短絡するとき（たとえば、認証プラグインがリクエストを拒否するとき）、またはリクエストがプロキシされ、後者の実行フェーズのいずれかが現在実行されているときに追加されます。

`kong.service.response.get_headers()`とは異なり、この関数は、Kong自体が追加したヘッダーを含む、受信時にクライアントに表示される *すべて* のヘッダーを返します。

{% if_version lte:3.2.x %}
デフォルトでは、この関数は最大 **100** 個のヘッダーを返します。オプションの `max_headers` 引数を指定してこの制限をカスタマイズできますが、 **1** より大きく、 **1000** 以下である必要があります。{% endif_version %}
{% if_version gte:3.3.x %}デフォルトでは、この関数は最大 **100** 個のヘッダー（または `lua_max_resp_headers` を使用して設定されたもの）を返します。オプションの `max_headers` 引数を指定してこの制限をカスタマイズできますが、 **1** より大きく **1000** 以下である必要があります。{% endif_version %}

**フェーズ** 

* header\_filter, response, body\_filter, log, admin\_api

**パラメータ** 

* **max\_headers** （`number`、 *オプション* ）: 解析されるヘッダーの数を制限します。

**戻り値** 

1. `table`: headers レスポンスでのヘッダーのテーブル
   表現。

2. `string`: err ヘッダーが `max_headers` より多く存在する場合、エラー `"truncated"` の文字列を返します。

**使用方法** 

```lua
-- Given an response from the Service with the following headers:
-- X-Custom-Header: bla
-- X-Another: foo bar
-- X-Another: baz

local headers = kong.response.get_headers()

headers.x_custom_header -- "bla"
headers.x_another[1]    -- "foo bar"
headers["X-Another"][2] -- "baz"
```

kong.response.get\_source\(\)
--------------------------------

この機能は、現在のレスポンスがどこから発生したかを判断するのに
役立ちます。Kongはリバースプロキシなので、リクエストをショートサーキットして独自の
応答を生成するか、プロキシされたサービスから応答が
返されます。

次の 3 つの可能な値を持つ文字列を返します。

* `"exit"`は、要求の処理中のある時点で `kong.response.exit()` への呼び出しがあった場合に返されます。これは、リクエストがプラグインまたはKong自体によって短絡された場合（無効な認証情報など）に発生します。
* `"error"`リクエスト処理中にエラーが発生した場合に返されます。 たとえば、アップストリームサービスへの接続中にタイムアウトが発生した場合。 
* `"service"`はプロキシを通してサービスに正常に 接続し、レスポンスがあった場合に返されます。

**フェーズ** 

* header\_filter, response, body\_filter, log, admin\_api

**戻り値** 

* `string`: ソース。

**使用方法** 

```lua
if kong.response.get_source() == "service" then
  kong.log("The response comes from the Service")
elseif kong.response.get_source() == "error" then
  kong.log("There was an error while processing the request")
elseif kong.response.get_source() == "exit" then
  kong.log("There was an early exit while processing the request")
end
```

kong.response.set\_status\(status\)
--------------------------------------

ダウンストリーム応答のHTTPステータスコードをクライアントに送信する前に変更できるようにします。

**フェーズ** 

* rewrite, access, header\_filter, response, admin\_api

**パラメータ** 

* **status** \(`number`\): 新しいステータス。

**戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
kong.response.set_status(404)
```

kong.response.set\_header\(name, value\)
-------------------------------------------

指定された値を持つレスポンスヘッダーを設定します。この関数は、同じ名前の既存のヘッダーを上書きします。

注: デフォルトでは、ヘッダー名のアンダースコアは自動的にダッシュに変換されます。この動作を無効にする場合は、Nginx 構成オプション`lua_transform_underscores_in_response_headers` を `off` に設定します。

この設定は、Kong構成ファイルで設定できます。

     nginx_http_lua_transform_underscores_in_response_headers = off

この設定を変更すると、自動アンダースコア変換に依存しているプラグインが壊れる可能性があることに注意してください。Transfer\-Encodingヘッダーはこの関数では設定できず、無視されます。

**フェーズ** 

* rewrite, access, header\_filter, response, admin\_api

**パラメータ** 

* **name** （`string`）：ヘッダー{% if_version lte:3.5.x -%}の名前

* **value** \(`string|number|boolean`\): ヘッダーの新しい値です。{% endif_version -%}
  {% if_version gte:3.6.x -%}

* **value** \(`array of strings|string|number|boolean`\): ヘッダーの新しい値です。{% endif_version %}
  **戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
kong.response.set_header("X-Foo", "value")
```

kong.response.add\_header（名前、値）
--------------------------------

指定された値を持つレスポンスヘッダーを追加します。
`kong.response.set_header()`とは異なり、この関数は同じ名前を持つ既存の
ヘッダーを削除しません。代わりに、同じ名前を持つ別のヘッダーがレスポンスに
追加されます。この名前のヘッダーがレスポンスにまだ存在しない
場合は、指定された値が追加されますが、次のものに類似します
`kong.response.set_header().`

**フェーズ** 

* rewrite, access, header\_filter, response, admin\_api

**パラメータ** 

* **name** （ `string`）: ヘッダー名。{% if_version lte:3.5.x -%}

* **値** （ `string|number|boolean` ）：ヘッダーの値。{% endif_version -%}
  {% if_version gte:3.6.x -%}

* **値** （ `array of strings|string|number|boolean` ）：ヘッダーの値。{% endif_version %}
  **戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
kong.response.add_header("Cache-Control", "no-cache")
kong.response.add_header("Cache-Control", "no-store")
```

kong.response.clear\_header\(name\)
--------------------------------------

クライアントに
送信されたレスポンス内の、指定されたヘッダーのすべての出現を削除します。

**フェーズ** 

* rewrite, access, header\_filter, response, admin\_api

**パラメータ** 

* **name** （`string`）：クリアされるヘッダーの名前

**戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
kong.response.set_header("X-Foo", "foo")
kong.response.add_header("X-Foo", "bar")

kong.response.clear_header("X-Foo")
-- from here onwards, no X-Foo headers will exist in the response
```

kong.response.set\_headers\(headers\)
----------------------------------------

応答のヘッダーを設定します。`kong.response.set_header()`とは異なり、`headers`引数はテーブルである必要があり、テーブル内の各キーは文字列（ヘッダーの名前に対応）で、各値は文字列または文字列の配列となります。

結果のヘッダーは辞書順に生成されます。同じ名前のエントリの順序（値が配列として指定されている場合）は保持されます。

この関数は、`headers`引数で指定されたものと同じ名前を持つ既存のヘッダーを上書きします。その他のヘッダーは変更されません。

Transfer\-Encodingヘッダーはこの関数では設定できず、無視されます。

**フェーズ** 

* rewrite, access, header\_filter, response, admin\_api

**パラメータ** 

* **headers** （`table`）：

**戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
kong.response.set_headers({
  ["Bla"] = "boo",
  ["X-Foo"] = "foo3",
  ["Cache-Control"] = { "no-store", "no-cache" }
})

-- Will add the following headers to the response, in this order:
-- X-Bar: bar1
-- Bla: boo
-- Cache-Control: no-store
-- Cache-Control: no-cache
-- X-Foo: foo3
```

kong.response.get\_raw\_body\(\)
------------------------------------

最後のチャンクが読み込まれたときに、本文全体を返します。

この関数を呼び出すと、内部リクエストコンテキスト変数にボディのバッファリングが開始され、チャンクが最後の1つでないときは、現在のチャンク（`ngx.arg[1]`）が`nil`に設定されます。最後のチャンクを読み込むと、関数は完全にバッファリングされたボディを返します。

**フェーズ** 

* `body_filter`

**戻り値** 

* `string`: body 最後のチャンクが読み込まれたときに完全な本文、 それ以外の場合は`nil`を返します。

**使用方法** 

```lua
local body = kong.response.get_raw_body()
if body then
  body = transform(body)
  kong.response.set_raw_body(body)
end
```

kong.response.set\_raw\_body\(body\)
----------------------------------------

応答の本文を設定します。

`body` 引数は文字列である必要があり、処理されることはありません。`Content-Length` ヘッダーが追加されている場合、この関数はヘッダーを変更することはできません。この関数を使用する場合は、`header_filter` フェーズなどで `Content-Length` ヘッダーもクリアする必要があります。

**フェーズ** 

* `body_filter`

**パラメータ** 

* **body** \(`string`\): 生のボディ。

**戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
kong.response.set_raw_body("Hello, world!")
-- or
local body = kong.response.get_raw_body()
if body then
  body = transform(body)
  kong.response.set_raw_body(body)
end
```

kong.response.exit\(status\[, body\[, headers\]\]\)
-----------------------------------------------------

この関数は、現在の処理を中断し、レスポンスを生成します。Kongがリクエストをプロキシする前に、プラグインがこれを使用してレスポンスを生成するのが一般的です（リクエストを拒否する認証プラグイン、またはキャッシュされたレスポンスを提供するキャッシングプラグインなど）。

演算子の意味をよりよく反映するために、この関数は`return`と組み合わせて使用することをお勧めします。

```lua
return kong.response.exit(200, "Success")
```

`kong.response.exit()`を呼び出すと、現在のフェーズのプラグインの実行フローが中断されます。後続のフェーズは引き続き呼び出されます。たとえば、プラグインが`access`フェーズで`kong.response.exit()`を呼び出した場合、そのフェーズでは他のプラグインは実行されませんが、`header_filter`、`body_filter`、および`log`フェーズは、プラグインとともに実行されます。プラグインは、リクエストがサービスにプロキシ **されず** 、代わりにKong自体によって生成される場合に対して防御的にプログラムする必要があります

1. 最初の引数 `status` は、クライアントに表示される応答のステータスコードを設定します。

   L4プロキシモードでは、提供される`status`コードは主にログ記録
   および統計目的であり、クライアントには直接表示されません。
   このモードでは、次のステータスコードのみがサポートされます。
   * 200 \- OK
   * 400 \- 不正なリクエスト
   * 403 \- 禁止
   * 500 \- 内部サーバーエラー
   * 502 \- 不正なゲートウェイ
   * 503 \- サービス利用不可

2. オプションの第二引数`body`は、レスポンスボディを設定します。文字列の場合、特別な処理は行われず、本文はそのまま送信されます。呼び出し元は、3番目の引数を使用して適切な`Content-Type`ヘッダーを設定する必要があります。

   便宜上、`body` をテーブルとして指定することもできます。その場合、
   `body`はJSONエンコードされており、`application/json`Content\-Typeヘッダーセットが含まれています。

   gRPCでは、この関数で `body` を送信することはできないため、代わりに `grpc-message` ヘッダーで `"body"` を送信します。
   * 本文がテーブルの場合は、本文の `message` フィールドを探し、 `grpc-message` ヘッダーとして使用します。
   * `Content-Type`ヘッダーに`application/grpc`を指定した場合、ボディは`grpc-message`ヘッダーを必要とせずに送信されます。

   L4 プロキシモードでは、`body` は、`nil` または文字列のみになります。自動 JSON エンコードは使用できません。`body` が指定されている場合、`status` の値に応じて次のようになります。
   * `status`が500、502、503の場合、 `body`はKongエラーログファイルに記録されます。
   * `status`がそれ以外の場合、 `body`がL4クライアントに送り返されます。

3. 3 番目のオプションの `headers` 引数は、送信するレスポンスヘッダーを指定する
   テーブルでもかまいません。指定した場合、その動作は
   `kong.response.set_headers()` のようになります。この引数は、L4 プロキシモードでは無視されます。

手動で指定しない限り、このメソッドは、便宜上、生成されたレスポンスの`Content-Length`ヘッダーを自動的に設定します。

**フェーズ** 

* preread、rewrite、access、admin\_api、header\_filter（ `body`がnilの場合のみ）

**パラメータ** 

* **status** （`number` ）：使用するステータス。
* **body** （`table|string`, *オプション* ）：使用する本体。
* **headers** （`table`、 *オプション* ）: 使用するヘッダー。

**戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
return kong.response.exit(403, "Access Forbidden", {
  ["Content-Type"] = "text/plain",
  ["WWW-Authenticate"] = "Basic"
})

---

return kong.response.exit(403, [[{"message":"Access Forbidden"}]], {
  ["Content-Type"] = "application/json",
  ["WWW-Authenticate"] = "Basic"
})

---

return kong.response.exit(403, { message = "Access Forbidden" }, {
  ["WWW-Authenticate"] = "Basic"
})

---

-- In L4 proxy mode
return kong.response.exit(200, "Success")
```

kong.response.error\(status\[, message\[, headers\]\]\)
---------------------------------------------------------

この関数は、現在の処理を中断し、エラーレスポンスを生成します。

演算子の意味をよりよく反映するために、この関数は`return`と組み合わせて使用することをお勧めします。

```lua
return kong.response.error(500, "Error", {["Content-Type"] = "text/html"})
```

1. `status` 引数は、クライアントに表示される応答のステータスコードを設定します。ステータスコードはエラーコード（399より大きいコード）である必要があります。

2. オプションの `message` 引数は、本文に書き込まれるエラーを説明するメッセージを設定します。

3. オプションの`headers`引数は、送信する応答ヘッダーを指定するテーブルでもかまいません。
   指定する場合、動作は、`kong.response.set_headers()`
   のようになります。

このメソッドは、JSON、XML、HTML、またはプレーンテキスト形式の応答を送信します。
実際のフォーマットは、次のオプションのいずれかをこの順番で使用して
決定されます。

* `headers`引数で、`Content-Type`ヘッダを使って手動で 指定します。
* 要求からの `Accept` ヘッダーに準拠します。
* `Content-Type`または`Accept`のヘッダーに設定がない場合、 レスポンスはデフォルトでJSON形式になります。便宜のため、生成されたレスポンスの`Content-Length` ヘッダーも参照してください。

**フェーズ** 

* rewrite、access、admin\_api、header\_filter（`body`がnilの場合のみ）

**パラメータ** 

* **status** \(`number`\): 使用するステータス（>399）。
* **message** \(`string`、 *任意* \): 使用されるエラーメッセージ。
* **headers** （`table`、 *オプション* ）: 使用するヘッダー。

**戻り値** 

* なし。無効な入力に対してはエラーが返されます。

**使用方法** 

```lua
return kong.response.error(403, "Access Forbidden", {
  ["Content-Type"] = "text/plain",
  ["WWW-Authenticate"] = "Basic"
})

---

return kong.response.error(403, "Access Forbidden")

---

return kong.response.error(403)
```
