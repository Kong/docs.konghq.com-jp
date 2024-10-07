---
title: "DBレスで宣言型構成"
---
従来、{{site.base_gateway}} は、ルート、サービス、プラグインなどの構成されたエンティティを保存するためにデータベースを常に必要としていました。Kongは、`kong.conf` という構成ファイルを使用して、データベースの利用とその[さまざまな設定](/gateway/{{page.release}}/reference/configuration/#datastore-section)を指定します。


{{site.base_gateway}} 、エンティティのメモリ内ストレージのみを使用して、データベースなしで実行できます。
これをDBレスモードと呼んでいます。{{site.base_gateway}} DB\-Lessを実行している場合、エンティティの構成は、
宣言型設定を使用して、YAMLまたはJSONの2つ目の設定ファイルで行われます。

{% include_cached /md/gateway/deployment-topologies.md topology='dbless' %}
> 
> *図1：DBレスモードでは、YAMLファイルを介して構成が適用されます。
> 
{{ site.base_gateway }}ノードはデータベースにも他のノードにも接続されていません。* 

DBレス・モードと宣言的コンフィギュレーションの組み合わせには、次のような利点があります。

* 依存関係の数の削減：ユースケースの設定全体がメモリ内に収まる場合は、データベースのインストールを管理する必要はありません。
* CI/CD シナリオにおける自動化: エンティティの構成は、Git リポジトリを通して管理される信頼できる唯一の情報源内に保持できます。
* {{site.base_gateway}}のデプロイメントオプションの幅が広がります。

{:.note}
> 
> **注：** [decK](/deck/)は構成を宣言型でも管理します。ただし、その場合はデータベースが同期、ダンプ、または同様の操作を実行する必要があります。
> そのためDBレスモードではdecKを使用できません。

宣言型構成
-----

宣言型構成の重要な考え方は、それが *命令型* ではなく、その逆の *宣言型* であるということです。「命令型」とは、構成が「これを実行して、それからあれを実行しなさい」という一連のものとして与えられることを意味します。「宣言型」とは、「私はこれが世界の現状であると宣言する」といったように、構成が一度に与えられることを意味します。

Kong Admin APIは、命令型構成ツールの例です。構成の
最新状態は、一連のAPI呼び出しで獲得できます。1つの呼び出しはサービスを
生成し、もう1つの呼び出しはルートを生成し、その他の呼び出しはプラグイン
などを追加します。

このような段階的な設定は、 *中間状態* が発生するという望ましくない
副作用をもたらします。上の例では、ルートを作成してからプラグインを追加するまでの間、ルートには適用されるプラグインがないという時間枠があります。

一方、宣言型設定ファイルには
必要なすべてのエンティティ用の設定が1つのファイルにまとめられています。そのファイルが読み込まれると

{{site.base_gateway}} 、構成全体が置き換えられます。段階的な変更の場合、
それらは宣言型構成ファイルに入れられます。そして
完全にリロードされます。常に、
Kong にロードされるファイルに記述されている構成が、システムの構成状態です。

DB レスモードでKongを設定
----------------

DBレスモードで{{site.base_gateway}}を使用するには、 `kong.conf`の`database`ディレクティブを`off`に設定します。 通常どおり、`kong.conf`を編集して`database=off`を設定するか、環境変数を使用してこれを実行できます。その後、通常どおりKongを起動できます。

    export KONG_DATABASE=off
    kong start -c kong.conf

Kongが起動したら、Admin APIの`/`エンドポイントにアクセスして、データベースなしで
実行されていることを確認します。Kongの構成全体が返されます。
`database`が`off`に設定されていることを確認します。

コマンド：

```sh
curl -i -X GET http://localhost:8001
```

応答例：

```json
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 6342
Content-Type: application/json; charset=utf-8
Date: Wed, 27 Mar 2019 15:24:58 GMT
Server: kong/{{page.versions.ce}}
{
    "configuration:" {
       ...
       "database": "off",
       ...
    },
    ...
    "version": "{{page.versions.ce}}"
}
```


{{site.base_gateway}}は実行中ですが、宣言型構成はまだ読み込まれていません。これは
、このノードの構成が空であることを意味します。ルート、
サービス、またはどんな種類のエンティティもありません。

コマンド：

```sh
curl -i -X GET http://localhost:8001/routes
```

応答例：

```json
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 23
Content-Type: application/json; charset=utf-8
Date: Wed, 27 Mar 2019 15:30:02 GMT
Server: kong/{{page.versions.ce}}

{
    "data": [],
    "next": null
}
```

宣言型構成ファイルの作成
------------

DBレスKongにエンティティをロードするには、宣言型の構成ファイルが必要です。次のコマンドで、開始するためのスケルトンファイルを作成します。

    kong config -c kong.conf init

このコマンドを実行すると、エンティティとその関係を宣言する構文の例が含まれる`kong.yml`ファイルが現在のディレクトリに作成されます。
このファイルのすべての例はデフォルトでコメントアウトされています。
例のコメント機能を解除（`#`マーカーを削除）したりその値を変更したりして実験できます。

宣言型構成形式
-------

{{site.base_gateway}} の宣言型設定フォーマットは
エンティティとその属性のリストで構成されます。これは小さいながらも完全な例で、
多くの機能を説明します。

```yaml
_format_version: "3.0"
_transform: true

services:
- name: my-service
  url: https://example.com
  plugins:
  - name: key-auth
  routes:
  - name: my-route
    paths:
    - /

consumers:
- username: my-user
  keyauth_credentials:
  - key: my-key
```

すべての設定オプションについては、[宣言型設定スキーマ](https://github.com/Kong/go-database-reconciler/blob/main/pkg/file/kong_json_schema.json)
を参照してください。

必須のメタデータは`_format_version: "3.0"`のみであり、
宣言型構成構文形式のバージョン番号を指定します。
これは、ファイルを解析するために必要なKongの最小バージョンとも一致します。

`_transform`メタデータはオプションのブーリアン（デフォルトは`true`）であり、スキーマの変換がインポート中に実行されるかどうかを制御します。このフィールドを使用する場合の大まかなルールは、プレーンテキストの認証情報（例：パスワード）をインポートする場合、Kongがデータベースにその情報を保存する前に暗号化/ハッシュできるよう、`true`に設定することです。 **すでにハッシュ/暗号化された** 認証情報をインポートする場合は、暗号化が2回行われないように`_transform`を`false`に設定します。

最上位では、コアエンティティなど、
上記の例のように`services`と`consumers`、または`keyauth_credentials`のようなプラグインによって作成されたカスタムエンティティ
で任意のKongエンティティを指定できます。これにより、宣言型
構成フォーマットは本質的に拡張可能であり、宣言型構成を処理する `kong config` コマンドが`kong.conf` を必要とするのは、
`plugins` ディレクティブが考慮されるようにするためです。

サービスを指すルートなど、エンティティが関係を持つ場合、この関係はネストによって指定できます。

ネストで指定できるのは 1 対 1 の関係だけです。上記の例のようにサービスに適用されるプラグインは、ネストを使ってその関係を示すことができます。サービスとコンシューマの両方に適用されるプラグインのように、2 つ以上のエンティティが関わる関係は、トップレベルのエントリ経由で行う必要があります。その場合、エンティティはプライマリキーや識別名（Admin API で参照する際に使用できる同じ識別子）で特定できます。以下は、サービスとコンシューマに適用されたプラグインの例です。

```yml
plugins:
- name: syslog
  consumer: my-user
  service: my-service
```

ファイルのチェック
---------

ファイルの編集が終わったら、{{site.base_gateway}} への読み込み前に構文のエラーがないか確認できます。

    kong config -c kong.conf parse kong.yml

レスポンス: 

    parse successful

ファイルの読み込み
---------

宣言型構成ファイルをKongにロードするには、次の2つの方法があります。\\n`kong.conf`または Admin APIです。

Kong の起動時に宣言型構成ファイルを読み込むには、`kong.conf` 内で `declarative_config` ディレクティブ（または、通常の `kong.conf` エントリと同じく、同等の環境変数 `KONG_DECLARATIVE_CONFIG`）を使用します。

    export KONG_DATABASE=off \
    export KONG_DECLARATIVE_CONFIG=kong.yml \
    kong start -c kong.conf

Admin APIで`/config`エンドポイントを
使用して、宣言型構成ファイルを実行中のKongノードにロードすることもできます。次の
例では`kong.yml`をロードします。

```sh
curl -i -X POST http://localhost:8001/config \
  --form config=@kong.yml
```

{:.important}
> 
> `/config` エンドポイントはメモリ内のエンティティのセット全体を、指定されたファイルに記述されたエンティティのセットで置き換えます。

KongをDBレスモードで起動するもう一つの方法は、`KONG_DECLARATIVE_CONFIG_STRING`環境変数を使用して、宣言型構成全体を文字列に含めることです。

    export KONG_DATABASE=off
    export KONG_DECLARATIVE_CONFIG_STRING='{"_format_version":"1.1", "services":[{"host":"httpbin.org","port":443,"protocol":"https", "routes":[{"paths":["/"]}]}],"plugins":[{"name":"rate-limiting", "config":{"policy":"local","limit_by":"ip","minute":3}}]}'
    kong start

Helm を使用した DB レス（KIC が無効）
-------------------------

{{site.base_gateway}}を、Kubernetesでイングレスコントローラ（`ingressController.enabled: false`）を使用せずDB\-lessモード（`env.database: "off"`）でデプロイする場合、実行するための{{site.base_gateway}}の宣言型構成を提供する必要があります。既存のConfigMap（`dblessConfig.configMap`）を提供するか、構成全体を`values.yaml`（`dblessConfig.config`）パラメータに配置することができます。詳細については、[デフォルトの`values.yaml`](https://github.com/Kong/charts/blob/main/charts/kong/values.yaml)の構成例を参照してください。

完全な宣言型構成ファイルを置き換えるには、Helm コマンドで`--set-file dblessConfig.config=/path/to/declarative-config.yaml`使用します。

外部から提供されたConfigMapは、デプロイメントアノテーションでハッシュ化または追跡されません。それ以降のConfigMapの更新では、新しい構成を適用するためにユーザーが開始するデプロイメントロールアウトが必要になります。外部から提供されたConfigMapコンテンツを更新した後、 `kubectl rollout restart deploy`を実行します。

DB\-lessモードでKongを使用
--------------------

DB レスモードで Kong を使用する際には注意すべき点がいくつかあります。

### メモリキャッシュの要件

エンティティの構成全体がKongキャッシュ内に収まる必要があります。メモリ内キャッシュが適切に構成されていることを確認します。`kong.conf`の`mem_cache_size`ディレクティブを参照してください。

### 中央データベースでの調整を排除

中央のデータベースがないため、複数の Kong ノードは中央の調整ポイントを持たず、クラスタのデータ伝播も行われません。つまり、ノード同士は完全に独立しています。

これは、宣言型構成を各ノードに個別にロードする必要があることを意味します。ノードは互いを認識していないため、`/config` エンドポイントを使用しても他の Kong ノードへの影響はありません。

### 読み取り専用Admin API

エンティティを構成する唯一の方法は宣言的な構成であるため、エンティティに
対するCRUD操作のエンドポイントは、DB\-lessモードでKongを実行中の
場合、Admin APIでは実質的に読み取り専用です。エンティティの検査のための`GET`
操作は通常通り機能しますが、`/services`や`/plugins`のようなエンドポイントでの`POST`、
`PATCH`、`PUT`または`DELETE`の試行では
`HTTP 405 Not Allowed`を返します。

この制限は、それ以外のデータベース操作に限定されます。特に、
`POST`を使用してターゲットのヘルス状態を設定することは、ノード固有の
メモリ内操作であるため、引き続き有効です。

#### Kong Manager の互換性

Kong Manager は、DB レスモードで動作する{{site.base_gateway}}との互換性を保証できません。{{site.base_gateway}}がこのモードで実行されている場合、Kong Managerを使用してエンティティを作成、更新、または削除することはできません。グローバルページとワークスペース概要ページの「概要」セクションにあるエンティティカウンターも正しく機能しません。

#### プラグインの互換性

KongのすべてのプラグインがDBレスモードと互換性があるわけではありません。設計上、一部のプラグインでは中央データベースでの調整やエンティティの動的作成が必要になります。

現在のプラグイン互換性については、[プラグインの互換性](/hub/plugins/compatibility/)を参照してください。

参照
---

* [宣言型構成スキーマ](https://github.com/Kong/go-database-reconciler/blob/main/pkg/file/kong_json_schema.json)

