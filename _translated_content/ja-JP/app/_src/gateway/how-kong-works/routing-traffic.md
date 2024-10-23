---
title: "プロキシリファレンス"
---
このドキュメントでは、{{site.base_gateway}} の **プロキシ機能** について、ルーティング機能と内部動作の詳細を説明します。


{{site.base_gateway}}は、次の設定プロパティによって調整できるいくつかのインターフェースを公開します。

* `proxy_listen`は、{{site.base_gateway}}クライアントからの **パブリックHTTP（gRPC、WebSocketなど）トラフィック** を受け入れ、それをアップストリームサービス（デフォルトは`8000`）にプロキシするアドレス/ポートのリストを定義します。
* `admin_listen` もアドレスとポートのリストを定義しますが、これらは Kong の構成機能である **Admin API** （デフォルトでは `8001`）を公開するため、管理者のみがアクセスできるように制限する必要があります。{% include_cached /md/admin-listen.md desc='short' release=page.release %}
* `stream_listen`は`proxy_listen`に似ていますが、レイヤー4（TCP、TLS）汎用プロキシ向けです。デフォルトではオフになっています。

用語解説
----

* `client`: {{site.base_gateway}} の プロキシポートに対してリクエストを行う *ダウンストリーム* クライアントを指します。
* `upstream service`: クライアントのリクエスト/接続が転送される {{site.base_gateway}} の背後にある自身の API/サービスを指します。
* `Service`: サービスエンティティは、その名のとおり、独自のアップストリームサービスのそれぞれを抽象化したものです。サービスの例としては、データ変換マイクロサービス、課金APIなどがあります。
* `Route`: これは、{{site.base_gateway}} ルートエンティティを指します。ルートは 
  {{site.base_gateway}} へのエントリポイントであり、リクエストをマッチングして特定のサービスにルーティングするためのルールを定義します。
* `Plugin`: これは {{site.base_gateway}} "プラグイン" を指します。これはプロキシライフサイクルで 実行されるビジネスロジックの一部です。プラグインは グローバル（すべての受信トラフィック）または特定のルートやサービス上 のAdmin APIで構成されます。

概要
---

要約すると、{{site.base_gateway}}は構成済みのプロキシポート（デフォルトでは`8000`と`8443`）でHTTPトラフィックをリッスンし、明示的に構成された`stream_listen`ポートでL4トラフィックをリッスンします。
{{site.base_gateway}}は、ユーザーが構成済みのルートに照らして受信するHTTPリクエストまたはL4接続を評価し、一致するものを見つけようとします。
特定のリクエストが特定のルートのルールに一致する場合、{{site.base_gateway}}はリクエストのプロキシ設定を処理します。

各ルートはサービスにリンクされている可能性があるため、{{site.base_gateway}}はお客様がルートと関連する
サービスに設定したプラグインを実行し、リクエストをアップストリームに
プロキシします。{{site.base_gateway}}のAdmin APIを介してルートを管理できます。ルートには
受信HTTPリクエストを照合してルーティングするために使用される特別な属性があります。
ルーティング属性はサブシステム（HTTP/HTTPS、gRPC/gRPCS、TCP/TLS）によって異なります。

サブシステムとルーティング属性：

* `http`：`methods`。`hosts`。`headers`、`paths`（`https`の場合は`snis`）
* `tcp`: `sources`、`destinations`（そして`tls`の場合は`snis`）
* `grpc`: `hosts`、`headers`、`paths`（そして`grpcs`の場合は`snis`）

サポートされていないルーティング属性（たとえば、`sources` または `destinations` フィールドを持つ `http` ルート）でルートを構成しようとすると、エラーメッセージが表示されます。

    HTTP/1.1 400 Bad Request
    Content-Type: application/json
    Server: kong/<x.x.x id="sl-md0000000">
    
    {
        "code": 2,
        "fields": {
            "sources": "cannot set 'sources' when 'protocols' is 'http' or 'https'"
        },
        "message": "schema violation (sources: cannot set 'sources' when 'protocols' is 'http' or 'https')",
        "name": "schema violation"
    }

{{site.base_gateway}}が、設定されたルートのいずれにも一致しないリクエストを受信した場合、
（またはルートが設定されていない場合）、次のように応答します。

```http
HTTP/1.1 404 Not Found
Content-Type: application/json
Server: kong/<x.x.x id="sl-md0000000">

{
    "message": "no route and no Service found with those values"
}
```

サービスの構成方法
---------

[サービスの設定](/gateway/{{page.release}}/get-started/services-and-routes)クイックスタートガイドでは、
[Admin API](/gateway/{{page.release}}/admin-api)経由で Kong を構成する方法を説明しています。

{{site.base_gateway}} にサービスを追加するには、Admin API に HTTP リクエストを送信します。

```bash
curl -i -X POST http://localhost:8001/services/ \
  -d 'name=foo-service' \
  -d 'url=http://foo-service.com'
```

レスポンス: 

```json
HTTP/1.1 201 Created
...

{
    "connect_timeout": 60000,
    "created_at": 1515537771,
    "host": "foo-service.com",
    "id": "d54da06c-d69f-4910-8896-915c63c270cd",
    "name": "foo-service",
    "path": "/",
    "port": 80,
    "protocol": "http",
    "read_timeout": 60000,
    "retries": 5,
    "updated_at": 1515537771,
    "write_timeout": 60000
}
```

このリクエストは、 {{site.base_gateway}}に「foo\-service」という名前のサービスを登録するように指示します。
これは `http://foo-service.com` \(上流\) を指します。

**注：** `url`引数は、`protocol`、 `host`、 `port`、および`path`属性を一度に設定するための省略法の因数です。

次に、{{site.base_gateway}} を介してこのサービスにトラフィックを送信するために、{{site.base_gateway}} へのエントリポイントとして機能するルートを指定する必要があります。

```bash
curl -i -X POST http://localhost:8001/routes/ \
  -d 'hosts[]=example.com' \
  -d 'paths[]=/foo' \
  -d 'service.id=d54da06c-d69f-4910-8896-915c63c270cd'
```

レスポンス: 

```json
HTTP/1.1 201 Created
...

{
    "created_at": 1515539858,
    "hosts": [
        "example.com"
    ],
    "id": "ee794195-6783-4056-a5cc-a7e0fde88c81",
    "methods": null,
    "paths": [
        "/foo"
    ],
    "preserve_host": false,
    "priority": 0,
    "protocols": [
        "http",
        "https"
    ],
    "service": {
        "id": "d54da06c-d69f-4910-8896-915c63c270cd"
    },
    "strip_path": true,
    "updated_at": 1515539858
}
```

これで、指定された`hosts`と `paths`に一致する受信リクエストを照合し、設定した`foo-service`に転送するルートを設定し、このトラフィックが`http://foo-service.com`にプロキシされるようになりました。


