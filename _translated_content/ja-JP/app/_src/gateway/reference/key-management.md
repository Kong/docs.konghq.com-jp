---
title: "Kong Gatewayでのキーとキーセット管理"
content_type: "reference"
badge: "free"
---
このページでは、{{site.base_gateway}}で非対象キーとキーセットを管理する{{site.base_gateway}}の機能について説明します。

一部の操作では、公開鍵と秘密鍵へのアクセスが必要です。このドキュメントでは、{{site.base_gateway}}を使用してこれらのキーへのアクセスを許可する方法についても説明します。

{% if_version gte:3.2.x %}
> 
> この機能は、{{site.konnect_short_name}}とKong Managerの両方で利用できます。

* **Konnect** では、 **Gateway Manager** エンティティとしてキーを管理できます。
* Kong Managerでは、 **API Gateway** のドロップダウンから利用できます。{% endif_version %} {:.note}

### ユースケース

一部の{{site.base_gateway}}プラグインは、JSON Webキーを構成するためのカスタムエンドポイントを提供します。新しい汎用エンドポイントは、各プラグインのカスタムエンドポイントを置き換えます。次の表に、新しいエンドポイントをサポートするプラグインを示します。

|                      プラグイン                      | サポートされているキー/キーセット |
|-------------------------------------------------|-------------------|
| [OpenID Connect](/hub/kong-inc/openid-connect/) | なし                |
| [JWT Signer](/hub/kong-inc/jwt-signer/)         | なし                |
| [JWT](/hub/kong-inc/jwt/)                       | なし                |
| [JWE Decrypt](/hub/kong-inc/jwe-decrypt/)       | あり                |

### キーエンドポイント

汎用 `Keys`エンドポイントでは、公開鍵でも秘密鍵でも、非対称鍵をJWKまたはPEMとして保存できます。キーを識別するために、設定可能な`kid`文字列が必要です。`kid`属性は、トークンの検証または復号化に使用するキーを識別する一般的な方法ですが、キーを識別する必要がある他のシナリオでも使用できます。

### キーセットのエンドポイント

JSON Webキーセットに1つまたは複数のキーを割り当てることができます。これは、特定のアプリケーションやサービスに使用する複数のキーを論理的にグループ化するのに役立ちます。キーセットは、キーを検索する場所をプラグインに指示したり、プラグインを *一部の* キーのみに制限するスコープメカニズムを備えているため、キーをプラグインに公開する方法としても推奨されます。

キーセットを使用してプラグインを設定する方法の詳細については、次のプラグインのドキュメントを参照してください。

* [OpenID Connect](/hub/kong-inc/openid-connect/)
* [JWT Signer](/hub/kong-inc/jwt-signer/)
* [JWT](/hub/kong-inc/jwt/)
* [JWE Decrypt](/hub/kong-inc/jwe-decrypt/)

{:.note}
> 
> **注：** キーセットを削除すると、関連するすべてのキーが削除されます。

### キー形式

現在、以下の2つの一般的な形式がサポートされています。

* JWK
* PEM

どちらの形式も、公開鍵指数や秘密鍵指数など同じ基本情報を伝達しますが、追加のメタ情報を指定できる場合があります。たとえば、JWK形式はPEMよりも多くの情報を伝達します。いわば、同じキーでありながら、1つのキーペアを複数の異なる形式（JWKまたはPEM）で表現できます。

### JWK形式でキーを作成し、キーセットと関連付けます。

以下のように、キーセットを作成します。

```bash
curl -i -X PUT http://HOSTNAME:8001/key-sets  \
  --data name=my-set
```

結果：

```json
{
  "created_at": 1669029622,
  "id": "2033cb3d-ef3b-4f6d-8395-bc3c2d5a0e4f",
  "name": "my-set",
  "tags": null,
  "updated_at": 1669029622
  }
```

キーを作成し、キーセットに関連付けます。

