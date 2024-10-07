---
title: "Kong Gateway Enterpriseの変更履歴"
no_version: true
---
サポートされているKong Gateway Enterpriseバージョンの変更履歴。

Kong Gateway OSSについては、[GitHubのOSS変更履歴](https://github.com/Kong/kong/tree/master/changelog)を参照してください。

提供終了が終了した製品バージョンについては、[変更履歴のアーカイブ](https://legacy-gateway--kongdocs.netlify.app/enterprise/changelog/)を参照してください。

3\.7\.1\.2
-------------

**リリース日** 2024/07/09

### 非推奨

* Debian 10、CentOS 7、RHEL 7は、2024年6月30日にサポート終了（EOL）となりました。このパッチの時点では、Kongはこれらのオペレーティングシステム用のKong Gateway 3\.7\.xインストールパッケージまたはDockerイメージを構築していません。Kongは、これらのシステムで実行されるKongバージョンに対して公式サポートを提供しなくなりました。

### 機能

#### プラグイン

* [**AWS Lambda**](/hub/kong-inc/aws-lambda)（`aws-lambda`）
  * 新しい構成パラメータ`empty_arrays_mode`が追加されました。これにより、Kong GatewayがLambda関数によって返された空の配列（`[]`）を空の配列（`[]`として送信するか、JSON応答で空のオブジェクト（`{}`）として送信するかを制御できるようになりました。

### 修正済み

* 3\.4\.x以降、公式ドキュメントが削除されたため、Dev Portalドキュメントリンクが利用できなかった問題を修正しました。

### 依存関係

* 起動時のイベント配信におけるレース条件の問題を修正するために、`lua-resty-events`を0\.3\.0に上げました。
* `lua-resty-healthcheck`を3\.1\.0に引き上げ、`lua-resty-events` libのバージョンチェックを削除しました。

3\.7\.1\.1
-------------

**リリース日** 2024/06/22

### 修正

* DNSクライアントがDNS応答で`ADDITIONAL SECTION`のコンテンツを誤って使用していた問題を修正しました。

3\.7\.1\.0
-------------

**リリース日** 2024/06/18

### 既知の問題

* DNSクライアントがDNS応答でコンテンツ`ADDITIONAL SECTION`を誤って使用するという修正に関する問題がありました。 この問題を回避するには、このパッチの代わりに3\.7\.1\.1をインストールしてください。

### 機能

#### プラグイン

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）
  * Content\-Typeパラメータの検証を有効にするかどうかを決定するための新しい構成フィールド`content_type_parameter_validation`が追加されました。

### 修正

#### コア

* **DNSクライアント** : Kong DNSクライアントが回答を解析する際に、ドメインとタイプが一致しないレコードを保存する問題を修正しました。 回答を解析するときに、RRタイプがクエリのタイプと異なる場合はレコードが無視されるようになりました。
* 接続の再試行中に、アップストリームエンティティの`host_header`属性がアップストリームへのリクエストでホストヘッダーとして正しく設定されない問題を修正しました。
* 管理者用の組み込みRBACロール（デフォルトのワークスペースでは`admin`、デフォルト以外のワークスペースでは`workspace-admin`）で、`/groups`および`/groups/*`エンドポイントへのCRUDアクションが禁止されるようになりました。
* `router_flavor`が`expressions`として設定されている場合に、従来モードのルートで優先度フィールドが設定される可能性がある問題を修正しました。

#### プラグイン

* [**AI Proxy**](/hub/kong-inc/ai-proxy/)（`ai-proxy`）

  * オブジェクトコンストラクターがインスタンスではなくクラスにデータを設定する問題を解決しました。

* [**Basic Authentication**](/hub/kong-inc/basic-auth/)（`basic-auth`）

  * Kong Gatewayバージョン3\.6より前では`realm`フィールドが認識されない問題を修正しました。

* [**Key Authentication**](/hub/kong-inc/key-auth/)（`key-auth`）

  * Kong Gatewayバージョン3\.7より前では`realm`フィールドが認識されない問題を修正しました。

* [**AI Rate Limiting Advanced**](/hub/kong-inc/ai-rate-limiting-advanced/)（`ai-rate-limiting-advanced`）

  * スライディングウィンドウを使用する場合のウィンドウ調整のロジックを修正しました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 特定の条件下で、匿名コンシューマが`nil`としてキャッシュされる問題を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * 中央のデータストアでネットワークが不安定になっても、タイマーのスパイクが発生しなくなりました。

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * `param_schema`が`$ref schema`である場合、プラグインがリクエストの処理に失敗することがある問題を修正しました。

### 依存関係

* `lua-resty-events`0\.2\.1にアップグレードしました。
* 多数のタイマーを実行する代わりに、同じアクティブなヘルスチェックターゲットに対してタイマーを再利用することによってメモリリークの問題を修正するために、`lua-resty-healthcheck`を3\.0\.1から3\.0\.2に引き上げました。
* `lua-resty-jsonschema-rs`を0\.1\.5に引き上げました。

3\.7\.0\.0
-------------

**リリース日** 2024/05/28

### 下位互換性のない変更と非推奨

* [**AI Proxy**](/hub/kong-inc/ai-proxy/)（`ai-proxy`）: Anthropicの新しいメッセージAPIをサポートするため、`llm/v1/chat`ルートタイプの`Anthropic`のアップストリームパスが`/v1/complete`から`/v1/messages`に変更されました。
  [\#12699](https://github.com/Kong/kong/issues/12699)

* **HashiCorp Vault** :

  * このバージョンから、HashiCorp VaultエンティティでAppRole認証方法を使用する場合、スペースのみで構成された文字列を`role_id`または`secret_id`値として指定することはできなくなりました。
  * このバージョンから、HashiCorp VaultエンティティでAppRole認証方法を使用する場合、少なくとも`secret_id`または`secret_id_file`のいずれかを指定することが必要になりました。 

* **高精度トレーシング** 機能は非推奨となり、削除されました。
  3\.7へのアップグレードの一環として、`kong.conf`ファイルから次のトレース関連のパラメータを削除します。

  * `tracing`
  * `tracing_write_strategy`
  * `tracing_write_endpoint`
  * `tracing_time_threshold`
  * `tracing_types`
  * `tracing_debug_header`
  * `generate_trace_details`

  代わりに、[OpenTelemetryインストルメンテーション](/gateway/latest/production/tracing/)に移行することをお勧めします。

### 機能

#### Admin API

* 検索フィールドにLHSブラケットのフィルタリングを追加しました。
* **監査ログ:** 
  * `request_timestamp`を`audit_objects`に追加しました。
  * LHS Bracketsフィルターの前後にエイリアスを追加しました。
  * `audit_requests`と`audit_objects`は`request_timestamp`でフィルタリングできるようになりました。
  * `audit_requests`のデフォルトの順序を、`request_timestamp`の降順で並べ替えるように変更しました。

#### 構成

* OpenSSL 3\.xでは、TLSv1\.1およびそれ以下のバージョンはデフォルトで無効になっています。 [\#12420](https://github.com/Kong/kong/issues/12420)
* [`nginx_wasm_main_shm_kv`](/gateway/latest/reference/configuration/#nginx_wasm_main_shm_kv)構成パラメータを導入しました。これにより、Wasmフィルターは、名前空間キーなしにProxy\-Wasm操作の`get_shared_data`と`set_shared_data`を使用できるようになりました。[\#12663](https://github.com/Kong/kong/issues/12663)
* 非推奨フィールドを識別するために、非推奨フィールド属性をスキーマに追加しました。 [\#12686](https://github.com/Kong/kong/issues/12686)
* 個々のフィルターを有効にするための[`wasm_filters`](/gateway/latest/reference/configuration/#wasm_filters)構成パラメータが追加されました。 [\#12843](https://github.com/Kong/kong/issues/12843)

#### コア

* AIの使用状況をカウントするために、匿名レポートに`events:ai:response_tokens`、`events:ai:prompt_tokens`、`events:ai:requests`を追加しました。
  [\#12924](https://github.com/Kong/kong/issues/12924)

* ルーターが`expressions`フレーバーに設定された状態で、コントロールプレーン（CP）が稼働しているときの構成処理が改善されました。[\#12967](https://github.com/Kong/kong/issues/12967)

  * 混合構成が検出され、下位データプレーン（DP）がCPに接続されている場合、構成はまったく送信されません。
  * CPで式が無効な場合、構成は送信されません。
  * 式が下位のDPで無効な場合は、そのDPに送信されます。DP検証は無効な構成をキャッチし、CPに通信します。 その結果、構成が部分的に適用される可能性があります。

* [`router_flavor`](/gateway/latest/reference/configuration/#router_flavor)が`expressions`の場合、ルートエンティティは次のフィールドをサポートするようになりました。`methods`、`hosts`、`paths`、`headers`、`snis`、`sources`、`destinations`、および`regex_priority`。
  これらのフィールドの意味は、[従来のルートエンティティ](/gateway/api/admin-ee/latest/#/Routes)と同じです。
  [\#12667](https://github.com/Kong/kong/issues/12667)

* EmmyLuaDebuggerを使用したデバッグのサポートが追加されました。この機能は技術プレビューであり、Kong Inc.によって正式にサポートされていません。[\#12899](https://github.com/Kong/kong/issues/12899)

* **分析** ：

  * `latencies.receive_ms`フィールドと`websocket`フィールドを追加しました。
  * `latencies.kong_gateway_ms`から受信時間とレイテンシを削除しました。
  * `sse`ブールフィールドをペイロードに追加しました。このフィールドは、サーバー送信イベントのリクエストと応答に関して`true`と設定されます。

#### Kong Manager

* Kong Managerは、Kongの式言語の構文強調表示と自動補完機能を備えたインタラクティブなブラウザ内エディタを使用して、式ルートの作成と編集をサポートするようになりました。 [\#217](https://github.com/Kong/kong-manager/issues/217)
* Kong Managerは、プラグインを構成する際のユーザーエクスペリエンスを向上させるために、パラメータをグループ化するようになりました。
* IDP（OIDCやLDAPなど）を使用してKong Managerを認証する場合、RBACロールのソースがその`role_source`フィールドに保存されるようになりました。これにより、IDPロールマッピングが変更された後の新規ログイン時に、ソースが`idp`である既存のロールを削除できます。また、ユーザーはロールのソースを`local`と`idp`の間で切り替えることもできます。

#### PDK

* ログシリアライザーに`latencies.receive`プロパティを追加しました。 [\#12730](https://github.com/Kong/kong/issues/12730)

#### プラグイン

**新しいプラグイン** ：

* [**AI Rate Limiting Advanced**](/hub/kong-inc/ai-rate-limiting-advanced/)（`ai-rate-limiting-advanced`）: このプラグインを使用すると、AIプロバイダーによるレート制限を実装できます。
* [**AI Azure Content Safety**](/hub/kong-inc/ai-azure-content-safety/)（`ai-azure-content-safety`）: Azure Content Safetyサービスを使用して、すべてのAI Proxyリクエストのイントロスペクションを適用できます。 このプラグインを使用すると、さまざまなモデレーションカテゴリのしきい値を設定できるようになり、レポート用に監査結果をKongログシリアライザーに報告します。

**既存のプラグインの更新:** 

* [**AI Proxy**](/hub/kong-inc/ai-proxy/)（`ai-proxy`）

  * AI Proxyは、クライアントからほとんどのプロンプトチューニングパラメータを読み取るようになり、`model_options`下のプラグイン構成パラメータはデフォルトになりました。 これにより、それぞれのプロバイダーのネイティブSDKを使用するためのサポートが修正されました。 [\#12903](https://github.com/Kong/kong/issues/12903)
  * AI Proxyに[`route_type`の`preserve`オプション](/hub/kong-inc/ai-proxy/configuration/#config-route_type)が追加され、リクエストと応答はアップストリームLLMに直接渡されるようになりました。これにより、AIサービスを呼び出すときに使用されるモデルやSDKsとの互換性が可能になります。[\#12903](https://github.com/Kong/kong/issues/12903)
  * サポートされているプロバイダーで、クライアントへの[イベントごとの返答ストリーミング](/hub/kong-inc/ai-proxy/how-to/streaming/)へのサポートを追加しました。 [\#12792](https://github.com/Kong/kong/issues/12792)
  * **Enterprise限定機能** : AI Proxyを備えたAzureプロバイダーを使用する場合の[マネージドID認証](/hub/kong-inc/ai-proxy/how-to/cloud-provider-authentication/)サポートを追加しました。

* [**Prometheus**](/hub/kong-inc/prometheus/)（`prometheus`）

  * Prometheusプラグインメトリクスにワークスペースラベルを追加しました。 [\#12836](https://github.com/Kong/kong/issues/12836)

* [**AI Prompt Guard**](/hub/kong-inc/ai-prompt-guard/)（`ai-prompt-guard`）

  * `allow`および`deny`パラメータの正規表現の最大長を500に増やしました。 [\#12731](https://github.com/Kong/kong/issues/12731)

* [**JWT**](/hub/kong-inc/jwt/)（`jwt`）

  * EdDSAアルゴリズムのサポートが追加されました。[\#12726](https://github.com/Kong/kong/issues/12726)
  * ES512、PS256、PS384、PS512アルゴリズムのサポートが追加されました。 [\#12638](https://github.com/Kong/kong/issues/12638)

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）と[**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）

  * 伝達モジュールが再調整されました。新しいオプションを使用すると、トレーシングヘッダーの伝達の構成をより適切に制御できます。 [\#12670](https://github.com/Kong/kong/issues/12670)

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * [DPoP（所持証明）トークン検証](/hub/kong-inc/openid-connect/how-to/demonstrating-proof-of-possession/)のサポートが追加されました。 この機能は構成パラメータ[`proof_of_possession_dpop`](/hub/kong-inc/openid-connect/configuration/#config-proof_of_possession_dpop)を使用して有効にできます。
  * 認可およびプッシュ認可（PAR）エンドポイントでのJWTセキュア認可リクエスト（JAR）のサポートが追加されました。 構成パラメータ[`require_signed_request_object`](/hub/kong-inc/openid-connect/configuration/#config-require_signed_request_object)を参照してください。
  * JARM応答モードのサポートが追加されました: `query.jwt`、`form_post.jwt`、`fragment.jwt`、および`jwt`。

* [**GraphQL Proxy Cache Advanced**](/hub/kong-inc/graphql-proxy-cache-advanced/)

  * Redisストラテジのサポートが追加されました。
  * リクエストがアップストリームに送信された状態で、未処理のエラーをバイパスで解決する機能が追加されました。[`bypass_on_err`](/hub/kong-inc/graphql-proxy-cache-advanced/unreleased/configuration/#config-bypass_on_err)構成オプションを使用して有効にします。

* [**JWT Signer**](/hub/kong-inc/jwt-signer/)（`jwt-signer`）

  * 外部JWKSサービスへのベーシック認証とmTLS認証のサポートが追加されました。
  * プラグインは、JWKSの定期的なローテーションをサポートするようになりました。たとえば、`access_token_jwks_uri`を自動的にローテーションするには、構成オプション[`access_token_jwks_uri_rotate_period`](/hub/kong-inc/jwt-signer/configuration/#config-access_token_jwks_uri_rotate_period)を設定できます。
  * プラグインは、アップストリームリクエストヘッダーの名前を[`original_access_token_upstream_header`](/hub/kong-inc/jwt-signer/configuration/#config-original_access_token_upstream_header)と[`original_channel_token_upstream_header`](/hub/kong-inc/jwt-signer/configuration/#config-original_channel_token_upstream_header)で指定することで、元のJWTをアップストリームリクエストヘッダーに追加できるようになりました。
  * [`access_token_upstream_header`](/hub/kong-inc/jwt-signer/configuration/#config-access_token_upstream_header)、[`channel_token_upstream_header`](/hub/kong-inc/jwt-signer/configuration/#config-channel_token_upstream_header)、[`original_access_token_upstream_header`](/hub/kong-inc/jwt-signer/configuration/#config-original_access_token_upstream_header)、および[`original_channel_token_upstream_header`](/hub/kong-inc/jwt-signer/configuration/#config-original_channel_token_upstream_header)は同じ値にできません。
  * このプラグインは、[`add_claims`](/hub/kong-inc/jwt-signer/configuration/#config-add_claims)と[`set_claims`](/hub/kong-inc/jwt-signer/configuration/#config-set_claims)の疑似JSON値をサポートするようになりました。JSON文字列を値として渡すことで、キーに複数の値を渡すという目標を達成できます。
  * アクセストークンとチャンネルトークンにクレームを個別に追加するための[`add_access_token_claims`](/hub/kong-inc/jwt-signer/configuration/#config-add_access_token_claims)、[`set_access_token_claims`](/hub/kong-inc/jwt-signer/configuration/#config-set_access_token_claims)、[`add_channel_token_claims`](/hub/kong-inc/jwt-signer/configuration/#config-add_channel_token_claims)、[`set_channel_token_claims`](/hub/kong-inc/jwt-signer/configuration/#config-set_channel_token_claims)が追加されました。
  * クレームの削除をサポートするために[`remove_access_token_claims`](/hub/kong-inc/jwt-signer/configuration/#config-remove_access_token_claims)と[`remove_channel_token_claims`](/hub/kong-inc/jwt-signer/configuration/#config-remove_channel_token_claims)が追加されました。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * カスタムベースパスを指定するための[`custom_base_path`](/hub/kong-inc/mocking/configuration/#config-custom_base_path)フィールドが追加されました。 [`deck file namespace`](/deck/latest/reference/deck_file_namespace/)コマンドと一緒に使用します。

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * [`default_consumer`](/hub/kong-inc/mtls-auth/configuration/#config-default_consumer)オプションを追加しました。これにより、クライアント証明書が有効だが既存のコンシューマに一致しない場合に、デフォルトのコンシューマを使用できるようになります。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）

  * `api_spec`がURIエンコードされているかどうかを示す新しいフィールド[`api_spec_encoded`](/hub/kong-inc/oas-validation/configuration/#config-api_spec_encoded)が追加されました。
  * カスタムベースパスを指定するための[`custom_base_path`](/hub/kong-inc/oas-validation/configuration/#config-custom_base_path)フィールドが追加されました。 [`deck file namespace`](/deck/latest/reference/deck_file_namespace/)コマンドと一緒に使用します。
  * プラグインは、OpenAPI仕様v3\.1\.0をサポートするようになりました。仕様バージョンがv3\.1\.0になると、プラグインは新しいJSONSchema Validatorに切り替わるようになりました。

### パフォーマンス

* 内部フックメカニズムをリファクタリングすることで、プロキシパフォーマンスが向上しました。 [\#12784](https://github.com/Kong/kong/issues/12784)
* `router_flavor`が`traditional_compatible`または`expressions`の場合のルーターのマッチングを高速化しました。 [\#12467](https://github.com/Kong/kong/issues/12467)
* **OpenTelemetry** : キューの最大バッチサイズを200に増やしました。 [\#12488](https://github.com/Kong/kong/issues/12488)
* トレーシングメカニズムを高速化しました。

### 修正

#### Admin API

* エンドポイント`POST /schemas/vaults/validate`を呼び出すと、GETしか実装されていないエンドポイント`/schemas/vaults/:name`と競合して405になる問題を修正しました。 [\#12607](https://github.com/Kong/kong/issues/12607)
* 重複したアップストリームターゲットが入力に含まれている場合に、`POST /config?flatten_errors=1`が適切な応答を返せない問題を修正しました。 [\#12797](https://github.com/Kong/kong/issues/12797)
* `/<workspace id="sl-md0000000">/admins`エンドポイントが誤って使用され、割り当てられたRBACのロールに基づいてワークスペースベースに関連付けられている管理者を返していました。所属するワークスペースに応じて管理者を返すように修正されました。

#### CLI

* `--db-timeout`がCLI引数で明示的に渡される場合でも、`pg_timeout`が`60s`に上書きされる問題を修正しました。 
* `kong`コマンドラインツールが`lua_ssl_trusted_certificate`構成オプションを無視する問題を修正しました。

#### クラスタリング

* AWS Secrets Managerに関連するクラスタ互換性チェックで、`AK-SK`環境変数を使用してIAMロールの権限がグラントされるように調整しました。 
* HCV Kubernetes認証パスに関連するクラスタ互換性チェックを調整しました。
* HashiCorp Vault Approle認証に関連するクラスタ互換性チェックを調整しました。
* ハイブリッドモードで、イベントフックが早期に検証される問題を修正しました。 この修正により、イベントフックの検証は、イベントフックが発行される時点まで遅延されます。

#### 構成

* `kong.conf.default`での`upstream_keepalive_max_requests`のデフォルト値を1,000から10,000に修正しました。 [\#12643](https://github.com/Kong/kong/issues/12643)
* 外部プラグイン（Go、Javascript、Python）が、Admin APIを介してプラグイン構成に変更を適用する際に失敗する問題を修正しました。[\#12718](https://github.com/Kong/kong/issues/12718)
* デフォルトで`proxy-wasm`からのLua DNSリゾルバの使用を無効にしました。 [\#12825](https://github.com/Kong/kong/issues/12825)
* `ssl_cipher_suite`が`old`に設定されている場合、gRPCのTLSのセキュリティレベルを`0`に設定します。 [\#12613](https://github.com/Kong/kong/issues/12613)

#### コア

* **DNSクライアント** : Kongは、オプションのタイムアウトの`resolv.conf`の非正の値を無視し、代わりにデフォルト値の2秒を使用するようになりました。[\#12640](https://github.com/Kong/kong/issues/12640)
* `kong.logrotate`のファイル権限を644に更新しました。 [\#12629](https://github.com/Kong/kong/issues/12629)
* ハイブリッドモードのデータプレーンで、Vaultリファレンスで構成されている証明書エンティティが時間通りに更新されないことがあった問題を修正しました。 [\#12868](https://github.com/Kong/kong/issues/12868)
* リクエストデバッグの出力で欠落していたルーターセクションを修正しました。 [\#12234](https://github.com/Kong/kong/issues/12234)
* 内部キャッシュロジックでmutexがロック解除されない可能性がある問題を修正しました。 [\#12743](https://github.com/Kong/kong/issues/12743)
* ルートの構成が変更された場合に、ルーターが正しく動作しない問題を修正しました。[\#12654](https://github.com/Kong/kong/issues/12654)
* `tls_passthrough`を`traditional_compatible`ルーターフレーバーと一緒に使用すると、SNIベースのルーティングが機能しない問題を修正しました。[\#12681](https://github.com/Kong/kong/issues/12681)
* `X-Kong-Upstream-Status`が`kong.conf`ファイルの`headers`パラメータで設定されていた場合でも、応答がヒットしてProxy Cacheプラグインによって返された場合に応答ヘッダーに表示されない問題を修正しました。[\#12744](https://github.com/Kong/kong/issues/12744)
* **Vault** :
  * `init_worker`フェーズでVaultリファレンスの解決をタイマーに延期することで、Vaultの初期化を修正しました。 [\#12554](https://github.com/Kong/kong/issues/12554)
  * TTLが設定されていない場合でも、Vaultシークレットが更新される問題を修正しました。 [\#12877](https://github.com/Kong/kong/issues/12877)
  * 接頭辞でVaultエンティティを取得するときに、Vaultが誤った（デフォルトの）ワークスペース識別子を使用する問題を修正しました。 [\#12572](https://github.com/Kong/kong/issues/12572)

* バランサーの`stop_healthchecks`関数で予期しないテーブルnilパニックが発生する問題を修正しました。 [\#12865](https://github.com/Kong/kong/issues/12865)
* Kong Gatewayは、アクセスの問題を回避するために、特権エージェントのワーカーIDとして`-1`を使用するようになりました。 [\#12385](https://github.com/Kong/kong/issues/12385)
* Kong GatewayがMessagePackベースのプラグインサーバー（PythonやJavascriptプラグインなどで使用される）を適切に再起動できない問題を修正しました。 [\#12582](https://github.com/Kong/kong/issues/12582)
* ダウンストリーム接続がHTTP/2またはHTTP/3ストリームモードの場合、OpenRestyアップストリームの新しいバージョンで`ngx.read_body()` APIがハードコードされる制限を元に戻しました。 [\#12658](https://github.com/Kong/kong/issues/12658)
* Kongの各キャッシュインスタンスは、独自のクラスタイベントチャネルを使用するようになりました。 このアプローチは、キャッシュ無効化イベントを分離し、不要なワーカーイベントの生成を削減します。 [\#12321](https://github.com/Kong/kong/issues/12321)
* 同じリクエストに対して複数のプラグインのデータを設定できるように、AIプラグインのテレメトリコレクションを更新しました。 [\#12583](https://github.com/Kong/kong/issues/12583)
* 低いulimit設定（ファイルを開く）により、`lua-resty-timer-ng`が利用可能な`worker_connections`を使い果たしてしまいKongが起動に失敗する問題を修正しました。このバグを修正するために、`lua-resty-timer-ng`ライブラリの同時実行範囲を`[512, 2048]`から`[256, 1024]`に減らしました。 [\#12606](https://github.com/Kong/kong/issues/12606)
* Protobufベースのプロトコルを使用する外部プラグインで`kong.Service.SetUpstream`メソッドを呼び出すと、`bad argument #2 to 'encode' (table expected, got boolean)`エラーが発生して失敗していた問題を修正しました。 [\#12727](https://github.com/Kong/kong/issues/12727)
* 不要なエラーログを回避するために、ストリームモジュールの分析を無効にしました。
* 最初の構成プッシュ後に、新しいデータプレーンがVaultリファレンスを解決できなかった問題を修正しました。これは、ライセンスのプリロードの問題が原因で発生していました。
* 既存のlmdbを読み込むときに、DPが、ライセンスが必要なVaultリファレンスを解決できない問題を修正しました。
* `admin_gui_auth_conf.scope`に`"openid"`が欠落している場合、または`admin_gui_auth`が`openid-connect`に設定されているときに`"offline_access"`が欠落している場合に、ユーザーがKong Gatewayを起動できない問題を修正しました。Kong Gatewayは、`admin_gui_auth_conf.scope`に`"openid"`が欠落している場合にのみ警告ログを出力するようになりました。

#### Kong Manager

* さまざまなUI関連の問題を修正し、Kong Managerのユーザーエクスペリエンスを改善しました。

  [\#185](https://github.com/Kong/kong-manager/issues/185) [\#188](https://github.com/Kong/kong-manager/issues/188) [\#190](https://github.com/Kong/kong-manager/issues/190) [\#195](https://github.com/Kong/kong-manager/issues/195) [\#199](https://github.com/Kong/kong-manager/issues/199) [\#201](https://github.com/Kong/kong-manager/issues/201) [\#202](https://github.com/Kong/kong-manager/issues/202) [\#207](https://github.com/Kong/kong-manager/issues/207) [\#208](https://github.com/Kong/kong-manager/issues/208) [\#209](https://github.com/Kong/kong-manager/issues/209) [\#213](https://github.com/Kong/kong-manager/issues/213) [\#216](https://github.com/Kong/kong-manager/issues/216)
* IDPでの認証時に、 **ロールを追加** ボタンが表示される問題を修正しました。現在、Kong ManagerがIDPでの認証に設定されている場合、このボタンは表示されなくなっています。

* RBACユーザーフォームページに表示されるドキュメントリンクを修正しました。

#### PDK

* `kong.request.get_forwarded_port`が`ngx.ctx.host_port`から誤った文字列を返す問題を修正しました。数値が正しく返されるようになりました。[\#12806](https://github.com/Kong/kong/issues/12806)

* ログシリアライザーのペイロードの`latencies.kong`の値に応答の受信時間が含まれなくなり、`X-Kong-Proxy-Latency`応答ヘッダーと同じ値になりました。応答の受信時間は新しい`latencies.receive`メトリクスに記録されるので、必要であれば、古い値は`latencies.kong + latencies.receive`として計算できます。[\#12795](https://github.com/Kong/kong/issues/12795)

  {:.note}
  > 
  > **注:** これは、ログシリアライザーを使用するすべてのロギングプラグインのペイロードにも影響します。
  > `file-log`、`tcp-log`、`udp-log`、`http-log`、`syslog`、および`loggly`です。
* **トレーシング** : トレースID解析のロバスト性を強化しました。
  [\#12848](https://github.com/Kong/kong/issues/12848)

#### プラグイン

* AIプラグインのエラー処理をクリーンアップして改善しました。

* [**AI Proxy**](/hub/kong-inc/ai-proxy/)（`ai-proxy`）

  * `/llm/v1/chat`ルートタイプで応答に分析が含まれていない問題を修正しました。 [\#12781](https://github.com/Kong/kong/issues/12781)

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * ACME更新中に証明書が正常に更新されない問題を修正しました。 [\#12773](https://github.com/Kong/kong/issues/12773)
  * Redis構成の移行を修正しました。
  * 秘密鍵に関して間違ったエラーログが出力される問題を修正しました。

* [**AWS Lambda**](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * AWS Lambda APIリクエストに起因するレイテンシが、Kong Gatewayのレイテンシの一部としてカウントされる問題を修正しました。 [\#12835](https://github.com/Kong/kong/issues/12835)

* [**JWT**](/hub/kong-inc/jwt/)（`jwt`）

  * ES384とES512アルゴリズムに無効な公開鍵を使用すると、プラグインが失敗する問題を修正しました。 [\#12724](https://github.com/Kong/kong/issues/12724)

* [**Key Authentication**](/hub/kong-inc/key-auth/)（`key-auth`）

  * すべての401応答に欠落していた`WWW-Authenticate`ヘッダーを追加しました。 [\#11794](https://github.com/Kong/kong/issues/11794)

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * `http_response_header_for_traceid`オプションが有効になっている場合に発生していた、OTELサンプリングモードのLuaパニックバグを修正しました。 [\#12544](https://github.com/Kong/kong/issues/12544)

* [**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）

  * Redis構成の移行を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `kong/tools/public/rate-limiting`をリファクタリングし、異なるプラグインを分離するための新しいインターフェース`new_instance`を追加しました。
    後方互換性のため、元のインターフェースは変更されていません。

    このライブラリをベースにしたカスタムのRate Limitingプラグインを使用している場合は、初期化コードを新しい形式に更新してください。たとえば、`'local ratelimiting = require("kong.tools.public.rate-limiting").new_instance("custom-plugin-name")'`などです。
    古いインターフェースは、今後のメジャーリリースで削除されます。
  * `rate-limiting`ライブラリを使用するプラグインを一緒に使用すると、互いに干渉し合い、カウンターデータを中央のデータストアに同期できなくなる問題を修正しました。

  * `redis`ストラテジで使用される`sync_rate`設定に関する問題を修正しました。
    `sync_rate = 0`中にRedis接続が中断された場合、プラグインは正確に`local`ストラテジにフォールバックするようになりました。

  * `sync_rate`が`0`より大きい値から`0`に変更された場合、名前空間が予期せずクリアされる問題を修正しました。

  * カウンター同期タイマーが適切に作成または破棄されないというタイマー関連の問題を修正しました。

  * プラグイン作成時ではなく、プラグイン実行中にカウンター同期タイマーを作成するようになったことで、無意味なエラーログを減らせるようになりました。

  * 複数のRate Limiting Advancedプラグインが同じ名前空間を共有しているときに、{{site.base_gateway}}がエラーログエントリのログを生成する問題を修正しました。

* [**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）

  * Redis構成の移行を修正しました。

* [**DeGraphQL**](/hub/kong-inc/degraphql/)（`degraphql`）

  * GraphQL変数が正しく解析されず、定義された型に強制変換されない問題を修正しました。
  * このプラグインは、新しい設定ハンドラを使用することで、より優れたエラー処理でGraphQLルーターを更新するようになりました。

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * 認証情報がユーザー名なしでエンコードされている場合に、Kong Gatewayがエラーをスローし、500コードを返す問題を修正しました。
  * LDAP検索が失敗した場合に例外がスローされる問題を修正しました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）、
  [**WebSocket Size Limit**](/hub/kong-inc/websocket-size-limit/)（`websocket-size-limit`）、
  [**WebSocket Validator**](/hub/kong-inc/websocket-validator/)（`websocket-validator`）、
  [**XML Threat Protection**](/hub/kong-inc/xml-threat-protection/)（`xml-threat-protection`）

  * プラグイン間の衝突を防ぐために、これらのプラグインの[優先順位](/gateway/latest/plugin-development/custom-logic/#plugins-execution-order)が更新されました。 バンドルされたプラグインの相対的な優先順位（および実行順序）は変更されません。

### 依存関係

* `atc-router`をv1\.6\.0からv1\.6\.2に更新しました [\#12231](https://github.com/Kong/kong/issues/12231)
* `libexpat`を2\.6\.2に更新しました [\#12910](https://github.com/Kong/kong/issues/12910)
* `lua-kong-nginx-module`を0\.8\.0から0\.11\.0にアップグレードしました [＃12752](https://github.com/Kong/kong/issues/12752)
* `lua-protobuf`を 0\.5\.1 にアップグレードしました [＃12834](https://github.com/Kong/kong/issues/12834)
* `lua-resty-acme`を0\.13\.0にアップグレードしました [＃12909](https://github.com/Kong/kong/issues/12909)
* `lua-resty-aws`を1\.3\.6から1\.4\.1に更新しました [\#12846](https://github.com/Kong/kong/issues/12846)
* `lua-resty-lmdb`を1\.4\.1から1\.4\.2に更新しました [\#12786](https://github.com/Kong/kong/issues/12786)
* `lua-resty-openssl`を1\.2\.0から1\.3\.1に更新しました [\#12665](https://github.com/Kong/kong/issues/12665)
* `lua-resty-timer-ng`を0\.2\.7に更新しました [\#12756](https://github.com/Kong/kong/issues/12756)
* PCREをレガシーの`libpcre` 8\.45から`libpcre2` 10\.43にアップグレードしました [＃12366](https://github.com/Kong/kong/issues/12366)
* `penlight`を1\.14\.0に更新しました [\#12862](https://github.com/Kong/kong/issues/12862)
* 便利なタイムゾーン設定のために、パッケージ`tzdata`をDEB Dockerイメージに追加しました [\#12609](https://github.com/Kong/kong/issues/12609)
* `lua-resty-http`を0\.17\.2に更新しました。[\#12908](https://github.com/Kong/kong/issues/12908)
* LuaRocksを3\.9\.2から3\.11\.0に更新しました[\#12662](https://github.com/Kong/kong/issues/12662)
* `ngx_wasm_module`を`91d447ffd0e9bb08f11cc69d1aa9128ec36b4526`に更新しました [\#12011](https://github.com/Kong/kong/issues/12011)
* `V8`を12\.0\.267\.17に更新しました [\#12704](https://github.com/Kong/kong/issues/12704)
* `Wasmtime`を19\.0\.0に更新しました [\#12011](https://github.com/Kong/kong/issues/12011)
* 予期しない入力を処理する際の`lua-cjson`のロバスト性を改善しました [\#12904](https://github.com/Kong/kong/issues/12904)
* サブモジュールの`kong-openid-connect`を2\.7\.1に更新しました
* `kong-lua-resty-kafka`を0\.18にアップグレードしました
* `lua-resty-luasocket`を1\.1\.2に更新し、[luasocket\#427](https://github.com/lunarmodules/luasocket/issues/427)を修正しました
* `lua-resty-mail`を1\.1\.0に更新しました
* [CVE\-2023](https://konghq.atlassian.net/browse/CVE-2023)および[CVE\-2022](https://konghq.atlassian.net/browse/CVE-2022)に対処するためにOpenSSL FIPSプロバイダーを3\.0\.9に更新しました
* `libpasswdqc`を2\.0\.3に更新しました
* `lua-resty-cookie`を0\.2\.0に更新しました
* `lua-resty-passwdqc`を2\.0に更新しました
* `xmlua`を1\.2\.1に更新しました
* `libxml2`を2\.12\.63にアップグレードしました
* `libxslt`を1\.1\.39に更新しました
* `msgpack-c`を6\.0\.1に更新しました
* `lua-resty-openssl-aux-module`の依存関係を削除しました

3\.6\.1\.7
-------------

**リリース日** 2024/07/09

### 機能

### 非推奨

* Debian 10、CentOS 7、RHEL 7は、2024年6月30日にサポート終了（EOL）となりました。このパッチの時点では、Kongはこれらのオペレーティングシステム用のKong Gateway 3\.6\.xインストールパッケージまたはDockerイメージを構築していません。Kongは、これらのシステムで実行されるKongバージョンに対して公式サポートを提供しなくなりました。

#### プラグイン

*3\.7\.1\.2からバックポート* 

* [**AWS Lambda**](/hub/kong-inc/aws-lambda)（`aws-lambda`）
  * 新しい構成パラメータ`empty_arrays_mode`が追加されました。これにより、Kong GatewayがLambda関数によって返された空の配列（`[]`）を空の配列（`[]`として送信するか、JSON応答で空のオブジェクト（`{}`）として送信するかを制御できるようになりました。

### 依存関係

* 起動時のイベント配信におけるレース条件の問題を修正するために、`lua-resty-events`を0\.3\.0に上げました。
* `lua-resty-healthcheck`を3\.1\.0に引き上げ、`lua-resty-events` libのバージョンチェックを削除しました。

3\.6\.1\.6
-------------

**リリース日** 2024/06/22

### 修正

* DNSクライアントがDNS応答で`ADDITIONAL SECTION`のコンテンツを誤って使用していた問題を修正しました。

3\.6\.1\.5
-------------

**リリース日** 2024/06/18

### 既知の問題

* DNSクライアントがDNS応答でコンテンツ`ADDITIONAL SECTION`を誤って使用するという修正に関する問題がありました。 この問題を回避するには、このパッチの代わりに3\.6\.1\.6をインストールしてください。

### 機能

#### Admin API

*3\.7\.0\.0からバックポート* 

* 検索フィールドにLHSブラケットのフィルタリングを追加しました。
* **監査ログ:** 
  * `request_timestamp`を`audit_objects`に追加しました。
  * LHS Bracketsフィルターの前後にエイリアスを追加しました。
  * `audit_requests`と`audit_objects`は`request_timestamp`でフィルタリングできるようになりました。
  * `audit_requests`のデフォルトの順序を、`request_timestamp`の降順で並べ替えるように変更しました。

#### プラグイン

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）
  * Content\-Typeパラメータの検証を有効にするかどうかを決定するための新しい構成フィールド`content_type_parameter_validation`が追加されました。

### 修正

#### Admin API

*3\.7\.0\.0からバックポート* 

* `/<workspace id="sl-md0000000">/admins`エンドポイントが誤って使用され、割り当てられたRBACのロールに基づいてワークスペースに関連付けられている管理者を返していました。この問題は修正され、特定のワークスペースの関連付けに応じて管理者が正確に返されるようになりました。

#### CLI

*3\.7\.0\.0からバックポート* 

* `--db-timeout`がCLI引数で明示的に渡される場合でも、`pg_timeout`が`60s`に上書きされる問題を修正しました。 

#### コア

* 管理者用の組み込みRBACロール（デフォルトのワークスペースでは`admin`、デフォルト以外のワークスペースでは`workspace-admin`）で、`/groups`および`/groups/*`エンドポイントへのCRUDアクションが禁止されるようになりました。

*3\.7\.1\.0からバックポート* 

* **DNSクライアント** : Kong DNSクライアントが回答を解析する際に、ドメインとタイプが一致しないレコードを保存する問題を修正しました。 回答を解析するときに、RRタイプがクエリのタイプと異なる場合はレコードが無視されるようになりました。
* 接続の再試行中に、アップストリームエンティティの`host_header`属性がアップストリームへのリクエストでホストヘッダーとして正しく設定されない問題を修正しました。

#### プラグイン

* [**Basic Authentication**](/hub/kong-inc/basic-auth/)（`basic-auth`）

  * Kong Gatewayバージョン3\.6より前では`realm`フィールドが認識されない問題を修正しました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 特定の条件下で、匿名コンシューマが`nil`としてキャッシュされる問題を修正しました。

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * `param_schema`が`$ref schema`である場合、プラグインがリクエストの処理に失敗することがある問題を修正しました。

*3\.7\.1\.0からバックポート* 

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * 中央のデータストアでネットワークが不安定になっても、タイマーのスパイクが発生しなくなりました。

*3\.7\.0\.0からバックポート* 

* [**ACME**](/hub/kong-inc/acme/)（`acme`）、[**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）、および[**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`） 
  * Redis構成の移行を修正しました。

### 依存関係

* エラーログを改良するために、`lua-resty-azure`を1\.4\.1から1\.5\.0に更新しました。
* `lua-resty-events`0\.2\.1にアップグレードしました。
* 多数のタイマーを実行する代わりに、同じアクティブなヘルスチェックターゲットに対してタイマーを再利用することによってメモリリークの問題を修正するために、`lua-resty-healthcheck`を3\.0\.1から3\.0\.2に引き上げました。
* 予期しない入力を処理する際の`lua-cjson`のロバスト性を改善しました。

3\.6\.1\.4
-------------

**リリース日** 2024/05/14

### 機能

#### プラグイン

* [**Portal Application Registration**](/hub/kong-inc/application-registration/)（`application-registration`） 
  * 認証情報によるコンシューマ認証を使用したサービスにアクセスできるようになりました。 この機能を使用するには、`enable_proxy_with_consumer_credential`を有効にします（デフォルトは`false`）。

*3\.7\.0\.0からバックポート* 

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）
  * `default_consumer`オプションを追加しました。これにより、クライアント証明書が有効だが既存のコンシューマに一致しない場合に、デフォルトのコンシューマを使用できるようになります。

### 修正

#### クラスタリング

* ハイブリッドモードで、イベントフックが早期に検証される問題を修正しました。 この修正により、イベントフックの検証は、イベントフックが発行される時点まで遅延されます。

#### コア

* ハイブリッドモードのデータプレーンで、Vaultリファレンスで構成されている証明書エンティティが時間通りに更新されないことがあった問題を修正しました。

* `init_worker`フェーズでVaultリファレンスの解決をタイマーに延期することで、Vaultの初期化を修正しました。

#### PDK

* `kong.request.get_forwarded_port`が`ngx.ctx.host_port`から誤った文字列を返す問題を修正しました。数値が正しく返されるようになりました。

#### プラグイン

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）、
  [**WebSocket Size Limit**](/hub/kong-inc/websocket-size-limit/)（`websocket-size-limit`）、
  [**WebSocket Validator**](/hub/kong-inc/websocket-validator/)（`websocket-validator`）、
  [**XML Threat Protection**](/hub/kong-inc/xml-threat-protection/)（`xml-threat-protection`）

  * プラグイン間の衝突を防ぐために、これらのプラグインの優先順位が更新されました。 バンドルされたプラグインの相対的な優先順位（および実行順序）は変更されません。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `kong/tools/public/rate-limiting`をリファクタリングし、異なるプラグインを分離するための新しいインターフェース`new_instance`を追加しました。
    後方互換性のため、元のインターフェースは変更されていません。

    このライブラリをベースにしたカスタムのRate Limitingプラグインを使用している場合は、初期化コードを新しい形式に更新してください。たとえば、`'local ratelimiting = require("kong.tools.public.rate-limiting").new_instance("custom-plugin-name")'`などです。
    古いインターフェースは、今後のメジャーリリースで削除されます。

### 依存関係

* `lua-protobuf`を0\.5\.1にアップグレードしました。

3\.6\.1\.3
-------------

**リリース日** 2024/04/16

### 修正

#### Kong Manager

* `admin_gui_path`がスラッシュでない場合に、管理者アカウントプロファイルページで404エラーが返される問題を修正しました。

#### プラグイン

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）
  * 短いトレースIDの解析のロバスト性が向上しました。

3\.6\.1\.2
-------------

**リリース日** 2024/04/08

### 機能

#### プラグイン

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * `api_spec`がURIエンコードされているかどうかを示す新しいフィールド`api_spec_encoded`が追加されました。

### 修正

#### クラスタリング

* AWS Secrets Managerに関連するクラスタリング互換性チェックを調整し、`AK-SK`環境変数を使用してIAMロールの権限を付与するようにしました。

#### 構成

* 外部プラグイン（Go、Javascript、Python）が、Admin APIを介してプラグイン構成に変更を適用する際に失敗する問題を修正しました。

#### コア

* `kong.logrotate`のファイル権限を644に更新しました。
* Vault: 
  * プレフィックスでVaultエンティティを取得するときに、Vaultが誤った（デフォルトの）ワークスペース識別子を使用する問題を修正しました。
  * 最初の構成プッシュ後に、新しいデータプレーンがVaultリファレンスを解決できなかった問題を修正しました。これは、ライセンスのプリロードの問題が原因で発生していました。

* `admin_gui_auth_conf.scope`に`"openid"`が欠落している場合、または`admin_gui_auth`が`openid-connect`に設定されているときに`"offline_access"`がある場合に、ユーザーがKong Gatewayを起動できない問題を修正しました。Kong Gatewayは、`admin_gui_auth_conf.scope`に`"openid"`が欠落している場合にのみ警告ログを出力するようになりました。

#### Kong Manager Enterprise

* ライセンスの有効期限の残り日数の表示を修正しました。
* RBACユーザーのRBACトークンのタイプを`password`に更新しました。

#### プラグイン

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * ACME更新中に証明書が正常に更新されない問題を修正しました。 

* [**DeGraphQL**](/hub/kong-inc/degraphql/)（`degraphql`）

  * GraphQL変数が正しく解析されず、定義された型に強制変換されない問題を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `rate-limiting`ライブラリを使用するプラグインを一緒に使用すると、互いに干渉し合い、カウンターデータを中央のデータストアに同期できなくなる問題を修正しました。

#### 依存関係

* `lua-resty-openssl`を1\.2\.1に更新しました
* PCREをレガシーの`libpcre` 8\.45から`libpcre2` 10\.43に更新しました
* `lua-kong-nginx-module`を0\.8\.1に更新しました
* `kong-lua-resty-kafka`を0\.18にアップグレードしました
* `lua-resty-luasocket`を1\.1\.2に更新し、[luasocket\#427](https://github.com/lunarmodules/luasocket/issues/427)を修正しました

3\.6\.1\.1
-------------

**リリース日** 2024/05/03

### 修正

#### クラスタリング

* HashiCorp Vault Approle認証に関連するクラスタ互換性チェックを調整しました。

#### コア

* リクエストデバッグの出力で欠落していたルーターセクションを修正しました。 
* ダウンストリーム接続がHTTP/2またはHTTP/3ストリームモードの場合、OpenRestyアップストリームの新しいバージョンで`ngx.read_body()` APIがハードコードされる制限を元に戻しました。 

#### Kong ManagerとKonnect

* プラグイン選択ページにカスタムプラグインが表示されない問題を修正しました。
* 式ルーターを使用しているときに、ルートフォームにサービスが事前に入力されなかった問題を修正しました。

#### プラグイン

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * `redis`ストラテジで使用される`sync_rate`設定に関する問題を修正しました。 `sync_rate = 0`中にRedis接続が中断された場合、プラグインは正確に`local`ストラテジにフォールバックするようになりました。
  * `sync_rate`が`0`より大きい値から`0`に変更された場合、名前空間が予期せずクリアされる問題を修正しました。
  * カウンター同期タイマーが適切に作成または破棄されないというタイマー関連の問題を修正しました。
  * プラグイン作成時ではなく、プラグイン実行中にカウンター同期タイマーを作成するようになったことで、無意味なエラーログを減らせるようになりました。

3\.6\.1\.0
-------------

**リリース日** 2024/02/26

### 機能

#### 構成

* OpenSSL 3\.xでは、TLSv1\.1およびそれ以下はデフォルトで無効になっています。 

#### プラグイン

*3\.7\.0\.0からバックポート* 

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）
  * キューの最大バッチサイズを200に増やしました。

### 修正

#### 一般

* 低いulimit設定（ファイルを開く）により、`lua-resty-timer-ng`が利用可能な`worker_connections`を使い果たしてしまいKongが起動に失敗する問題を修正しました。このバグを修正するために、`lua-resty-timer-ng`ライブラリの同時実行範囲を`[512, 2048]`から`[256, 1024]`に減らしました。 

#### 構成

*3\.7\.0\.0からバックポート* 

* `ssl_cipher_suite`が`old`に設定されている場合、gRPCのTLSのセキュリティレベルを`0`に設定します。 

#### クラスタリング

* HCV Kubernetes認証パスに関連するクラスタ互換性チェックを調整しました。

#### プラグイン

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * `http_response_header_for_traceid`オプションが有効になっている場合に発生するOTELサンプリングモードのLuaパニックバグを修正しました。

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * 認証情報がユーザー名なしでエンコードされている場合に、Kong Gatewayがエラーをスローし、500コードを返す問題を修正しました。

3\.6\.0\.0
-------------

**リリース日** 2024/02/12

### 下位互換性のない変更と非推奨

* Kong Gateway 3\.6\.0\.0が正常に機能するには、ファイル記述の数を1024より高く制限する必要があります。この要件は、今後のバージョンで削除されます。Kong Gateway 3\.6\.0\.0を実行するときは、`ulimit -n`を少なくとも4096に設定することをお勧めします。

* Wasm関連の他の`nginx.conf`ディレクティブとの曖昧さを避けるため、Wasm `shm_kv` nginx.confディレクティブの接頭辞が`nginx_wasm_shm_`から`nginx_wasm_shm_kv_`に変更されました。
  [\#11919](https://github.com/Kong/kong/issues/11919)

* コンシューマグループ（`/consumer_groups`）とコンシューマ（`/consumers`）のリスティングエンドポイントは、結果をページ分割して応答するようになりました。リストのJSONキーが、`consumer_groups`や`consumers`ではなく`data`に変更されました。

* OpenSSL 3\.2では、デフォルトのSSL/TLSセキュリティレベルが1から2に変更されました。これは、セキュリティレベルが112ビットのセキュリティに設定されていることを意味します。その結果、以下のことが禁止されます。

  * 2048ビット未満のRSAキー、DSAキー、DHキー
  * 224ビットより短いECCキー
  * RC4を使用する任意の暗号スイート
  * SSLバージョン3。また、圧縮は無効になります。

* 最近のOpenRestyの更新にはTLS 1\.3が含まれており、TLS 1\.1は非推奨になっています。
  引き続きTLS 1\.1をサポートする必要がある場合は、[`ssl_cipher_suite`](/gateway/latest/reference/configuration/#ssl_cipher_suite)設定を`old`に設定します。

* カスタムコードで`ngx.var.http_*`を使用してHTTPヘッダーにアクセスしている場合、1回のリクエストで同じヘッダーを複数回使用すると、その変数の動作が少し変わります。
  以前は最初の値のみを返していましたが、現在はコンマで区切られたすべての値を返します。Kong GatewayのPDKヘッダーゲッターとセッターは以前と同じように動作します。

* Kong Managerは、OpenID Connectプラグインのセッション管理メカニズムを使用するようになりました。OIDCで認証する場合、`admin_gui_session_conf`は不要になりました。代わりに、セッション関連の構成パラメータは`admin_gui_auth_conf`（`session_secret`など）で設定されます。

  詳細については、[移行ガイド](/gateway/3.6.x/kong-manager/auth/oidc/migrate/)を参照してください。

#### プラグイン

* [**ACME**](/hub/kong-inc/acme/)（`acme`）、[**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）、および[**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）

  * プラグイン間でのRedis構成の標準化。Redis構成は、他のプラグイン間で共有される共通スキーマに従うようになりました。 [\#12300](https://github.com/Kong/kong/issues/12300) [\#12301](https://github.com/Kong/kong/issues/12301)

* [**Azure Functions**](/hub/kong-inc/azure-functions/)（`azure-functions`）:

  * Azure Functionsプラグインで、アップストリーム/リクエストURIが削除され、Azure APIのリクエスト時に[`routeprefix`](/hub/kong-inc/azure-functions/configuration/#config-routeprefix)構成フィールドのみを使用してリクエストパスが構築されるようになりました。 

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）

  * コンテンツタイプが`application/json`でない場合、プラグインはスキーマの検証をバイパスするようになりました。

* [**Proxy Cache Advanced**](/hub/kong-inc/proxy-cache-advanced/)（`proxy-cache-advanced`）

  * OSSからEnterpriseへの移行をブロックしていた不要な`proxy-cache-advanced/migrations/001_035_to_050.lua`ファイルを削除しました。 これは、`0.3.5`と`0.5.0`の間のKong Gatewayバージョンからアップグレードする場合にのみ、下位互換性のない変更となります。

* [**SAML**](/hub/kong-inc/saml)（`saml`）

  * SAMLプラグインと他のコンシューマベースのプラグイン間の統合を修正するために、SAMLプラグインの優先度を1010に調整しました。

### 機能

#### Admin API

* Admin APIのルートエンドポイント（`/`）にKong Gatewayエディションを追加しました。 [\#12097](https://github.com/Kong/kong/issues/12097)
* `127.0.0.1:8007`で`status_listen`がデフォルトで有効になりました。[\#12304](https://github.com/Kong/kong/issues/12304)
* FIPS有効化ステータスがライセンスの変更に応答するようになりました。 現在のステータスを表示するために、新しいエンドポイント`/fips-status`を導入しました。
* `/consumer_group/consumers`と`/consumer/consumer_groups`にページ分割のサポートを追加しました。

#### CLI

* CLIの変更移行コマンドを実行した後、ワークスペースエンティティカウンターを自動的に再初期化します。

#### クラスタリング

* コントロールプレーンAPI応答にデータプレーン証明書の有効期限を追加しました（`/clustering/data-planes`）。 [\#11921](https://github.com/Kong/kong/issues/11921)
* 同種のデータプレーン（DP）デプロイメントに対するレジリエンスサポートが追加されました。データプレーンは、インポーターとエクスポーターとして同時に機能できるようになりました。今後、Kong Gatewayは、構成のエクスポート時に同時実行の制御を試みるようになります。 
* Konnectで実行されているデータプレーンノードは、無効な構成や一時的なエラーなど、構成のリロード障害をコントロールプレーンに報告するようになりました。
* Kong Gatewayは、データプレーンからコントロールプレーンへの接続エラーの原因となる可能性のある構成オプションを示すログエントリを出力するようになりました。

#### 構成

* Kong Managerが有効になっているがAdmin APIが有効になっていない場合に、Kong Gatewayに警告メッセージが表示されるようになりました。[\#12071](https://github.com/Kong/kong/issues/12071)
* 中間構成にDHE\-RSA\-CHACHA20\-POLY1305暗号を追加しました。 [\#12133](https://github.com/Kong/kong/issues/12133)
* `dns_no_sync`構成オプションのデフォルト値が`off`に変更されました。 [\#11869](https://github.com/Kong/kong/issues/11869)
* KongのプロキシロケーションブロックにNginxディレクティブを注入するためのサポートを追加しました。 [\#11623](https://github.com/Kong/kong/issues/11623)
* LMDBキャッシュはKongのバージョン（メジャーとマイナー）によって検証されるようになりました。マイナーバージョンアップグレード中の互換性の問題を回避するため、タグの不一致があればコンテンツを消去します。[\#12026](https://github.com/Kong/kong/issues/12026)
* `dns_stale_ttl`のデフォルトを1時間に変更しました。これにより、リゾルバにダウンタイムが発生した場合でも、古いDNSレコードをより長い時間使用できるようになります。 [\#12087](https://github.com/Kong/kong/issues/12087)
* `nginx_http_keepalive_requests`と`upstream_keepalive_max_requests`のデフォルト値を`10000`に更新しました。これらの変更は、スループットの高いシステムでより適切に動作するように最適化されています。 スループットの低い設定では、これらの新しい設定は、すべてのアップストリームの使用を開始するため、以前よりも多くのリクエストを要するという、ロードバランシングに目に見える影響を与える可能性があります。[\#12223](https://github.com/Kong/kong/issues/12223)

#### コア

* AI Proxy、AI Request Transformer、AI Response Transformerプラグインのテレメトリの収集を追加しました。モデルとプロバイダーの使用に関するものです。 [\#12495](https://github.com/Kong/kong/issues/12495)
* Kong prebuild nginxに`ngx_brotli`モジュールを追加しました。 Kong GatewayでBrotli圧縮を有効にする方法については、[ドキュメント](/gateway/latest/production/performance/brotli/)を参照してください。 [\#12367](https://github.com/Kong/kong/issues/12367)
* プライマリキーを、完全なエンティティとしてDAO関数に渡せるようになりました。 [\#11695](https://github.com/Kong/kong/issues/11695)
* Kong Gateway DockerイメージのDebianバリアントは、Debian 12を使用して構築されるようになりました。 [\#12218](https://github.com/Kong/kong/issues/12218)
* 式ルーターは`!`（not）演算子をサポートするようになりました。これにより、`!(http.path =^ "/a")`および`!(http.path == "/a" || http.path == "/b")`のようなルートを作成できます。[\#12419](https://github.com/Kong/kong/issues/12419)
* 応答が`kong`または`upstream`によって生成されていることを示す`source`プロパティを、ログシリアライザーに追加しました。 [\#12052](https://github.com/Kong/kong/issues/12052)
* システムのパッケージマネージャーを使用してアンインストールした後、Kongが所有するディレクトリがクリーンアップされていることを確認します。 [\#12162](https://github.com/Kong/kong/issues/12162)
* Kong Gatewayが、式ルーターで[`http.path.segments.len`フィールドと`http.path.segments.*`フィールド](/gateway/latest/key-concepts/routes/expressions/#matching-fields)をサポートするようになり、個々のセグメントまたはセグメントの範囲によって受信（正規化）リクエストパスをマッチングして、セグメントの合計数を確認できるようになりました。[\#12283](https://github.com/Kong/kong/issues/12283)
* 式を使用して定義されたHTTPルートで、`net.src.*`と`net.dst.*`の一致フィールドにアクセスできるようになりました。 [\#11950](https://github.com/Kong/kong/issues/11950)
* `kong.*`名前空間の`proxy-wasm`プロパティを介してKong Gatewayの値を取得および設定するためのサポートが拡張されました。 [\#11856](https://github.com/Kong/kong/issues/11856)
* メタスキーマに`examples`フィールドを追加しました。
* 分析プッシャーに新しい`upstream_status`および`source`プロパティを追加しました。
* 分析の`consumer_groups`サポートを追加しました。
* HashiCorp Vaultのシークレット管理バックエンドは、AppRole認証方法をサポートするようになりました。KubernetesでHashiCorp Vaultを使用する際の名前空間認証とユーザー定義の認証パスのサポートを追加しました。
* Kong Gatewayは、一貫性を高めるために、リクエストIDヘッダーによって提供される値をすべてのリクエストIDフィールドに使用するようになりました。
* ドットキー（`a.b.c`など）は、監査リクエストと監査オブジェクトの両方から除外され、単数キー（`password`など）は再帰的に除外されるようになりました。
* Kong Gateway Enterpriseコンテナイメージは、ビルドの来歴とともに生成され、Cosignを使用して署名されるようになりました。 署名と来歴証明はDocker Hubリポジトリに公開されます。 ビルドの来歴は、公開されている来歴証明を使用して[cosign/slsa\-verifierによって検証](/gateway/3.6.x/kong-enterprise/provenance-verification/)できます。

#### Kong Manager Enterprise

* [Kong Managerでグループマッピング](/gateway/3.6.x/kong-manager/auth/oidc/mapping/)を使用する際に、RBACトークンを使用して認証できるようになりました（OIDCまたはLDAPなど）。
* UIからRoute by Headerプラグインを作成および編集するためのサポートを追加しました。
* 新規ユーザーがKong Gatewayを使い始めやすくなるよう、オンボーディングフローを追加しました。
* ワークスペースと概要サマリーページのデザインが新しくなりました。

#### Kong Managerオープンソース

* すべてのエンティティフォームにJSON/YAML形式のプレビューを追加しました。 [\#157](https://github.com/Kong/kong-manager/issues/157)
* UI/UX向上のため、resigned基本コンポーネントを採用しました。 [\#131](https://github.com/Kong/kong-manager/issues/131) [\#166](https://github.com/Kong/kong-manager/issues/166)
* Kong ManagerとKonnectは、プラグイン選択ページとプラグインフォームページで同じUIを共有するようになりました。 [\#143](https://github.com/Kong/kong-manager/issues/143) [\#147](https://github.com/Kong/kong-manager/issues/147)

#### PDK

* JSON数値エンコーディングの精度を14桁から16桁に引き上げました。 [\#12019](https://github.com/Kong/kong/issues/12019)

#### パフォーマンス

* ARM64上の誤ったLuaJIT LDP/STP融合を修正しました。これにより、誤ったロジックが発生することがありました。
* `lua-resty-timer-ng`ライブラリの同時実行範囲を`[32, 256]`から`[512, 2048]`に増やしました。 [\#12275](https://github.com/Kong/kong/issues/12275)
* ルートの統計を構築するときに協力的に譲歩し、プロキシパスのレイテンシへの影響を軽減します。 [\#12013](https://github.com/Kong/kong/issues/12013)

#### プラグイン

**新しいプラグイン** ：

* [**AI Proxy**](/hub/kong-inc/ai-proxy/)（`ai-proxy`）：各種AIプロバイダーの大規模言語モデル（LLM）との統合を簡素化できます。 [\#12323](https://github.com/Kong/kong/issues/12323)
* [**AI Prompt Decorator**](/hub/kong-inc/ai-prompt-decorator/)（`ai-prompt-decorator`）: プロンプトのチューニングのために、`llm/v1/chat`メッセージをコンシューマLLMリクエストの先頭と末尾に追加します。 [\#12336](https://github.com/Kong/kong/issues/12336)
* [**AI Prompt Guard**](/hub/kong-inc/ai-prompt-guard/)（`ai-prompt-guard`）: パターンマッチングに基づいて、LLMリクエストの許可リストまたはブロックリストを設定します。[\#12427](https://github.com/Kong/kong/issues/12427)
* [**AI Prompt Template**](/hub/kong-inc/ai-prompt-template/)（`ai-prompt-template`）：変数で置き換え可能なLLMプロンプトテンプレートの配列を設定します。 [\#12340](https://github.com/Kong/kong/issues/12340)
* [**AI Request Transformer**](/hub/kong-inc/ai-request-transformer/)（`ai-request-transformer`）: 処理中のクライアントリクエストをLLMに渡して変換またはサニタイズします。 [\#12426](https://github.com/Kong/kong/issues/12426)
* [**AI Response Transformer**](/hub/kong-inc/ai-response-transformer/)（`ai-response-transformer`）: 処理中のアップストリーム応答をLLMに渡して変換またはサニタイズします。[\#12426](https://github.com/Kong/kong/issues/12426)

これらのプラグインの詳細については、[AI Gatewayクイックスタート](/gateway/latest/get-started/ai-gateway/)を参照してください。

**既存のプラグイン** :

* **コンシューマグループのサポート** : 次のプラグインをコンシューマグループに絞り込めるようになりました。

  * IP Restriction
  * Rate Limiting
  * Request Termination
  * Proxy Cache
  * Proxy Cache Advanced

* [**ACL**](/hub/kong-inc/acl/)（`acl`）

  * プラグインに構成パラメータ`include_consumer_groups`が追加され、Kongのコンシューマグループを許可リストと拒否リストに追加するかどうか指定できるようになりました。

* [**AppDynamics**](/hub/kong-inc/app-dynamics/)（`app-dynamics`）

  * このプラグインは、`CONTROLLER_CERTIFICATE_FILE`および`CONTROLLER_CERTIFICATE_DIR`環境構成オプションを介した自己署名証明書の使用をサポートするようになりました。

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * このプラグインは、非標準の`asn1`整数のデコードと、冗長な先頭パディングでエンコードされた列挙型のデコードをサポートするようになりました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 構成パラメータ`scopes`、`login_redirect_uri`、`logout_redirect_uri`、および`introspection_headers_values`が、Kong Vault内のシークレットとして参照できるようになりました。
  * ヘッダーからの注入をサポートするために`token_post_args_client`構成パラメータを拡張しました。
  * コード交換のための明示的な証明キー（PKCE）のサポートを追加しました。
  * プッシュされた認可リクエスト（PAR）のサポートを追加しました。
  * `tls_client_auth`および`self_signed_tls_client_auth`認証方法のサポートを追加し、IdPを使用した[mTLSクライアント認証](/hub/kong-inc/openid-connect/how-to/client-authentication/mtls/)が可能になりました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * トレースのサンプリングレートは、Kong Gatewayのグローバル設定ではなく、OpenTelemetryプラグインの[`config.sampling_rate`](/hub/kong-inc/opentelemetry/configuration/#configsampling_rate)プロパティを介して設定できるようになりました。[\#12054](https://github.com/Kong/kong/issues/12054)

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * RLAスライディングウィンドウの重みの解像度を向上しました。

### 修正

#### Admin API

* Admin APIでの認証失敗に対するエラー応答を強化しました。[\#12456](https://github.com/Kong/kong/issues/12456)
* `/rbac/roles/:role/endpoints`エンドポイントが`actions`を配列として受け入れない問題を修正しました。
* ワークスペースのリスティングAPIは、現在のユーザーがエンドポイントに関連付けられているワークスペースのみを表示するようになりました。
* タイムスタンプフィールドによるページ分割やソートを行うとHTTP 500エラーが返される問題を修正しました（`created_at`）など。
* 同じRBACユーザーで、`user_token`を同じ値で更新しようとした場合に固有の違反エラーが報告される問題を修正しました。
* 管理者とRBACユーザーが自分のロールを更新できないようにしました。

#### CLI

* CLIは、CEからEEに移行するときにワークスペースエンティティカウンターを再初期化しなくなりました。

#### クラスタリング

* データプレーン（DP）から空のPINGフレームを受信すると、データプレーン（DP）のステータス更新に失敗するバグを修正しました。 [\#11917](https://github.com/Kong/kong/issues/11917)
* ハイブリッドモードでデータプレーンのログシリアライザー出力にワークスペース名が表示される問題を修正しました。
* `cluster_telemetry_endpoint`構成が無効になっている場合のメッセージプッシュエラーログが削減されました。
* クラスタリング分析では、特権エージェントのワーカーIDとして`-1`が表示されるようになりました。

#### 構成

* `declarative_config_flattened`関数内の弱く型付けされた`of`関数によってデータ損失エラーが引き起こされる問題を修正しました。 [\#12167](https://github.com/Kong/kong/issues/12167)
* Kong Gatewayはカスタム`proxy_access_log`値を遵守するようになりました。 [\#12073](https://github.com/Kong/kong/issues/12073)

#### コア

* 他のエンティティが参照中のCA証明書は、削除できなくなりました。 関連するCAストアキャッシュは、CA証明書の更新時に無効になります。 [\#11789](https://github.com/Kong/kong/issues/11789)
* Cookie名はRFC 6265に照らして検証されるようになり、以前の検証よりも多くの文字を使用できます。[\#11881](https://github.com/Kong/kong/issues/11881)
* スキーマに変換定義がある場合にのみ、nullが削除されるようになりました。 ほとんどのスキーマでは変換が定義されていないため、これによりパフォーマンスが向上します。 [\#12284](https://github.com/Kong/kong/issues/12284)
* 内部エラーコード494がトリガーされたときに、`error_handler`が意味のある応答本文を提供できなかったバグを修正しました。 [\#12114](https://github.com/Kong/kong/issues/12114)
* `expressions`ルーターフレーバーのヘッダー値マッチング（`http.headers.*`）で、大文字と小文字が区別されるようになりました。 この変更は、ヘッダー値マッチングが常に大文字と小文字を無視して実行される`traditional_compatible`モードには影響しません。[\#11905](https://github.com/Kong/kong/issues/11905)
* プラグインが失敗したときに表示される誤ったエラーメッセージを修正しました。 [\#11800](https://github.com/Kong/kong/issues/11800)
* LuaJITエラーによって発生する断続的なldoc障害を修正しました。 [\#11983](https://github.com/Kong/kong/issues/11983)
* `NGX_WASM_MODULE_BRANCH`環境変数は、Kongをビルドするときに`ngx_wasm_module`リポジトリブランチを設定するようになりました。 [\#12241](https://github.com/Kong/kong/issues/12241)
* ハングするリスクを回避するために、`syncQuery()`の非同期タイマーを削除しました。 [\#11900](https://github.com/Kong/kong/issues/11900)
* トレースの修正: 
  * DNSクエリの失敗によりトレーシングが失敗する問題を修正しました。 [\#11935](https://github.com/Kong/kong/issues/11935)
  * DNSスパンは、コソケットクエリに加えて、アップストリームDNSクエリでも正しく生成されるようになりました。 [\#11996](https://github.com/Kong/kong/issues/11996)

* `http`および`stream`サブシステム内の式ルートの検証がより厳格になりました。 以前これらは同じ検証スキーマを共有していたため、ストリームルートであっても、管理者は`http.path`のようなフィールドを使用して式ルートを設定できていましたが、これは許可されなくなりました。[\#11914](https://github.com/Kong/kong/issues/11914)
* Kong Gatewayで`keys`エンティティの秘密鍵と公開鍵の検証プロセスを追加して、相互が確実に一致するようにしました。[\#11923](https://github.com/Kong/kong/issues/11923)
* フィルターがアクセスハンドラの再入をトリガーするときに発生していた`proxy-wasm`の`previous plan already attached`エラーを修正しました。 [\#12452](https://github.com/Kong/kong/issues/12452)
* すべてのワークスペースで欠落しているエンドポイントの追加が必要となっていたRBACの問題を修正しました。
* Redi流量制限ツールからの混乱を招くデバッグログエントリを無視しました。
* ワークロードIDが、データプレーンのレジリエンスに対して機能しない問題を修正しました。
* シークレットを取得できなかったときに、GCPバックエンドVaultがエラーメッセージを非表示にする問題を修正しました。
* フィルターを使用する際に、リクエストデバッグの出力で欠落していた`workspace_id`を追加しました。
* 基盤となるAWS認証情報の有効期限が切れたときにIAM認証トークンが更新されない問題を修正しました。
* Redisの`timeout`警告メッセージは、タイムアウトが明示的に設定されている場合にのみ出力されます。設定されていない場合は、デフォルトのタイムアウト値が使用されます。
* 外部プラグインサーバーの起動時に表示される不正確な重大レベルのログを削除しました。 OpenRestyの制限により、これらのログを抑制することはできません。ソケット可用性の検出機能を削除することにしました。

#### Kong Manager Enterprise

* `session`、`response_mode`、RP開始ログアウトなど、OpenID Connectを使用したAdmin GUI認証に関する問題を修正しました。
* 外部ソース（OIDCやLDAPなど）からロールをマッピングする場合、［チーム］でのUIの説明を修正しました。
* Kong Managerは、`/keys/*`エンドポイントに対する権限なしで、特定のキーセットにスコープ設定されたキーの操作をサポートするようになりました。
* OpenID Connect経由でAdmin APIを認証する際に発生していたさまざまな問題を修正しました。

#### Kong Managerオープンソース

* 通知テキスト形式を標準化しました。[\#140](https://github.com/Kong/kong-manager/issues/140)

#### PDK

* スパンの不要な作成とガベージコレクションを回避し、パフォーマンスを最適化しました。 [\#12080](https://github.com/Kong/kong/issues/12080)
* `response.set_header` 文字列のテーブル配列を含むヘッダー引数を正しくサポートするようになりました。 [\#12164](https://github.com/Kong/kong/issues/12164)
* `kong.response.exit`を使用すると、ユーザーが設定したTransfer\-Encodingヘッダーが削除されない問題を修正しました。 [\#11936](https://github.com/Kong/kong/issues/11936)
* **プラグインサーバー** : リクエストのたびに新しいプラグインインスタンスが作成される問題を修正しました。 [\#12020](https://github.com/Kong/kong/issues/12020)

#### プラグイン

* [**Basic Authentication**](/hub/kong-inc/basic-auth/)（`basic-auth`）

  * 401応答に欠落していた`WWW-Authenticate`ヘッダを追加しました。 [\#11795](https://github.com/Kong/kong/issues/11795)

* [**Datadog**](/hub/kong-inc/datadog/)（`datadog`）

  * サービスレスルートでプラグインがトリガーされないバグを修正しました。 Datadogプラグインは常にトリガーされ、`name`タグの値（`service_name`）は空の値として設定されます。 [\#12068](https://github.com/Kong/kong/issues/12068)

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）

  * リクエストボディがすでに読み込まれている場合、プラグインは非ストリーミングプロキシにフォールバックするようになりました。
  * ペイロードが`client_body_buffer_size`を超えた場合にリクエストペイロードが破棄される問題を修正しました。

* [**JWE Decrypt**](/hub/kong-inc/jwe-decrypt/)（`jwe-decrypt`）

  * エラーメッセージのタイプミスを修正しました。

* [**JWT Signer**](/hub/kong-inc/jwt-signer/)（`jwt-signer`）

  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。
  * 200以外の応答後に`groups_required`が予期しないコードを返す原因となっていたキャッシュ関連の問題を修正しました。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * 有効な再帰スキーマが常に拒否される問題を修正しました。
  * `responses`に`default`や`2XX`のようなワイルドカードコードが含まれていると、プラグインがモック応答を返さない問題を修正しました。
  * 失効チェックが`revocation_check_mode = IGNORE_CA_ERROR`で失敗した場合、プラグインは`notice`ログエントリを出力するようになりました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）

  * Refパラメータスキーマが逆参照されていないために、プラグインがランタイムエラーをスローするバグを修正しました。
  * サービスレスルートのメトリクスを公開しました。
  * AnyTypeスキーマとスタイルキーワードが定義されているパラメータを検証する際に、プラグインがランタイムエラーをスローする問題を修正しました。
  * Cookieのパラメータが検証されない問題を修正しました。
  * `nullable`キーワードが有効にならない問題を修正しました。
  * 正規表現のエスケープ文字が含まれている場合にリクエストパスが一致しない問題を修正しました。 URIコンポーネントのエスケープ文字は、誤ってエスケープ解除されていました。

* [**OAuth 2\.0 Introspection**](/hub/kong-inc/oauth2-introspection/)（`oauth2-introspection`）

  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。
  * `authorization_value`構成パラメータを暗号化できるようになりました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 特に、ログアウトのためにクエリ文字列を渡す場合、`ngx.var.request_uri`ではなく`kong.request.get_forwarded_path()`の正規化バージョンを使用することで、ログアウトURIサフィックスの検出の問題を修正しました。
  * `introspection_headers_values`構成パラメータを暗号化できるようになりました。
  * 不要な引数`ignore_signature.userinfo`を`userinfo_load`関数から削除しました。
  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。
  * 構成`issuer`と`extra_jwks_uris`に同じURIが含まれている場合に発生するキャッシュキーの衝突を修正しました。
  * プラグインは、トークンの有効期限チェックの境界条件を正しく処理するようになりました。
  * プラグインは、トークンの有効期限を計算するときに時間を更新するようになりました。

* [**Prometheus**](/hub/kong-inc/prometheus/)（`prometheus`）

  * サービスレスルートのメトリクスを公開しました。[\#11781](https://github.com/Kong/kong/issues/11781)

* [**Proxy Cache Advanced**](/hub/kong-inc/proxy-cache-advanced/)（`proxy-cache-advanced`）

  * OSSからEnterpriseへの移行をブロックしていた不要な`proxy-cache-advanced/migrations/001_035_to_050.lua`ファイルを削除しました。 これは、`0.3.5`と`0.5.0`の間のKong Gatewayバージョンからアップグレードする場合にのみ、下位互換性のない変更となります。

* [**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）

  * このプラグインでは、`sync_rate`がRedisポリシーで使用される場合に、カウンターの精度が向上するようになりました。 [\#11859](https://github.com/Kong/kong/issues/11859)
  * すべてのカウンターが同じレートで同じデータベースに同期される問題を修正しました。 [\#12003](https://github.com/Kong/kong/issues/12003)

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * プラグインは、Redisパイプライン内のクエリエラーをチェックするようになりました。
  * プラグインは、`configure()`フェーズを呼び出すときに`sync_rate`が`nil`か`null`かをチェックするようになりました。 `nil`または`null`の場合、プラグインはデータベースまたはRedisとの同期をスキップします。

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * プラグインは、`json`がリクエストコンテンツタイプのサブタイプ（たとえば`application/merge-patch+json`）のサフィックス値である場合に、リクエストボディスキーマを検証するようになりました。

* [**Route Transformer Advanced**](/hub/kong-inc/route-transformer-advanced/)（`route-transformer-advanced`）

  * エラーメッセージを改善しました。

* [**SAML**](/hub/kong-inc/saml/)（`saml`）

  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

### 依存関係

* `atc-router`を1\.2\.0 から 1\.6\.0にアップグレードしました [\#12231](https://github.com/Kong/kong/issues/12231)
* `kong-lapis`を1\.14\.0\.3から1\.16\.0\.1に更新しました [＃12064](https://github.com/Kong/kong/issues/12064)
* `LPEG`を1\.0\.2から1\.1\.0に更新しました [\#11955](https://github.com/Kong/kong/issues/11955)
* `lua-messagepack`を0\.5\.2から0\.5\.3に更新しました [\#11956](https://github.com/Kong/kong/issues/11956)
* `lua-messagepack`を0\.5\.3から0\.5\.4に更新しました [\#12076](https://github.com/Kong/kong/issues/12076)
* `lua-resty-aws`を1\.3\.5から1\.3\.6に更新しました [\#12439](https://github.com/Kong/kong/issues/12439)
* `lua-resty-healthcheck`を3\.0\.0から3\.0\.1にアップグレードしました [＃12237](https://github.com/Kong/kong/issues/12237)
* `lua-resty-lmdb`を1\.3\.0から1\.4\.1に更新しました [\#12026](https://github.com/Kong/kong/issues/12026)
* `lua-resty-timer-ng`を0\.2\.5から0\.2\.6に更新しました [\#12275](https://github.com/Kong/kong/issues/12275)
* `OpenResty`を1\.21\.4\.2から1\.25\.3\.1に更新しました [\#12327](https://github.com/Kong/kong/issues/12327)
* `OpenSSL`を3\.1\.4から3\.2\.1に更新しました [\#12264](https://github.com/Kong/kong/issues/12264)

<!-- -->
* `resty-openssl`を0\.8\.25から1\.2\.0に更新します [\#12265](https://github.com/Kong/kong/issues/12265)

<!-- -->
* `ngx_brotli`をマスターブランチにアップグレードし、ツールチェーンの問題により、rhel7、rhel9\-arm64、amazonlinux\-2023\-arm64で無効にしました。 [\#12444](https://github.com/Kong/kong/issues/12444)
* `lua-resty-healthcheck`を1\.6\.3から3\.0\.0に更新しました [\#11834](https://github.com/Kong/kong/issues/11834)
* `ngx_wasm_module`を`a7087a37f0d423707366a694630f1e09f4c21728`に更新しました [\#12011](https://github.com/Kong/kong/issues/12011)
* `Wasmtime`を`14.0.3`に更新しました [\#12011](https://github.com/Kong/kong/issues/12011)
* サブモジュールの`kong-openid-connect`を2\.7\.0に更新しました
* `kong-redis-cluster`を1\.5\.3に更新しました
* `jq`を1\.7\.1に更新しました
* `luasec`を1\.3\.2に更新しました

### 既知の問題

* 最近のOpenRestyの更新にはTLS 1\.3が含まれており、TLS 1\.1は非推奨になっています。 引き続きTLS 1\.1をサポートする必要がある場合は、[`ssl_cipher_suite`](/gateway/latest/reference/configuration/#ssl_cipher_suite)設定を`old`に設定します。
* カスタムコードで`ngx.var.http_*`を使用してHTTPヘッダーにアクセスする場合、1回のリクエストで同じヘッダーを複数回使用すると、その変数の動作が少し変わります。以前は最初の値のみを返していましたが、現在はカンマで区切られたすべての値を返します。KongのPDKヘッダーゲッターとセッターは以前と同じように動作します。

3\.5\.0\.7
-------------

**リリース日** 2024/07/09

### 非推奨

* Debian 10、CentOS 7、RHEL 7は、2024年6月30日にサポート終了（EOL）となりました。このパッチの時点では、Kongはこれらのオペレーティングシステム用のKong Gateway 3\.5\.xインストールパッケージまたはDockerイメージを構築していません。Kongは、これらのシステムで実行されるKongバージョンに対して公式サポートを提供しなくなりました。

### 機能

#### プラグイン

*3\.7\.1\.2からバックポート* 

* [**AWS Lambda**](/hub/kong-inc/aws-lambda)（`aws-lambda`）
  * 新しい構成パラメータ`empty_arrays_mode`が追加されました。これにより、Kong GatewayがLambda関数によって返された空の配列（`[]`）を空の配列（`[]`として送信するか、JSON応答で空のオブジェクト（`{}`）として送信するかを制御できるようになりました。

3\.5\.0\.6
-------------

**リリース日** 2024/06/22

### 修正

* DNSクライアントがDNS応答で`ADDITIONAL SECTION`のコンテンツを誤って使用していた問題を修正しました。

3\.5\.0\.5
-------------

**リリース日** 2024/06/18

### 既知の問題

* DNSクライアントがDNS応答でコンテンツ`ADDITIONAL SECTION`を誤って使用するという修正に関する問題がありました。 この問題を回避するには、このパッチの代わりに、3\.5\.0\.6をインストールしてください。

### 機能

#### Admin API

*3\.7\.0\.0からバックポート* 

* 検索フィールドにLHSブラケットのフィルタリングを追加しました。
* **監査ログ:** 
  * `request_timestamp`を`audit_objects`に追加しました。
  * LHS Bracketsフィルターの前後にエイリアスを追加しました。
  * `audit_requests`と`audit_objects`は`request_timestamp`でフィルタリングできるようになりました。

#### プラグイン

*3\.6\.1\.4からバックポート* 

* [**Portal Application Registration**](/hub/kong-inc/application-registration/)（`application-registration`） 
  * 認証情報によるコンシューマ認証を使用したサービスにアクセスできるようになりました。 この機能を使用するには、`enable_proxy_with_consumer_credential`を有効にします（デフォルトは`false`）。

### 修正

#### コア

*3\.7\.1\.0からバックポート* 

* **DNSクライアント** : Kong DNSクライアントが回答を解析する際に、ドメインとタイプが一致しないレコードを保存する問題を修正しました。 回答を解析するときに、RRタイプがクエリのタイプと異なる場合はレコードが無視されるようになりました。
* 接続の再試行中に、アップストリームエンティティの`host_header`属性がアップストリームへのリクエストでホストヘッダーとして正しく設定されない問題を修正しました。
* 管理者用の組み込みRBACロール（デフォルトのワークスペースでは`admin`、デフォルト以外のワークスペースでは`workspace-admin`）で、`/groups`および`/groups/*`エンドポイントへのCRUDアクションが禁止されるようになりました。

#### プラグイン

*3\.7\.1\.0からバックポート* 

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 特定の条件下で、匿名コンシューマが`nil`としてキャッシュされる問題を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * 中央のデータストアでネットワークが不安定になっても、タイマーのスパイクが発生しなくなりました。

#### Admin API

*3\.7\.0\.0からバックポート* 

* `/<workspace id="sl-md0000000">/admins`エンドポイントが誤って使用され、割り当てられたRBACのロールに基づいてワークスペースに関連付けられている管理者を返していました。この問題は修正され、特定のワークスペースの関連付けに応じて管理者が正確に返されるようになりました。

*3\.6\.0\.0からバックポート* 

* ユーザーがロールを持っていないワークスペースを表示するワークスペースリスティングAPIの問題を修正しました。 APIには、ユーザーがアクセスできるワークスペースのみが表示されるようになりました。

### 依存関係

* エラーログを改良するために、`lua-resty-azure`を1\.4\.1から1\.5\.0に更新しました。
* `lua-resty-events`0\.2\.1にアップグレードしました。
* 多数のタイマーを実行する代わりに、同じアクティブなヘルスチェックターゲットに対してタイマーを再利用することによってメモリリークの問題を修正するために、`lua-resty-healthcheck`を1\.6\.4から1\.6\.5に更新しました。

3\.5\.0\.4
-------------

**リリース日** 2024/05/20

### 下位互換性のない変更

*3\.6\.1\.0からバックポート* 

* OpenSSL 3\.2では、SSL/TLSのデフォルトのセキュリティレベルは1から2、つまり112ビットセキュリティに変更されています。 この変更の結果、以下の要素は禁止されています。 
  * 2048ビット未満のRSAキー、DSAキー、DHキー
  * 224ビットより短いECCキー
  * RC4を使用する任意の暗号スイート
  * SSLバージョン3。また、圧縮は無効になります。

* 最近のOpenRestyの更新にはTLS 1\.3が含まれており、TLS 1\.1は非推奨になっています。 引き続きTLS 1\.1をサポートする必要がある場合は、[`ssl_cipher_suite`](/gateway/3.5.x/reference/configuration/#ssl_cipher_suite)設定を`old`に設定します。

### 機能

#### 構成

*3\.7\.0\.0からバックポート* 

* OpenSSL 3\.xでは、TLSv1\.1およびそれ以下はデフォルトで無効になっています。 

*3\.6\.0\.0からバックポート* 

* 同種のデータプレーンデプロイメントに対するレジリエンスサポートが追加されました。 データプレーンは、インポーターとエクスポーターとして同時に機能できるようになり、Kong Gatewayは、構成をエクスポートするときに同時実行を制御しようとします。

#### コア

*3\.6\.1\.0からバックポート* 

* HashiCorp Vaultのシークレット管理バックエンドは、AppRole認証方法をサポートするようになりました。

*3\.6\.0\.0からバックポート* 

* [Kong Managerでグループマッピング](/gateway/3.5.x/kong-manager/auth/oidc/mapping/)を使用する際に、RBACトークンを使用して認証できるようになりました（OIDCまたはLDAPなど）。
* 式ルーター:
  * 式ルーターが`!`（not）演算子をサポートするようになりました。これにより、`!(http.path =^ "/a")`や`!(http.path == "/a" || http.path == "/b")`のようなルートを作成できるようになります。 
  * Kong Gatewayが、式ルーターで[`http.path.segments.len`フィールドと`http.path.segments.*`フィールド](/gateway/3.5.x/key-concepts/routes/expressions/#matching-fields)をサポートするようになり、個々のセグメントまたはセグメントの範囲によって受信（正規化）リクエストパスをマッチングして、セグメントの合計数を確認できるようになりました。
  * 式を使用して定義されたHTTPルートで、[`net.src.*`と`net.dst.*`](/gateway/3.5.x/key-concepts/routes/expressions/#matching-fields)の一致フィールドにアクセスできるようになりました。 

#### Admin API

*3\.7\.0\.0からバックポート* 

* `audit_requests`のデフォルトの順序を、`request_timestamp`の降順で並べ替えるように変更しました。

*3\.6\.0\.0からバックポート* 

* Admin APIのルートエンドポイントにKong Gatewayエディションを追加しました。 

#### プラグイン

*3\.7\.0\.0からバックポート* 

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`） 
  * `default_consumer`オプションを追加しました。これにより、クライアント証明書が有効だが既存のコンシューマに一致しない場合に、デフォルトのコンシューマを使用できるようになります。

*3\.6\.1\.2からバックポート* 

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * `api_spec`がURIエンコードされているかどうかを示す新しいフィールド`api_spec_encoded`が追加されました。

*3\.6\.0\.0からバックポート*
[**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

* プラグインは、非標準の`asn1`整数のデコードと、冗長な先頭パディングでエンコードされた列挙型のデコードをサポートするようになりました。

### 修正

#### Admin API

*3\.6\.0\.0からバックポート* 

* タイムスタンプフィールドによるページ分割やソートを行うとHTTP 500エラーが返される問題を修正しました（`created_at`）など。
* 管理者またはRBACユーザーは、自分のロールを更新できなくなりました。

#### クラスタリング

* ハイブリッドモードで、イベントフックが早期に検証される問題を修正しました。 この修正により、イベントフックの検証は、イベントフックが発行される時点まで遅延されます。

*3\.6\.1\.2からバックポート* 

* AWS Secrets Managerに関連するクラスタリング互換性チェックを調整し、`AK-SK`環境変数を使用してIAMロールの権限を付与するようにしました。

*3\.6\.1\.0からバックポート* 

* HCV Kubernetes認証パスに関連するクラスタ互換性チェックを調整しました。

*3\.6\.0\.0からバックポート* 

* `cluster_telemetry_endpoint`構成が無効になっている場合のメッセージプッシュエラーログが削減されました。

#### 構成

*3\.7\.0\.0からバックポート* 

* 外部プラグイン（Go、Javascript、Python）が、Admin APIを介してプラグイン構成に変更を適用する際に失敗する問題を修正しました。
* `ssl_cipher_suite`が`old`に設定されている場合、gRPCのTLSのセキュリティレベルを`0`に設定します。 

#### コア

* ハイブリッドモードのデータプレーンで、Vaultリファレンスで構成されている証明書エンティティが時間通りに更新されないことがあった問題を修正しました。

* Kong Gatewayで外部プラグインサーバーが自動的に起動しない問題を修正しました。

*3\.7\.0\.0からバックポート* 

* `init_worker`フェーズでVaultリファレンスの解決をタイマーに延期することで、Vaultの初期化を修正しました。 
* `kong.logrotate`のファイル権限を644に更新しました。
* リクエストデバッグの出力で欠落していたルーターセクションを修正しました。 
* Vault: 
  * プレフィックスでVaultエンティティを取得するときに、Vaultが誤った（デフォルトの）ワークスペース識別子を使用する問題を修正しました。
  * 最初の構成プッシュ後に、新しいデータプレーンがVaultリファレンスを解決できなかった問題を修正しました。これは、ライセンスのプリロードの問題が原因で発生していました。

*3\.6\.0\.0からバックポート* 

* `expressions`ルーターフレーバーのヘッダー値マッチング（`http.headers.*`）で、大文字と小文字が区別されるようになりました。 この変更は、ヘッダー値マッチングが常に大文字と小文字を無視して実行される`traditional_compatible`モードには影響しません。
* `http`および`stream`サブシステム内の式ルートの検証がより厳格になりました。 以前これらは同じ検証スキーマを共有していたため、ストリームルートであっても、管理者は`http.path`のようなフィールドを使用して式ルートを設定できていましたが、これは許可されなくなりました。
* すべてのワークスペースで欠落しているエンドポイントの追加が必要となっていたRBACの問題を修正しました。
* ワークロードIDが、データプレーンのレジリエンスに対して機能しない問題を修正しました。
* シークレットを取得できなかったときに、GCPバックエンドVaultがエラーメッセージを非表示にする問題を修正しました。

#### Kong Manager Enterprise

*3\.6\.1\.3からバックポート* 

* `admin_gui_path`がスラッシュでない場合に、管理者アカウントプロファイルページで404エラーが返される問題を修正しました。

*3\.6\.1\.2からバックポート* 

* ライセンスの有効期限の残り日数の表示を修正しました。ワークスペースページとトップバナーの間で日数が一致していませんでした。
* RBACユーザーのRBACトークンのタイプを`password`に更新しました。

#### PDK

*3\.7\.0\.0からバックポート* 

* `kong.request.get_forwarded_port`が`ngx.ctx.host_port`から誤った文字列を返す問題を修正しました。数値が正しく返されるようになりました。

#### プラグイン

*3\.6\.1\.4からバックポート* 

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）、 [**WebSocket Size Limit**](/hub/kong-inc/websocket-size-limit/)（`websocket-size-limit`）、 [**WebSocket Validator**](/hub/kong-inc/websocket-validator/)（`websocket-validator`）、 [**XML Threat Protection**](/hub/kong-inc/xml-threat-protection/)（`xml-threat-protection`）
  * プラグイン間の衝突を防ぐために、これらのプラグインの優先順位が更新されました。 バンドルされたプラグインの相対的な優先順位（および実行順序）は変更されません。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * `kong/tools/public/rate-limiting`をリファクタリングし、異なるプラグインを分離するための新しいインターフェース`new_instance`を追加しました。
    後方互換性のため、元のインターフェースは変更されていません。

    このライブラリをベースにしたカスタムのRate Limitingプラグインを使用している場合は、初期化コードを新しい形式に更新してください。たとえば、`'local ratelimiting = require("kong.tools.public.rate-limiting").new_instance("custom-plugin-name")'`などです。
    古いインターフェースは、今後のメジャーリリースで削除されます。

*3\.6\.1\.3からバックポート* 

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry)（`opentelemetry`）
  * 短いトレースIDの解析のロバスト性が向上しました。

*3\.6\.1\.2からバックポート* 

* [**ACME**](/hub/kong-inc/acme/)（`acme`）
  * ACME更新中に証明書が正常に更新されない問題を修正しました。 

* [**DeGraphQL**](/hub/kong-inc/degraphql/)（`degraphql`）
  * GraphQL変数が正しく解析されず、定義された型に強制変換されない問題を修正しました。

* [**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）、[**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）、[**GraphQL Rate Limiting Advanced**](/hub/kong-inc/graphql-rate-limiting-advanced/)（`graphql-rate-limiting-advanced`）、[**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）
  * `rate-limiting`ライブラリを使用するプラグインを一緒に使用すると、互いに干渉し合い、カウンターデータを中央のデータストアに同期できなくなる問題を修正しました。

*3\.6\.1\.1からバックポート* 

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * `redis`ストラテジで使用される`sync_rate`設定に関する問題を修正しました。 `sync_rate = 0`中にRedis接続が中断された場合、プラグインは正確に`local`ストラテジにフォールバックするようになりました。
  * `sync_rate`が`0`より大きい値から`0`に変更された場合、名前空間が予期せずクリアされる問題を修正しました。
  * カウンター同期タイマーが適切に作成または破棄されないというタイマー関連の問題を修正しました。
  * プラグイン作成時ではなく、プラグイン実行中にカウンター同期タイマーを作成するようになったことで、無意味なエラーログを減らせるようになりました。

*3\.6\.1\.0からバックポート* 

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * 認証情報がユーザー名なしでエンコードされている場合に、Kong Gatewayが500エラーコードを返す問題を修正しました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry)（`opentelemetry`）

  * *3\.6\.1\.0からバックポート* : `http_response_header_for_traceid`オプションが有効になっている場合に発生するOTELサンプリングモードのLuaパニックバグを修正しました。

*3\.6\.0\.0からバックポート* 

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）
  * リクエストボディがすでに読み込まれている場合、プラグインは非ストリーミングプロキシにフォールバックするようになりました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）
  * `introspection_headers_values`を暗号化された参照可能なフィールドとしてマークしました。
  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

* [**OAuth 2\.0 Introspection**](/hub/kong-inc/oauth2-introspection/)（`oauth2-introspection`）
  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

* [**SAML**](/hub/kong-inc/saml)（`saml`）
  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

* [**JWT Signer**](/hub/kong-inc/jwt-signer/)（`jwt-signer`）
  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`） 
  * 200以外の応答後に`groups_required`が予期しないコードを返す原因となっていたキャッシュ関連の問題を修正しました。
  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * Cookieのパラメータが検証されない問題を修正しました。

### パフォーマンス

#### 構成

*3\.6\.0\.0からバックポート* 

* `nginx_http_keepalive_requests`と`upstream_keepalive_max_requests`のデフォルト値を10000に更新しました。

#### コア

* 予期しない入力を処理する際の`lua-cjson`のロバスト性を改善しました。
* 頻繁なメモリ割り当てや割り当て解除を避けるために、リクエスト間でマッチコンテキストを再利用します。

#### プラグイン

*3\.7\.0\.0からバックポート* 

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry)（`opentelemetry`）
  * キューの最大バッチサイズを200に増やしました。

### 依存関係

* `atc-router`を1\.2\.0から1\.6\.0に更新しました。
* `lua-protobuf`を0\.5\.1にアップグレードしました。
* `lua-resty-openssl`を1\.2\.1にアップグレードしました。
* OpenSSLを3\.1\.4から3\.2\.0に更新しました。
* `resty-openssl`を0\.8\.25から1\.2\.0に更新しました。
* `lua-kong-nginx-module`を0\.8\.1に更新しました。
* `kong-lua-resty-kafka`を`0.18`に更新しました。
* `lua-resty-luasocket`を`1.1.2`に更新し、[luasocket\#427](https://github.com/lunarmodules/luasocket/issues/427)を修正しました。
* `lua-resty-healthcheck`を1\.6\.4に更新しました。
* `lua-resty-aws`を1\.3\.6にアップグレードしました。

3\.5\.0\.3
-------------

**リリース日** 2024/01/26

### 機能

* Kong Gateway DockerイメージのDebianバリアントは、Debian 12を使用して構築されるようになりました。
  [\#7673](https://github.com/Kong/kong/issues/7673)

* Admin APIとKong Managerの両方で、ネストされたコンシューマリストとコンシューマグループリストのページ分割へのサポートを追加しました。

### 修正

#### Kong Manager

* 動的な順序付けドロップダウンリストにカスタムプラグインが表示されない問題を修正しました。
* `default`以外のワークスペースで、ターゲットページに404エラーが表示される問題を修正しました。
* 現在のワークスペースのロールを`workspace-super-admin`で作成できない問題を修正しました。

### 依存関係

* `kong-redis-cluster`を1\.5\.3に更新しました。

3\.5\.0\.2
-------------

**リリース日** 2023/12/21

### 下位互換性のない変更

#### プラグイン

* [**SAML**](/hub/kong-inc/saml)（`saml`）
  * SAMLプラグインと他のコンシューマベースのプラグイン間の統合を修正するために、SAMLプラグインの優先度を1010に調整しました。

### 機能

#### 構成

* [`dns_no_sync`](/gateway/3.4.x/reference/configuration/#dns_no_sync)オプションのデフォルト値が`off`に変更されました。

#### プラグイン

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）
  * 構成`scopes`、`login_redirect_uri`、`logout_redirect_uri`をKong Vault内のシークレットとして参照できるようになりました。
  * `token_post_args_client`を拡張して、ヘッダーからの注入をサポートします。

### 修正

#### コア

* 流量制限のRedisツールからの混乱を招くデバッグログを無視しました。
* フィルターを使用する際に、リクエストデバッグの出力で欠落していた`workspace_id`を修正しました。
* クエリがハングするリスクを回避するために、syncQuery\(\)の非同期タイマーを削除しました。
* LuaJITエラーによって発生する断続的なldoc障害を修正しました。[\#7494](https://github.com/Kong/kong/issues/7494)

#### PDK

* リクエストのたびに新しいプラグインインスタンスが作成されるプラグインサーバーの問題を修正しました。

#### プラグイン

* [**OAuth 2\.0 Introspection**](/hub/kong-inc/oauth2-introspection/)（`oauth2-introspection`）
  * `authorization_value`を暗号化されたフィールドとしてマークしました。

* [**JWE Decrypt**](/hub/kong-inc/jwe-decrypt/)（`jwe-decrypt`）
  * `jwe-decrypt`エラーメッセージのタイプミスを修正しました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）
  * `ngx.var.request_uri`ではなく`kong.request.get_forwarded_path()`の正規化バージョンを使用することで、ログアウトのためにクエリ文字列を渡す場合を筆頭に、ログアウトURIサフィックスの検出の問題を修正しました。
  * トークンの有効期限を計算する際の時間を更新しました。

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）
  * ペイロードが`client_body_buffer_size`を超えた場合にリクエストペイロードが破棄される問題を修正しました。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）
  * 有効な再帰スキーマが常に拒否される問題を修正しました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * AnyTypeスキーマと定義済みスタイルキーワードでパラメータを検証する間に、プラグインがランタイムエラーをスローする問題を修正しました。
  * null許容のキーワードが有効にならない問題を修正しました。
  * URIコンポーネントのエスケープ文字が誤ってエスケープ解除される問題を修正しました。
  * Refパラメータスキーマが逆参照されていないために、プラグインがランタイムエラーをスローするバグを修正しました。[\#7544](https://github.com/Kong/kong/issues/7544)

* [**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）
  * すべてのカウンターが同じレートで同じDBに同期される問題を修正しました。[\#7315](https://github.com/Kong/kong/issues/7315)

#### Admin API

* 同じRBACユーザーで、`user_token`を同じ値で更新しようとした場合に固有の違反エラーが報告される問題を修正しました。

#### クラスタリング

* ハイブリッドモードで、データプレーンのログシリアライザー出力にワークスペース名が表示される問題を修正しました。

#### デフォルト

* 外部プラグインサーバーを起動する際の重大レベルのログを修正しました。OpenRestyの制限により、これらのログを抑制することはできません。ソケット可用性の検出機能を削除することにしました。

#### 構成

* カスタム`proxy_access_log`を遵守します。[\#7435](https://github.com/Kong/kong/issues/7435)

### パフォーマンス

#### 構成

* `dns_stale_ttl`のデフォルトを1時間に変更しました。これにより、リゾルバにダウンタイムが発生した場合でも、古いDNSレコードをより長い時間使用できるようになります。

### 依存関係

* `OpenResty`を1\.21\.4\.2から1\.21\.4\.3に更新しました [\#7518](https://github.com/Kong/kong/issues/7518)
* `resty-openssl`を0\.8\.25から1\.0\.2に更新しました [\#7418](https://github.com/Kong/kong/issues/7418)
* `luasec`を1\.3\.2に更新しました

3\.5\.0\.1
-------------

**リリース日** 2023/11/14

### 修正

#### Kong Manager

* 構成カードの一部の値が正しく表示されない問題を修正しました。

3\.5\.0\.0
-------------

**リリース日** 2023/11/08

### 下位互換性のない変更と非推奨

* [**Session**](/hub/kong-inc/session/)プラグイン：新しい構成フィールド、`read_body_for_logout`（デフォルト値は`false`）が導入されました。
  この変更により、`read_body_for_logout`が明示的に`true`に設定されない限り、`logout_post_arg`の動作は考慮されなくなります。
  変更後はPOSTリクエストを中心に、Sessionプラグインがログアウト検出のためにリクエストボディを自動的に読み取ることはできなくなります。

* このリリース以降、Kong Enterprise Portal（開発者ポータル）と呼ばれる製品コンポーネントは、Kong Gateway Enterprise（旧称Kong Enterprise）ソフトウェアパッケージに含まれなくなりました。Kong Enterprise Portalを購入した既存のお客様は、引き続きご利用が可能で、専用のメカニズムを通じてサポートをお受けいただけます。

  過去にKong Enterprise Portalを購入し、このリリースまたはKong Gateway Enterpriseの将来のリリースでも引き続き使用したい場合は、[Kongサポート](https://support.konghq.com/support/s/)に詳細をお問い合わせください。
* このリリースで、Vitalsと呼ばれる製品コンポーネントは非推奨となり、Kong Gateway Enterpriseに含まれなくなりました。
  Kong Konnectユーザーは、Vitalsの機能のスーパーセットを提供する[API分析](/konnect/analytics/)サービスを活用できます。

  Vitalsは、Kong Enterprise 3\.4 LTSリリースを通じて、既存のお客様に対し2026年8月まで引き続きサポートされます。
* [`dns_no_sync`](/gateway/3.5.x/reference/configuration/#dns_no_sync)オプションのデフォルト値が`on`に変更されました。[\#11871](https://github.com/kong/kong/pull/11871)。

* Kong Gatewayでは、動的なプラグイン順序付けを使用するにはEnterpriseライセンスが必要になりました。

### 機能

#### Enterprise

* 現在のAWS Vaultバックエンドが`CredentialProviderChain`をサポートするように変更し、`AK-SK`環境変数を使用してIAMロールの権限を付与しないことをユーザーが選択できるようになりました。
* Microsoft AzureのKeyVault Secrets Engineのサポートを追加しました。[`vault_azure_*`](/gateway/3.5.x/reference/configuration/#vault_azure_vault_uri)構成パラメータを使用して設定します。
* ライセンス管理: 
  * Kong Enterpriseライセンスの有効期限から30日間続く新しい猶予期間を実装しました。 猶予期間中は、すべてのオープンソース機能が利用可能になり、Enterprise機能は読み取り専用モードに設定されます。
  * ルート、プラグイン、ライセンス、デプロイ情報などのカウンターのサポートをライセンスレポートに追加しました。
  * ライセンスエンドポイントの出力にチェックサムを追加しました。

* Kong Enterpriseパッケージは、Kong Gateway Enterpriseに改名されました。 この変更はドキュメントにのみ影響し、Kong Gatewayコードには一切影響しません。

##### Kong Manager

* ワークスペースとそれに関連するすべてのリソースを削除する機能を追加しました。 以前は、ワークスペースに関連付けられているすべてのエンティティを手動で削除するまで、ワークスペースを削除できませんでした。 強制削除では、ワークスペースを削除しているときに、ワークスペースに関連付けられているすべてのエンティティを自動的に削除できます。 詳細については、[ワークスペースの削除](/gateway/3.5.x/kong-manager/workspaces/#delete-a-workspace)を参照してください。
* AzureのKeyVault Secrets Engineのサポートを追加しました。
* プラグインをコンシューマグループにスコープ設定できるようになりました。
* コンシューマグループポリシーの削除を実装しました。
* 洗練された外観と操作性により、エンティティの詳細ページのユーザーエクスペリエンスが向上しました。
* **概要** ページと **ワークスペース** ページの新しいデザインにより、ユーザーエクスペリエンスが向上しました。
* VaultフォームでTTLフィールドがサポートされるようになりました。

#### コア

* ログに記録されたリクエストの出力に[`analytics_debug`](/gateway/3.5.x/reference/configuration/#analytics_debug)オプションを追加しました。

* [`cluster_fallback_export_s3_config`](/gateway/3.5.x/reference/configuration/#cluster_fallback_export_s3_config)オプションを追加して、Kongエクスポーター構成S3の`putObject`リクエストに構成テーブルを追加できるようにしました。

* コンテナイメージにトラブルシューティングツールを追加しました。

* `workspaces.get_workspace()` データベースに直接クエリするのではなく、キャッシュからワークスペースを取得するようになりました。

* Vaultのスキーマを取得するための新しいエンドポイント[`/schemas/vaults/:name`](/gateway/api/admin-ee/latest/#/Information/get-schemas-vaults-vault_name)を導入しました。
  [\#11727](https://github.com/Kong/kong/pull/11727)

* `privileged_agent`の名前を[`dedicated_config_processing`](/gateway/3.5.x/reference/configuration/#dedicated_config_processing)に変更し、デフォルトで`dedicated_config_processing`を有効にしました。
  [\#11784](https://github.com/Kong/kong/pull/11784)

* デバッグツール:

  * 一意のリクエストIDが追加されました。このIDは、エラーログ、アクセスログ、エラーテンプレート、ログシリアライザー、および新しい`X-Kong-Request-Id`ヘッダーに入力されます。 この構成は、[`headers`](/gateway/3.5.x/reference/configuration/#headers)および[`headers_upstream`](/gateway/3.5.x/reference/configuration/#headers_upstream)構成オプションを使用して、アップストリームとダウンストリームに対してカスタマイズできます。[\#11663](https://github.com/Kong/kong/pull/11663)
  * デバッグリクエストヘッダー`X-Kong-Request-Debug-Output`のサポートを追加しました。 これにより、特定のリクエストで特定のコンポーネントが消費した時間を確認できます。 有効にするには、[`request_debug`](/gateway/3.5.x/reference/configuration/#request_debug)構成パラメータを使用します。 このヘッダーは、Kong Gatewayのレイテンシの原因を診断するのに役立ちます。詳細については、[リクエストのデバッグ](/gateway/3.5.x/production/debug-request/)ガイドを参照してください。 [\#11627](https://github.com/Kong/kong/pull/11627)

* プラグインのエンティティが変更されたときに呼び出される`Plugin:configure(configs)`関数をプラグインに実装できるようにしました。これは、現在のプラグイン構成の配列を受け取ります。アクティブな構成がない場合はnilを受け取ります。
  この関数についての詳細は、プラグインの[カスタムロジックの実装](/gateway/3.5.x/plugin-development/custom-logic/)を参照してください。
  [\#11703](https://github.com/Kong/kong/pull/11703)

* 異なるリクエストからのアクセスを検出できるリクエスト対応テーブルを実装しました。
  [\#11017](https://github.com/Kong/kong/pull/11017)

* WebAssembly（Wasm）:

  * オプションのWasmフィルター構成スキーマのサポートを追加しました。 [\#11568](https://github.com/Kong/kong/pull/11568)
  * Wasmフィルター構成でJSONのサポートが改善されました。 [\#11697](https://github.com/Kong/kong/pull/11697)

  詳細については、[Proxy\-Wasmフィルターの構成](/gateway/3.5.x/plugin-development/wasm/filter-configuration/)ガイドを参照してください。

#### Kong Managerオープンソース

* エンティティ構成カードに`JSON`および`YAML`形式を追加しました。 [\#111](https://github.com/Kong/kong-manager/pull/111)
* プラグインのフォームフィールドに、バックエンドスキーマからの説明が表示されるようになりました。 [\#66](https://github.com/kong/kong-manager/pull/66)
* プラグインフォームに`protocols`フィールドを追加しました。[\#93](https://github.com/kong/kong-manager/pull/93)
* 特定の条件が満たされた場合に、アップストリームターゲットリストには`Mark Healthy`と`Mark Unhealthy`のアクション項目が表示されるようになりました。[\#86](https://github.com/kong/kong-manager/pull/86)

#### プラグイン

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * パス一致評価のための新しいプロパティ`include_base_path`を追加しました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）

  * パス一致評価のための新しいプロパティ`include_base_path`を追加しました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 新しいフィールド、`unauthorized_destroy_session`を追加しました。 `true`に設定すると、不正なリクエストを受信したときにユーザーのセッションCookieを削除してセッションを破棄します。 
  * 新しいフィールド、`using_pseudo_issuer`を追加しました。 `true`に設定すると、プラグインインスタンスは発行者から構成を検出しません。
  * トークンの失効とイントロスペクションのための公開クライアントのサポートを追加しました。
  * パラメータ名`introspection_token_param_name`および`revocation_token_param_name`を指定するためのサポートを追加しました。
  * mTLSの所持証明のサポートを追加しました。この機能は、`proof_of_possession_mtls`を有効にすることで使用できます。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)

  * パラメータ`header_type`に新しい値を追加しました。これによりKong Gatewayは、アップストリームに転送するリクエストのヘッダーに、Datadogヘッダーを注入できるようになりました。

* [**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）

  * Redis接続でのシークレットローテーションのサポートを追加しました。 [\#10570](https://github.com/Kong/kong/pull/10570)

* [**CORS**](/hub/kong-inc/cors)（`cors`）

  * クロスオリジンのプリフライトリクエストでの`Access-Control-Request-Private-Network`ヘッダーのサポートを追加しました。[\#11523](https://github.com/kong/kong/pull/11523)。

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * Redisストレージ用の新しい構成フィールド`scan_count`を公開しました。これは、`scan`呼び出しで返されるキーの数を制御します。 [\#11532](https://github.com/kong/kong/pull/11532)

* [**Session**](/hub/kong-inc/session/)（`session`）

  * デフォルト値が`false`の新しい構成フィールド`read_body_for_logout`が導入されました。 この変更により`logout_post_arg`の動作が変更され、`read_body_for_logout`が明示的に`true`に設定されていないかぎり、考慮されなくなります。

  変更後はPOSTリクエストを中心に、Sessionプラグインがログアウト検出のためにリクエストボディを自動的に読み取ることはできなくなります。

### 修正

#### Enterprise

* クラスタストラテジを使用しているときにKongノードがキーリングマテリアルを送信できないというキーリングの問題を修正しました。
* Kong Managerを介して静的なリソースを提供するための強制コンテンツセキュリティポリシー（CSP）ヘッダーが適用されました。
* 数値のグループ名タイプを持つグループロールの取得に関連するRBACの問題を修正しました。
* Kong Managerの`admin_gui_auth`メソッドとして`openid-connect`を使用する場合、一部の`admin_gui_auth_conf`必須設定がハードコードされるようになりました。
* Kong Gatewayをハイブリッドモードで実行しているときに、Vitalsでデータプレーンのホスト名が`nil`になる問題を修正しました。

##### Admin API

* エンティティを削除しても、カスケードエンティティの`rbac_role_entities`レコードが削除されない問題を修正しました。
* Admin APIでコンテンツタイプとして`application/x-www-form-urlencoded`を使用すると、異なるワークスペースで競合するルートが作成される問題を修正しました。
* `application_services`および`application_instances`エンドポイントにアクセスする際に、プラグインをクエリするパフォーマンスを最適化しました。
* ユーザーが、Admin APIを介したメールによって開発者を完全に削除できない問題を修正しました。
* `validate_fips`にFIPSの状態とライセンスタイプのチェックを追加しました。
* FIPSを無料モードから削除しました。
* 有効なライセンスを受け取ったときにFIPSモードを遅延して有効化する機能を実装し、Kong Gatewayの起動をブロックするのではなく、警告を発するようにしました。このアプローチにより、ライセンスなしで非FIPSコンテンツを通常どおり使用できるようになります。また、FIPSモードは有効なライセンスがある場合にのみアクティブになります。ライセンスが存在しない場合は、警告ログでサービスが開始され、有効なライセンスが追加されるまでFIPSモードは無効のままになります。さらに、Admin API経由で有効なライセンスを削除すると、FIPSモードが無効にならずに警告が表示されます。
* Admin APIおよびポータルAPI経由で管理者認証に失敗した場合のエラー応答を統合しました。

##### Kong Manager

* 管理者が追加されていない場合に管理者ページが保留のままになる問題を解決しました。
* アプリケーションリスト内のサービス名が、バックエンドから直接返されるように更新されました。
* サイドバーで1つのメニュー項目を共有するエンティティのブレッドクラムとRBAC権限を修正しました。
* ルートフォームのサービスクエリエンドポイントを修正しました。
* サービスドキュメントフォームでアップロードファイルに入力する内容が適切に機能しない問題を修正しました。

#### コア

* Prometheusの重要なメトリクスではないチャート`Current Database Availability`を削除しました。
* コンシューマグループの名前とIDの両方に基づくキャッシュ無効化を実装しました。
* HTTP/2ストリームリセット攻撃を早期に検出するためのNginxパッチを適用し、[CVE\-2023\-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)に対処しました。
* DB\-lessモードおよびハイブリッドモードでKey AuthenticationプラグインのTTLが機能しない問題を解決しました。[\#11464](https://github.com/kong/kong/pull/11464)
* PostgreSQLデータベースをクエリするときに異常なソケット接続が再利用される問題に対処しました。[\#11480](https://github.com/kong/kong/pull/11480)
* プラグインが応答ハンドラを使用したときにアップストリームのSSL障害を引き起こす問題を修正しました。 [\#11502](https://github.com/Kong/kong/issues/11502)
* `tls_passthrough`プロトコルが式フレーバールーターで動作しない問題を修正しました。 [\#11538](https://github.com/kong/kong/issues/11538)
* リクエストの`x-datadog-parent-id`ヘッダーの値が短い10進文字列の場合に、トレーシングデータをDatadogに送信できない問題を修正しました。 [\#11599](https://github.com/kong/kong/issues/11599)
* パッチ適用時にビルドが失敗する問題を解決しました。[\#11696](https://github.com/kong/kong/issues/11696)
* 宣言型構成ファイルがDBレスモードのときにVault参照を使用できるようになりました。 [\#11845](https://github.com/kong/kong/issues/11845)
* 初期化中に、Vaultキャッシュが適切にウォームアップされるようになりました。 [\#11827](https://github.com/kong/kong/issues/11827)
* VaultシークレットがVaultから削除された場合に、Vaultの復活時間が遵守されるようになりました。 [\#11852](https://github.com/kong/kong/issues/11852)
* `lapis` binおよび`luarocks-admin` binを復元しました。[\#11551](https://github.com/Kong/kong/pull/11551)

#### Kong Managerオープンソース

* Kong Managerに誤ったポート情報が表示される問題を解決しました。 [\#103](https://github.com/kong/kong-manager/pull/103)。
* Kong ManagerにProxy Cacheプラグインをインストールできないバグを修正しました。 [\#104](https://github.com/kong/kong-manager/pull/104)

#### プラグイン

* プラグインを実装するための新しいハンドラを追加しました。アクティブなプラグインの構成がない場合、構成は`nil`になります。この変更は、Acme、Prometheus、Rate Limiting Advancedプラグインで確認できます。
* Kong Gatewayでは、動的なプラグイン順序付けを使用するのにライセンスが必要になりました。
* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）
  * 失効チェック中のキャッシュネットワーク障害を防ぐために問題を修正しました。

* [**Response Transformer**](/hub/kong-inc/response-transformer/)（`response-transformer`）
  * フラッディングされたJSONデコードの問題に関連する警告ログを解決しました。

* [**カナリア**](/hub/kong-inc/canary)（`canary`） 
  * `config.start`のカスタム検証を削除して、過去の時刻に設定できるようにしました。

* [**SAML**](/hub/kong-inc/saml)（`saml`）
  * Redisセッションストレージが正しく構成されていない場合、ユーザーは、無限にリダイレクトされるのではなく、500エラーを受け取るようになりました。
  * `session was not found`メッセージの重大度を`info`に下げました。

* [**Mocking**](/hub/kong-inc/mocking)（`mocking`）
  * パスパラメータが非ASCII文字と正しく一致するようになりました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * リクエストボディが必須でない場合でも、`application/json`以外のコンテンツタイプが拒否される問題を修正しました。
  * `notify_only_request_validation_failure`がtrueに設定されているときに、一定のシナリオでヌルポインタの例外が発生していた問題を修正しました。 
  * パスのパラメータが非ASCII文字と一致しない問題を修正しました。
  * 有効な再帰スキーマが常に拒否される問題を修正しました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry)（`opentelemetry`）
  * `balancer`インストルメンテーションが有効になっていると、トレースで親IDが無効になる問題を修正しました。 [\#11830](https://github.com/Kong/kong/pull/11830)

* [TCP Log](/hub/kong-inc/tcp-log)（`tcp-log`）
  * TLS接続を再利用する際の不要なハンドシェイクに関連する問題を解決しました。 [\#11848](https://github.com/Kong/kong/pull/11848)

* [**AWS Lambda**](/hub/kong-inc/aws-lambda)（`aws-lambda`）
  * EKS環境でIRSAを用いてIAM認証情報を取得する際、プラグインレベルのプロキシ構成が有効になりました。 この改善により、EKS IRSA認証情報プロバイダー（`TokenFileWebIdentityCredentials`）は、AWS STSサービスから認証情報を取得するときに、プラグインレベルのプロキシ構成を介してリクエストを正しくルーティングできるようになります。 [\#11551](https://github.com/Kong/kong/pull/11551)
  * プラグインは、Lambdaサービス関連のフィールドによってAWS Lambdaサービスをキャッシュするようになりました。 [\#11821](https://github.com/kong/kong/pulls/11821)

#### PDK

* Vaultのいくつかの問題に対処し、Vaultコードベースをリファクタリングしました。
* リクエストのライフサイクルで`kong.response.get_raw_body()`が複数回呼び出されるときに、応答本文が繰り返される問題を修正しました。 [\#11424](https://github.com/Kong/kong/pull/11424)
* トレーシング: タイムスタンプの精度が異なるために、一部の親スパンが子スパンよりも先に終了する問題を修正しました。 [\#11484](https://github.com/Kong/kong/pull/11484)
* `kong.log.serialize`関数内のリクエスト間のデータ干渉に関連するバグを修正しました。 [\#11566](https://github.com/Kong/kong/issues/11566)

### 依存関係

* `resty.openssl`を0\.8\.23から0\.8\.25にアップグレードしました [＃11518](https://github.com/Kong/kong/issues/11518)
* ARM64上のIR\_\*LOADのLuaJIT登録割り当てが誤っている問題を修正しました [\#11638](https://github.com/Kong/kong/issues/11638)
* ARM64での非整列アクセスに対するLDP/STP融合を修正しました [\#11639](https://github.com/Kong/kong/issues/11639)
* lua\-kong\-nginx\-moduleを0\.6\.0から0\.8\.0に更新しました [\#11663](https://github.com/Kong/kong/issues/11663)
* ARM64での誤ったLuaJIT LDP/STP融合を修正しました [\#11537](https://github.com/Kong/kong/issues/11537)
* `lua-resty-healthcheck`を1\.6\.2から1\.6\.3に更新しました [\#11360](https://github.com/Kong/kong/issues/11360)
* `openresty`を1\.21\.4\.1から1\.21\.4\.2に更新しました [\#11360](https://github.com/Kong/kong/issues/11360)
* `lua-resty-aws`を1\.3\.1から1\.3\.5に更新しました。 [\#11551](https://github.com/Kong/kong/issues/11551) [\#11613](https://github.com/Kong/kong/issues/11613)
* `wasmtime`バージョンを8\.0\.1から12\.0\.2に更新しました。 [\#11738](https://github.com/Kong/kong/issues/11738)
* `openssl`を3\.1\.2から3\.1\.4に更新しました [\#11844](https://github.com/Kong/kong/issues/11844)
* `kong-lapis`を1\.14\.0\.2から1\.14\.0\.3に更新しました [\#11849](https://github.com/Kong/kong/issues/11849)
* OpenID Connectプラグインサブモジュール`kong-openid-connect`を2\.5\.5から2\.5\.9にアップグレードしました
* Kong CLIの依存関係:
  * `curl`を8\.3\.0から8\.4\.0に更新しました
  * `nghttp2`を1\.56\.0から1\.57\.0にアップグレードしました

3\.4\.3\.12
--------------

### 機能

**リリース日** 2024/08/08

### 非推奨

* Debian 10、CentOS 7、RHEL 7は、2024年6月30日にサポート終了（EOL）となりました。このパッチの時点では、Kongはこれらのオペレーティングシステム用のKong Gateway 3\.7\.xインストールパッケージまたはDockerイメージを構築していません。Kongは、これらのシステムで実行されるKongバージョンに対して公式サポートを提供しなくなりました。

### 機能

#### コア

*3\.6\.0\.0からバックポート* 

* Kong Gateway Enterpriseコンテナイメージは、ビルドの来歴とともに生成され、Cosignを使用して署名されるようになりました。 署名と来歴証明はDocker Hubリポジトリに公開されます。 ビルドの来歴は、公開されている来歴証明を使用して[cosign/slsa\-verifierによって検証](/gateway/3.4.x/kong-enterprise/provenance-verification/)できます。

### 修正

#### コア

* `kong.logrotate`構成ファイルは、アップグレード中に上書きされなくなりました。

  この変更により、`apt`および`deb`パッケージを介してアップグレードするDebianユーザーに追加のプロンプトが表示されます。
  Kongのパッケージで提供されているデフォルトを受け入れるには、次のコマンドをアーキテクチャとアップグレードするバージョンに調整したうえで使用してください。

  ```sh
  DEBIAN_FRONTEND=noninteractive apt upgrade kong-enterprise-edition_3.4.3.11_arm64.deb
  ```

*3\.7\.0\.0からバックポート* 

* 最初の構成プッシュ後に、新しいデータプレーンがVaultリファレンスを解決できなかった問題を修正しました。これは、ライセンスのプリロードの問題が原因で発生していました。

#### プラグイン

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * コンシューマグループのオーバーライド構成の`window_size`が、プラグインのデフォルト構成の`window_size`と異なる場合に発生する問題を修正しました。そのコンシューマグループの流量制限は、ローカルストラテジにフォールバックします。

*3\.7\.0\.0からバックポート* 

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`） 
  * LDAP検索が失敗した場合に例外がスローされる問題を修正しました。

3\.4\.3\.11
--------------

**リリース日** 2024/06/22

### 修正

* DNSクライアントがDNS応答で`ADDITIONAL SECTION`のコンテンツを誤って使用していた問題を修正しました。

3\.4\.3\.10
--------------

**リリース日** 2024/06/18

### 既知の問題

* DNSクライアントがDNS応答でコンテンツ`ADDITIONAL SECTION`を誤って使用するという修正に関する問題がありました。 この問題を回避するには、このパッチの代わりに、3\.4\.3\.11をインストールしてください。

### 修正

#### Admin API

*3\.7\.0\.0からバックポート* 

* `/<workspace id="sl-md0000000">/admins`エンドポイントが誤って使用され、割り当てられたRBACのロールに基づいてワークスペースに関連付けられている管理者を返していました。この問題は修正され、特定のワークスペースの関連付けに応じて管理者が正確に返されるようになりました。

### 依存関係

* `lua-resty-events`0\.2\.1にアップグレードしました。

3\.4\.3\.9
-------------

**リリース日** 2024/06/08

### 機能

#### Admin API

*3\.7\.0\.0からバックポート* 

* 検索フィールドにLHSブラケットのフィルタリングを追加しました。
* **監査ログ:** 
  * `request_timestamp`を`audit_objects`に追加しました。
  * LHS Bracketsフィルターの前後にエイリアスを追加しました。
  * `audit_requests`と`audit_objects`は`request_timestamp`でフィルタリングできるようになりました。

### 修正

#### Admin API

*3\.6\.0\.0からバックポート* 

* ユーザーがロールを持っていないワークスペースを表示するワークスペースリスティングAPIの問題を修正しました。 APIには、ユーザーがアクセスできるワークスペースのみが表示されるようになりました。

#### コア

* `cluster_cert`または`cluster_ca_cert`が、base64でデコードされる前に`lua_ssl_trusted_certificate`に挿入されていた問題を修正しました。
* **Vitals** : コントロールプレーンに接続する各データプレーンが、コントロールプレーン上で冗長なテーブルローテータータイマーの作成をトリガーする問題を修正しました。

*3\.7\.1\.0からバックポート* 

* **DNSクライアント** : Kong DNSクライアントが回答を解析する際に、ドメインとタイプが一致しないレコードを保存する問題を修正しました。 回答を解析するときに、RRタイプがクエリのタイプと異なる場合はレコードが無視されるようになりました。
* 接続の再試行中に、アップストリームエンティティの`host_header`属性がアップストリームへのリクエストでホストヘッダーとして正しく設定されない問題を修正しました。
* 管理者用の組み込みRBACロール（デフォルトのワークスペースでは`admin`、デフォルト以外のワークスペースでは`workspace-admin`）で、`/groups`および`/groups/*`エンドポイントへのCRUDアクションが禁止されるようになりました。

#### プラグイン

*3\.7\.1\.0からバックポート* 

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）
  * 特定の条件下で、匿名コンシューマが`nil`としてキャッシュされる問題を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * 中央のデータストアでネットワークが不安定になっても、タイマーのスパイクが発生しなくなりました。

### 依存関係

* エラーログを改良するために、`lua-resty-azure`を1\.4\.1から1\.5\.0に更新しました。
* 多数のタイマーを実行する代わりに、同じアクティブなヘルスチェックターゲットに対してタイマーを再利用することによってメモリリークの問題を修正するために、`lua-resty-healthcheck`を1\.6\.4から1\.6\.5に更新しました。

3\.4\.3\.8
-------------

**リリース日** 2024/05/16

### 機能

#### Admin API

*3\.7\.0\.0からバックポート* 

* `audit_requests`のデフォルトの順序を、`request_timestamp`の降順で並べ替えるように変更しました。

### 修正

#### Admin API

*3\.6\.0\.0からバックポート* 

* タイムスタンプフィールドによるページ分割やソートを行うとHTTP 500エラーが返される問題を修正しました（`created_at`）など。

#### プラグイン

*3\.6\.1\.4からバックポート* 

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）、
  [**WebSocket Size Limit**](/hub/kong-inc/websocket-size-limit/)（`websocket-size-limit`）、
  [**WebSocket Validator**](/hub/kong-inc/websocket-validator/)（`websocket-validator`）、
  [**XML Threat Protection**](/hub/kong-inc/xml-threat-protection/)（`xml-threat-protection`）

  * プラグイン間の衝突を防ぐために、これらのプラグインの優先順位が更新されました。 バンドルされたプラグインの相対的な優先順位（および実行順序）は変更されません。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `kong/tools/public/rate-limiting`をリファクタリングし、異なるプラグインを分離するための新しいインターフェース`new_instance`を追加しました。
    後方互換性のため、元のインターフェースは変更されていません。

    このライブラリをベースにしたカスタムのRate Limitingプラグインを使用している場合は、初期化コードを新しい形式に更新してください。たとえば、`'local ratelimiting = require("kong.tools.public.rate-limiting").new_instance("custom-plugin-name")'`などです。
    古いインターフェースは、今後のメジャーリリースで削除されます。

### 依存関係

* 予期しない入力を処理する際の`lua-cjson`のロバスト性を改善しました。
* TCPソケットキープアライブをサポートするために、`kong-lua-resty-kafka`を 0\.19にアップグレードしました。

3\.4\.3\.7
-------------

**リリース日** 2024/04/23

### 機能

#### プラグイン

*3\.6\.1\.4からバックポート* 

* [**Portal Application Registration**](/hub/kong-inc/application-registration/)（`application-registration`） 
  * 認証情報によるコンシューマ認証を使用したサービスにアクセスできるようになりました。 この機能を使用するには、`enable_proxy_with_consumer_credential`を有効にします（デフォルトは`false`）。

### 修正

#### クラスタリング

*3\.6\.1\.4からバックポート* 

* ハイブリッドモードで、イベントフックが早期に検証される問題を修正しました。 この修正により、イベントフックの検証は、イベントフックが発行される時点まで遅延されます。

#### コア

*3\.7\.0\.0からバックポート* 

* ハイブリッドモードのデータプレーンで、Vaultリファレンスで構成されている証明書エンティティが時間通りに更新されないことがあった問題を修正しました。 

#### PDK

*3\.6\.1\.4からバックポート* 

* `kong.request.get_forwarded_port`が`ngx.ctx.host_port`から誤った文字列を返す問題を修正しました。数値が正しく返されるようになりました。

### 依存関係

* `lua-protobuf`を0\.5\.1にアップグレードしました。

3\.4\.3\.6
-------------

**リリース日** 2024/04/15

### 機能

#### Kong Manager Enterprise

*3\.5\.0\.0からバックポート* 

* Microsoft AzureのKeyVault Secrets Engineのサポートを追加しました。

#### プラグイン

*3\.6\.1\.2からバックポート* 

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * `api_spec`がURIエンコードされているかどうかを示す新しいフィールド`api_spec_encoded`が追加されました。

### 修正

#### 構成

*3\.7\.0\.0からバックポート* 

* 外部プラグイン（Go、Javascript、Python）が、Admin APIを介してプラグイン構成に変更を適用する際に失敗する問題を修正しました。

#### Kong Manager Enterprise

* 開発者ポータル構成で、 **開発者メタフィールド** タブにLatin1の範囲外の文字が含まれている場合にログインが失敗する問題を修正しました。

*3\.6\.1\.3からバックポート* 

* `admin_gui_path`がスラッシュでない場合に、管理者アカウントプロファイルページで404エラーが返される問題を修正しました。

#### プラグイン

*3\.6\.1\.2からバックポート* 

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * ACME更新中に証明書が正常に更新されない問題を修正しました。 

* [**DeGraphQL**](/hub/kong-inc/degraphql/)（`degraphql`）

  * GraphQL変数が正しく解析されず、定義された型に強制変換されない問題を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `rate-limiting`ライブラリを使用するプラグインを一緒に使用すると、互いに干渉し合い、カウンターデータを中央のデータストアに同期できなくなる問題を修正しました。

*3\.6\.1\.3からバックポート* 

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry)（`opentelemetry`）
  * 短いトレースIDの解析のロバスト性が向上しました。

#### プラグイン

### 依存関係

*3\.6\.1\.2からバックポート* 

* `lua-kong-nginx-module`を0\.8\.1に更新しました
* `lua-resty-luasocket`を1\.1\.2に更新し、[luasocket\#427](https://github.com/lunarmodules/luasocket/issues/427)を修正しました

3\.4\.3\.5
-------------

**リリース日** 2024/03/21

### 最新の変更

* OpenSSL 3\.2では、デフォルトのSSL/TLSセキュリティレベルが1から2に変更されました。これは、セキュリティレベルが112ビットのセキュリティに設定されていることを意味します。その結果、以下のことが禁止されます。

  * 2048ビット未満のRSAキー、DSAキー、DHキー
  * 224ビットより短いECCキー
  * RC4を使用する任意の暗号スイート
  * SSLバージョン3。また、圧縮は無効になります。

* 最近のOpenRestyの更新にはTLS 1\.3が含まれており、TLS 1\.1は非推奨になっています。
  引き続きTLS 1\.1をサポートする必要がある場合は、[`ssl_cipher_suite`](/gateway/3.4.x/reference/configuration/#ssl_cipher_suite)設定を`old`に設定します。

### 機能

#### 構成

* Microsoft AzureのKeyVault Secrets Engineのサポートを追加しました。[`vault_azure_*`](/gateway/3.4.x/reference/configuration/#vault_azure_vault_uri)構成パラメータを使用して設定します。 
* OpenSSL 3\.xでは、TLSv1\.1およびそれ以下はデフォルトで無効になっています。 

#### コア

* 式ルーターは`! (not)`演算子をサポートするようになりました。これにより、`!(http.path =^ "/a")`および`!(http.path == "/a" || http.path == "/b")`のようなルートを作成できます。
* デバッグリクエストヘッダー`X-Kong-Request-Debug-Output`のサポートを追加しました。 これにより、特定のリクエストで特定のコンポーネントが消費した時間を確認できます。 有効にするには、[`request_debug`](/gateway/3.4.x/reference/configuration/#request_debug)構成パラメータを使用します。 このヘッダーは、Kong Gatewayのレイテンシの原因を診断するのに役立ちます。詳細については、[リクエストのデバッグ](/gateway/3.4.x/production/debug-request/)ガイドを参照してください。 
* Kong Gatewayが、式ルーターで[`http.path.segments.len`フィールドと`http.path.segments.*`フィールド](/gateway/3.4.x/key-concepts/routes/expressions/#matching-fields)をサポートするようになり、個々のセグメントまたはセグメントの範囲によって受信（正規化）リクエストパスをマッチングして、セグメントの合計数を確認できるようになりました。
* 式を使用して定義されたHTTPルートで、[`net.src.*`と`net.dst.*`](/gateway/3.4.x/key-concepts/routes/expressions/#matching-fields)の一致フィールドにアクセスできるようになりました。 
* 現在のAWS Vaultバックエンドが`CredentialProviderChain`をサポートするように変更し、`AK-SK`環境変数を使用してIAMロールの権限を付与しないことをユーザーが選択できるようになりました。
* HashiCorp Vaultのシークレット管理バックエンドは、AppRole認証方法をサポートするようになりました。
* OSS機能は期限切れのライセンスで引き続き機能し、構成済みのKong Enterprise機能は読み取り専用モードで引き続き動作します。Kong Gatewayは、ライセンスの有効期限が切れてから30日間の猶予期間内に、毎日重要なメッセージをログに記録するようになります。
* [Kong Managerでグループマッピング](/gateway/3.4.x/kong-manager/auth/oidc/mapping/)を使用する際に、RBACトークンを使用して認証できるようになりました（OIDCまたはLDAPなど）。
* Vaultのスキーマを取得するための新しいエンドポイント[`/schemas/vaults/:name`](/gateway/api/admin-ee/latest/#/Information/get-schemas-vaults-vault_name)を導入しました。 

#### プラグイン

* プラグインのエンティティが変更されたときに呼び出される`Plugin:configure(configs)`関数をプラグインに実装できるようにしました。これは、現在のプラグイン構成の配列を受け取ります。アクティブな構成がない場合はnilを受け取ります。
  この関数についての詳細は、プラグインの[カスタムロジックの実装](/gateway/3.4.x/plugin-development/custom-logic/)を参照してください。

* [**CORS**](/hub/kong-inc/cors)（`cors`）

  * クロスオリジンのプリフライトリクエストでの`Access-Control-Request-Private-Network`ヘッダーのサポートを追加しました。

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * `default_consumer`オプションを追加しました。これにより、クライアント証明書が有効だが既存のコンシューマに一致しない場合に、デフォルトのコンシューマを使用できるようになります。

### 修正

#### 構成

* `ssl_cipher_suite`が`old`に設定されている場合、gRPCのTLSのセキュリティレベルを`0`に設定します。 
* 欠落している`azure_vault`構成オプションを`kong.conf`ファイルに追加しました。

#### コア

* `expressions`ルーターフレーバーのヘッダー値マッチング（`http.headers.*`）で、大文字と小文字が区別されるようになりました。 この変更は、ヘッダー値マッチングが常に大文字と小文字を無視して実行される`traditional_compatible`モードには影響しません。
* `kong.logrotate`のファイル権限を644に更新しました。
* `http`および`stream`サブシステム内の式ルートの検証がより厳格になりました。 以前これらは同じ検証スキーマを共有していたため、ストリームルートであっても、管理者は`http.path`のようなフィールドを使用して式ルートを設定できていましたが、これは許可されなくなりました。
* エンティティを削除しても、カスケードエンティティの`rbac_role_entities`レコードが削除されない問題を修正しました。
* `cluster_telemetry_endpoint`構成が無効になっている場合のメッセージプッシュエラーログが削減されました。
* Vaults: 接頭辞でVaultエンティティを取得するときに、Vaultが誤った（デフォルトの）ワークスペース識別子を使用する問題を修正しました。 

#### Kong Manager

* ライセンスの有効期限の残り日数の表示を修正しました。
* RBACユーザーの編集中は、ユーザートークンの入力フィールドが非表示になるようになりました。
* グループマッピングに関するいくつかの問題を修正しました。

#### プラグイン

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）

  * リクエストボディがすでに読み込まれている場合、プラグインは非ストリーミングプロキシにフォールバックするようになりました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry)（`opentelemetry`）

  * `http_response_header_for_traceid`オプションが有効になっている場合に発生するOTELサンプリングモードのLuaパニックバグを修正しました。
  * キューの最大バッチサイズを200に増やしました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * `introspection_headers_values`を暗号化された参照可能なフィールドとしてマークしました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `redis`ストラテジで使用される`sync_rate`設定に関する問題を修正しました。 `sync_rate = 0`中にRedis接続が中断された場合、プラグインは正確に`local`ストラテジにフォールバックするようになりました。
  * `sync_rate`が`0`より大きい値から`0`に変更された場合、名前空間が予期せずクリアされる問題を修正しました。
  * カウンター同期タイマーが適切に作成または破棄されないというタイマー関連の問題を修正しました。
  * プラグイン作成時ではなく、プラグイン実行中にカウンター同期タイマーを作成するようになったことで、無意味なエラーログを減らせるようになりました。

* [**JWT Signer**](/hub/kong-inc/jwt-signer/)（`jwt-signer`）、[**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）、[**OAuth 2\.0 Introspection**](/hub/kong-inc/oauth2-introspection/)（`oauth2-introspection`）、[**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）、および[**SAML**](/hub/kong-inc/saml)（`saml`）

  * PDK `kong.client.authenticate`関数を使用したコンシューマグループのスコープ設定のサポートを追加しました。

### パフォーマンス

#### 構成

* `nginx_http_keepalive_requests`と`upstream_keepalive_max_requests`のデフォルト値を10000に更新しました。

#### コア

* 頻繁なメモリ割り当てや割り当て解除を避けるために、リクエスト間でマッチコンテキストを再利用します。

### 依存関係

* `atc-router`を1\.2\.0から1\.6\.0に更新しました
* `lua-resty-openssl`を1\.2\.0から1\.2\.1にアップグレードしました
* `kong-lua-resty-kafka`を0\.17から0\.18に更新しました

3\.4\.3\.4
-------------

**リリース日** 2024/02/10

### 機能

#### コア

* KubernetesでHashiCorp Vaultを使用する際の名前空間認証とユーザー定義の認証パスのサポートを追加しました。

#### クラスタリング

* 同種のデータプレーン（DP）デプロイメントに対するレジリエンスサポートが追加されました。データプレーンは、インポーターとエクスポーターとして同時に機能できるようになりました。今後、Kong Gatewayは、構成のエクスポート時に同時実行の制御を試みるようになります。 

### 修正

#### コア

* ワークロードIDが、データプレーンのレジリエンスに対して機能しない問題を修正しました。
* シークレットを取得できなかったときに、GCPバックエンドVaultがエラーメッセージを非表示にする問題を修正しました。
* リクエストがアップストリームにプロキシされていない場合、スパンが`http.status_code`で計測されない問題を修正しました。

#### 構成

* `declarative_config_flattened`関数内の弱く型付けされた`of`関数によってデータ損失エラーが引き起こされる問題を修正しました。 

#### プラグイン

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`） 
  * 200以外の応答後に`groups_required`が予期しないコードを返す原因となっていたキャッシュ関連の問題を修正しました。
  * 認証情報がユーザー名なしでエンコードされている場合に、Kong Gatewayが500エラーコードを返す問題を修正しました。

### 依存関係

* OpenSSLを3\.1\.4から3\.2\.1に更新しました [\#7762](https://github.com/Kong/kong/issues/7762)
* `resty-openssl`を0\.8\.25から1\.2\.0に更新しました [\#7741](https://github.com/Kong/kong/issues/7741)
* `lua-resty-aws`を1\.3\.5から1\.3\.6に更新しました

3\.4\.3\.3
-------------

**リリース日** 2024/01/17

### 機能

#### コア

* Kong Gateway DockerイメージのDebianバリアントは、Debian 12を使用して構築されるようになりました。 [\#7672](https://github.com/Kong/kong/issues/7672)

#### Admin API

* Admin APIのルートエンドポイントにKong Gatewayエディションを追加しました。 [\#7674](https://github.com/Kong/kong/issues/7674)

#### プラグイン

* **[AppDynamics](/hub/kong-inc/app-dynamics/)** : 自己署名証明書を使用するために、AppDynamicsプラグインに`CONTROLLER_CERTIFICATE_FILE`および`CONTROLLER_CERTIFICATE_DIR`環境変数構成を追加しました。

### 修正

#### ポータル

* 間違ったドメインやプロトコルへのリダイレクトエラーを防止するために、ポータルルートパスのリダイレクト用に相対URLを導入しました。

#### コア

* すべてのワークスペースで欠落しているエンドポイントの追加が必要となっていたRBACの問題を修正しました。

#### プラグイン

* **[OAS\-Validation](/hub/kong-inc/oas-validation/)** : Cookieのパラメータが検証されない問題を修正しました。

#### Admin API

* 管理者またはRBACユーザーは、自分のロールを更新できなくなりました。

#### Kong Manager

* 動的な順序付けドロップダウンリストにカスタムプラグインが表示されない問題を修正しました。

* 現在のワークスペースのロールを、ロール`workspace-super-admin`の管理者が作成できない問題を修正しました。

### 依存関係

* `kong-redis-cluster`を1\.5\.3に更新しました

* 複数のヘルスチェックインスタンスがクリアされないとヘルスチェックモジュールが正しく動作しないというバグを修正するために、`lua-resty-healthcheck`を1\.6\.4に更新しました。

3\.4\.3\.2
-------------

**リリース日** 2023/12/22

### 機能

#### プラグイン

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`） 
  * プラグインは、非標準の`asn1`整数のデコードと、冗長な先頭パディングでエンコードされた列挙型のデコードをサポートするようになりました。

### 修正

#### コア

* `application_services/application_instances`エンドポイントにアクセスする際に、プラグインをクエリするパフォーマンスを最適化しました。

#### Kong Manager

* Kong ManagerのDev Portalのアプリケーションリストに一部のサービスが表示されない問題を修正しました。
* 仕様アップロード入力をクリックしてもファイルの選択がトリガーされない問題を修正しました。

3\.4\.3\.1
-------------

**リリース日** 2023/12/15

### 下位互換性のない変更

#### プラグイン

* [**SAML**](/hub/kong-inc/saml)（`saml`）: SAMLプラグインと他のコンシューマベースのプラグイン間の統合を修正するために、SAMLプラグインの優先度を1010に調整しました。

### 機能

#### コア

* 一意のリクエストIDが、エラーログ、アクセスログ、エラーテンプレート、ログシリアライザー、および新しい X\-Kong\-Request\-Idヘッダー（`headers`および`headers_upstream`構成オプションを使用してアップストリーム/ダウンストリーム用に構成可能）に入力されるようになりました。 [\#7207](https://github.com/Kong/kong/issues/7207)
* [`dns_no_sync`](/gateway/3.4.x/reference/configuration/#dns_no_sync)オプションのデフォルト値が`off`に変更されました。

#### プラグイン

* [**AWS Lambda**](/hub/kong-inc/aws-lambda)（`aws-lambda`）: AWS\-Lambdaプラグインは、基盤となるAWSライブラリとして`lua-resty-aws`を使用し、リファクタリングされました。このリファクタリングにより、AWS\-Lambdaプラグインのコードベースが簡素化され、複数のIAM認証シナリオのサポートが追加されます。
  [\#7079](https://github.com/Kong/kong/issues/7079)

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 構成`scopes`、`login_redirect_uri`、`logout_redirect_uri`をKong Vault内のシークレットとして参照できるようになりました。
  * `token_post_args_client`を拡張して、ヘッダーからの注入をサポートします。

### 修正

#### 構成

* カスタム`proxy_access_log`を遵守します。[\#7436](https://github.com/Kong/kong/issues/7436)

#### コア

* プラグインが失敗したときにエラーメッセージを正しく出力します。 [\#7079](https://github.com/Kong/kong/issues/7079)
* LuaJITエラーによって発生する断続的な`ldoc`障害を修正しました。[\#7491](https://github.com/Kong/kong/issues/7491)
* イールドできないフェーズでセマフォを使用しないように、Vaultのtry関数を修正しました。 [\#7114](https://github.com/Kong/kong/issues/7114)
* 宣言型構成がDBレスモードのときにVault参照を使用できます。 [\#7483](https://github.com/Kong/kong/issues/7483)
* コンシューマグループの名前とIDに基づいてキャッシュを正しく無効化します。
* ハングするリスクを回避するために、syncQuery\(\)の非同期タイマーを削除しました。
* 外部プラグインサーバーを起動する際の重大レベルのログを修正しました。OpenRestyの制限により、これらのログを抑制することはできません。ソケット可用性の検出機能を削除することにしました。

#### Admin API

* 同じRBACユーザーで、user\_tokenを同じ値で更新しようとした場合に固有の違反エラーが報告される問題を修正しました。

#### Kong Manager

* デフォルト以外のワークスペースのサービスで \[アプリケーション\] タブが表示されない問題を修正しました。

#### クラスタリング

* ハイブリッドモードで、データプレーンのログシリアライザー出力にワークスペース名が表示される問題を修正しました。
* ハイブリッドモードの場合に、Vitalsでデータプレーンのホスト名が`nil`になる問題を修正しました。

#### PDK

* `kong.log.serialize`機能で発生していたリクエスト間のデータ干渉に関連するバグを修正しました。 [\#7327](https://github.com/Kong/kong/issues/7327)
* **プラグインサーバー** : リクエストのたびに新しいプラグインインスタンスを作成する問題を修正しました。

#### プラグイン

* [**AWS Lambda**](/hub/kong-inc/aws-lambda)（`aws-lambda`）:

  * これらのLambdaサービス関連のフィールドによって、AWS Lambdaサービスをキャッシュしました。[\#7079](https://github.com/Kong/kong/issues/7079)

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）:

  * ペイロードが`client_body_buffer_size`を超えた場合にリクエストペイロードが破棄される問題を修正しました。

* [**JWE Decrypt**](/hub/kong-inc/jwe-decrypt/)（`jwe-decrypt`）:

  * エラーメッセージのタイプミスを修正しました。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）:

  * パスパラメータが非ASCII文字と一致しない問題を修正しました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）:

  * Refパラメータスキーマが逆参照されていない場合に、プラグインがランタイムエラーをスローするバグを修正しました。[\#7543](https://github.com/Kong/kong/issues/7543)
  * 有効な再帰スキーマが常に拒否される問題を修正しました。
  * AnyTypeスキーマと定義済みスタイルキーワードでパラメータを検証する間に、プラグインがランタイムエラーをスローする問題を修正しました。
  * null許容のキーワードが有効にならない問題を修正しました。
  * URIコンポーネントのエスケープ文字が誤ってエスケープ解除される問題を修正しました。
  * パスパラメータが非ASCII文字と一致しない問題を修正しました。

* [**OAuth 2\.0 Introspection**](/hub/kong-inc/oauth2-introspection/)（`oauth2-introspection`）:

  * `oauth2-introspection`プラグインの`authorization_value`を暗号化されたフィールドとしてマークしました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）：

  * Dev PortalがOIDCで有効になっていて、管理者が正常にログインしてセッションを取得すると、500エラーが発生する問題を修正しました。
  * トークンの有効期限を計算する際の更新時間を修正しました。

* [**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）:

  * すべてのカウンターが同じレートで同じDBに同期される問題を修正しました。[\#7314](https://github.com/Kong/kong/issues/7314)

* [**TCP Log**](/hub/kong-inc/tcp-log)（`tcp-log`）:

  * TLS接続を再利用する際の不要なハンドシェイクの問題を修正しました。[\#7114](https://github.com/Kong/kong/issues/7114)

### パフォーマンス

#### 構成

* `dns_stale_ttl`のデフォルトを1時間に変更しました。これにより、リゾルバにダウンタイムが発生した場合でも、古いDNSレコードをより長い時間使用できるようになります。 

### 依存関係

#### コア

* `openresty`を1\.21\.4\.1から1\.21\.4\.3に更新しました [\#7206](https://github.com/Kong/kong/issues/7206)
* `resty-openssl`を0\.8\.25から1\.0\.2に更新しました [\#7417](https://github.com/Kong/kong/issues/7417)
* `lua-resty-healthcheck`を1\.6\.2から1\.6\.3に更新しました [\#7206](https://github.com/Kong/kong/issues/7206)
* `lua-kong-nginx-module`を0\.6\.0から0\.8\.0に更新しました [\#7207](https://github.com/Kong/kong/issues/7207)
* jqを1\.7に更新しました
* luasecを1\.3\.2に更新しました

#### デフォルト

* `lua-resty-aws`を1\.2\.3から1\.3\.0に更新しました [\#7079](https://github.com/Kong/kong/issues/7079)
* `lua-resty-aws`を1\.3\.2から1\.3\.5に更新しました [\#7318](https://github.com/Kong/kong/issues/7318)

3\.4\.2\.0
-------------

**リリース日** 2023/11/10

### 機能

#### Enterprise

* ライセンス管理: 
  * Kong Enterpriseライセンスの有効期限から30日間続く新しい猶予期間を実装しました。 猶予期間中は、すべてのオープンソース機能が利用可能になり、Enterprise機能は読み取り専用モードに設定されます。
  * ルート、プラグイン、ライセンス、デプロイ情報などのカウンターのサポートをライセンスレポートに追加しました。
  * ライセンスエンドポイントの出力にチェックサムを追加しました。

### 修正

#### コア

* DNSクライアントが、設定されたタイムアウトを予測どおりに遵守しない問題を修正しました。また一時的なネットワークおよびDNSサーバーの障害時に、DNSクライアントが誤って解決される原因となっていた関連問題も修正しました。[\#11386](https://github.com/Kong/kong/pull/11386)
* [`dns_no_sync`](/gateway/3.4.x/reference/configuration/#dns_no_sync)オプションのデフォルト値が`on`に変更されました。[\#11871](https://github.com/kong/kong/pull/11871)。
* 流量制限に関するRedisからの混乱を招くログエントリを無視しました。

#### Kong Manager

* ルートの構成中に、一部のサービスで正確な名前またはIDが表示されない問題を修正しました。

#### プラグイン

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）
  * バランサーのインストルメンテーションが有効になっていると、トレースで親IDが無効になる問題を修正しました。[\#11830](https://github.com/Kong/kong/pull/11830)
  * 新しい`aws`ヘッダータイプをサポートしていない古いDPに、ハイブリッドモードの互換性を追加します。[\#11686](https://github.com/Kong/kong/pull/11686)

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）
* 新しい`aws`ヘッダータイプをサポートしていない古いDPに、ハイブリッドモードの互換性を追加します。[\#11686](https://github.com/Kong/kong/pull/11686)
* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）
  * `using_pseudo_issuer`が伝達された後に使用されない問題を修正しました。

### 依存関係

#### Enterprise

* OpenSSLを3\.1\.2から3\.1\.4にアップグレードしました
* コンテナイメージにトラブルシューティングツールを追加しました
* `ngx_wasm_module`バージョンをプレリリース0\.1\.1に更新しました

3\.4\.1\.1
-------------

**リリース日** 2023/10/12

### 修正

#### コア

* HTTP/2ストリームリセット攻撃を早期に検出するためのNginxパッチを適用しました。
  この変更は、特定された脆弱性[CVE\-2023\-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)に直接対処するものです。

  この脆弱性とそれに対するKongの対処に関する詳細については、当社の[ブログ記事](https://konghq.com/blog/product-releases/novel-http2-rapid-reset-ddos-vulnerability-update)をご覧ください。

#### プラグイン

* [**SAML**](/hub/kong-inc/saml/)（`saml`）:`session was not found`メッセージの重大度を`info`に調整しました。

### 依存関係

* `libxml2`を2\.10\.3から2\.11\.5に更新しました

3\.4\.1\.0
-------------

**リリース日** 2023/09/28

### 下位互換性のない変更

* [**GraphQL Rate Limiting Advanced**](/hub/kong-inc/graphql-rate-limiting-advanced/)（`graphql-rate-limiting-advanced`）: スキーマ検証が更新され、Redisクラスタモードがサポートされるようになりました。このスキーマの変更は、このプラグインの他の実装には影響しません。

### 機能

#### コア

* 式ルートでHTTPクエリパラメータをサポートします。

#### プラグイン

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）:
  * 新しいフィールド`unauthorized_destroy_session`は、trueに設定すると、リクエストが不正な場合にユーザーのセッションCookieを削除してセッションを破棄します。デフォルトは`true`です。セッションを保持するには、`false`に設定します。
  * 新しいフィールド`using_pseudo_issuer`は、trueに設定すると、プラグインインスタンスが発行者から構成を検出しません。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）: パラメータ`header_type`に新しい値が追加され、Kongは、アップストリームサービスと通信するときに、転送されたリクエストのヘッダーにDatadogヘッダーをシームレスに注入できるようになりました。

### 修正

#### コア

* `nginx_http_proxy_wasm_isolation`構成値が有効にならない原因となっていた、ハードコードされたProxy\-Wasm分離レベル設定を削除しました。
* DB\-lessモードおよびハイブリッドモードでKey AuthプラグインのTTLが機能しない問題を解決しました。
* Postgresデータベースをクエリするときに異常なソケット接続が再利用される問題を修正しました。
* プラグインが応答ハンドラを使用したときにアップストリームのSSL障害が起こる問題を修正しました。
* `tls_passthrough`プロトコルがルーター式フレーバーで動作しない問題を修正しました。
* 認証されたココンシューマが複数のコンシューマグループに属している場合にプラグインが正常にトリガーされない問題を修正しました。
* クラスタストラテジを使用しているときにKongノードがキーリングマテリアルを送信できないというキーリングの問題を修正しました。
* リクエストの`x-datadog-parent-id`ヘッダーの値が短い10進文字列の場合に、トレーシングデータをDatadogに送信できない問題を修正しました。
* グループ名のタイプが数値であるグループロールを、RBACが取得する方法を修正しました。
* 外部プラグインサーバーを起動する際の重大レベルのログを修正しました。OpenRestyの制限により、これらのログを抑制することはできません。ソケット可用性の検出機能を削除することにしました。

#### PDK

* Vaultのいくつかの問題を修正し、Vaultコードベースをリファクタリングしました。
  * Vaultリファレンスの解決に失敗した場合、DAOを空の文字列にフォールバックさせます
  * 参照をローテーションする際にノードレベルのmutexを使用します
  * 構成変更時に参照を更新します
  * プラグインが参照する値は、リクエストごとに1回のみ更新されます
  * 有効な構成オプションのみをVault実装に渡します
  * 複数の値を持つシークレットを、ローテーションするときに1回だけ解決します
  * コントロールプレーンではVaultシークレットのローテーションタイマーを開始しません
  * ネガティブキャッシュを再度有効にします
  * `kong.vault.try`関数を再実装します
  * 構成が変更された場合、ローテーションから参照を削除します

* トレーシング: タイムスタンプの精度が異なるために、一部の親スパンが子スパンよりも先に終了する問題を修正しました。

#### プラグイン

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）: 伝達されたトレーシングヘッダーに無効な親IDが発生する問題を修正します。
* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）：失効チェックの実行時にネットワーク障害をキャッシュしなくなりました。
* [**Canary**](/hub/kong-inc/canary/)（`canary`）: `start`フィールドに過去の時刻を入力できるようにします。
* [**SAML**](/hub/kong-inc/saml/)（`saml`）: Redisセッションストレージが正しく構成されていない場合、ユーザーは、無限にリダイレクトされるのではなく、500エラーを受け取るようになりました。
* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）: ログアウト時のトークン失効の問題を修正しました。検出された失効エンドポイントを使用しているときに、アクセストークンを取り消すはずのコードがリフレッシュトークンを取り消していました。

### Kong Manager

* Kong Managerが[Gateway Admin API \- EE（ベータ版）](/gateway/api/admin-ee/3.4.0.x/)に直接リンクするようになりました

### 依存関係

* ARM64上の誤ったLuaJIT LDP/STP融合を修正しました。これにより、誤ったロジックが発生することがありました。

3\.4\.0\.0
-------------

**リリース日** 2023/08/09

### 下位互換性のない変更と非推奨

* **Cassandra DBのサポートの削除:** Cassandra DBのサポートは削除されました。Kong Gatewayのデータストアとしてはサポートされなくなりました。
  [\#10931](https://github.com/Kong/kong/pull/10931)。

* **Alpineサポートの削除:** AlpineパッケージとAlpineをベースにしたDockerイメージはサポートされなくなりました。
  Kong Gateway 3\.4\.0\.0以降、Kongは新しいAlpineイメージやパッケージを構築していません。
  [\#10926](https://github.com/Kong/kong/pull/10926)

* **Ubuntu 18\.04サポートの削除** : Ubuntu 18\.04（"Bionic"）でのKong Gatewayの実行のサポートが非推奨となりました。[Ubuntu 18\.04の標準サポートは2023年6月をもって終了しました](https://wiki.ubuntu.com/Releases)。
  Kong Gateway 3\.4\.0\.0以降、Kongは新しいUbuntu 18\.04イメージまたはパッケージをビルドしていません。また、KongはUbuntu 18\.04でのパッケージのインストールをテストしません。

  Kong GatewayをUbuntu 18\.04にインストールする必要がある場合は、[以前のバージョン](/gateway/3.1.x/install/linux/ubuntu/)のドキュメントを参照してください。
* Amazon Linux 2022アーティファクトは、AWS独自の名前変更に基づいて、Amazon Linux 2023に名前が変更されます。

* LMDB暗号化が無効になりました。`declarative_config_encryption_mode`のオプションは`kong.conf`から削除されました。

* `/consumer_groups/:id/overrides`エンドポイントが非推奨となり、より汎用的なプラグインスコープメカニズムが推奨されるようになりました。
  詳細は、新しい[コンシューマグループ](/gateway/api/admin-ee/3.4.0.x/#/consumer_groups/get-consumer_groups)エンティティを参照してください。

* 構成プロパティ`admin_api_uri`の名前を`admin_gui_api_url`に変更しました。古い`admin_api_uri`プロパティは非推奨とみなされ、Kong Gatewayの将来のバージョンでは完全に削除されます。

* Kongが提供していたRHEL8 Dockerのイメージは、RHEL9 Dockerのイメージに置き換えられました。RHEL8パッケージは、[Kongのパッケージリポジトリから](https://cloudsmith.io/~kong/repos/gateway-34/packages/?q=distribution%3Arhel+AND+distribution%3A8)引き続き入手可能です。

### 機能

#### デプロイメント

* Kong Gatewayを[RHEL 9](https://cloudsmith.io/~kong/repos/gateway-34/packages/?q=distribution%3Arhel+AND+distribution%3A9)で利用できるようになりました。

#### Enterprise

* `/workspaces`に[`cascade`](/gateway/latest/admin-api/workspaces/reference/#delete-a-workspace)オプションを導入して、1度のリクエストでワークスペースとそのすべてのエンティティを削除できるようになりました。

* コンシューマグループは、コアエンティティになりました。コンシューマグループを使用すると、選択したコンシューマのグループにさまざまな構成を適用できます。
  次のプラグイが、コンシューマグループにスコープ設定できるようになりました。

  * Rate Limiting Advanced
  * Request TransformerとRequest Transformer Advanced
  * Response TransformerとResponse Transformer Advanced

  詳細については、[コンシューマグループ](/gateway/latest/kong-enterprise/consumer-groups/)のドキュメントを参照してください。
* Vault構成に新しい`ttl`オプションが追加され、構成されたVaultから参照が自動的に再取得される間隔を、ユーザーが定義できるようにしました。

  詳細については、[シークレットローテーション](/gateway/latest/kong-enterprise/secrets-management/secrets-rotation/)のドキュメントを参照してください。
* ワークスペース名がロギングペイロードに表示されるようになりました。

#### Kong Manager

* **Kong Managerオープンソースエディション（OSS）** を導入しました。Kong Gateway OSS用の無料のオープンソースUIです。
  [\#11131](https://github.com/Kong/kong/pull/11131)

  [Kong Manager OSS](/gateway/latest/kong-manager-oss/)を使用すると、Admin APIを使用してすべてのKong Gatewayオブジェクトを表示して編集できます。Kong Admin APIと直接やり取りするため、別のデータベースは必要ありません。
  このUIを使用すると、Kong Gatewayのすべての構成を一目で確認できます。

  3\.4\.0\.0以降、Kong Manager OSSはKong Gateway OSSにバンドルされています。
  新しいKong Gateway OSSインスタンスをインストールして、試してみましょう。

  最も早く始める方法は、[クイックスタートのスクリプト](https://github.com/Kong/kong-manager#getting-started)を使うことです。

  詳細は、[Kong Manager OSSリポジトリ](https://github.com/Kong/kong-manager)をチェックしてください。
* エンティティの編集ページのユーザーエクスペリエンスが、洗練された外観と操作性によって向上しました。

* ネストされたエンティティの構成ページを削除することで、ユーザーパスを簡素化しました。

#### コア

* **ベータ機能:** WebAssembly のベータ版（`proxy-wasm`）を導入しました。
  [\#11218](https://github.com/Kong/kong/pull/11218)

  このリリースでは、[`Kong/ngx-wasm-module`](https://github.com/Kong/ngx_wasm_module)がKong Gatewayに統合されています。
* `/schemas`エンドポイントは、スキーマの一部としてクロスフィールド検証に関する追加情報を返すようになりました。
  これにより、Admin APIを使用するツールが実行するクライアント側の検証が改善されます。

* ストリームサブシステムで`expressions`および`traditional_compatible`ルーターフレーバーが有効になりました。
  [\#11071](https://github.com/Kong/kong/pull/11071)

* アップストリーム`host_header`とルーター`preserve_host`構成のパラメータが、ストリームTLSプロキシで機能するようになりました。
  [\#11244](https://github.com/Kong/kong/pull/11244)

* 宣言型スキーマはDBレスモードで起動時に完全に初期化されるようになり、リクエストパスでオンデマンドで実行する必要がなくなりました。この変更の影響は、`/config` APIエンドポイントを介した構成更新時における応答レイテンシの減少に最も顕著に表れています。
  [\#10932](https://github.com/Kong/kong/pull/10932)

* トレーシング：HTTPリクエストスパンに新たな属性、`http.route`を追加しました。
  [\#10981](https://github.com/Kong/kong/pull/10981)

* トレーシング: アップストリームのホスト名が`balancer_data.hostname`で利用可能な場合に、それを記録するスパン属性`net.peer.name`を追加しました。この変更に貢献してくれた[@backjo](https://github.com/backjo)に感謝します。
  [\#10723](https://github.com/Kong/kong/pull/10729)

* DB\-lessおよびハイブリッドモードで最も一般的にデプロイされている構成サイズに対応するため、`lmdb_map_size`構成のデフォルト値を`128m`から`2048m`に変更しました。
  [\#11047](https://github.com/Kong/kong/pull/11047)

* ハイブリッドモードで最も一般的にデプロイされている構成サイズに対応するため、`cluster_max_payload`構成のデフォルト値を`4m`から`16m`に変更しました。
  [\#11090](https://github.com/Kong/kong/pull/11090)

* Kong HTMLエラーテンプレートからKongブランドを削除しました。
  [\#11150](https://github.com/Kong/kong/pull/11150)

#### プラグイン

* プラグインキュー関連のパラメータの検証が改善されました。[\#10840](https://github.com/Kong/kong/pull/10840)

  * `max_batch_size`、`max_entries`、および`max_bytes`は、`number`ではなく`integer`として宣言されるようになりました。
  * `initial_retry_delay`と`max_retry_delay`は、0\.001（秒単位）より大きい数値にする必要があります。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `redis`ストラテジが、ストラテジ接続の失敗をキャッチできるようになりました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * このプラグインはエラーリーズンヘッダーをサポートするようになりました。 このヘッダーは、`expose_error_code`を`false`に設定することでオフにできます。
  * OpenID Connectは、`token_cache_key_include_scope`を`true`に設定することで、トークンキャッシュキーにスコープを追加できるようになりました。

* [**Kafka Log**](/hub/kong-inc/kafka-log/)（`kafka-log`）

  * Kafka Logプラグインが`custom_fields_by_lua`構成をサポートし、Luaコードを使用してログフィールドを動的に変更できるようになりました。

* [**GraphQL Rate Limiting Advanced**](/hub/kong-inc/graphql-rate-limiting-advanced/)（`graphql-rate-limiting-advanced`）

  * このプラグインの`host`フィールドが、Kongアップストリームターゲットを受け入れるようになりました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * AWS X\-Ray伝達ヘッダーのサポートが導入されました。 フィールド`header_type`は、この特定の伝達ヘッダーを処理するために`aws`値を受け入れるようになりました。[\#11075](https://github.com/Kong/kong/pull/11075)
  * `endpoint`パラメータが参照可能になり、シークレットとしてVaultに保存できるようになりました。 [\#11220](https://github.com/Kong/kong/pull/11220)

* [**IP Restriction**](/hub/kong-inc/ip-restriction/)（`ip-restriction`）

  * `tcp`、`tls`、`grpc`、および`grpcs`プロトコルのサポートが追加されました。

    この変更に貢献してくれた[@scrudge](https://github.com/scrudge)に感謝します。
    [\#10245](https://github.com/Kong/kong/pull/10245)

* [**Prometheus**](/hub/kong-inc/prometheus/)（`prometheus`）

* Prometheusプラグインは、スクレイピング中のプロキシレイテンシの影響を軽減するように最適化されました。[\#10949](https://github.com/Kong/kong/pull/10949)
  [\#11040](https://github.com/Kong/kong/pull/11040)
  [\#11065](https://github.com/Kong/kong/pull/11065)

### 修正

#### Enterprise

* `send`スレッドの例外が原因でテレメトリが壊れた場合に発生する可能性があった、潜在的なメモリリークと再接続の問題を修正しました。
* Telemetry: テレメトリWebSocketを壊していた問題を修正しました: 
  * Vitalsをデータベースにフラッシュする際に、テレメトリWebSocketがレイテンシによってブロックされる問題を修正しました。キューをバッファとして使用することにより、データプレーンからVitalsデータを受信するプロセスは、Vitalsをコントロールプレーンのデータベースにフラッシュするプロセスから切り離されました。
  * Konnectモードにおいて、リクエストのカウンターがゼロの場合、予期しない原因でテレメトリWebSocketが壊れる問題を修正しました。

* 監査データを生成するときに、空の`request_id`を受け取る可能性がある問題を修正しました。
* ヘッダー`x-datadog-parent-id`がKong Gatewayに渡されなかった場合に発生するエラーを修正しました。
* 3\.3\.0\.0でイベントフックを壊していたキュー関連の問題を修正しました。
* Kong Gatewayがsystemdによって制御されている場合にSAMLプラグインが動作するよう、データファイルライブラリを更新しました。
* ワークスペースがキャッシュのコンシューマにうまく添付できない問題を修正しました。
* Arm64でのLuaJITクラッシュを修正し、M1でLuaJITを有効にしました。
* Vaultから`KONG_LICENSE_DATA`をプルするときにライセンスを読み込めない問題を修正しました。

#### Kong Manager

* EnterpriseライセンスがAdmin APIを介して投稿されたときに、Kong Managerが最新の構成を取得できない問題を修正しました。
* Kong ManagerがポータルGUIと統合されたときに発生していた、CORSの誤った動作を修正しました。
* Kong ManagerのOIDCが間違ったユーザー名を指定した場合に、`invalid credentials`を処理しない問題を修正しました。
* `admin_auth`が`openid-connect`に設定されている場合、`workspace access`の`admins tab`ページに表示する警告メッセージを追加しました。
* `/services/<service-name-or-id id="sl-md0000000">/application_instances`でカスタム権限エンドポイントが機能しない問題を修正しました。

#### Dev Portal

* ポータルのドキュメントページで、アプリケーション登録プラグインを無効にしても、サービスから **\[登録\]** ボタンが削除されない問題を修正しました。
* Dev PortalでOASドキュメントを表示する際に、APIを拡張しようとするとUIがハングする問題を修正しました。

#### コア

* 宣言型構成で入力に照らして適切な一意性チェックが実行されるようになりました。 以前は、プライマリキーとエンドポイントキーが競合するエントリを認識せずに削除したり、競合する一意のフィールドを認識せずに受け入れたりしていました。 [\#11199](https://github.com/Kong/kong/pull/11199)
* 動的ログレベル設定イベントを使用するワーカーが、通知ログに間違った参照を使用するバグを修正しました。 [\#10897](https://github.com/Kong/kong/pull/10897)
* Kong Gatewayが再びsystemdによって制御できるように、systemdユニット定義に`User=`仕様を追加しました。[\#11066](https://github.com/Kong/kong/pull/11066)
* サンプリングレートが個々のスパンに適用され、分割されたトレースが生成されるバグを修正しました。 [\#11135](https://github.com/Kong/kong/pull/11135)
* ルートが複数ありサービスが作成されなかった場合に、`traditional_compatible`モードでルーターが失敗するバグを修正しました。 [\#11158](https://github.com/Kong/kong/pull/11158)
* `route.protocols`が`grpc`または`grpcs`に設定されている場合、`expressions`ルーターが正常に機能しない問題を修正しました。 [\#11082](https://github.com/Kong/kong/pull/11082)
* `expressions`ルーターがHTTPSリダイレクトを構成できない問題を修正しました。 [\#11166](https://github.com/Kong/kong/pull/11166)
* Kong CLI `nginx.conf`に必要なディレクティブを注入することで、`kong vault get` CLIコマンドがDB\-lessモードで動作するようになりました。[\#11127](https://github.com/Kong/kong/pull/11127) [\#11291](https://github.com/Kong/kong/pull/11291)
* Goプラグインサーバープロセスがクラッシュすると、Kong Gatewayを介してプロキシされた後続のリクエストが、一貫性のない構成でGoプラグインを実行する問題を修正しました。この問題は、同じGoプラグインが異なるルートまたはサービスエンティティに適用されている場合にのみ影響します。[\#11306](https://github.com/Kong/kong/pull/11306)

#### Admin API

* 特定の状況下で、`POST /config?flatten_errors=1`が例外をスローし500エラーを返す問題を修正しました。[\#10896](https://github.com/Kong/kong/pull/10896)
* `custom_fields_by_lua`のキーにドット（`.`）文字が含まれている場合に、`/schemas/plugins/validate`エンドポイントが有効なプラグイン構成を検証できなかった問題を修正しました。 [\#11091](https://github.com/Kong/kong/pull/11091)

#### Status API

* DB\-lessモードまたはデータプレーン上で操作する際に、Status APIからデータベース情報を削除しました。 [\#10995](https://github.com/Kong/kong/pull/10995)

#### プラグイン

* [**OAuth 2\.0 Introspection**](/hub/kong-inc/oauth2-introspection/)（`oauth2-introspection`）

  * テーブルではないJSONを含むリクエストを処理する場合に、プラグインが失敗する問題を修正しました。

* [**gRPC Gateway**](/hub/kong-inc/grpc-gateway/)（`grpc-gateway`）

  * 要素が1つである配列のエンコードに失敗する問題を修正しました。
  * 空（すべてデフォルト値）のメッセージを正しくフレーム解除できない問題を修正しました。 [\#10836](https://github.com/Kong/kong/pull/10836)

* [**Response Transformer**](/hub/kong-inc/response-transformer/)（`response-transformer`）と
  [**Request Transformer Advanced**](/hub/kong-inc/request-transformer-advanced/)（`request-transformer-advanced`）

  * アップストリームが、サブタイプとして`+json`サフィックスを持つContent\-Typeを返したときに、プラグインが応答本文を変換しない問題を修正しました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 可視性を向上させるために、いくつかのログレベルを`notice`から`error`に変更しました。
  * `log`と`message`に正しいテーブルキーを設定しました。
  * 無効で不透明なトークンが提供されたが検証に失敗した場合、プラグインは正しいエラーを出力するようになりました。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * パスノードに任意の要素が定義されている場合にプラグインがエラーをスローする問題を修正しました。

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * いくつかの失効検証の問題を修正しました。
    * `revocation_check_mode=IGNORE_CA_ERROR`の場合、CRL失効の失敗は無視されます。
    * CRLがストアに追加されると、常にこのCRLファイルを使用してCRL失効チェックが実行されます。
    * クライアントからリーフ証明書のみが送信される場合、OCSP検証は`no issuer certificate in chain`エラーで失敗していました。
    * `http_timeout` が正しく設定されていませんでした。

  * CRL失効検証を最適化しました。
  * `skip_consumer_lookup`が有効になっていて`authenticated_group_by`が`null`に設定されている場合に、予期しないエラーが発生するバグを修正しました。

* [**Kafka Log**](/hub/kong-inc/kafka-log/)（`kafka-log`）と[**Kafka Upstream**](/hub/kong-inc/kafka-upstream/)（`kafka-upstream`）

  * ブローカーのリーダーが変わったときに、プラグインがブローカーとの接続を失う可能性がある問題を修正しました。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）

  * パスパラメータが有効であっても、プラグインが検証を渡せない問題を修正しました。
  * メソッド仕様に`requestBody`が定義されていない場合でも、プラグインが常に応答本文を検証していた問題を修正しました。
  * 大きな絶対値の数値同士を比較すると、数値が指数表記に変換されるため不正確になる問題を修正しました。 

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * 無効なリクエストの応答メッセージを最適化しました。

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * ハイブリッドモードの`kong`ストレージでサニティテストが機能しない問題を修正しました。 [\#10852](https://github.com/Kong/kong/pull/10852)

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)

  * `redis`ポリシーの精度に影響を与えていた問題を修正しました。この変更に貢献してくれた[@giovanibrioni](https://github.com/giovanibrioni)に感謝します。 [\#10559](https://github.com/Kong/kong/pull/10559)

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）

  * インストルメンテーションが有効になっているときにトレースが正しく生成されなかった問題を修正しました。 [\#10983](https://github.com/Kong/kong/pull/10983)

### 依存関係

* `kong-redis-cluster`を1\.5\.0から1\.5\.1に更新しました
* `lua-resty-ljsonschema`を1\.1\.3から1\.15に更新しました
* `lua-resty-kafka`を0\.15から0\.16に更新しました
* `lua-resty-aws`を1\.2\.2から1\.2\.3に更新しました
* `lua-resty-openssl`を0\.8\.20から0\.8\.23に更新しました [\#10837](https://github.com/Kong/kong/pull/10837) [\#11099](https://github.com/Kong/kong/pull/11099)
* `kong-lapis`を1\.8\.3\.1から1\.14\.0\.2に更新しました [\#10841](https://github.com/Kong/kong/pull/10841)
* `lua-resty-events`を0\.1\.4から0\.2\.0に更新しました [\#10883](https://github.com/Kong/kong/pull/10883) [\#11083](https://github.com/Kong/kong/pull/11083) [\#11214](https://github.com/Kong/kong/pull/11214)
* `lua-resty-session`を4\.0\.3から4\.0\.4に更新しました [\#11011](https://github.com/Kong/kong/pull/11011)
* `OpenSSL`を1\.1\.1tから3\.1\.1に更新しました [\#10180](https://github.com/Kong/kong/pull/10180) [\#11140](https://github.com/Kong/kong/pull/11140)
* `pgmoon`を1\.16\.0から1\.16\.2（Kongのフォーク）に更新しました [\#11181](https://github.com/Kong/kong/pull/11181) [\#11229](https://github.com/Kong/kong/pull/11229)
* `atc-router`を1\.0\.5から1\.2\.0に更新しました [\#10100](https://github.com/Kong/kong/pull/10100) [\#11071](https://github.com/Kong/kong/pull/11071)
* `lua-resty-lmdb`を1\.1\.0から1\.3\.0に更新しました [\#11227](https://github.com/Kong/kong/pull/11227)

### 既知の問題

* 参照可能な構成フィールドの一部で不適切なフィールド検証が原因で参照値が受け入れられません。例：`http-log`プラグインの`http_endpoint`フィールド、`opentelemetry`プラグインの`endpoint`フィールド。 

3\.3\.1\.1
-------------

**リリース日** 2023/10/12

### 下位互換性のない変更

* **Ubuntu 18\.04サポートの削除** : Ubuntu 18\.04（"Bionic"）でのKong Gatewayの実行のサポートが非推奨となりました。[Ubuntu 18\.04の標準サポートは2023年6月をもって終了しました](https://wiki.ubuntu.com/Releases)。
  Kong Gateway 3\.2\.2\.4以降、Kongは新しいUbuntu 18\.04イメージまたはパッケージをビルドしていません。また、KongはUbuntu 18\.04でのパッケージのインストールをテストしません。

  Kong GatewayをUbuntu 18\.04にインストールする必要がある場合は、[インストール手順](/gateway/3.2.x/install/linux/ubuntu/)にある以前の3\.2\.xパッチバージョンに置き換えます。
* Amazon Linux 2022アーティファクトが、AWSの名称変更に合わせて、Amazon Linux 2023とラベル付けされるようになりました。

* CentOSパッケージはリリースから削除され、将来のバージョンではサポートされなくなりました。

### 機能

#### プラグイン

* [**GraphQL Rate Limiting Advanced**](/hub/kong-inc/graphql-rate-limiting-advanced/)（`graphql-rate-limiting-advanced`）および[**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）: Redisストラテジは、ストラテジ接続の失敗をキャッチするようになりました。

### 修正

#### コア

* HTTP/2ストリームリセット攻撃を早期に検出するためのNginxパッチを適用しました。
  この変更は、特定された脆弱性[CVE\-2023\-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)に直接対処するものです。

  この脆弱性とそれに対するKongの対処に関する詳細については、当社の[ブログ記事](https://konghq.com/blog/product-releases/novel-http2-rapid-reset-ddos-vulnerability-update)をご覧ください。
* PostgreSQLデータベースにクエリを実行すると、異常なソケット接続が誤って再利用される問題を修正しました。

* クラスタストラテジの使用時に、Kong Gatewayノードがキーリングデータの送信に失敗するというキーリングの問題を修正しました。

* Goプラグインサーバープロセスがクラッシュすると、Kong Gatewayを介してプロキシされた後続のリクエストが、一貫性のない構成でGoプラグインを実行する問題を修正しました。この問題は、同じGoプラグインが異なるルートまたはサービスエンティティに適用されている場合にのみ影響します。

* サンプリングレートが個々のスパンに適用され、分割されたトレースが生成されるバグを修正しました。

* ワーカーキューの問題を修正しました。

  * ワーカーがシャットダウンモードにありすぐ利用可能なデータが増える場合、ワーカーキューがバッチ単位でクリアされるようになり、`max_coalescing_delay`を待つ必要がなくなりました。 [\#11376](https://github.com/Kong/kong/pull/11376)
  * `max_entries`が`max_batch_size`に設定されている場合に、ワーカーがクラッシュする可能性があるプラグインキューの競合状態を修正しました。 [\#11378](https://github.com/Kong/kong/pull/11378)

* systemdユニット定義に`User=`仕様を追加し、Kong Gatewayが再びsystemdによって制御できるようになりました。[\#11066](https://github.com/Kong/kong/pull/11066)

#### プラグイン

* [**SAML**](/hub/kong-inc/saml/)（`saml`）: Redisセッションストレージが正しく構成されていない場合、ユーザーは、無限にリダイレクトされるのではなく、500エラーを受け取るようになりました。

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）：

  * プラグインは、`log`と`message`のテーブルキーを正しく設定するようになりました。
  * 無効で不透明なトークンが提供されて検証に失敗した場合、プラグインは正しいエラーを出力するようになりました。

* [**Response Transformer Advanced**](/hub/kong-inc/response-transformer-advanced/)（`response-transformer-advanced`）: `if_status`が指定されたステータスと一致しない場合、プラグインは応答本文を読み込まなくなりました。

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）: 証明書失効チェックの実行時にプラグインがネットワーク障害をキャッシュする問題を修正しました。

### 依存関係

* `libxml2`を2\.10\.2から2\.11\.5に更新しました
* `lua-resty-kafka`を0\.15から0\.16に更新しました
* `OpenSSL`を1\.1\.1tから3\.1\.1に更新しました 

3\.3\.1\.0
-------------

**リリース日** 2023/07/03

### 修正

* 特定の状況下で、`POST /config?flatten_errors=1`が例外をスローし500エラーを返す問題を修正しました。
* ヘッダー`x-datadog-parent-id`がKongに渡されない場合にエラーが発生するバグを修正しました。
* `event_hooks`がトリガーせず、ログでエラーが発生する、キューに関連するバグを修正しました。
* データファイルライブラリを更新しました。Kongがsystemdで起動されたときに、SAMLプラグインが読み込まれませんでした。
* `anonymous_reports=false`を設定しても匿名レポートを非表示にできないバグを修正しました。
* `kong/kong-gateway:3.3.0.0-alpine`に`resty.dns.resolver`パッチが欠落していたJenkinsの問題を修正しました。
* ワークスペースをキャッシュのコンシューマに添付する際に時々発生する問題を修正しました。

#### プラグイン

* テーブルではないJSONを含むリクエストが失敗するというOauth 2\.0 Introspectionプラグインの問題を修正しました。

### 非推奨

* **Alpine非推奨のお知らせ:** KongはAlpineイメージとパッケージのサポートを今年中に終了する意向を発表しました。これらのイメージとパッケージは3\.2で利用可能であり、3\.3でも引き続き利用可能になります。Kong Gateway 3\.4では、Alpineイメージとパッケージのビルドを停止します。

* **Cassandraの非推奨と削除のお知らせ:** Kong GatewayのバックエンドデータベースとしてCassandraを使用することは非推奨です。
  {{site.base_gateway}} 3\.4での削除が予定されています。

3\.3\.0\.0
-------------

**リリース日** 2023/05/19

### 下位互換性のない変更と非推奨

* **Alpine非推奨のお知らせ:** KongはAlpineイメージとパッケージのサポートを今年中に終了する意向を発表しました。これらのイメージとパッケージは、3\.3でも引き続き利用可能です。Kong Gateway 3\.4では、Alpineイメージとパッケージのビルドを停止します。

* **Cassandraの非推奨と削除のお知らせ:** Kong GatewayのバックエンドデータベースとしてCassandraを使用することは非推奨です。
  {{site.base_gateway}} 3\.4での削除が予定されています。

#### コア

* 複数のパスを持つルートを優先順位が異なる複数の`atc`ルートに分割することで、`traditional_compat`ルーターモードと`traditional`モードの動作との互換性が向上しました。Kong Gateway 3\.0で新しいルーターが導入されて以来、`traditional_compat`モードでは、異なる接頭辞パス長と正規表現がルートに混在している場合でも、各ルートには1つの優先順位のみが割り当てられるようになっています。これは、複数パスが`traditional`ルーターで処理されていた方法ではなく、動作が変更されたことによって、別々の優先順位値がルートの各パスに割り当てられるようになります。[\#10615](https://github.com/Kong/kong/pull/10615)

* **トレーシング** : `tracing_sampling_rate`のデフォルトは、以前の1（すべてのリクエストをトレース）ではなく、0\.01（100のリクエストごとに1つトレース）になりました。すべてのリクエストをトレースすると、ほとんどのプロダクションシステムで不必要なリソースが浪費されます。
  [\#10774](https://github.com/Kong/kong/pull/10774)

#### プラグイン

* プラグインのバッチキューイング：

  * [**HTTP Log**](/hub/kong-inc/http-log/)（`http-log`）、[**StatsD**](/hub/kong-inc/statsd/)（`statsd`）、
    [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）、[**Datadog**](/hub/kong-inc/datadog/)（`datadog`）

    プラグインキューイングシステムを改良した結果、一部のプラグインパラメータは想定どおり機能しないようになっています。
    該当プラグインでキューを使用する場合は、新しいパラメータの構成が必要です。
    詳細については、各プラグインのドキュメントを参照してください。
  * モジュール`kong.tools.batch_queue`の名前が`kong.tools.queue`に変更され、APIが変更されました。カスタムプラグインがキューを使用する場合は、新しいAPIを使用するよう更新する必要があります。
    [\#10172](https://github.com/Kong/kong/pull/10172)

* [**AppDynamics**](/hub/kong-inc/app-dynamics/)（`app-dynamics`）

  * プラグインのバージョンは、Kong Gatewayのバージョンに合わせて更新されました。

* [**HTTP Log**](/hub/kong-inc/http-log/)（`http-log`）

  * ログサーバーが3xx HTTPステータスコードで応答した場合、プラグインはそれをエラーと見なし、再試行構成に従って再試行するようになりました。以前は、3xxステータスコードは成功として解釈され、ログエントリが削除されていました。 [\#10172](https://github.com/Kong/kong/pull/10172)

* **[Pre\-function](/hub/kong-inc/pre-function/)（`pre-function`）と[Post\-function](/hub/kong-inc/post-function/)** （`post-function`）

  * `kong.cache`は、サーバーレス関数プラグイン専用のキャッシュインスタンスを指すようになりました。グローバルKong Gatewayキャッシュへのアクセスは提供されません。`kong.conf`の特定のフィールドへのアクセスも制限されています。 [\#10417](https://github.com/Kong/kong/pull/10417)

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）このプラグインは、内部バッファリングにキューを使用するようになりました。キューイング動作を制御するために、標準キューパラメータセットを使用できます。
  [\#10753](https://github.com/Kong/kong/pull/10753)

### 機能

#### Enterprise

* [データプレーンのレジリエンス機能](/gateway/latest/kong-enterprise/cp-outage-handling-faq/)を使用する場合、バックエンドのAmazon S3またはGCP Cloud Storageサービスのサーバー側証明書は、HTTPSを経由する場合に検証されるようになりました。
* AWSまたはGCPバックエンドで[シークレットを管理する](/gateway/latest/kong-enterprise/secrets-management/)場合、バックエンドサーバーの証明書は、HTTPSを経由する場合に検証されるようになりました。
* Kong Enterpriseでは、[AWS IAMデータベース認証を使用してAmazon RDS（PostgreSQL）データベースに接続](/gateway/latest/kong-enterprise/aws-iam-auth-to-rds-database/)できるようになりました。
* Kong Manager:
  * Kong ManagerとKonnectは、ナビゲーションー、サイドバー、すべてのエンティティリストで同じUIを共有するようになりました。
  * 式ルーターが有効な場合のルート一覧の表示が改善されました。
  * **CA証明書** と **TLS検証** がKong Gatewayサービスフォームでサポートされるようになりました。
  * フリーモードのナビゲーションバーにGitHubスターを追加しました。
  * Konnect CTAをフリーモードでアップグレードしました。

* SPDXとCycloneDXのSBOMファイルが、Kong GatewayのDockerイメージ用に生成されるようになりました。

#### Kong Gateway・Kong Konnect付き

* [データプレーンのラベル](/konnect/runtime-manager/runtime-instances/custom-dp-labels/)を構成し、Konnectにメタデータ情報を提供できるようになりました。 [\#10471](https://github.com/Kong/kong/pull/10471)
* Kong Gateway DB\-lessモードからKonnectに分析を送信することがサポートされるようになりました。

#### コア

* `runloop` `init`エラー応答コンテンツタイプは、`Accept`ヘッダー値に準拠するようになりました。
  [\#10366](https://github.com/Kong/kong/pull/10366)

* カスタムエラーテンプレートを構成できるようになりました。
  [\#10374](https://github.com/Kong/kong/pull/10374)

* デフォルトで解析されるリクエストヘッダー、応答ヘッダー、URI引数、POST引数の最大数が、次の新しい構成パラメータで構成できるようになりました: [`lua_max_req_headers`](/gateway/latest/reference/configuration/#lua_max_req_headers)、[`lua_max_resp_headers`](/gateway/latest/reference/configuration/#lua_max_resp_headers)、[`lua_max_uri_args`](/gateway/latest/reference/configuration/#lua_max_uri_args)、および[`lua_max_post_args`](/gateway/latest/reference/configuration/#lua_max_post_args)。[\#10443](https://github.com/Kong/kong/pull/10443)

* コアエンティティとバンドルされたプラグインのエンティティに、有効期限の切れた行を効率的かつタイムリーに削除するPostgreSQLトリガーを追加しました。[\#10389](https://github.com/Kong/kong/pull/10389)

* 構成可能なノードIDのサポートが追加されました。
  [\#10385](https://github.com/Kong/kong/pull/10385)

* リクエストと応答のバッファリングオプションが、HTTP 2\.0の受信リクエストに対して有効になりました。

  この変更に貢献してくれた[@PidgeyBE](https://github.com/PidgeyBE)に感謝します。
  [\#10204](https://github.com/Kong/kong/pull/10204) [\#10595](https://github.com/Kong/kong/pull/10595)
* `KONG_UPSTREAM_DNS_TIME`を`kong.ctx`に追加し、Kongがアップストリームにプロキシする際にDNS解決にかかる時間を記録するようになりました。[\#10355](https://github.com/Kong/kong/pull/10355)

* 動的ログレベルのデフォルトのタイムアウトは60秒になりました。
  [\#10288](https://github.com/Kong/kong/pull/10288)

#### Admin API

* 次のエンティティに新しい`updated_at`フィールドが追加されました: `ca_certificates`、`certificates`、`consumers`、`targets`、`upstreams`、`plugins`、`workspaces`、`clustering_data_planes`、`consumer_group_consumers`、`consumer_group_plugins`、`consumer_groups`、`credentials`、`document_objects`、`event_hooks`、`files`、`group_rbac_roles`、`groups`、`keyring_meta`、`legacy_files`、`login_attempts`、`parameters`、`rbac_role_endpoints`、`rbac_role_entities`、`rbac_roles`、`rbac_users`、および`snis`。 [\#10400](https://github.com/Kong/kong/pull/10400)
* `/upstreams/<upstream id="sl-md0000000">/health?balancer_health=1`エンドポイントは常に、新しい属性`balancer_health`を介してバランサーの状態を表示します。これは常に`HEALTHY`または`UNHEALTHY`を返し、アップストリームの全体的なヘルスステータスが`HEALTHCHECKS_OFF`であっても、バランサーの実際の状態を報告します。これはデバッグに役立ちます。 [\#5885](https://github.com/Kong/kong/pull/5885)
* **ベータ版** : Kong Gateway Admin APIのOpenAPI仕様が利用可能になりました: 
  * [Kong Gateway Admin API \- OSS仕様](/gateway/api/admin-oss/3.3.x/)
  * [Kong Gateway Admin API \- Enterprise仕様](/gateway/api/admin-ee/3.3.0.x/)

#### Status API

* `status_listen`サーバーは、Kong Gatewayの状態を監視するための`/status/ready` APIが追加され、強化されました。このエンドポイントは、`GET`リクエストを受信すると`200`の応答を返します。ただし、有効かつ空でない構成が読み込まれ、Kong Gatewayがユーザーのリクエストを処理する準備が整っている場合に限られます。

  ロードバランサーはこの機能を頻繁に利用して、受信リクエストの配信にKong Gatewayが利用可能なことを確認します。
  [\#10610](https://github.com/Kong/kong/pull/10610)
  [\#10787](https://github.com/Kong/kong/pull/10787)
* **ベータ版** : [Kong Gateway Status API](/gateway/api/status/v1/)のOpenAPI仕様が利用可能になりました。

#### PDK

* PDKは、`kong.plugin.get_id`を使用してプラグインのIDを取得できるようになりました。 [\#9903](https://github.com/Kong/kong/pull/9903)
* トレーシングモジュール: トレーシングバックエンドでのフィルタリングを簡素化するためにスパンの名前を変更しました。詳細については[`kong.tracing`](/gateway/latest/plugin-development/pdk/kong.tracing/)を参照してください。 [\#10577](https://github.com/Kong/kong/pull/10577)

#### プラグイン

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * このプラグインは、`keys`および`key_sets`で`account_key`の構成をサポートするようになりました。 [\#9746](https://github.com/Kong/kong/pull/9746)
  * このプラグインは、Redisストレージの`namespace`の構成をサポートするようになりました。後方互換性により、デフォルトでは空の文字列になります。 [\#10562](https://github.com/Kong/kong/pull/10562)

* [**Proxy Cache**](/hub/kong-inc/proxy-cache/)（`proxy-cache`）

  * キャッシュキーのURIを小文字として処理できるように、構成パラメータ`ignore_uri_case`を追加しました。 [\#10453](https://github.com/Kong/kong/pull/10453)

* [**Proxy Cache Advanced**](/hub/kong-inc/proxy-cache-advanced/)（`proxy-cache-advanced`）

  * `content_type`にワイルドカードとパラメータ一致のサポートを追加しました。
  * キャッシュキーのURIを小文字として処理できるように、構成パラメータ`ignore_uri_case`を追加しました。 [\#10453](https://github.com/Kong/kong/pull/10453)

* [**HTTP Log**](/hub/kong-inc/http-log/)（`http-log`）

  * 文字セット宣言を必要とするログコレクターをサポートするため、`Content-Type`ヘッダーに`application/json; charset=utf-8`オプションを追加しました。 [\#10533](https://github.com/Kong/kong/pull/10533)

* [**Datadog**](/hub/kong-inc/datadog/)（`datadog`）

  * `host`構成パラメータが参照可能になりました。[\#10484](https://github.com/Kong/kong/pull/10484)

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）と[**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * これらのプラグインは、HTTP応答ヘッダーの`traceid`を16進形式に変換するようになりました。 [\#10534](https://github.com/Kong/kong/pull/10534)

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * ダウンストリームのDatadogトレースでスパンが正しく相関されるようになりました。 [10531](https://github.com/Kong/kong/pull/10531)
  * `header_type`フィールドを追加しました。以前は、`header_type`は`preserve`にハードコードされていました。 次のいずれかの値に設定できるようになりました: `preserve`、`ignore`、`b3`、`b3-single`、`w3c`、`jaeger`、または`ot`。 [\#10620](https://github.com/Kong/kong/pull/10620)
  * プロキシの背後にあるときにクライアントIPをキャプチャするために、新しいスパン属性`http.client_ip`を追加しました。 [\#10723](https://github.com/Kong/kong/pull/10723)
  * `http_response_header_for_traceid`構成パラメータを追加しました。このフィールドに文字列値を設定すると、応答内の対応するヘッダーが設定されます。 [\#10379](https://github.com/Kong/kong/pull/10379)

* [**AWS Lambda**](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * LambdaサービスAPIエンドポイントのスキーム構成をサポートするために、構成パラメータ`disable_https`を追加しました。 [\#9799](https://github.com/Kong/kong/pull/9799)

* [**Request Transformer Advanced**](/hub/kong-inc/request-transformer-advanced/)（`request-transformer-advanced`）

  * プラグインは、次のKong Gateway構成パラメータを尊重するようになりました: [`untrusted_lua`](/gateway/latest/reference/configuration/#untrusted_lua)、[`untrusted_lua_sandbox_requires`](/gateway/latest/reference/configuration/#untrusted_lua_sandbox_requires)、[`untrusted_lua_sandbox_environment`](/gateway/latest/reference/configuration/#untrusted_lua_sandbox_environment)。これらのパラメータは、高度なテンプレート（Lua式）に適用されます。

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * 検証に失敗した場合、エラーが記録されるようになりました。

* [**JWT Signer**](/hub/kong-inc/jwt-signer/)（`jwt-signer`）

  * JWTにクレームを追加できる構成フィールド`add_claims`を追加しました。

### 修正

#### Enterprise

* Kong Enterpriseのsystemdユニットは、3\.2\.xxバージョンで誤って`kong.service`に名前が変更されていました。以前のリリースとの一貫性を保つために、`kong-enterprise-edition.service`に戻されました。
* RBACが有効な場合にKong Gatewayがキーリングを生成できない問題を修正しました。
* FIPSモードの`lua_ssl_verify_depth`を通常モードと同じ深度に一致するように修正しました。
* 開発者登録の応答からメールフィールドを削除しました。
* Websocketリクエストでは、トレースが有効になっているときにバランサースパンが生成されるようになりました。
* 現在のライセンスが有効でない場合、`/licenses/`エンドポイントを介したライセンスの管理が失敗する問題を修正しました。
* 動的な再順序付けが適用されると、並べ替えが混同するプラグインイテレータの問題を解決しました。この修正により、すべてのシナリオで適切な並べ替え動作が保証されます。
* Kong Manager:
  * Kong ManagerでVault名を変更するとエラーがスローされる問題を修正しました。
  * 現在アクティブなタブを選択すると、垂直タブのコンテンツが空白になるというタブの問題を修正しました。
  * `/register`ルートが、代わりに`/login`にジャンプすることがあった問題を修正しました。
  * StatsDプラグインから **カスタム識別子** フィールドを削除しました。このフィールドはKong Managerのメトリクスの下に表示されていましたが、プラグインのスキーマにこのフィールドは存在しません。

#### Kong Gateway・Kong Konnect付き

* 標準の期限切れライセンス通知は、Konnectモード（`konnect_mode=on`）で実行されているデータプレーンには適用されないため、ログに表示されなくなりました。
* Konnectモードで実行されているデータプレーンの新しいライセンスアラートの動作: 
  * 有効期限まで16日以上ある場合、アラートは発行されません。
  * ライセンスの有効期限が16日以内に切れる場合、警告レベルのアラートが毎日発行されます。
  * ライセンスの有効期限が切れると、重大レベルのアラートが毎日発行されます。

#### コア

* アップストリームのKeepAliveプールでCRC32衝突が発生する問題を修正しました。 [\#9856](https://github.com/Kong/kong/pull/9856)
* ハイブリッドモード： 
  * コントロールプレーンが、古いバージョンのデータプレーンのAWS LambdaおよびZipkinプラグインの構成をダウングレードしない問題を修正しました。[\#10346](https://github.com/Kong/kong/pull/10346)
  * コントロールプレーンが、古いバージョンのデータプレーンのSessionプラグインのフィールド名を正しく変更しない問題を修正しました。[\#10352](https://github.com/Kong/kong/pull/10352)

* DB\-lessのKong Gatewayに旧式の構成スタイルが使用されている場合に、正規表現ルートの検証が時々スキップされる問題を修正しました。 [\#10348](https://github.com/Kong/kong/pull/10348)
* トレースによって予期しない動作が発生する可能性がある問題を修正しました。 [\#10364](https://github.com/Kong/kong/pull/10364)
* Kong Gatewayが`header_filter`フェーズでアップストリームからステータスコードを変更したときに、バランサーのパッシブヘルスチェックが間違ったステータスコードを使用する問題を修正しました。 [\#10325](https://github.com/Kong/kong/pull/10325) [\#10592](https://github.com/Kong/kong/pull/10592)
* ネストされたレコードでスキーマ検証が失敗してもエラーが正しく伝達されない問題を修正しました。 [\#10449](https://github.com/Kong/kong/pull/10449)
* ダングリング状態のUnixソケットが正常に停止されなかった場合に、Kong GatewayがDockerコンテナで再起動できなくなる問題を修正しました。 [\#10468](https://github.com/Kong/kong/pull/10468)
* 従来のルーターの送信元または宛先のソート機能が`invalid order function for sorting`エラーになる問題を修正しました。 [\#10514](https://github.com/Kong/kong/pull/10514)
* 頻繁なDNSクエリによって発生する`resty.dns.client`のUDPソケットリークを修正しました。 [\#10691](https://github.com/Kong/kong/pull/10691)
* mlcacheオプション`shm_set_tries`のタイプミスを修正しました。 [\#10712](https://github.com/Kong/kong/pull/10712)
* Goプラグインサーバーの起動が遅く、デッドロックが発生する問題を修正しました。[\#10561](https://github.com/Kong/kong/pull/10561)
* トレーシング： 
  * 受信伝播ヘッダーの`sampled`フラグが間違って処理され、一部のスパンのみに影響する原因となっていた問題を修正しました。 [\#10655](https://github.com/Kong/kong/pull/10655)
  * `http_client`スパンがOpenResty HTTPクライアントリクエスト用に作成されない問題を修正しました。 [\#10680](https://github.com/Kong/kong/pull/10680)
  * バランサースパンの開始時間と終了時間の精度が低下する近似値の問題を修正しました。 [\#10681](https://github.com/Kong/kong/pull/10681)
  * `tracing_sampling_rate`のデフォルトは、以前の1（すべてのリクエストをトレース）ではなく、0\.01（100のリクエストごとに1つトレース）になりました。すべてのリクエストをトレースすると、ほとんどのプロダクションシステムで不必要なリソースが浪費されます。 [\#10774](https://github.com/Kong/kong/pull/10774)

* 停止しようとするとKong Gatewayがエラーになる原因となっていた、Vault参照の問題を修正しました。 [\#10775](https://github.com/Kong/kong/pull/10775)
* 構成が変更された場合でも、Vault構成が固定されキャッシュされたままになる問題を修正しました。[\#10776](https://github.com/Kong/kong/pull/10776)
* 次のPostgreSQL TTLクリーンアップタイマーの問題を修正しました: 
  * タイマーは、Admin APIを有効にした従来のノードとコントロールプレーンノードでのみ動作するようになりました。[\#10405](https://github.com/Kong/kong/pull/10405)
  * Kong Gatewayは、バッチごとに`50.000`行のバッチ削除ループを、TTL対応の各テーブルで実行するようになりました。 [\#10407](https://github.com/Kong/kong/pull/10407)
  * クリーンアップジョブは、60秒ごとにではなく、5分ごとに実行されるようになりました。 [\#10389](https://github.com/Kong/kong/pull/10389)
  * Kong Gatewayは、データベースサーバー側のタイムスタンプに基づいて、期限切れの行を削除するようになりました。これは、Kong Gatewayとデータベースサーバー間の時計時間の違いによって発生する潜在的な問題を回避するためです。 [\#10389](https://github.com/Kong/kong/pull/10389)

* URI引数`custom_id`の値が空の場合に`/consumer`APIがクラッシュする問題を修正しました。 [\#10475](https://github.com/Kong/kong/pull/10475)

#### PDK

* `request.get_uri_captures`は、jsonificationの配列としてタグ付けされた名前のない部分を返すようになりました。 [\#10390](https://github.com/Kong/kong/pull/10390)
* サンプリングレートが機能しないPDKのトレースの問題を修正しました。 [\#10485](https://github.com/Kong/kong/pull/10485)

#### プラグイン

* [**JWE Decrypt**](/hub/kong-inc/jwe-decrypt/)（`jwe-decrypt`）、[**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）、[**Vault Authentication**](/hub/kong-inc/vault-auth/)（`vault-auth`）

  * `jwe-decrypt`、`oas-validation`、および`vault-auth`に不足していたスキーマフィールド`protocols`を追加しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)

  * `redis`流量制限ストラテジでは、Redisクラスタがダウンした場合にエラーが返されるようになりました。
  * 流量制限`cluster_events`が従来のクラスタモードで間違ったデータをブロードキャストする問題を修正しました。
  * コントロールプレーン（CP）は名前空間を作成したり同期したりしなくなりました。

* [**StatsD Advanced**](/hub/kong-inc/statsd-advanced/)（`statsd-advanced`）

  * プラグインの名前を`statsd`から`statsd-advanced`に変更しました。

* [**LDAP Authentication Advanced**](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * プラグインは認可の前に認証を実行し、ユーザーが認可されたグループに属していない場合は403 HTTPコードを返すようになりました。
  * プラグインは、グループが空でない場合にグループを空の配列に設定できるようになりました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * プラグインの再構成が有効にならない問題を修正しました。
  * スパンが正しく伝達されず、トレーシングバックエンドで間違った階層がレンダリングされる問題を修正しました。 [\#10663](https://github.com/Kong/kong/pull/10663)

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * `allowed_content_types`パラメータの検証機能が厳格すぐるため、`-`文字を含むメディアタイプを使用できない問題を修正しました。

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）

  * ロギングプラグインで間違った`latencies.proxy`が使用される問題を修正しました。 このプラグインは、次のフェーズではなく、フォワードプロキシで`ctx.WAITING_TIME`を評価するようになりました。

* [**Request Termination**](/hub/kong-inc/request-termination/)（`request-termination`）

  * プラグインが`uri-captures`を返さない原因となっていた`echo`オプションの問題を修正しました。 [\#10390](https://github.com/Kong/kong/pull/10390)

* [**Request Transformer**](/hub/kong-inc/request-transformer/)（`request-transformer`）

  * リクエストが不正なクエリパラメータで断続的にプロキシされる問題を修正しました。 [10539](https://github.com/Kong/kong/pull/10539)
  * プラグインは、`untrusted_lua`構成パラメータの値を尊重するようになりました。 [\#10327](https://github.com/Kong/kong/pull/10327)

* [**OAuth2**](/hub/kong-inc/oauth2/)（`oauth2`）

  * 間違ったサービスに最初にアクセスした場合にOAuth2トークンが`nil`としてキャッシュされる問題を修正しました。 [\#10522](https://github.com/Kong/kong/pull/10522)
  * このプラグインは、あるプラグインインスタンスで作成された認証コードが、別のプラグインインスタンスで作成されたアクセストークンと交換されるのを防ぐようになりました。 [\#10011](https://github.com/Kong/kong/pull/10011)

* [**gRPC Gateway**](/hub/kong-inc/grpc-gateway/)（`grpc-gateway`）

  * JSONペイロードに`null`値があると、`pb.encode`中にキャッチされない例外がスローされる問題を修正しました。 [\#10687](https://github.com/Kong/kong/pull/10687)
  * JSON内の空の配列が誤って`"{}"`としてエンコードされる問題を修正しました。標準に準拠するために、現在は`"[]"`としてエンコードされます。 [\#10790](https://github.com/Kong/kong/pull/10790)

### 依存関係

* 次の問題を修正するために、データファイルライブラリの依存関係を更新しました: 
  * Kong Gatewayは、読み取り専用ファイルシステムにインストールした場合に動作しませんでした。
  * Kong Gatewayは、systemdから起動すると動作しませんでした。

* `lua-resty-session`を4\.0\.2から4\.0\.3に更新しました [\#10338](https://github.com/Kong/kong/pull/10338)
* `lua-protobuf`を0\.3\.3から0\.5\.0に更新しました [\#10137](https://github.com/Kong/kong/pull/10413) [\#10790](https://github.com/Kong/kong/pull/10790)
* `lua-resty-timer-ng`を0\.2\.3から0\.2\.5に更新しました [\#10419](https://github.com/Kong/kong/pull/10419) [\#10664](https://github.com/Kong/kong/pull/10664)
* `lua-resty-openssl`を0\.8\.17から0\.8\.20にアップグレードしました [＃10463](https://github.com/Kong/kong/pull/10463) [＃10476](https://github.com/Kong/kong/pull/10476)
* `lua-resty-http`を0\.17\.0\.beta.1から0\.17\.1にアップグレードしました [＃10547](https://github.com/Kong/kong/pull/10547)
* `lua-resty-aws`を1\.1\.2から1\.2\.2に更新しました
* `lua-resty-gcp`を0\.0\.11から0\.0\.12に更新しました
* `LuaSec`を1\.2\.0から1\.3\.1に更新しました [\#10528](https://github.com/Kong/kong/pull/10528)
* `lua-resty-acme`を0\.10\.1から0\.11\.0に更新しました [\#10562](https://github.com/Kong/kong/pull/10562)
* `lua-resty-events`を1\.1\.3から0\.1\.4にアップグレードしました [\#10634](https://github.com/Kong/kong/pull/10634)
* `lua-kong-nginx-module`を0\.5\.1から0\.6\.0にアップグレードしました [\#10288](https://github.com/Kong/kong/pull/10288)
* `lua-resty-lmdb`を1\.0\.0から1\.1\.0に更新しました [\#10766](https://github.com/Kong/kong/pull/10766)
* `kong-openid-connect`を2\.5\.4から2\.5\.5に更新しました

### 既知の問題

* 既知の問題のため、Kongのバージョン3\.0\.x～3\.3\.xではページレベルのLMDB暗号化を有効にしないことをお勧めします。

  `declarative_config_encryption_mode`を設定せず、デフォルト値の`off`のままにしておきます。ディスク上の構成を暗号化するには、引き続きディスクレベルの暗号化を使用します。
* DB\-lessモードで実行し`flatten_errors=1`が設定されている場合に、無効な構成を`/config`エンドポイントに送信すると、Kong Gatewayは誤って500を返します。
  構成が無効であるため、これは400になるはずです。

* OpenID Connect（OIDC）プラグインが`config.client_secret`フィールドでHashiCorp Vaultを参照するように構成されている場合（`{vault://hcv/clientSecret}`など）、シークレットを正しく検索しません。

3\.2\.2\.5
-------------

**リリース日** 2023/10/12

### 修正

#### コア

* HTTP/2ストリームリセット攻撃を早期に検出するためのNginxパッチを適用しました。
  この変更は、特定された脆弱性[CVE\-2023\-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)に直接対処するものです。

  この脆弱性とそれに対するKongの対処に関する詳細については、当社の[ブログ記事](https://konghq.com/blog/product-releases/novel-http2-rapid-reset-ddos-vulnerability-update)をご覧ください。
* クラスタストラテジの使用時に、Kong Gatewayノードがキーリングデータの送信に失敗するというキーリングの問題を修正しました。

* PostgreSQLデータベースにクエリを実行すると、異常なソケット接続が誤って再利用される問題を修正しました。

* systemdユニット定義に`User=`仕様を追加し、Kong Gatewayが再びsystemdによって制御できるようになりました。[\#11066](https://github.com/Kong/kong/pull/11066)

#### プラグイン

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）: 証明書失効チェックの実行時にプラグインがネットワーク障害をキャッシュする問題を修正しました。

* [**SAML**](/hub/kong-inc/saml/)（`saml`）: Redisセッションストレージが正しく構成されていない場合、ユーザーは、無限にリダイレクトされるのではなく、500エラーを受け取るようになりました。

### 依存関係

* `libxml2`を2\.10\.2から2\.11\.5に更新しました

3\.2\.2\.4
-------------

**リリース日** 2023/09/15

### 下位互換性のない変更と非推奨

* **Ubuntu 18\.04サポートの削除** : Ubuntu 18\.04（"Bionic"）でのKong Gatewayの実行のサポートが非推奨となりました。[Ubuntu 18\.04の標準サポートは2023年6月をもって終了しました](https://wiki.ubuntu.com/Releases)。
  Kong Gateway 3\.2\.2\.4以降、Kongは新しいUbuntu 18\.04イメージまたはパッケージをビルドしていません。また、KongはUbuntu 18\.04でのパッケージのインストールをテストしません。

  Kong GatewayをUbuntu 18\.04にインストールする必要がある場合は、[インストール手順](/gateway/3.2.x/install/linux/ubuntu/)にある以前の3\.2\.xパッチバージョンに置き換えます。

<!-- -->
* Amazon Linux 2022アーティファクトは、AWS独自の名前変更に基づいて、Amazon Linux 2023に名前が変更されます。
* CentOSパッケージはリリースから削除され、将来のバージョンではサポートされなくなりました。

### 修正

#### Enterprise

* Kongがsystemdによって制御されている場合にSAMLプラグインが再び動作するよう、データファイルライブラリを更新しました。
* `anonymous_reports=false`を設定しても匿名レポートを非表示にできない問題を修正しました。
* Goプラグインサーバープロセスがクラッシュすると、Kongを介してプロキシされた後続のリクエストが、一貫性のない構成でGoプラグインを実行する問題を修正しました。この問題は、同じGoプラグインが異なるルートまたはサービスエンティティに適用されている場合にのみ影響していました。

#### プラグイン

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）
  * `log`と`message`に正しいテーブルキーを設定しました。
  * 無効で不透明なトークンが提供されたものの検証に失敗した場合、エラーが正しく出力されるようになりました。

* [**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）
  * Redis流量制限ストラテジでは、Redisクラスタがダウンした場合にエラーが返されるようになりました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * コントロールプレーン（CP）は、名前空間の作成や、Redisとカウンターの同期を試みることがなくなりました。

* [**Response Transformer Advanced**](/hub/kong-inc/response-transformer-advanced/)（`response-transformer-advanced`） 
  * `if_status`が一致しない場合、応答本文はロードされません。

#### Kong Manager

* Zipkinプラグインによってユーザーが`static_tags`構成を編集できなくなる問題を修正しました。
* プラグインのインストールページに、使用できないDatadog Tracingプラグインが表示される問題を修正しました。
* StatsDプラグインから一部のメトリクスが欠落していた問題を修正しました。
* デフォルト以外の`admin_gui_path`構成を使用しているときに、ロケールファイルが見つからない問題を修正しました。
* アプリケーションインスタンスのエンドポイント権限が期待どおりに機能しない問題を修正しました。
* 一部のアイコンが判読できない記号や文字として表示される問題を修正しました。
* 他のワークスペースにあるエンティティのサービスまたはルートのリンクをクリックすると、ユーザーがデフォルトのワークスペースの下のページにリダイレクトされる問題を修正しました。
* Kong Managerで間違ったユーザー名が指定された場合、OpenID Connectをリダイレクトできない問題を修正しました。

### 依存関係

* `lua-resty-kafka`を0\.15から0\.16に更新しました
* `OpenSSL`を1\.1\.1tから3\.1\.1に更新しました 

3\.2\.2\.3
-------------

**リリース日** 2023/06/07

### 修正

* `/config`エンドポイントのエラーを修正しました。`flatten_errors=1`が設定されていて、無効な構成がエンドポイントに送信された場合、誤って500エラーが返されていました。

### 非推奨

* **Alpine非推奨のお知らせ:** KongはAlpineイメージとパッケージのサポートを今年中に終了する意向を発表しました。これらのイメージとパッケージは3\.2で利用可能であり、3\.3でも引き続き利用可能になります。Kong Gateway 3\.4では、Alpineイメージとパッケージのビルドを停止します。

3\.2\.2\.2
-------------

**リリース日** 2023/05/19

### 修正

#### コア

* チャンクエンコードされた応答データの破損につながっていた、OpenResty `ngx.print`チャンクエコーディングの重複した空きバッファの問題を修正しました。 [\#10816](https://github.com/Kong/kong/pull/10816) [\#10824](https://github.com/Kong/kong/pull/10824)
* 頻繁なDNSクエリによって発生する`resty.dns.client`のUDPソケットリークを修正しました。 [\#10691](https://github.com/Kong/kong/pull/10691)

#### プラグイン

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * `sync_rate`設定が低いために発生するログフラッディングの問題を修正しました。

3\.2\.2\.1
-------------

**リリース日** 2023/04/03

### 修正

* Dynatraceの実装を修正しました。ビルドシステムの問題により、Kong Gateway 3\.2\.xパッケージのうち3\.2\.2\.1より前のものには、Dynatraceに必要なデバッグシンボルが含まれていませんでした。

### 非推奨

* **Alpine非推奨のお知らせ:** KongはAlpineイメージとパッケージのサポートを今年中に終了する意向を発表しました。これらのイメージとパッケージは3\.2で利用可能であり、3\.3でも引き続き利用可能になります。Kong Gateway 3\.4では、Alpineイメージとパッケージのビルドを停止します。

3\.2\.2\.0
-------------

**リリース日** 2023/03/22

### 修正

#### Enterprise

* Kong 3\.2\.1\.0と3\.2\.1\.1で、`alpine`および`ubuntu` ARM64アーティファクトがHTTP/2リクエストを誤って処理し、プロトコルが失敗していました。これらのアーティファクトは削除されました。
* デフォルトのlogrotateファイル`/etc/logrotate.d/kong-enterprise-edition`を追加しました。このファイルは、このリリースより前のKong Gatewayのすべての3\.xバージョンには存在していませんでした。

#### プラグイン

* [**SAML**](/hub/kong-inc/saml/)（`saml`）

  * SAMLプラグインが読み取り専用ファイルシステムで動作するようになりました。
  * SAMLプラグインは、フィールド`session_auth_ttl`を処理できるようになりました（3\.2\.0\.0以降では削除されていました）。

* Datadog Tracingプラグイン: Datadog Tracingプラグインに最新の問題がいくつか見つかったため、3\.2リリースから削除することにしました。今後のリリースで、問題を修正したプラグインを再度追加する予定です。

### 既知の問題

* GPGキーの変更により、yumを使用してこのリリースをインストールすると`Public key for kong-enterprise-edition-3.2.1.0.rhel7.amd64.rpm is not installed`エラーをトリガーします。パッケージは *署名されていますが* 、メタデータサービスとは異なる（ローテーションされた）キーで署名されているため、yumでエラーが発生します。このエラーを回避するには、[download.konghq.com](https://download.konghq.com/)からパッケージを手動でダウンロードしてインストールしてください。

3\.2\.1\.0
-------------

**リリース日** 2023/02/28

### 非推奨

* 非推奨のAlpine Linuxイメージとパッケージ。

  Kongは、Alpineイメージとパッケージのサポートを今年後半に終了する意向を発表しています。これらのイメージとパッケージは3\.2で利用可能であり、3\.3でも引き続き利用可能になります。Kong Gateway 3\.4では、Alpineイメージとパッケージのビルドを停止します。

### 最新の変更

* デフォルトのPostgreSQL SSLバージョンをTLS 1\.2に更新しました。`kong.conf`の場合:

  * デフォルトの[`pg_ssl_version`](/gateway/latest/reference/configuration/#postgres-settings)が`tlsv1_2`になりました。
  * この構成オプションの有効な値が、次の値のみを受け入れるように制限しました: `tlsv1_1`、`tlsv1_2`、`tlsv1_3`、または`any`。

  上記には、PostgreSQL 12\.x以降での設定`ssl_min_protocol_version`が反映されます。該当パラメータの詳細については、[PostgreSQLドキュメント](https://postgresqlco.nf/doc/en/param/ssl_min_protocol_version/)を参照してください。

  `kong.conf`のデフォルト設定を使用するには、PostgresサーバーがTLS 1\.2以降のバージョンをサポートしていることを確認するか、TLSのバージョンを自分で設定します。`tlsv1_2`より前のTLSバージョンはすでに非推奨になっており、PostgreSQL 12\.x以降では安全ではないと見なされます。
* デバッグのために`Kong-Debug`ヘッダーを制限できるよう、[`allow_debug_header`](/gateway/latest/reference/configuration/#allow_debug_header)構成プロパティを`kong.conf`に追加しました。このオプションのデフォルトは`off`です。

  これまで`Kong-Debug`ヘッダーを使用してデバッグ情報を提供していた場合は、`allow_debug_header: on`を設定して引き続きデバッグ情報を入手してください。
* [**JWTプラグイン**](/hub/kong-inc/jwt/)（`jwt`）

  * JWTプラグインは、JWTトークンの検索場所に異なるトークンがあるリクエストをすべて拒否するようになりました。[\#9946](https://github.com/Kong/kong/pull/9946)

* セッションライブラリのアップグレード [\#10199](https://github.com/Kong/kong/pull/10199):

  * [`lua-resty-session`](https://github.com/bungle/lua-resty-session)ライブラリがv4\.0\.0にアップグレードされました。
    このバージョンには、Sessionライブラリの完全なリライトが含まれており、後方互換性はありません。

    このライブラリは、[**Session**](/hub/kong-inc/session/)、[**OpenID Connect**](/hub/kong-inc/openid-connect/)、[**SAML**](/hub/kong-inc/saml/)の各プラグインによって使用されます。これは、Kong ManagerおよびDev Portalのセッションを含む、バックグラウンドでSessionまたはOpenID Connectプラグインを使用するすべてのセッション構成にも影響します。

    このバージョンにアップグレードすると、既存のセッションはすべて無効になります。このバージョンでセッションが期待どおりに動作するには、すべてのノードでKong Gateway 3\.2\.xまたはそれ以降を実行する必要があります。そのためアップグレード中は、バージョンが混在するプロキシノードをできるだけ実行しないことをお勧めします。その間、無効なセッションによってエラーが発生し、部分的なダウンタイムが発生する可能性があります。
  * パラメータ:

    * `cookie_lifetime`を置き換える新しいパラメータ`idling_timeout`のデフォルト値は900です。違う設定をしないかぎり、セッションはアイドリング状態が900秒（15分）続くと期限切れになります。
    * 新しいパラメータ`absolute_timeout`のデフォルト値は86400です。違う設定をしないかぎり、セッションは86400秒（24時間）後に期限切れになります。
    * 多くのセッションパラメータの名前が変更されたか、削除されました。構成は以前の構成どおりに引き続き機能しますが、将来の予期しない動作を回避するために、構成を調整することをお勧めします。すべてのセッション構成の変更と既存のセッション構成を変換する方法のガイダンスについては、[3\.2のアップグレードガイド](/gateway/latest/upgrade/#session-library-upgrade)を参照してください。

### 機能

* 便宜上、基盤となるオペレーティングシステム（OS）のDockerタグ（`latest`、`3.2.1.0`、`3.2`など）をDebianからUbuntuに変更しました。

#### コア

* `router_flavor`が`traditional_compatible`に設定されている場合、Kong Gatewayは従来のルーターでなく、式ルーターを使用して作成されたルートを検証し、作成されたルートに互換性があることを確認します。 [\#9987](https://github.com/Kong/kong/pull/9987)
* `/config` APIエンドポイントはDBレスモードで`flatten_errors`クエリパラメータを使用して、すべてのスキーマ検証エラーを単一配列にフラット化できるようになりました。 [\#10161](https://github.com/Kong/kong/pull/10161)
* アップストリームのエンティティに[`latency`](/gateway/latest/how-kong-works/load-balancing/#balancing-algorithms)という新しいロードバランシングアルゴリズムオプションが追加されました。 このアルゴリズムは、以前のリクエストからの各ターゲットの応答レイテンシに基づいてターゲットを選択します。 [\#9787](https://github.com/Kong/kong/pull/9787)
* Nginx `charset`ディレクティブをNginxディレクティブ投入で構成できるようになりました。 Kong Gatewayの構成で[`nginx_http_charset`](/gateway/latest/reference/configuration/#nginx_http_charset)を使用して設定します [＃10111](https://github.com/Kong/kong/pull/10111)
* サービスのアップストリームTLS構成がストリームサブシステムに拡張されました。 [\#9947](https://github.com/Kong/kong/pull/9947)
* 新たに追加された構成パラメータ[`ssl_session_cache_size`](/gateway/latest/reference/configuration/#ssl_session_cache_size)で、Nginxディレクティブ`ssl_session_cache`を設定できるようになりました。この設定パラメータはデフォルトでは`10m`です。この変更に貢献してくれた[Michael Kotten](https://github.com/michbeck100)さんに感謝します。 [\#10021](https://github.com/Kong/kong/pull/10021)
* [`status_listen`](/gateway/latest/reference/configuration/#status_listen)はHTTP2をサポートするようになりました。[\#9919](https://github.com/Kong/kong/pull/9919)
* 共有Redisコネクタは、クラスタ接続のユーザー名\+パスワード認証をサポートするようになり、既存の単一ノード接続サポートが改善されました。これは、共有Redis構成を使用するすべてのプラグインに自動的に適用されます。

#### Enterprise

* **FIPSのサポート** ：
  * OpenID Connect、Key Authentication \- Encrypted、およびJWT Signerプラグインは、[FIPS 140\-2に準拠](/gateway/latest/kong-enterprise/fips-support/)するようになりました。

    FIPSモードで{{site.base_gateway}} 3\.1から3\.2へ移行中に、`key-auth-enc`プラグインを使用している場合、既存のすべての`key-auth-enc`認証情報に[PATCHまたはPOSTリクエスト](/hub/kong-inc/key-auth-enc/#create-a-key)を送信して、SHA256で再ハッシュします。
  * FIPS準拠のKong Gatewayパッケージは、PostgreSQL SSL接続をサポートするようになりました。

##### Kong Manager

* 式フィールドのエディターを改善しました。式ルーターを使用するすべてのフィールドに、構文ハイライト、自動補完、ルート検証が追加されました。
* `rbac_user_name`と`request_source`を追加して監査ログを改善しました。 新しい`request_source`フィールドと`path`フィールドのデータを組み合わせることで、ログインとログアウトイベントをログから判断できるようになりました。 [監査ログの解釈](/gateway/latest/kong-enterprise/audit-log/#kong-manager-authentication)の詳細については、ドキュメントを参照してください。
* ライセンス情報をKong Managerからファイルにコピーまたはダウンロードできるようになりました。
* Kong Managerは、OIDCベースの認証に`POST`メソッドをサポートするようになりました。
* キーおよびキーセットをKong Managerで構成できるようになりました。
* `http`メソッドバッジの配色を最適化しました。

#### プラグイン

* **プラグインエンティティ** ：特定のプラグインエンティティを識別するオプションフィールド、`instance_name`を追加しました。
  [\#10077](https://github.com/Kong/kong/pull/10077)

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）

  * Kongフェーズの期間を構成プロパティ、`phase_duration_flavor`を通じてスパンタグとして設定するためのサポートを追加しました。 [\#9891](https://github.com/Kong/kong/pull/9891)

* [**HTTP Log**](/hub/kong-inc/http-log/)（`http-log`）

  * `headers`構成パラメータが参照可能となり、Vaultに安全に保存できるようになりました。 [\#9948](https://github.com/Kong/kong/pull/9948)

* [**AWS Lambda**](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * 構成パラメータ、`aws_imds_protocol_version`を追加して、IMDSプロトコルのバージョンを選択できるようになりました。 このオプションのデフォルトは`v1`ですが、`v2`に設定するとIMDSv2を有効にできます。 [\#9962](https://github.com/Kong/kong/pull/9962)

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * このプラグインは、個々のサービス、ルート、コンシューマにスコープを設定できるようになりました。 [\#10096](https://github.com/Kong/kong/pull/10096)

* [**StatsD**](/hub/kong-inc/statsd/)（`statsd`）

  * `tag_style`構成パラメータに追加し、プラグインが[タグ](https://github.com/prometheus/statsd_exporter#tagging-extensions)を使用してメトリクスを送信できるようになりました。 デフォルトパラメータの`nil`は、メトリクスに追加されているタグがないことを示します。 [\#10118](https://github.com/Kong/kong/pull/10118)

* [**Session**](/hub/kong-inc/session/)（`session`）、[**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）、[**SAML**](/hub/kong-inc/saml/)（`saml`）

  * これらのプラグインは、`lua-resty-session` v4\.0\.0を使用するようになりました。

    この更新には、単一のCookie、グローバルタイムアウト、永続Cookie内などで複数のセッションをオーディエンスが管理できるような新しいセッション機能が含まれています。

    この更新により、これらのプラグインには非推奨になったり削除されたりしたパラメータも多数あります。各プラグインで変更されるパラメータの完全な一覧については、個々のプラグインのドキュメントを参照してください。
    * [Sessionの変更履歴](/hub/kong-inc/session/#changelog)
    * [OpenID Connectの変更履歴](/hub/kong-inc/openid-connect/#changelog)
    * [SAMLの変更履歴](/hub/kong-inc/saml/#changelog)

* [**GraphQL Rate Limiting Advanced**](/hub/kong-inc/graphql-rate-limiting-advanced/)（`graphql-rate-limiting-advanced`）および[**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * これらのプラグインは、ハイブリッドモードとDBレスモードで、デフォルトの`cluster`ストラテジを含む任意のストラテジを使用して`sync_rate = -1`をサポートするようになりました。

* [**OPA**](/hub/kong-inc/opa/)（`opa`）

  * このプラグインでOPAサーバーからのカスタムメッセージを処理できるようになりました。

* [**Canary**](/hub/kong-inc/canary/)（`canary`）

  * カナリアプラグインで`start`フィールドにデフォルト値を追加しました。 設定されていない場合、開始時刻はデフォルトで現在のタイムスタンプになります。

* **プラグインのドキュメントの改善** 

  * プラグインの互換性テーブルを、[技術互換性ページ](/hub/plugins/compatibility/)と[ライセンスティア](/hub/plugins/license-tiers/)ページに分割しました。
  * [サポートされるネットワークプロトコル](/hub/plugins/compatibility/#protocols)と[エンティティスコープ](/hub/plugins/compatibility/#scopes)がより明確になるようにプラグインの互換性情報を更新しました。
  * 次のプラグインのドキュメントに例を追加する改訂を行いました。 
    * [CORS](/hub/kong-inc/cors/)
    * [File Log](/hub/kong-inc/file-log/)
    * [HTTP Log](/hub/kong-inc/http-log/)
    * [JWT Signer](/hub/kong-inc/jwt-signer/)
    * [Key Auth](/hub/kong-inc/key-auth/)
    * [OpenID Connect](/hub/kong-inc/openid-connect/)
    * [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)
    * [SAML](/hub/kong-inc/saml/)
    * [StatsD](/hub/kong-inc/statsd/)

### 修正

#### コア

* `ttl`の計算時にPostgreSQL `FLOOR`関数を再び追加して、`ttl`が常に整数として返されるようにしました。 [\#9960](https://github.com/Kong/kong/pull/9960)
* PostreSQL接続プールの構成を公開しました。 [\#9603](https://github.com/Kong/kong/pull/9603)
* **Nginx** ：アップストリーム応答にデフォルトの文字セットが含まれていない場合、この文字セットは`Content-Type`応答ヘッダーに追加されなくなりました。[\#9905](https://github.com/Kong/kong/pull/9905)
* 有効な宣言型構成の読み込み後に構成ハッシュの値が`00000000000000000000000000000000`に誤って設定される問題を修正しました。 [\#9911](https://github.com/Kong/kong/pull/9911)
* バッチキューモジュールを更新し、コンシューマがエントリーの処理に失敗しても、キューが際限なく増大しないようにしました。現在、古いバッチは削除され、エラーが記録されるようになっています。[\#10247](https://github.com/Kong/kong/pull/10247)
* 応答がバッファリングされる場合、`X-Kong-Upstream-Status`が発行されない問題を修正しました。 [\#10056](https://github.com/Kong/kong/pull/10056)
* 無効なJWKエントリのエラーメッセージを改善しました。[\#9904](https://github.com/Kong/kong/pull/9904)
* 環境変数とVault参照から`#`文字が正しく解析されない問題を修正しました。 [10132](https://github.com/Kong/kong/pull/10132)
* コントロールプレーン（CP）でAWS LambdaおよびZipkinプラグインの構成が古いバージョンのデータプレーン（DP）向けにダウングレードされない問題を修正しました。 [\#10346](https://github.com/Kong/kong/pull/10346)
* `3.0`より古い構成フォーマットの使用時に、正規表現ルートの検証がスキップされるDBレスモードでの問題を修正しました。 [\#10348](https://github.com/Kong/kong/pull/10348)

#### Enterprise

* データプレーン（DP）とコントロールプレーン（CP）の間の転送プロキシがテレメトリポート8006をサポートしない問題を修正しました。
* PostgreSQL mTLSエラー、`bad client cert type`を修正しました。
* Admin APIの`/licenses`エンドポイントに関する次の問題を修正しました。 
  * Enterpriseライセンスがクラスタ内の他のノードで選択されていませんでした。
  * Vitalsのルートにアクセスできませんでした。
  * ハイブリッドモードでVitalsが表示されませんでした。

* RBACに関する次の問題を修正しました。 
  * ワークスペース管理者がコンシューマグループに流量制限ポリシーを追加できない問題を修正しました。
  * あるワークスペースの管理者が他のワークスペースの管理者権限を有していた問題を修正しました。 現在、ワークスペースの管理者権限は、原則どおり自分のワークスペースのみに制限されています。
  * RBACのロールの優先順位に関する問題を修正しました。現在は、正しい順番どおり、拒否（ネガティブ）ルールを含むRBACルールが、許可（非ネガティブ）ロールよりも優先されるようになりました。

##### Vitals

* Vitalsでサービスレスルートのステータスコードが追跡されない問題を修正しました。
* Admin APIエラー、`/vitals/reports/:entity_type is not available`を修正しました。

##### Kong Manager

* スコープが設定されているプラグインにバインドされたサービス、ルート、またはコンシューマの更新中に、`404 Not Found`エラーがトリガーされる問題を修正しました。
* `tags`フィールドが、証明書、ルート、アップストリーム構成の各ページの高度なフィールドセクションの外部に移動しました。 現在、タグフィールドは、展開してすべてのフィールドを表示しなくても、表示されます。
* キーとキーセットエンティティ向けのユーザーインターフェースを改善しました。
* Kong Managerでコンシューマグループにタグを追加できるようになりました。
* プラグインの **JSONをコピー** ボタンをクリックしても、すべての構成がコピーされない問題を修正しました。
* パスワードリセットフォームで一致するパスワードがチェックされず、一致しないパスワードの送信が許可される問題を修正しました。
* KonnectまたはEnterpriseのアップグレードプロンプトへのリンクを追加しました。
* IdPユーザーが、ロールやグループメンバーシップにかかわらず、Kong Managerにログインできる問題を修正しました。 従来これらのユーザーは、デフォルトのワークスペースを含むワークスペースの概要ダッシュボードを閲覧できましたが、他の操作はできませんでした。 現在は、帰属するグループやロールがないIdPユーザーがKong Managerにログインしようとすると、アクセスが拒否されるようになっています。

#### プラグイン

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）

  * グローバルプラグインのサンプル比率でルート固有の比率が上書きされる問題を修正しました。 [\#9877](https://github.com/Kong/kong/pull/9877)
  * 小数点を含む`trace-id`文字列と`parent-id`文字列が正しく処理されない問題を修正しました。

* [**JWT**](/hub/kong-inc/jwt/)（`jwt`）

  * このプラグインは、JWTトークンの検索場所に異なるトークンがあるリクエストを拒否するようになりました。

    この問題を報告してくれたLatacoraのJackson 'Che\-Chun' Kuoさんに感謝します。
    [\#9946](https://github.com/Kong/kong/pull/9946)

* [**Datadog**](/hub/kong-inc/datadog/)（`datadog`）[**、OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）、[**StatsD**](/hub/kong-inc/statsd/)（`statsd`）

  * このプラグインのバッチキュー処理で、メトリクスが複数回公開される問題を修正しました。 この問題によりメモリリークが発生し、メモリ使用量が無制限に増加していました。 [\#10052](https://github.com/Kong/kong/pull/10052) [\#10044](https://github.com/Kong/kong/pull/10044)

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * 非準拠仕様を次のように修正しました。
    * スパンの`http.uri`で、フィールドが完全なHTTP URIになりました。 [\#10036](https://github.com/Kong/kong/pull/10036)
    * `http.status_code`がステータスコードのあるリクエストのスパンに表示されるようになりました。 [\#10160](https://github.com/Kong/kong/pull/10160)
    * `http.flavor`がdoubleでなくstring値になりました。 [\#10160](https://github.com/Kong/kong/pull/10160)

  * 他の形式のトレースが取得され、報告および伝播されるトレースIDの長さが不適切になる可能性のある問題を修正しました。 この問題が原因で、Kong Gatewayから発信されるトレースがターゲットサービスに誤って接続し、Kong Gatewayとターゲットサービスから別々のトレースが送信されていました。 [\#10332](https://github.com/Kong/kong/pull/10332)

* [**OAuth2**](/hub/kong-inc/oauth2/)（`oauth2`）

  * `refresh_token_ttl`の範囲はスキーマバリデーターによって`0`から`100000000`の範囲に制限されるようになりました。 以前は、数値が大きすぎるとリクエストに失敗していました。 [\#10068](https://github.com/Kong/kong/pull/10068)

* [**OpenID Connect**](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 匿名のコンシューマを名前で指定できない問題を修正しました。
  * `authorization_cookie_httponly`と`session_cookie_httponly`パラメータが、`false`に設定されている場合も含めて、常に`true`に設定される問題を修正しました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * このプラグインの動作をRate Limitingプラグインの動作と一致させました。 従来は、`HTTP 429`ステータスコードが返された場合、流量制限関連のヘッダーがPDKモジュール`kong.response.exit()`から欠落していました。 そのため、このプラグインはExit Transformerプラグインなどの他のKongコンポーネントと互換性がありませんでした。

* [**Response Transformer**](/hub/kong-inc/response-transformer/)（`response-transformer`）

  * `allow.json`構成パラメータで、ネストされたJSONオブジェクトと配列構文を使用できない問題を修正しました。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * UUIDパターンのマッチングに関する問題を修正しました。

* [**SAML**](/hub/kong-inc/saml/)（`saml`）

  * `session_cookie_httponly`パラメータが、`false`に設定されている場合を含めて常に`true`に設定される問題を修正しました。

* [**Key Authenticationが暗号化されました**](/hub/kong-inc/key-auth-enc/)（`key-auth-enc`）

  * `ttl`パラメータを修正し、暗号化キーに`ttl`を設定できるようになりました。
  * このプラグインでタグを使用できない問題を修正しました。

### 依存関係

* `lua-resty-openssl`を0\.8\.15から0\.8\.17にアップグレードしました
* `libexpat`を2\.4\.9から2\.5\.0にアップグレードしました
* `kong-openid-connect`をv2\.5\.0からv2\.5\.2にアップグレードしました
* `openssl`を1\.1\.1qから1\.1\.1tにアップグレードしました
* `libyaml`がKong Gatewayで構築されなくなりました。代わりにSystem `libyaml`が使用されます。
* `luarocks`を3\.9\.1から3\.9\.2にアップグレードしました [\#9942](https://github.com/Kong/kong/pull/9942)
* `atc-router`を1\.0\.1から1\.0\.5にアップグレードしました [＃9925](https://github.com/Kong/kong/pull/9925) [＃10143](https://github.com/Kong/kong/pull/10143) [＃10208](https://github.com/Kong/kong/pull/10208)
* `lua-resty-openssl`を0\.8\.15から0\.8\.17にアップグレードしました [＃9583](https://github.com/Kong/kong/pull/9583) [＃10144](https://github.com/Kong/kong/pull/10144)
* `lua-kong-nginx-module`を0\.5\.0から0\.5\.1にアップグレードしました [＃10181](https://github.com/Kong/kong/pull/10181)
* `lua-resty-session`を3\.10から4\.0\.0にアップグレードしました [＃10199](https://github.com/Kong/kong/pull/10199) [＃10230](https://github.com/Kong/kong/pull/10230)
* `libxml`を2\.10\.2から2\.10\.3にアップグレードし、[CVE\-2022\-40303](https://nvd.nist.gov/vuln/detail/cve-2022-40303)と[CVE\-2022\-40304](https://nvd.nist.gov/vuln/detail/cve-2022-40304)を解決しました。

3\.1\.1\.6
-------------

**リリース日** 2023/10/12

### 修正

#### コア

* HTTP/2ストリームリセット攻撃を早期に検出するためのNginxパッチを適用しました。
  この変更は、特定された脆弱性[CVE\-2023\-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)に直接対処するものです。

  この脆弱性とそれに対するKongの対処に関する詳細については、当社の[ブログ記事](https://konghq.com/blog/product-releases/novel-http2-rapid-reset-ddos-vulnerability-update)をご覧ください。

### 依存関係

* `libxml2`を2\.10\.2から2\.11\.5に更新しました

3\.1\.1\.5
-------------

**リリース日** 2023/08/25

### 機能

* RedisのRate Limitingストラテジで、接続障害が検出されるようになりました。
* Kong 管理者を自動的に作成するためのパラメータ、`admin_auto_create`が追加されました。
* Kong Managerは、OIDCベースの認証に対する`POST`応答メソッドをサポートするようになりました。

### 修正

#### Enterprise

* 動的な並び替えが適用されると並び替え順序が混同する、プラグインイテレータに関する問題を修正しました。この修正により、すべてのシナリオで適切な並べ替え動作が保証されます。
* `resty.dns.client`がUDPソケットをリークする問題を修正しました。
* `anonymous_reports=false`を設定しても匿名レポートが非表示にならないバグを修正しました。
* ハイブリッドモードで、Vitalsと分析がクラスタのテレメトリエンドポイントから伝達できない問題を修正しました。
* ARMアーティファクトのHTTP2リクエストハンドルを修正しました。
* OpenResty ngx.printのチャンクエンコーディングをバックポートすることで、チャンクでエンコードされた応答データの破損を招いていたバッファのダブルフリーバグを修正しました。[\#10816](https://github.com/Kong/kong/pull/10816)[\#10824](https://github.com/Kong/kong/pull/10824)
* Goプラグインサーバープロセスがクラッシュすると、Kongを介してプロキシされた後続のリクエストが、一貫性のない構成でGoプラグインを実行する問題を修正しました。この問題は、同じGoプラグインが異なるルートまたはサービスエンティティに適用されている場合にのみ影響します。
* Dynatraceの実装を修正しました。

**Kong Manager** ：

* 構成リンクによりユーザーがデフォルトワークスペースにリダイレクトされる問題を修正しました。
* OpenID Connectの使用中に無効な認証情報を渡すとリダイレクトしないKong Managerでの問題を修正しました。

#### プラグイン

* Request Transformer Advanced：一部のリクエストが間違ったクエリパラメータでプロキシされる問題を修正しました。
* Response Transformer Advanced：プラグインの使用時に小数点以下の大きな桁数が四捨五入される問題を修正しました。
* Rate Limiting Advanced： 
  * コントロールプレーン（CP）が流量制限の高度なカウンターをRedisと同期しようとする問題を修正しました。
  * `rl cluster_events`が従来のクラスタモードで間違ったデータをブロードキャストするバグを修正しました。

* Oauth2：`refresh_token`がインスタンス全体で共有される可能性があるバグを修正しました。

### 依存関係

* `OpenSSL`を1\.1\.1tから3\.1\.1に更新しました 
* `lua-resty-openssl`を0\.8\.15から0\.8\.22にアップグレードしました
* `lua-resty-kafka`を0\.15から0\.16に更新しました

3\.1\.1\.4
-------------

**リリース日** 2023/05/16

### 機能

* Kong Manager with OIDC： 
  * 管理者の自動作成を有効または無効にする構成オプション [`admin_auto_create`](/gateway/latest/kong-manager/auth/oidc/mapping/)を追加しました。 このオプションのデフォルトは`true`です。

### 修正

#### コア

* 頻繁なDNSクエリによって発生する`resty.dns.client`のUDPソケットリークを修正しました。 [\#10691](https://github.com/Kong/kong/pull/10691)
* ハイブリッドモード：Vitals/分析がクラスタのテレメトリエンドポイントから伝達できない問題を修正しました。
* `alpine`と`ubuntu`のARM64アーティファクトがHTTP/2リクエストを誤処理し、プロトコルが失敗する問題を修正しました。
* チャンクエンコードされた応答データの破損につながっていた、OpenResty `ngx.print`チャンクエコーディングの重複した空きバッファの問題を修正しました。 [\#10816](https://github.com/Kong/kong/pull/10816) [\#10824](https://github.com/Kong/kong/pull/10824)
* Dynatraceの実装を修正しました。ビルドシステムの問題により、Kong Gateway 3\.1\.xパッケージのうち3\.1\.1\.4より前のものには Dynatraceに必要なデバッグシンボルが含まれていませんでした。

#### Enterprise

**Kong Manager** ：

* StatsDプラグインの次の構成フィールドを修正しました。

  * 欠落していたメトリクスフィールド、`consumer_identifier`、`service_identifier`、`workspace_identifier`を追加しました。
  * 存在しない`custom_identifier`フィールドを削除しました。

* プラグインの`Copy JSON`操作で、プラグインの構成全体がコピーされない問題を修正しました。

* Zipkinプラグインで、Kong Manager UIを通じて`static_tags`を追加できない問題を修正しました。

* 欠落していたデフォルト値をVault構成ページに追加しました。

* フリーモードバナーの壊れたKonnectリンクを修正しました。

* OIDC認証の問題：

  * Kong ManagerがOIDC認証に使用する`/auth`エンドポイントが、HTTP POSTメソッドを正しくサポートするようになりました。
  * Kong ManagerのOIDC認証で、デフォルトロール（`workspace-super-admin`、`workspace-read-only`、`workspace-portal-admin`、`workspace-admin`）が新たに作成されるワークスペースから欠落していましたが、この問題を修正しました。 
  * OIDCを通じて新規登録されたDev Portalアカウントを持つユーザーが、Kong Gatewayコンテナを再起動するまでDev Portal にログインできない問題を修正しました。 この問題は、`by_username_ignore_case`が`true`に設定されている場合に発生し、キャッシュからコンシューマが常に読み込まれる誤動作の原因になっていました。

#### プラグイン

* [**Request Transformer Advanced**](/hub/kong-inc/request-transformer-advanced/)（`request-transformer-advanced`）
  * 一部のリクエストが間違ったクエリパラメータでプロキシされる問題を修正しました。

3\.1\.1\.3
-------------

**リリース日** 2023/01/30

### 修正

#### Enterprise

* `ca-certificates`の依存関係がパッケージと画像から誤って削除される問題を修正しました。 この問題によりSSL接続で共通のルート認証局を使用できませんでした。

### アップグレード

{{site.base_gateway}}を2\.8\.x.xから3\.1\.1\.3に直接アップグレードできるようになりました。以前は、3\.0\.xにアップグレードしてから、最新の3\.xバージョンにアップグレードする必要がありました。

3\.1\.1\.2
-------------

**リリース日** 2023/01/24

### 機能

#### Enterprise

* **Dev Portal** ： 
  * Dev Portal APIが、`/files`エンドポイントでオプションの`fields`クエリパラメータをサポートするようになりました。 このパラメータを使用すると、応答に含めるファイルオブジェクトフィールドを指定できます。

#### コア

* `router_flavor`が`traditional_compatible`の場合、従来のルーターでなく
  式ルーターを使用して作成されたルートを検証して、作成されたルートが実際に互換性があることを確認します。
  [\#10088](https://github.com/Kong/kong/pull/10088)

* `kong migrations up`が、3\.0ルーターと互換性のないルートを報告して、移行を中断するようになりました。これにより管理者は調整する機会を得ることができます。

  [\#10092](https://github.com/Kong/kong/pull/10092)
  [\#10101](https://github.com/Kong/kong/pull/10101)

### 修正

#### Enterprise

* **Kong Manager** ： 
  * 次のプラグインリストの問題を修正しました。 
    * TLS Handshake ModifierおよびTLS Metadata Headersの各プラグイン向けに、欠落していたアイコンとカテゴリを追加しました。
    * 非推奨プラグインである、Kubernetes Sidecar Injector、Collector、Upstream TLSのエントリを削除しました。
    * Kong ManagerからApache OpenWhiskプラグインを削除しました。このプラグインは、[LuaRocks経由で手動でインストール](/hub/kong-inc/openwhisk/)する必要があります。
    * 内部専用のKonnect Application Authプラグインを削除しました。

  * 認証方法としてOpenID Connectを使用する場合、他のページにリダイレクトしたりページを更新したりすると、Kong Managerが時々ログアウトする問題を修正しました。 
  * スコープが設定されているプラグインにバインドされたサービス、ルート、またはコンシューマの更新中に、`404 Not Found`エラーがトリガーされる問題を修正しました。
  * `['create'] /services/*/plugins`権限を持つ管理者がサービスの下位にプラグインを作成できない問題を修正しました。
  * `default`以外のワークスペースでコンシューマグループを表示すると、`404 Not Found`エラーが発生する問題を修正しました。

#### コア

* Insoで生成される正規表現がKong Gatewayで機能しない問題を修正しました。
* ワーカーがクラッシュする潜在的な問題に対処するために、`atc-router`を`1.0.2`にアップグレードしました。 [\#9927](https://github.com/Kong/kong/pull/9927)

#### ハイブリッドモード

* `/licenses`エンドポイントを使用してライセンスがデプロイされた後に、Vitalsデータが表示されない問題を修正しました。 Kong Gatewayは、ライセンスのプリロード中にVitalsサブシステムを再初期化できるイベントをトリガーするようになりました。
* データプレーン（DP）とコントロールプレーン（CP）の間の転送プロキシがテレメトリポート`8006`をサポートしない問題を修正しました。
* 構成を同期するためのWebSocketプロトコルのサポートを削除し直し、 2\.8\.xxデータプレーン（DP）との下位互換性を復活させました。 [\#10067](https://github.com/Kong/kong/pull/10067)

#### プラグイン

* [**Datadog**](/hub/kong-inc/datadog/)（`datadog`）[**、OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）、[**StatsD**](/hub/kong-inc/statsd/)（`statsd`）

  * このプラグインのバッチキュー処理で、メトリクスが複数回公開される問題を修正しました。 この問題によりメモリリークが発生し、メモリ使用量が無制限に増加していました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * `local`ストラテジの問題を修正しました。修正前にはこのストラテジは、`window_size`が`fixed`に設定されているときに正常に機能せず、ウィンドウがまだ有効であっても、キャッシュが期限切れになっていました。 

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）

  * バンドル済みプラグインリストにOAS Validationプラグインを再度追加しました。現在、このプラグインは、追加の構成をせずに、`kong.conf`を通じてデフォルトで使用できるようになっています。 
  * パススキーマの仕様の取得に失敗すると、プラグインから誤ったエラーメッセージが返される問題を修正しました。
  * 応答ボディスキーマにコンテンツフィールドがない場合に発生していた`500`エラーを修正しました。

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * ルートの更新後にプラグインが古いルートキャッシュを使用する問題を修正しました。

### 非推奨

* `/vitals/reports/:entity_type`エンドポイントのサポートが非推奨になりました。代わりに、Vitals APIの次のエンドポイントのいずれかを使用します。

  * `/vitals/reports/consumer`には、`/{workspace_name}/vitals/status_codes/by_consumer`を代わりに使用します
  * `/vitals/reports/service`には、`/{workspace_name}/vitals/status_codes/by_service`を代わりに使用します
  * `/vitals/reports/hostname`には、`/{workspace_name}/vitals/nodes`を代わりに使用します

  詳細については、[Vitalsのドキュメント](/gateway/latest/kong-enterprise/analytics/#vitals-api)を参照してください。

### 既知の問題

* `ca-certificates`の依存関係がパッケージと画像から欠落しているため、
  SSL接続で共通のルート認証局を使用できなくなっています。

  3\.1\.1\.3にアップグレードして、解決します。

3\.1\.0\.0
-------------

**リリース日** 2022/12/06

### 機能

#### Enterprise

* シークレット管理のためにHashiCorp Vaultsの名前空間を指定できるようになりました。

* HashiCorp VaultのバックエンドがKubernetesサービスアカウントからVaultトークンを取得するためのサポートを追加しました。
  次の構成パラメータを参照してください。

  * [`keyring_vault_auth_method`](/gateway/latest/reference/configuration/#keyring_vault_auth_method)
  * [`keyring_vault_kube_role`](/gateway/latest/reference/configuration/#keyring_vault_kube_role)
  * [`keyring_vault_kube_api_token_file`](/gateway/latest/reference/configuration/#keyring_vault_kube_api_token_file)

* FIPS 140\-2パッケージ：

  * Kong Gateway Enterpriseで、[Red Hat Enterprise 8とUbuntu 22\.04向けのFIPS 140\-2準拠パッケージ](/gateway/latest/kong-enterprise/fips-support/)が提供されるようになりました。
  * Kong Gateway FIPSディストリビューションが、PostgreSQLへのTLS接続をサポートするようになりました。

* コンシューマグループまたはグループ内のコンシューマを削除せずに、[コンシューマグループ構成を削除](/gateway/latest/kong-enterprise/consumer-groups/#delete-consumer-group-configurations)できるようになりました。

* **Kong Manager** ：

  * Kong Managerにベースパスを構成できるようになりました（例：`localhost:8445/manager`）。この結果、{{site.base_gateway}}経由ですべてのトラフィックをプロキシできるようになり、たとえば、1つのポートからAPIとKong Managerの両方のトラフィックをプロキシできます。また、新しいKong Manager基本パスを使用して、Kong Managerへのアクセス制御用プラグインを追加できます。詳細については、「[Kong Managerを有効化](/gateway/latest/kong-manager/enable/)」を参照してください。
  * Kong Managerでコンシューマグループを作成できるようになりました。その結果、必要な数の流量制限ティアを定義して、コンシューマののサブセットに適用できるため、コンシューマを個別に管理する必要がありません。詳細については、「[Kong Managerでコンシューマグループを作成](/gateway/latest/kong-manager/consumer-groups/)」を参照してください。
  * コンシューマに`key-auth-enc`認証情報を追加できるようになりました。
  * OpenID Connectプラグイン： **認証** タブに認証変数が追加されました。
  * Kong Managerの概要タブのパフォーマンスが最適化されました。
  * Kong Managerを通じてシークレット管理用Vaultを構成できるようになりました。 新しい［Vaults］メニューを使用して、Kong GatewayがサポートするVaultを設定、管理します。 すべての構成オプションの詳細については、[Vault Backendsの参考資料](/gateway/latest/kong-enterprise/secrets-management/backends/)を参照してください。 
  * 動的プラグインの順序付けと連動させるためのサポートを追加しました。
  * 証明書の詳細を表示する機能が追加されました。
  * プラグインUIにフィールド説明付きツールチップを追加しました。
  * リストのページサイズをすべてのページで固定すると同時に、ページサイズの選択肢を増やしました。 

#### コア

* `kong.conf` SSLプロパティをVaultまたは環境変数に保存できるようになりました。 その結果、このプロパティをコンテンツに直接構成したり、base64でエンコードされたコンテンツに構成したりできます。 [\#9253](https://github.com/Kong/kong/pull/9253)
* スキーマ内でのエンティティの完全変換サポートを追加しました。 [\#9431](https://github.com/Kong/kong/pull/9431)
* スキーマ`map`タイプフィールドが参照可能とマークできるようになりました。 [\#9611](https://github.com/Kong/kong/pull/9611)
* [ログレベルを動的に変更](/gateway/latest/production/logging/update-log-level-dynamically/)するためのサポートを追加しました。 [\#9744](https://github.com/Kong/kong/pull/9744)
* `keys`エンティティと`key-sets`エンティティのサポートを追加しました。これらエンティティはさまざまな形式（JWK、PEM）の非対称キーの管理に使用されます。 詳細については、[キー管理](/gateway/latest/reference/key-management/)を参照してください。 [\#9737](https://github.com/Kong/kong/pull/9737)

#### ハイブリッドモード

* データプレーン（DP）のノードIDが、再起動後も保持されるようになりました。 [\#9067](https://github.com/Kong/kong/pull/9067)
* ハイブリッドモード接続用にHTTP CONNECTのフォワードプロキシのサポートが追加されました。新しい構成オプション、`cluster_use_proxy`、`proxy_server`、`proxy_server_ssl_verify` が追加されました。 詳細については、[フォワードプロキシを通じたCP/DP通信](/gateway/latest/production/networking/cp-dp-proxy/)を参照してください。 [\#9758](https://github.com/Kong/kong/pull/9758) [\#9773](https://github.com/Kong/kong/pull/9773)

#### パフォーマンス

* デフォルト値の`lua_regex_cache_max_entries`を引き上げました。正規表現ルートの数が多すぎ、かつ `router_flavor`が `traditional`である場合は、警告がスローされます。[\#9624](https://github.com/Kong/kong/pull/9624)
* DatadogとStatsDプラグインにバッチキューを追加して、タイマーの使用量が減るようにしました。 [\#9521](https://github.com/Kong/kong/pull/9521)

#### OSサポート

* Kong GatewayはEnterpriseパッケージで、Amazon Linux 2022をサポートするようになりました。
* Kong GatewayはオープンソースパッケージとEnterpriseパッケージの両方で、Ubuntu 22\.04をサポートするようになりました。

#### PDK

* `kong.client.tls.request_client_certificate`を拡張して、使用可能なCA証明書の識別名（DN）リストのヒントを設定できるようになりました。 [\#9768](https://github.com/Kong/kong/pull/9768)

#### プラグイン

**新しいプラグイン：** 

* [**AppDynamics**](/hub/kong-inc/app-dynamics/)（`app-dynamics`） 
  * Kong GatewayをAppDynamics APMプラットフォームと統合します。

* [**JWE Decrypt**](/hub/kong-inc/jwe-decrypt/)（`jwe-decrypt`）
  * リクエスト内のインバウンドトークン（JWE）を復号化できます。

* [**OAS Validation**](/hub/kong-inc/oas-validation/)（`oas-validation`）
  * OpenAPI 3\.0またはSwagger API仕様に基づいて、HTTPリクエストと応答を検証します。

* [**SAML**](/hub/kong-inc/saml/)（`saml`）
  * サービスプロバイダー（Kong Gateway）とIDプロバイダー（IdP）の間でSAML v2\.0認証と承認を提供します。

* [**XML Threat Protection**](/hub/kong-inc/xml-threat-protection/)（`xml-threat-protection`） 
  * この新しいプラグインでは、XMLペイロードの構造をチェックすることで、XML攻撃のリスクを軽減できます。構造チェックでは、要素と属性の最大複雑度（ツリーの深さ）と最大サイズを検証します。

**既存のプラグインの更新:** 

* [**ACME**](/hub/kong-inc/acme/)（`acme`）

  * 構成プロパティ、`config.storage_config.redis.ssl`、`config.storage_config.redis.ssl_verify`、`config.storage_config.redis.ssl_server_name` を通じて、Redis SSLのサポートを追加しました。 [\#9626](https://github.com/Kong/kong/pull/9626)

* [**AWS Lambda**](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * `awsgateway_compatible`入力データに`requestContext`フィールドを追加しました [＃9380](https://github.com/Kong/kong/pull/9380)

* [**認証プラグイン**](/hub/#authentication)：

  * `anonymous`フィールドをコンシューマのユーザー名として構成できるようになりました。 このフィールドでは、認証に失敗した場合に「匿名」コンシューマとして使用する文字列を構成できます。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * `headers`フィールドにな参照可能な属性を追加し、Vaultに保存できるようにしました。 [\#9611](https://github.com/Kong/kong/pull/9611)

* [**Forward Proxy**](/hub/kong-inc/forward-proxy/)（`forward-proxy`）

  * `x_headers`フィールドが追加されました。このフィールドには、プラグインが次のヘッダーを処理する方法が示されています：
    `X-Real-IP`、`X-Forwarded-For`、`X-Forwarded-Proto`、`X-Forwarded-Host`、`X-Forwarded-Port`。

    このフィールドでは以下のいずれかのオプションを指定できます。
    * `append`：チェーン内のこのホップからの情報を上記ヘッダーに追加します。これはデフォルト設定です。
    * `transparent`：Kong Gatewayがプロキシではないかのように、ヘッダーを変更せずに維持します。
    * `delete`：Kong Gatewayが発信元クライアントであるかのように、すべてのヘッダーを削除します。

    すべてのオプションは信頼のおけるIP設定に従っており、チェーンの最後のホップからのヘッダーは発信元が信頼のおけるIPを使用するクライアントでない場合は、無視されます。

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * `included_status_codes`フィールドと`random_status_code`フィールドを追加しました。これらのフィールドでプラグインのHTTPステータスコードを構成できます。
  * このプラグインで、例を定義しなくても、スキーマ定義に基づいてランダムな応答を自動生成できるようになりました。
  * 動作ヘッダー、`X-Kong-Mocking-Delay`、`X-Kong-Mocking-Example-Id`、`X-Kong-Mocking-Status-Code`を送信することで、動作を制御したり、特定の応答を取得したりできるようになりました。 
  * 現在、このプラグインは以下をサポートするようになっています。 
    * MIMEタイプの優先度の一致
    * すべてのHTTPコード
    * `$ref`

* [**Mutual TLS Authentication**](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * SSLハンドシェイク中に`CertificateRequest`メッセージでCA DNの送信をサポートするために`config.send_ca_dn`構成パラメータを追加しました。
  * `allow_partial_chain`構成パラメータを追加して、中間証明書のみで証明書を検証できるようになりました。

* [**OPA**](/hub/kong-inc/opa/)（`OPA`）

  * `include_uri_captures_in_opa_input`フィールドを追加しました。このフィールドをtrueに設定すると、現在のリクエストのKong Gatewayのルートパスフィールドに[正規表現取得グループ](/gateway/latest/reference/proxy/#using-regex-in-paths)がキャプチャされる場合は、OPAへの入力として含められます。

* [**Proxy Cache Advanced**](/hub/kong-inc/proxy-cache-advanced/)（`proxy-cache-advanced`）

  * `config.redis.cluster_addresses`構成プロパティを通じたRedisクラスタとの統合サポートを追加しました。

* [**Rate Limiting**](/hub/kong-inc/rate-limiting/)（`rate-limiting`）

  * 流量制限リクエストのHTTPステータスコードと応答ボディをカスタマイズできるようになりました。 [@utix](https://github.com/utix)さん、ご協力ありがとうございます。 [\#8930](https://github.com/Kong/kong/pull/8930)

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * APIを使用して顧客グループを削除するためのサポートが追加されました。
  * `429`をカウントするかどうかを制御するために`config.disable_penalty`をスライディングウィンドウモードで追加しました。 

* [**Request Transformer Advanced**](/hub/kong-inc/request-transformer-advanced/)（`request-transformer-advanced`）

  * JSONペイロードの変換時に、ネストされたJSONオブジェクトと配列をナビゲートするためのサポートを追加しました。
  * プラグインでVault参照がサポートされるようになりました。

* [**Request Validator**](/hub/kong-inc/request-validator/)（`request-validator`）

  * このプラグインは、`config.allowed_content_types`パラメータで`charset`オプションをサポートするようになりました。 

* [**Response Rate Limiting**](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）

  * 構成プロパティ、`redis_ssl`（`true`または`false`に設定可能）、`ssl_verify`、`ssl_server_name`を通じてRedis SSLのサポートを追加しました。 [@dominikkukacka](https://github.com/dominikkukacka)さん、ありがとうございます。 [\#8595](https://github.com/Kong/kong/pull/8595)

* [**Route Transformer Advanced**](/hub/kong-inc/route-transformer-advanced/)（`route-transformer-advanced`）

  * `config.escape_path`構成パラメータを追加して、変換されたパスをエスケープできるようにしました。 

* [**Session**](/hub/kong-inc/session/)（`session`）

  * ブラウザが閉じられてもCookieを永続化できる新しい構成、`cookie_persistent` を追加しました。デフォルト設定の`false` では、ブラウザを再起動するとCookieは保持されません。 [@tschaume](https://github.com/tschaume) さん、ご協力ありがとうございました。 [\#8187](https://github.com/Kong/kong/pull/8187)

* [**Vault Authentication**](/hub/kong-inc/vault-auth/)（`vault-auth`）

  * KV Secrets Engine v2のサポートを追加しました。

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）

  * Zipkinプラグインに`response_header_for_traceid`フィールドを追加しました。 このフィールドに文字列値が指定される場合、プラグインでは対応するヘッダーが応答に設定されます。 [\#9173](https://github.com/Kong/kong/pull/9173)

* WebSocketサービス/ルートのサポートがロギングプラグイン用に追加されました。

  * http\-log
  * file\-log
  * udp\-log
  * tcp\-log
  * loggly
  * syslog
  * kafka\-log

### 既知の制限

* 動的ログレベル使用時に、ログレベルを`alert`を設定しても、`info`と`error`のエントリがログに表示されます。

### 修正

#### Enterprise

* `user_token`フィールドの更新後にRBACトークンが再ハッシュされない問題を修正しました。
* `admin_gui_auth_conf`がJSON形式の値を受け入れないため、シークレットのVault参照機能を使用できない問題を修正しました。 
* Admin GUIログが正しいログファイルに保存されない問題を修正しました。
* Vaultの使用中に、Kong Gatewayが無料Enterpriseモードで起動できない問題を修正しました。 
* `TRACE`メソッドリクエストの応答ボディを更新しました。
* 重みが`0`に設定されているターゲットは、ヘルスチェックに含まれなくなりました。`upstreams/<upstream id="sl-md0000000">/health`エンドポイントからステータスをチェックすると、ステータス結果は`HEALTHCHECK_OFF`になります。 従来、`upstreams/<upstream id="sl-md0000000">/health`エンドポイントでは、`weight=0`のターゲットは`HEALTHY`と誤って報告され、ヘルスチェックでは`UNDEFINED`と報告されていました。
* データベースがダウンしている期間におけるAdmin APIの応答ステータスコードを`500`から`200`に更新しました。
* Admin API `/licenses`エンドポイントを使用してコントロールプレーン（CP）からデータプレーン（DP）にライセンスを渡す際に発生していた問題を修正しました。 
* ハイブリッドモード時にライセンスエンティティを最初に処理しないと、エンティティ検証に失敗するライセンス関連の問題を修正しました。 
* リダイレクトに関するWebSocketの問題を修正し、Kong Gatewayが`ws`リクエストを`wss`にリダイレクトして、`wss`のルーティングがHTTP/HTTPSと同等になるようにしました。 
* **Kong Manager** ： 
  * すべてのKong Managerのアクセスログの記録を追加しました。
  * **新しいワークスペース** ボタンが時々使用できない問題を修正しました。
  * Kong Managerでのプラグイン構成の名前表示を修正しました。
  * 提案リストに多数の項目がある場合、リストから一部の項目が欠落する問題を修正しました。 
  * 非推奨になったVitalsレポート機能がKong Managerから削除されました。
  * ルートやサービスなどのスコープが設定されているエンティティとやり取りする権限を持つ管理者が想定どおりの操作を実行できない問題を修正しました。 
  * `/admins`権限を持つ管理者がサインイン後に強制的にログアウトされる問題を修正しました。 
  * 管理者が多数のワークスペースの権限を持つことでKong Managerの読み込みが遅くなるパフォーマンス上の問題を修正しました。 

#### コア

* 外部プラグインが処理不能な例外によりクラッシュしたことが原因で、自動的に再起動した後にCPU使用率が高くなる問題を修正しました。 [\#9384](https://github.com/Kong/kong/pull/9384)
* バランサーのアップストリームに`use_srv_name`オプションを追加しました。 [\#9430](https://github.com/Kong/kong/pull/9430)
* スパンが正常に作成されない、`header_filter`の計測に関する問題を修正しました。 [\#9434](https://github.com/Kong/kong/pull/9434)
* `traditional_compatible`モードでのルーター構築時の問題を修正しました。 従来は、空のテーブルがフィールドに含まれる場合、無効な式が生成されていました。 [\#9451](https://github.com/Kong/kong/pull/9451)
* `paths`フィールドが無効な場合、ルーターの再構築時にmutexが適切にリリースされない問題を修正しました。 [\#9480](https://github.com/Kong/kong/pull/9480)
* `KONG_PREFIX`が相対パスに設定されていない場合、`kong docker-start`に失敗する問題を修正しました。 [\#9337](https://github.com/Kong/kong/pull/9337)
* `kong start`でのエラー処理とプロセスクリーンアップに関する問題を修正しました。 [\#9337](https://github.com/Kong/kong/pull/9337)
* プレフィックスパスの正規化に関する問題を修正しました。 [\#9760](https://github.com/Kong/kong/pull/9760)
* Admin APIの最大リクエスト引数の数を100から1000に引き上げました。 Admin APIは、リクエストパラメータが制限を超えた場合、制限を超えるパラメータを切り捨てる代わりに、`400`エラーを返すようになりました。 [\#9510](https://github.com/Kong/kong/pull/9510)
* ページングサイズのパラメータを現在のリクエストで指定すると、次のページに伝達されるようになりました。 [\#9503](https://github.com/Kong/kong/pull/9503)

#### ハイブリッドモード

* データプレーン（DP）とコントロールプレーン（CP）ワーカーとの最初の接続が確立されたときに、競合状態が原因で構成プッシュイベントが削除される問題を修正しました。 [\#9616](https://github.com/Kong/kong/pull/9616)

#### CLI

* 保留中のタイマージョブによるCLIパフォーマンスの低下を解消しました。 [\#9536](https://github.com/Kong/kong/pull/9536)

#### PDK

* `kong.request.get_uri_captures` （`kong.request.getUriCaptures`）のサポートを追加しました [＃9512](https://github.com/Kong/kong/pull/9512)
* パラメータタイプ`kong.service.request.set_raw_body`（`kong.service.request.setRawBody`）、戻り値`kong.service.response.get_raw_body`（`kong.service.request.getRawBody`）、ボディパラメータタイプ`kong.response.exit`をバイトに修正しました。 古いバージョンのgo PDKはこの変更後に互換性がなくなることに注意してください。 [\#9526](https://github.com/Kong/kong/pull/9526)

#### プラグイン

* 欠落していた`protocols`フィールドを次のプラグインスキーマに追加しました。

  * Azure Functions（`azure-functions`）
  * gRPC Gateway（`grpc-gateway`）
  * gRPC Web（`grpc-web`）
  * サーバーレスPre\-function（`pre-function`）
  * Prometheus（`prometheus`）
  * Proxy Caching（`proxy-cache`）
  * Request Transformer（`request-transformer`）
  * Session（`session`）
  * Zipkin（`zipkin`）

  [\#9525](https://github.com/Kong/kong/pull/9525)

<!-- -->
* [**AWS Lambda**](/hub/kong-inc/aws-lambda/)（`aws-lambda`）
  * ECS環境で環境変数を読み取れない原因となっていた問題を修正しました。 [\#9460](https://github.com/Kong/kong/pull/9460)
  * Lambda出力の`isBase64Encoded`フィールドにNULL値を指定すると、エラーログエントリは`502`コードの付いた、よりわかりやすいものになります。 [\#9598](https://github.com/Kong/kong/pull/9598)

<!-- -->
* [**Azure Functions**](/hub/kong-inc/azure-functions/)（`azure-functions`）
  * 次の状況でこのプラグインで呼び出しを行うと失敗する問題を修正しました。 
    * このプラグインがサービスが存在しないルートに関連付けられている状況。
    * ルートが関連付けられているサービスに`path`の値がある状況。 [\#9177](https://github.com/Kong/kong/pull/9177)

<!-- -->
* [**HTTP Log**](/hub/kong-inc/http-log/)（`http-log`）

  * キューIDのシリアル化に`queue_size`と`flush_timeout`が含まれない問題を修正しました。 [\#9789](https://github.com/Kong/kong/pull/9789)

* [**Mocking**](/hub/kong-inc/mocking/)（`mocking`）

  * `accept`ヘッダーが分割されず、ワイルドカードを使用すると機能しない問題を修正しました。`accept`ヘッダーの`;q=`（q係数の重み付け）がサポートされるようになりました。

* [**OPA**](/hub/kong-inc/opa/)（`opa`）

  * 冗長な非推奨のコードをプラグインから削除しました。

* [**OpenTelemetry**](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

  * デフォルトの伝播ヘッダーが`w3c`に正しく構成されない問題を修正しました。 [\#9457](https://github.com/Kong/kong/pull/9457)
  * データ競合を回避するために、ワーカーレベルのテーブルキャッシュを`BatchQueue`と置き換えました。 [\#9504](https://github.com/Kong/kong/pull/9504)
  * w3c traceparentの伝播時に、`parent_id`がスパンに設定されない問題を修正しました。 [\#9628](https://github.com/Kong/kong/pull/9628)

* [**Proxy Cache Advanced**](/hub/kong-inc/proxy-cache-advanced/)（`proxy-cached-advanced`）

  * Kong Gatewayが`config.ssl=false`を使用してRedis SSLポート`6379`に接続するときに、このプラグインでエラーがキャッチされるようになりました。

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）

  * このプラグインの共有ディクショナリでTTLが必ず`config.sync_rate`を上回るようになりました。そうでない場合、Kong Gatewayは共有ディクショナリのすべてのリクエストカウンターを失います。

* [**Request Transformer**](/hub/kong-inc/request-transformer/)（`request-transformer`）

  * ヘッダーの名前を変更すると既存のヘッダーが上書きされ、予期しない結果につながるバグを修正しました。 [\#9442](https://github.com/Kong/kong/pull/9442)

* [**Request Termination**](/hub/kong-inc/request-termination/)（`request-termination`）

  * このプラグインで、`status_code`を`null`に設定できなくなりました。 [\#9400](https://github.com/Kong/kong/pull/9400)

* [**Response Transformer**](/hub/kong-inc/response-transformer/)（`response-transformer`）

  * 予期しないボディを受信すると、プラグインが壊れるバグを修正しました。 [\#9463](https://github.com/Kong/kong/pull/9463)

* [**Zipkin**](/hub/kong-inc/zipkin/)（`zipkin`）

  * 無効なOT baggageパターンにより、ZipkinプラグインがOT baggageヘッダーを解析できない問題を修正しました。 [\#9280](https://github.com/Kong/kong/pull/9280)

### 最新の変更

#### ハイブリッドモード

* 従来のハイブリッド構成プロトコルは削除され、3\.0\.0\.0で導入されたwRPCプロトコルが採用されました。 2\.8\.xyから3\.1\.0\.0へのローリングアップグレードはサポートされていないため、3\.0\.xxにアップグレードした後に、3\.1\.0\.0へのローリングアップグレードを実行する必要があります。詳細については、[Kong Gateway 3\.1\.xのアップグレード](/gateway/3.1.x/upgrade/)を参照してください。[\#9740](https://github.com/Kong/kong/pull/9740)

3\.0\.1\.0
-------------

**リリース日** 2022/11/02

### 機能

#### プラグイン

* [Request Transformer Advanced](/hub/kong-inc/request-transformer-advanced/)（`request-transformer-advanced`） 
  * このプラグイン構成で`key:value`ペアに保存されている値が参照可能になり、Vaultに[シークレット](/gateway/latest/kong-enterprise/secrets-management/)として保存できるようになりました。

### 修正

#### Enterprise

* **Kong Manager** ：

  * 構成可能なRBACエンドポイント権限から`all_routes`を削除しました。 このエンドポイントはエンドポイントリストに誤って掲載されており、何も構成しませんでした。
  * 権限のないIDPユーザーがKong Managerにログインできる問題を修正しました。 これらのユーザーは、Kong Managerのリソースにアクセスできませんでしたが、ログイン画面の先に進むことはできました。
  * 有効なEnterpriseライセンスがある環境で、`default`ワークスペースへのアクセス権がない管理者にKong Enterpriseへのアップグレードを指示するメッセージが表示される問題を修正しました。
  * Kong Managerテーブルのページ分割に関する問題を修正しました。
  * 壊れていた`Learn more`リンクを修正しました。
  * グループとロールとのマッピングで、スペースを含むグループ名をサポートしない問題を修正しました。
  * Kong Manager UIにおけるクロスサイトスクリプティング（XSS）に関するセキュリティ脆弱性を修正しました。
  * 特定のエンドポイント（個別のサービスやルートなど）に適用された権限がKong Manager UIに反映されないRBACの問題を修正しました。
  * Kong ManagerからNew Relicを削除しました。以前は、`VUE_APP_NEW_RELIC_LICENSE_KEY`と `VUE_APP_SEGMENT_WRITE_KEY`が無効な値でKong Managerに公開されていました。
  * サービスとルートページにある読み取り専用ユーザー向けのアクションドロップタウンメニューを削除しました。
  * Dev Portalアプリケーション向けの **構成を編集** ボタンを修正しました。
  * ロールページに削除済みロールが一覧表示されるRBACの問題を修正しました。
  * ワークスペースの削除後に孤立したロールが残るため、 **チーム** > **管理者** ページが壊れる問題を修正しました。
  * 欠落していたプラグイン構成用の **JSONをコピー** ボタン追加しました。
  * グローバルワークスペースダッシュボードの **新しいワークスペース** ボタンが、最初のページの読み込み時にクリックできない問題を修正しました。
  * UIからサービスごとに複数のドキュメントを追加する機能を削除しました。 各サービスでサポートされるドキュメントは1つのみであり、UIにこの設定を反映させました。
  * Upstream Timeoutプラグインは、アイコンが追加され、トラフィック制御の構成要素となりました。
  * コンシューマ認証情報リストからACL認証情報を削除しようとすると発生するエラーを修正しました。 このエラーは、プラグインの名前、`acl`、およびそのエンドポイント、`/acls`が一致しないために発生していました。
  * ワークスペースのDev Portalを有効または無効にしてもKong Managerメニューが変更しない、Dev Portalでのキャッシュの問題を修正しました。

* `kong/kong-gateway` Dockerイメージで使用される`alpine`のバージョンが固定されなくなりました。
  以前は使用バージョンが3\.10に固定されていたため、古い`alpine`ビルドが作成されていました。

#### コア

* Kongでの`resty.events`の初期化方法に関する問題を修正しました。従来、このコードでは
  `ngx.config.prefix()`
  を使用して、resty.eventsモジュールに提供するリスニングソケットパスを決定していましたが、Nginxが相対パスプレフィックスで起動される場合に破損が生じる原因となっていました。
  つまり、3\.0\.xを起動するのに、2\.8\.xと同じデフォルト構成を使用できませんでした。

  Kongでは使用可能な場合、`ngx.config.prefix()`でなく、絶対パスに正規化済みの`kong.configuration.prefix`が優先されるようになりました。
  `kong.configuration.prefix`
  が定義されていない場合、絶対パスに解決（変換）後の`ngx.config.prefix()`
  の結果が使用されます。[\#9337](https://github.com/Kong/kong/pull/9337)
* HashiCorp Vaultのシークレット管理参照に関する問題を修正しました。デフォルトでは、`kong start`の使用時に、Kongは環境変数を使用してNginxにシークレットを渡します。Nginxは`kong start`を呼び出さずに直接起動されていたため、初期化時にシークレットを利用できませんでした。[\#9478](https://github.com/Kong/kong/pull/9478)

* Amazon Linux RPMのインストール手順を修正しました。

3\.0\.0\.0
-------------

**リリース日** 2022/09/09

{:.important}
> 
> **重要** ：Kong Gateway 3\.0\.0\.0はメジャーリリースには下位互換性のない変更が含まれています。[アップグレード](/gateway/latest/upgrade/)を試みる前に、[下位互換性のない変更と非推奨事項](#breaking-changes-and-deprecations)、[既知の制限事項](#known-limitations)を確認してください。

### 機能

#### Enterprise

* Kong Gatewayで[動的なプラグインの順序付け](/gateway/3.0.x/kong-enterprise/plugin-ordering/)をサポートするようになりました。
  プラグインの実行順序を指定することで、プラグインの静的優先順位を変更できます。
  たとえば、認証プラグインの前に`rate-limiting`などのプラグインを実行できるようになります。

* Kong Gatewayで、FIPSパッケージを提供するようになりました。このパッケージでは、主要ライブラリがOpenSSLから[BoringSSL](https://boringssl.googlesource.com/boringssl/)に置き換わっています。BoringSSLでは暗号化操作にFIPS 140\-2準拠のBoringCryptoがコアとして使用されます。

  FIPSモードを有効にするには、[`fips`](/gateway/3.0.x/reference/configuration/#fips)を`on`に設定します。
  FIPSモードはUbuntu 20\.04でのみサポートされます。

  {:.note}
  > 
  > **注** ：Kong Gateway FIPSパッケージは現在、SSLのPostgreSQL接続と互換性がありません。
* Kong GatewayにWebSocket検証機能が追加されました。WebSocketはHTTP上で動作する永続的な接続タイプです。

  Kong Gateway 2\.xは、以前は限定的なWebSocket接続をサポートしており、プラグインは各フレームではなく初期接続フェーズでのみ実行されていました。
  現在、Kong Gatewayは、WebSocketフレームをターゲットとするプラグインを実装することで、WebSocketトラフィックをより詳細に制御できるようになっています。

  このリリースには次の要素が含まれます。
  * `ws`プロトコルと`wss`プロトコル向けの[サービス](/gateway/3.0.x/admin-api/#service-object)と[ルート](/gateway/3.0.x/admin-api/#route-object)のサポート
  * 2つの新しいプラグイン：[WebSocket Size Limit](/hub/kong-inc/websocket-size-limit/)と[WebSocket Validator](/hub/kong-inc/websocket-validator/)
  * WebSocketプラグイン開発機能（ **ベータ機能** ） 
    * PDKモジュール：[kong.websocket.client](/gateway/3.0.x/plugin-development/pdk/kong.websocket.client/)と[kong.websocket.upstream](/gateway/3.0.x/plugin-development/pdk/kong.websocket.upstream/)
    * [新しいプラグインハンドラー](/gateway/3.0.x/plugin-development/custom-logic/#websocket-plugin-development)

  [プラグイン開発ガイド](/gateway/3.0.x/plugin-development/custom-logic/#websocket-plugin-development)で、WebSocketプラグインの開発方法を学びましょう。
* Kong Managerはこのリリースで、設計をリファクタリングし、ユーザーエクスペリエンスを向上させました。

  注目すべき変更点：
  * 特定のワークスペースとマルチワークスペースレベルの両方で、ワークスペースダッシュボードを見直しました。
  * ライセンスメトリクスが概要ページの最上部に表示されるようになりました。
  * ワークスペースの選択を二次的な問題にするために、レイアウトとナビゲーションを再構築しました。
  * 権限がない場合、ポータルボタンはグレーアウトされます。
  * 電話ホームメトリクスにライセンスレベルを追加しました。
  * ツールチップを追加しました。

* [シークレット管理](/gateway/3.0.x/kong-enterprise/secrets-management/)が一般提供されるようになりました。

  * シークレットマネージャーにGCP統合サポートが追加され、GCPがVaultバックエンドとして利用できるようになりました。
  * `/vaults-beta`エンティティは非推奨となり、`/vaults`エンティティに置き換えられました。 [\#8871](https://github.com/Kong/kong/pull/8871) [\#9217](https://github.com/Kong/kong/pull/9217)

* Kong GatewayでSlimとUBIイメージが提供されるようになりました。
  Slimイメージは、Kong Gatewayを実行するためにインストールされた最小限のパッケージで構築されたDockerコンテナです。
  3\.0以降、Kong Dockerイメージには、Gatewayの実行に必要なソフトウェアのみが含まれます。
  これにより、セキュリティスキャン中に誤検知された脆弱性がフラグ付けされることはなくなります。

  他の依存関係を保持または追加する場合は、[カスタムのKong Dockerイメージをビルド](/gateway/3.0.x/install/docker/build-custom-images/)できます。
* 便利なDockerタグ（`latest`、`3.0.0.0`、`3.0`など）のベースOSがAlpineからDebian に切り替わりました。

* キーリング暗号化のキー回復を追加しました。
  追加に伴い、Admin APIの新しいエンドポイント、[`/keyring/recover`](/gateway/3.0.x/admin-api/db-encryption/#recover-keyring-from-database)が公開され、`kong.conf`に[`keyring_recovery_public_key`](/gateway/3.0.x/reference/configuration/#keyring_recovery_public_key)を設定する必要があります。

* DBレスとハイブリッドモードにあるデータプレーン（DP）で、[AES\-256\-GCM](https://www.rfc-editor.org/rfc/rfc5288.html)または[chacha20\-poly1305](https://www.rfc-editor.org/rfc/rfc7539.html)暗号化アルゴリズムを使用して、宣言型構成ファイルを暗号化できるようになりました。

  [`declarative_config_encryption_mode`](/gateway/3.0.x/reference/configuration/#declarative_config_encryption_mode)構成パラメータを使用して、希望の暗号化モードを設定します。

#### コア

* このリリースでは、新しいルーター実装である`atc-router`が導入されました。
  このルーターは、複雑なルーティング要件を処理できる強力なルーティング言語
  Rustで記述され、従来の互換モードで使用することも、新しい式ベースの言語を使用することもできます。

  新しいルーターには次のメリットがあります。
  * Kongの構成の変更時にルーターの再ビルド時間が短縮
  * リクエストのルーティング時にランタイムパフォーマンスが向上
  * 1万ルートあたりのP99レイテンシが1\.5秒から0\.1秒に短縮

  ルーターの詳細については、以下を参照してください。
  * [式を使用したルートの構成](/gateway/3.0.x/key-concepts/routes/expressions/)
  * [ルーター式言語の参考資料](/gateway/3.0.x/reference/expressions-language/language-references/)
  * [\#8938](https://github.com/Kong/kong/pull/8938)

* ストリームモードで遅延応答を実装しました。
  [\#6878](https://github.com/Kong/kong/pull/6878)

* 一意性検出のためにターゲットエンティティに`cache_key`を追加しました。
  [\#8179](https://github.com/Kong/kong/pull/8179)

* OpenTelemetry API仕様と互換性のあるトレーシングAPIを導入し、
  組み込みの計測機能を追加します。

  トレーシングAPIは、外部のエクスポータープラグインと併用するためのものです。
  組み込み計測タイプとサンプリングレートは
  [`opentelemetry_tracing`](/gateway/3.0.x/reference/configuration/#opentelemetry_tracing)と[`opentelemetry_tracing_sampling_rate`](/gateway/3.0.x/reference/configuration/#opentelemetry_tracing_sampling_rate)オプションを通じて構成できます。
  [\#8724](https://github.com/Kong/kong/pull/8724)
* ロードバランシングのために、アップストリームの`hash_on`に`path`、`uri_capture`、`query_arg`
  オプションを追加しました。
  [\#8701](https://github.com/Kong/kong/pull/8701)

* Unixドメインソケットベースの`lua-resty-events`
  を導入して、共有メモリベースの`lua-resty-worker-events`を置き換えました。
  [\#8890](https://github.com/Kong/kong/pull/8890)

* エンティティ向けに`table_name`フィールドを導入しました。
  このフィールドでテーブル名を指定できます。以前は、名前はエンティティ`name`属性によって推測されていました。
  [\#9182](https://github.com/Kong/kong/pull/9182)

* アップストリーム向けのアクティブヘルスチェックに`headers`を追加しました。
  [\#8255](https://github.com/Kong/kong/pull/8255)

* ホスト名を使用するターゲットエンティティは、不要な場合にも解決されていました。現在、ターゲットが削除または更新されると、そのターゲットに関連付けられているDNSレコードは、解決されるホスト名のリストから削除されるようになっています。
  [\#8497](https://github.com/Kong/kong/pull/8497) [9265](https://github.com/Kong/kong/pull/9265)

* DNSコードのエラー処理とデバッグ情報が改善されました。
  [\#8902](https://github.com/Kong/kong/pull/8902)

* Kong Gatewayは、プレフィックスディレクトリでダングリング状態になっているUnixソケットを検出して削除することで、不正なシャットダウンからの回復を試みるようになりました。
  [\#9254](https://github.com/Kong/kong/pull/9254)

* 新しいCLIコマンド、`kong migrations status`は、移行ステータスをJSONファイル形式で生成します。

* `AAAA`が`dns_order`を使用する実験であるという警告が削除されました。

#### パフォーマンス

* Kong Gatewayはハイブリッドモードのコントロールプレーン（CP）ノードに不要なイベントハンドラを登録しなくなりました。 [\#8452](https://github.com/Kong/kong/pull/8452)。
* パフォーマンスを向上するために新しいタイマーライブラリを採用しています（プラグインサーバーを除く）。 [\#8912](https://github.com/Kong/kong/pull/8912)
* `additional_section`をデフォルトでアクティベートすることで、DNSクエリでのキャッシュの使用を増加させました。 [\#8895](https://github.com/Kong/kong/pull/8895)
* `pdk.request.get_header`はより高速な実装に変更されており、呼び出されるたびにすべてのヘッダーが取得されることはなくなっています。 [\#8716](https://github.com/Kong/kong/pull/8716)
* データプレーン（DP）でルーター、プラグインイテレータ、バランサーが条件付きで再ビルドされるようになっています。 [\#8519](https://github.com/Kong/kong/pull/8519)、[\#8671](https://github.com/Kong/kong/pull/8671) 
* 譲歩することでコードを読み込む構成をより協調的にしました。 [\#8888](https://github.com/Kong/kong/pull/8888)
* JSONでなく、LuaJITエンコーダーを使用して、LMDBでの値のシリアル化を高速化します。 [\#8942](https://github.com/Kong/kong/pull/8942)
* インフレートとJSONデコーディングを同時に実行しないことで、ブロックを回避し、データプレーン（DP）の再読み込みを迅速化します。 [\#8959](https://github.com/Kong/kong/pull/8959)
* 一部のイベントの重複を停止しました。 [\#9082](https://github.com/Kong/kong/pull/9082)
* `string.buffer`と`tablepool`を使用することで、構成ハッシュの計算パフォーマンスを向上させました。 [\#9073](https://github.com/Kong/kong/pull/9073)
* LMDBのルートとサービスにKongキャッシュを使用しないことで、DBレスモードでのキャッシュの使用量を減らしました。 [\#8972](https://github.com/Kong/kong/pull/8972)

#### Admin API

* タイマー統計とワーカー情報を取得するために、新しい`/timers` Admin APIエンドポイントを追加しました。 [\#8912](https://github.com/Kong/kong/pull/8912) [\#8999](https://github.com/Kong/kong/pull/8999)
* `/`エンドポイントにプラグインの優先度が含まれるようになりました。 [\#8821](https://github.com/Kong/kong/pull/8821)

#### ハイブリッドモード

* wRPCプロトコルのサポートが追加され、wRPC経由で構成を同期できるようになりました。 wRPCはProtoBufでエンコードし、WebSocketで転送するRPCプロトコルです。 [\#8357](https://github.com/Kong/kong/pull/8357) 
  * 以前のバージョンとの互換性を維持するために、コントロールプレーン（CP）が前のプロトコルにフォールバックして古いデータプレーン（DP）に対応できるようなサポートを追加しました。 [\#8834](https://github.com/Kong/kong/pull/8834)
  * wRPCプロトコルでサポートされるサービスを交渉するためのサポートを追加しました。 今後もサポート可能なサービスを拡充していきます。 [\#8926](https://github.com/Kong/kong/pull/8926)

* 宣言型構成のエクスポートがPostgreSQLのトランザクション内で実行されるようになりました。 [\#8586](https://github.com/Kong/kong/pull/8586)

#### プラグイン

* バージョン3\.0以降、バンドルされたプラグインのすべてのバージョンは
  Kong Gatewayバージョンと同じです。
  [\#8772](https://github.com/Kong/kong/pull/8772)

  [プラグインのドキュメント](/hub/)で言及されるのが、個々のプラグインのバージョンでなく、Kong Gatewayのバージョンになりました。
* **新しいプラグイン** ：

  * [OpenTelemetry](/hub/kong-inc/opentelemetry/)（`opentelemetry`）

    OTLP/HTTPの互換性のあるバックエンドにトレーシング計測をエクスポートします。
    `opentelemetry_tracing`
    の構成は、Kong Gatewayのコアトレーシングスパンを収集するために有効にされる必要があります。
    [\#8826](https://github.com/Kong/kong/pull/8826)
  * [TLS Handshake Modifier](/hub/kong-inc/tls-handshake-modifier/)（`tls-handshake-modifier`）

    同じリクエストに対応する他のプラグインに証明書を利用可能にします。
  * [TLS Metadata Headers](/hub/kong-inc/tls-metadata-headers/)（`tls-metadata-headers`）

    HTTPヘッダー経由でTLSクライアント証明書のメタデータをアップストリームサービスにプロキシします。
  * [WebSocket Size Limit](/hub/kong-inc/websocket-size-limit/)（`websocket-size-limit`）

    受信WebSocketメッセージの最大サイズを演算子で指定できるようにします。
  * [WebSocket Validator](/hub/kong-inc/websocket-validator/)（`websocket-validator`）

    個々のWebSocketメッセージをユーザー指定スキーマに照らして検証してからプロキシします。

* [ACME](/hub/kong-inc/acme/)（`acme`）

  * `allow_any_domain`フィールドを追加しました。デフォルトではfalseに設定されており、trueに設定するとゲートウェイは `domains`フィールドを無視します。 [\#9047](https://github.com/Kong/kong/pull/9047)

* [AWS Lambda](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * `aws_assume_role_arn`および `aws_role_session_name`構成パラメータを通じて、アカウント間での呼び出しに対応できるようになりました。 [\#8900](https://github.com/Kong/kong/pull/8900)
  * このプラグインは、プロキシ統合モードで動作する際、有効な戻り値として文字列タイプ`statusCode` を受け入れるようになりました。 [\#8765](https://github.com/Kong/kong/pull/8765)
  * このプラグインは、IAMロールのARNごとにAWS認証情報キャッシュを分離するようになりました。 [\#8907](https://github.com/Kong/kong/pull/8907)

* コレクター（`collector`）

  * 非推奨のコレクタープラグインは削除されました。

* [DeGraphQL](/hub/kong-inc/degraphql/)（`degraphql`）

  * GraphQLサーバーパスが`graphql_server_path`構成パラメータで構成できるようになりました。

* [Kafka Upstream](/hub/kong-inc/kafka-upstream/)（`kafka-upstream`）と
  [Kafka Log](/hub/kong-inc/kafka-log)（`kafka-log`）

  * `SCRAM-SHA-512`認証メカニズムのサポートを追加しました。

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * このプラグインでグループメンバーシップに基づいた認証が可能になりました。 新しい構成パラメータである`groups_required`は文字列の配列要素で、リクエストが承認されるためにユーザーが必ず所属しなければならないグループを示します。 
  * グループ属性に記号「`.`」を使用できるようになりました。
  * パスワードフィールドに記号「`:`」を使用できるようになりました。

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * 証明書失効リスト（CRL）とOCSPサーバーのサポートが次のパラメータで導入されました：`http_proxy_host`、`http_proxy_port`、`https_proxy_host`、`https_proxy_port`。

* [OPA](/hub/kong-inc/opa/)（`opa`）

  * 新しい構成パラメータ`include_body_in_opa_input`：有効にされると、`input.request.http.body`でのOPA入力の文字列として生のボディが、ボディサイズ`input.request.http.body_size`で含まれるようになります。

  * 新しい構成パラメータ`include_parsed_json_body_in_opa_input`：有効にされ、かつコンテンツタイプが`application/json`である場合、`input.request.http.parsed_body`でのOPA入力の文字列入力に解析されたJSONが追加されます。

* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`）

  * 高カーディナリティメトリクスがデフォルトで無効になりました。

  * メトリクスの収集時にプロキシトラフィックへのパフォーマンスペナリティを引き下げました。

  * 次のメトリクス名は、可能な限り標準化するために単位を追加する調整を加えました。

    *  `http_status`を`http_requests_total`に変更。

    *  `latency`から`kong_request_latency_ms`（HTTP）、`kong_upstream_latency_ms`、`kong_kong_latency_ms`、`session_duration_ms`（ストリーム）。

      Kongとアップストリームのレイテンシは、それぞれ異なるレベルのコマンドで動作する可能性があります。メモリのオーバーヘッドを削減するには、これらのバケットを分離します。
    *  `kong_bandwidth`を`kong_bandwidth_bytes`に変更。

    *  `nginx_http_current_connections`と`nginx_stream_current_connections`は、`nginx_hconnections_total`（または`nginx_current_connections`）に統合されました。

    *  `request_count`と`consumer_status`は、http\_requests\_totalに統合されました。

      `per_consumer`構成が`false`に設定されている場合、`consumer`ラベルは空になります。`per_consumer`構成が`true`の場合、`consumer`ラベルが入力されます。

  * 次のメトリクスを削除しました。`http_consumer_status`

  * 新しいメトリクス：

    * `session_duration_ms`：ストリーム接続を監視します。
    * `node_info`：ノードのIDとKong Gatewayバージョンを出力する、1に設定された単一ゲージ。

  * `http_requests_total`に新しいラベル`source`が追加されました。`exit`、`error`、`service`のいずれかに設定できます。

  * すべてのメモリメトリクスに新しいラベル、`node_id`が付けられました。

  * Kongに同梱されているGrafanaダッシュボードを更新しました

* [StatsD](/hub/kong-inc/statsd/)（`statsd`）

  * **オープンソースプラグインの新機能** ：[StatsD Advanced](/hub/kong-inc/statsd-advanced/)プラグインのすべての機能が[StatsD](https://docs.konghq.com/hub/kong-inc/statsd)プラグインにバンドルされるようになりました。 [\#9046](https://github.com/Kong/kong/pull/9046)

* [Zipkin](/hub/kong-inc/zipkin/)（`zipkin`）

  * `http_span_name`構成パラメータで、スパン名にHTTPパスを含めるためのサポートを追加しました。 [\#8150](https://github.com/Kong/kong/pull/8150)
  * `connect_timeout`、`send_timeout`、`read_timeout`構成パラメータを通じて、ソケット接続と送信/読み取りタイムアウトのサポートを追加しました。 アップストリームコレクターを利用できないか動作が遅い場合に、`ngx.timer`が飽和しにくくなります。 [\#8735](https://github.com/Kong/kong/pull/8735)

#### 構成

* 開発者とオペレーターは、Kong Gatewayの実行時に使用するOpenRestyインストールを、システムにインストール済みのものを使用するのでなく、`openresty_path`の構成を通じて指定できるようになりました。 [\#8412](https://github.com/Kong/kong/pull/8412)
* リッスンオプション、`admin_listen`、`proxy_listen`、`stream_listen`に`ipv6only`が追加されました。 [\#9225](https://github.com/Kong/kong/pull/9225)
* リッスンオプション、`admin_listen`、`proxy_listen`、`stream_listen`に`so_keepalive`が追加されました。 [\#9225](https://github.com/Kong/kong/pull/9225)
* LMDB DBレス構成を永続化し、JSONベースの構成キャッシュを削除して起動時間を短縮します。 [\#8670](https://github.com/Kong/kong/pull/8670)
* `nginx_events_worker_connections=auto`の下限は、現在、1024です。 [\#9276](https://github.com/Kong/kong/pull/9276)
* `nginx_main_worker_rlimit_nofile=auto`の下限は、現在、1024です。 [\#9276](https://github.com/Kong/kong/pull/9276)

#### PDK

* 新しいPDK関数、`kong.request.get_start_time()`を追加しました。 この関数は、リクエストの開始時刻をUnixエポックミリ秒単位で返します。 [\#8688](https://github.com/Kong/kong/pull/8688)
* `cache_key`から何も見つからない場合、`kong.db.*.cache_key()`関数は`.id`にフォールバックするようになりました。 [\#8553](https://github.com/Kong/kong/pull/8553)

### 既知の制限

* Kong Managerは現在、以下の機能をサポートしていません。

  * シークレット管理
  * プラグインの順序付け
  * 式ベースのルーティング

* 2\.8\.x（およびその前）から3\.0\.xへのブルーグリーン移行はサポートされていません。

  * これは既知の問題であり、次の2\.8リリースで修正される予定です。これがアップグレードの要件である場合、Kong演算子は3\.0\.0\.0へのアップグレードを開始する前に、そのバージョンにアップグレードする必要があります。 
  * 詳細については、[Kong Gatewayのアップグレード](/gateway/latest/upgrade/)を参照してください。

* OpenTracing：このリリースでは`nginx-opentracing`に問題があるため、OpenTracingユーザーは当面アップグレードを控えることをお勧めします。
  この問題は次回パッチ/マイナーリリースで修正される予定です。

* Kong Gateway FIPSパッケージは現在、SSLのPostgreSQL接続と互換性がありません。

### 下位互換性のない変更と非推奨

#### デプロイメント

* Amazon Linux 1のコンテナとパッケージは非推奨となり、製造が中止されました。 Amazon Linux 1は、[2020年12月31日にサポート終了（EOL）となりました](https://aws.amazon.com/blogs/aws/update-on-amazon-linux-ami-end-of-life)。 [Kong/docs.konghq.com \#3966](https://github.com/Kong/docs.konghq.com/pull/3966)
* Debian 8（Jessie）コンテナとパッケージは非推奨となり、製造が中止されました。 Debian 8は、[2020年6月30日にサポート終了（EOL）となりました](https://www.debian.org/News/2020/20200709)。 [Kong/kong\-build\-tools \#448](https://github.com/Kong/kong-build-tools/pull/448)

#### コア

* 3\.0以降、Kong Gatewayスキーマライブラリの`process_auto_fields`機能は、`select`コンテキストが指定されている場合、渡されるデータをディープコピーしません。その理由は、データがほとんどの場合、`pgmoon`や`lmdb`のようなドライバーから取得されると考えられるケースにおいて、テーブルの過度なディープコピーを防ぐことにあります。

  カスタムプラグインで所定のテーブルが上書きされないように
  `process_auto_fields`
  が使用される場合、このパラメータを関数にすぐ渡す前に、独自のコピーを作成する必要があります。[\#8796](https://github.com/Kong/kong/pull/8796)
* KongプラグインまたはDAOスキーマの非推奨の`shorthands`フィールドは削除され、代わりに入力された`shorthand_fields`が使用されます。
  カスタムスキーマでまだ
  `shorthands`を使用している場合は、`shorthand_fields`
  を使用するように更新する必要があります。[\#8815](https://github.com/Kong/kong/pull/8815)

* `legacy = true/false`
  属性のサポートがKongスキーマとKongフィールドスキーマから削除されました。
  [\#8958](https://github.com/Kong/kong/pull/8958)

* `Kong.serve_admin_api`の非推奨のエイリアスが削除されました。カスタムNginxテンプレートをまだ使用している場合は、`Kong.admin_content`に変更してください。
  [\#8815](https://github.com/Kong/kong/pull/8815)

* Kongシングルトンモジュール、`kong.singletons`は、PDK `kong.*`が優先されるため、削除されました。
  [\#8874](https://github.com/Kong/kong/pull/8874)

* データプレーン（DP）構成キャッシュが削除されました。
  構成はLMDBによって自動的に永続化されるようになりました。
  [\#8704](https://github.com/Kong/kong/pull/8704)

* `ngx.ctx.balancer_address`は、`ngx.ctx.balancer_data`が優先されるため、削除されました。[\#9043](https://github.com/Kong/kong/pull/9043)

* `route.path`の正規化ルールが変更されました。Kong Gatewayは正規化されていないパスを保存しますが、正規表現パスは常に
  正規化されたURIとパターンマッチします。以前、Kong Gatewayは、異なる形式のURIマッチを確実にするために、
  正規表現パスパターンでパーセントエンコーディングを置き換えていました。
  この仕組みはサポートされなくなり、[rfc3986](https://datatracker.ietf.org/doc/html/rfc3986#section-2.2)
  で定義される予約文字を除くすべての文字が、パーセントエンコーディングなしに書き込まれるようになっています。
  [\#9024](https://github.com/Kong/kong/pull/9024)

* Kong Gatewayでは、`route.path`が正規表現パターンかどうかを推測するのにヒューリスティックを使用しなくなりました。3\.0以降は、すべての正規表現パスは`"~"`プレフィックスで始める必要があり、`"~"`で始まらないパスはすべてプレーンテキストと見なされます。
  移行プロセスでは、2\.xから3\.0にアップグレードする際に、正規表現パスが自動的に変換されます。
  [\#9027](https://github.com/Kong/kong/pull/9027)

* `route.path`での変更に合わせて、宣言型構成のバージョン番号（`_format_version`）を`3.0`にアップグレードしました。
  古いバージョンを使用する宣言型構成は、移行中に`3.0`にアップグレードされます。

  {:.important}
  > 
  > **宣言型構成ファイルは2\.8以前から3\.0に同期（`deck sync`）しないでください。**
  > 古い構成ファイルが構成を上書きし、互換性の問題を引き起こします。
  > 更新された構成を取得するには、移行完了後に3\.0ファイルの`deck dump`を実行します。

  [\#9078](https://github.com/Kong/kong/pull/9078)
* タグにスペースを含めることができるようになりました。
  [\#9143](https://github.com/Kong/kong/pull/9143)

* `nginx-opentracing`モジュールのサポートは、`3.0`
  以降、非推奨となり、`4.0`でKongから削除されます（詳細は、[既知の制限事項](#known-limitations)
  セクションを参照）。

* atc\-routerで、正規表現[look\-around](https://www.regular-expressions.info/lookaround.html)と[backreferences](https://www.regular-expressions.info/backref.html)のサポートを除外しました。これらは使用頻度の低い機能であるため、サポートを排除すると正規表現のマッチング速度が向上します。現在、look\-aroundまたはbackreferencesを使用している正規表現がある場合、Kongを起動しようとするとエラーが表示され、互換性がない正規表現が明示されます。look\-aroundまたはbackreferences機能を排除するには、`traditional`ルーターフレーバーに切り替えるか、正規表現を変更します。

#### Admin API

* Admin API エンドポイントの`/vitals/reports`は削除されました。
* `POST`リクエストが`/targets` エンドポイントに対するものである場合、既存エンティティを更新できなくなりました。このリクエストでは新しいエンティティの作成のみできます。 [\#8596](https://github.com/Kong/kong/pull/8596)、 [\#8798](https://github.com/Kong/kong/pull/8798)。`/targets` の変更に`POST`リクエストを使用するスクリプトがある場合、そのスクリプトを適切なエンドポイントへの`PUT` リクエストに変更してから、{{site.base_gateway}} 3\.0に更新してください。
* 重複したターゲットに対する挿入および更新操作では`409`エラーが返されます。 [\#8179](https://github.com/Kong/kong/pull/8179)、 [\#8768](https://github.com/Kong/kong/pull/8768)
* サーバーで利用可能と報告されたプラグインのリストが、ブール値の`true`ではなく、プラグインごとのメタデータのテーブルを返すようになりました。 [\#8810](https://github.com/Kong/kong/pull/8810)

#### PDK

* `kong.request.get_path()` PDK関数は、呼び出し元に返される文字列に対してパスの正規化を実行するようになりました。 リクエストパスの正規化されていない生のバージョンは、`kong.request.get_raw_path()` 経由で取得できます。 [\#8823](https://github.com/Kong/kong/pull/8823)
* `pdk.response.set_header()`、`pdk.response.set_headers()`、`pdk.response.exit()`は、手動で設定された`Transfer-Encoding`ヘッダーを無視し、警告を発するようになりました。 [\#8698](https://github.com/Kong/kong/pull/8698)
* PDKのバージョン管理は終了しました。 [\#8585](https://github.com/Kong/kong/pull/8585)
* JavaScript PDKは、`kong.request.getRawBody`、`kong.response.getRawBody`、`kong.service.response.getRawBody`で`Uint8Array`を返すようになりました。 Python PDKは、`kong.request.get_raw_body`、`kong.response.get_raw_body`、`kong.service.response.get_raw_body` で`bytes`を返します。 以前は、これらの関数は文字列を返していました。 [\#8623](https://github.com/Kong/kong/pull/8623)
* `go_pluginserver_exe`および`go_plugins_dir`ディレクティブはサポートされなくなりました。 [\#8552](https://github.com/Kong/kong/pull/8552)。[Goプラグインサーバー](https://github.com/Kong/go-pluginserver) を使用している場合、アップグレードする前に [Go PDK](https://github.com/Kong/go-pdk)を使用するためにプラグインを移行してください。

#### プラグイン

* プラグインのDAOは、読み込み順序が明示的になるように配列に列挙することが義務付けられました。ハッシュのようなテーブルへの読み込みは、サポートされなくなりました。
  [\#8988](https://github.com/Kong/kong/pull/8988)

* プラグインの`handler.lua`ファイルに有効な`PRIORITY`（整数）と`VERSION`（"x.y.z" 形式）フィールドが必ず必要になりました。
  これらがなければ、プラグインの読み込みに失敗します。
  [\#8836](https://github.com/Kong/kong/pull/8836)

* 古い`kong.plugins.log-serializers.basic`ライブラリは、PDK関数、`kong.log.serialize`が優先されるため、削除されました。PDKを使用するにはプラグインをアップグレードしてください。
  [\#8815](https://github.com/Kong/kong/pull/8815)

* 非推奨となっていたレガシーのプラグインスキーマのサポートは削除されました。カスタムプラグインでなお使用されている古い（`0.x era`）スキーマがある場合は、アップグレードする必要があります。
  [\#8815](https://github.com/Kong/kong/pull/8815)

* 一部のプラグインの優先度を更新しました。

  これはプラグインの実行順序に影響する可能性があるため、カスタムプラグインを実行するユーザーにとって重要ですが、
  Kong Gatewayの標準的なインストールにおけるプラグインの実行順序は変更されません。

  古いプラグインと新しいプラグインの優先度の値：
  * `acme`を`1007`から次の値へ変更：`1705`
  * `basic-auth`を`1001`から次の値へ変更：`1100`
  * `canary`を`13`から次の値へ変更：`20`
  * `degraphql`を`1005`から次の値へ変更：`1500`
  * `graphql-proxy-cache-advanced`を`100`から次の値へ変更：`99`
  * `hmac-auth`を`1000`から次の値へ変更：`1030`
  * `jwt`を`1005`から次の値へ変更：`1450`
  * `jwt-signer`を`999`から`1020`へ変更。
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

* Kongプラグインは、`CREDENTIAL_USERNAME`（`X-Credential-Username`）をサポートしなくなりました。
  認証情報にアップストリームヘッダーを設定する場合は、定数`CREDENTIAL_IDENTIFIER`（`X-Credential-Identifier`）を使用してください。
  [\#8815](https://github.com/Kong/kong/pull/8815)

* [ACL](/hub/kong-inc/acl/)（`acl`）、[Bot Detection](/hub/kong-inc/bot-detection/)（`bot-detection`）、[IP Restriction](/hub/kong-inc/ip-restriction/)（`ip-restriction`）

  * 非推奨の構成パラメータ、`blacklist`と`whitelist`を削除しました。 [\#8560](https://github.com/Kong/kong/pull/8560)

* [ACME](/hub/kong-inc/acme/)（`acme`）

  * `auth_method`構成パラメータのデフォルト値が、`token`になりました。

* [AWS Lambda](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * AWSリージョンが必須になりました。このリージョンは、`aws_region`フィールドパラメータまたは環境変数を使用してプラグイン構成から設定できます。
  * このプラグインで`host`フィールドと`aws_region`フィールドを同時に設定できるようになりました。また、SigV4署名が常に適用されます。 [\#8082](https://github.com/Kong/kong/pull/8082)

* [HTTP Log](/hub/kong-inc/http-log/)（`http-log`）

  * `headers` フィールドには以前は値の配列を入力できましたが、現在はヘッダー名ごとに1つの文字列のみ入力可能です。 [\#6992](https://github.com/Kong/kong/pull/6992)

* [JWT](/hub/kong-inc/jwt/)（`jwt`）

  * 認証されたJWTは nginxコンテキスト（`ngx.ctx.authenticated_jwt_token`）に挿入できなくなりました。その名前で設定されている値に依存するカスタムプラグインは、3\.0にアップグレードする前に、Kongの共有コンテキスト（`kong.ctx.shared.authenticated_jwt_token`）を代わりに使用するように更新する必要があります。 

* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`）

  * 高カーディナリティメトリクスがデフォルトで無効になりました。

  * メトリクスの収集時にプロキシトラフィックへのパフォーマンスペナリティを引き下げました。

  * 次のメトリクス名は、可能な限り標準化するために単位を追加する調整を加えました。

    *  `http_status`を`http_requests_total`に変更。

    *  `latency`から`kong_request_latency_ms`（HTTP）、`kong_upstream_latency_ms`、`kong_kong_latency_ms`、`session_duration_ms`（ストリーム）。

      Kongとアップストリームのレイテンシは、それぞれ異なるレベルのコマンドで動作する可能性があります。メモリのオーバーヘッドを削減するには、これらのバケットを分離します。
    *  `kong_bandwidth`を`kong_bandwidth_bytes`に変更。

    *  `nginx_http_current_connections`と`nginx_stream_current_connections`は、`nginx_connections_total`に統合されました。

    *  `request_count`と`consumer_status`は、`http_requests_total`に統合されました。

      `per_consumer`構成が`false`に設定されている場合、`consumer`ラベルは空になります。`per_consumer`構成が`true`の場合、`consumer`ラベルが入力されます。

  * 次のメトリクスを削除しました。`http_consumer_status`

  * 新しいメトリクス：

    * `session_duration_ms`：ストリーム接続を監視します。
    * `node_info`：ノードのIDとKong Gatewayバージョンを出力する、1に設定された単一ゲージ。

  * `http_requests_total`に新しいラベル`source`が追加されました。`exit`、`error`、`service`のいずれかに設定できます。

  * すべてのメモリメトリクスに新しいラベル、`node_id`が付けられました。

  * Kongに同梱されているGrafanaダッシュボードを更新しました
    [\#8712](https://github.com/Kong/kong/pull/8712)

  * このプラグインはステータスコード、レイテンシ、帯域幅、アップストリームのヘルスチェックメトリクスをデフォルトでエクスポートしませんが、
    今後も、`status_code_metrics`、`lantency_metrics`、`bandwidth_metrics`、`upstream_health_metrics`をそれぞれ設定することでこの機能をオンにできます。[\#9028](https://github.com/Kong/kong/pull/9028)

* **[Pre\-function](/hub/kong-inc/pre-function/)（`pre-function`）と[Post\-function](/hub/kong-inc/post-function/)** （`post-function`）

  * 非推奨の`config.functions`構成パラメータをサーバーレス関数プラグインのスキーマから削除しました。 代わりに、`config.access`フェーズを使用してください。 [\#8559](https://github.com/Kong/kong/pull/8559)

* [StatsD](/hub/kong-inc/statsd/)（`statsd`）：

  * サービスに関連するすべてのメトリクス名に`service.`プレフィックス`kong.service.<service_identifier id="sl-md0000000">.request.count`が付くようになりました。 
    * メトリクス`kong.<service_identifier id="sl-md0000000">.request.status.<status id="sl-md0000000">`は`kong.service.<service_identifier id="sl-md0000000">.status.<status id="sl-md0000000">`に改名されました。
    * メトリクス`kong.<service_identifier id="sl-md0000000">.user.<consumer_identifier id="sl-md0000000">.request.status.<status id="sl-md0000000">`は`kong.service.<service_identifier id="sl-md0000000">.user.<consumer_identifier id="sl-md0000000">.status.<status id="sl-md0000000">`に改名されました。

  * メトリクス`status_count`および`status_count_per_user`からメトリクス`*.status.<status id="sl-md0000000">.total`が削除されました。

  [\#9046](https://github.com/Kong/kong/pull/9046)
* [Rate Limiting](/hub/kong-inc/rate-limiting/)（`rate-limiting`）、[Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）、[Response Rate Limiting](/hub/kong-inc/response-ratelimiting/)（`response-ratelimiting`）：

  * デフォルトポリシーがすべてのデプロイメントモードでローカルになりました。 [\#9344](https://github.com/Kong/kong/pull/9344)

* **非推奨** ：[StatsD Advanced](/hub/kong-inc/statsd-advanced/)（`statsd-advanced`）：

  * StatsD Advancedプラグインは非推奨となり、4\.0で削除されます。 現在、すべての機能が[StatsD](/hub/kong-inc/statsd/)プラグインで利用できるようになっています。

* [Proxy Cache](/hub/kong-inc/proxy-cache/)（`proxy-cache`）、[Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)（`proxy-cache-advanced`）、[GraphQL Proxy Cache Advanced](/hub/kong-inc/graphql-proxy-cache-advanced/)（`graphql-proxy-cache-advanced`）

  * 上記プラグインは`ngx.ctx.proxy_cache_hit`に応答データを保存しなくなりました。 現在、応答データを必要とするログ取得プラグインは、データを`kong.ctx.shared.proxy_cache_hit`から読み取る必要があります。 [\#8607](https://github.com/Kong/kong/pull/8607)

#### 構成

* `X-Credential-Username`の値を持つKong定数`CREDENTIAL_USERNAME` は削除されました。 [\#8815](https://github.com/Kong/kong/pull/8815)
* `lua_ssl_trusted_certificate`のデフォルト値が`system`に変更されました [\#8602 は](https://github.com/Kong/kong/pull/8602)、システム CA ストアから信頼された CA リストを自動的に読み込みます。
* `.lua`形式を使用して CLI ツールの`kong`から宣言型構成ファイルをインポートできなくなりました。 JSON形式とYAML形式のみがサポートされています。Kong Gatewayのアップデート手順で `kong config db_import config.lua`を実行する場合は、アップグレード前に`config.lua`ファイルを`config.json`または`config.yml`ファイルに変換します。 [\#8898](https://github.com/Kong/kong/pull/8898)
* データプレーンの設定キャッシュメカニズムとそれに関連する設定オプション \(`data_plane_config_cache_mode` と `data_plane_config_cache_path`\) は削除され、LMDBが優先されました。

#### 移行

* 移行ヘルパーライブラリ \(主にカサンドラの移行に使用\) は、Kong Gateway では提供されなくなりました。 [\#8781](https://github.com/Kong/kong/pull/8781)
* PostgreSQLの移行には、カサンドラの移行のように `up_f`の部分を含め、呼び出す関数を指定できるようになりました `up_f`部分は `up`部分が PostgreSQL とカサンドラ両方のデータベースに対して実行された後に呼び出されます 

### 修正

#### Enterprise

* [キーリングモジュール](/gateway/latest/kong-enterprise/db-encryption/)の初期化中にエラーが発生すると、コントロールプレーン（CP）がクラッシュするキーリング暗号化に関する問題を修正しました。

* ソフトウェアの再読み込み後に、キーリングモジュールがキーを復号化しない問題を修正しました。

* 次のページ分割の問題を修正しました。

  * コンシューマのページ分割に関する問題を修正しました。
  * DAOを使用して外部キーフィールドを反復処理しているときに2番目のページを読み込むときに発生する問題を修正しました。 [\#9255](https://github.com/Kong/kong/pull/9255)

* コントロールプレーン（CP）の再起動後にサービスルートの更新に失敗する問題を修正しました。

* **Vitals** :

  * データプレーン（DP）上の`anonymous_reports`で`phone_home`を無効にしました。
  * Kong Gatewayのバージョン情報が、テレメトリリクエストクエリパラメータで送信されるようになりました。

* **Kong Manager** ：

  * ワークスペースダッシュボードの読み込み状態を修正しました。
    以前は、リクエストデータがなくても、既存のサービスがあるダッシュボードで、ユーザーにサービスの追加を求めるプロンプトが表示されていました。

  * DatadogプラグインでサポートされていないメトリクスをKong Managerで選択できる問題を修正しました。

  * Kong Managerのアップストリーム構成で使用可能な値を修正しました。
    以前は、本来、小数を使用できるフィールドで整数のみが受け入れられていました。

  * 更新後の値にカンマ（`,`）が含まれている場合、`pre-function`プラグイン構成を保存も更新もできない問題を修正しました。

  * サービス契約ページのサービス名フィールドに、サービス表示名が正しく表示されるようになりました。
    従来は、サービスIDが表示されていました。

  * CA証明書の更新後に、ページが証明書ビューに戻らない問題を修正しました。

  * サービス概要ページのサービスURLにポートが欠落している問題を修正しました。

  * ワークスペースダッシュボードでページを切り替えてもDev Portal URLが更新されない問題を修正しました。

  * プラグインに関する次の問題を修正しました。

    * Exit Transformerプラグインは、Kong Managerを通じて追加されたLua関数を読み込めるようになりました。
    * CORSプラグインが`config.origins`フィールドの正規表現を適切に処理するようになりました。
    * Datadogプラグインが`tags`フィールドに配列を受け入れるようになりました。以前は、誤って文字列が通常設定となっていました。

  * **ホスト** 列に基づきルートを並び替えてから、ページ分割されたリストで **次へ** をクリックすると発生していた`HTTP 500`エラーを修正しました。

  * 開発者ロールの割り当てがKong Managerに表示されない問題を修正しました。
    従来は、Dev Portalセクションの「権限」タブにロールが表示される場合、開発者リストは新しい開発者が追加されても更新されませんでした。
    Kong Managerは、Dev Portalの割り当て先を取得する際、間違ったURLを構築していました。

  * 管理者がデフォルトのワークスペースにロールがない場合、ワークスペースを切り替えられない問題を修正しました。

  * Kong ManagerのDev Portal設定での表示の問題を修正しました。

  * リソースに対する権限を持たない管理者ロールを表示しようとすると発生するエラーを改善しました。
    「`404 workspace not found`」というメッセージが表示される代わりに、ロールの閲覧権限がないことがユーザーに通知されるようになりました。

* Nginxの再読み込み後にデータプレーン（DP）が再読み込みされ、ライセンスが失われる問題を修正しました。

* 依存関係の問題を修正しました:

  * `kong-gql`: 変数定義を修正し、非 null 型またはリスト型の変数を正しく処理できるようにしました。
  * `lua-resty-openssl-aux-module`: リクエストから`SSL_CTX`を取得する際の問題を修正しました。

#### コア

* スキーマバリデーターは、`null`を宣言型から正しく変換するようになりました。 構成を`nil`に変更します。 [\#8483](https://github.com/Kong/kong/pull/8483)
* Kongは、ルーターとプラグインイテレータのタイマーを、前の実行が終了した後にのみ再スケジュールし、不必要な同時実行を回避できるようになりました。 [\#8567](https://github.com/Kong/kong/pull/8567)
* 外部プラグインが返された JSON を null メンバーで正しく処理するようになりました。 [\#8611](https://github.com/Kong/kong/pull/8611)
* 環境変数のアドレスが変更されても、初期化実行後にそのアドレスが復旧したかどうかが チェックされないという問題を修正しました。 [\#8581](https://github.com/Kong/kong/pull/8581)
* Goプラグインサーバインスタンスが再起動に更新されない 問題を修正しました。 [\#8547](https://github.com/Kong/kong/pull/8547)
* Kongが読み込み中にDNSリゾルブタイマーを再スケジュールしようとすると発生する問題を修正しました。 [\#8702](https://github.com/Kong/kong/pull/8702)
* プライベートストリーム API がより大きなメッセージペイロードに対応するために書き直されました。 [\#8641](https://github.com/Kong/kong/pull/8641)
* `PATCH`メソッドの使用時にアップストリームに送信されたクライアント証明書が更新されない問題を修正しました。 [\#8934](https://github.com/Kong/kong/pull/8934)
* コントロールプレーン（CP）と wRPC モジュールの相互作用により、 `pcall`なしで`export_deflated_reconfigure_payload`を呼び出すと Kong がクラッシュする問題を修正しました。 [\#8668](https://github.com/Kong/kong/pull/8668)
* すべての`.proto`ファイルを`/usr/local/kong/include`に移動し、優先度順に並べ替えました。 [\#8914](https://github.com/Kong/kong/pull/8914)
* 無効なオプションを使用して構成を作成または更新すると、予期しない404エラーが発生する問題を修正しました。 [\#8831](https://github.com/Kong/kong/pull/8831)
* 一部の PDK API を呼び出すときにクラッシュが発生する問題を修正しました。 [\#8604](https://github.com/Kong/kong/pull/8604)
* go PDK 呼び出しが配列を返すときにクラッシュが発生する問題を修正しました。 [\#8891](https://github.com/Kong/kong/pull/8891)
* Kong が終了すると、プラグインサーバーも正常にシャットダウンされるようになりました。 [\#8923](https://github.com/Kong/kong/pull/8923)
* CLI はデフォルトで`y`を使用しないため、 `[Y/n]`ではなく`[y/n]`でプロンプトを表示するようになりました。 [\#9114](https://github.com/Kong/kong/pull/9114)
* Kongが初期化時にカサンドラに接続できない場合に表示されるエラーメッセージが改善されました。 [\#8847](https://github.com/Kong/kong/pull/8847)
* `off`戦略で Vault サブスキーマがロードされない問題を修正しました。 [\#9174](https://github.com/Kong/kong/pull/9174)
* スキーマが`process_auto_fields`の前に選択変換を実行するようになりました。 [\#9049](https://github.com/Kong/kong/pull/9049)
* `worker_consistency = eventual`の場合に、 {{site.base_gateway}}アップストリームを追跡するために使用するタイマーが多すぎる問題を修正しました。 [\#8694](https://github.com/Kong/kong/pull/8694)、 [\#8858](https://github.com/Kong/kong/pull/8858)
* ホスト名のみで設定されたターゲットのホスト名のみを使用してターゲットステータスを設定できない問題を修正しました。 [\#8797](https://github.com/Kong/kong/pull/8797)
* カスケード削除後、一部のエンティティのキャッシュエントリが正しく無効にならなかった問題を修正しました。 [\#9261](https://github.com/Kong/kong/pull/9261)
* {{site.base_gateway}}がすでに実行されているときに`kong start`を実行しても、既存の`.kong_env`ファイル[\#9254](https://github.com/Kong/kong/pull/9254)を上書きされなくなりました 

#### Admin API

* Admin APIが`/status`をリクエストするときに`HTTP/2`をサポートするようになりました。 [\#8690](https://github.com/Kong/kong/pull/8690)
* Admin API が`OPTIONS`リクエストで`Allow`および`Access-Control-Allow-Methods`ヘッダーを表示しない問題を修正しました。

#### プラグイン

* 優先順位が衝突するプラグインは、名前に基づいて決定論的にソートされるようになりました。
  [\#8957](https://github.com/Kong/kong/pull/8957)

* 外部プラグイン: プラグインインスタンスがイベントハンドラーで`instances_id`を失った場合、Kong Gateway はログ記録をより適切に処理するようになりました。[\#8652](https://github.com/Kong/kong/pull/8652)

* [ACME](/hub/kong-inc/acme/)（`acme`）

  * `auth_method`構成パラメータのデフォルト値が`token`に設定されました。 [\#8565](https://github.com/Kong/kong/pull/8565)
  * `domains_matcher`のキャッシュを追加しました。 [\#9048](https://github.com/Kong/kong/pull/9048)

* [HTTP Log](/hub/kong-inc/http-log/)（`http-log`）

  * ログ出力は、プラグインが実行されているワークスペースに制限されるようになりました。以前、 本プラグインはワークスペース外からのリクエストをログに記録することができました。

* [AWS Lambda](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * プラグインのスキーマから非推奨の`proxy_scheme`フィールドを削除しました。 [\#8566](https://github.com/Kong/kong/pull/8566)
  * URI が Request Transformer プラグインによって定義されたルールに従えない問題を修正するために、パスを`request_uri`から`upstream_uri`に変更しました。 [\#9058](https://github.com/Kong/kong/pull/9058) [\#9129](https://github.com/Kong/kong/pull/9129)

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)

  * 不正な base64 エンコードによって発生するプロキシ認証エラーを修正しました。
  * Nginx リクエストホストヘッダーを上書きする場合は小文字を使用します。
  * プラグインでは、複数値の応答ヘッダーが許可できるようになりました。

* [gRPC ゲートウェイ](/hub/kong-inc/grpc-gateway/)\(`grpc-gateway`\)

  * URI引数からのブールフィールドの処理を修正しました。 [\#9180](https://github.com/Kong/kong/pull/9180)

* [HMAC Auth](/hub/kong-inc/hmac-auth/) \(`hmac-auth`\)

  * `ngx.var.uri`を使った非推奨の署名フォーマットを削除しました。[\#8558](https://github.com/Kong/kong/pull/8558)

* [LDAP Authentication](/hub/kong-inc/ldap-auth/) \(`ldap-auth`\)

  * FFI を介して OpenSSL API で ASN.1 パーサーをリファクタリングしました。 [\#8663](https://github.com/Kong/kong/pull/8663)

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * `base_dn`がドメインルートの場合に Kong Manager LDAP Authenticationが失敗する問題を修正しました。

* [Mocking](/hub/kong-inc/mocking/) \(`mocking`\)

  * `204`レスポンスが正しく処理されず、次のエラーが表示される問題を修正しました： `"No examples exist in API specification for this resource"`
  * `204`レスポンス仕様が空のコンテンツ要素に対応するようになりました。

* [OpenID Connect](/hub/kong-inc/openid-connect/) \(`openid-connect`\)
  openid\-connect

  * `kong_oauth2`コンシューママッピングの問題を修正しました。

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \) と[Response Rate Limiting](/hub/kong-inc/response-ratelimiting/)\( `response-ratelimiting` \)

  * `cluster`ポリシーが2つ以上のメトリクス \(たとえば、`second`と`day`\) で使用された場合に発生する PostgreSQL デッドロックの問題を修正しました。 [\#8968](https://github.com/Kong/kong/pull/8968)

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * `get_window`を呼び出すときのエラー処理を修正し、ウィンドウ予約にバッファを追加しました。
  * ハイブリッドモードまたはDB\-lessモードでストラテジが`cluster`に設定されている場合のプラグインストラテジー構成のエラー処理が修正されました。

* **[Pre\-function](/hub/kong-inc/pre-function/)（`pre-function`）と[Post\-function](/hub/kong-inc/post-function/)** （`post-function`）

  * クラッシュを引き起こす可能性のある問題を修正しました。 [\#9269](https://github.com/Kong/kong/pull/9269)

* [Syslog](/hub/kong-inc/syslog/) \(`syslog`\)

  * `conf.facility`のデフォルト値が`user` に設定されました。 [\#8564](https://github.com/Kong/kong/pull/8564)

* [Zipkin](/hub/kong-inc/zipkin/)（`zipkin`）

  * バランサースパンに Nginx からアップストリームへの接続時間が 含まれるように修正しました。 [\#8848](https://github.com/Kong/kong/pull/8848)
  * ヘッダーフィルターの開始時間の計算を修正しました。 [\#9230](https://github.com/Kong/kong/pull/9230)
  * [プラグインを最新のJaegerヘッダ仕様と互換性を持たせ](https://www.jaegertracing.io/docs/1.29/client-libraries/#tracespan-identity)、`parent_id`を任意にしました。 [\#8352](https://github.com/Kong/kong/pull/8352)

#### クラスタリング

* クラスターリスナーは、 `proxy_error_log` の代わりにログファイルに `admin_error_log` の値を使用するようになりました。 [\#8583](https://github.com/Kong/kong/pull/8583)
* 起動時にキャッシュに値を設定する前にKongの役割をチェックする一部のビジネスロジックの タイプミスを修正しました。[\#9060](https://github.com/Kong/kong/pull/9060)
* ハイブリッドモードで、サービスが `enabled: false` に設定されており、そのサービスにプラグインが有効になっているルートがあった場合、新しいデータプレーン（DP）は空の設定を受け取るという問題を修正しました。 [\#8816](https://github.com/Kong/kong/pull/8816)
* 新しいyield構成の読み込みコードによる競合状態を回避するために`config_version`をローカライズしました。 [\#8188](https://github.com/Kong/kong/pull/8818)

#### PDK

* `kong.response.get_source()`プラグインがアクセスフェーズで
  runtime例外をスローした場合、exitの代わりにエラーを返すようになりました。
  [\#8599](https://github.com/Kong/kong/pull/8599)

* `kong.tools.uri.normalize()`予約文字と非予約文字をより正確にエスケープするようになりました。
  [\#8140](https://github.com/Kong/kong/pull/8140)

* RFC3987のルートパスのバリデーションが削除され、オペレータが`/something|`のような無効なパスURIを持つルートを作成できるようになりました。この検証は、今後のリリースで再度追加される予定です。

### 依存関係

* `openresty`を 1\.19\.9\.1 から 1\.21\.4\.1 にアップグレードしました [\#8850](https://github.com/Kong/kong/pull/8850)
* `pgmoon`を 1\.13\.0 から 1\.15\.0 にアップグレードしました [\#8908](https://github.com/Kong/kong/pull/8908) [\#8429](https://github.com/Kong/kong/pull/8429)
* `openssl`を1\.1\.1nから1\.1\.1qまでアップグレードしました [\#9074](https://github.com/Kong/kong/pull/9074) [\#8544](https://github.com/Kong/kong/pull/8544) [\#8752](https://github.com/Kong/kong/pull/8752) [\#8994](https://github.com/Kong/kong/pull/8994)
* `resty.openssl`を 0\.8\.8 から 0\.8\.10 にアップグレードしました [\#8592](https://github.com/Kong/kong/pull/8592) [\#8753](https://github.com/Kong/kong/pull/8753) [\#9023](https://github.com/Kong/kong/pull/9023)
* `inspect`が3\.1\.2から3\.1\.3にアップグレードしました [\#8589](https://github.com/Kong/kong/pull/8589)
* `resty.acme`を 0\.7\.2 から 0\.8\.1 にアップグレードしました [\#8680](https://github.com/Kong/kong/pull/8680) [\#9165](https://github.com/Kong/kong/pull/9165)
* `luarocks`を 3\.8\.0 から 3\.9\.1 にアップグレードしました [\#8700](https://github.com/Kong/kong/pull/8700) [\#9204](https://github.com/Kong/kong/pull/9204)
* `luasec`を 1\.0\.2 から 1\.2\.0 にアップグレードしました [\#8754](https://github.com/Kong/kong/pull/8754) [\#8754](https://github.com/Kong/kong/pull/9205)
* `resty.healthcheck`を 1\.5\.0 から 1\.6\.1 にアップグレードしました [\#8755](https://github.com/Kong/kong/pull/8755) [\#9018](https://github.com/Kong/kong/pull/9018) [\#9150](https://github.com/Kong/kong/pull/9150)
* `resty.cassandra`を 1\.5\.1 から 1\.5\.2 にアップグレードしました [\#8845](https://github.com/Kong/kong/pull/8845)
* `penlight`を 1\.12\.0 から 1\.13\.1 にアップグレードしました [\#9206](https://github.com/Kong/kong/pull/9206)
* `lua-resty-mlcache`を 2\.5\.0 から 2\.6\.0 にアップグレードしました [\#9287](https://github.com/Kong/kong/pull/9287)
* Dev Portal の`lodash`を 4\.17\.11 から 4\.17\.21 にアップグレードしました
* Kong Manager の`lodash`を 4\.17\.15 から 4\.17\.21 にアップグレードしました

2\.8\.4\.12
--------------

**リリース日** ：2024/07/29

### 下位互換性のない変更と非推奨

* Debian 10 と RHEL 7 は、2024 年 6 月 30 日にサポート終了 \(EOL\) となりました。 このパッチの時点では、Kong はこれらのオペレーティングシステム用の Kong Gateway 2\.8\.x インストールパッケージまたは Docker イメージを構築していません。 Kong は、これらのシステムで実行される Kong バージョンに対して公式サポートを提供しなくなりました。

### 修正

* AWS2 x86\_64環境がクロスビルドされました。
* 非推奨のパッケージのビルドコードをクリーンアップしました。
* RPMパッケージを再配置可能にしました。

2\.8\.4\.11
--------------

**リリース日** 2024/06/22

### 修正

* DNSクライアントがDNS応答で`ADDITIONAL SECTION`のコンテンツを誤って使用していた問題を修正しました。

2\.8\.4\.10
--------------

**リリース日** 2024/06/18

### 既知の問題

* DNSクライアントがDNS応答でコンテンツ`ADDITIONAL SECTION`を誤って使用するという修正に関する問題がありました。 この問題を回避するには、このパッチの代わりに2\.8\.4\.11をインストールしてください。

### 機能

* RHEL 8のDockerイメージを追加しました。

### 修正

#### コア

*3\.7\.1\.0からバックポート* 

* **DNSクライアント** : Kong DNSクライアントが回答を解析する際に、ドメインとタイプが一致しないレコードを保存する問題を修正しました。 回答を解析するときに、RRタイプがクエリのタイプと異なる場合はレコードが無視されるようになりました。

*3\.4\.3\.9からバックポート済み* 

* **Vitals** : コントロールプレーンに接続する各データプレーンが、コントロールプレーン上で冗長なテーブルローテータータイマーの作成をトリガーする問題を修正しました。

#### プラグイン

*3\.7\.0\.0からバックポート* 

* [**Rate Limiting Advanced**](/hub/kong-inc/rate-limiting-advanced/)（`rate-limiting-advanced`）
  * `kong/tools/public/rate-limiting`をリファクタリングし、異なるプラグインを分離するための新しいインターフェース`new_instance`を追加しました。
    後方互換性のため、元のインターフェースは変更されていません。

    このライブラリをベースにしたカスタムのRate Limitingプラグインを使用している場合は、初期化コードを新しい形式に更新してください。たとえば、`local ratelimiting = require("kong.tools.public.rate-limiting").new_instance("custom-plugin-name")`などです。
    古いインターフェースは、今後のメジャーリリースで削除されます。

### 依存関係

* 予期しない入力を処理する際の`lua-cjson`のロバスト性を改善しました。

2\.8\.4\.9
-------------

**リリース日** ：2024/04/19

### 修正

#### コア

*3\.3\.0\.0からバックポート* 

* 構成が変更された場合でも、Vault構成が固定されキャッシュされたままになる問題を修正しました。

#### PDK

*3\.7\.0\.0からバックポート* 

* `kong.request.get_forwarded_port`が`ngx.ctx.host_portand` から誤った文字列を返す問題を修正しました。数値が正しく返されるようになりました。

#### プラグイン

*3\.6\.1\.2からバックポート* 

* [**DeGraphQL**](/hub/kong-inc/degraphql/)（`degraphql`）
  * GraphQL変数が正しく解析されず、定義された型に強制変換されない問題を修正しました。

2\.8\.4\.8
-------------

**リリース日** ：2024/03/26

### 機能

#### 構成

* OpenSSL 3\.xでは、TLSv1\.1およびそれ以下はデフォルトで無効になっています。 
* **パフォーマンス：** `nginx_http_keepalive_requests`と`upstream_keepalive_max_requests`のデフォルト値を`10000`に引き上げました。 これらの変更は、スループットの高いシステムでより適切に動作するように最適化されています。 スループットの低い設定では、これらの新しい設定は、すべてのアップストリームの使用を開始するため、以前よりも多くのリクエストを要するという、 ロードバランシングに目に見えて影響を与える可能性があります。

### 修正

#### 構成

* 外部プラグイン（Go、Javascript、Python）が、Admin APIを介してプラグイン構成に変更を適用する際に失敗する問題を修正しました。
* `ssl_cipher_suite`が`old`に設定されている場合、gRPCのTLSのセキュリティレベルを`0`に設定します。 

#### コア

* `kong.logrotate`のファイル権限を644に更新しました。
* リクエストデバッグの出力で欠落していたルーターセクションを修正しました。 
* データベースがダウンしたときに`/metrics`エンドポイントがエラーをスローする問題を修正しました。
* DNSモジュールのUDPソケットリークを修正しました。

#### プラグイン

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)\(`ldap-auth-advanced`\)

  * 200以外の応答後に`groups_required`が予期しないコードを返す原因となっていたキャッシュ関連の問題を修正しました。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * `sync_rate`が`0`に設定され、 `redis`ストラテジが使用されている場合、Redisの接続が中断されたときにプラグインが`local`ストラテジに正しく戻らない問題を修正しました。

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \)、[Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)、 [GraphQL Rate Limiting Advanced](/hub/kong-inc/graphql-rate-limiting-advanced/)\( `graphql-rate-limiting-advanced` \)、および[Response Rate Limiting](/hub/kong-inc/response-ratelimiting/)\( `response-ratelimiting` \)

  * `rate-limiting`ライブラリを使用するプラグインを一緒に使用すると、互いに干渉し合い、カウンターデータを中央のデータストアに同期できなくなる問題を修正しました。

### 依存関係

* OpenSSLを3\.1\.4から3\.1\.5にアップグレードしました
* `lua-kong-nginx-module`を0\.2\.3にアップグレードしました
* `kong-lua-resty-kafka`を0\.18にアップグレードしました
* `lua-resty-luasocket`を1\.1\.2に更新し、[luasocket\#427](https://github.com/lunarmodules/luasocket/issues/427)を修正しました

2\.8\.4\.7
-------------

**リリース日** ：2024/02/08

### 修正

#### プラグイン

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)
  * カウンター同期タイマーが適切に作成または破棄されないタイマー関連の問題を修正しました。
  * プラグイン作成時ではなく、実行時にカウンター同期タイマーを作成するようになり、無意味なエラーログを減らすことができるようになりました。
  * プラグインは、Redis 接続が失敗した場合に`info`および`log`レベルのメッセージを返すようになりました。これらのエラーメッセージは、以前は見つかりませんでした。
  * プラグインは、Redisパイプライン内のクエリエラーをチェックするようになりました。
  * `sync_rate` を0より大きい値から0に変更すると、名前空間が予期せずクリアされる問題を修正しました。

2\.8\.4\.6
-------------

**リリース日** 2024/01/17

### 修正

#### コア

* カスタム`proxy_access_log`を尊重します。 [\#7437](https://github.com/Kong/kong/issues/7437)
* LuaJIT エラーによって発生する断続的なldoc障害を修正しました。 [\#7492](https://github.com/Kong/kong/issues/7492)

#### Enterprise

* リゾルバーのダウンタイムが発生した場合に古いDNSレコードをより長く使用できるように、`dns_stale_ttl` のデフォルトを1時間に引き上げました。
* GCPバックエンドのあるVaultでは、シークレットを取得できなかったときにエラーメッセージが非表示になるバグを修正しました。
* SSL検証がCLIモードで失敗したためにGCP vaultがシークレットを取得できなかった問題を修正しました。 GCPに基づくシークレット管理を使用するユーザーは、 `system`CAストアが`lua_ssl_trusted_certificate`構成に含まれていることも確認する必要があります。

#### プラグイン

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`） 
  * トークンの有効期限を計算する際の時間を更新しました。

### 依存関係

#### コア

* `resty-openssl`を 0\.8\.25 から 1\.0\.2 にアップグレードしました。 [\#7414](https://github.com/Kong/kong/issues/7414)
* Alpine ベースイメージを 3\.16 から 3\.19 にアップグレードしました。 [\#7732](https://github.com/Kong/kong/issues/7732)

#### Enterprise

* 複数のヘルスチェックインスタンスがクリアされていない場合にヘルスチェックモジュールが正しく動作しないバグを修正するため、`lua-resty-healthcheck`を1\.6\.4にアップグレードしました。

2\.8\.4\.5
-------------

**リリース日：** 2023/11/28

### 機能

#### コア

* 指定されたリクエスト内の一部のコンポーネントによって消費された時間を観察するためのサポートが追加されました。
* 一意のリクエストIDが追加されました。このIDは、エラーログ、アクセスログ、エラーテンプレート、ログシリアライザー、および新しい`X-Kong-Request-Id`ヘッダーに入力されます。 この構成は、[`headers`](/gateway/2.8.x/reference/configuration/#headers)および[`headers_upstream`](/gateway/2.8.x/reference/configuration/#headers_upstream)構成オプションを使用して、アップストリームとダウンストリームに対してカスタマイズできます。[\#11663](https://github.com/Kong/kong/pull/11663)

#### Enterprise

* ライセンス管理: 
  * ルート、プラグイン、ライセンス、デプロイ情報などのカウンターのサポートをライセンスレポートに追加しました。
  * ライセンスエンドポイントの出力にチェックサムを追加しました。

#### プラグイン

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`） 
  * 新しいフィールド `unauthorized_destroy_session`を追加しました。`true`に設定すると、不正なリクエストを受信したときにユーザーのセッション Cookie を削除してセッションを破棄します。

### 修正

#### コア

* Redi流量制限ツールからの混乱を招くデバッグログをしました。
* ハングのリスクを防ぐために、 `syncQuery()`の非同期タイマーを削除しました。
* 設定されたタイムアウトをより予測可能な方法で実行するように DNSクライアントを更新しました。
* pluginserver protobufインクルードがパッケージ内の正しいパスに配置されていることを確認しました。
* コンシューマーグループタグの不足していたサポートを追加しました。
* `proxy_access_log`が`off`の場合に Kong Gateway が起動に失敗する問題を修正しました。
* ハングのリスクを防ぐために、 `syncQuery()`の非同期タイマーを削除しました。
* `self`を渡さずに`store_connection`を呼び出す問題を修正しました。
* Kong Gateway は、ログのシリアル化にルート、サービス、およびコンシューマーオブジェクトのディープコピーを使用するようになりました。
* デバッグリクエストヘッダー`X-Kong-Request-Debug-Output`のサポートを追加しました。 これにより、特定のリクエストで特定のコンポーネントが消費した時間を確認できます。 有効にするには、[`request_debug`](/gateway/2.8.x/reference/configuration/#request_debug)構成パラメータを使用します。 このヘッダーは、Kong Gatewayのレイテンシの原因を診断するのに役立ちます。詳細については、[リクエストのデバッグ](/gateway/latest/production/debug-request/)ガイドを参照してください。 [\#11627](https://github.com/Kong/kong/pull/11627)
* クラスタストラテジを使用しているときにキーリングマテリアルのブロードキャストに失敗する問題を修正しました。
* PostgreSQL データベースを照会するときに異常なソケット接続が再利用される問題に対処しました。
* インスタンスがリセットされたときに無効化を引き起こすプラグインサーバーの問題を修正しました。

#### Enterprise

* ローカル変数`pkey`がパッケージ`pkey`をシャドウイングする問題を修正しました。この問題により、 `pkey.new`を呼び出すときに`attempt to call field 'new' (a nil value)`のエラーメッセージが表示されていました。

#### プラグイン

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/) \(`mtls-auth`\) 
  * 失効チェック中のキャッシュネットワーク障害を防ぐために問題を修正しました。

* [AWS\-Lambda](/hub/kong-inc/aws-lambda/) \( `aws-lambda` \) 
  * AWSメタデータの検出による起動遅延を解消するために、初回使用時にAWSライブラリを徐々に初期化します。

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`） 
  * これで`401`があってもセッションを維持できるようになりました。
  * ログアウト時のトークン失効の問題を修正しました。検出された失効エンドポイントを使用しているときに、コードがアクセストークンの代わりにリフレッシュトークンを取り消していました。

* コレクター（`collector`） 
  * 非推奨の Collector プラグインがまだ使用されていたため、バージョン 2\.8\.4\.1 以上にアップグレードした後、Kong Gateway を起動できない問題を修正しました。

* [Request Validator](/hub/kong-inc/request-validator/) \(`request-validator`\)
  * `allowed_content_types`構成に`-`文字を含めることができない問題を修正しました。

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \) 
  * 流量制限に関する Redis からの混乱を招くログエントリを無視しました。

* [Prometheus](/hub/kong-inc/prometheus/) \(`prometheus`\) 
  * スクレイピング中のアップストリームのヘルス反復待ち時間の急増を削減しました。

#### Admin API

* 同じRBACユーザーで、`user_token`を同じ値で更新しようとした場合に固有の違反エラーが報告される問題を修正しました。
* `user_token`の自己更新でユニーク違反が報告されなくなりました。

### 依存関係

#### コア

* lua\-kong\-nginx\-module を 0\.2\.0 から 0\.2\.2 にアップグレードしました。
* lua\-resty\-aws を 1\.3\.2 から 1\.3\.5 にアップグレードしました。
* nginx\-1\.19\.9\_06\-set\-ssl\-option\-ignore\-unexpected\-eof をパッチしました

#### Enterprise

* jqを1\.7にアップグレードしました。
* OpenSSL を 3\.1\.4 にアップグレードしました。
* クエリ中にタイムアウトが発生すると、Postgresソケットがアクティブに閉じるようになりました。[\#11480](https://github.com/Kong/kong/pull/11480)
* Dynatrace テストケースを追加しました。
* `mockbin.com`の使用は非推奨です。
* 2\.8パッケージに`.proto`ファイルを含めます。
* COPYRIGHTファイルを2\.8に更新します。

#### Kong Manager Enterprise

* GW v2\.8\.4\.5 用にkong\_adminを v0\.14\.26 にアップグレードしました。
* 既知の CVE 脆弱性を修正するために moment.js を v2\.29\.4 にアップグレードしました。

2\.8\.4\.4
-------------

**リリース日** 2023/10/12

### 修正

#### コア

* HTTP/2ストリームリセット攻撃を早期に検出するためのNginxパッチを適用しました。
  この変更は、特定された脆弱性[CVE\-2023\-44487](https://nvd.nist.gov/vuln/detail/CVE-2023-44487)に直接対処するものです。

  この脆弱性とそれに対するKongの対処に関する詳細については、当社の[ブログ記事](https://konghq.com/blog/product-releases/novel-http2-rapid-reset-ddos-vulnerability-update)をご覧ください。

2\.8\.4\.3
-------------

**リリース日** 2023/09/18

### 下位互換性のない変更と非推奨

* **Ubuntu 18\.04のサポートが削除されました** : Ubuntu 18\.04 \("Bionic"\) での Kong Gateway の実行のサポートは廃止されました。
  [Ubuntu 18\.04 の標準サポートは 2023 年 6 月をもって終了しました](https://wiki.ubuntu.com/Releases)。
  Kong Gateway 2\.8\.4\.3 以降、Kongは新しいUbuntu 18\.04イメージやパッケージをビルドしておらず、
  KongはUbuntu 18\.04へのパッケージインストールをテストしません。

* Amazon Linux 2022アーティファクトは、AWS独自の名前変更に基づいて、Amazon Linux 2023に名前が変更されます。

### 機能

#### プラグイン

* [AWS\-Lambda](/hub/kong-inc/aws-lambda/) \( `aws-lambda` \) 
  * AWS Lambda プラグインは、基盤となる AWS ライブラリとして`lua-resty-aws`を使用してリファクタリングされました。このリファクタリングにより、AWS Lambda プラグインのコードベースが簡素化され、複数の IAM 認証シナリオのサポートが追加されます。

### 修正

#### コア

* `dbless-reconfigure`の匿名レポートタイプが設定`anonymous_reports=false`の匿名レポートを遵守しない問題を修正しました。
* {{site.base_gateway}} 2\.8\.4\.2 で、デフォルト以外のワークスペースで Admin API を使用して開発者を作成できない問題を修正しました。
* Redis が流量制限ストラテジの接続失敗をキャッチする問題を修正しました。

#### プラグイン

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)
  * プラグインが予期せず流量制限をトリガする問題を修正しました。
  * 複数のRate Limiting Advancedプラグインが同じ名前空間を共有しているときに、{{site.base_gateway}}がエラーログエントリのログを生成する問題を修正しました。

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`） 
  * ベアラートークンをデコードするときにプラグインが`invalid introspection results`を含むログを返す問題を修正しました。

* [Response Transformer Advanced](/hub/kong-inc/response-transformer-advanced/)（ `response-transformer-advanced` ） 
  * `if_status`が一致しない場合に応答本文が読み込まれる問題を修正しました。

#### PDK

* カスタマイズされたヘッダーが失われる原因となっていた終了フックのバグを修正しました。

### パフォーマンス

#### 構成

* `upstream_keepalive_pool_size`のデフォルト値を 512 に、 `upstream_keepalive_max_requests`デフォルト値を 1000 に引き上げました。

### 依存関係

* `lua-protobuf`を 0\.3\.3 から 0\.4\.2 にアップグレードしました
* `lua-resty-aws`を 1\.0\.0 から 1\.3\.1 にアップグレードしました
* `lua-resty-gcp`0\}を0\.0\.5から0\.0\.13にアップグレードしました

2\.8\.4\.2
-------------

**リリース日** 2023/07/07

### 修正

* *バッファされたプロキシ* が使用されている場合に、`error_page` ディレクティブのような内部リダイレクトがリクエストを扱うワーカープロセスを妨害する可能性があるバグを修正しました。

#### Kong Manager

* Zipkin プラグインが Kong Manager UI 経由で`static_tags`を追加できない問題を修正しました。
* 一部のアイコンが正しくレンダリングされない問題を修正しました。

#### プラグイン

* テーブルではないJSONを含むリクエストが失敗するというOauth 2\.0 Introspectionプラグインの問題を修正しました。
* Go プラグインサーバーの起動が遅く、デッドロックが発生する問題を修正しました。

### 依存関係

* `OpenSSL`を1\.1\.1tから3\.1\.1に更新しました 
* Dev Portal の`lodash`を 4\.17\.11 から 4\.17\.21 にアップグレードしました
* Kong Manager の`lodash`を 4\.17\.15 から 4\.17\.21 にアップグレードしました

2\.8\.4\.1
-------------

**リリース日** 2023/05/25

### 下位互換性のない変更

#### プラグイン

* [Request Validator](/hub/kong-inc/request-validator/) \(`request-validator`\)
  * プラグインでパラメーター付きの `content-type` を含むリクエストが、パラメーターなしの`content-type` と一致するようになりました。

### 機能

* Redisクラスタ：Redisクラスタ6以降のバージョンにユーザー名とパスワードの認証が追加されました。

### 修正

* `user_token`フィールドの更新後にRBACトークンが再ハッシュされない問題を修正しました。
* Dynatrace の実装を修正しました。ビルドシステムの問題により、2\.8\.4\.1 より前のKong Gateway 2\.8\.4パッケージには、Dynatraceが必要とするデバッグシンボルが含まれていませんでした。

#### プラグイン

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)

  * フォワードプロキシを介してアップストリームから HTTP `408`を受信するときに発生する問題を修正しました。 Nginx はこのコードでプロセスを終了し、その結果、Nginx はコンテンツなしでリクエストを終了しました。

* [Request Validator](/hub/kong-inc/request-validator/) \(`request-validator`\)

  * プラグインでパラメーター付きの `content-type` を含むリクエストが、パラメーターなしの`content-type` と一致するようになりました。

### 依存関係

* `pgmoon`を 2\.2\.0\.1 から 2\.3\.2\.0 にアップグレードしました。

2\.8\.4\.0
-------------

**リリース日** ：2023/03/28

### 機能

#### プラグイン

* [AWS Lambda](/hub/kong-inc/aws-lambda/) \(`aws-lambda`\)
  * 構成パラメータ、`aws_imds_protocol_version`を追加して、IMDSプロトコルのバージョンを選択できるようになりました。 このオプションのデフォルトは`v1`ですが、`v2`に設定するとIMDSv2を有効にできます。 [\#9962](https://github.com/Kong/kong/pull/9962)

### 修正

#### Enterprise

* OpenTracing モジュールが Amazon Linux 2 パッケージに含まれていなかった問題を修正しました。
* ハイブリッドモード：データプレーン（DP）で暗号化を有効にすると、再起動後にデータプレーンが停止しなくなる問題を修正しました。
* 2\.8\.1\.x とそれ以降のバージョンで誤って`kong.service`と名付けられていた systemd ユニットファイルを修正しました。 以前のバージョンと合わせるため、`kong-enterprise-edition.service`に名前を戻しました。

##### Kong Manager

* Dev Portalを有効にする際に発生した文字数制限エラー`[postgres] ERROR: value too long for type character(32)`を修正します。 以前は、自動生成された UUID の長さよりも短い文字数制限でした。
* Kong ManagerがOIDC認証に使用する`/auth`エンドポイントが、HTTP POSTメソッドを正しくサポートするようになりました。
* OIDCを通じて新規登録されたDev Portalアカウントを持つユーザーが、Kong Gatewayコンテナを再起動するまでDev Portal にログインできない問題を復旧しました。

#### コア

* 2\.8\.2\.x 以降のバージョンで壊れていた Ubuntu ARM64 イメージを修正しました。
* ルーター: ワーカーが再生成されたときにルーターが古いデータを使用する問題を修正しました。 [\#9396](https://github.com/Kong/kong/pull/9396) [\#9485](https://github.com/Kong/kong/pull/9485)
* バッチキューモジュールを更新し、コンシューマがエントリーの処理に失敗しても、 キューが際限なく増大しないようにしました。現在、古いバッチは削除され、エラーが記録されるようになりました。[\#10247](https://github.com/Kong/kong/pull/10247)

#### プラグイン

* 欠落していた`protocols`フィールドを次のプラグインスキーマに追加しました。

  * Azure Functions（`azure-functions`）
  * gRPC Gateway（`grpc-gateway`）
  * gRPC Web（`grpc-web`）
  * サーバーレスPre\-function（`pre-function`）
  * Prometheus（`prometheus`）
  * Proxy Caching（`proxy-cache`）
  * Request Transformer（`request-transformer`）
  * Session（`session`）
  * Zipkin（`zipkin`）

  [\#9525](https://github.com/Kong/kong/pull/9525)
* [HTTP Log](/hub/kong-inc/http-log/)（`http-log`）

  * このプラグインのバッチキュー処理で、メトリクスが複数回公開される問題を修正しました。 この問題によりメモリリークが発生し、メモリ使用量が無制限に増加していました。 [\#10052](https://github.com/Kong/kong/pull/10052) [\#10044](https://github.com/Kong/kong/pull/10044)

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * ルートの更新後にプラグインが古いルートキャッシュを使用する問題を修正しました。

* [Key Authentication \- Encrypted](/hub/kong-inc/key-auth-enc)\( `key-auth-enc` \)

  * 複数のワークスペースに存在する API キーを使用すると 401 エラーが発生する問題を修正しました。 これは、キャッシュの問題が原因で発生していました。

2\.8\.2\.4
-------------

**リリース日** ：2023/01/23

### 修正

* Kong GatewayがBoringSSL PCREライブラリを静的にリンクするようになりました。 この修正により、2\.8\.2\.3で発生した、BoringSSLライブラリが動的にリンクされ、 ライブラリのバージョンによってはリクエストのルーティング時に正規表現のコンパイルに失敗する問題が復旧しました。

2\.8\.2\.3
-------------

**リリース日** ：2023/01/06

### 修正

#### Enterprise

**Kong Manager：** 

* RBACのロールの優先順位に関する問題を修正しました。 拒否（ネガティブ）ルールを含むRBACルールが、許可（非ネガティブ）ロールよりも正しく優先されるようになりました。
* 概要ページのワークスペースフィルタリングのページ分割を修正しました。

#### コア

* ルートが50,000を超える環境でルートを更新しようとすると `500`エラー応答が発生するルーターの問題を修正しました。
* ハイブリッドモードで汎用メッセージングプロトコル接続が切断されるたびに発生していたタイマーリークを修正しました。
* PostgreSQL で SSL が構成され、Kong Gateway でストリームプロキシを使用して`stream_listen`が構成されている場合に発生する`tlshandshake`メソッドエラーを修正しました。

#### プラグイン

* [HTTP Log](/hub/kong-inc/http-log/)（`http-log`）

  * 空のヘッダーによって発生する`could not update kong admin`内部エラーを修正しました。 このエラーは、Kong Ingress Controller でこのプラグインを使用したときに発生しました。

* [JWT](/hub/kong-inc/jwt/)（`jwt`）

  * JWT プラグインが未検証のトークンをアップストリームに転送する可能性がある問題を修正しました。

* [JWT署名者](/hub/kong-inc/jwt-signer/)\( `jwt-signer` \)

  * エラー`attempt to call local 'err' (a string value)`を修正しました。

* [Mocking](/hub/kong-inc/mocking/) \(`mocking`\)

  * UUIDパターンのマッチングに関する問題を修正しました。

* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`）

  * プラグインのパフォーマンスへの影響を軽減するオプションを提供しました。 高カーディナリティメトリック`on`または`off`を切り替えるための新しい`kong.conf`オプションが追加されました： [`prometheus_plugin_status_code_metrics`](/gateway/2.8.x/reference/configuration/#prometheus_plugin_status_code_metrics) 、 [`prometheus_plugin_latency_metrics`](/gateway/2.8.x/reference/configuration/#prometheus_plugin_latency_metrics) 、 [`prometheus_plugin_bandwidth_metrics`](/gateway/2.8.x/reference/configuration/#prometheus_plugin_bandwidth_metrics) 、 [`prometheus_plugin_upstream_health_metrics`](/gateway/2.8.x/reference/configuration/#prometheus_plugin_upstream_health_metrics) 。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * `kong_locks`ディクショナリのメンテナンスサイクルロックリークを修正しました。 Kong Gateway では、名前空間が更新されると、メンテナンスサイクルスケジュールから古い名前空間がクリアされるようになりました。

* [Request Transformer](/hub/kong-inc/request-transformer/) \(`request-transformer`\)

  * 空の配列が空のオブジェクトに変換される問題を修正しました。 空の配列が保持されるようになりました。

### 既知の制限

* 必要なPCREライブラリは動的にリンクされています。以前のバージョンではライブラリが静的にリンクされていました。システムのPCREバージョンによっては、リクエストをルーティングするときに正規表現のコンパイルが失敗することがあります。2\.8\.2\.4 以降では、Kong Gateway は PCRE ライブラリを静的にリンクするようになります。

2\.8\.2\.2
-------------

**リリース日** ：2022/12/01

### 修正

#### コア

タイマーの問題の修正：

* Datadog と StatsD プラグインにバッチキューを追加し、タイマーの使用量を減らし、`lua_max_running_timers are not enough` タイマーのエラーを修正しました。

  リクエストが処理されるたびに、ログフェーズの間に新しい実行中のタイマーが即座に作成されました。このため、トラフィックが集中すると
  タイマーが不足し、内部タイマーが不規則に停止して自動回復できなくなるという、
  予測不可能な事態を引き起こしていました。
  これが`lua_max_running_timers are not enough`タイマーエラーを引き起こし、データプレーンをクラッシュさせていました。

  [\#9521](https://github.com/Kong/kong/pull/9521)
* ハイブリッドモードで汎用メッセージングプロトコルの
  接続が切れるたびに発生していたタイマーリークを修正しました

2\.8\.2\.1
-------------

**リリース日** 2022/11/21

### 修正

#### Enterprise

* **Kong Manager：** 

  * RBAC ロールを編集するために管理者が特定の`rbac/role`権限を必要とする問題を修正しました。 管理者は、 `/admins`権限を使用して RBAC ロールを編集できるようになりました。
  * アップストリーム更新フォームでクライアント証明書 ID が正しく表示されない問題を修正しました。
  * ユーザーが複数のドキュメントをアップロードできるサービスドキュメント UI の問題を修正しました。各サービス1つのドキュメントのみを サポートするため、ドキュメントは正しく表示されませんでした。現在は、新しいドキュメントをアップロードすると、以前のドキュメントが上書きされるようになりました。
  * グローバルワークスペースダッシュボードの **新しいワークスペース** ボタンが、最初のページの読み込み時にクリックできない問題を修正しました。
  * ロールページに削除済みロールが一覧表示されるRBACの問題を修正しました。
  * Kong ManagerからNew Relicを削除しました。以前は、`VUE_APP_NEW_RELIC_LICENSE_KEY`と `VUE_APP_SEGMENT_WRITE_KEY`が無効な値でKong Managerに公開されていました。
  * 特定のエンドポイント（個別のサービスやルートなど）に適用された権限がKong Manager UIに反映されないRBACの問題を修正しました。
  * グループとロールとのマッピングで、スペースを含むグループ名をサポートしない問題を修正しました。
  * 個々のワークスペースダッシュボードで\[ **すべて表示** \] を右クリックして \[リンクを新しいタブで開く\] または \[リンクをコピー\] を選択すると、プラグインがデフォルトのワークスペースにリダイレクトされ、`HTTP 404` エラーを発生させる問題を修正しました。

* **Dev Portal** ：メディアタイプがベンダー固有の場合、Dev Portalのレスポンス例がレンダリングされない問題を修正しました。

#### コア

* 重みが`0`に設定されているターゲットは、ヘルスチェックに含まれなくなりました。`upstreams/<upstream id="sl-md0000000">/health`エンドポイントからステータスをチェックすると、ステータス結果は`HEALTHCHECK_OFF`になります。
  従来、`upstreams/<upstream id="sl-md0000000">/health`エンドポイントでは、`weight=0`のターゲットは`HEALTHY`と誤って報告され、ヘルスチェックでは`UNDEFINED`と報告されていました。

* ログにアクセスする権限がなかったデフォルトの`logrotate`構成を修正しました。

#### プラグイン

* [Kafka Upstream](/hub/kong-inc/kafka-upstream/) \(`kafka-upstream`\)

  * `producer_async=false`の構成で Kafka Upstream プラグインを使用するときに発生する`Bad Gateway`エラーを修正しました。

* [Response Transformer](/hub/kong-inc/response-transformer/) \(`response-transformer`\)

  * プラグインが文字列応答を処理できない問題を修正しました。

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * プラグインが原因で Kong Gateway データプレーンでリクエストが何も表示されずに失敗する問題を修正しました。

* [Request Transformer](/hub/kong-inc/request-transformer/) \(`request-transformer`\)

  * 空の配列が空のオブジェクトに変換される問題を修正しました。 空の配列が保持されるようになりました。

* [Azure Functions](/hub/kong-inc/azure-functions/) \(`azure-functions`\)

  * 次の状況でこのプラグインで呼び出しを行うと失敗する問題を修正しました。 
    * このプラグインがサービスが存在しないルートに関連付けられている状況。
    * ルートの関連サービスに`path`の値がありました

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)\(`ldap-auth-advanced`\)

  * `group_member_attribute`によって参照される操作属性が検索クエリの結果に返されない問題を修正しました。

2\.8\.2\.0
-------------

**リリース日** 2022/10/12

### 修正

#### Enterprise

* **Kong Manager** ：

  * ロールのないワークスペースがロール数で正しくソートされない問題を修正しました。
  * Kong Manager UIにおけるクロスサイトスクリプティング（XSS）に関するセキュリティ脆弱性を修正しました。
  * `admin_gui_auth`を設定せずに管理者を登録すると`500`エラーが発生する問題を修正しました。
  * 権限のないIDPユーザーがKong Managerにログインできる問題を修正しました。 これらのユーザーは、Kong Managerのリソースにアクセスできませんでしたが、ログイン画面の先に進むことはできました。

* OpenSSL の脆弱性 [CVE\-2022\-2097](https://nvd.nist.gov/vuln/detail/CVE-2022-2097) および [CVE\-2022\-2068](https://nvd.nist.gov/vuln/detail/CVE-2022-2068) を修正しました。

* ハイブリッドモード：コントロールプレーンがデータプレーンに正しい数のコンシューマーエントリを送信していなかったコンシューマーグループの問題を修正しました。

* ハイブリッドモード：コントロールプレーン（CP）を再起動した後に `PATCH`リクエストを送信してルートを更新すると、500エラー応答が発生する問題を修正しました。

#### プラグイン

* [AWS Lambda](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * プラグインがECS環境の環境変数を読み取れず、パーミッションエラーが発生する問題を修正しました。

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)

  * `https_proxy`構成パラメータが設定されていない場合、DNS エラーを回避するためにデフォルトで`http_proxy`に設定されます。

* [GraphQL Proxy Cache Advanced](/hub/kong-inc/graphql-proxy-cache-advanced/)\(`graphql-proxy-cache-advanced`\)と[Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)

  * プラグインが安定して動作することを妨げていたエラー`function cannot be called in access phase (only in: log)`を修正しました。

* [GraphQL Rate Limiting Advanced](/hub/kong-inc/graphql-rate-limiting-advanced/) \(`graphql-rate-limiting-advanced`\)

  * ハイブリッドモードまたはDBレスモードで `cluster` ストラテジを使用すると、プラグインがクラッシュする代わりに`500`エラーを返すようになりました。

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * グループ属性で文字`.`と`:`が使用できるようになりました。

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 管理者を複数のワークスペースに追加できず、権限が更新されないというOIDCロールマッピングの問題を修正しました。

* [Request Transformer Advanced](/hub/kong-inc/request-transformer-advanced/)（ `request-transformer-advanced` ）

  * 空の配列が空のオブジェクトに変換される問題を修正しました。 空の配列が保持されるようになりました。

* [Route Transformer Advanced](/hub/kong-inc/route-transformer-advanced/)\(`route-transformer-advanced`\)

  * `%20`または空白を含む URI が`400 Bad Request`を返す問題を修正しました。

2\.8\.1\.4
-------------

**リリース日** 2022/08/23

* 脆弱性 [CVE\-2022\-37434](https://nvd.nist.gov/vuln/detail/CVE-2022-37434) および [CVE\-2022\-24975](https://nvd.nist.gov/vuln/detail/CVE-2022-24975) を修正しました。

* シークレット管理をフリーモードで使用する場合、[利用できるのは環境変数バックエンドのみです](/gateway/2.8.x/plan-and-deploy/security/secrets-management/backends/env/)。AWS、GCP、HashiCorpのボールトバックエンドにはEnterpriseライセンスが必要です。

* Kong Manager でエンティティの詳細ページが空になり、既存のエンティティがリストされない問題を修正しました。
  次のエンティティが影響を受けました。

  * サービスページのルートリスト
  * アップストリーム
  * 証明書
  * SNI
  * RBACの役割

* 既存のホストとポートでアップストリームを作成するときにブラウザが動かなくなる問題を修正しました。

#### プラグイン

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * データプレーン（DP）ノードが、毎回IdPから新しいJWKを取得しようとしていたハイブリッドモードでのキャッシュの問題を修正しました。 データプレーン（DP）は、まずキャッシュされた JWK を検索します。

* [Proxy Caching Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)

  * 既存の構成でユーザーがクラスタアドレスを削除できない問題を修正しました。

### 依存関係

* AWSボールトが有効な場合のメモリ使用量を削減するには、 `lua-resty-aws`バージョンを 0\.5\.4 に上げます。[＃23](https://github.com/Kong/lua-resty-aws/pull/23)
* GCP Vault が有効な場合のメモリ使用量を削減するために、 `lua-resty-gcp`バージョンを 0\.0\.5 にアップグレードします。[＃7](https://github.com/Kong/lua-resty-gcp/pull/7)

2\.8\.1\.3
-------------

**リリース日** 2022/08/05

### 機能

#### Enterprise

* シークレットマネージャーにGCP統合サポートが追加され、GCPがVaultバックエンドとして利用できるようになりました。

#### プラグイン

* [AWS Lambda](/hub/kong-inc/aws-lambda/) \(`aws-lambda`\)
  * `aws_assume_role_arn`および `aws_role_session_name`構成パラメータを通じて、アカウント間での呼び出しに対応できるようになりました。 [\#8900](https://github.com/Kong/kong/pull/8900)

### 修正

#### Enterprise

* コントロールプレーン（CP）でのログファイルディスク使用率が高すぎる問題を修正しました。
* キーリングの暗号化において、ソフトリロード後にキーリングが復号化されない問題を修正しました。
* ルーターが現在のワークスペース内だけでなく、他のワークスペースとの静的ルートの衝突も検出するようになりました。
* ハイブリッドモードの導入でカスタムプラグインを使用する場合、コントロールプレーンが互換性の問題を検出し、それを使用できないデータプレーンにプラグイン構成を送信しなくなるようになりました。コントロールプレーンは、互換性のあるコントロールプレーン（CP）にカスタムプラグイン構成を送信し続けます。
* Kong PDK機能`kong.response.get_source()`を最適化しました。

#### Kong Manager

* 管理者の作成に関する問題を修正しました。 以前は、ロールなしで管理者が作成されると、管理者はアルファベット順にリストされている最初のワークスペースにアクセスできました。
* SNIリストに関するいくつかの問題を修正しました。 以前は、SSL 証明書 ID フィールドで並べ替えた後、SNI リストは空でした。2\.8\.1\.1 では、SNI リストの SSL 証明書 ID フィールドが空欄でした。

#### プラグイン

* [Mocking](/hub/kong-inc/mocking/) \(`mocking`\)

  * プラグインが例で空の値を受け付けない問題を修正しました。

* [ACME](/hub/kong-inc/acme/)（`acme`）

  * `domains`プラグインパラメータを空のままにできるようになりました。 `domains`が空の場合、すべてのTLDが許可されます。以前は、このパラメーターは任意と表示されていましたが、空のままにしておくと、プラグインは証明書をまったく取得しませんでした。

* [Response Transformer Advanced](/hub/kong-inc/response-transformer-advanced/)（ `response-transformer-advanced` ）

  * ネストされた配列の解析に関する問題を修正しました。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * カサンドラの`cluster`ストラテジタイムスタンプの精度に関する問題を修正しました。

2\.8\.1\.2
-------------

**リリース日** 2022/07/15

### 修正

#### Enterprise

* ハイブリッドモードで、サービスが `enabled: false` に設定されており、そのサービスにプラグインが有効になっているルートがあった場合、新しいデータプレーン（DP）は空の設定を受け取るという問題を修正しました。 
* `kong.conf`で`worker_consistency`が`eventual`に設定されたときに発生するタイマーリークを修正しました。 この問題により、タイマーが使い果たされ、Kong Gatewayが使用する他のタイマーを開始できず、 `too many pending timers`エラーが発生しました。
* `lua-resty-lock`からのメモリリークを修正しました。
* グローバルプラグインがワークスペースのスコープ外で動作できるように修正しました

#### Kong Manager と Dev Portal

* Kong Manager で組織内のすべての Dev Portal 開発者が表示されない問題を修正しました。
* 開発者ロールの割り当てがKong Managerに表示されない問題を修正しました。 従来は、Dev Portalセクションの「権限」タブにロールが表示される場合、開発者リストは新しい開発者が追加されても更新されませんでした。 Kong Managerは、Dev Portalの割り当て先を取得する際、間違ったURLを構築していました。
* Kong Manager での空の文字列の処理を修正しました。以前は、Kong Manager は空の文字列を null 値ではなく`""`として処理していました。
* コンテンツがオブジェクトの詳細ページに収まらない問題を修正し、Kong Manager のデザインを改善しました。
* Safari で Kong Manager のリンクやボタンをクリックできない場合がある問題を修正しました。
* オブジェクトリストからCopy IDボタンをクリックした後、ユーザーがオブジェクトの詳細ページに移動する問題を修正しました。
* Vitals が無効になっている場合にリクエスト数とエラー率が正しく表示されない問題を修正しました。

#### プラグイン

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \) と[Response Rate Limiting](/hub/kong-inc/response-ratelimiting/)\( `response-ratelimiting` \)

  * `cluster`ポリシーが 2 つ以上のメトリクス \(たとえば、 `second`と`day`\)で使用された場合に発生する PostgreSQLのデッドロックの問題を修正しました。

* [HTTP Log](/hub/kong-inc/http-log/)（`http-log`）

  * ログ出力は、プラグインが実行されているワークスペースに制限されるようになりました。以前、 本プラグインはワークスペース外からのリクエストをログに記録することができました。

* [Mocking](/hub/kong-inc/mocking/) \(`mocking`\)

  * `204`レスポンスが正しく処理されず、次のエラーが表示される問題を修正しました： `"No examples exist in API specification for this resource"`
  * `204`レスポンス仕様が空のコンテンツ要素に対応するようになりました。

### 非推奨

* **Amazon Linux 1** : 
  [Amazon Linux AMIが2020年12月31日をもって標準サポートを終了したため、Amazon Linux 1上でKong Gatewayを実行するサポートは非推奨となりました](https://aws.amazon.com/blogs/aws/update-on-amazon-linux-ami-end-of-life)。
  Kong Gateway 3\.0\.0\.0 以降、Kongは新しいAmazon Linux 1を構築していない
  イメージまたはパッケージがインストールされていない場合、Kong は Amazon Linux 1 でのパッケージのインストールをテストしません。

  Amazon Linux 1にKong Gatewayをインストールする必要がある場合は、
  [以前のバージョン](/gateway/2.8.x/install-and-run/amazon-linux/)のドキュメントをご覧ください。
* **Debian 8** ：Debian 8 \(「Jessie」\) は
  [サポート終了 \(End of Life\)に達したため、Debian 8 \(「Jessie」\) での Kong Gateway のサポートは非推奨となりました](https://www.debian.org/News/2020/20200709)。
  Kong Gateway 3\.0\.0\.0 以降、Kong は Debian 8 
  \(「Jessie」\) のイメージやパッケージの新規ビルドは行っておらず、 Kong は 
  Debian 8 \(「Jessie」\) へのパッケージインストールをテストしません。

  Debian 8 \(「Jessie」\) に Kong Gateway をインストールする必要がある場合は、
  [以前のバージョン](/gateway/2.8.x/install-and-run/debian/)のドキュメントをご覧ください。
* **Ubuntu 16\.04** ：[Ubuntu 16\.04 の標準サポートは 2021 年 4 月をもって非推奨となったため](https://wiki.ubuntu.com/Releases)、
  Ubuntu 16\.04 \(「Xenial」\) での Kong Gateway の実行サポートは廃止されました。
  Kong Gateway 3\.0\.0\.0 以降、Kongは新しいUbuntu 16\.04
  イメージやパッケージをビルドしておらず、KongはUbuntu 16\.04へのパッケージインストールをテストしていません。

  Ubuntu 16\.04にKong Gatewayをインストールする必要がある場合は、
  [以前のバージョン](/gateway/2.8.x/install-and-run/ubuntu/)のドキュメントをご覧ください。

2\.8\.1\.1
-------------

**リリース日** ：2022/05/27

### 機能

#### Enterprise

* 以下の構成パラメータを使用して、開発者ポータルでアプリケーションステータスとアプリケーションリクエストのメールを有効にできます： 
  * [`portal_application_status_email`](/gateway/latest/reference/configuration/#portal_application_status_email)：アプリケーション要求ステータス更新メールを開発者に送信できるようにします。
  * [`portal_application_request_email`](/gateway/latest/reference/configuration/#portal_application_request_email) : `smtp_admin_emails`で指定されたユーザーにサービスアクセス要求メールを送信できるようにします。
  * [`portal_smtp_admin_emails`](/gateway/latest/reference/configuration/#portal_smtp_admin_emails): `smtp_admin_emails`で設定された値を上書きして、ポータル管理者のメールを送信するメールアドレスを指定してください。

* ポータルメールテンプレートで`email.developer_meta`フィールドを使用する機能が追加されました。たとえば、`{% raw %}{{email.developer_meta.preferred_name}}{% endraw %}`。

#### プラグイン

* [AWS Lambda](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * プロキシ統合モードで作業している場合、`statusCode`フィールドが文字列データ型を 受け付けるようになりました。

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * 証明書失効リスト（CRL）とOCSPサーバーのサポートが次のパラメータで導入されました：`http_proxy_host`、`http_proxy_port`、`https_proxy_host`、`https_proxy_port`。

* [Kafka Upstream](/hub/kong-inc/kafka-upstream/)\(`kafka-upstream`\)と[Kafka Log](/hub/kong-inc/kafka-log/)\(`kafka-log`\)

  * `SCRAM-SHA-512`認証メカニズムのサポートを追加しました。

### 修正

#### Enterprise

* 多数のエンティティを持つ組織におけるKong Admin APIとKong Managerのパフォーマンスが
  向上しました。

* [キーリングモジュール](/gateway/latest/plan-and-deploy/security/db-encryption/)の初期化中にエラーが発生すると、コントロールプレーン（CP）がクラッシュするキーリング暗号化に関する問題を修正しました。

* Kong Manager が組織内ですべての RBAC ユーザーとコンシューマーを表示しない問題を修正しました。

* リストの行の一部の領域がクリックできない問題を修正しました。

#### プラグイン

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * Rate Limiting Advancedプラグインが使用されていなかった時に、 レート制限時に表示される流量制限のエラーを修正しました。
  * キーの有効期限の追跡が正しくないために、流量制限カウンタが応答ヘッダを更新しないエラーを修正しました。 Redis キーの有効期限が`lua_shared_dict kong_rate_limiting_counters`で適切に追跡されるようになりました。

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)

  * HTTPSリクエストの`invalid header value`エラーを修正しました。このプラグインは、複数値の応答ヘッダーを受け入れるようになりました。
  * `=` 文字を含むベーシック認証ヘッダーが転送されないというエラーを復旧しました。
  * スキームにプロキシが設定されていない場合に発生するリクエストエラーを修正しました。その `https`プロキシは、指定されていない場合は`http`プロキシにフォールバックするようになり、 `http` プロキシは`https`にフォールバックします。

* [GraphQL Rate Limiting Advanced](/hub/kong-inc/graphql-rate-limiting-advanced/) \(`graphql-rate-limiting-advanced`\)

  * GraphQL ASTを非 null 型またはリスト型で構築する場合の`deserialize_parse_tree`ロジックを修正しました 

2\.8\.1\.0
-------------

**リリース日** ：2022/04/07

### 修正

#### Enterprise

* RBACで、 `endpoint=/kong workspace=*`のすべてのワークスペースから`/kong`エンドポイントにアクセスできない問題を修正しました
* 最上位レベルの`endpoint=*`権限を持たない管理者が、 `endpoint=/rbac`権限を持っていても RBAC ルールを追加できないという RBAC の問題を修正しました。これらの管理者は、現在のワークスペースに対してのみ RBAC ルールを追加できるようになりました。
* Kong Manager 
  * 指定された値にカンマがある場合でもサーバーレス関数を保存可能
  * カスタムプラグインでプラグインの設定を表示しているときに \[編集\] ボタンを表示
  * Dev Portal の権限を編集すると表示される404 エラーを修正
  * デフォルト以外のワークスペースにのみアクセスできる管理者がワークスペースを表示できない問題を修正
  * 管理者がデフォルト以外のワークスペースにのみアクセスできる場合にワークスペース名を表示
  * Cassandra使用時のテーブルフィルタリングとソートのサポートを追加
  * RBAC編集ページのRBACトークンで\#文字をサポート
  * アップストリームターゲットでアクションを実行し発生する404エラーを修正

* 開発者ポータル 
  * 現在のセッションに関する情報がnginx ワーカースレッドにバインドするように修正。これにより、ワーカーが複数のリクエストを同時に処理しているときにデータ漏洩を防ぐことができます

* ノードの再起動時にキーが予期せずローテーションされる動作を修正
* RBAC トークン検証の実行時にキャッシュを追加
* ログメッセージ「プラグインイテレータが再構築中に変更されました」が誤って`error`として記録されました。このリリースでは、 `info`ログレベルに変換されます。
* Rate Limiting Advancedプラグインで流量制限カウンタがいっぱいになったときに発生する500エラーを修正しました。
* 条件付き再構築を追加することで、ルーター、プラグイン、イテレーター、バランサーのパフォーマンスを向上させました

#### プラグイン

* [HTTP Log](/hub/kong-inc/http-log/) \(`http-log`\)
  * ログを送信するときに、提供されたクエリ文字列パラメータを含めます。`http_endpoint`

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)
  * `host`ヘッダーを上書きする場合は小文字を使用してください

* [StatsD Advanced](/hub/kong-inc/statsd-advanced/) \(`statsd-advanced`\)
  * `workspace_identifier`を次のように設定するためのサポートが追加されました。`workspace_name`

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)
  * プラグインが有効になっていない場合は、名前空間の作成をスキップします。これにより、「\[rate\-limiting\-advanced\] 共有辞書が指定されていません」というエラーがログに記録されなくなります。

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)\(`ldap-auth-advanced`\)
  * `:`文字を含むパスワードをサポートする

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`） 
  * 有効なアップストリームヘッダーを提供します。例：`X-Consumer-Id`、`X-Consumer-Username`

* [JWT署名者](/hub/kong-inc/jwt-signer/)\( `jwt-signer` \)
  * HMAC アルゴリズムで署名された JWT を有効にするには、 `enable_hs_signatures`任意を実装します。

### 依存関係

* `openssl`を1\.1\.1kから1\.1\.1n までアップグレードし、CVE\-2022\-0778 [\#8635](https://github.com/Kong/kong/pull/8635)を解決しました
* `openresty`を 1\.19\.3\.2 から 1\.19\.9\.1 にアップグレードしました[\#7727](https://github.com/Kong/kong/pull/7727)

2\.8\.0\.0
-------------

**リリース日** ：2022/03/02

### 機能

#### Enterprise

* Kong Manager のテーブルの改善：\(PostgreSQL ベースのインスタンスのみ\)

  * **表示** アイコンを使用する代わりに、エントリにアクセスするにはテーブル行をクリックします。
  * テーブルの上にある **Filters** ドロップダウンでテーブルを検索し、 フィルタリングします。
  * 列のタイトルをクリックして、任意のテーブルを並べ替えます。
  * テーブルにページネーションが追加されました。

* Kong Manager with OIDC \(英語\)：

  * OpenID Connectで管理者を自動的に作成する際に、RBACトークンを有効または無効にする設定オプション[`admin_auto_create_rbac_token_disabled`](/gateway/latest/configure/auth/kong-manager/oidc-mapping/)を追加しました。

* ライセンスが存在する場合、 `license_key` `api`シグナルに含まれるようになりました。
  [`anonymous_reports`](/gateway/latest/reference/configuration/#anonymous_reports) .

#### Dev Portal

* 新しい `/developers/export`エンドポイントでは、開発者の一覧とそのステータスを CSV形式でエクスポートできます。

#### コア

* **ベータ機能** ：Kong Gateway 2\.8\.0\.0では、[シークレット管理とvaultのサポートを導入しました。](/gateway/latest/kong-enterprise/secrets-management/)
  ユーザー名やパスワードのような秘密鍵を、安全なvaultに秘密として保管できるようになりました。
  Kongゲートウェイはこれらの秘密を参照することができ、環境をより安全にすることができます。

  ベータ版には、次の Vault 実装に対する`get`サポートが含まれています。
  * [AWS Secrets Manager](/gateway/latest/kong-enterprise/secrets-management/backends/aws-sm/)
  * [HashiCorp Vault](/gateway/latest/kong-enterprise/secrets-management/backends/hashicorp-vault/)
  * [環境変数](/gateway/latest/kong-enterprise/secrets-management/backends/env/)

  このサポートの一環として、一部のプラグインには *リファレンス可能な* 次のようにマークされた特定のフィールドがあります
  詳細については、Kong Gateway 2\.8の変更ログのプラグインセクションをご参照ください。

  シークレット管理をテストするには、
  [入門ガイド](/gateway/latest/kong-enterprise/secrets-management/getting-started/)、
  Kong Admin API [`/vaults-beta`エンティティ](/gateway/latest/admin-api/#vaults-beta-entity)のドキュメントをご確認ください。

  {:.important}
  > 
  > この機能はベータ版です。サポートと実装の詳細が限られている
  > 変更される可能性があります。つまり、ステージング環境でのテストのみを意図しており、 **本番環境にデプロイすべきではない** ことを意味します。
* 透過的な動的 TLS SNI 名をカスタマイズできます。

  [@Murphy\-hub](https://github.com/Murphy-hub)、ありがとうございます
  [\#8196](https://github.com/Kong/kong/pull/8196)
* ルートは、正規表現によるヘッダーの一致に対応するようになりました。

  [@vanhtuan0409](https://github.com/vanhtuan0409)、ありがとうございます
  [＃6079](https://github.com/Kong/kong/pull/6079)
* ハイブリッドモードの展開用に [`cluster_max_payload`](/gateway/latest/reference/configuration/#cluster_max_payload)
  を設定できるようになりました。この構成オプションは、コントロールプレーンからデータプレーンに渡って送信できる最大ペイロード
  サイズを設定します。`payload too big`エラー
  が発生し、データプレーンに適用されないような大きな構成がある
  環境では、この設定で制限を調整してください。

  [@andrewgkew](https://github.com/andrewgkew)、ありがとうございます
  [\#8337](https://github.com/Kong/kong/pull/8337)

#### パフォーマンス

* 大きな構成のための宣言的構成ハッシュの計算を改善しました。
  新しい方法はより速く、より少ないメモリしか使用しません。[\#8204](https://github.com/Kong/kong/pull/8204)

* ルーターには、次のような複数の改善点があります。

  * ルーターは2倍の速さで構築します
  * 失敗はキャッシュされ、より速く破棄される（ネガティブキャッシュ）
  * ヘッダーが一致するルートはキャッシュされます

  これらの変更は、DBレス環境で再構築する場合に
  特に顕著になるはずです。
  [\#8087](https://github.com/Kong/kong/pull/8087)
  [\#8010](https://github.com/Kong/kong/pull/8010)

#### Admin API

* KongノードがDBレスまたはデータプレーンモードで動作している場合、`status`エンドポイントによって現在の宣言的構成ハッシュが返されるようになりました。 [\#8214](https://github.com/Kong/kong/pull/8214) [\#8425](https://github.com/Kong/kong/pull/8425)

#### プラグイン

* [Canary](/hub/kong-inc/canary/)\(`canary`\)

  * `canary_by_header_name`を設定する機能が追加されました。このパラメータ は、リクエストに存在するとき、設定されているカナリア機能を 上書きするヘッダー名を受け入れます。 
    * 設定されたヘッダーが値`always`で存在する場合、 リクエストは常にカナリアアップストリームに進みます。
    * このヘッダーが値`never`で存在する場合、リクエストはカナリアのアップストリームに進みません。

* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`）

  * 次の 3 つの新しいメトリックが追加されました。
    * `kong_db_entities_total` \(ゲージ\)：データベース内のエンティティの総数。
    * `kong_db_entity_count_errors` \(カウンター\)：`kong_db_entities_total`の測定中に発生したエラーの数を測定します。
    * `kong_nginx_timers`\(ゲージ\): 実行中または保留中の状態の Nginx タイマーの合計数。トラック`ngx.timer.running_count()`と `ngx.timer.pending_count()` . [\#8387](https://github.com/Kong/kong/pull/8387)

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * セッションの保存と取得のための Redis ACL サポート \(Redis v6\.0\.0\+\) が追加されました。
    `session_redis_username` と`session_redis_password` の設定パラメータを使用します。

    {:.important}
    > 
    > これらのパラメータは`session_redis_auth`フィールドに取って代わりますが、これは現在 **非推奨** であり、3\.x.xで削除される予定です。
  * 配布された要求のサポートが追加されました。分散クレームを明示的に解決するようにOIDCに指示するには、`resolve_distributed_claims`設定パラメータを`true`に設定します。

    分散クレームは、クレームを含むJSONオブジェクトの`_claim_names`と`_claim_sources`メンバーで表現されます。
  * **ベータ機能：** `client_id`、`client_secret`、`session_secret`、`session_redis_username`、 `session_redis_password`設定フィールドが参照可能としてマークされるようになりました。
    これは、[秘密鍵](/gateway/latest/kong-enterprise/secrets-management/getting-started/)としてvaultに安全に格納できることを意味します。[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)には特定の書式に従わなければなりません。

* [Forward Proxy Advanced](/hub/kong-inc/forward-proxy/)\(`forward-proxy`\)

  * mTLSサポートのための`http_proxy_host`、`http_proxy_port`、`https_proxy_host`、
    `https_proxy_port`設定パラメータを追加しました。

    {:.important}
    > 
    > 現在、これらのパラメータは **非推奨となっており** 、3\.xxで削除される予定で、`proxy_port`フィールドと`proxy_host`フィールドを置き換えます。
  * `auth_password`と`auth_username`の設定フィールドは参照可能としてマークされるようになりました。これは、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/)として保管庫に安全に保存できることを意味します。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に沿う必要があります。

* [Kafka Upstream](/hub/kong-inc/kafka-upstream/)\(`kafka-upstream`\)と[Kafka Log](/hub/kong-inc/kafka-log/)\(`kafka-log`\)

  * `cluster_name`構成パラメータを使用して Kafka クラスターを識別する機能が追加されました。
    デフォルトでは、このフィールドはランダムな文字列を生成します。また、独自のカスタムクラスタ識別子を設定することもできます。

  * **ベータ機能：** `authentication.user`と`authentication.password`の設定フィールドが参照可能としてマークされるようになりました。これは、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/)としてvaultに安全に保存できることを意味します。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に従う必要があります。

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)（`ldap-auth-advanced`）

  * **ベータ機能：** `ldap_password`と`bind_dn`の設定フィールドが参照可能としてマークされるようになりました。これは、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/)としてvaultに安全に保存できることを意味します。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に従う必要があります。

* [Vault Authentication](/hub/kong-inc/vault-auth/) \(`vault-auth`\)

  * **ベータ機能：** `vaults.vault_token`フォームフィールドは参照可能としてマークされるようになり、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/) としてvaultに安全に格納できるようになりました。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に沿う必要があります。

* [GraphQL Rate Limiting Advanced](/hub/kong-inc/graphql-rate-limiting-advanced/) \(`graphql-rate-limiting-advanced`\)

  * Redis ACL サポートが追加されました \(Redis v6\.0\.0\+ および Redis Sentinel v6\.2\.0\+\)。

  * `redis.username` と `redis.sentinel_username` の設定パラメータを追加しました。

  * **ベータ機能** ：`redis.username`、`redis.password`、`redis.sentinel_username`、`redis.sentinel_password` 設定フィールドが参照可能としてマークされるようになりました。これは、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/)としてvaultに安全に保存できることを意味します。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に従う必要があります。

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \)

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * Redis ACL サポートが追加されました \(Redis v6\.0\.0\+ および Redis Sentinel v6\.2\.0\+\)。

  * `redis.username` と `redis.sentinel_username` の設定パラメータを追加しました。

  * **ベータ機能** ：`redis.username`、`redis.password`、`redis.sentinel_username`、`redis.sentinel_password` 設定フィールドが参照可能としてマークされるようになりました。これは、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/)としてvaultに安全に保存できることを意味します。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に従う必要があります。

* [Response Rate Limiting](/hub/kong-inc/response-ratelimiting/) \(`response-ratelimiting`\)

  * Redis ACL サポートが追加されました \(Redis v6\.0\.0\+ および Redis Sentinel v6\.2\.0\+\)。

  * `redis_username` 設定パラメータを追加しました。

    オリジナルの投稿をしてくださった[@27ascii](https://github.com/27ascii)に感謝いたします。
    [\#8213](https://github.com/Kong/kong/pull/8213)

* [Response Transformer Advanced](/hub/kong-inc/response-transformer-advanced/)（ `response-transformer-advanced` ）

  * PDKからのレスポンスバッファリングを使用します。

* [Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)

  * Redis ACL サポートが追加されました \(Redis v6\.0\.0\+ および Redis Sentinel v6\.2\.0\+\)。

  * `redis.sentinel_username`と`redis.sentinel_password`の設定パラメータを追加しました。

  * **ベータ機能：** `redis.password`、`redis.sentinel_username`、`redis.sentinel_password` 設定フィールドが参照可能としてマークされ、[秘密](/gateway/latest/kong-enterprise/secrets-management/getting-started/)としてvaultに安全に格納できるようになりました。リファレンスは[特定の形式](/gateway/latest/kong-enterprise/secrets-management/reference-format/)に従う必要があります。

* [jq](/hub/kong-inc/jq/)\(`jq`\)

  * PDKからのレスポンスバッファリングを使用します。

* [ACME](/hub/kong-inc/acme/)（`acme`）

  * `rsa_key_size` 設定パラメータを追加しました。

    [lodrantl](https://github.com/lodrantl)、ありがとうございます！
    [\#8114](https://github.com/Kong/kong/pull/8114)

### 修正

#### Enterprise

* タイマーを使い果たし、Kongが使用する他のタイマーを開始できず、エラー`too many pending timers`を表示するタイマーリークを修正しました。

* `data_plane_config_cache_mode`が`off`に設定されている場合、データプレーン（DP）がコントロールプレーンからの更新を受信しない問題を修正しました。

* TLSを使用したルートまたはサービスへのアクセス時に発生する`attempt to index local 'workspace'`エラーを修正しました。

* [`cluster_telemetry_server_name`](/gateway/latest/reference/configuration/#cluster_telemetry_server_name)が明示的に設定されていない場合、自動生成・登録されない問題を修正しました。

* [`cluster_allowed_common_names`](/gateway/latest/reference/configuration/#cluster_allowed_common_names)の設定を修正しました
  ハイブリッドモードで証明書検証にPKIを使用する場合、オプションでコントロールプレーン（CP）への接続を許可するコモンネームのリストを構成できるようになりました。設定されていない場合、コントロールプレーン（CP）証明書と同じ親ドメインを持つデータプレーンのみが許可されます。

#### Kong Manager

* Azure ADを使用した場合に、Kong ManagerへのOIDC認証が失敗する問題を修正しました。

* Kong Manager の「チーム」ページのパフォーマンスに関する問題を修正しました。

* Kong Manager のチェックボックスで、
  OAuth2プラグインの`hash_secret`値のチェックボックスが *「Required（必須）」* と表示され、ユーザーが
  チェックを外せない問題を修正しました。

* プラグインから`service.id`をクリアしようとすると、
  Kong Manager がプラグイン設定を更新しない問題を修正しました。

* Kong ManagerのRoute作成で、新しいRouteの
  デフォルトが`http`になる問題を修正しました。これで、Route を作成すると`http,https`が正しいデフォルト値を選択します。

* Kong Managerの設定ページで、RouteオブジェクトとServiceオブジェクトのプロトコルオプションとして`udp`が正確にリストされるようになりました。

* Kong ManagerのOIDC認証でエラー
  `“attempt to call method 'select_by_username_ignore_case' (a nil value)”`
  が発生し、OIDCでログインできない問題を修正しました。

* OAuth2トークン作成時のレイテンシに関する問題を修正しました。Kong Manager UIでは
  カウントが必要ないため、これらのトークンはワークスペースエンティティカウンタによって
  追跡されなくなりました。

* プラグインリストテーブルが **「適用先」** カラムでソートできない問題を修正しました。

#### Dev Portal

* SMTPコンフィギュレーションが壊れているか応答がない場合、APIは文字列ではなくJavaScriptオブジェクト（`[Object object]`）であるエラーメッセージで応答していました。これは、ユーザーが壊れたSMTPで
  任意のポータルに登録したときに発生しました。エラーが発生した場合、APIは次の文字列`Error sending email`を返します。

* `/document_objects`と`/services/:id/document_objects`エンドポイントは、サービスごとに複数のドキュメントを受け付けなくなりました。各サービスは1つのドキュメント
  しか持てないため、問題となっていました。代わりに、これらのエンドポイントの1つにドキュメントを投稿すると、
  前のドキュメントが上書きされるようになりました。

#### コア

* [RFC\-3546](https://datatracker.ietf.org/doc/html/rfc3546#section-3.1)
  によると、ドットはホスト名の一部ではないため、
  ルータが末尾にドット\(`.`\)を持つSNI FQDNに遭遇した場合、ドットは無視されます。[\#8269](https://github.com/Kong/kong/pull/8269)

* ワイルドカードとポート \(`route.*:80`\) の両方を持つルートを、
  ワイルドカードのみのルート \(`route.*`\) 
  よりも優先しないルーターのバグを修正しました。
  [\#8233](https://github.com/Kong/kong/pull/8233)

* 内部DNSクライアントは、`search .`\#8307 のような特殊なケースで`/etc/resolv.conf`に表示されることがあるシングルドット（`.`）ドメインによって混乱することはありません。
  [\#8307](https://github.com/Kong/kong/pull/8307)

* Cassandraコネクタは移行の一貫性レベルを記録するようになりました。

  [@mpenick](https://github.com/mpenick)、ありがとうございます。
  [\#8226](https://github.com/Kong/kong/pull/8226)

#### バランサー

* アップストリームが更新されても、ターゲットはヘルスステータスを維持するようになりました。
  [\#8394](https://github.com/Kong/kong/pull/8394)

* `error`ログレベル
  を誤って使用していた1つのデバッグメッセージが、適切な`debug`ログレベルにダウングレードされました。
  [\#8410](https://github.com/Kong/kong/pull/8410)

#### クラスタリング

* コントロールプレーン（CP）との接続時にSSLに失敗した場合に表示されていた不可解なエラーメッセージを、より有用なものに差し替えました。 [\#8260](https://github.com/Kong/kong/pull/8260)

#### Admin API

* アップストリームをページ分割するときに表示される、誤った`next`フィールドを修正しました。 [\#8249](https://github.com/Kong/kong/pull/8249)

#### PDK

* フェーズチェックを実行するときにフェーズ名が正しく選択されるようになりました。 [\#8208](https://github.com/Kong/kong/pull/8208)
* go\-PDKで、`kong.request.getrawbody`が 一時ファイルにバッファリングされる十分な大きさだった場合、空文字列を返してしまうバグを修正しました。 [\#8390](https://github.com/Kong/kong/pull/8390)

#### プラグイン

* **外部プラグイン** ：

  * ヘッダーのProtobufの構造
    とNULL値の表現に誤りがあり、go\-pdkの起動時にエラーが発生していた問題を修正しました。
    [\#8267](https://github.com/Kong/kong/pull/8267)

  * `ConsumerSpec`と`AuthenticateArgs`をアンラップします。

    [@raptium](https://github.com/raptium)さん、ありがとうございます。
    [\#8280](https://github.com/Kong/kong/pull/8280)
  * ストリームサブシステムでHTTPヘッダーを読み込もうとする問題を修正しました。
    [\#8414](https://github.com/Kong/kong/pull/8414)

* [CORS](/hub/kong-inc/cors/) \(`cors`\)

  * CORSプラグインは、ヘッダー`Access-Control-Allow-Origin`が`*`に設定されている場合、`Vary: Origin`ヘッダーを送信しなくなりました。

    [@jkla\-dr](https://github.com/jkla-dr)、ありがとうございました。
    [\#8401](https://github.com/Kong/kong/pull/8401)

* [AWS Lambda](/hub/kong-inc/aws-lambda/)（`aws-lambda`）

  * HTTPプロキシを使用するように設定されている場合の不正な動作を復旧し、3\.0で削除するために`proxy_scheme`設定属性を非推奨としました。[\#8406](https://github.com/Kong/kong/pull/8406)

* [OAuth2](/hub/kong-inc/oauth2/) \(`oauth2`\)

  * このプラグインは、論理ORで設定され、他の認証プラグインと組み合わせて使用される場合、`X-Authenticated-UserId`と `X-Authenticated-Scope` ヘッダをクリアします。 [\#8422](https://github.com/Kong/kong/pull/8422)

* [Datadog](/hub/kong-inc/datadog/) \(`datadog`\)

  * プラグインスキーマは、 設定オプションのデフォルト値を 2箇所に分けて表示するのではなく、1箇所にまとめて表示するようになりました。 [\#8315](https://github.com/Kong/kong/pull/8315)

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \)

  * `ngx.shared.dict`演算の実行後にnil値チェックを追加することにより、 nil値に対する算術関数の実行に関連する500エラーを修正しました。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * コンシューマグループが適用されているにもかかわらず、
    適切な構成が提供されていない場合に発生する500エラーを修正しました。特定のコンシューマグループ設定が存在しない場合、
    コンシューマグループのデフォルトは
    元のプラグイン設定になります。

  * タイマーが切れて失敗する原因となっていたタイマーリークを修正しました。
    Kong が使用する他のタイマーを起動すると、エラー`too many pending timers`が表示されます。

    以前は、プラグインはネームスペースのメンテナンス処理ごとに1つのタイマーを使用していたため、
    流量制限を行うネームスペースの数が多いインスタンスでは
    タイマーの使用量が増えていました。現在、すべての名前空間のメンテナンスに単一のタイマーが使用されています。
  * `local`ストラテジがDB\-lessおよびハイブリッドデプロイメントで機能しない問題を修正しました。`local`ストラテジが定義されている場合、`sync_rate = null`と`sync_rate = -1`を
    許可するようになりました。

* [Exit Transformer](/hub/kong-inc/exit-transformer/) \(`exit-transformer`\)

  * Exit Transformerプラグインがプラグインイテレーターを壊し、後のプラグインが実行できなくなる問題を修正しました。

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/)（`mtls-auth`）

  * TLSを使用したルートまたはサービスへのアクセス時に発生する`attempt to index local 'workspace'`エラーを修正しました。

* [OAuth2 Introspection](/hub/kong-inc/oauth2-introspection/)\( `oauth2-introspection` \)

  * IDPがリバースプロキシの背後にある場合のTLS接続に関する問題を修正しました。

* [Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)

  * 大きなファイルをキャッシュするときに発生した`X-Cache-Status:Miss`エラーを修正しました。

* [Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)

  * 大きなファイルをキャッシュするときに発生した`X-Cache-Status:Miss`エラーを修正しました。

* [Response Transformer Advanced](/hub/kong-inc/response-transformer-advanced/)（ `response-transformer-advanced` ）

  * `body_filter`フェーズでは、プラグインは `nil` ではなく空文字列をボディに設定するようになりました。

* [jq](/hub/kong-inc/jq/)\(`jq`\)

  * プラグインに出力がない場合、元のレスポンスボディを復元しようとするのではなく、 生のボディを返すようになりました。

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * 間違った構成値をロードしていたネガティブキャッシュを修正しました。

* [JWT署名者](/hub/kong-inc/jwt-signer/)\( `jwt-signer` \)

  * `enable_hs_signatures`設定パラメータが機能しない問題を修正しました。プラグインは、nil値での演算を避けるために、 有効期限を早めに定義するようになりました。

### 依存関係

* OpenSSLを1\.1\.1lから1\.1\.1mまでアップグレードしました [\#8191](https://github.com/Kong/kong/pull/8191)
* `resty.session`を 3\.8 から 3\.10 にアップグレードしました [＃8294](https://github.com/Kong/kong/pull/8294)
* `lua-resty-openssl`を 0\.8\.5 にアップグレードしました [＃8368](https://github.com/Kong/kong/pull/8368)
* Dev Portal の`lodash`を 4\.17\.11 から 4\.17\.21 にアップグレードしました
* Kong Manager の`lodash`を 4\.17\.15 から 4\.17\.21 にアップグレードしました

### 非推奨

* 外部`go-pluginserver`プロジェクトは、[組み込みサーバーアプローチ](/gateway/latest/reference/external-plugins/)に取って代わられ、非推奨とされています。

* Kong Gateway 2\.8\.0\.0 以降、Kongは新しいオープンソースの
  CentOSイメージを構築していません。オープンソースのKong GatewayをCentOS上で実行するためのサポートは、
  [CentOSがEnd of Life（OEL）](https://www.centos.org/centos-linux-eol/)に達したため、非推奨となりました。

  現在、Kong Gateway EnterpriseをCentOS上で実行することはサポートされていますが、CentOS
  Kong Gateway 3\.xx では完全に非推奨となる予定です。
* OpenID Connectプラグイン：`session_redis_auth`フィールドは
  現在は非推奨であり、3\.xxで削除される予定です。代わりに
  `session_redis_username`と `session_redis_password`を使用します。

* Forward Proxy Advancedプラグイン：`proxy_port`フィールドと`proxy_host`フィールドは現在は非推奨であり、3\.xx で削除される予定です。`http_proxy_host` と `http_proxy_port`、または `https_proxy_host` と
  `https_proxy_port`を使います。

* AWS Lambdaプラグイン：`proxy_scheme`フィールドは非推奨となり、3\.xxで削除されます。

2\.7\.2\.0
-------------

**リリース日** ：2022/04/07

### 修正

#### Enterprise

* RBACで、 `endpoint=/kong workspace=*`のすべてのワークスペースから`/kong`エンドポイントにアクセスできない問題を修正しました
* 最上位レベルの`endpoint=*`権限を持たない管理者が、 `endpoint=/rbac`権限を持っていても RBAC ルールを追加できないという RBAC の問題を修正しました。これらの管理者は、現在のワークスペースに対してのみ RBAC ルールを追加できるようになりました。
* Kong Manager 
  * OpenID Connect \(OIDC\)使用時に`search_user_info`オプションを有効にします。
  * アップストリームページの壊れたドキュメントリンクを修正
  * 指定された値にカンマがある場合でもサーバーレス関数を保存可能
  * カスタムプラグインでプラグインの設定を表示しているときに \[編集\] ボタンを表示

* ノードの再起動時にキーが予期せずローテーションされる動作を修正
* RBAC トークン検証の実行時にキャッシュを追加
* ログメッセージ「プラグインイテレータが再構築中に変更されました」が誤って`error`として記録されました。このリリースでは、 `info`ログレベルに変換されます。
* Rate Limiting Advancedプラグインで流量制限カウンタがいっぱいになったときに発生する500エラーを修正しました。

#### プラグイン

* [HTTP Log](/hub/kong-inc/http-log/) \(`http-log`\)
  * ログを送信するときに、提供されたクエリ文字列パラメータを含めます。`http_endpoint`

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)
  * `https_proxy_host`を次の値と同じ値に設定することで、HTTPSリクエストのタイムアウト問題を修正しました：`http_proxy_host`
  * `host`ヘッダーを上書きする場合は小文字を使用してください
  * HTTPSリクエストで保護されたプロキシを使用する際のベーシック認証のサポートを追加します

* [StatsD Advanced](/hub/kong-inc/statsd-advanced/) \(`statsd-advanced`\)
  * `workspace_identifier`を次のように設定するためのサポートが追加されました。`workspace_name`

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)
  * プラグインが有効になっていない場合は、名前空間の作成をスキップします。これにより、「\[rate\-limiting\-advanced\] 共有辞書が指定されていません」というエラーがログに記録されなくなります。

* [Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)
  * 大きなファイルはメモリ使用量のためキャッシュされず、`X-Cache-Status:Miss`レスポンスとなります。これは解決されました

* [GraphQL Proxy Cache Advanced](/hub/kong-inc/graphql-proxy-cache-advanced/)\(`graphql-proxy-cache-advanced`\) 
  * 大きなファイルはメモリ使用量のためキャッシュされず、`X-Cache-Status:Miss`レスポンスとなります。これは解決されました

* [LDAP Authentication Advanced](/hub/kong-inc/ldap-auth-advanced/)\(`ldap-auth-advanced`\)
  * `:`文字を含むパスワードをサポートする

### 依存関係

* `openssl`を1\.1\.1kから1\.1\.1n までアップグレードし、CVE\-2022\-0778 [\#8635](https://github.com/Kong/kong/pull/8635)を解決しました
* `openresty`を 1\.19\.3\.2 から 1\.19\.9\.1 にアップグレードしました[\#7727](https://github.com/Kong/kong/pull/7727)

2\.7\.1\.2
-------------

**リリース日** ：2022/02/17

### 修正

#### Enterprise

* Kong ManagerのOIDC認証でエラー `“attempt to call method 'select_by_username_ignore_case' (a nil value)”` が発生し、OIDCでログインできない問題を修正しました。
* `cluster_allowed_common_names`に追加されたコモンネームが機能しない問題を修正しました。

2\.7\.1\.1
-------------

**リリース日** ：2022/02/04

### 修正

#### Enterprise

* 管理者が複数のワークスペースにアクセスできる場合に発生していた Kong Managerのパフォーマンスに関する問題を修正しました。
* TLSを使用してルートまたはサービスにアクセスする際に発生する`attempt to index local 'workspace'` エラーを修正しました。

2\.7\.1\.0
-------------

**リリース日** ：2022/01/27

### 機能

#### Enterprise

* ハイブリッドモードの展開用に [`cluster_max_payload`](/gateway/latest/reference/configuration/#cluster_max_payload) を設定できるようになりました。この構成オプションは、コントロールプレーンからデータプレーンに渡って送信できる最大ペイロード サイズを設定します。`payload too big`エラー が発生し、データプレーンに適用されないような大きな構成がある 環境では、この設定で制限を調整してください。
* ハイブリッド・モードで 証明書検証にPKIを使用する場合、 [`cluster_allowed_common_names`](/gateway/latest/reference/configuration/#cluster_allowed_common_names)オプションを使用して、コントロールプレーン（CP）への接続を許可するコモンネームのリストを構成できるようになりました。設定されていない場合、コントロールプレーン（CP）認証と同じ親ドメインを持つ データプレーン（DP）のみが許可されます。

### 修正

#### Enterprise

* Azure ADを使用した場合に、Kong ManagerへのOIDC認証が失敗する問題を修正しました。 
* タイマーを使い果たし、Kongが使用する他のタイマーを開始できず、エラー`too many pending timers`を表示するタイマーリークを修正しました。
* `data_plane_config_cache_mode`が`off`に設定されている場合、 データプレーン（DP）がコントロールプレーンからの更新を受信しない問題を修正しました。

#### コア

* 前のタイマーが終了した場合にのみ、解決タイマーを再スケジュールします。 [\#8344](https://github.com/Kong/kong/pull/8344)
* プラグイン、およびサブスキーマで実装されたエンティティは、 以前は非サブスキーマエンティティにのみ使用可能だった`transformations` と`shorthand_fields`プロパティを使用できるようになりました。 [\#8146](https://github.com/Kong/kong/pull/8146)

#### プラグイン

* [Rate Limiting](/hub/kong-inc/rate-limiting/)\( `rate-limiting` \)

  * `ngx.shared.dict`演算の実行後にnil値チェックを追加することにより、
    nil値に対する算術関数の実行に関連する500エラーを修正しました。

  * タイマーが切れて失敗する原因となっていたタイマーリークを修正しました。
    Kong が使用する他のタイマーを起動すると、エラー`too many pending timers`が表示されます。

    以前は、プラグインはネームスペースのメンテナンス処理ごとに1つのタイマーを使用していたため、
    流量制限を行うネームスペースの数が多いインスタンスでは
    タイマーの使用量が増えていました。現在、すべての名前空間のメンテナンスに単一のタイマーが使用されています。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * コンシューマグループが適用されているにもかかわらず、 適切な構成が提供されていない場合に発生する500エラーを修正しました。特定のコンシューマグループ設定が存在しない場合、 コンシューマグループのデフォルトは 元のプラグイン設定になります。

* [Exit Transformer](/hub/kong-inc/exit-transformer/) \(`exit-transformer`\)

  * Exit Transformerプラグインがプラグインイテレーターを壊し、後のプラグインが 実行できなくなる問題を修正しました。

2\.7\.0\.0
-------------

**リリース日** ：2021年12月16日

### 機能

#### Enterprise

* Kong Gatewayは、[Debian 10と11](/gateway/2.7.x/install-and-run/debian/)へのインストールをサポートするようになりました。

* このリリースでは、定義されたコンシューマのサブセットに対してカスタムの流量制限設定を管理できる新しいエンティティであるコンシューマグループを導入しています。
  流量制限にコンシューマグループを使用するには、[Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)プラグインを `enforce_consumer_groups` と `consumer_groups` パラメータで構成し、`/consumer_groups` エンドポイントを使用してグループを管理します。

  これは、同じレート制限設定を持つコンシューマを管理する場合に便利です。コンシューマグループを作成し、
  グループに対して1つのRate Limiting Advancedプラグインインスタンスを作成することで、
  プロキシ遅延を減らすことができます。
  * [コンシューマグループのリファレンス](/gateway/2.7.x/admin-api/consumer-groups/reference/)

  * [コンシューマグループの例](/gateway/2.7.x/admin-api/consumer-groups/examples/)

    この機能は現在、宣言的コンフィギュレーションではサポートされていません。

* データプレーン構成キャッシュを暗号化したり、完全にオフにできるようになりました。
  次の 2 つの新しい構成任意が追加されました。

  * [`data_plane_config_cache_mode`](/gateway/2.7.x/reference/configuration/#data_plane_config_cache_mode) ： キャッシュは`unencrypted`、 `encrypted`、または`off`になります。
  * [`data_plane_config_cache_path`](/gateway/2.7.x/reference/configuration/#data_plane_config_cache_path)： この設定を使用して、キャッシュのカスタムパスを指定します。

* `/license/report`API エンドポイントが[月次スループット使用レポート](/gateway/2.7.x/plan-and-deploy/licenses/report)を提供するようになりました。

#### Dev Portal

* Dev Portal API が、ソートされたリスト結果用の `sort_by={attribute}` および `sort_desc` クエリパラメータをサポートするようになりました。

* [OpenID Connect \(OIDC\)](/hub/kong-inc/openid-connect/)による Dev Portal 認証の改善：
  OIDC 認証が有効になっている場合、ユーザーが IDP 認証情報を使用して Dev Portal に
  初めてアクセスしようとすると、事前に入力された登録フォームが表示されます。
  フォームを送信してDev Portalのアカウントを作成し、そのアカウントをIDPにリンクします。

  リンク後、IDP 認証情報を使用して、この Dev 
  Portal アカウントに直接アクセスできます。
* Dev Portal API および GUI に TLSv1\.3 サポートが追加されました。

#### Kong Manager

* [OpenID Connectを使用してKong Managerを保護する](/gateway/2.7.x/configure/auth/kong-manager/oidc-mapping/)場合、Kong Managerで管理者を手動で作成し、そのロールをIDプロバイダにマッピングする必要がなくなります。代わりに、管理者は初回ログイン時に作成され、IdPのグループメンバーシップに基づいてロールが割り当てられます。この機能により、Kong ManagerとDev Portalの両方に管理者を作成する際の
  問題も一部解決されました。

  {:.important}
  > 
  > **重要：** **この機能で下位互換性のない変更が生じます** 。`admin_claim`パラメータは、以前のバージョンで必要だった`consumer_claim`パラメータに取って代わります。
  > Kong ManagerでOIDC認証を有効にするには、OIDC
  > 設定ファイルを更新する必要があります。詳細については、
  > [OIDC認証グループマッピングをご参照ください。](/gateway/2.7.x/configure/auth/kong-manager/oidc-mapping/)
* Kong Managerは、OpenID Connectプラグインを設定するための簡素化され整理されたフォームを提供するようになりました。ユーザーは、プラグインを設定するために必要なパラメータの共通セットを簡単に特定し、
  必要に応じてカスタム設定を追加できるようになりました。

#### コア

* サービスエンティティに必須の[`enabled`フィールド](/gateway/2.7.x/admin-api/#service-object)が追加されました
  デフォルトは`true`です。`false`に設定すると、
  サービスはプロキシルータに追加されません。[\#8113](https://github.com/Kong/kong/pull/8113)

  ハイブリッドモードの場合：
  * サービスに対して`enabled`が`false`に設定されている場合、そのサービスとそれに接続されたルートとプラグインはデータプレーンにエクスポートされません。
  * プラグインで`enabled`が`false`に設定されている場合、サービスの設定に関係なく、プラグインもデータプレーンにエクスポートされないようになりました。

* プラグインのDAOは、読み込み順序が明示的になるように、配列にリストされなければなりません。ハッシュのようなテーブルで読み込むことは **非推奨** となりました。
  [\#7942](https://github.com/Kong/kong/pull/7942)

* 接続を終了せずに[SNIに基づいてTLSトラフィックをルーティング](/gateway/2.7.x/reference/proxy/#proxy-tls-passthrough-traffic)
  する機能を追加しました。
  [\#6757](https://github.com/Kong/kong/pull/6757)

#### パフォーマンス

このリリースでは、パフォーマンスの向上に向けた取り組みを継続しました。

* プラグインイテレータのパフォーマンスと JITability を改善しました [\#7912](https://github.com/Kong/kong/pull/7912) [\#7979](https://github.com/Kong/kong/pull/7979)
* Kongコアコンテキストの読み取りと書き込みを簡素化し、パフォーマンスを向上しました。 [＃7919](https://github.com/Kong/kong/pull/7919)
* DBレス構成をリロードする際のプロキシのロングテールレイテンシーを減らしました [\#8133](https://github.com/Kong/kong/pull/8133)

#### PDK

* `body_filter`フェーズの2つの新しい関数を追加しました： [`kong.response.get_raw_body`](/gateway/2.7.x/pdk/kong.response/)と [`kong.response.set_raw_body`](/gateway/2.7.x/pdk/kong.response/)[\#7887](https://github.com/Kong/kong/pull/7877)

#### プラグイン

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`）

  * JWTアルゴリズム RS384 のサポートが追加されました。
  * このプラグインでは、 Redisクラスターのノードを`session_redis_cluster_nodes`フィールドを通して ホスト名で指定できるようになりました。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * `enforce_consumer_groups`と`consumer_groups`パラメータを追加し、コンシューマグループのサポートを導入しました。コンシューマグループを使用すると、 定義されたコンシューマのサブセットに対してカスタム流量制限設定を管理できます。

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)

  * 2つのプロキシ認証パラメータ`auth_username`と`auth_password`が追加されました。

* [Mocking](/hub/kong-inc/mocking/) \(`mocking`\)

  * `random_examples`パラメータを追加しました。この設定を使用して、模擬回答セットから ランダムに1つの例を選択します。

* [IP制限](/hub/kong-inc/ip-restriction/)\(`ip-restriction`\)

  * `status`と`message`の設定により、応答ステータスとメッセージをカスタマイズできるようになりました。[\#7728](https://github.com/Kong/kong/pull/7728)

    パッチを提供してくださった[timmkelley](https://github.com/timmkelley)に感謝いたします！

* [Datadog](/hub/kong-inc/datadog/) \(`datadog`\)

  * `distribution`メトリックタイプのサポートを追加しました。[\#6231](https://github.com/Kong/kong/pull/6231)

    パッチを提供してくださった[onematchfox](https://github.com/onematchfox)に感謝します！
  * 新しいプラグイン設定`service_tag`、`consumer_tag`、`status_tag`により、サービス、コンシューマ、ステータスタグをカスタマイズできるようにします。
    [\#6230](https://github.com/Kong/kong/pull/6230)

    パッチを提供してくださった[onematchfox](https://github.com/onematchfox)に感謝します！

* [gRPCゲートウェイ](/hub/kong-inc/grpc-gateway/)（`grpc-gateway`）と[gRPC Web](/hub/kong-inc/grpc-web/) （`grpc-web`）

  * 両プラグインはタイムスタンプトランスコードとインクルード`.proto`ファイル機能を共有します。\[\#7950\([https://github.com/Kong/kong/pull/7950](https://github.com/Kong/kong/pull/7950)\)

* [gRPC ゲートウェイ](/hub/kong-inc/grpc-gateway/)\(`grpc-gateway`\)

  * このプラグインは、インポートされた `.proto` ファイルで定義されたサービスとメソッドを処理するようになりました。 [\#8107](https://github.com/Kong/kong/pull/8107)

* [流量制限](/hub/kong-inc/rate-limiting/)（`rate-limiting`）
  設定プロパティ
  `redis_ssl` を `true`または`false`に設定可能、`redis_ssl_verify`、および
  `redis_server_name` を使用した Redis SSL のサポートを追加しました。
  [\#6737](https://github.com/Kong/kong/pull/6737)

  パッチを提供してくれた[gabeio](https://github.com/gabeio)に感謝します！

#### プラグインの暗号化

* プラグイン上でいくつかのフィールドが暗号化済みとしてマークされています。
  [キーリング暗号化なら](/gateway/2.7.x/plan-and-deploy/security/db-encryption/)
  が有効になっています。これらのフィールドは暗号化されます。

  * ACME：`account_email`、 `eab_kid`、および`eab_hmac_kid`
  * AWS Lambda：`aws_key`と`aws_secret`
  * Azure Functions：`apikey`と`clientid`
  * Basic Auth：`basicauth_credentials.password`
  * HTTP Log：`http_endpoint`
  * LDAP Auth Advanced：`ldap_password`
  * Loggly：`key`
  * OAuth2： `config.provision_key`パラメータ値と `oauth2_credentials.provision_key`フィールド
  * OpenID Connect：`client_id` 、 `client_secret` 、 `session_auth` 、および `session_redis_auth`
  * Session：`secret`
  * Vault：`vaults.vault_token`および`vault_credentials.secret_token`

  {:.note}
  > 
  > **注意** : 特定のプラグインにおいて、深くネストされたフィールドの暗号化に関する
  > 既知の問題があります。暗号化されているものの、現在動作していないフィールドを持つプラグインについては、[既知の問題セクション](#known-issues)をご参照ください。

### 修正

#### Enterprise

* Vitalsレポートの生成に関する問題を修正しました。InfluxDBでVitalsを実行し、2XX、4XX、5XX以外のステータスコードを含むレポートを生成しようとすると、レポート生成に失敗していました。この復旧により、予想されるコード以外のプロキシされたトラフィック
  はエラーにならず、代わりにVitalsレポートの
  カウント合計として表示されます。

* ハイブリッドモードにおけるレイテンシの問題を修正しました。以前は、データプレーンに多数の設定変更を
  同時に適用すると、すべてのアップストリームリクエストで
  高いレイテンシが発生していました。

* ユーザは、適切な権限を持っている限り、どのワークスペースからでも`super-admin`ロールを持つ管理者を正常に削除できるようになり、関連するコンシューマエンティティも削除されます。これにより、新しいユーザーが利用できるユーザー名が空きます。以前は、`super-admin`ロールを持つ管理者を、
  最初に作成されたワークスペースとは異なるワークスペースから削除しても、関連する
  コンシューマエンティティは削除されず、ユーザ名はロックされたままでした。たとえば、ワークスペース`dev`で作成された
  管理者がワークスペース`QA`から削除された場合、
  この問題が発生します。

* 電話ホームメトリクスはTLSで送信されるようになり、
  Kong Gatewayの使用状況に関する分析データは暗号化された接続を介して送信されるようになりました。

#### Dev Portal

* Dev Portal の OpenID Connect 認証が、`portal_auth`で設定された `login_redirect_uri`と `forbidden_redirect_uri`の値に基づいてユーザーを適切にリダイレクトするようになりました。これらの値が`portal_auth`で設定されていない場合、リダイレクト値はポータルテンプレートファイル`portal.conf.yaml`から取得されます。

* Dev Portalの認証方法としてOpenID Connectを使用している場合、開発者への更新が関連するコンシューマに正しく伝搬されるようになりました。

* Dev Portalのフッターのリンクを修正しました。

* Dev Portalのアクセシビリティを改善し、
  ラベル、名前、見出し、色のコントラストに関する様々な問題を修正しました：

  * キーボードからアクセス可能な応答例と「試してみる」セクション
  * フォーム入力にラベルが追加されました
  * 選択可能な要素にアクセス可能な名前が追加されました
  * アクティブな要素の一意のID
  * 見出しレベルは1つだけ増加し、正しい順序になっています
  * ボタンのコントラストの改善

* Dev Portalのアプリ登録サービスリストを表示する際の
  情報ツールチップのクラッシュとレンダリングの問題を修正しました。

* Dev Portalアプリケーションサービスリストでページ分割ができるように修正しました。

* Dev Portalのテーブル境界線のスタイルに関する問題を修正しました。

#### Kong Manager

* Kong Managerのアイコンの配置で、 **「削除（ゴミ箱）」** アイコンが **「表示」** リンクと重なり、ユーザーが誤って **「削除」** をクリックしてしまう問題を修正しました。

#### コア

* Goプラグインで`pluginsocket.proto`ファイルが見つからない問題を修正しました。
* 設定のリロード時にバランサーのキャッシュがリセットされるようになりました。 [\#7924](https://github.com/Kong/kong/pull/7924)
* 構成の再読み込みによって、新しいDNSリゾルバータイマーが開始されなくなりました。 [\#7943](https://github.com/Kong/kong/pull/7943)
* 複数ノードのCassandraクラスタをブートストラップする際に、スキーマの合意が発生する前に マイグレーションが挿入を試みることがあった問題を修正しました。 [\#7667](https://github.com/Kong/kong/pull/7667)
* カスタムプラグインがそのカスタムDAO上で相互に依存するエンティティスキーマ を持ち、それらが正しくない順序でロードされた場合に起こる断続的なボットエラーを修正しました。 [\#7911](https://github.com/Kong/kong/pull/7911)
* `encrypted=true`が各プラグインのサブスキーマではなく、メインのプラグインオブジェクトに適用される問題を修正しました。 

#### PDK

* `kong.log.inspect`ログレベルが `warn` から `debug`になりました。また、テキストボックスがより見やすくレンダリングされるようになりました。 [\#7815](https://github.com/Kong/kong/pull/7815)

#### プラグイン

* [LDAP Authentication](/hub/kong-inc/ldap-auth/) \(`ldap-auth`\)

  * パスワードにコロン（`:`）が含まれる場合に、ベーシック認証ヘッダーが正しく解析されない問題を修正しました。[\#7977](https://github.com/Kong/kong/pull/7977)

    [beldahanit](https://github.com/beldahanit)さん、問題をご報告いただきありがとうございます。

* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`）

  * コントロールプレーン（CP）はバランサーを初期化しないため、 表示する実際のメトリクスがありません。 [\#7992](https://github.com/Kong/kong/pull/7922)

* [Request Validator](/hub/kong-inc/request-validator/) \(`request-validator`\)

  * バージョン1\.1\.3で複数の値を配列として解析する際の変更を元に戻しました。 ヘッダとクエリ引数を`primitive`として指定した場合、配列としてマージされるのではなく、重複して指定された場合に個別に検証されるようになりました。
  * CSV値を囲む空白は、RFCによれば重要ではない（空白はオプションである）ため、削除されました。
  * `openapi3-deserialiser`を2\.0\.0に変更し、アップグレードしました

* [プロキシを転送](/hub/kong-inc/forward-proxy/)\(`forward-proxy` \)

  * このプラグインは、以前はDEBUGログに非推奨の警告を追加していた`lua-resty-http`依存の非推奨機能を使用しなくなりました。
  * このプラグインは、以前はDEBUGログに非推奨の警告を追加していた`upstream_host`依存の 非推奨機能を使用しなくなりました。

* [OAuth2 Introspection](/hub/kong-inc/oauth2-introspection/)\( `oauth2-introspection` \)

  * このプラグインは、以前はDEBUGログに非推奨の警告を追加していた`lua-resty-http`依存の非推奨機能を使用しなくなりました。

* [GraphQL Rate Limiting Advanced](/hub/kong-inc/graphql-rate-limiting-advanced/) \(`graphql-rate-limiting-advanced`\)

  * プラグイン有効化後に HTTP 500ステータスコードを引き起こすプラグイン初期化コードを修正しました。

* [Mutual TLS Authentication](/hub/kong-inc/mtls-auth/) \(`mtls-auth`\)

  * CRLキャッシュが適切に無効化されず、すべての証明書が 無効に表示される問題を修正しました。

* [Proxy Cache Advanced](/hub/kong-inc/proxy-cache-advanced/)\(`proxy-cache-advanced`\)

  * ログフェーズでプラグインが呼び出されたときに 発生していた`function cannot be called in access phase`エラーを修正しました。

* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)

  * 設定されたウィンドウサイズと制限パラメータの数が等しくない場合に、`config.limit`と`config.window_size`カウント のスキーマエンティティチェックを修正しました。

### 依存関係

* テストで使用される`go-pdk`を v0\.6\.0 から v0\.7\.1 にアップグレードしました [＃7964](https://github.com/Kong/kong/pull/7964)
* `grpcurl`を 1\.8\.2 から 1\.8\.5 にアップグレードしました [＃7959](https://github.com/Kong/kong/pull/7959)
* `lua-pack`を 1\.0\.5 から 2\.0\.0 にアップグレードしました [＃8004](https://github.com/Kong/kong/pull/8004)
* `luaossl`への依存関係が削除されます。
* `lua-resty-openapi3-deserializer`（ライブラリの依存関係） 
  * `1.1.0`のヘッダー修正を元に戻しました。プリミティブ型の場合、値は配列ではなくプレーンな文字列として返されます。 重複する値は、 逆シリアル化のために個別に提供されます。
  * RFCによれば、ヘッダーから空白を取り除くことは重要ではありません。

### 非推奨

* Kong Immunityは非推奨、削除されており、Kong Gatewayではご利用いただけません。

* Kong Gateway
  のバックエンドデータベースとしてのCassandraは、このリリースで非推奨となり、将来のバージョンで削除される予定です。

  Cassandraの削除対象はKong Gateway 3\.4リリースです。
  Kong Gateway 3\.0リリース以降、一部の新機能は
  Cassandraでサポートされない場合があります。ユーザーがCassandraで対処できた
  ユースケースを満たす十分な時間と選択肢を提供することです。
* 古い`BasePlugin`モジュールは非推奨であり、将来のバージョンのKongでは削除される予定予定です。
  [ドキュメント](/gateway/2.7.x/plugin-development/custom-logic/#migrating-from-baseplugin-module)で移行のヒントをご参照ください。

* プラグインDAOのハッシュ構文は非推奨です。プラグインDAOはハッシュではなく配列にする必要があります。

### 既知の問題

* Kong Gatewayには、キーリング暗号化がプラグイン内の深くネストされたフィールドで機能しないようにするバグがあるため、`encrypted=true`設定は以下のプラグインとフィールドには影響しません：

  * JWT署名者：`jwt_signer_jwks.previous[...].`のうちで `d` 、 `p` 、 `q` 、 `dp` 、 `dq` 、 `qi` 、 `k``jwt_signer_jwks.keys[...]`
  * Kafka Log：`config.authentication.user`と`config.authentication.password`
  * Kafka Upstream：`config.authentication.user`と`config.authentication.password`
  * OpenID Connect：フィールド`d` 、 `p` 、 `q` 、 `dp` 、 `dq` 、 `qi` 、 `oth` 、 `r` 、 `t` 、および`k` `openid_connect_jwks.previous[...].`内と `openid_connect_jwks.keys[...]`

* コンシューマグループは、decKによる宣言型構成ではサポートされていません。構成にコンシューマグループがある場合、deckはそれを無視します。

* カスタムプラグインでSSL証明書を使用している場合、`ngc.ctx`に証明書フェーズを設定する必要があるかもしれません。

2\.6\.1\.0
-------------

**リリース日** ：2022/04/07

### 修正

#### Enterprise

* RBACで、 `endpoint=/kong workspace=*`のすべてのワークスペースから`/kong`エンドポイントにアクセスできない問題を修正しました
* 最上位レベルの`endpoint=*`権限を持たない管理者が、 `endpoint=/rbac`権限を持っていても RBAC ルールを追加できないという RBAC の問題を修正しました。これらの管理者は、現在のワークスペースに対してのみ RBAC ルールを追加できるようになりました。

#### プラグイン

* [HTTP Log](/hub/kong-inc/http-log/) \(`http-log`\)
  * ログを送信するときに、提供されたクエリ文字列パラメータを含めます。`http_endpoint`

### 依存関係

* `openssl`を1\.1\.1kから1\.1\.1n までアップグレードし、CVE\-2022\-0778 [\#8635](https://github.com/Kong/kong/pull/8635)を解決しました
* `luarocks`を 3\.7\.1 から 3\.8\.0 にアップグレードしました[\#8630](https://github.com/Kong/kong/pull/8630)
* `openresty`を 1\.19\.3\.2 から 1\.19\.9\.1 にアップグレードしました[\#7727](https://github.com/Kong/kong/pull/7727)

2\.6\.0\.4
-------------

**リリース日** ：2022/02/10

### 修正

#### Enterprise

* Kong ManagerのOIDC認証でエラー `“attempt to call method 'select_by_username_ignore_case' (a nil value)”` が発生し、OIDCでログインできない問題を修正しました。

2\.6\.0\.3
-------------

**リリース日** ：2022/01/27

### 機能

#### Enterprise

* ハイブリッドモードの展開用に [`cluster_max_payload`](/gateway/latest/reference/configuration/#cluster_max_payload) を設定できるようになりました。この構成オプションは、コントロールプレーンからデータプレーンに渡って送信できる最大ペイロード サイズを設定します。`payload too big`エラー が発生し、データプレーンに適用されないような大きな構成がある 環境では、この設定で制限を調整してください。

### 修正

#### Enterprise

* 電話ホームメトリクスはTLSで送信されるようになり、 Kong Gatewayの使用状況に関する分析データは暗号化された接続を介して送信されるようになりました。
* Azure ADを使用した場合に、Kong ManagerへのOIDC認証が失敗する問題を修正しました。 
* タイマーを使い果たし、Kongが使用する他のタイマーを開始できず、エラー`too many pending timers`を表示するタイマーリークを修正しました。
* Kong Managerのアイコンの配置で、 **「削除（ゴミ箱）」** アイコンが **「表示」** リンクと重なり、ユーザーが誤って **「削除」** をクリックしてしまう問題を修正しました。

#### Dev Portal

* Dev Portalアプリケーションサービスリストでページ分割ができるように修正しました。
* 表の境界線のスタイル設定の問題を修正しました。
* モーダルアクセシビリティの問題を修正しました。

#### プラグイン

* [Rate Limiting](/hub/kong-inc/rate-limiting/)（ `rate-limiting` ）と [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced)\( `rate-limiting-advanced` \) 
  * タイマーが切れて失敗する原因となっていたタイマーリークを修正しました。
    Kong が使用する他のタイマーを起動すると、エラー`too many pending timers`が表示されます。

    以前は、プラグインはネームスペースのメンテナンス処理ごとに1つのタイマーを使用していたため、
    流量制限を行うネームスペースの数が多いインスタンスでは
    タイマーの使用量が増えていました。現在、すべての名前空間のメンテナンスに単一のタイマーが使用されています。

2\.6\.0\.2
-------------

**リリース日：** 2021/12/03

### 修正

#### Dev Portal

* Dev Portalのフッターのリンクを修正しました。

* Dev Portalのアクセシビリティを改善し、
  ラベル、名前、見出し、色のコントラストに関する様々な問題を修正しました：

  * キーボードからアクセス可能な応答例と「試してみる」セクション
  * フォーム入力にラベルが追加されました
  * 選択可能な要素にアクセス可能な名前が追加されました
  * アクティブな要素の一意のID
  * 見出しレベルは1つだけ増加し、正しい順序になっています
  * ボタンのコントラストの改善

* PATCH リクエストで許可されたフィールドのみを受け付けるように 
  Dev Portal API `/applications` エンドポイントを修正しました。

* Dev Portalのアプリ登録サービスリストを表示する際の
  情報ツールチップのクラッシュとレンダリングの問題を修正しました。

#### プラグイン

* [OpenID Connect](/hub/kong-inc/openid-connect/)（`openid-connect`） 
  * このプラグインでは、 Redisクラスターのノードを`session_redis_cluster_nodes`フィールドを通して ホスト名で指定できるようになりました。

2\.6\.0\.1
-------------

**リリース日：** 2021年11月18日

### 修正

#### Enterprise

* Vitalsレポートの生成に関する問題を修正しました。InfluxDBでVitalsを実行し、2XX、4XX、5XX以外のステータスコードを含むレポートを生成しようとすると、レポート生成に失敗していました。この復旧により、予想されるコード以外のプロキシされたトラフィック
  はエラーにならず、代わりにVitalsレポートの
  カウント合計として表示されます。

* ハイブリッドモードにおけるレイテンシの問題を修正しました。以前は、データプレーンに多数の
  構成変更を同時に適用すると、すべての
  アップストリームリクエストで高いレイテンシが発生していました。

* Dev Portalの一意でないIDに関するアクセシビリティの問題を修正しました。

* Dev Portalの認証方法としてOpenID Connectを使用している場合、開発者への更新が関連するコンシューマに正しく伝搬されるようになりました。

* ユーザは、適切な権限を持っている限り、どのワークスペースからでも`super-admin`ロールを持つ管理者を正常に削除できるようになり、関連するコンシューマエンティティも削除されます。これにより、新しいユーザーが利用できるユーザー名が空きます。以前は、`super-admin`ロールを持つ管理者を、
  最初に作成されたワークスペースとは異なるワークスペースから削除しても、関連する
  コンシューマエンティティは削除されず、ユーザ名はロックされたままでした。たとえば、ワークスペース`dev`で作成された
  管理者がワークスペース`QA`から削除された場合、
  この問題が発生します。

### 依存関係

* kong\-redis\-cluster を`1.1-0`から`1.2.0`に変更しました。 
  * この更新により、クラスタ全体が再起動され、 新しいIPアドレスを使用して起動した場合、クラスタクライアントは自動的に回復することができます。

2\.6\.0\.0
-------------

**リリース日：** 2021年10月14日

### 機能

#### Enterprise

#### コア

このリリースには、新しいスキーマエンティティバリデータ`mutually_exclusive`の追加が含まれています。以前は、`only_one_of`バリデータは少なくとも1つのフィールドが設定されている必要がありました。この新しいエンティティバリデータでは、どちらか一方または両方のフィールドのみを設定することができます。
[\#7765](https://github.com/Kong/kong/pull/7765)

#### 構成

* サポートされていない実験的な機能として、 `dns_order`で IPV6 を有効にします。 [\#7819](https://github.com/Kong/kong/pull/7819)。
* テンプレートレンダラーは`os.getenv`を使用できるようになりました。 [\#6872](https://github.com/Kong/kong/pull/6872)。

#### ハイブリッドモード

* コントロールプレーンがより新しいバージョンを使用している場合、データプレーンはいくつかの未知のフィールドを削除できます。 [\#7827](https://github.com/Kong/kong/pull/7827)。

#### Admin API

* すべての Admin API エンドポイントに HTTP HEAD メソッドのサポートが追加されました。[\#7796](https://github.com/Kong/kong/pull/7796)
* OPTIONS リクエストのサポートを改善しました。以前は、Admin APIはすべてのOPTIONSリクエストに対して同じリプライを返していましたが、現在はOPTIONSリクエストはAdmin APIが持つルートにのみリプライします。存在しないルートには、 404が戻ってきました。さらに、Allow ヘッダーが応答に追加されました。AllowとAccess\-Control\-Allow\-Methodsの両方が 特定のAPIがサポートするメソッドのみを含むようになりました。 [\#7830](https://github.com/Kong/kong/pull/7830)

#### プラグイン

* **新しいプラグイン:** [jq](/hub/kong-inc/jq/) \(`jq`\) jq プラグインは、API リクエストまたはレスポンスに含まれる JSON オブジェクトに対して任意の jq 変換を有効にします。
* [Kafka Log](/hub/kong-inc/kafka-log/)\( `kafka-log` \) Kafka Logプラグインは、TLS、mTLS、および SASL 認証をサポートするようになりました。SASL認証は、PLAIN、SCRAM\-SHA\-256、 デリゲーショントークンをサポートしています。
* [Kafka Upstream](/hub/kong-inc/kafka-upstream/) \(`kafka-upstream`\) Kafka Upstream プラグインは、TLS、mTLS、および SASL 認証をサポートするようになりました。SASL認証は、PLAIN、SCRAM\-SHA\-256、 デリゲーショントークンをサポートしています。
* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)
  * Rate Limiting Advancedプラグインに新しい識別子タイプ`path`が追加され、リクエストパスのマッチングによる流量制限が可能になりました。
  * プラグインはスキーマに`local`ストラテジーを持つようになりました。ローカルストラテジーは自動的に`config.sync_rate`を\-1にします。
  * 設定可能な最大同期レートは 1 秒間隔でした。この同期速度は、 最小許容間隔を1秒から0\.020秒（20ms）に短縮することで向上しました。

* [OPA](/hub/kong-inc/opa/)（`opa`） OPAプラグインにリクエストパスパラメータが追加され、管理者がパスに対するポリシーを設定しやすくなりました。
* [カナリア](/hub/kong-inc/canary/) \(`canary`\) Canaryプラグインに、ヘッダーをハッシュする任意が追加されました（IPにフォールバックし、次にランダムになります）。
* [OpenID Connect](/hub/kong-inc/openid-connect/)（ `openid-connect` ） v2\.1\.0 にアップグレードし、古いデータプレーン（DP）とのバージョン互換性を維持します。 
  * v2\.0\.x の機能は次のとおりです： 
    * OpenID Connectプラグインは、 `userinfo`エンドポイントからの JWT応答を処理できるようになりました。
    * プラグインがJWE Introspectionをサポートするようになりました。

  * v2\.1\.x の機能には次のものが含まれます： 
    * このプラグインに新しいパラメータ `by_username_ignore_case`が追加され、`consumer_by` ユーザ名値を Identity Providerのクレームと大文字小文字を区別せずに一致させることができるようになりました。

* [Request Transformer Advanced](/hub/kong-inc/request-transformer-advanced/)（`request-transformer-advanced`） 
  * このリリースには、URLエンコード変換されたパスの修正が含まれています。プラグインは、`ngx.var.upstream_uri`を置き換えて上流のURI を設定するためにPDK関数を使うようになりました。

* [AWS\-ラムダ](/hub/kong-inc/aws-lambda/)（`aws-lambda`） プラグインは `AWS_REGION` と `AWS_DEFAULT_REGION` 環境変数を使用してAWSリージョンの検出を試みます（プラグインの設定で指定されていない場合）。 これにより、ユーザーはKongノードごとに「リージョン」を指定できるようになり、 Kongが配置されているのと同じリージョンでラムダ関数を呼び出す機能が追加されます。 [\#7765](https://github.com/Kong/kong/pull/7765)
* [Datadog](/hub/kong-inc/datadog/)（`datadog`）Datadog プラグインで`host`と `port`の設定オプションを環境変数 `KONG_DATADOG_AGENT_HOST` と`KONG_DATADOG_AGENT_PORT` から設定できるようになりました。このアップデートにより、ユーザーは Kongノード単位で異なるデスティネーションを設定できるようになり、マルチDCのセットアップが容易になり、 KubernetesではDatadogエージェントをデーモンセットとして実行できるようになりました。 [\#7463](https://github.com/Kong/kong/pull/7463)
* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`） Prometheusプラグインに新しいメトリック `data_plane_cluster_cert_expiry_timestamp` が追加され、ハイブリッドモードでの監視を改善するために、データプレーン（DP）の有効期限タイムスタンプ `cluster_cert` を公開するようになりました。[\#7800](https://github.com/Kong/kong/pull/7800)。
* [GRPCゲートウェイ](/hub/kong-inc/grpc-gateway/)\(grpc\-gateway\) 
  * gRPC側の`.google.protobuf.Timestamp`型のフィールドは現在 REST側でISO8601文字列との間でトランスコードされます。 [\#7538](https://github.com/Kong/kong/pull/7538)
  * `..?foo.bar=x&foo.baz=y`のようなURI引数は、 `{"foo": {"bar": "x", "baz": "y"}}`と等価な構造化フィールドとして解釈されます。 [\#7564](https://github.com/Kong/kong/pull/7564)

* [Request Termination](/hub/kong-inc/request-termination/) \(`request-termination`\)
  * Request Terminationプラグインは、トリガーのような名前のヘッダーやクエリパラメータを持つリクエストに対してのみプラグインを有効にする、新しい`trigger`設定オプションを含むようになりました。この設定オプション は、実際に処理されているトラフィックに影響を与えることなく、デバッグの大きな助けとなります。 [\#6744](https://github.com/Kong/kong/pull/6744)。
  * `request-echo`設定任意が追加されました。設定されている場合、プラグインは受信リクエストのコピーで応答します。 この設定オプションは、特に新しい`trigger`オプションと組み合わせると、Kong Gatewayが1つ以上の他のプロキシやLBの後ろにある場合のトラブルシューティングを容易にします。[\#6744](https://github.com/Kong/kong/pull/6744)。

### 依存関係

* `openresty`を 1\.19\.3\.2 から[1\.19\.9\.1](https://openresty.org/en/changelog-1019009.html)にアップグレードしました [＃7430](https://github.com/Kong/kong/pull/7727)
* `openssl`を`1.1.1k`から`1.1.1l`に上げました [7767](https://github.com/Kong/kong/pull/7767)
* `lua-resty-http`を 0\.15 から 0\.16\.1 にアップグレードしました [＃7797](https://github.com/kong/kong/pull/7797)
* `Penlight`を 1\.11\.0 にアップグレードしました [＃7736](https://github.com/Kong/kong/pull/7736)
* `lua-resty-http`を 0\.15 から 0\.16\.1 にアップグレードしました [＃7797](https://github.com/kong/kong/pull/7797)
* `lua-protobuf`を 0\.3\.2 から 0\.3\.3 にアップグレードしました [\#7656](https://github.com/Kong/kong/pull/7656)
* `lua-resty-openssl`を 0\.7\.3 から 0\.7\.4 にアップグレードしました [＃7657](https://github.com/Kong/kong/pull/7657)
* `lua-resty-acme`を 0\.6 から 0\.7\.1 にアップグレードしました [＃7658](https://github.com/Kong/kong/pull/7658)
* `grpcurl`を 1\.8\.1 から 1\.8\.2 にアップグレードしました [＃7659](https://github.com/Kong/kong/pull/7659)
* `luasec`を 1\.0\.1 から 1\.0\.2 にアップグレードしました [\#7750](https://github.com/Kong/kong/pull/7750)
* `lua-resty-ipmatcher`を 0\.6\.1 にアップグレードしました [＃7703](https://github.com/Kong/kong/pull/7703)

### 修正

#### Enterprise

* このリリースには、メトリックを挿入する際の Vitals InfluxDB タイムスタンプ生成に関する問題の修正が含まれています。
* Kong Gateway \(Enterprise\) は`consumer_reset_secrets`をエクスポートしなくなりました。
* Kongプロセス開始時（例えば、kong start）にキーリングデータが適切に生成されず、 有効化されない問題を修正しました。
* Kong Gatewayは、暗号化解除が試みられたときに、暗号化されたフィールドを早期に返すようになりました。これにより、Kong Gatewayが暗号化解除されたフィールドを認識できるように、キーをインポートする時間が与えられます。 以前は、データベースにキーリング暗号化されたデータフィールドがある場合、Kong Gatewayは`init*`フェーズ（`init`または`init_worker`フェーズ、つまりKongプロセスが開始されたとき）でそれらを復号化しようとし、「no request found」のようなエラーが発生していました。 **Kongプロセスの開始後も、キーをインポートする必要があります。** 
* [キーリング暗号化](/gateway/latest/kong-enterprise/db-encryption/)機能は、アルファ品質状態ではなくなりました。
* このリリースにはCentOS Dockerイメージビルドの修正が含まれており、CentOS 7イメージが適切に生成されるようになっています。

#### コア

* バランサの再試行で `:authority` 擬似ヘッダが正しく設定されるようになりました。 [\#7725](https://github.com/Kong/kong/pull/7725)。
* バランサーの再作成中にヘルスチェックが停止するようになりました。[\#7549](https://github.com/Kong/kong/pull/7549)。
* 誤った`Accept`ヘッダが予期しないHTTP 500を引き起こす可能性がある問題を修正しました。 [\#7757](https://github.com/Kong/kong/pull/7757)。
* Kongは`Proxy-Authentication`リクエストヘッダと`Proxy-Authenticate`レスポンスヘッダを削除しなくなりました。 [\#7724](https://github.com/Kong/kong/pull/7724)。
* Kong が正規表現とプレフィックスパスの両方を持つルートを正しく並べ替えない問題を修正しました。 [\#7695](https://github.com/Kong/kong/pull/7695)

#### ハイブリッドモード

* データプレーン（DP）のコンフィグスレッドが適切に終了するようにし、半デッドロック状態を防ぎます。 [\#7568](https://github.com/Kong/kong/pull/7568)

##### CLI

* `kong config parse`Goプラグインサーバーが有効になっている場合でもクラッシュしなくなりました。 [\#7589](https://github.com/Kong/kong/pull/7589)。

##### 構成

* 宣言的構成パーサは、不明な外部参照を印刷する際に、より正しいエラーを印刷するようになりました。 [\#7756](https://github.com/Kong/kong/pull/7756)。
* 宣言的構成のYAMLアンカーが適切に処理されます。[\#7748](https://github.com/Kong/kong/pull/7748)。

##### Admin API

* `GET /upstreams/:upstreams/targets/:target`ターゲットウェイトが0の場合、404を返さなくなりました。[\#7758](https://github.com/Kong/kong/pull/7758)

##### PDK

* `kong.response.exit`見つかった場合はカスタマイズされた「Content\-Length」ヘッダーを使用するようになりました。 [\#7828](https://github.com/Kong/kong/pull/7828)。

##### プラグイン

* [ACME](/hub/kong-inc/acme/) \(`acme`\) ワイルドカードドメインのドットはエスケープされています。 [\#7839](https://github.com/Kong/kong/pull/7839)。
* [Prometheus](/hub/kong-inc/prometheus/)（ `prometheus` ） アップストリームのヘルス情報に、これまで欠落していた`subsystem`フィールドが含まれるようになりました。 [\#7802](https://github.com/Kong/kong/pull/7802)。
* [プロキシキャッシュ](/hub/kong-inc/proxy-cache/) \(`proxy-cache`\) プラグインがキャッシュからデータを取得しても返さない場合がある問題を修正しました。 [\#7775](https://github.com/Kong/kong/pull/7775)

2\.6\.0\.0 \(beta1\)
-------------------------

**リリース日：** 2021/10/04

### 機能

#### Enterprise

#### コア

このリリースには、新しいスキーマエンティティバリデータ`mutually_exclusive`の追加が含まれています。以前は、`only_one_of`バリデータは少なくとも1つのフィールドが設定されている必要がありました。この新しいエンティティバリデータでは、どちらか一方または両方のフィールドのみを設定することができます。
[\#7765](https://github.com/Kong/kong/pull/7765)

#### 構成

* サポートされていない実験的な機能として、 `dns_order`で IPV6 を有効にします。 [\#7819](https://github.com/Kong/kong/pull/7819)。
* テンプレートレンダラーは`os.getenv`を使用できるようになりました。 [\#6872](https://github.com/Kong/kong/pull/6872)。

#### ハイブリッドモード

* コントロールプレーンがより新しいバージョンを使用している場合、データプレーンはいくつかの未知のフィールドを削除できます。 [\#7827](https://github.com/Kong/kong/pull/7827)。

#### Admin API

* すべての Admin API エンドポイントに HTTP HEAD メソッドのサポートが追加されました。[\#7796](https://github.com/Kong/kong/pull/7796)
* OPTIONS リクエストのサポートを改善しました。以前は、Admin APIはすべてのOPTIONSリクエストに対して同じリプライを返していましたが、現在はOPTIONSリクエストはAdmin APIが持つルートにのみリプライします。存在しないルートには、 404が戻ってきました。さらに、Allow ヘッダーが応答に追加されました。AllowとAccess\-Control\-Allow\-Methodsの両方が 特定のAPIがサポートするメソッドのみを含むようになりました。 [\#7830](https://github.com/Kong/kong/pull/7830)

#### プラグイン

* **新しいプラグイン:** [jq](/hub/kong-inc/jq/) \(`jq`\) jq プラグインは、API リクエストまたはレスポンスに含まれる JSON オブジェクトに対して任意の jq 変換を有効にします。
* [Kafka Log](/hub/kong-inc/kafka-log/)\( `kafka-log` \) Kafka Logプラグインは、TLS、mTLS、および SASL 認証をサポートするようになりました。SASL認証は、PLAIN、SCRAM\-SHA\-256、 デリゲーショントークンをサポートしています。
* [Kafka Upstream](/hub/kong-inc/kafka-upstream/) \(`kafka-upstream`\) Kafka Upstream プラグインは、TLS、mTLS、および SASL 認証をサポートするようになりました。SASL認証は、PLAIN、SCRAM\-SHA\-256、 デリゲーショントークンをサポートしています。
* [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)\( `rate-limiting-advanced` \)
  * Rate Limiting Advancedプラグインに新しい識別子タイプ`path`が追加され、リクエストパスのマッチングによる流量制限が可能になりました。
  * プラグインはスキーマに`local`ストラテジーを持つようになりました。ローカルストラテジーは自動的に`config.sync_rate`を\-1にします。
  * 設定可能な最大同期レートは 1 秒間隔でした。この同期速度は、 最小許容間隔を1秒から0\.020秒（20ms）に短縮することで向上しました。

* [OPA](/hub/kong-inc/opa/)（`opa`） OPAプラグインにリクエストパスパラメータが追加され、管理者がパスに対するポリシーを設定しやすくなりました。
* [カナリア](/hub/kong-inc/canary/) \(`canary`\) Canaryプラグインに、ヘッダーをハッシュする任意が追加されました（IPにフォールバックし、次にランダムになります）。
* [OpenID Connect](/hub/kong-inc/openid-connect/)（ `openid-connect` ） v2\.1\.0 にアップグレードし、古いデータプレーン（DP）とのバージョン互換性を維持します。 
  * v2\.0\.x の機能は次のとおりです： 
    * OpenID Connectプラグインは、 `userinfo`エンドポイントからの JWT応答を処理できるようになりました。
    * プラグインがJWE Introspectionをサポートするようになりました。

  * v2\.1\.x の機能には次のものが含まれます： 
    * このプラグインに新しいパラメータ `by_username_ignore_case`が追加され、`consumer_by` ユーザ名値を Identity Providerのクレームと大文字小文字を区別せずに一致させることができるようになりました。

* [AWS\-ラムダ](/hub/kong-inc/aws-lambda/)（`aws-lambda`） プラグインは `AWS_REGION` と `AWS_DEFAULT_REGION` 環境変数を使用してAWSリージョンの検出を試みます（プラグインの設定で指定されていない場合）。 これにより、ユーザーはKongノードごとに「リージョン」を指定できるようになり、 Kongが配置されているのと同じリージョンでラムダ関数を呼び出す機能が追加されます。 [\#7765](https://github.com/Kong/kong/pull/7765)
* [Datadog](/hub/kong-inc/datadog/)（`datadog`）Datadog プラグインで`host`と `port`の設定オプションを環境変数 `KONG_DATADOG_AGENT_HOST` と`KONG_DATADOG_AGENT_PORT` から設定できるようになりました。このアップデートにより、ユーザーは Kongノード単位で異なるデスティネーションを設定できるようになり、マルチDCのセットアップが容易になり、 KubernetesではDatadogエージェントをデーモンセットとして実行できるようになりました。 [\#7463](https://github.com/Kong/kong/pull/7463)
* [Prometheus](/hub/kong-inc/prometheus/)（`prometheus`） Prometheusプラグインに新しいメトリック `data_plane_cluster_cert_expiry_timestamp` が追加され、ハイブリッドモードでの監視を改善するために、データプレーン（DP）の有効期限タイムスタンプ `cluster_cert` を公開するようになりました。[\#7800](https://github.com/Kong/kong/pull/7800)。
* [GRPCゲートウェイ](/hub/kong-inc/grpc-gateway/)\(grpc\-gateway\) 
  * gRPC側の`.google.protobuf.Timestamp`型のフィールドは現在 REST側でISO8601文字列との間でトランスコードされます。 [\#7538](https://github.com/Kong/kong/pull/7538)
  * `..?foo.bar=x&foo.baz=y`のようなURI引数は、 `{"foo": {"bar": "x", "baz": "y"}}`と等価な構造化フィールドとして解釈されます。 [\#7564](https://github.com/Kong/kong/pull/7564)

* [Request Termination](/hub/kong-inc/request-termination/) \(`request-termination`\)
  * Request Terminationプラグインは、トリガーのような名前のヘッダーやクエリパラメータを持つリクエストに対してのみプラグインを有効にする、新しい`trigger`設定オプションを含むようになりました。この設定オプション は、実際に処理されているトラフィックに影響を与えることなく、デバッグの大きな助けとなります。 [\#6744](https://github.com/Kong/kong/pull/6744)。
  * `request-echo`設定任意が追加されました。設定されている場合、プラグインは受信リクエストのコピーで応答します。 この設定オプションは、特に新しい`trigger`オプションと組み合わせると、Kong Gatewayが1つ以上の他のプロキシやLBの後ろにある場合のトラブルシューティングを容易にします。[\#6744](https://github.com/Kong/kong/pull/6744)。

### 依存関係

* `openresty`を 1\.19\.3\.2 から[1\.19\.9\.1](https://openresty.org/en/changelog-1019009.html)にアップグレードしました [＃7430](https://github.com/Kong/kong/pull/7727)
* `openssl`を`1.1.1k`から`1.1.1l`に上げました [7767](https://github.com/Kong/kong/pull/7767)
* `lua-resty-http`を 0\.15 から 0\.16\.1 にアップグレードしました [＃7797](https://github.com/kong/kong/pull/7797)
* `Penlight`を 1\.11\.0 にアップグレードしました [＃7736](https://github.com/Kong/kong/pull/7736)
* `lua-resty-http`を 0\.15 から 0\.16\.1 にアップグレードしました [＃7797](https://github.com/kong/kong/pull/7797)
* `lua-protobuf`を 0\.3\.2 から 0\.3\.3 にアップグレードしました [\#7656](https://github.com/Kong/kong/pull/7656)
* `lua-resty-openssl`を 0\.7\.3 から 0\.7\.4 にアップグレードしました [＃7657](https://github.com/Kong/kong/pull/7657)
* `lua-resty-acme`を 0\.6 から 0\.7\.1 にアップグレードしました [＃7658](https://github.com/Kong/kong/pull/7658)
* `grpcurl`を 1\.8\.1 から 1\.8\.2 にアップグレードしました [＃7659](https://github.com/Kong/kong/pull/7659)
* `luasec`を 1\.0\.1 から 1\.0\.2 にアップグレードしました [\#7750](https://github.com/Kong/kong/pull/7750)
* `lua-resty-ipmatcher`を 0\.6\.1 にアップグレードしました [＃7703](https://github.com/Kong/kong/pull/7703)

### 修正

#### Enterprise

* このリリースには、メトリックを挿入する際の Vitals InfluxDB タイムスタンプ生成に関する問題の修正が含まれています。
* Kong Gateway \(Enterprise\) は`consumer_reset_secrets`をエクスポートしなくなりました。
* Kongプロセス開始時（例えば、kong start）にキーリングデータが適切に生成されず、 有効化されない問題を修正しました。
* Kong Gatewayは、暗号化解除が試みられたときに、暗号化されたフィールドを早期に返すようになりました。これにより、Kong Gatewayが暗号化解除されたフィールドを認識できるように、キーをインポートする時間が与えられます。 以前は、データベースにキーリング暗号化されたデータフィールドがある場合、Kong Gatewayは`init*`フェーズ（`init`または`init_worker`フェーズ、つまりKongプロセスが開始されたとき）でそれらを復号化しようとし、「no request found」のようなエラーが発生していました。 **Kongプロセスの開始後も、キーをインポートする必要があります。** 
* [キーリング暗号化](/gateway/latest/kong-enterprise/db-encryption/)機能は、アルファ品質状態ではなくなりました。

#### コア

* バランサの再試行で `:authority` 擬似ヘッダが正しく設定されるようになりました。 [\#7725](https://github.com/Kong/kong/pull/7725)。
* バランサーの再作成中にヘルスチェックが停止するようになりました。[\#7549](https://github.com/Kong/kong/pull/7549)。
* 誤った`Accept`ヘッダが予期しないHTTP 500を引き起こす可能性がある問題を修正しました。 [\#7757](https://github.com/Kong/kong/pull/7757)。
* Kongは`Proxy-Authentication`リクエストヘッダと`Proxy-Authenticate`レスポンスヘッダを削除しなくなりました。 [\#7724](https://github.com/Kong/kong/pull/7724)。
* Kong が正規表現とプレフィックスパスの両方を持つルートを正しく並べ替えない問題を修正しました。 [\#7695](https://github.com/Kong/kong/pull/7695)

#### ハイブリッドモード

* データプレーン（DP）のコンフィグスレッドが適切に終了するようにし、半デッドロック状態を防ぎます。 [\#7568](https://github.com/Kong/kong/pull/7568)

##### CLI

* `kong config parse`Goプラグインサーバーが有効になっている場合でもクラッシュしなくなりました。 [\#7589](https://github.com/Kong/kong/pull/7589)。

##### 構成

* 宣言的構成パーサは、不明な外部参照を印刷する際に、より正しいエラーを印刷するようになりました。 [\#7756](https://github.com/Kong/kong/pull/7756)。
* 宣言的構成のYAMLアンカーが適切に処理されます。[\#7748](https://github.com/Kong/kong/pull/7748)。

##### Admin API

* `GET /upstreams/:upstreams/targets/:target`ターゲットウェイトが0の場合、404を返さなくなりました。[\#7758](https://github.com/Kong/kong/pull/7758)

##### PDK

* `kong.response.exit`見つかった場合はカスタマイズされた「Content\-Length」ヘッダーを使用するようになりました。 [\#7828](https://github.com/Kong/kong/pull/7828)。

##### プラグイン

* [ACME](/hub/kong-inc/acme/) \(`acme`\) ワイルドカードドメインのドットはエスケープされています。 [\#7839](https://github.com/Kong/kong/pull/7839)。
* [Prometheus](/hub/kong-inc/prometheus/)（ `prometheus` ） アップストリームのヘルス情報に、これまで欠落していた`subsystem`フィールドが含まれるようになりました。 [\#7802](https://github.com/Kong/kong/pull/7802)。
* [プロキシキャッシュ](/hub/kong-inc/proxy-cache/) \(`proxy-cache`\) プラグインがキャッシュからデータを取得しても返さない場合がある問題を修正しました。 [\#7775](https://github.com/Kong/kong/pull/7775)

