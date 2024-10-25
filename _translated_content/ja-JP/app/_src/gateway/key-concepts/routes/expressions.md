---
title: "式を使用してルートを設定する方法"
content-type: "how-to"
---
複雑または大規模な構成を扱う際、式を使用してルートを設定すると、
柔軟性とパフォーマンスが向上します。
このハウツーガイドでは、Expressionsルーターに切り替える方法と、新しい表現力豊かなドメイン固有言語でルートを設定する方法について説明します。
このガイドの残りの部分に進む前に
[式言語リファレンス](/gateway/latest/reference/expressions-language/)を理解しておきましょう。

前提条件
----

[kong.conf](/gateway/latest/production/kong-conf/)を編集して`router_flavor = expressions`行を追加し、 {{site.base_gateway}}を再起動します。
{:.note}
{% if_version gte:3.7.x %}
> 
> **注:** 式を有効にした後も、ルートオブジェクトの従来の照合フィールド（`paths` や `methods` など）は引き続き設定できます。新しい `expression` フィールドで式を指定できます。ただし、これらを従来の一致フィールドと同時に構成することはできません。さらに、`expression` フィールドと組み合わせて使用する新しい `priority` フィールドでは、式ルートの評価順序を指定できます。
> {% endif_version %}
> {% if_version lte:3.6.x %}
> **注:** 式を有効にすると、ルートオブジェクトに従来存在する一致フィールド \( `paths`や`methods`など\) は構成できなくなり、 `expression`フィールドで式を指定する必要があります。設定した式ルートの評価順序を指定できる新しいフィールド `priority` が追加されます。
> {% endif_version %}

式を使用したルートの作成
------------

