---
title: "カスタムトレースエクスポーターの書き方"
content-type: "how-to"
---
Kongは、[OTLP/HTTP](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/protocol/otlp.md#otlphttp)の実装とともにOpenTelemetryプラグインをコアにバンドルしていますが、大規模な独自のエクスポーターを作成することもできます。

スパンの収集
------

スパンはトレーサーのバッファに格納されます。バッファは、バックエンドに送信されるのを待機しているスパンのキューです。

`span_processor`関数を使用して、バッファにアクセスし、スパンを処理できます。

```lua
-- Use the global tracer
local tracer = kong.tracing

-- Process the span
local span_processor = function(span)
    -- clone the span so it can be processed after the original one is cleared
    local span_dup = table.clone(span)
    -- you can transform the span, add tags, etc. to other specific data structures
end
```

`span_processor`関数は、プラグインの`log`フェーズで呼び出す必要があります。

完全な例
----

カスタムトレースエクスポーターの例については、[Github](https://github.com/Kong/kong/tree/master/spec/fixtures/custom_plugins/kong/plugins/tcp-trace-exporter)を参照してください。