{{site.base_gateway}}は透過的プロキシで、デフォルトでは、HTTP仕様で要求される`Connection`、`Date`などのさまざまヘッダーを除いて、リクエストをそのままアップストリームサービスに転送します。

ルートとマッチング機能
-----------

では、{{site.base_gateway}}がリクエストを設定されたルーティング属性とどのように照合するかについて説明しましょう。


{{site.base_gateway}}は、HTTP/HTTPS、TCP/TLS、および GRPC/GRPCS プロトコルのネイティブプロキシをサポートします。前述のように、これらのプロトコルはそれぞれ異なるルーティング属性のセットを受け入れます。

* `http`：`methods`。`hosts`。`headers`、`paths`（`https`の場合は`snis`）
* `tcp`: `sources`、`destinations`（そして`tls`の場合は`snis`）
* `grpc`: `hosts`、`headers`、`paths`（そして`grpcs`の場合は`snis`）

これらのフィールドはすべて **オプション** ですが、少なくとも **1つ** を指定する必要があることに留意してください。

リクエストがルートにマッチする条件は以下のとおりです。

* リクエストには、構成された **すべての** フィールドを含める **必要があります** 。
* リクエスト内のフィールド値の少なくとも1つは、構成済みの値と一致する **必要があります** （フィールド構成は1つ以上の値を受け入れますが、リクエストが必要とするのは一致とみなされる値のうち1つのみです） 

いくつかの例を見てみましょう。次のように設定されたルートについて考えてみます。

```json
{
    "hosts": ["example.com", "foo-service.com"],
    "paths": ["/foo", "/bar"],
    "methods": ["GET"]
}
```

このルートに一致する可能性のあるリクエストの一部は次のようになります。

```http
GET /foo HTTP/1.1
Host: example.com
```

```http
GET /bar HTTP/1.1
Host: foo-service.com
```

```http
GET /foo/hello/world HTTP/1.1
Host: example.com
```

これら3つのリクエストはすべて、ルート定義で設定された条件をすべて満たしています。

しかし、以下のリクエストは設定された条件にマッチ **しません** 。

```http
GET / HTTP/1.1
Host: example.com
```

```http
POST /foo HTTP/1.1
Host: example.com
```

```http
GET /foo HTTP/1.1
Host: foo.com
```

これら3つのリクエストはすべて、構成された条件の2つのみを満たします。最初のリクエストのパスは、構成された`paths`のいずれとも一致せず、2つ目のリクエストのHTTPメソッド、3つ目のリクエストのHostヘッダーとも一致しません。

ルーティングプロパティの連携の仕組みを理解したところで、各プロパティを個別に調べてみましょう。

### リクエストヘッダー


{{site.base_gateway}} は任意の HTTP ヘッダーによるルーティングをサポートします。この
機能の特別なケースとして、ホストヘッダーによるルーティングがあります。

Hostヘッダーに基づいたルーティングのリクエストは、特にこれが意図されたHTTP Hostヘッダーの使用方法であるため、{{site.base_gateway}}を介してトラフィックをプロキシするための最も簡単な方法です。{{site.base_gateway}}は、ルートエンティティの`hosts`フィールド経由で実行するのを容易にします。

`hosts`は複数の値を使用できますが、Admin APIから指定する場合はカンマ区切りである必要があり、JSONペイロードで次のように表現されます。

```bash
curl -i -X POST http://localhost:8001/routes/ \
  -H 'Content-Type: application/json' \
  -d '{"hosts":["example.com", "foo-service.com"]}'
```

レスポンス: 

    HTTP/1.1 201 Created
    ...

しかし、Admin APIはフォームURLエンコードされたコンテンツタイプもサポートしているので、
`[]`表記を使用して配列を指定できます。

```bash
curl -i -X POST http://localhost:8001/routes/ \
  -d 'hosts[]=example.com' \
  -d 'hosts[]=foo-service.com'
```

レスポンス: 

    HTTP/1.1 201 Created
    ...

このルートの`hosts`条件を満たすには、クライアントからのすべてのホストヘッダーは次のいずれかに設定する必要があります。

    Host: example.com

または:

    Host: foo-service.com

同様に、他のヘッダーをルーティングに使用できます。

```sh
curl -i -X POST http://localhost:8001/routes/ \
  -d 'headers.region=north'
```

レスポンス: 

    HTTP/1.1 201 Created

`Region`ヘッダーが`North`に設定された受信リクエストは、そのルートに
ルーティングされます。

    Region: North

#### ワイルドカードホスト名の使用

柔軟性を実現するために、{{site.base_gateway}}を使用すると`hosts`フィールドのワイルドカードを使用してホスト名を指定できるようになります。ワイルドカードホスト名を使用すると、一致するすべてのHostヘッダーが条件を満たすため、特定のルートと一致できます。

ワイルドカードのホスト名には、 **必ず** 、左端にアスタリスクを **1つのみ** 、 **または** 右端にドメインのラベルを含めます。
以下に例を示します。

* `*.example.com`であれば、`a.example.com` や`x.y.example.com` のような Host値がマッチします。
* `example.*`であれば、`example.com` や`example.org` のようなホストの値がマッチします。

完全な例は次のようになります。

```json
{
    "hosts": ["*.example.com", "service.com"]
}
```

これにより、次のリクエストがこのルートに一致するようになります。

```http
GET / HTTP/1.1
Host: an.example.com
```

```http
GET / HTTP/1.1
Host: service.com
```

#### `preserve_host` プロパティ

プロキシ時の {{site.base_gateway}} のデフォルトの動作は、
サービスの `host` で指定されたホスト名に、上流リクエストのホストのヘッダーを設定します。`preserve_host`フィールドは、 {{site.base_gateway}}にそうしないように指示するブールフラグを受け入れます。

たとえば、`preserve_host`プロパティが変更されておらず、ルートがそのように
構成されている場合:

```json
{
    "hosts": ["service.com"],
    "service": {
        "id": "..."
    }
}
```

クライアントから{{site.base_gateway}}へのリクエストは、次のようなものが考えられます。

```http
GET / HTTP/1.1
Host: service.com
```


{{site.base_gateway}}はサービスの`host`プロパティからホストヘッダー値を抽出し、
次のアップストリームリクエストを送信します。

```http
GET / HTTP/1.1
Host: <my-service-host.com id="sl-md0000000">
```

ただし、 `preserve_host=true`を使用してルートを明示的に構成すると、次のようになります。

```json
{
    "hosts": ["service.com"],
    "preserve_host": true,
    "service": {
        "id": "..."
    }
}
```

そして、クライアントからの同じ要求を想定すると次のようになります。

```http
GET / HTTP/1.1
Host: service.com
```


{{site.base_gateway}}は、クライアントのリクエストのホストを保持し、代わりに、次のアップストリームリクエストを送信します。

