---
title: "Proxy-Wasmフィルタの作成"
book: "plugin_dev"
chapter: 12
---
はじめてのProxy\-Wasmフィルター
----------------------

言語の選択
-----

フィルター開発を実現させるには、選択したプログラミング言語が次の条件を満たす必要があります。

1. ビルドターゲットとして WebAssembly をサポートします
2. [Proxy\-Wasm](https://github.com/proxy-wasm/spec) SDK がある

以下の言語 SDKs は現在 {{site.base_gateway}} で十分にテストされており、初めてのフィルターとして適切な選択です。

* [Go](https://github.com/tetratelabs/proxy-wasm-go-sdk/)
* [Rust](https://github.com/proxy-wasm/proxy-wasm-rust-sdk/)

他の言語やProxy\-Wasm SDKsは理論的にはサポートされていますが、まだ完全にテストされていません。

Kongでは、GoとRust Proxy\-Wasmフィルターのテンプレートを利用できます。独自フィルターの作成を開始したり、そのまま構築して、最初の`hello world`タイプフィルターを手に入れましょう。

* [Go Proxy\-Wasmフィルターテンプレート](https://github.com/Kong/proxy-wasm-go-filter-template/)
* [Rust Proxy\-Wasmフィルターテンプレート](https://github.com/Kong/proxy-wasm-rust-filter-template/)

### Kongを使用したフィルターのデプロイ

フィルターを記述して構築した後、次の手順ではKongを構成してWasmサポートを有効にし、フィルターを追加して、フィルターをルートまたはサービスに関連付けます。その後、Kongを起動し、フィルターを通過するリクエストを発行できるようになります。

#### 構成

{{site.base_gateway}} で WebAssembly サポートを有効にする必要があります。

```console
$ export KONG_WASM=on
```

さらに、`wasm_filters_path`パラメータは、

{{site.base_gateway}}が実行時にフィルターをロードする順番に構成する必要があります。これは、`.wasm`ファイルを
含む任意のフォルダになります。

ローカル開発中、
短いフィードバックループが必要な場合は、このパラメータを
ビルドツールチェーンの出力ディレクトリ（コンパイルされた
`<filter-name id="sl-md0000000">.wasm` ファイルが生成される場所）に設定できます。

```console
$ export KONG_WASM_FILTERS_PATH=/path/to/my_filter/build
```

#### Kong サービスおよびルートへのリンク

さて、次のステップは、Kong Filter Chainエンティティを作成することです。
前のステップで作成したフィルターを使い、Kongルート
またはサービスに関連付けます。これは宣言的なyaml設定ファイルを作成することによって行われます。
例：`kong.yml`:

```yaml
---
_format_version: '3.0'
services:
  - name: my-wasm-service
    url: http://httpbin.org
    routes:
      - name: my-wasm-route
        paths:
          - /
    filter_chains:
      - name: my-wasm-demo
        filters:
          - name: my_filter
            config: >-
              {
                "my_greeting": "Hello from Kong using WebAssembly!"
              }
```

```console
$ export KONG_DATABASE=off
$ export KONG_DECLARATIVE_CONFIG="$PWD/kong.yml"
```

#### Kongの起動

これでKongを開始できます。

```console
$ export KONG_PREFIX="$PWD/wasm-servroot"
$ kong prepare
$ kong start
```

Proxy\-Wasmフィルターの使用準備が整っています。

```console
$ http --headers http://127.0.0.1:8000/

HTTP/1.1 200 OK
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Thu, 03 Aug 2023 19:28:27 GMT
Vary: Accept-Encoding
Via: kong/3.4.0
X-Greeting: Hello from Kong using WebAssembly!
X-Kong-Proxy-Latency: 1
X-Kong-Upstream-Latency: 342
```

参考文献
----

* [Kong WebAssembly リファレンス](/gateway/latest/reference/wasm)
* [プロキシ用WebAssembly \(Proxy\-Wasm\) 仕様](https://github.com/proxy-wasm/spec)
* [ngx\_wasm\_module](https://github.com/Kong/ngx_wasm_module)
* [ngx\_wasm\_module Proxy\-Wasmドキュメント](https://github.com/Kong/ngx_wasm_module/blob/main/docs/PROXY_WASM.md)
* [Kongの`filter_chains`エンティティ](/gateway/latest/reference/wasm/#filter-chain)
* [Go SDK](https://github.com/tetratelabs/proxy-wasm-go-sdk/)
* [Rust SDK](https://github.com/proxy-wasm/proxy-wasm-rust-sdk/)

