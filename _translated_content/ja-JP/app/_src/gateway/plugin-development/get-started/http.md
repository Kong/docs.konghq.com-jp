---
title: "外部サービスの利用"
book: "plugin_dev_getstarted"
chapter: 5
---

{{site.base_gateway}}は、セキュリティ、トラフィック管理、監視、分析に関する機能を提供します。さらに、 {{site.base_gateway}}をビジネスロジックの補助的な開発レイヤーとして使用できます。アップストリームサービスからのAPI応答をサードパーティサービスのデータで装飾する目的が、一般的なユースケースです。

前提条件
----

このページはカスタムプラグインを開発するための[「はじめに」](/gateway/{{page.release}}/plugin-development/get-started/index)
ガイドの第4章です。これらの手順はガイドの前の章を参照しており、同じ開発者ツールの
前提条件を必要とします。

詳細な手順
-----

以下は、httpクライアントを使用して外部サービスからデータを取得し、JSON値を解析する方法を示すステップバイステップの説明です。解析されたデータを使用して、APIゲートウェイクライアントに返す前に応答データに値を追加する方法を説明します。

### HTTP および JSON のサポートを含める

まず、HTTP と JSON の構文解析サポートへのアクセスを可能にする 2 つの新しいライブラリを `handler.lua` 
ファイルにインポートします。