```http
GET / HTTP/1.1
Host: service.com
```

### 追加のリクエストヘッダー

`Host`以外のヘッダーによってリクエストをルーティングすることも可能です。

これを行うには、ルートで`headers`プロパティを使用します。

```json
{
    "headers": { "version": ["v1", "v2"] },
    "service": {
        "id": "..."
    }
}
```

次のようなヘッダーを持つリクエストがあるとします。

```http
GET / HTTP/1.1
version: v1
```

このリクエストは、サービスにルーティングされます。次の場合も同じことが起こります。

```http
GET / HTTP/1.1
version: v2
```

ただし、このリクエストはサービスにルーティングされません。

```http
GET / HTTP/1.1
version: v3
```

**注** ：`headers` キーは論理値`AND`で、値は論理値`OR`です。

### リクエストパス

ルートを一致させる別の方法は、リクエストパスを使用することです。この
ルーティング条件を満たすには、クライアントリクエストの正規化されたパスには、`paths`属性の値の次のいずれかを接頭辞に
**付ける必要があります** 。

たとえば、ルートが次のように構成されているとします。

```json
{
    "paths": ["/service", "/hello/world"]
}
```

次のリクエストが一致します。

```http
GET /service HTTP/1.1
Host: example.com
```

```http
GET /service/resource?param=value HTTP/1.1
Host: example.com
```

```http
GET /hello/world/resource HTTP/1.1
Host: anything.com
```

これらのリクエストごとに、 {{site.base_gateway}}は正規化されたURLパスにプレフィックスとしてルートの`paths`値のいずれかが付いていることを検出します。デフォルトでは、 {{site.base_gateway}}はURLパスを変更せずにリクエストをアップストリームにプロキシします。

パスプレフィックスを使用してプロキシする場合、 **最長のパスが最初に評価されます** 。これにより、2つのパス`/service`と`/service/resource` を持つ2つのルートを定義し、前者が後者を「シャドウ」しないようにできます。

#### パスでの正規表現の使用

パスが正規表現を見なされるためには、先頭に`~`を付ける必要があります。

    paths: ["~/foo/bar$"]

`~`が先頭に付いていないパスはプレーンテキストとみなされます。

    "paths": ["/users/\d+/profile", "/following"]

