---
title: "トレース API リファレンス"
content-type: "Reference"
---
はじめに
----

Gateway バージョン 3\.0\.0 では、トレース取得 API は Kong コアアプリケーションの一部になりました。API は `kong.tracing` 名前空間にあります。

トレース取得 API は [OpenTelemetry API の仕様](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md)に準拠しています。この仕様では、API をモジュールのツールとして使用する方法を定義します。OpenTelemetry API に精通している場合は、トレース取得 API も理解しやすいでしょう。

トレーシングAPIを使用すると、次の操作でモジュールの計測を設定できます。

* [スパン](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#span)
* [属性](https://opentelemetry.io/docs/specs/semconv/general/attributes/)

トレーサーを作成
--------

Kongは、コアモジュールとプラグインを計測するために、内部的にグローバルトレーサーを使用します。

{% if_version lte:3.1.x %}
デフォルトでは、トレーサーは[NoopTracer](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#get-a-tracer)です。トレーサーは、 `opentelemetry_tracing`構成が有効になっている場合に最初に初期化されます。

{% endif_version %}
{% if_version gte:3.2.x %}
デフォルトでは、トレーサーは[NoopTracer](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#get-a-tracer)です。トレーサーは、 `tracing_instrumentations`構成が有効になっている場合に最初に初期化されます。{% endif_version %}
新しいトレーサーは手動、またはグローバルトレーサーインスタンスを使用して作成できます。

```lua
local tracer

-- Create a new tracer
tracer = kong.tracing.new("custom-tracer")

-- Use the global tracer
tracer = kong.tracing
```

### トレースをサンプリング

トレーサーのサンプリングレートは、次のように構成できます。

```lua
local tracer = kong.tracing.new("custom-tracer", {
  -- Set the sampling rate to 0.1
  sampling_rate = 0.1,
})
```

`sampling_rate`が`0.1`の場合、10のリクエストのうち1つがトレースされることを意味します。レートが`1`の場合は、すべてのリクエストがトレースされることを意味します。

スパンを作成
------

スパンは、トレース内の 1 つの操作を表します。スパンをネストしてトレースツリーを形成できます。各トレースにはルートスパンが含まれます。ルートスパンは通常、操作全体を記述し、オプションでサブ操作に対して 1 つ以上のサブスパンを記述します。

```lua
local tracer = kong.tracing

local span = tracer:start_span("my-span")
```

spanプロパティは、`start_span`メソッドにテーブルを渡すことによって設定できます。

```lua
local span = tracer:start_span("my-span", {
  start_time_ns = ngx.now() * 1e9, -- override the start time
  span_kind = 2, -- SPAN_KIND
                  -- UNSPECIFIED: 0
                  -- INTERNAL: 1
                  -- SERVER: 2
                  -- CLIENT: 3
                  -- PRODUCER: 4
                  -- CONSUMER: 5
  should_sample = true, -- by setting it to `true` to ignore the sampling decision
})
```

完了したら、必ずスパンを終了してください。

```lua
span:finish() -- ends the span
```

{:.Note}
> 
> **注：** スパンテーブルは、スパンが終了すると、クリアされてテーブルプールに入れられるため、スパン終了後は使用しないでください。

アクティブなスパンを取得または設定
-----------------

アクティブなスパンとは現在実行されているスパンです。

オーバーヘッドを回避するために、`set_active_span`メソッドを呼び出してアクティブなスパンは手動で設定されます。
スパンの終了時に、アクティブなスパンは終了したスパンの親になります。

アクティブスパンを設定または取得するには、次のサンプルコードを使用できます。

```lua
local tracer = kong.tracing
local span = tracer:start_span("my-span")
tracer.set_active_span(span)

local active_span = tracer.active_span() -- returns the active span
```

### スコープ

トレーサーは、名前空間キーによって特定のコンテキストにスコープされます。

特定の名前空間のアクティブなスパンを取得するには、以下を使用します。

```lua
-- get global tracer's active span, and set it as the parent of new created span
local global_tracer = kong.tracing
local tracer = kong.tracing.new("custom-tracer")

local root_span = global_tracer.active_span()
local span = tracer.start_span("my-span", {
  parent = root_span
})
```

スパン属性を設定
--------

スパンの属性は、キーと値のペアのマップであり、テーブルを
`set_attributes`メソッドに渡すことで設定できます。

```lua
local span = tracer:start_span("my-span")
```

OpenTelemetryの仕様では一般的なセマンティック属性が定義されており、これを使用して範囲を記述できます。
また、UIでスパンを視覚化することも意味があります。

```lua
span:set_attribute("key", "value")
```

以下のスパンのセマンティック規則が定義されています。

* [一般](https://opentelemetry.io/docs/specs/semconv/general/attributes/): さまざまな種類の操作を説明する際に使用できる一般的なセマンティック属性。
* [HTTP](https://opentelemetry.io/docs/specs/semconv/http/http-spans/)：HTTPクライアントおよびサーバーの範囲用。
* [データベース](https://opentelemetry.io/docs/specs/semconv/database/): SQL および NoSQL クライアント呼び出し範囲用。
* [RPC/RMI](https://opentelemetry.io/docs/specs/semconv/rpc/rpc-spans/) : リモート プロシージャ コール \(gRPC など\) スパン用。
* [メッセージング](https://opentelemetry.io/docs/specs/semconv/messaging/messaging-spans/)：メッセージングシステム（キュー、パブリッシュ/サブスクライブなど）スパン用。
* [FaaS](https://opentelemetry.io/docs/specs/semconv/faas/faas-spans/): [Function as a Service](https://en.wikipedia.org/wiki/Function_as_a_service) \(AWS Lambda など\) のスパン用。
* [例外](https://opentelemetry.io/docs/specs/semconv/exceptions/exceptions-spans/): スパンに関連付けられた例外を記録します。
* [互換性](https://opentelemetry.io/docs/specs/semconv/general/trace-compatibility/)：互換性コンポーネントによって生成されたスパンの場合、例えばOpenTracingシムレイヤーなど。

スパンイベントの設定
----------

スパンのイベントは、`add_event`メソッドにテーブルを渡すことによって設定できる時系列イベントです。

```lua
local span = kong.tracing:start_span("my-span")
span:add_event("my-event", {
  -- attributes
  ["key"] = "value",
})
```

### エラーメッセージを記録する

このイベントはエラーメッセージを記録するためにも使用できます。

```lua
local span = kong.tracing:start_span("my-span")
span:record_error("my-error-message")

-- or (same as above)
span:add_event("exception", {
  ["exception.message"] = "my-error-message",
})
```

スパンステータスを設定
-----------

スパンのステータスはステータスコードであり、`set_status`メソッドにテーブルを渡すことによって設定できます。

```lua
local span = kong.tracing:start_span("my-span")

-- Status codes:
-- - `0` unset
-- - `1` ok
-- - `2` error
span:set_status(2)
```

スパンのリリース（オプション）
---------------

スパンはプールに保存され、`release`メソッドを呼び出すことでリリースできます。

```lua
local span = kong.tracing:start_span("my-span")
span:release()
```

デフォルトでは、Nginx リクエストが終了した後にスパンは解放されます。

トレースの視覚化
--------

OpenTelemetry との互換性があるため、トレースは任意の OpenTelemetry UI を通してそのまま視覚化できます。

トレースを視覚化する方法については、[OpenTelemetryプラグイン](/hub/kong-inc/opentelemetry/)を参照してください。

リファレンス
------

* [PDKのトレーシング](/gateway/{{page.release}}/plugin-development/pdk/kong.tracing/)
* [OpenTelemetry プラグイン](/hub/kong-inc/opentelemetry/)

