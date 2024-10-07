---
title: "WebAssembly"
content_type: "reference"
badge: "free"
---
KongのWebAssembly
----------------

Luaプラグインに加えて、{{site.base_gateway}} はプロキシ機能を拡張し、カスタムビジネスロジックを実行する手段としてWebAssemblyをサポートしています。これは、[Proxy\-Wasm仕様](https://github.com/proxy-wasm/spec)のKongの[オープンソース実装](https://github.com/Kong/ngx_wasm_module) で可能になります。

コンセプト
-----

### フィルター

Proxy\-Wasmのコンテキストでは、「フィルター」はリクエスト/応答処理の特定の段階で実行される、
特定のコンポーネントまたはモジュールを指します。フィルターは
次のようなアクションを実行できます。

* リクエストヘッダーまたは応答ヘッダーの読み取りまたは書き込み
* リクエストボディまたは応答本文の読み取りまたは書き込み
* リクエストを早期に終了し、自分で生成した応答を送信する

KongのLuaプラグインサポートに精通している場合は、フィルターをプラグインに部分的に類似したものとして捉えることができます。

フィルターとLuaプラグインでは、実行時のファイル形式が異なります。Luaプラグインは、通常、複数の異なるLuaソースコードファイルにまたがり、すべてパッケージ化され、テキスト形式でデプロイされます。
フィルターはソースコードから、`.wasm`拡張子の付いた単一のバイナリファイルにコンパイルされる必要があります。

{{site.base_gateway}}で使用するには、すべてのフィルターファイルがディスク上の単一のディレクトリに存在しており、
`wasm_filters_path`構成ディレクティブで示されている必要が
あります。

### フィルターチェーン

フィルターチェーンは、特定のサービスやルートへの各リクエストに対して実行される一つ以上のフィルターを表すデータベースエンティティであり、それぞれに設定があります。

以下に例を示します。

```yaml
---
filter_chains:
  name: my-filter-chain
  # Filter Chains can be toggled on/off for testing
  enabled: true
  filters:
    # this maps to $wasm_filters_path/my_filter.wasm
    - name: my_filter
      # at runtime, configuration is passed to the filter as a byte array
      #
      # filter configuration is inherently untyped--schema and
      # encoding/serialization are left up to the filter code itself
      config: "foo=bar baz=bat"

    - name: my_other_filter
      # individual filters within a chain can be toggled on/off for testing
      enabled: false
      config: >-
          {
            "some_settings": {
              "a": "b",
              "c": "d"
            },
            "foo": "bar"
          }

    # the same filter may occur more than once in a single chain, and all   entries will be
    # executed
    - name: my_filter
      config: "foo=123 baz=456"
```

#### 関係

フィルターチェーンはサービスまたはルートにリンクする **必要があり** 、{{site.base_gateway}}バージョン3\.4では
1対1の関係になっています。

* サービスまたはルートは、単一のフィルターチェーンにのみリンクできます。
* フィルターチェーンは、単一のサービスまたはルートにのみリンクできます。

フィルター実行動作
---------

特定のリクエストに対して実行されるフィルターのリストは、Luaプラグインの実行後の`access`フェーズ中に
決定されます。リクエストのサービスエンティティが
フィルターチェーンにリンクされている場合、フィルターが実行されます。もし
リクエストに一致したルートエンティティがフィルターチェーンにリンクされている場合は、
サービス関連のフィルター（もしあれば）が実行された *後で*
そのフィルターが実行されます。Admin APIのエンドポイント`/routes/{route_name_or_id}/filters/enabled`を介して、
特定のルートの完全なフィルター実行プランを調べることができます。

フィルターは常にフィルターチェーン内で定義された順序で実行されます。
同じフィルターが、サービスチェーンやルートチェーンから、あるいは同じフィルターチェーン内で、複数回実行計画に現れる可能性があります。各エントリは、独自の構成で実行されます。

### LuaプラグインとProxy\-Wasmフィルターを同時に使用できますか？

はい。しかし、各リクエストフェーズにおいて、LuaプラグインはWasmフィルターより *前に* 実行されることに注意してください。

この順序には、注意すべき重要な副作用が1つあります。Lua プラグインが `access` フェーズ中またはその前にリクエストを終了した場合（例外をスローするか、明示的に `kong.response.exit()` または他のPDK関数でレスポンスを送信することによる）、他のフェーズを含め、 **リクエストに対してWasmフィルタは実行されません** （例：`header_filter`など）。

制限事項と既知の問題
----------

### Kong PDK

この記事の執筆時点では、フィルターはKongの
Lua\-land PDKユーティリティで利用できず、Proxy\-Wasmで定義される機能により制限されています。

### フィルター構成スキーマ

フィルター構成が型指定されていません。つまり、Admin APIを使用してフィルターチェーンインスタンスを作成する場合などに、Kongは事前にフィルター構成を確認できなくなります。
これはProxy\-Wasm仕様の制限であるため、Kongの実装にとってはそれほど大きな制限とはなりません。

### Proxy\-Wasm機能

Proxy\-Wasm仕様は比較的新しいものであり、現在も進化を続けています。この仕様の一部の動作はまだ完全に実装されていない可能性があります。また、{{site.base_gateway}}や他のProxy\-Wasm実装との間で動作が一致しない場合もあります。

詳細については、[ngx\_wasm\_module Proxy\-Wasmドキュメント](https://github.com/Kong/ngx_wasm_module/blob/main/docs/PROXY_WASM.md#current-limitations)を参照してください。

その他の参考資料
--------

* [Proxy\-Wasmフィルターの作成](/gateway/latest/plugin-development/wasm/filter-development-guide)
* [WebAssembly for Proxies（Proxy\-Wasm）の仕様](https://github.com/proxy-wasm/spec)
* [`ngx_wasm_module`](https://github.com/Kong/ngx_wasm_module)
* [`ngx_wasm_module`Proxy\-Wasmドキュメント](https://github.com/Kong/ngx_wasm_module/blob/main/docs/PROXY_WASM.md)
* [Go SDK](https://github.com/tetratelabs/proxy-wasm-go-sdk/)
* [Rust SDK](https://github.com/proxy-wasm/proxy-wasm-rust-sdk/)