[lua\-resty\-http](https://github.com/ledgetech/lua-resty-http)ライブラリを使用して、HTTPクライアントをサードパーティに接続します。このライブラリは、ここで使用する単一のHTTPリクエストを作成するための簡単なインターフェースを提供しますが、ライブラリを使用する際のすべてのオプションについてはドキュメントを参照してください。

JSONサポートには、高速かつ標準に準拠したJSONサポートを提供する[lua\-cjson](https://github.com/mpx/lua-cjson)
ライブラリを使用します。lua\-cjsonは、
例外のないエンコーディングサポートとデコーディングサポートを可能にするライブラリの`safe`バリアント
をサポートしています。

新しく導入された要素を最上位の`handler.lua`に追加します。

```lua
local http  = require("resty.http")
local cjson = require("cjson.safe")
```

### サードパーティのhttpリクエストの呼び出し

`lua-resty-http`ライブラリは、サードパーティのサービスへの
連絡に使用できる単純なHTTPリクエスト関数（`request_uri`）を提供します。ここでは
*httpbin.org/anything* への`GET`リクエストの呼び出しを表示しますレスポンスに
様々な情報をエコーバックするAPIです。

`handler.lua`モジュール内の`MyPluginHandler:response`関数の先頭に以下を追加します。

```lua
local httpc = http.new()

local res, err = httpc:request_uri("http://httpbin.org/anything", {
  method = "GET",
})
```

サードパーティのサービスへのリクエストが成功した場合、 `res`
変数には応答が含まれます。成功したレスポンスの処理方法を示す前のエラーイベントはどうなりますか？
`err` の戻り値にエラーが表示されます。
それらを処理するためのオプションを見てみましょう。

### 応答エラーを処理する

{{site.base_gateway}}
[プラグイン開発キット](/gateway/{{page.release}}/plugin-development/pdk/)
は、エラー状態の処理に役立つさまざまな機能を提供しています。

この例では、アップストリームサービスからの応答を処理し、
サードパーティのサービスからの値を使用してクライアントの応答を装飾します。
サードパーティのサービスへのリクエストが失敗した場合は、選択肢があります。
応答の処理を終了してエラーをクライアントに返すか、
または、応答の処理を続行するとカスタムヘッダーロジックは完了しません。
この例では、
応答処理を終了し、クライアントに`500`内部サーバーエラーを返しています。

`httpc:request_uri`を呼び出したら、以下のコードを `MyPluginHandler:response` 関数にすぐに追加してください。

```lua
if err then
  return kong.response.error(500,
    "Error when trying to access 3rd party service: " .. err,
    { ["Content-Type"] = "text/html" })
end
```

代わりに処理を続行することを選択した場合は、次のように、`MyPluginHandler:response` 関数からのエラーメッセージと`return`をログに記録できます。

```lua
if err then
    kong.log("Error when trying to access 3rd party service:", err)
    return
end
```

### サードパーティのレスポンスからのJSONデータを処理する

このサードパーティのサービスはレスポンス本文に JSON オブジェクトを返します。ここでは、JSON 本文から単一の値を抽出して解析する方法を説明します。

`request_uri` 関数から受け取った `res.body` 値を渡す `lua-cjson` ライブラリ内の `decode` 関数を使用します。

```lua
local body_table, err = cjson.decode(res.body)
```

`decode`関数は値のタプルを返します。最初の値に含まれるのはデコードに成功した結果で、解析データの掲載テーブルとしてJSON形式で表示されます。
エラーが発生した場合は、2番目の値にエラー情報（あるいは成功時に`nil`）が表示されます。

上記の HTTP 処理と同様に、エラーが発生した場合はクライアントにエラーレスポンスを返し、処理を停止します。前の行の後の `MyPluginHandler:response` 関数に以下を追加します。

```lua
if err then
  return kong.response.error(500,
    "Error while decoding 3rd party service response: " .. err,
    { ["Content-Type"] = "text/html" })
end
```

この時点で、エラー状態にならなければ、クライアントの応答を装飾するために使用できるサードパーティからの有効な回答が得られます。 *httpbin* サービスは、要求されたURLのエコーである`url`フィールドを含むさまざまなフィールドを返します。この例では、前のエラー処理の後に次のコードを追加して、 `url`フィールドをクライアントのレスポンスに渡します。

```lua
kong.response.set_header(conf.response_header_name, body_table.url)
```

コードの各セクションを分類しました。以下は `handler.lua` ファイルの完全なコード一覧です。

### 完全なコードの一覧

```lua
local http  = require("resty.http")
local cjson = require("cjson.safe")

local MyPluginHandler = {
  PRIORITY = 1000,
  VERSION = "0.0.1",
}

function MyPluginHandler:response(conf)

  kong.log("response handler")

  local httpc = http.new()

  local res, err = httpc:request_uri("http://httpbin.org/anything", {
    method = "GET",
  })

  if err then
    return kong.response.error(500,
      "Error when trying to access 3rd party service: " .. err,
      { ["Content-Type"] = "text/html" })
  end

  local body_table, err = cjson.decode(res.body)

  if err then
    return kong.response.error(500,
      "Error when decoding 3rd party service response: " .. err,
      { ["Content-Type"] = "text/html" })
  end

  kong.response.set_header(
    conf.response_header_name,
    body_table.url)

end

return MyPluginHandler
```

### テストコードを更新する

この段階では、`pongo run` コマンドを再実行して以前に構築した統合テストを実行すると
、エラーとなります。ヘッダーの期待値が `response` から `http://httpbin.org/anything` に変更されました。`spec/my-plugin/01-integration_spec.lua` ファイルを更新して、ヘッダーの新しい値をアサートします。

```lua
-- validate the value of that header
assert.equal("http://httpbin.org/anything", header_value)
```

`pongo run`を使用してテストを再実行し、成功を確認します。

```sh
...
[----------] Global test environment teardown.
[==========] 2 tests from 1 test file ran. (23171.28 ms total)
[  PASSED  ] 2 tests.
```

このガイドでは、作業を開始するための例を示します。実際のプラグイン開発シナリオでは、実際に使用されているサードパーティのサービスへのネットワーク呼び出しを行うのではなく、模擬サービスを使用してテスト依存関係を提供することで、サードパーティサービスの統合テストを構築する必要があります。Pongoは、この目的のためにテスト依存関係をサポートしています。テスト依存関係の設定の詳細については、[Pongoのドキュメント](https://github.com/Kong/kong-pongo?tab=readme-ov-file#test-dependencies)をご覧ください。

次の手順
----

{{site.base_gateway}}プラグインをデプロイすることは、エンドツーエンドの開発パイプラインを構築する
最後のステップです。次の章では、デプロイのオプションについて説明し、
手順を説明するガイドを提供します。