```bash
curl -i -X POST http://HOSTNAME:8001/keys  \
  --data name=my-first-jwk \
  --data jwk='{"kty":"RSA","kid":"42","use":"enc","n":"pjdss8ZaDfEH6K6U7GeW2nxDqR4IP049fk1fK0lndimbMMVBdPv_hSpm8T8EtBDxrUdi1OHZfMhUixGaut-3nQ4GG9nM249oxhCtxqqNvEXrmQRGqczyLxuh-fKn9Fg--hS9UpazHpfVAFnB5aCfXoNhPuI8oByyFKMKaOVgHNqP5NBEqabiLftZD3W_lsFCPGuzr4Vp0YS7zS2hDYScC2oOMu4rGU1LcMZf39p3153Cq7bS2Xh6Y-vw5pwzFYZdjQxDn8x8BG3fJ6j8TGLXQsbKH1218_HcUJRvMwdpbUQG5nvA2GXVqLqdwp054Lzk9_B_f1lVrmOKuHjTNHq48w","e":"AQAB","d":"ksDmucdMJXkFGZxiomNHnroOZxe8AmDLDGO1vhs-POa5PZM7mtUPonxwjVmthmpbZzla-kg55OFfO7YcXhg-Hm2OWTKwm73_rLh3JavaHjvBqsVKuorX3V3RYkSro6HyYIzFJ1Ek7sLxbjDRcDOj4ievSX0oN9l-JZhaDYlPlci5uJsoqro_YrE0PRRWVhtGynd-_aWgQv1YzkfZuMD-hJtDi1Im2humOWxA4eZrFs9eG-whXcOvaSwO4sSGbS99ecQZHM2TcdXeAs1PvjVgQ_dKnZlGN3lTWoWfQP55Z7Tgt8Nf1q4ZAKd-NlMe-7iqCFfsnFwXjSiaOa2CRGZn-Q","p":"4A5nU4ahEww7B65yuzmGeCUUi8ikWzv1C81pSyUKvKzu8CX41hp9J6oRaLGesKImYiuVQK47FhZ--wwfpRwHvSxtNU9qXb8ewo-BvadyO1eVrIk4tNV543QlSe7pQAoJGkxCia5rfznAE3InKF4JvIlchyqs0RQ8wx7lULqwnn0","q":"ven83GM6SfrmO-TBHbjTk6JhP_3CMsIvmSdo4KrbQNvp4vHO3w1_0zJ3URkmkYGhz2tgPlfd7v1l2I6QkIh4Bumdj6FyFZEBpxjE4MpfdNVcNINvVj87cLyTRmIcaGxmfylY7QErP8GFA-k4UoH_eQmGKGK44TRzYj5hZYGWIC8","dp":"lmmU_AG5SGxBhJqb8wxfNXDPJjf__i92BgJT2Vp4pskBbr5PGoyV0HbfUQVMnw977RONEurkR6O6gxZUeCclGt4kQlGZ-m0_XSWx13v9t9DIbheAtgVJ2mQyVDvK4m7aRYlEceFh0PsX8vYDS5o1txgPwb3oXkPTtrmbAGMUBpE","dq":"mxRTU3QDyR2EnCv0Nl0TCF90oliJGAHR9HJmBe__EjuCBbwHfcT8OG3hWOv8vpzokQPRl5cQt3NckzX3fs6xlJN4Ai2Hh2zduKFVQ2p-AF2p6Yfahscjtq-GY9cB85NxLy2IXCC0PF--Sq9LOrTE9QV988SJy_yUrAjcZ5MmECk","qi":"ldHXIrEmMZVaNwGzDF9WG8sHj2mOZmQpw9yrjLK9hAsmsNr5LTyqWAqJIYZSwPTYWhY4nu2O0EY9G9uYiqewXfCKw_UngrJt8Xwfq1Zruz0YY869zPN4GiE9-9rzdZB33RBw8kIOquY3MK74FMwCihYx_LiU2YTHkaoJ3ncvtvg"}' \
  --data kid=42 \
  --data set.name=my-set
```

結果：

```json
{
  "id":"92a245af-8cb6-4175-b3a9-9383cbb9848f",
  "jwk":
          { kty":"RSA",
            "kid":"42",
            "use":"enc",
            ..."},
  "created_at":1669029250,
  "updated_at":1669029250,
  "name":"my-first-jwk",
  "tags":null,
  "set":
          { "id":"cb5b5df8-0161-4fdf-a2ce-cefc9481d5f9"},
  "kid":"42",
  "pem":null
  }
```

### PEM形式でキーを作成し、キーセットと関連付ける

以下のように、キーセットを作成します。

```bash
curl -i -X PUT http://HOSTNAME:8001/key-sets  \
  --data name=my-other-set
```

結果：

```json
{
  "created_at": 1669029622,
  "id": "2033cb3d-ef3b-4f6d-8395-bc3c2d5a0e4f",
  "name": "my-other-set",
  "tags": null,
  "updated_at": 1669029622
  }
```

PEMでエンコードされたキーを作成し、キーセットと関連付けます。

```bash
curl -i -X POST http://HOSTNAME:8001/keys  \
  --data name=my-first-pem-key \
  --data pem.private_key=@path/to/private_key.pem \
  --data pem.public_key=@path/to/public_key.pem \
  --data kid=23 \
  --data set.name=my-other-set
```

結果：

```json
{
  "id":"92a245af-8cb6-4175-b3a9-9383cbb9848f",
  "jwk": null
  "pem": {
          "public_key": "----BEGIN...",
          "private_key": "----BEGIN...."
  }
  "created_at":1669029250,
  "updated_at":1669029250,
  "name":"my-first-pem-key",
  "tags":null,
  "set":
          { "id":"cb5b5df8-0161-4fdf-a2ce-cefc9481d5f9"},
  "kid":"23",
  }
```

