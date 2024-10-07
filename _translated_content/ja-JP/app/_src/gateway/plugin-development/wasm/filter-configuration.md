---
title: "proxy-wasmフィルタの構成"
book: "plugin_dev"
chapter: 13
---
proxy\-wasmフィルタの構成
-------------------

{:.note}
> 
> これが書かれた時点では、JSONスキーマの[Draft 4](https://json-schema.org/specification-links#draft-4)仕様のみがサポートされています。

### 目的

[Proxy\-Wasm 仕様](https://github.com/proxy-wasm/spec)では、フィルターのランタイム構成はバイト配列としてのみ定義されます。構成処理（エンコード、シリアル化、検証）の詳細は、実装するフィルターに任されます。

その結果、Proxy\-Wasm での実装では、フィルターインスタンスが作成される実行時まで構成を検証する手段がありません。
{{site.base_gateway}} では、無効な構成でフィルタを実行すると 500 エラーがトリガーされます。これは、Kongには、リクエストの処理を続行するのは安全ではないと想定する以外に選択肢がないためです。

これは、フィルターのデプロイメントにおいて課題となります。オペレーターは、ユーザーによる不適切な構成のフィルター作成をどのように防げばよいでしょうか。

### フィルター設定のJSONスキーマ

JSON 形式で構成を受け取るフィルターは、オプションで 
{{site.base_gateway}} の [JSON スキーマ](https://json-schema.org/)検証を利用できます。

これは、`wasm_filters_path`ディレクトリでフィルターのバイトコードに隣接する
フィルターメタデータファイルに、JSONスキーマを提供することで実現されます。

    /path/to/wasm-filters
    ├── my-filter.wasm
    ├── my-filter.meta.json
    ├── other-filter.wasm
    └── other-filter.meta.json

設定スキーマは`config_schema`メタデータフィールドで提供されます。以下に例を示します。

```json
{
  "config_schema": {
    "type": "object",
    "properties": {
      "my-property": {
        "type": "string"
      }
    },
    "required": [
      "my-property"
    ]
  }
}
```

Kongでスキーマを使用すると、リクエストボディや応答ボディの読み書きを実行する際に、フィルター構成を不透明な文字列として扱うのでなく、JSONとして解釈できます。

スキーマがない場合、JSONフィルターの構成はAPIリクエスト次の文字列として
シリアル化する必要があります。

    POST /services/test/filter-chains HTTP/1.1
    Host: kong
    User-Agent: curl/8.0.1
    Content-Type: application/json
    Accept: application/json
    Content-Length: 143
    
    {
      "name": "my-filter-chain",
      "filters": [
        {
          "name": "my-filter",
          "config": "{ \"my-property\": \"My value\" }"
        }
      ]
    }

ただしスキーマを使用すると、構成はJSONとして受け入れられます。

    POST /services/test/filter-chains HTTP/1.1
    Host: kong
    User-Agent: curl/8.0.1
    Content-Type: application/json
    Accept: application/json
    Content-Length: 151
    
    {
      "name": "my-filter-chain",
      "filters": [
        {
          "name": "my-filter",
          "config": {
            "my-property": "My value"
          }
        }
      ]
    }

### 例

この例では、 `set-response-header`は単一のレスポンスヘッダを設定するフィルタです。ヘッダー名と値を持つJSONの構成を受け取ります。

`$KONG_WASM_FILTERS_PATH/set-response-header.meta.json` の内容は次のとおりです。

```json
{
  "config_schema": {
    "type": "object",
    "properties": {
      "header-name": {
        "type": "string"
      },
      "header-value": {
        "type": "string"
      }
    },
    "required": [
      "header-name",
      "header-value"
    ]
  }
}
```

スキーマが確立されると、検証に合格しない構成は拒否されます。

    POST /services/test/filter-chains HTTP/1.1
    Host: kong
    User-Agent: curl/8.0.1
    Content-Type: application/json
    Accept: application/json
    Content-Length: 171
    
    {
      "name": "my-filter-chain",
      "filters": [
        {
          "name": "set-response-header",
          "config": {
            "header-name": "X-My-Custom-Header"
          }
        }
      ]
    }

応答：

    HTTP/1.1 400 Bad Request
    Access-Control-Allow-Credentials: true
    Access-Control-Allow-Origin: *
    Connection: keep-alive
    Content-Length: 203
    Content-Type: application/json; charset=utf-8
    Date: Mon, 06 Nov 2023 00:11:33 GMT
    Server: kong/3.6.0
    X-Kong-Admin-Latency: 6
    
    {
        "code": 2,
        "fields": {
            "filters": [
                {
                    "config": "property header-value is required"
                }
            ]
        },
        "message": "schema violation (filters.1: {\n  config = \"property header-value is required\"\n})",
        "name": "schema violation"
    }

一方で、有効な構成は受け入れられます。

    POST /services/test/filter-chains HTTP/1.1
    Host: kong
    User-Agent: curl/8.0.1
    Content-Type: application/json
    Accept: application/json
    Content-Length: 211
    
    {
      "name": "my-filter-chain",
      "filters": [
        {
          "name": "set-response-header",
          "config": {
            "header-name": "X-My-Custom-Header",
            "header-value": "Hello, Wasm!"
          }
        }
      ]
    }

応答：

    HTTP/1.1 201 Created
    Access-Control-Allow-Credentials: true
    Access-Control-Allow-Origin: *
    Connection: keep-alive
    Content-Length: 348
    Content-Type: application/json; charset=utf-8
    Date: Mon, 06 Nov 2023 00:21:30 GMT
    Server: kong/3.6.0
    X-Kong-Admin-Latency: 23
    
    {
        "created_at": 1699230090,
        "enabled": true,
        "filters": [
            {
                "config": {
                    "header-name": "X-My-Custom-Header",
                    "header-value": "Hello, Wasm!"
                },
                "enabled": true,
                "name": "set-response-header"
            }
        ],
        "id": "da8659e3-db85-435f-ba62-f95ec7615ddc",
        "name": "my-filter-chain",
        "route": null,
        "service": {
            "id": "992c4491-9962-48ff-8d5d-249ba82848d1"
        },
        "tags": null,
        "updated_at": 1699230090
    }

参考文献
----

* [JSON スキーマ](https://json-schema.org/)
* [Proxy\-Wasmフィルタを作成](/gateway/latest/plugin-development/wasm/filter-development-guide)
* [WebAssembly for Proxies（Proxy\-Wasm）の仕様](https://github.com/proxy-wasm/spec)

