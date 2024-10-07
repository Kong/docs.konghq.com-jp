---
nav_title: "概要"
---
JWTプラグインを使用すると、[RFC 7519](https://tools.ietf.org/html/rfc7519)で指定されたHS256またはRS256で署名されたJSONウェブトークンを含むリクエストを検証できます。

このプラグインを有効にすると、JWTに署名する際に使用する必要があるJWT認証情報（公開鍵と秘密鍵）が各コンシューマに付与されます。その後、以下のいずれかを介してトークンをパスできます。

* クエリ文字列パラメータ
* クッキー
* HTTPリクエストヘッダー

Kongは、トークンの署名が検証された場合にアップストリームサービスへのリクエストをプロキシするか、そうでない場合はリクエストを破棄します。KongはRFC 7519（`exp`と`nbf`）の一部の登録済みクレームの検証を実行することもできます。Kongが複数の異なるトークンを発見した場合、たとえそれらが有効であっても、リクエストはJWTの密輸を防ぐために拒否されます。

プラグインの使用
--------

プラグインを使用するには、まずコンシューマを作成し、それを1つ以上のJWT認証情報（トークンを検証するために使用される公開鍵とプライベート鍵を保持する）に関連付ける必要があります。コンシューマは、最終的なサービスを利用する開発者を表します。

### コンシューマを作成

認証情報を既存の[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)オブジェクトに関連付ける必要があります。コンシューマは多くの認証情報を持つことができます。

{% navtabs %}
{% navtab Kong Admin API %}
コンシューマを作成するには、次のリクエストを実行します。

```bash
curl -d "username=<user123 id="sl-md0000000">&custom_id=<some_custom_id id="sl-md0000000">" http://localhost:8001/consumers/
```

{% endnavtab %}
{% navtab Declarative (YAML) %}
宣言型構成ファイルには、1つ以上のコンシューマが必要です。これらは`consumers:` yamlセクションで作成できます。

```yaml
consumers:
- username: <user123 id="sl-md0000000">
  custom_id: <some_custom_id id="sl-md0000000">
```

{% endnavtab %}
{% endnavtabs %}

どちらの場合も、パラメータは以下のとおりです。

|           パラメータ           |                                    説明                                     |
|---------------------------|---------------------------------------------------------------------------|
| `username`<br> *セミオプション*  | コンシューマのユーザー名。このフィールドか`custom_id`のいずれかを指定する必要があります。                        |
| `custom_id`<br> *セミオプション* | コンシューマを別のデータベースにマップするために使用されるカスタム識別子。このフィールドか`username`のいずれかを指定する必要があります。 |

### JWT認証情報を作成

{% navtabs %}
{% navtab Kong Admin API %}
次のHTTPリクエストを発行することで、新規HS256 JWT認証情報をプロビジョニングできます。

```bash
curl -X POST http://localhost:8001/consumers/<consumer id="sl-md0000000">/jwt -H "Content-Type: application/x-www-form-urlencoded"
```

応答：

```json
HTTP/1.1 201 Created
{
    "algorithm": "HS256",
    "consumer": {
        "id": "789955d4-7cbf-469a-bb64-8cd00bd0f0db"
    },
    "created_at": 1652208453,
    "id": "95d4ee08-c68c-4b69-aa18-e6efad3a4ff0",
    "key": "H8WBDhQlcfjoFmIiYymmkRm1y0A2c5WU",
    "rsa_public_key": null,
    "secret": "n415M6OrVnR4Dr1gyErpta0wSKQ2cMzK",
    "tags": null
}
```

{% endnavtab %}
{% navtab Declarative (YAML) %}
宣言型構成ファイルの`jwt_secrets:` yamlエントリにJWT認証情報を追加できます。

```yaml
jwt_secrets:
- consumer: <consumer id="sl-md0000000">
```

{% endnavtab %}
{% endnavtabs %}

どちらの場合も、フィールド/パラメータは次のように機能します。

|    フィールド/パラメータ    | デフォルト |                                                      説明                                                      |
|-------------------|-------|--------------------------------------------------------------------------------------------------------------|
| `consumer`        |       | 認証情報を関連付けるための[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)エンティティの`id`または`username`プロパティ。 |
| `key`<br> *オプション* |       | 認証情報を識別する一意の文字列。空白の場合、自動生成されます。                                                                              |

{%- if_version gte:3.7.x %}
`algorithm`<br> *オプション* \| `HS256` \| トークンの署名を検証するために使用されるアルゴリズム。`HS256`、 `HS384`、 `HS512`、 `RS256`、 `RS384`、 `RS512`、 `ES256`、`ES384`、`ES512`、 `PS256`、 `PS384`、`PS512`、`EdDSA`。
`rsa_public_key`<br> *オプション* \| \| `algorithm`が`RS*`、 `ES*`、 `PS*`、または`EdDSA*`の場合、トークンの署名を検証するために使用する公開鍵 \(PEM形式\)。
{%- endif_version -%}
{%- if_version lte:3.6.x %}
`algorithm`<br> *オプション* \| `HS256` \| トークンの署名を検証するために使用されるアルゴリズム。`HS256`、 `HS384`、 `HS512`、 `RS256`、 `RS384`、 `RS512`、 `ES256`、`ES384`。
`rsa_public_key`<br> *オプション* \| \| `algorithm`が`RS256`、 `RS384`、 `RS512`、 `ES256`、または`ES384`の場合、トークンの署名を検証するために使用する公開鍵 \(PEM 形式\)。
{%- endif_version %}
`secret`<br> *オプション* \| \| `algorithm`が`HS256`、 `HS384`、または`HS512`の場合、この認証情報のJWTに署名するために使用される秘密鍵。空白の場合、自動生成されます。

{:.important}
> 
> **decKおよび{{site.kic_product_name}}ユーザーへの注意事項：** decKおよび{{site.kic_product_name}}で使用される宣言型構成では、上記の要件とは異なる追加の検証要件が課されます。これは、これらの宣言型構成はデフォルトに依存せず、独自のアルゴリズム固有の要件を実装し、`rsa_public_key`フィールド以外のすべてのフィールドが必須となるためです。
> <br/><br/>
> {% if_version gte:3.7.x %} 常に`key`、`algorithm`、`secret`に入力する必要があります。`RS`、`ES`、`PS`のアルゴリズムを使用する場合は、`secret`のダミー値を使用します。
> {% endif_version %}
> {% if_version lte:3.6.x %} `key`、`algorithm`、`secret`。`RS256`または `ES256`アルゴリズムを使用する場合は、`secret`のダミー値を使用します。
> {% endif_version %}

### JWT認証情報を削除

以下のHTTPリクエストを発行することで、コンシューマのJWT認証情報を削除できます。

```bash
curl -X DELETE http://localhost:8001/consumers/<consumer id="sl-md0000000">/jwt/<jwt-id id="sl-md0000000">
```

応答：

    HTTP/1.1 204 No Content

* `consumer`：認証情報を関連付けるための[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)エンティティの`id`または`username`プロパティ。
* `jwt-id`: JWT認証情報の`id` 。

### JWT認証情報を一覧表示

以下のHTTPリクエストを発行することで、コンシューマのJWT認証情報を一覧表示できます。

```bash
curl -X GET http://localhost:8001/consumers/<consumer id="sl-md0000000">/jwt
```

応答：

    HTTP/1.1 200 OK

* `consumer`：認証情報を一覧表示するための[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)エンティティの`id`または`username`プロパティ。

```json
{
	"data": [
        {
            "algorithm": "HS256",
            "consumer": {
                "id": "789955d4-7cbf-469a-bb64-8cd00bd0f0db"
            },
            "created_at": 1652208453,
            "id": "95d4ee08-c68c-4b69-aa18-e6efad3a4ff0",
            "key": "H8WBDhQlcfjoFmIiYymmkRm1y0A2c5WU",
            "rsa_public_key": null,
            "secret": "n415M6OrVnR4Dr1gyErpta0wSKQ2cMzK",
            "tags": null
        }
    ],
    "next": null
}
```

### 秘密鍵を使用してJWTを作成（HS256）

コンシューマの認証情報を確立し、`HS256`を使用して署名することを推定した場合、以下のようにJWTを作成する必要があります（[RFC 7519](https://tools.ietf.org/html/rfc7519)に従って）。

まず、以下のヘッダーである必要があります。

```json
{
    "typ": "JWT",
    "alg": "HS256"
}
```

次に、クレームには、構成済みのクレーム（`config.key_claim_name`から）の秘密の`key`が含まれている **必要があります** 。デフォルトで、そのクレームは`iss`（発行者フィールド）です。その値を、以前に作成した認証情報の`key`に設定します。クレームには他の値が含まれている場合があります。Kong `0.13.1`以降、クレームはJWTペイロードとヘッダーの順序で両方が検索されます。

```json
{
    "iss": "YJdmaDvVTJxtcWRCvkMikc8oELgAVNcz"
}
```

[https://jwt.io](https://jwt.io)で、ヘッダー（HS256）、クレーム（`iss`など）、この`key``C50k0bcahDhLNhLKSUBSR1OMiFGzNZ7X`に関連付けられた`secret`を使用したJWTデバッガーを使用した場合、次のJWTトークンが生成されます。

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJZSmRtYUR2VlRKeHRjV1JDdmtNaWtjOG9FTGdBVk5jeiIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.WuLdHyvZGj2UAsnBl6YF9A4NqGQpaDftHjX18ooK8YY

### JWTでリクエストを送信

JWTは、`config.header_names`（デフォルトで`Authorization`を含む）で構成された場合、Kongによりヘッダーとしてリクエストに含まれるようになりました。

```bash
curl http://localhost:8000/<route-path id="sl-md0000000"> \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJZSmRtYUR2VlRKeHRjV1JDdmtNaWtjOG9FTGdBVk5jeiIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.WuLdHyvZGj2UAsnBl6YF9A4NqGQpaDftHjX18ooK8YY'
```

`config.uri_param_names` （デフォルトで`jwt`を含む）で構成されている場合はクエリ文字列パラメータとして。

```bash
curl http://localhost:8000/<route-path id="sl-md0000000">?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJZSmRtYUR2VlRKeHRjV1JDdmtNaWtjOG9FTGdBVk5jeiIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.WuLdHyvZGj2UAsnBl6YF9A4NqGQpaDftHjX18ooK8YY
```

または、`config.cookie_names`（デフォルトでは無効）で名前が構成された場合はクッキーとして。

```bash
curl --cookie jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJZSmRtYUR2VlRKeHRjV1JDdmtNaWtjOG9FTGdBVk5jeiIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.WuLdHyvZGj2UAsnBl6YF9A4NqGQpaDftHjX18ooK8YY http://localhost:8000/<route-path id="sl-md0000000">
```

gRPCリクエストでは、ヘッダーにJWTを含めることができます。

```bash
grpcurl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJZSmRtYUR2VlRKeHRjV1JDdmtNaWtjOG9FTGdBVk5jeiIsImV4cCI6MTQ0MjQzMDA1NCwibmJmIjoxNDQyNDI2NDU0LCJpYXQiOjE0NDI0MjY0NTR9.WuLdHyvZGj2UAsnBl6YF9A4NqGQpaDftHjX18ooK8YY' ...
```

リクエストはKongによって検査され、その動作はJWTの有効性に依存します。

|           リクエスト           | アップストリームサービスにプロキシされた |   応答ステータスコード   |
|---------------------------|----------------------|----------------|
| JWTなし                     | いいえ                  | 401            |
| `iss`クレームが欠落しているか無効       | いいえ                  | 401            |
| 無効な署名                     | いいえ                  | 401            |
| 有効な署名                     | はい                   | アップストリームサービスから |
| 有効な署名、無効な検証済みクレーム *オプション* | いいえ                  | 401            |

{:.note}
> 
> **注：** JWTが有効で、アップストリームサービスにプロキシされている場合、 {{site.base_gateway}}はコンシューマを特定するヘッダーを追加する以外、リクエストには変更を行いません。JWTはアップストリームサービスに転送され、その有効性が保証されます。JWTをbase64でデコードし、それらを活用するのはご使用のサービスの役割となりました。

### （オプション）検証済みのクレーム

Kongは、 [RFC 7519](https://tools.ietf.org/html/rfc7519)で定義されているように、登録済みクレームの検証も実行できます。クレームの検証を実行するには、クレームを`config.claims_to_verify`プロパティに追加します。

既存のJWTプラグインにパッチを適用できます。

```bash
# This adds verification for both nbf and exp claims:
curl -X PATCH http://localhost:8001/plugins/{jwt plugin id} \
  --data "config.claims_to_verify=exp,nbf"
```

サポートされているクレーム：

| クレーム名 |              検証              |
|-------|------------------------------|
| `exp` | JWTを処理に受け入れてはならない有効期限を指定します。 |
| `nbf` | JWTを処理に受け入れることができない時間を特定します。 |

### （オプション）Base64でエンコードされたシークレット

シークレットにバイナリデータが含まれている場合は、KongでBase64でエンコードして保存できます。このオプションは、プラグインの構成で有効にします。

既存のルートにパッチを適用できます。

```bash
curl -X PATCH http://localhost:8001/routes/<route-id id="sl-md0000000">/plugins/<jwt-plugin-id id="sl-md0000000"> \
  --data "config.secret_is_base64=true"
```

次に、コンシューマのシークレットをbase64でエンコードします。

```bash
# secret is: "blob data"
curl -X POST http://localhost:8001/consumers/<consumer id="sl-md0000000">/jwt \
  --data "secret=YmxvYiBkYXRh"
```

そして、元のシークレット \(「blobデータ」\) を使用してJWTに署名します。

### 公開鍵/プライベート鍵（RS256 または ES256）を使用してJWTを作成

RS256またはES256を使用してJWTを検証する場合は、JWT認証情報を作成する際に、`RS256`または`ES256`を`algorithm`として選択し、公開鍵を`rsa_public_key`フィールド（ES256署名トークンを含む）に明示的にアップロードします。例：

```bash
curl -X POST http://localhost:8001/consumers/<consumer id="sl-md0000000">/jwt \
  -F "rsa_public_key=@/path/to/public_key.pem" \
```

応答：

```json
HTTP/1.1 201 Created
{
    "created_at": 1442426001000,
    "id": "bcbfb45d-e391-42bf-c2ed-94e32946753a",
    "key": "a36c3049b36249a3c9f8891cb127243c",
    "rsa_public_key": "-----BEGIN PUBLIC KEY----- ...",
    "consumer": {
      "id": "8a21c4fa-e65e-4258-8673-540e85e67b33"
    }
}
```

署名を作成するときは、ヘッダーが以下であることを確実にしてください。

```json
{
    "typ": "JWT",
    "alg": "RS256"
}
```

次に、クレームは、構成済みクレーム（`config.key_claim_name`から）のシークレットの`key`フィールド（これはトークンの生成に使用されるプライベート鍵では **なく** 、この認証情報の識別子です）を含む **必要があります** 。そのクレームはデフォルトでは`iss` \(発行者フィールド\) です。その値を、以前に作成した認証情報の`key`に設定します。クレームには他の値が含まれている場合があります。Kong バージョン`0.13.1`以降、クレームはJWTペイロードとヘッダーの順序で両方が検索されます。

```json
{
    "iss": "a36c3049b36249a3c9f8891cb127243c"
}
```

次に、プライベート鍵を使用して署名を作成します。[https://jwt.io](https://jwt.io)でJWTデバッガーを使用して、適切なヘッダー（RS256）、クレーム（ `iss`など）、関連付けられた公開鍵を設定します。次に、結果の値を`Authorization`ヘッダーに追加します。例：

```bash
curl http://localhost:8000/<route-path id="sl-md0000000"> \
  -H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiIxM2Q1ODE0NTcyZTc0YTIyYjFhOWEwMDJmMmQxN2MzNyJ9.uNPTnDZXVShFYUSiii78Q-IAfhnc2ExjarZr_WVhGrHHBLweOBJxGJlAKZQEKE4rVd7D6hCtWSkvAAOu7BU34OnlxtQqB8ArGX58xhpIqHtFUkj882JQ9QD6_v2S2Ad-EmEx5402ge71VWEJ0-jyH2WvfxZ_pD90n5AG5rAbYNAIlm2Ew78q4w4GVSivpletUhcv31-U3GROsa7dl8rYMqx6gyo9oIIDcGoMh3bu8su5kQc5SQBFp1CcA5H8sHGfYs-Et5rCU2A6yKbyXtpHrd1Y9oMrZpEfQdgpLae0AfWRf6JutA9SPhst9-5rn4o3cdUmto_TBGqHsFmVyob8VQ'
```

### 公開鍵/プライベート鍵の生成

完全に新しい公開鍵/プライベート鍵のペアを作成するには、次のコマンドを実行します。

```bash
openssl genrsa -out private.pem 2048
```

このプライベート鍵は秘密にしておく必要があります。プライベート鍵に対応する公開鍵を生成するには、次のコマンドを実行します。

```bash
openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```

上記のコマンドを実行すると、公開鍵は`public.pem`に書き込まれますが、プライベート鍵は`private.pem`に書き込まれます。

### Auth0でのJWTプラグインの使用

[Auth0](https://auth0.com/)は一般的な認証用のソリューションで、JWTに大きく依存します。Auth0はRS256に依存しており、base64エンコードは行わず、トークンの署名に使用される公開鍵を公的にホストします。以下の例では、アカウント名が`{COMPANYNAME}`として参照されています。

まず、そのサービスを使用するサービスとルートを作成します。

*注: Auth0はbase64でエンコードされたシークレットを使用しません。* 

サービスを作成します。

```bash
curl -i -f -X POST http://localhost:8001/services \
  --data "name=example-service" \
  --data "url=http://httpbin.org"
```

次にルートを作成します。

```bash
curl -i -f -X POST http://localhost:8001/routes \
  --data "service.id=<example-service-id id="sl-md0000000">" \
  --data "paths[]=/example_path"
```

#### JWTプラグインを追加

ルートにプラグインを追加します。

```bash
curl -X POST http://localhost:8001/routes/<route-id id="sl-md0000000">/plugins \
  --data "name=jwt"
```

Auth0アカウントのX509証明書をダウンロードします。

```bash
curl -o <companyname id="sl-md0000000">.pem https://<companyname id="sl-md0000000">.<region-id id="sl-md0000000">.auth0.com/pem
```

X509 証明書から公開鍵を抽出します。

```bash
openssl x509 -pubkey -noout -in <companyname id="sl-md0000000">.pem > pubkey.pem
```

Auth0公開鍵を持つコンシューマを作成します。

```bash
curl -i -X POST http://localhost:8001/consumers \
  --data "username=<username id="sl-md0000000">" \
  --data "custom_id=<custom_id id="sl-md0000000">"
curl -i -X POST http://localhost:8001/consumers/<consumer id="sl-md0000000">/jwt \
  -F "algorithm=RS256" \
  -F "rsa_public_key=@./pubkey.pem" \
  -F "key=https://<companyname id="sl-md0000000">.auth0.com/" # the `iss` field
```

JWTプラグインはデフォルトで`key_claim_name`をトークンの`iss`フィールドに対して検証します。Auth0によって発行されたキーには、`iss`フィールドが`http://<companyname id="sl-md0000000">.auth0.com/`に設定されます。コンシューマを作成する場合、[jwt.io](https://jwt.io)を使用して、`key`パラメータの`iss`フィールドを有効にできます。

リクエストを送信します。Auth0によって署名されたトークンのみが機能します。

```bash
curl -i http://localhost:8000 \
  -H "Host:example.com" \
  -H "Authorization:Bearer <token id="sl-md0000000">"
```

### アップストリームヘッダー

JWTが有効でコンシューマが認証されている場合、プラグインはアップストリームサービスにプロキシする前に一部のヘッダーを追加して、以下のコードでコンシューマを識別できるようにします。

* `X-Consumer-ID`、KongのコンシューマのID.
* `X-Consumer-Custom-ID`、コンシューマの`custom_id`（設定されている場合）
* `X-Consumer-Username`、コンシューマの`username`（設定されている場合）
* `X-Credential-Identifier`、認証情報の識別子 \(設定されている場合\)。
* `X-Anonymous-Consumer`、認証に失敗した場合は`true`に設定され、代わりに`anonymous`コンシューマが設定されます。

この情報を利用して、追加のロジックを実装することができます。`X-Consumer-ID`値を使用してKong Admin APIのクエリを実行し、コンシューマについての詳細な情報を取得できます。

### JWTにページ番号を付ける

以下のリクエストを使用して、すべてのコンシューマのJWTにページ番号を付けることができます

```bash
curl -X GET http://localhost:8001/jwts
```

応答：

```json
{
	"next": null,
	"data": [
		{
			"id": "0701ad83-949c-423f-b553-091d5a6bae52",
			"secret": "C50k0bcahDhLNhLKSUBSR1OMiFGzNZ7X",
			"key": "YJdmaDvVTJxtcWRCvkMikc8oELgAVNcz",
			"tags": null,
			"rsa_public_key": null,
			"consumer": {
				"id": "8a21c1fa-e65e-4558-8673-540e85e67b33"
			},
			"algorithm": "HS256",
			"created_at": 1664462115
		},
		{
			"id": "e14d775e-3b52-45cc-be9e-b39cdb3f7ebf",
			"secret": "gFrOZAYWH1osQ6ivkesT4qqH16DHpKu0",
			"key": "7TrSUEFcEP7KZb2QrGT9IQXcEWrmszC8",
			"tags": null,
			"rsa_public_key": null,
			"consumer": {
				"id": "8a21c1fa-e65e-4558-8673-540e85e67b33"
			},
			"algorithm": "HS256",
			"created_at": 1664398496
		}
	]
}
```

別のパスを使用して、コンシューマ別のリストをフィルタリングできます。

```bash
curl -X GET http://localhost:8001/consumers/<username-or-id id="sl-md0000000">/jwt
```

応答：

```json
{
    "next": null,
    "data": [
        {
            "created_at": 1511389527000,
            "id": "0dfc969b-02be-42ae-9d98-e04ed1c05850",
            "algorithm": "ES256",
            "key": "vcc1NlsPfK3N6uU03YdNrDZhzmFF4S19",
            "secret": "b65Rs6wvnWPYaCEypNU7FnMOZ4lfMGM7",
            "consumer": {
               "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" 
              }
        }
    ]
}
```

`username or id`: JWTを一覧表示する必要があるコンシューマのユーザー名またはID。

### JWTに関連付けられたコンシューマを取得

次のリクエストを使用して、JWTに関連付けられた[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を取得します。

```bash
curl -X GET http://localhost:8001/jwts/<key-or-id id="sl-md0000000">/consumer
```

応答：

```json
{
	"id": "8a21c1fa-e65e-4558-8673-540e85e67b33",
	"username_lower": "username",
	"custom_id": "123412312312312312312312",
	"type": 0,
	"created_at": 1664398410,
	"username": "Username",
	"tags": null
}
```

`key or id`：関連付けられた[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を取得するためのJWTの`id`または`key`プロパティ。

