---
title: "サービスとルート"
content-type: "tutorial"
book: "get-started"
chapter: 2
---

{{site.base_gateway}}管理者は、オブジェクトモデルを使用して、必要なトラフィック管理ポリシーを定義します。そのモデル内の 2 つの重要なオブジェクトは、[サービス](/gateway/api/admin-ee/latest/#/Services)と[ルート](/gateway/api/admin-ee/latest/#/Routes/list-route)です。サービスとルートは、協調的な方法で構成され、リクエストとレスポンスがシステムを通過するルーティングパスを定義します。

以下の大まかな概要は、ルートに到着したリクエストがサービスに転送され、レスポンスは
逆の経路をとることを示しています。

{% mermaid %}
flowchart LR
  A(API client)
  B("`Route 
  (/mock)`")
  C("`Service
  (example_service)`")
  D(Upstream 
  application)
  
  A <--requests
  responses--> B
  subgraph id1 ["`
  **KONG GATEWAY**`"]
    B <--requests
    responses--> C
  end
  C <--requests
  responses--> D

  style id1 rx:10,ry:10
  
{% endmermaid %}

### サービスの定義

{{site.base_gateway}}では、サービスとは既存のアップストリームアプリケーションを抽象化したものです。サービスには、プラグイン構成やポリシーなどのオブジェクトのコレクションを保存でき、ルートと関連付けることができます。

サービスを定義する場合、管理者は *名前* とアップストリームアプリケーション接続情報を提供します。接続の詳細は、`url`フィールドに単一の文字列として入力するか、`protocol`、`host`、`port`、`path`の個別の値を個別に入力できます。

サービスはアップストリームアプリケーションと一対多の関係にあるため、管理者は高度なトラフィック管理行動を作成できます。

### ルートとは

ルートとは、アップストリームアプリケーション内のリソースへのパスのことです。ルートはサービスに追加され、基礎となるアプリケーションへの
アクセスを可能にします。{{site.base_gateway}}ではルートは通常、{{site.base_gateway}}アプリケーションで
公開されるエンドポイントにマップされます。ルートは、サービス関連するリクエストに一致するルールを
定義することもできます。このため、1つのルートで複数のエンドポイントを参照できます。基本的なルートには名前、
パス、既存のサービスのリファレンスが必要です。

また、次の方法でルートを構成することもできます。

* プロトコル：アップストリームアプリケーションとの通信に使用されるプロトコル。
* ホスト：ルートに一致するドメインのリスト
* メソッド：ルートに一致する HTTP メソッド
* ヘッダー: リクエストのヘッダー内に想定される値の一覧
* リダイレクトステータスコード：HTTPSステータスコード
* タグ：ルートをグループ化するための文字列のオプションセット


{{site.base_gateway}}がリクエストをどうルーティングするかの説明については、[ルート](/gateway/{{page.release}}/key-concepts/routes/)
を参照してください。

サービスとルートの管理
-----------

次のチュートリアルでは、
{{site.base_gateway}} の [Admin API](/gateway/latest/admin-api/) を使用したサービスやルートの管理やテストに関して一通り説明します。{{site.base_gateway}} は、[{{site.konnect_saas}}](/konnect/) と [decK](/deck/latest/) を含む、コンフィギュレーション管理のための他のオプションも提供しています。

チュートリアルのこのセクションでは、次の手順を完了します。

* HTTP リクエストとレスポンス用テスト機能を提供する[httpbin](https://httpbin.org/) APIを指す サービスを作成する。
* 実行中の{{site.base_gateway}}上のクライアントが利用できる URL パスを指定してルートを定義します。
* 新しいhttpbinサービスを使用してテストリクエストをエコーすると、
  {{site.base_gateway}} APIリクエストのプロキシ方法を理解しやすくなります。 

### 前提条件

この章は、「 *Get Started with Kong* 」シリーズの一部です。最適な効果を得るために、
シリーズを最初からご覧になることをお勧めします。

導入である[Get Kong](/gateway/latest/get-started/)には、ローカル{{site.base_gateway}}を実行するためのツールの前提条件と手順が含まれています。

[Get Kong](/gateway/latest/get-started/)の手順をまだ完了していない場合は、
続行する前に完了してください。

### サービスの管理

1. **サービスの作成** 

   新しいサービスを追加するには、 `POST`リクエストを{{site.base_gateway}}の
   Admin API `/services`ルートに送信します。

   ```sh
   curl -i -s -X POST http://localhost:8001/services \
     --data name=example_service \
     --data url='http://httpbin.org'
   ```

   上記リクエストでは、アップストリーム URL `http://httpbin.org`にマップされる新しいサービスを作成するよう{{site.base_gateway}}に指示しています。

   上記例にはリクエストボディに次の2つの文字列が含まれています。
   * `name`：サービス名
   * `url` : サービスの`host`、`port`、および`path`属性を設定する引数

   リクエストが成功すると、{{site.base_gateway}} からの `201` レスポンスヘッダーが表示され、
   サービスが作成され、レスポンスボディが次のようになっていることを確認します。

   ```text
   {
     "host": "httpbin.org",
     "name": "example_service",
     "enabled": true,
     "connect_timeout": 60000,
     "read_timeout": 60000,
     "retries": 5,
     "protocol": "http",
     "path": null,
     "port": 80,
     "tags": null,
     "client_certificate": null,
     "tls_verify": null,
     "created_at": 1661346938,
     "updated_at": 1661346938,
     "tls_verify_depth": null,
     "id": "3b2be74e-335b-4f25-9f08-6c41b4720315",
     "write_timeout": 60000,
     "ca_certificates": null
   }
   ```

   作成リクエストで明示的に指定されていないフィールドは
   現在の{{site.base_gateway}}構成に基づくデフォルト値が自動的に付与されます。
2. **サービス構成の表示** 

   サービスを作成すると、上記の応答に示すように、{{site.base_gateway}}によって一意の`id`が割り当てられます。
   `id`フィールド、またはサービスの作成時に指定された名前は、後続のリクエストでサービスを
   識別するために使用できます。これはサービスのURLで、`/services/{service name or id}`の形式を取ります。

   サービスの現在の状態を表示するには、サービスURLに対して`GET`リクエストを行います。

   ```sh
   curl -X GET http://localhost:8001/services/example_service
   ```

   リクエストが成功すると、レスポンスボディにはサービスの現在の構成が含まれ、次のスニペットのようになります。

   ```text
   {
     "host": "httpbin.org",
     "name": "example_service",
     "enabled": true,
     ...
   }
   ```

3. **サービスの更新** 

   既存のサービス構成は、 `PATCH`リクエストをサービス URL に送信することで
   動的に更新できます。

   サービス再試行を`5`から`6`に動的に設定するには、この`PATCH`リクエストを送信します。

   ```sh
   curl --request PATCH \
     --url localhost:8001/services/example_service \
     --data retries=6
   ```

   応答本文には、更新された値を含む完全なサービス構成が含まれています。

   ```sh
   {
     "host": "httpbin.org",
     "name": "example_service",
     "enabled": true,
     "retries": 6,
     ...
   }
   ```

4. **サービスの一覧表示** 

   ベースの`/services`URLに`GET`リクエストを送信すると、現在のすべてのサービスを一覧表示できます。

   ```sh
   curl -X GET http://localhost:8001/services
   ```

[管理APIドキュメント](/gateway/latest/admin-api/#update-service)は、完全なサービス更新仕様を提供します。

ブラウザで次の URL に移動すると、Kong Manager UI でサービスの構成を表示することもできます。

* Kong Manager OSS: [http://localhost:8002/services](http://localhost:8002/services)
* Kong Manager Enterprise: [http://localhost:8002/default/services](http://localhost:8002/default/services)（`default` はワークスペース名）。

### ルートの管理

1. **ルートを作成** 

   ルートは、リクエストが{{site.base_gateway}}によってプロキシされる方法を定義します。`POST`リクエストをサービスURLに送信することで、
   特定のサービスに関連するルートを作成できます。

   `/mock`パスに新しいルートを設定して、トラフィックを以前に作成した`example_service`サービスに誘導します。

   ```sh
   curl -i -X POST http://localhost:8001/services/example_service/routes \
     --data 'paths[]=/mock' \
     --data name=example_route
   ```

   ルートが正常に作成された場合、API は次のような`201`応答コードと応答本文を返します。

   ```text
   {
     "paths": [
       "/mock"
     ],
     "methods": null,
     "sources": null,
     "destinations": null,
     "name": "example_route",
     "headers": null,
     "hosts": null,
     "preserve_host": false,
     "regex_priority": 0,
     "snis": null,
     "https_redirect_status_code": 426,
     "tags": null,
     "protocols": [
       "http",
       "https"
     ],
     "path_handling": "v0",
     "id": "52d58293-ae25-4c69-acc8-6dd729718a61",
     "updated_at": 1661345592,
     "service": {
       "id": "c1e98b2b-6e77-476c-82ca-a5f1fb877e07"
     },
     "response_buffering": true,
     "strip_path": true,
     "request_buffering": true,
     "created_at": 1661345592
   }
   ```

2. **ルート設構成の表示** 

   サービスと同様に、ルートを作成すると、上記の応答に示すように、{{site.base_gateway}}が
   一意の`id`を割り当てます。ルートを作成するときは、後続のリクエストでルートを識別するために`id`フィールド、または指定された名前を
   使用できます。
   ルートURLは、次のいずれかの形式になります。
   * `/services/{service name or id}/routes/{route name or id}`
   * `/routes/{route name or id}`

   `example_route`ルートの現在の状態を表示するには、ルートURLに`GET`リクエストを行います。

   ```sh
   curl -X GET http://localhost:8001/services/example_service/routes/example_route
   ```

   応答本文には、ルートの現在の構成が含まれています。

   ```text
   {
     "paths": [
       "/mock"
     ],
     "methods": null,
     "sources": null,
     "destinations": null,
     "name": "example_route",
     "headers": null,
     "hosts": null,
     "preserve_host": false,
     "regex_priority": 0,
     "snis": null,
     "https_redirect_status_code": 426,
     "tags": null,
     "protocols": [
       "http",
       "https"
     ],
     "path_handling": "v0",
     "id": "189e0a57-205a-4f48-aec6-d57f2e8a9985",
     "updated_at": 1661347991,
     "service": {
       "id": "3b2be74e-335b-4f25-9f08-6c41b4720315"
     },
     "response_buffering": true,
     "strip_path": true,
     "request_buffering": true,
     "created_at": 1661347991
   }
   ```

3. **ルートの更新** 

   サービスと同様に、ルートはルート URL に `PATCH` リクエストを送信することで動的に更新できます。

   タグは、グループ化やフィルタリングのためにルートに関連付けることができるオプションの文字列セットです。
   `PATCH` リクエストを[サービスエンドポイント](/gateway/latest/admin-api/#update-route)に送り、ルートを指定することによって、タグを割り当てることができます。

   `tutorial` の値を持つタグを割り当ててルートを更新します。

       curl --request PATCH \
         --url localhost:8001/services/example_service/routes/example_route \
         --data tags="tutorial"

   上記の例では、ルート URL にサービスおよびルートの `name` フィールドを使用しました。

   タグが正常に適用された場合、応答本文には次のJSON値が含まれます。

   ```text
   ...
   "tags":["tutorial"]
   ...
   ```

4. **ルートの一覧表示** 

   Admin APIは、現在構成されているすべてのルートの一覧表示もサポートしています。

   ```sh
   curl http://localhost:8001/routes
   ```

   このリクエストは、HTTP `200`ステータスコードと、この{{site.base_gateway}}インスタンスに設定されているルート
   すべての内容を含むJSONレスポンスボディオブジェクト配列を返します。。応答は次のようになります。

   ```text
   {
     "next": null,
     "data": [
       {
         "paths": [
           "/mock"
         ],
         "methods": null,
         "sources": null,
         "destinations": null,
         "name": "example_route",
         "headers": null,
         "hosts": null,
         "preserve_host": false,
         "regex_priority": 0,
         "snis": null,
         "https_redirect_status_code": 426,
         "tags": [
           "tutorial"
         ],
         "protocols": [
           "http",
           "https"
         ],
         "path_handling": "v0",
         "id": "52d58293-ae25-4c69-acc8-6dd729718a61",
         "updated_at": 1661346132,
         "service": {
           "id": "c1e98b2b-6e77-476c-82ca-a5f1fb877e07"
         },
         "response_buffering": true,
         "strip_path": true,
         "request_buffering": true,
         "created_at": 1661345592
       }
     ]
   }
   ```

[管理APIドキュメント](/gateway/api/admin-ee/latest/#/Routes/list-route/)にはルートオブジェクトの管理に必要な情報が網羅されています。

Kong Manager UIでルートの設定を表示するには、ブラウザで次のURLに移動します：[http://localhost:8002/default/routes](http://localhost:8002/default/routes)

リクエストのプロキシ
----------

KongはAPIゲートウェイであり、クライアントからのリクエストを受信し、現在の構成に基づいて適切なアップストリームアプリケーションをルーティングします。以前に公正されたサービスをルートを使用すると、`http://localhost:8000/mock`を使用して`https://httpbin.org/`にアクセスできるようになりました。

デフォルトでは、 {{site.base_gateway}}のAdmin APIはポート`8001`で管理リクエストをリッスンします。これは、 *コントロールプレーン（CP）* と呼ばれる場合があります。クライアントはポート `8000` を使用してデータ要求を行います。これは多くの場合、 *データプレーン（DP）* と呼ばれます。

Httpbin は、クライアントに対して行われたリクエストに関する情報をエコーバックする`/anything`リソースを提供します。{{site.base_gateway}}を介してリクエストを`/anything`リソースにプロキシします。

```sh
curl -X GET http://localhost:8000/mock/anything
```

次のような応答が表示されます。

```text
{
  "startedDateTime": "2022-08-24T13:44:28.449Z",
  "clientIPAddress": "172.19.0.1",
  "method": "GET",
  "url": "http://localhost/anything",
  "httpVersion": "HTTP/1.1",
  "cookies": {},
  "headers": {
    "host": "httpbin.org",
    "connection": "close",
    "accept-encoding": "gzip",
    "x-forwarded-for": "172.19.0.1,98.63.188.11, 162.158.63.41",
    "cf-ray": "73fc85d999f2e6b0-EWR",
    "x-forwarded-proto": "http",
    "cf-visitor": "{\"scheme\":\"http\"}",
    "x-forwarded-host": "localhost",
    "x-forwarded-port": "80",
    "x-forwarded-path": "/mock/anything",
    "x-forwarded-prefix": "/mock",
    "user-agent": "curl/7.79.1",
    "accept": "*/*",
    "cf-connecting-ip": "00.00.00.00",
    "cdn-loop": "cloudflare",
    "x-request-id": "1dae4762-5d7f-4d7b-af45-b05720762878",
    "via": "1.1 vegur",
    "connect-time": "0",
    "x-request-start": "1661348668447",
    "total-route-time": "0"
  },
  "queryString": {},
  "postData": {
    "mimeType": "application/octet-stream",
    "text": "",
    "params": []
  },
  "headersSize": 588,
  "bodySize": 0
}
```