式を使用して新しいルートオブジェクトを作成するには、[サービスエンドポイント](/gateway/latest/admin-api/#update-route)に `POST` リクエストを送信してください。

```sh
curl --request POST \
  --url http://localhost:8001/services/example-service/routes \
  --form-string expression='http.path == "/mock"'
```

この例では、既存のサービス `example-service` にパス `/mock` で新しいルートオブジェクトを関連付けました。
表現式 DSL を使用すると、複雑なルーターの一致条件を作成することもできます。

```sh
curl --request POST \
  --url http://localhost:8001/services/example-service/routes \
  --header 'Content-Type: multipart/form-data' \
  --form-string 'expression=(http.path == "/mock" || net.protocol == "https")'
```

### 式を使用して複雑なルートを作成

`POST`リクエスト内で演算子を使用して複雑なルートオブジェクトを記述できます。

```sh

curl --request POST \
  --url http://localhost:8001/services/example-service/routes \
  --header 'Content-Type: multipart/form-data' \
  --form-string name=complex_object \
  --form-string 'expression=(net.protocol == "http" || net.protocol == "https") &&
                (http.method == "GET" || http.method == "POST") &&
                (http.host == "example.com" || http.host == "example.test") &&
                (http.path ^= "/mock" || http.path ^= "/mocking") &&
                http.headers.x_another_header == "example_header" && (http.headers.x_my_header == "example" || http.headers.x_my_header == "example2")'
```

一般的なユースケースとそれら向けの式ルートの記述方法については、[式言語の概要](/gateway/latest/reference/expressions-language/)ページを参照してください。

利用可能なすべての言語機能のリストについては、[表現言語リファレンス](/gateway/latest/reference/expressions-language/language-references/)を参照してください。

マッチングフィールド
----------

次の表では、式ベースのルーターを使用する場合に使用可能な照合フィールドと、それに関連するタイプについて説明しています。

{% if_version gte:3.4.x %}
| フィールド | タイプ | HTTPサブシステムで利用可能 | ストリームサブシステムで利用可能 | 説明 |
|-------|-------|-------|-------|------- |
| `net.protocol` | `String` | ✅ | ✅ | ルートのプロトコル。`Route`エンティティの`protocols`フィールドにほぼ匹敵します。 **注：** `Route`エンティティの構成済み`protocols`は、常に、生成ルートの最上位レベルに追加されますが、`net.prococol`フィールドを使用すると追加の制約を式内に直接入力できます。|
| `tls.sni`  | `String`  | ✅ | ✅ | TLS経由の接続の場合は、ClientHelloパケットからの`server_name`拡張。|
| `http.method` | `String` | ✅ | ❌ | 受信HTTPリクエストのメソッド（たとえば、`"GET"`または`"POST"`）。|
|`http.host` | `String` | ✅ | ❌ | 受信HTTPの`Host`ヘッダー。|
| `http.path` | `String` | ✅ | ❌ | [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986#section-6)の定義ルールに従って正規化されたリクエストパス。このフィールド値には、存在する可能性のあるクエリパラメータは含まれて **いません** 。|
| `http.path.segments.<segment_index id="sl-md0000000">` | `String` | ✅ | ❌ | ゼロベースのインデックスを持つ入力（正規化されたもの）`http.path`から抽出されたパスセグメント。たとえば、リクエストパス`"/a/b/c/"`または`"/a/b/c"`の場合、 `http.path.segments.1` `"b"`を返します。|
| `http.path.segments.<segment_index id="sl-md0000000">_<segment_index id="sl-md0000000">` | `String` | ✅ | ❌ | 指定された閉区間内の入力（正規化されたもの）`http.path`から抽出されたパスセグメントは、`"/"`によって結合されます。インデックスはゼロベースです。たとえば、リクエストパス`"/a/b/c/"`または`"/a/b/c"`の場合、 `http.path.segments.0_1` `"a/b"`を返します。|
| `http.path.segments.len` | `Int` | ✅ | ❌ | 入力（正規化されたもの）からのセグメント数 `http.path`。たとえば、リクエストパス`"/a/b/c/"`または`"/a/b/c"`の場合、 `http.path.segments.len` `3`を返します。|
| `http.headers.<header_name id="sl-md0000000">` | `String[]` | ✅ | ❌ | リクエストヘッダー`<header_name id="sl-md0000000">`の値。 **注：** ヘッダー名は常に、アンダースコアと小文字の形式で正規化されるため、`Foo-Bar`、`Foo_Bar`、`fOo-BAr` はすべて `http.headers.foo_bar` フィールドの値になります。|
| `http.queries.<query_parameter_name id="sl-md0000000">` | `String[]` | ✅ | ❌ | クエリパラメータ`<query_parameter_name id="sl-md0000000">`の値。|
| `net.src.ip` | `IpAddr` | ✅ | ✅ | クライアントの IP アドレス。|
| `net.src.port` | `Int` | ✅ | ✅ | クライアントが接続に使用するポート番号。|
| `net.dst.ip` | `IpAddr` | ✅ | ✅ | {{site.base_gateway}}が着信接続を受け入れるリッスン IP アドレス。|
| `net.dst.port` | `Int` | ✅ | ✅ | {{site.base_gateway}}が着信接続を受け入れるリスニングポート番号。 |
{% endif_version %}

{% if_version lte:3.4.x %}
| フィールド | タイプ | HTTPサブシステムで利用可能 | ストリームサブシステムで利用可能 | 説明 |
|-------|-------|-------|-------|------- |
| `net.protocol` | `String` | ✅ | ✅ | ルートのプロトコル。`Route`エンティティの`protocols`フィールドにほぼ匹敵します。 **注：** `Route`エンティティの構成済み`protocols`は、常に、生成ルートの最上位レベルに追加されますが、`net.prococol`フィールドを使用すると追加の制約を式内に直接入力できます。|
| `tls.sni`  | `String`  | ✅ | ✅ | TLS経由の接続の場合は、ClientHelloパケットからの`server_name`拡張。|
| `http.method` | `String` | ✅ | ❌ | 受信HTTPリクエストのメソッド（たとえば、`"GET"`または`"POST"`） |
| `http.host` | `String` | ✅ | ❌ | 受信HTTPリクエストの`Host`ヘッダー。|
| `http.path` | `String` | ✅ | ❌ | [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986#section-6)に定義された定義ルールに従って正規化されたリクエストパス。このフィールド値には、存在する可能性のあるクエリパラメータは含まれて **いません** 。|
| `http.headers.<header_name id="sl-md0000000">`                         | `String[]` | ✅  | ❌  | リクエストヘッダー、`<header_name id="sl-md0000000">`の値。 **注：** ヘッダー名は常に、アンダースコアと小文字の形式で正規化されるため、`Foo-Bar`、`Foo_Bar`、`fOo-BAr`はすべて`http.headers.foo_bar`フィールドの値になります。|
| `http.queries.<query_parameter_name id="sl-md0000000">`                | `String[]` | ✅  | ❌  | クエリパラメータ、`<query_parameter_name id="sl-md0000000">`の値。|
| `net.src.ip`                          | `IpAddr`   | ❌  | ✅  | クライアントのIPアドレス。                                                          |
| `net.src.port`                        | `Int`      | ❌  | ✅  | クライアントが接続に使用するポート数。                                     |
| `net.dst.ip`                          | `IpAddr`   | ❌  | ✅  | {{site.base_gateway}}が受信接続を受け入れるリスニングIPアドレス。|
| `net.dst.port`                        | `Int`      | ❌  | ✅  | {{site.base_gateway}}が受信接続を受け入れるリスニングポート番号。|
{% endif_version %}

その他の情報
------

* [式言語に関する参考資料](/gateway/latest/reference/expressions-language/)
* [Expressionsルーターエンジンのリポジトリ](https://github.com/Kong/atc-router)

