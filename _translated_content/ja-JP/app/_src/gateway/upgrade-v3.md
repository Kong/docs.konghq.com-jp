---
title: "Kong Gatewayのアップグレード"
---
`kong migrations`コマンドを使用して、メジャー、マイナー、およびパッチ{{site.base_gateway}}リリース
にアップグレードします。

コマンドを使用して、すべての{{site.base_gateway}}オープンソースエンティティを
{{site.base_gateway}}（Enterprise）へ移行することもできます。[{{site.ce_product_name}}から{{site.base_gateway}}への移行](/gateway/{{page.release}}/migrate-ce-to-ke/)を参照してください。

移行中に問題が発生した場合は、
[Kongサポート](https://support.konghq.com/support/s/)までお問い合わせください。

{{site.base_gateway}}リリースのアップグレードパス
-------------------

Kongは、製品のバージョン管理において、メジャーバージョン、マイナーバージョン、パッチバージョンを区別する構造化されたアプローチを採用しています。

3\.0\.xへのアップグレードは **メジャー** アップグレードです。
Kong 3\.0\.xが移行をサポートする最も低いバージョンは2\.1\.xです。

{:.important}
> 
> **重要** ：2\.8\.2以下のバージョンから3\.0\.xへの従来モードでのブルーグリーン移行はサポートされていません。
> 2\.8\.2\.xリリースには、ブルーグリーン移行のサポートが含まれています。ダウンタイムなしで従来モードの移行を実行する場合は、少なくとも2\.8\.2\.0にアップグレードしてから[3\.0\.xに移行してください](#migrate-db)。

最新バージョンに直接アップグレードすることもできますが、
このドキュメントに記載されている2\.xシリーズおよび3\.xシリーズ間における下位互換性のない変更
（直近のバージョンと以前のバージョンの両方）および
[オープンソース（OSS）](https://github.com/Kong/kong/blob/release/3.0.x/CHANGELOG.md#300)と
[Enterprise](/gateway/changelog/#3000)ゲートウェイの変更ログを確認してください。{{site.base_gateway}}以降は
オープンソースの基盤上に構築されているため、OSSにおける下位互換性のない変更はすべての{{site.base_gateway}}パッケージに影響します。


{{site.base_gateway}}では、1\.xから3\.0\.xへの直接アップグレードをサポートしていません。
1\.xを使用している場合は、まずは2\.1\.0にアップグレードしてください。その後、3\.0\.xにアップグレードしてください。

いずれの場合も、[アップグレードに関する考慮事項](#upgrade-considerations-and-breaking-changes)、[データベースの移行](#migrate-db)の各手順を順番に一読することをお勧めします。

アップグレードに関する考慮事項と下位互換性のない変更
--------------------------

アップグレードする前に、このリストを確認し、現在のインストールに影響する構成および下位互換性のない変更がないか確認してください。

### デプロイメント

Amazon Linux 1およびDebian 8 \(Jessie\) のコンテナおよびパッケージは非推奨となり、 {{site.base_gateway}}の新しいバージョンでは生成されなくなりました。

#### ブルーグリーンデプロイメント

**従来モード** ：2\.8\.1およびそれ以前のバージョンから3\.0\.0へのブルーグリーンアップグレードは現在サポートされていません。
これは既知の問題であり、次の2\.8リリースで修正される予定です。2\.8のリリースバージョンがリリースされたら、2\.xユーザーは3\.0へのブルーグリーンアップグレードを開始する前に、そのバージョンにアップグレードする必要があります。

**ハイブリッドモード** ：以下の[アップグレード手順](#migrate-db)を参照してください。

### 依存関係

提供されているバイナリパッケージ（DebianとRHELを除く）を使用している場合は、ゲートウェイに必要なすべての依存関係
がバンドルされているため、このセクションは省略できます。

{{ site.base_gateway }} 3\.0以降、DebianとRHELのイメージは最小限の依存関係でビルドされ、公開前に自動セキュリティスキャナを通じて実行されます。これらには、{{site.base_gateway}}を実行するために必要な最小限の要素だけが含まれています。
ベースイメージと依存関係をさらにカスタマイズする場合は、[独自のDockerイメージを構築](/gateway/{{page.release}}/install/docker/build-custom-images)できます。

Debian、RHELを使用している場合、または手動で依存関係を構築している場合は、以前のリリースから変更が加えられているため、最新のパッチを使用して再構築する必要があります。

{{site.base_gateway}} 3\.0\.xに必要なOpenRestyのバージョンは、[1\.21\.4\.1](https://openresty.org/en/ann-1021004001.html)です。アップグレードされたOpenRestyに加えて、[lua\-kong\-nginx\-module](https://github.com/Kong/lua-kong-nginx-module)の最新リリースを含む、この新しいバージョン用の正しい[OpenRestyパッチ](https://github.com/Kong/kong-build-tools/tree/master/openresty-build-tools/patches)が必要です。[kong\-build\-tools](https://github.com/Kong/kong-build-tools)リポジトリには[openresty\-build\-tools](https://github.com/Kong/kong-build-tools/tree/master/openresty-build-tools)が含まれており、必要なパッチやモジュールを使ってOpenRestyをより簡単にビルドすることができます。

### 移行

移行ヘルパーライブラリ（主にCassandraの移行に使用）は、{{site.base_gateway}}では提供されなくなりました。

PostgreSQLの移行で、カサンドラの移行の場合と同様に、`up_f`部分を含め、呼び出す関数を指定できるようになりました。
`up_f`部分は、`up`部分がPostgreSQLとカサンドラの両方のデータベースに対して実行された後に呼び出されます

### Kongプラグイン

インストールに新しいプラグインを追加する場合は、プラグイン名を指定して
`kong migrations up`を実行する必要があります。たとえば、`KONG_PLUGINS=tls-handshake-modifier`など。

3\.0リリースには以下の新しいプラグインが含まれています。

* [OpenTelemetry](/hub/kong-inc/opentelemetry/) \(`opentelemetry`\)
* [TLS Handshake Modifier](/hub/kong-inc/tls-handshake-modifier/) \(`tls-handshake-modifier`\)
* [TLS Metadata Headers](/hub/kong-inc/tls-metadata-headers/) \(`tls-metadata-headers`\)
* [WebSocket Size Limit](/hub/kong-inc/websocket-size-limit/) \(`websocket-size-limit`\)
* [WebSocket Validator](/hub/kong-inc/websocket-validator/)（`websocket-validator`）

現在、Kongプラグインでは`CREDENTIAL_USERNAME`（ `X-Credential-Username`）をサポートしていません。
認証情報のアップストリームヘッダーの設定時は、定数 `CREDENTIAL_IDENTIFIER`（`X-Credential-Identifier`）を使用してください。

プラグインのスコープをコンシューマグループに限定する機能は {{site.base_gateway}} バージョン3\.4で追加されました。異なるバージョンが混在する {{site.base_gateway}} クラスタ（3\.4コントロールプレーンで3\.3以上のデータプレーン）の実行は、コンシューマグループスコープのプラグインを使用している場合、サポートされません。

#### 廃止および変更されたパラメータ

[StatsD Advanced](/hub/kong-inc/statsd-advanced/)プラグインは非推奨となり、4\.0で削除されます。
現在、すべての機能が[StatsD](/hub/kong-inc/statsd/)プラグインで利用できるようになっています。

以下のプラグインの設定パラメータが変更または削除されました。必要に応じて、設定を慎重に確認し、更新する必要があります。

**[ACL](/hub/kong-inc/acl/)、[Bot Detection](/hub/kong-inc/bot-detection/)、および[IP Restriction](/hub/kong-inc/ip-restriction/)** 

* 非推奨の`blacklist`と`whitelist`構成パラメータを削除しました。代わりに `allow` または `deny` を使用します。

**[ACME](/hub/kong-inc/acme/)** 

* 設定パラメータのデフォルト値は`auth_method` `token`になりました。

**[AWS Lambda](/hub/kong-inc/aws-lambda/)** 

* AWSリージョンが必須になりました。`aws_region`フィールドパラメータまたは環境変数でプラグインを構成して設定できます。
* 現在、プラグインでは、`host`フィールドと`aws_region`フィールドを同時に設定できるようになり、常にSigV4署名が適用されます。

**[HTTP Log](/hub/kong-inc/http-log/)** 

* 以前は値の配列を取っていた`headers`フィールドは、ヘッダー名ごとに1つの文字列しか入力できなくなりました。

**[JWT](/hub/kong-inc/jwt/)** 

* 認証されたJWTはnginxコンテキスト（ `ngx.ctx.authenticated_jwt_token`）に入れられなくなりました。その名前で設定されている値に依存するカスタムプラグインは、3\.0にアップグレードする前に、代わりにKongの共有コンテキスト（ `kong.ctx.shared.authenticated_jwt_token` ）を使用するように更新する必要があります。

**[Prometheus](/hub/kong-inc/prometheus/)** 

* 高カーディナリティメトリックはデフォルトで無効になりました。

* 次のメトリクス名は、可能な限り標準化するために単位を追加する調整を加えました。

  * `http_status`を`http_requests_total`に変更。
  * `latency` `kong_request_latency_ms`（HTTP）、`kong_upstream_latency_ms`、`kong_kong_latency_ms`、`session_duration_ms`（ストリーム）に対して。 Kongのレイテンシとアップストリームのレイテンシは、異なるマグニチュードで動作できます。メモリのオーバーヘッドを削減するには、これらのバケットを分離します。
  * `kong_bandwidth`を`kong_bandwidth_bytes`に変更。
  * `nginx_http_current_connections`と `nginx_stream_current_connections` は `nginx_connections_total`に統合されました。
  * `request_count`と `consumer_status` は `http_requests_total`に統合されました。`per_consumer`構成が`false`に設定されている場合、`consumer`ラベルは空になります。`per_consumer`の構成が`true`の場合、`consumer`ラベルが入力されます。

* その他のメトリクスの変更：

  * `http_consumer_status`メトリクスを削除しました。
  * `http_requests_total`に新しいラベル`source`が追加されました。`exit`、`error`、`service`のいずれかに設定できます。
  * すべてのメモリメトリクスに新しいラベルが付けられました: `node_id`。
  * プラグインはステータスコード、レイテンシ、帯域幅、アップストリームのヘルスチェックメトリクスをデフォルトでエクスポートしません。今後も、`status_code_metrics`、`lantency_metrics`、`bandwidth_metrics`、`upstream_health_metrics`をそれぞれ設定することでこの機能をオンにできます。

**[Pre\-function](/hub/kong-inc/pre-function/)プラグインと[Post\-function](/hub/kong-inc/post-function/)プラグイン** 

* 非推奨の`config.functions`構成パラメータを`post-function`と`pre-function`プラグインのスキーマから削除しました。代わりに、`config.access`フェーズを使用してください。

**[StatsD](/hub/kong-inc/statsd/)** 

* サービスに関連するすべてのメトリック名に `service.`接頭辞 `kong.service.<service_identifier id="sl-md0000000">.request.count` が付くようになりました。

  * メトリクス`kong.<service_identifier id="sl-md0000000">.request.status.<status id="sl-md0000000">`は`kong.service.<service_identifier id="sl-md0000000">.status.<status id="sl-md0000000">`に改名されました。
  * メトリクス`kong.<service_identifier id="sl-md0000000">.user.<consumer_identifier id="sl-md0000000">.request.status.<status id="sl-md0000000">`は`kong.service.<service_identifier id="sl-md0000000">.user.<consumer_identifier id="sl-md0000000">.status.<status id="sl-md0000000">`に改名されました。

* メトリクス`status_count`および`status_count_per_user`からメトリクス`*.status.<status id="sl-md0000000">.total`が削除されました。

**[Proxy Cache](/hub/kong-inc/proxy-cache/)、[Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)、[GraphQL Proxy Cache Advanced](/hub/kong-inc/graphql-proxy-cache-advanced/)** 

* これらのプラグインは、応答データを`ngx.ctx.proxy_cache_hit`に保存しなくなりました。
* そのため、現在、応答データを必要とするログ取得プラグインは、データを`kong.ctx.shared.proxy_cache_hit`から読み取る必要があります。

### カスタムプラグインとPDK

* プラグインのDAOは、読み込み順序が明示的になるように、配列にリストされなければなりません。ハッシュのようなテーブルへの読み込みは、サポートされなくなりました。

* プラグインは、`handler.lua`ファイルに有効な`PRIORITY`（整数）と`VERSION`（"x.y.z" 形式）フィールドを持つことが必要になりました。
  そうでなければ、プラグインを読み込めません。

* 古い`kong.plugins.log-serializers.basic`ライブラリはPDK関数`kong.log.serialize` が優先されたため、削除されました
  。PDKを使用するにはプラグインをアップグレードしてください。

* 非推奨となっていたレガシーのプラグインスキーマのサポートは削除されました。カスタムプラグインがいまだに古い（`0.x era`）スキーマを使用している場合は、それらをアップグレードする必要があります。

* 一部のプラグインの優先度を更新しました。

  これは、プラグインが実行される順序に影響する可能性があるため、カスタムプラグインを実行する人にとって重要です。
  これにより、標準の{{site.base_gateway}}インストールにおけるプラグインの実行順序は変更されません。

  古いプラグインと新しいプラグインの優先度の値：
  * `acme`を`1007`から次の値へ変更：`1705`
  * `basic-auth`を`1001`から次の値へ変更：`1100`
  * `canary`を`13`から次の値へ変更：`20`
  * `degraphql`を`1005`から次の値へ変更：`1500`
  * `graphql-proxy-cache-advanced`を`100`から次の値へ変更：`99`
  * `hmac-auth`を`1000`から次の値へ変更：`1030`
  * `jwt`を`1005`から次の値へ変更：`1450`
  * `jwt-signer` `999`から`1020`に変更されました。
  * `key-auth`を`1003`から次の値へ変更：`1250`
  * `key-auth-advanced`を`1003`から次の値へ変更：`1250`
  * `ldap-auth`を`1002`から次の値へ変更：`1200`
  * `ldap-auth-advanced`を`1002`から次の値へ変更：`1200`
  * `mtls-auth`を`1006`から次の値へ変更：`1600`
  * `oauth2`を`1004`から次の値へ変更：`1400`
  * `openid-connect`を`1000`から次の値へ変更：`1050`
  * `rate-limiting`を`901`から次の値へ変更：`910`
  * `rate-limiting-advanced`を`902`から次の値へ変更：`910`
  * `route-by-header`を`2000`から次の値へ変更：`850`
  * `route-transformer-advanced`を`800`から次の値へ変更：`780`
  * `pre-function`を`+inf`から次の値へ変更：`1000000`
  * `vault-auth`を`1003`から次の値へ変更：`1350`

* `kong.request.get_path()` PDK関数は、呼び出し元に返される文字列においてパスの正規化を行うようになりました。生および非正規化バージョンのリクエストパスは
  `kong.request.get_raw_path()`経由で取得できます。

* `pdk.response.set_header()`、`pdk.response.set_headers()`、`pdk.response.exit()`は、手動で設定された`Transfer-Encoding`ヘッダーを無視し、警告を発するようになりました。

* PDKのバージョン管理は終了しました。

* JavaScript PDKは、`kong.request.getRawBody`、`kong.response.getRawBody`、`kong.service.response.getRawBody`で`Uint8Array`を返すようになりました。
  Python PDKは、`kong.request.get_raw_body`、`kong.response.get_raw_body`、`kong.service.response.get_raw_body`で`bytes`を返します。
  以前は、これらの関数は文字列を返していました。

* `go_pluginserver_exe`および`go_plugins_dir`ディレクティブはサポートされなくなりました。
  [Goプラグインサーバー](https://github.com/Kong/go-pluginserver)を使用している場合は、アップグレードする前に
  [Go PDK](https://github.com/Kong/go-pdk)を使用するようプラグインを移行してください。

* 3\.0以降、{{site.base_gateway}}のスキーマライブラリの`process_auto_fields`機能は、所与のコンテキストが`select`である場合、渡されるデータをディープコピーしません。
  その理由は、ほとんどの場合、データが`pgmoon`や`lmdb`のようなドライバーから取得されるとKongが考えるテーブルを過度にディープコピーしないようにするためです。

  カスタムプラグインが指定されたテーブルをオーバライドしない`process_auto_fields`に依存していた場合は、今すぐ関数に渡す前に独自のコピーを作成する必要があります。
* KongプラグインまたはDAOスキーマの非推奨の`shorthands`フィールドは削除され、
  代わりに入力された`shorthand_fields`が使用されます。カスタムスキーマでまだ`shorthands`を使用している場合は、
  `shorthand_fields`を使用するよう更新する必要があります。

* `legacy = true/false`属性のサポートは、KongスキーマおよびKongフィールドスキーマから削除されました。

* Kongシングルトンモジュール`kong.singletons`は、PDK `kong.*`が優先されたため削除されました。

### 新しいルーター

現在、
{{site.base_gateway}}では、`route.path` が正規表現パターンかどうかを推測するためにヒューリスティックを使用していません。3\.0以降は、すべての正規表現パスは `"~"` 接頭辞で始める必要があり、`"~"` で始まらないパスはすべてプレーンテキストと見なされます。
移行プロセスでは、2\.xから3\.0にアップグレードする際に、正規表現パスが自動的に変換されます。

`route.path`の正規化ルールが変更されました。{{site.base_gateway}}は、正規化されていないパスを保存するようになりましたが、正規表現のパスは常に正規化されたURIのパターンと一致します。以前は、正規表現のパスのパターンで{{site.base_gateway}}をパーセントエンコーディングに置き換えることで、異なる形式のURIを一致させていました。この仕組みはサポートされなくなりました。ただし、[rfc3986](https://datatracker.ietf.org/doc/html/rfc3986#section-2.2)で定義される予約文字は例外であり、他のすべての文字はパーセントエンコーディングなしに書き込まれます。

### 宣言型とDB\-less

宣言型構成のバージョン番号（`_format_version`）は、`route.path`の変更により`3.0`に引き上げられました。古いバージョンの宣言型構成は、移行中に`3.0`にアップグレードされます。

{:.important}
> 
> **宣言型構成ファイルを2\.8以前から3\.0に同期（`deck gateway sync`）しないでください。**
> 古い構成ファイルが構成を上書きし、互換性の問題を引き起こします。
> 更新された構成を取得するには、移行が完了した後に3\.0ファイルの`deck gateway dump`を実行します。

`.lua`形式を使用してCLIツールの`kong`から宣言型構成ファイルをインポートできなくなりました。JSON形式とYAML形式のみがサポートされています。{{site.base_gateway}}でのアップデート手順に`kong config db_import config.lua`の実行が含まれる場合、`config.lua`ファイルを`config.json`または`config.yml`ファイルに変換してからアップグレードします。

### Admin API

{% if_version lte:3.4.x %}
Admin APIエンドポイント`/vitals/reports`は削除されました。`/targets`エンドポイントに対する{% endif_version %}
`POST`は、既存エンティティを更新できなくなりました。このリクエストでは新しいエンティティの作成のみできます。`/targets`を変更するために`POST`リクエストを使用するスクリプトがある場合、適切なエンドポイントへの`PUT`リクエストに変更してからKong 3\.0に更新します。

サーバーで利用できると報告されたプラグインのリストは、ブール値の`true`ではなく、プラグインごとのメタデータのテーブルを返すようになりました。

### 構成

`X-Credential-Username`の値を持つKong定数`CREDENTIAL_USERNAME`は
削除されました。

システムCAストアから信頼されたCAリストを自動的に読み込むために、デフォルト値の`lua_ssl_trusted_certificate`が`system`に変更されました。

データプレーンの構成キャッシュメカニズムとそれに関連する構成オプション
（`data_plane_config_cache_mode`と`data_plane_config_cache_path`）は削除され、代わりにLMDBが使用されます。

`ngx.ctx.balancer_address`は、`ngx.ctx.balancer_data`を優先し、削除されました。

### KubernetesのKongに関する考慮事項

Helmチャートはアップグレード移行プロセスを自動化します。`helm upgrade`を実行すると、
チャートは`kong migrations up`を実行するための初期ジョブを生成し、その後、更新されたバージョンで新しい
Kongポッドを生成します。これらのポッドの準備が整うと、トラフィックの処理が開始されて
古いポッドは終了します。これが完了すると、チャートは
`kong migrations finish`を実行するために別のジョブを生成します。

移行自体は自動化されていますが、チャートは、推奨されているアップグレードパスに従うことを自動的に保証するものではありません。複数のマイナー
{{site.base_gateway}}バージョンからさかのぼってアップグレードする場合は、推奨されるアップグレードパスを確認してください。

必須ではありませんが、ユーザーはチャートのバージョンと{{site.base_gateway}}バージョンを個別にアップグレードする必要があります。問題が発生した場合、これは問題がKubernetesリソースの変更に起因するものか、{{site.base_gateway}}の変更変更に起因するものかを明確にするのに役立ちます。

Kong for Kubernetesのバージョンアップグレードに関する具体的な考慮事項については、
[アップグレードに関する考慮事項](https://github.com/Kong/charts/blob/main/charts/kong/UPGRADE.md)を参照してください。

#### Kongデプロイメントは複数のリリースに分割されています

標準のチャートアップグレード自動化プロセスでは、{{site.base_gateway}}クラスタ内で{{site.base_gateway}}リリースが1つだけであると想定しており、`migrations up`と`migrations finish`の両方のジョブを実行します。

{{site.base_gateway}}デプロイメントを複数のHelmリリースに分割する場合（たとえば、プロキシ専用ノードや管理者専用ノードなど）、アップグレードの順番に応じて、どの移行ジョブを実行するかを設定する必要があります。

複数のリリースに分割されたクラスタを処理するには、次の操作を行う必要があります。

1. 次のいずれかのリリースをアップグレードします。

   ```shell
   helm upgrade RELEASENAME -f values.yaml \
   --set migrations.preUpgrade=true \
   --set migrations.postUpgrade=false
   ```

2. 残りのリリースのうち、1つを除くすべてを次のようにアップグレードします。

   ```shell
   helm upgrade RELEASENAME -f values.yaml \
   --set migrations.preUpgrade=false \
   --set migrations.postUpgrade=false
   ```

3. 最終リリースを次のようにアップグレードします。

   ```shell
   helm upgrade RELEASENAME -f values.yaml \
   --set migrations.preUpgrade=false \
   --set migrations.postUpgrade=true
   ```

これにより、すべてのインスタンスが、
`kong migrations finish`を実行する前に新しい{{site.base_gateway}}パッケージを使用していることが保証されます。

### ハイブリッドモードに関する考慮事項

{:.important}
> 
> **重要:** 現在[ハイブリッドモード](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/)で実行している場合は、
> 最初にコントロールプレーン（CP）をアップグレードし、次にデータプレーン（DP）をアップグレードします。

* 現在、クラシック（従来モード）で2\.8\.xバージョンを実行している場合で、ハイブリッドモードの実行に切り替えたい場合は、移行を行った後、ハイブリッドモード [インストール手順](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/setup/) に従ってください。
* カスタムプラグイン（独自のプラグインまたは{{site.base_gateway}}に付属していないサードパーティ製プラグイン）は、ハイブリッドモードでコントロールプレーン（CP）とデータプレーン（DP）の両方にインストールする必要があります。プラグインは、最初にコントロールプレーン（CP）にインストールし、次にデータプレーン（DP）にインストールします。
* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)プラグインは、 ハイブリッドモードでの `cluster`ストラテジをサポートしていません。代わりに、`redis`ストラテジを使用する必要があります。

### テンプレートの変更

2\.0\.x以降の{{site.base_gateway}}のマイナーバージョンとメジャーバージョンの間では、Nginx構成ファイルに変更があります。

3\.0\.xでは、非推奨の`Kong.serve_admin_api`エイリアスが削除されました。
カスタムNginxテンプレートをまだ使用している場合は、 `Kong.admin_content`に変更してください。

{% navtabs %}
{% navtab OSS %}
バージョン間のすべての構成変更を表示するには、[Kongリポジトリ](https://github.com/kong/kong)を複製し、構成テンプレート上で `git diff` を実行します。読みやすくするためには `-w` を使用します。

以前のバージョンと3\.0\.xの違いを確認する方法は次のとおりです。

    git clone https://github.com/kong/kong
    cd kong
    git diff -w 2.0.0 3.0.0 kong/templates/nginx_kong*.lua

開始バージョン番号（例では2\.0\.0）を現在使用しているバージョン番号に調整します。

パッチファイルを生成するには、次のコマンドを使用します。

    git diff 2.0.0 3.0.0 kong/templates/nginx_kong*.lua > kong_config_changes.diff

開始バージョン番号を現在使用しているバージョン番号（例では2\.0\.0）に調整します。

{% endnavtab %}
{% navtab Enterprise %}

{{site.base_gateway}}のデフォルトテンプレートは、{{site.base_gateway}}インスタンスを実行しているシステム上で、以下のコマンドを使用して見つけることができます：`find / -type d -name "templates" | grep kong`

アップグレードするときは、必ず古いクラスタと新しいクラスタの両方でこのコマンドを実行してください。
ファイルを比較して変更箇所を特定し、必要に応じて適用します。

{% endnavtab %}
{% endnavtabs %}

2\.1\.xからのアップグレード\- 2\.8\.xから3\.0\.xへ\{\#migrate\-db\}
-----------------------------------------------------------------

バックアップデータストアを新しいバージョンに移行するには、次の手順に従います。
新しいデータストアを使用して`kong.conf`ファイルのみを移行する場合は、
[新しいデータストアに3\.0\.xをインストールする](#install-30x-on-a-fresh-data-store)手順を参照してください。

すべてのアップグレードと同様に、プロセスを開始する前にデータストアのバックアップを作成します。

移行中にAdmin APIで構成を変更しないでください。予期しない動作や構成の破壊につながる可能性があります。

**バージョン3\.0\.xに移行するためのバージョン前提条件** 

2\.7\.xまたはそれ以前のバージョンから移行する場合は、まず[2\.8\.1に移行](#upgrade-from-10x---22x-to-28x)します。

2\.8\.xに移行したら、以下のセクションの手順に従い、3\.0\.xに移行します。

### 2\.8\.x（x>=2）から3\.0\.xへのアップグレード 従来モード（ブルーグリーン）の場合

{:.note}
> 
> これらの手順は2\.8\.2\.xでのみ機能します。

1. データベースを複製します。
2. 3\.0\.xをダウンロードして、複製されたデータストアを指すように構成します。 `kong migrations up`と`kong migrations finish`を実行します。
3. 3\.0\.xクラスタを起動します。
4. 現在、古いバージョン（2\.8\.x）と新しいバージョン（3\.0\.x）の両方で クラスタを同時に実行できるようになりました。3\.0\.xノードのプロビジョニングを開始します。
5. 徐々に、古いノードから 3\.0\.xクラスタへトラフィックを移行します。トラフィックを監視し、移行がスムーズに行われているか 確認します。
6. トラフィックが3\.0\.xクラスタに完全に移行されると、古いノードを廃止します。

### 3\.0\.xへのアップグレード ハイブリッドモードの場合

データプレーンは、移行プロセス中にトラフィックを処理できます。

1. 3\.0\.xをダウンロードします。
2. 古いコントロールプレーンを廃止します。
3. 新しいコントロールプレーンが、古いコントロールプレーンと同じデータストアを指すように構成します。`kong migrations up`と`kong migrations finish`を実行します。
4. 新しいコントロールプレーンを開始します。古いデータプレーンは、コントロールプレーンへの接続障害についてエラーを発する可能性があります。
5. 新しいデータプレーンを開始します。
6. 徐々にトラフィックを古いデータプレーンから 3\.0\.xデータプレーンに移行します。トラフィックを監視し、移行がスムーズに行われているか 確認します。
7. トラフィックが3\.0\.xクラスタに完全に移行されると、古いノードを廃止します。

新しいデータストアで3\.0\.xをインストールします
-----------------------------

新しいデータストアにインストールする場合、{{site.base_gateway}} 3\.0\.xに
`kong migrations bootstrap`コマンドがあります。以下のコマンドを実行して、
新しいデータストアから新しい3\.0\.xクラスタを準備します。

デフォルトでは、`kong`CLIツール
は`/etc/kong/kong.conf`から構成をロードしますが、任意で、
構成ファイルへのパスを示す`-c`フラグを使用できます。

```bash
kong migrations bootstrap [-c /path/to/kong.conf]
kong start [-c /path/to/kong.conf]
```

{{site.base_gateway}}がすでにシステム上で実行されている場合は、
利用可能な[インストール方法](/gateway/{{page.release}}/install/)で最新のバージョンを
入手してインストールし、以前のインストールを上書きします。

**構成を変更する予定がある場合は、これが絶好のタイミングです** 。

次に、以下のように移行を実行してデータベーススキーマをアップグレードします。

```shell
kong migrations up [-c configuration_file]
```

コマンドが成功し、移行が実行されなかった
（出力なし）の場合は、以下のように
{{site.base_gateway}}の[リロード](/gateway/{{page.release}}/reference/cli/#kong-reload)を行う必要があります。

```shell
kong reload [-c configuration_file]
```

**お知らせ** ：`kong reload`はNginx `reload`シグナルを活用し、前任者が任期を終える前にシームレスに
新しい担当者が仕事を引き継げるようにします。このようにして、{{site.base_gateway}}は
実行中の接続を切断することなく、新たな構成経由で新しいリクエストを供給します。