ルーターが正規表現を処理する方法の詳細については、[Expressionsを使用する際のパフォーマンス上の考慮事項](/gateway/latest/key-concepts/routes/expressions/#performance-considerations-when-using-expressions)を参照してください。

##### 評価の順序

ルーターは、ルートが設定されている `Route` の `regex_priority` フィールドを使用してルートを評価します。`regex_priority` の値が大きいほど、優先順位が高くなります。

```json
[
    {
        "paths": ["~/status/\d+"],
        "regex_priority": 0
    },
    {
        "paths": ["~/version/\d+/status/\d+"],
        "regex_priority": 6
    },
    {
        "paths": /version,
    },
    {
        "paths": ["~/version/any/"],
    }
]
```

このシナリオでは、 {{site.base_gateway}}は、次の順序で定義されたURIに対して受信リクエストを評価します。

1. `/version/\d+/status/\d+`
2. `/status/\d+`
3. `/version/any/`
4. `/version`

正規表現の数が多いルーターは、他のルールを対象としたトラフィックを消費する可能性があります。正規表現は構築と実行に非常にコストがかかり、簡単に最適化することもできません。
[ルーター式言語](/gateway/latest/reference/router-expressions-language/)を使用して、複雑な正規表現を作成することを避けることができます。

{% if_version lte:3.1.x %}
予期しない動作が見られた場合、リクエストヘッダーで`Kong-Debug: 1`を送信すると、{% endif_version %}の応答ヘッダーに一致したルートIDが示されます。

{% if_version gte:3.2.x %}
予期しない動作が見られる場合は、Kongデバッグヘッダーを使用して原因を追跡します。

1. `kong.conf`で[`allow_debug_header: on`](/gateway/{{page.release}}/reference/configuration/#allow_debug_header)を設定します。
2. トラブルシューティング目的で、リクエストヘッダーで`Kong-Debug: 1`を送信し、応答ヘッダーで一致したルートIDを示します。{% endif_version %}

通常どおり、リクエストはルートの`hosts`および`methods`プロパティと一致する必要があり、{{site.base_gateway}}は[最も多くのルールと一致する](#matching-priorities)ものが見つかるまでルートをトラバースします。

##### グループのキャプチャ

グループの取得もサポートされており、一致したグループはパスから抽出され、プラグインで使用できます。次の正規表現を考えてみましょう。

    /version/(?<version id="sl-md0000000">\d+)/users/(?<user id="sl-md0000000">\S+)

そして、次のリクエストパス:

    /version/1/users/john


{{site.base_gateway}}はリクエストパスを一致とみなし、（他のルーティング属性を考慮したうえで）ルート全体が一致する場合、抽出される取得グループは`ngx.ctx`変数形式でプラグインから使用できます。

```lua
local router_matches = ngx.ctx.router_matches

-- router_matches.uri_captures is:
-- { "1", "john", version = "1", user = "john" }
```

##### 特殊文字のエスケープ

次に、Regexで見つかる文字は、
[RFC 3986](https://tools.ietf.org/html/rfc3986)などに応じた予約文字であることが多いため
、パーセントエンコード
する必要があります。Admin APIを **使用して正規表現パスでルートを設定する場合
、必要に応じて、ペイロードをURLエンコードしてください** 。例えば、
`curl`を使用し、 `application/x-www-form-urlencoded` MIME タイプを使用します:

```bash
curl -i -X POST http://localhost:8001/routes \
  --data-urlencode 'uris[]=/status/\d+'
```

レスポンス: 

    HTTP/1.1 201 Created
    ...

`curl`はペイロードを自動的にURLエンコードしないことに注意してください。また、`--data-urlencode`の使用により、{{site.base_gateway}}のAdmin APIによって `+`文字がURLデコードされスペースとして解釈されることを防ぐことができます。

#### `strip_path` プロパティ

ルートと一致するようにパスの接頭辞を指定するのが望ましいかもしれませんが、これを
アップストリームのリクエストに含めないでください。これを行うには、次のように設定して`strip_path`
ブールプロパティを使用してください。

```json
{
    "paths": ["/service"],
    "strip_path": true,
    "service": {
        "id": "..."
    }
}
```

このフラグを有効にすると{{site.base_gateway}}が指示され、このルートを一致してサービスにプロキシする場合に、アップストリームリクエストのURLに一致したURLパスの一部が含まれては **いけません** 。たとえば、次のクライアントのリクエストを上記のルートに送信します。

```http
GET /service/path/to/resource HTTP/1.1
Host: ...
```

これにより、{{site.base_gateway}}は以下のアップストリームリクエストを送信します。

```http
GET /path/to/resource HTTP/1.1
Host: ...
```

同じように、`strip_path` が有効になっているルートで正規表現パスが定義されている場合、リクエスト URL マッチングシーケンス全体が削除されます。以下に例を示します。

```json
{
    "paths": ["/version/\d+/service"],
    "strip_path": true,
    "service": {
        "id": "..."
    }
}
```

指定されたRegexパスに一致する次のHTTPリクエスト：

```http
GET /version/1/service/path/to/resource HTTP/1.1
Host: ...
```

次のように {{site.base_gateway}} で上流にプロキシされます。

```http
GET /path/to/resource HTTP/1.1
Host: ...
```

#### 正規化動作

単純なルートマッチのバイパスを防止するため、クライアントから受信するリクエストURIは、ルーターのマッチングが実施される前に、[RFC 3986](https://tools.ietf.org/html/rfc3986)に従って必ず正規化されます。
具体的には、受信リクエストURIには次の正規化手法が使用されます。この手法が選択されるのは、リクエストURIのセマンティックを通常変更しないためです。

1. パーセントエンコーディングされた3文字の組み合わせが大文字に変換されます。例：`/foo%3a` は `/foo%3A`となります。
2. 予約されていない文字のパーセントエンコーディングされた 3 文字の組み合わせがデコードされます。例: `/fo%6F` は `/foo` となります。
3. ドットセグメントは必要に応じて削除されます。例：`/foo/./bar/../baz` は `/foo/baz`となります。
4. 重複するスラッシュは統合されます。例：`/foo//bar` は `/foo/bar`となります。

ルートオブジェクトの `paths` 属性も正規化されます。パスがプレーンテキストまたは正規表現パスの場合、まず最初に決定することで
達成されます。結果に基づいて、さまざまな正規化技術
が使用されます。

プレーンテキストのルートパスの場合：

上記と同じ正規化手法である1〜4が使用されます。

正規表現ルートパスの場合。

方法 1 と 2 のみが使用されます。さらに、デコードされた文字が正規表現
のメタ文字になる場合は、バックスラッシュでエスケープされます。


{{site.base_gateway}}は、ルーターの一致を実行する前に、受信したリクエストURIを
正規化します。その結果、アップストリームのサービスに送信されるリクエストURIも、
元のURIセマンティックを保持する正規化された形式となります。

### リクエストHTTPメソッド

`methods`フィールドはHTTPメソッドに応じてリクエストをマッチングします。
複数の値を指定できますが、デフォルト値は空です（HTTPメソッドはルーティングには使用されません）。

次のルートは`GET`と`HEAD`経由のルーティングを許可します。

```json
{
    "methods": ["GET", "HEAD"],
    "service": {
        "id": "..."
    }
}
```

このようなルートは、次のリクエストと照合されます。

```http
GET / HTTP/1.1
Host: ...
```

```http
HEAD /resource HTTP/1.1
Host: ...
```

しかし、`POST`や`DELETE`リクエストにはマッチしません。これにより、ルート上でプラグインを設定する際のより詳細な設定が可能になります。たとえば、同じサービスを指す2つのルートを想像してみます。1つ目は無制限で認証されていない
`GET` リクエスト、2つ目は認証済みで流量制限された`POST`リクエストのみを許可するとします（認証プラグインおよび流量制限プラグインをそのようなリクエストに適用することによる
\)。

### リクエスト元

{:.note}
> 
> **注：** このセクションは、TCPおよびTLSルートにのみ適用されます。

`sources` ルーティング属性によって、
受信接続 IP および/またはポートソースのリストによるルートの照合が行われます。

次のルートは、送信元IP/ポートのリストを経由したルーティングを許可します。

```json
{
    "protocols": ["tcp", "tls"],
    "sources": [{"ip":"10.1.0.0/16", "port":1234}, {"ip":"10.2.2.2"}, {"port":9123}],
    "id": "...",
}
```

上記ルートは、CIDR範囲"10\.1\.0\.0/16"内にあるIPアドレスもしくはIPアドレス"10\.2\.2\.2"から、またはポート"9123"から発信されるTCP接続またはTLS接続と一致します。

### リクエスト先

{:.note}
> 
> **注：** このセクションは、TCPおよびTLSルートにのみ適用されます。

`destinations`属性は、 `sources`と同様に、
着信接続IPおよび/またはポートのリストによるルートのマッチングを許可しますが、
TCP/TLS 接続の宛先をルーティング属性として使用します。

### SNI のリクエスト

安全なプロトコル（`https`、`grpcs`、または`tls`）を使用する場合は、[サーバー
名表示](https://en.wikipedia.org/wiki/Server_Name_Indication)はルーティング属性として使用できます。次のルートは
SNI経由のルーティングを許可します。

```json
{
  "snis": ["foo.test", "example.com"],
  "id": "..."
}
```

TLS接続のSNI拡張子に設定されたホスト名と一致する受信リクエストは、このルートにルーティングされます。前述のように、SNIルーティングはTLSのみでなく、HTTPSなどTLSに伝送されるその他ののプロトコルにも適用され、複数のSNIがルートで指定されている場合、それらのいずれかは受信するリクエストのSNIと受信するリクエスト（名前間のOR）と一致する可能性があります。

SNIはTLSハンドシェイク時に示され、TLS接続が確立した後は変更できません。これは、例えば、同じkeepalive接続を再利用する複数のリクエストが、`Host` ヘッダーに関係なく、ルーター照合を実行する間に同じSNIホスト名を持つことを意味します。
が設立されました。これは、複数のリクエストを送るkeepaliveコネクションが、ルーター照合中（`Host`ヘッダーに関係なく）、同じSNIホスト名を持つことを意味します。

SNI と `Host` ヘッダーマッチャーが一致しないルートを作成することは可能ですが、一般的には推奨されません。

優先順位の一致
-------

ルートは、その`headers`、`hosts`、`paths`、および`methods`（さらに安全なルートである`"https"`、`"grpcs"`、`"tls"`の`snis`）フィールドに基づいて一致ルールを定義する場合があります。{{site.base_gateway}}が受信するリクエストをルートに一致させるには、既存のすべてのフィールドが満たされる必要があります。ただし、{{site.base_gateway}}を使用すると、2つ以上のルートを同じ値が含まれているフィールドで構成できるようになり、これが発生すると、{{site.base_gateway}}は優先度ルールを提供します。

ルールは次のとおりです。 **リクエストを評価するとき、 {{site.base_gateway}}は最初に最も多くの
ルールを持つルートを一致させようとします** 。

例えば、2つのルートが次のように設定されているとします。

```json
{
    "hosts": ["example.com"],
    "service": {
        "id": "..."
    }
},
{
    "hosts": ["example.com"],
    "methods": ["POST"],
    "service": {
        "id": "..."
    }
}
```

2番目のルートには`hosts`フィールド **と** `methods`フィールドがあるため、
まず、{{site.base_gateway}}によって評価されます。そうすることで、2番目のルートのために意図された呼び出しを最初のルートが「シャドーイング」しないようにすることができます。

したがって、このリクエストは最初のルートと一致します。

```http
GET / HTTP/1.1
Host: example.com
```

そして、このリクエストは2つ目のリクエストと一致します：

```http
POST / HTTP/1.1
Host: example.com
```

この理論に従うと、3 番目のルートが `hosts` 
フィールド、`methods` フィールド、`uris` フィールドで構成する場合、まず

{{site.base_gateway}} によって評価されることになります。

与えられたリクエストのルール数が2つのルート `A` と`B`で同じ場合は
、次のタイブレーカールールが記載された順序で適用されます。ルート `A` は、次の場合に `B` よりも選択されます。

* `A`には「プレーンな」ホストヘッダーのみが含まれ、`B`には「ワイルドカード」ホストヘッダーが1つまたは複数含まれる。 
* `A`は、`B`よりも多くの非ホストヘッダを持っています。
* `A`には " 正規表現" パスが少なくとも 1 つあり、`B` には "プレーン" パスしかありません。
* `A`の最長パスは、`B`の最長パスよりも長くなります。
* `A.created_at < B.created_at`

プロキシ動作
------

上記のプロキシルールは、{{site.base_gateway}}が受信するリクエストをアップストリームサービスに転送する方法を詳述しています。以下では、
{{site.base_gateway}}がHTTPリクエストと登録されたルートを *一致させる* ときと、リクエストが実際に *転送* されるときの間に内部で発生する内容を詳述しています。

### ロードバランシング


{{site.base_gateway}}は、アップストリームサービスのインスタンスのプール全体にわたるプロキシされたリクエストを分散するロードバランシング機能を実装します。

ロードバランシングの設定に関する詳細は、[ロードバランシンリファレンス](/gateway/{{page.release}}/how-kong-works/load-balancing)を参照してください。

### プラグインを実行


{{site.base_gateway}}は、プロキシされたリクエストのリクエスト/応答ライフサイクル内に組み込まれる「プラグイン」を介して拡張できます。
プラグインは、お使いの環境でさまざまな操作とプロキシされたリクエストで変換を実行できます。

プラグインは、グローバルに（すべてのプロキシトラフィックに対して）実行したり、あるいは特定のルートやサービス上で実行したりするように構成できます。どちらの場合も、Admin API 経由で[プラグイン構成](/gateway/{{page.release}}/admin-api#plugin-object)を作成する必要があります。

ルート（および関連するサービスエンティティ）が一致すると、{{site.base_gateway}}は
これらのエンティティのいずれかに関連するプラグインを実行します。ルートに構成されているプラグインは、
サービスに構成されているプラグインの前に実行されますが、それ以外は通常の
[プラグインの関連付け](/gateway/{{page.release}}/admin-api/#precedence)のルールが適用されます。

これらの設定済みのプラグインは `access` フェーズを実行します。詳細は、[プラグイン開発ガイド](/gateway/{{page.release}}/plugin-development)を参照してください。

### プロキシとアップストリームのタイムアウト

{{site.base_gateway}}が必要なロジックをすべて（プラグインを含む）を実行したら、リクエストをアップストリームサービスに転送する準備が完了します。これはNginxの[`ngx_http_proxy_module`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)を介して行われます。以下のサービスのプロパティに従って、{{site.base_gateway}}と特定のアップストリーム間の接続の希望のタイムアウトを構成できます。

* `connect_timeout`: アップストリームサービスへの接続を確立するためのタイムアウトをミリ秒単位で定義します。デフォルトは`60000`です。
* `write_timeout`: リクエストを上流サービスに転送するための連続した2 つの 書き込み操作間のタイムアウトをミリ秒で 定義します。デフォルトは`60000`です。
* `read_timeout`: アップストリームのサービスからリクエストを 受け取るための2つの連続する読み込み操作の間のタイムアウトをミリ秒単位で 定義します。デフォルトは`60000`です。


{{site.base_gateway}}はHTTP/1\.1経由でリクエストを送信し、次のヘッダーを設定します。

* `Host: <your_upstream_host id="sl-md0000000">`、このドキュメントで前述したとおりです。
* `Connection: keep-alive` アップストリーム接続の再利用を可能にします。
* `X-Real-IP: <remote_addr id="sl-md0000000">`ここでは、`$remote_addr`は、[ngx\_http\_core\_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_remote_addr)で提供された名前と同名の変数ベアリングです。 `$remote_addr`は [ngx\_http\_realip\_module](http://nginx.org/en/docs/http/ngx_http_realip_module.html)によって上書きされる可能性が高いことにご注意ください。
* `X-Forwarded-For: <address id="sl-md0000000">`ここでは、`<address id="sl-md0000000">`は、 同名のリクエストヘッダーに追加された[ngx\_http\_realip\_module](http://nginx.org/en/docs/http/ngx_http_realip_module.html)によって提供された `$realip_remote_addr`の内容です。
* `X-Forwarded-Proto: <protocol id="sl-md0000000">` ここでは `<protocol id="sl-md0000000">` はクライアントが使用するプロトコルです。`$realip_remote_addr` が **信頼できる** アドレスの 1 つである場合は、同じ名前のリクエストヘッダーが提供されれば転送されます。それ以外の場合は、[ngx\_http\_core\_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_scheme) が提供する `$scheme` 変数の値が使用されます。
* `X-Forwarded-Host: <host id="sl-md0000000">`。`<host id="sl-md0000000">`はクライアントから送信されるホスト名です。 `$realip_remote_addr`が **信頼できる** アドレスの1つである場合、同じ名前のリクエストヘッダーが提供されると転送されます。 それ以外の場合は、[ngx\_http\_core\_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_host)が提供する`$host`変数の値が使用されます。
* `X-Forwarded-Port: <port id="sl-md0000000">`ここは、リクエストを承認する`<port id="sl-md0000000">`サーバーのポートです。`$realip_remote_addr`が **信頼できる** アドレスの1つである場合、同じ名前のリクエストヘッダーが提供されると転送されます。 それ以外の場合は、[ngx\_http\_core\_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_server_port)が提供する`$server_port`変数の値が使用されます。
* `X-Forwarded-Prefix: <path id="sl-md0000000">`、ここでは`<path id="sl-md0000000">`はリクエストのパスであり、 {{site.base_gateway}}によって承認されています。`$realip_remote_addr`が **信頼できる** アドレスの1つである場合は、 同じ名前のリクエストヘッダーが提供されていれば 転送されます。それ以外の場合は、[ngx\_http\_core\_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_server_port)によって提供される`$request_uri`変数の値（ クエリ文字列は削除されます）が 使用されます。

{:.note}
> 
> **注** ：{{site.base_gateway}} は空のパスに`"/"`を返しますが、リクエストパスで他の正規化は行いません。

他のすべてのリクエストヘッダーは{{site.base_gateway}}によってそのまま転送されます。

唯一の例外となるのがWebSocketプロトコルを使用する場合です。その場合、{{site.base_gateway}}はクライアントとアップストリームサービスの間のプロトコルをアップグレードできるように次のヘッダーを設定します。

* `Connection: Upgrade`
* `Upgrade: websocket`

このトピックに関する詳細は、
[プロキシWebSocketトラフィック](/gateway/{{page.release}}/how-kong-works/routing-traffic#proxy-websocket-traffic)セクションで確認できます。

### エラーと再試行

プロキシ中にエラーが発生すると、{{site.base_gateway}} はNginxの基本的な[再試行](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream_tries)メカニズムを使ってリクエストを次のアップストリームに渡します。

ここには、次の2つの構成可能な要素があります。

1. 再試行回数：`retries`プロパティを使用してサービスごとに構成できます。詳細については、[Admin API](/gateway/{{page.release}}/admin-api)を参照してください。

2. 正確には何がエラーになるのか：ここで{{site.base_gateway}}はNginxのデフォルトを使用しています。これは、サーバーとの接続を確立したり、サーバーにリクエストを渡したり、レスポンスヘッダーを読み取ったりする間に､エラー又はタイムアウトが発生することを意味します。

{% if_version gte:3.8.x %}
二つ目のオプションは、Nginxの[`proxy_next_upstream`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream)ディレクティブに基づいています。このオプションは {{site.base_gateway}}から直接設定することはできませんが、Nginx のカスタム設定を使って追加することができます。 詳細は[設定リファレンス](/gateway/{{page.release}}/reference/configuration/)を参照してください。
{% endif_version %}


### 応答


{{site.base_gateway}}アップストリームサービスから受信した応答を、ストリーミング形式でダウンストリームクライアントに送信します。この時点で{{site.base_gateway}}は、`header_filter`フェーズにフックを実装するルートやサービスに追加される後続プラグインを実行します。

登録されているすべてのプラグインの `header_filter` フェーズが実行されると、{{site.base_gateway}} によって次のヘッダーが追加され、ヘッダー一式がクライアントに送信されます。

* `Via: kong/x.x.x`ここでは、`x.x.x` は使用中の {{site.base_gateway}} バージョンです
* `X-Kong-Proxy-Latency: <latency id="sl-md0000000">`。`latency`は、{{site.base_gateway}}がクライアントからリクエストを受信する時間からアップストリームサービスにリクエストを送信するまでの時間をミリ秒単位で示したものです。 
* `X-Kong-Upstream-Latency: <latency id="sl-md0000000">`、 `latency`は、{{site.base_gateway}}がアップストリームサービス応答の最初のバイトを待っていたミリ秒単位の時間です。

ヘッダーがクライアントに送信されると、{{site.base_gateway}} が `body_filter` フックを実装するルートやサービスの登録済みプラグインの実行を開始します。Nginx のストリーミングの性質により、このフックは複数回呼び出される可能性があります。このような `body_filter` フックによって正常に処理されたアップストリームレスポンスの各チャンクは、クライアントに送り返されます。`body_filter` フックの詳細については、[プラグイン開発ガイド](/gateway/{{page.release}}/plugin-development)を参照してください。

フォールバックルートの構成
-------------

{{site.base_gateway}}のプロキシ機能によって提供される柔軟性の実用的なユースケースとして、「フォールバックルート」を実装してみます。これにより、{{site.base_gateway}}がHTTP `404`、「ルートが見つかりません」で応答するのを避けるために、このようなリクエストをキャッチして特別なアップストリームサービスにプロキシするか、プラグインを適用することができます（このようなプラグインは、たとえば、リクエストをプロキシせずに、別のステータスコードまたは応答でリクエストを終了できます）。

以下はこのようなフォールバックルートの例です。

```json
{
    "paths": ["/"],
    "service": {
        "id": "..."
    }
}
```

ご想像のとおり、{{site.base_gateway}}へのHTTPリクエストはすべてこのルートに一致します。すべてのURIの先頭にはルート文字`/`が付くためです。
[リクエストパス](#request-path)セクションで学んだように、{{site.base_gateway}}は最長URLパスを最初に評価します。一方、`/`パスが
{{site.base_gateway}}によって評価されるのは最後になり、実質的な「フォールバック」ルートとして最終手段としてのみ照合されます。
その後、トラフィックを特別なサービスに送信したり、このルートに任意に配置するプラグインに適用できます。

ルートのTLSの構成
----------


{{site.base_gateway}} 、基礎接続ごとに TLS 証明書を動的に提供する手段を提供します。TLS 証明書はコアによって直接処理され、Admin API経由で
構成可能です。この機能を利用するには、TLS 経由で{{site.base_gateway}}に接続するクライアントが[Server 
Name Indication](https://en.wikipedia.org/wiki/Server_Name_Indication)拡張をサポートしている必要があります。

TLS 証明書は、{{site.base_gateway}} Admin API の 2 つのリソースによって処理されます。

* `/certificates`：キーと証明書が保存されます。
* `/snis`は、登録済みの証明書をサーバー名表示に関連付けます 。

これら2つのリソースに関するドキュメントは、[Admin API参考資料](/gateway/{{page.release}}/admin-api)を参照してください。

特定のルートでTLS証明書を構成する方法は次のとおりです。まず、TLS証明書とキーを
Admin API経由でアップロードします。

```bash
curl -i -X POST http://localhost:8001/certificates \
  -F "cert=@/path/to/cert.pem" \
  -F "key=@/path/to/cert.key" \
  -F "snis=*.tls-example.com,other-tls-example.com"
```

レスポンス: 

    HTTP/1.1 201 Created
    ...

`snis` フォームパラメータは便利なパラメータです。内部的には、SNI を直接挿入し、アップロードされた証明書を SNI に関連付けます。

上記の `snis` で定義されている SNI 名の 1 つにワイルドカードが含まれていることに注意してください
（`*.tls-example.com`）。SNI の左端（プレフィックス）または
右端（サフィックス）にワイルドカードを 1 つ含めることもできます。これは、複数のサブドメインを管理する場合に便利です。それぞれ SNI を作成せずに、ワイルドカード名で構成された単一
の `sni` を複数
のサブドメインの一致に使用できます。

有効なワイルドカードの位置は、`mydomain.*`、 `*.mydomain.com`、`*.www.mydomain.com`です。

{{site.base_gateway}}構成の次のパラメータを使用して、デフォルトの証明書を追加できます。

1. [`ssl_cert`](/gateway/{{page.release}}/reference/configuration/#ssl_cert)
2. [`ssl_cert_key`](/gateway/{{page.release}}/reference/configuration/#ssl_cert_key)

または、SNIが`*`のデフォルトの証明書を動的に構成します。

```bash
curl -i -X POST http://localhost:8001/certificates \
  -F "cert=@/path/to/default-cert.pem" \
  -F "key=@/path/to/default-cert.key" \
  -F "snis=*"
```

レスポンス: 

    HTTP/1.1 201 Created
    ...

`snis`のマッチングは、以下の優先順位が優先されます。

1. 正確なSNIの一致証明書
2. 接頭辞ワイルドカードによる証明書の検索
3. サフィックスワイルドカードによる証明書の検索
4. SNIに関連付けられている証明書の検索`*`
5. ファイルシステム上のデフォルトの証明書

ここで、次のルートを {{site.base_gateway}}内に登録する必要があります。このルートへのリクエストは、以下のように、便宜上Hostヘッダーのみを使ってマッチさせます。

```bash
curl -i -X POST http://localhost:8001/routes \
  -d 'hosts=prefix.tls-example.com,other-tls-example.com' \
  -d 'service.id=d54da06c-d69f-4910-8896-915c63c270cd'
```

レスポンス: 

    HTTP/1.1 201 Created
    ...

これで、ルートが{{site.base_gateway}}によってHTTPS経由で提供されることが期待できます。

```bash
curl -i https://localhost:8443/ \
  -H "Host: prefix.tls-example.com"
```

レスポンス: 

    HTTP/1.1 200 OK
    ...

接続を確立し、TLSハンドシェイクをネゴシエートする際に、クライアントがSNI拡張子の一部として`prefix.tls-example.com`を送信する場合、{{site.base_gateway}}は以前に構成された`cert.pem`証明書を提供します。証明書はHTTPSとTLSの両方の接続向けです。

### クライアントプロトコルの制限

ルートには、すべきクライアントプロトコルを制限する `protocols` プロパティがあります
聞いてください。 この属性で設定できる値は、`"http"` 、
`"https"` 、 `"grpc"` 、 `"grpcs"` 、 `"tcp"` 、または`"tls"` です。

`http` と `https` を含むルートは、両方のプロトコルのトラフィックを受け入れます。

```json
{
    "hosts": ["..."],
    "paths": ["..."],
    "methods": ["..."],
    "protocols": ["http", "https"],
    "service": {
        "id": "..."
    }
}
```

プロトコルを指定しない場合でも、ルートはデフォルトで
`["http", "https"]` になるため、同じ効果があります。

ただし、`https` *のみ* のルートでは、HTTPS経由のトラフィック *のみ* が許可されます。また、信頼できるIPによってTLS終了が以前に *発生した場合* は、暗号化されていないトラフィック *も* 許可されます。TLS終了は、リクエストが[trusted\_ips](/gateway/{{page.release}}/reference/configuration/#trusted_ips)の構成済みのIPのいずれかから送信された場合と、`X-Forwarded-Proto: https`ヘッダーが設定されている場合に有効と見なされます。

```json
{
    "hosts": ["..."],
    "paths": ["..."],
    "methods": ["..."],
    "protocols": ["https"],
    "service": {
        "id": "..."
    }
}
```

上記のようなルートがリクエストに一致したが、そのリクエストが
有効な事前の TLS 終了のないプレーンテキストの場合、 {{site.base_gateway}}次のように応答します。

```http
HTTP/1.1 426 Upgrade Required
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: Upgrade
Upgrade: TLS/1.2, HTTP/1.1
Server: kong/x.y.z

{"message":"Please use HTTPS protocol"}
```

生のTCP（必ずしもHTTPである必要なし）向けルートは、`protocols`属性で`"tcp"`を使用して作成できます。

```json
{
    "hosts": ["..."],
    "paths": ["..."],
    "methods": ["..."],
    "protocols": ["tcp"],
    "service": {
        "id": "..."
    }
}
```

同様に、`"tls"`値を使用して、生のTLSトラフィック（必ずしもHTTPSではない）を
受け入れるルートを作成することもできます。

```json
{
    "hosts": ["..."],
    "paths": ["..."],
    "methods": ["..."],
    "protocols": ["tls"],
    "service": {
        "id": "..."
    }
}
```

`TLS` *のみ* のルートでは、TLS経由のトラフィック *のみ* が許可されます。

TCP と TLS の両方を同時に受け入れることもできます。

    {
        "hosts": ["..."],
        "paths": ["..."],
        "methods": ["..."],
        "protocols": ["tcp", "tls"],
        "service": {
            "id": "..."
        }
    }


L4 TLSプロキシが動作するためには、`tls` `protocol`を受け入れるルートを作成し、適切なTLS証明書をアップロードし、その`sni`属性が着信コネクションのSNIと一致するように適切に設定されている必要があります。設定方法については、[ルートのTLS設定](#configuring-tls-for-a-route)セクションを参照してください。

WebSocketトラフィックのプロキシ
--------------------


{{site.base_gateway}}は、基盤となるNginx実装のおかげでWebSocketトラフィックをサポートします。クライアントとアップストリームサービスの間で{{site.base_gateway}}を *介して* WebSocket接続を確立したい場合、WebSocketハンドシェイクを確立する必要があります。これは、HTTPアップグレードメカニズムを介して行われます。{{site.base_gateway}}に行われたクライアントリクエストは以下のようになります。

```http
GET / HTTP/1.1
Connection: Upgrade
Host: my-websocket-api.com
Upgrade: WebSocket
```

これにより、{{site.base_gateway}}は、標準HTTPプロキシのホップバイホップの性質上、`Connection`ヘッダーと`Upgrade`ヘッダーを拒否するのではなく、アップストリームサービスに転送するようになります。

### WebSocketプロキシモード

{{site.base_gateway}}でWebSocketトラフィックをプロキシする方法は2つあります。

* HTTP\(S\) サービスとルート
* WS\(S\) サービスとルート

#### HTTP\(S\) サービスとルート

`http` および `https` プロトコルを使用するサービスとルートは、特別な構成を行わなくても WebSocket 接続を処理できます。この方法では、WebSocket セッションは通常のHTTPリクエストと同じように動作し、リクエスト/レスポンスのデータは不透明なバイトストリームとして扱われます。

```yaml
services:
  - name: my-http-websocket-service
    protocol: http
    host: 1.2.3.4
    port: 80
    path: /
    routes:
      - name: my-http-websocket-route
        protocols:
          - http
          - https
```

#### WS\(S\) サービスとルート

{:.badge .enterprise}

HTTPサービスとルートに加え、 {{site.ee_product_name}}には、
`ws` \(WebSocket\-over\-http\) および`wss` \(WebSocket\-over\-https\) オプション
サービス`protocol`、ルート`protocols` が含まれます。`http` / `https`とは対照的に、 `ws`
および`wss`サービスは、基盤となるWebSocket接続を完全に制御します。
つまり、WebSocketプラグインと[WebSocket PDK](/gateway/{{page.release}}/plugin-development/pdk/kong.websocket.client/)を使って、メッセージ単位でビジネスロジックを実行できます（メッセージ検証、アカウンティング、流量制限など）。

```yaml
services:
  - name: my-dedicated-websocket-service
    protocol: ws
    host: 1.2.3.4
    port: 80
    path: /
    routes:
      - name: my-dedicated-websocket-route
        protocols:
          - ws
          - wss
```

{:.note}
> 
> **注記** ：WebSocketメッセージのデコードと暗号化は、プロトコルに依存しない`http(s)`サービスの動作と比較した場合、ゼロでないパフォーマンスオーバヘッドの値が付属しています。APIに`ws(s)`サービスが提供する追加の機能が必要でない場合は、代わりに`http(s)`サービスを使用することをおすすめします。

### WebSocketとTLS

どのサービス/ルートが使用されているかに関係なく（`http(s)`または`ws(s)`）、

{{site.base_gateway}}はプレーンおよびTLS WebSocket接続を、それぞれ
`http`および`https`のポートで受け入れます。クライアントからのTLS接続を強制するには、
[ルート](/gateway/{{page.release}}/admin-api/#add-route)の`protocols`プロパティを`https`または`wss`にのみ
設定します。

[サービス](/gateway/{{page.release}}/admin-api/#add-service)がアップストリームWebSocketサービスを指すように設定する場合、{{site.base_gateway}}とアップストリーム間で使用するプロトコルを慎重に選択する必要があります。

TLS を使用する場合は、サービス `protocol` プロパティで `https`（または `wss`）プロトコルと適切なポート（通常は443）を使用して、アップストリーム WebSocket サービスを定義する必要があります。TLS を使用せずに接続するには、代わりに `http`（または `ws`）プロトコルとポート（通常は 80）を `protocol` 内で使用する必要があります。

{{site.base_gateway}}にTLSを終了させる場合、クライアントからのみ`https`/`wss`を許可することができますが、アップストリームサービスはプレーンテキスト（`http`または`ws`）でプロキシします。

プロキシ gRPC トラフィック
----------------

gRPCプロキシは{{site.base_gateway}}でネイティブにサポートされています。gRPCサービスを管理し、{{site.base_gateway}}でgRPCリクエストをプロキシするには、gRPCサービスのサービスとルートを作成します。

gRPCでは、オブザーバビリティとロギングのプラグインのみがサポートされています。
\-gRPCでサポートされているプラグインには、たとえば
Kong Hubページの「サポートされているプロトコル」フィールドの下に記載されている"grpc"と"grpcs"があります。
[File Log](/hub/kong-inc/file-log)プラグインのページをチェックしてください。

プロキシ TCP/TLS トラフィック
-------------------

TCP および TLS プロキシは {{site.base_gateway}} でネイティブでサポートされています。

このモードでは、`stream_listen`エンドポイントに到達する受信接続のデータはアップストリームを通じて渡されます。TLS接続もこのモードを使用してクライアントから終了できます。

このモードを使用するには、`stream_listen`を定義する以外に、プロトコルタイプが`tcp`または`tls`の適切なルート/サービスオブジェクトを作成する必要があります。

{{site.base_gateway}} による TLS 終了が必要な場合は、次の条件を満たす必要があります。

1. TLS接続が接続される{{site.base_gateway}}ポートでは、`ssl`フラグを有効にする必要があります
2. TLSの終了に使用できる証明書/キーは[ルートのTLSの構成](#configuring-tls-for-a-route)内に表示されるよう、{{site.base_gateway}}内に存在する必要があります。


{{site.base_gateway}}は接続しているクライアントのTLS SNIサーバー名拡張子を使用して、使用する適切なTLS証明書を見つけます。

サービス側では、{{site.base_gateway}}とアップストリームサービス間の接続を
暗号化する必要があるかどうかに応じて、`tcp`または`tls`のプロトコル型をそれぞれ設定できます。
つまり、このモードでは以下のすべてのセットアップがサポートされます。

1. クライアント <\- TLS \-> Kong <\- TLS \-> アップストリーム
2. クライアント<\- TLS \-> Kong<\- Cleartext \-> アップストリーム
3. Client <\- Cleartext \-> Kong <\- TLS \-> Upstream

**注:** L4 プロキシモードでは、サポートされるプロトコルリスト内の、`tcp` または `tls` を保持するプラグインのみがサポートされます。このリストは、[Kong Hub](/hub/) のそれぞれのドキュメントで確認できます。

[トップに戻る](#introduction)

プロキシ TLS パススルー トラフィック
---------------------


{{site.base_gateway}} は、終了させることなく TLS リクエストをプロキシする、
SNI プロキシをサポートしています。


{{site.base_gateway}}接続している
クライアントのTLS SNI拡張を使用して、一致するルートとサービスを検索し、
着信 TLS トラフィックを復号化せずに、完全なアップストリームの TLS 要求を転送します。

このモードでは次の手順を実行する必要があります。

* プロトコル`tls_passthrough`でルートおぷじぇくとを作成し、`snis`フィールドを1つ以上のSNIに設定します。
* 対応するサービスオブジェクトのプロトコルを`tcp`に設定します。
* `stream_listen` ディレクティブに `ssl` フラグがあるポートにリクエストを送信します。

個別のSNIおよび証明書エンティティは必要なく、使用されません。

ルートが同時に`tls`と`tls_passthrough`プロトコルを一致させることはできません。ただし、同じSNIは異なるルートで`tls`と`tls_passthrough`を一致できます。

ルートを`tls_passthrough`に設定し、サービスを`tls`に設定することもできます。この
モードでは、アップストリームへの接続は TLS で 2 回暗号化されます。

{:.note}
> 
> **注：** このモードでプラグインを実行するには、プラグインの `protocols` フィールドが`tls_passthrough` を含む必要があります。

[トップに戻る](#introduction)

結論
---

このガイドを通じて、リクエストが関連サービスにルーティングされるルートを照合する仕組みから、WebSocketプロトコルの使用を許可する方法、動的TLS証明書を設定する方法まで、{{site.base_gateway}}の基礎となるプロキシのメカニズムについての知識を得ていただけたことを願っています。

このウェブサイトはオープンソースであり、
[github.com/Kong/docs.konghq.com](https://github.com/Kong/docs.konghq.com/) 。
この文書へのフィードバックや改善提案など、お気軽にお寄せください。

まだお読みでない場合は、これまで取り上げてきたトピックに密接に関連する[ロードバランシング参考資料](/gateway/{{page.release}}/how-kong-works/load-balancing)を読むこともおすすめします。

