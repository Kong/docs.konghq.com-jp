---
title: "イベントフックのリファレンス"
badge: "enterprise"
---
{% include_cached /md/enterprise/event-hooks-intro.md %}

{:.note}
> 
> **注:** 現時点でイベントフックは Konnect では動作しません。

すべてのイベントフックを一覧表示する
------------------

**エンドポイント** 
<div class="endpoint get">/event\-hooks</div> 

**応答** 

```json
{
    "data": [
        {
            "config": {
                "body": null,
                "body_format": true,
                "headers": {
                    "content-type": "application/json"
                },
                "headers_format": false,
                "method": "POST",
                "payload": {
                    "text": "{% raw %}Admin account \`{{ entity.username }}\` {{ operation}}d; email address set to \`{{ entity.email }}\`{% endraw %}"
                },
                "payload_format": true,
                "secret": null,
                "ssl_verify": false,
                "url": "https://hooks.slack.com/services/foo/bar/baz"
            },
            "created_at": 1627588552,
            "event": "admins",
            "handler": "webhook-custom",
            "id": "937df175-3db2-4e6d-8aa1-d95c94a76089",
            "on_change": null,
            "snooze": null,
            "source": "crud"
        },
        {
            "config": {
                "headers": {},
                "secret": null,
                "ssl_verify": false,
                "url": "https://webhook.site/a1b2c3-d4e5-g6h7-i8j9-k1l2m3n4o5p6"
            },
            "created_at": 1627581575,
            "event": "consumers",
            "handler": "webhook",
            "id": "c57340ab-9fed-40fd-bb7e-1cef8d37c2df",
            "on_change": null,
            "snooze": null,
            "source": "crud"
        },
        {
            "config": {
                "functions": [
                    "return function (data, event, source, pid)\n  local user = data.entity.username\n  error(\"Event hook on consumer \" .. user .. \"\")\nend\n"
                ]
            },
            "created_at": 1627595513,
            "event": "consumers",
            "handler": "lambda",
            "id": "c9fdd58d-5416-4d3a-9467-51e5cfe4ca0e",
            "on_change": null,
            "snooze": null,
            "source": "crud"
        }
    ],
    "next": null
}
```

すべてのソースを一覧表示する
--------------

ソースは、イベントフックをトリガーするアクションです。`/sources`のJSON出力は次のパターンに従います。

* 1レベル = ソース。このソースはイベントフックをトリガーするアクションです。
* 第2のレベル = イベントフックがイベントをリッスンする Kongエンティティのイベント。
* 3 番目のレベル = Webhook カスタムペイロードで使用可能なテンプレートパラメータ。

たとえば、次の例では、`balancer` がソース、`health` がイベント、`upstream_id`、`ip`、`port`、`hostname`、`health` が使用可能なテンプレートパラメーターです。

**エンドポイント** 
<div class="endpoint get">/event\-hooks/sources/</div> 

**応答** 

```json
{
    "data": {
        "balancer": {
            "health": {
                "fields": [
                    "upstream_id",
                    "ip",
                    "port",
                    "hostname",
                    "health"
                ]
            }
        },
        "crud": {
            "acls": {
                "fields": [
                    "operation",
                    "entity",
                    "old_entity",
                    "schema"
                ]
            },
            . . .

        "rate-limiting-advanced": {
            "rate-limit-exceeded": {
                "description": "Run an event when a rate limit has been exceeded",
                "fields": [
                    "consumer",
                    "ip",
                    "service",
                    "rate",
                    "limit",
                    "window"
                ],
                "unique": [
                    "consumer",
                    "ip",
                    "service"
                ]
            }
        }
    }
}
```

{:.note}
> 
> **注：** 全文を掲載するには長すぎるため、レスポンスは短縮されています。
> レスポンスの中央にある省略記号は、欠落しているコンテンツを表します。

ソースのすべてのイベントを一覧表示
-----------------

イベントは、イベント フックがイベントをリッスンする Kong エンティティです。このエンドポイントを使用すると、
特定のソースに関連付けられたすべてのイベントを一覧表示できます。
<div class="endpoint get">/event\-hooks/sources/\{source\}/</div> 

次のレスポンスは、`dao:crud` ソースのすべてのイベントを一覧表示します。

**応答** 

```json
{
    "data": {
        "create": {
            "fields": [
                "operation",
                "entity",
                "old_entity",
                "schema"
            ]
        },
        "delete": {
            "fields": [
                "operation",
                "entity",
                "old_entity",
                "schema"
            ]
        },
        "update": {
            "fields": [
                "operation",
                "entity",
                "old_entity",
                "schema"
            ]
        }
    }
}
```

webhookを追加する
------------

**エンドポイント** 
<div class="endpoint post">/event\-hooks</div> 

**リクエストボディ** 

|                              属性 |                                                                  説明                                                                  |
|--------------------------------:|--------------------------------------------------------------------------------------------------------------------------------------|
|             `event`<br> *オプション* | イベントフックがイベントをリッスンするKongエンティティについて説明する文字列。                                                                                            |
|                  `handler` *必須* | webhook、webhook\-custom、log、lambdaの4つのハンドラオプションのいずれかを説明する文字列。                                                                       |
|                   `source` *必須* | イベントフックをトリガーするアクションを説明する文字列。                                                                                                         |
|            `snooze`<br> *オプション* | イベントトリガーの遅延時間を秒単位で記述する任意の整数で、統合のスパムを回避する目的があります。                                                                                     |
|         `on_change`<br> *オプション* | ペイロードのキー部分に変更があった場合に、イベントをトリガーするかどうかを示すオプションのブール値。                                                                                   |
|               `config.url` *必須* | イベントデータをペイロードとしてJSON POSTリクエストが行われるURL。                                                                                              |
|    `config.headers`<br> *オプション* | Webhookリクエストで送信する追加のHTTPヘッダーを定義するオブジェクト（例：`{"X-Custom-Header": "My Value"}` ）                                                        |
|     `config.secret`<br> *オプション* | リモート検証のためにリモート webhook への署名で使用されるオプションの文字列。設定すると、Kong はイベントフックのボディを HMAC\-SHA1 で署名し、それをリモート エンドポイントへのヘッダー `x-kong-signature` に含めます。 |
| `config.ssl_verify`<br> *オプション* | イベントフックが送信されるリモートHTTPSサーバーのSSL証明書を検証するかどうかを示すブール値。デフォルトは `false` です。                                                                 |

**応答** 

```json
{
    "config": {
        "headers": {},
        "secret": null,
        "ssl_verify": false,
        "url": "https://webhook.site/a1b2c3-d4e5-g6h7-i8j9-k1l2m3n4o5p6"
    },
    "created_at": 1627581575,
    "event": "consumers",
    "handler": "webhook",
    "id": "c57340ab-9fed-40fd-bb7e-1cef8d37c2df",
    "on_change": null,
    "snooze": null,
    "source": "crud"
}
```

カスタムwebhookの追加
--------------

**エンドポイント** 
<div class="endpoint post">/event\-hooks</div> 

**リクエストボディ** 

|                                  属性 |                                                                                                         説明                                                                                                         |
|------------------------------------:|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                 `event`<br> *オプション* | イベントフックがイベントをリッスンするKongエンティティについて説明する文字列。                                                                                                                                                                          |
|                      `handler` *必須* | webhook、webhook\-custom、log、lambdaの4つのハンドラオプションのいずれかを説明する文字列。                                                                                                                                                     |
|                       `source` *必須* | イベントフックをトリガーするアクションを説明する文字列。                                                                                                                                                                                       |
|                `snooze`<br> *オプション* | イベントトリガーの遅延時間を秒単位で記述する任意の整数で、統合のスパムを回避する目的があります。                                                                                                                                                                   |
|             `on_change`<br> *オプション* | ペイロードのキー部分に変更があった場合に、イベントをトリガーするかどうかを示すオプションのブール値。                                                                                                                                                                 |
|                   `config.url` *必須* | イベントデータをペイロードとしてJSON POSTリクエストが行われるURL。                                                                                                                                                                            |
|                `config.method` *必須* | カスタムwebhookの作成に使用されるHTTPメソッド。                                                                                                                                                                                      |
|        `config.payload`<br> *オプション* | 構成可能なペイロード本文を記述するキーと値のペアを含むオブジェクト。テンプレートをサポートします。使用可能なテンプレートパラメータの完全なリストは、 `/sources` API出力の`fields` JSONオブジェクトにあります。                                                                                              |
| `config.payload_format`<br> *オプション* | restyテンプレート法で`config.payload`を書式設定するかどうかを示すオプションのブール値（デフォルトは`true`）`false`に設定すると、ペイロードは生のオブジェクトとして送信されます。                                                                                                          |
|           `config.body`<br> *オプション* | リモートリクエストで送信されるオプションの文字列。                                                                                                                                                                                          |
|    `config.body_format`<br> *オプション* | restyテンプレート法で`config.body`を書式設定するかどうかを示すオプションのブール値（デフォルトは`true`）`false`に設定すると、ボディは生のオブジェクトとして送信されます。特定の`source`に定義されている使用可能なすべてのパラメータを確認するには、`/event-hooks/source`エンドポイントによって表示されるソースフィールドを確認します。                  |
|                    `config.headers` | Webhookリクエストで送信する追加のHTTPヘッダーを定義するオブジェクト（例：`{"Content-type": "application/json", "X-Custom-Header": "My Value"}` ）                                                                                                  |
| `config.headers_format`<br> *オプション* | restyテンプレート法で`config.headers`を書式設定するかどうかを示すオプションのブール値（デフォルトは`false`）`true`に設定すると、 `config.headers`値はテンプレートとして扱われます。特定の`source`に定義されている使用可能なすべてのパラメータを確認するには、 `/event-hooks/sources`エンドポイントによって表示されるソースフィールドを確認します。 |
|         `config.secret`<br> *オプション* | リモート検証のためにリモート webhook への署名で使用されるオプションの文字列。設定すると、Kong はイベントフックのボディを HMAC\-SHA1 で署名し、それをリモート エンドポイントへのヘッダー `x-kong-signature` に含めます。                                                                               |
|     `config.ssl_verify`<br> *オプション* | イベントフックが送信されるリモートHTTPSサーバーのSSL証明書を検証するかどうかを示すブール値。デフォルト値は`false`です。                                                                                                                                                |

**応答** 

```json
{
    "config": {
        "body": null,
        "body_format": true,
        "headers": {
            "content-type": "application/json"
        },
        "headers_format": false,
        "method": "POST",
        "payload": {
            "text": "Admin account `{{ entity.username }}` {{ operation }}d; email address set to `{{ entity.email }}`"
        },
        "payload_format": true,
        "secret": null,
        "ssl_verify": false,
        "url": "https://hooks.slack.com/services/foo/bar/baz"
    },
    "created_at": 1627588552,
    "event": "admins",
    "handler": "webhook-custom",
    "id": "937df175-3db2-4e6d-8aa1-d95c94a76089",
    "on_change": null,
    "snooze": null,
    "source": "crud"
}
```

ログイベントフックを追加する
--------------

**エンドポイント** 
<div class="endpoint post">/event\-hooks</div> 

**リクエストボディ** 

|                      属性 |                               説明                               |
|------------------------:|----------------------------------------------------------------|
|     `event`<br> *オプション* | イベントフックがイベントをリッスンするKongエンティティについて説明する文字列。                      |
|          `handler` *必須* | webhook、webhook\-custom、log、lambdaの4つのハンドラオプションのいずれかを説明する文字列。 |
|           `source` *必須* | イベントフックをトリガーするアクションを説明する文字列。                                   |
|    `snooze`<br> *オプション* | イベントトリガーの遅延時間を秒単位で記述する任意の整数で、統合のスパムを回避する目的があります。               |
| `on_change`<br> *オプション* | ペイロードのキー部分に変更があった場合に、イベントをトリガーするかどうかを示すオプションのブール値。             |

**応答** 

```json
{
  "config": {},
  "on_change": null,
  "created_at": 1627346155,
  "snooze": null,
  "id": "13a16f91-68b6-4384-97f7-d02763a551ac",
  "handler": "log",
  "source": "crud",
  "event": "routes"
}
```

Lambdaイベントフックの追加
----------------

**エンドポイント** 
<div class="endpoint post">/event\-hooks</div> 

**リクエストボディ** 

|                      属性 |                               説明                               |
|------------------------:|----------------------------------------------------------------|
|     `event`<br> *オプション* | イベントフックがイベントをリッスンするKongエンティティについて説明する文字列。                      |
|          `handler` *必須* | webhook、webhook\-custom、log、lambdaの4つのハンドラオプションのいずれかを説明する文字列。 |
|           `source` *必須* | イベントフックをトリガーするアクションを説明する文字列。                                   |
|    `snooze`<br> *オプション* | イベントトリガーの遅延時間を秒単位で記述する任意の整数で、統合のスパムを回避する目的があります。               |
| `on_change`<br> *オプション* | ペイロードのキー部分に変更があった場合に、イベントをトリガーするかどうかを示すオプションのブール値。             |
| `config.functions` *必須* | イベントフックで実行する Lua コード関数の配列。                                     |

**応答** 

```json
{
    "config": {
        "functions": [
            "return function (data, event, source, pid)\n  local user = data.entity.username\n  error(\"Event hook on consumer \" .. user .. \"\")\nend\n"
        ]
    },
    "created_at": 1627595513,
    "event": "consumers",
    "handler": "lambda",
    "id": "c9fdd58d-5416-4d3a-9467-51e5cfe4ca0e",
    "on_change": null,
    "snooze": null,
    "source": "crud"
}
```

イベントフックのテスト
-----------

イベントをトリガーせずに、イベントフックを手動でトリガーすると便利です。たとえば、統合をテストしたり、フックのサービスが Kong からペイロードを受信しているかどうかを確認したい場合があります。

任意のデータを`/event-hooks/:id-of-hook/test`にPOSTすると、`/test`エンドポイントは提供されたデータをイベントペイロードとして実行します。

**エンドポイント** 
<div class="endpoint post">/event\-hooks/\{event\-hook\-id\}/test</div> 

**応答** 

```json
{
    "data": {
        "consumer": {
            "username": "Jane Austen"
        },
        "event": "consumers",
        "source": "crud"
    },
    "result": {
        "body": "",
        "headers": {
            "Cache-Control": "no-cache, private",
            "Content-Type": "text/plain; charset=UTF-8",
            "Date": "Fri, 30 Jul 2021 16:07:09 GMT",
            "Server": "nginx/1.14.2",
            "Transfer-Encoding": "chunked",
            "Vary": "Accept-Encoding",
            "X-Request-Id": "f1e703a5-d22c-435c-8d5d-bc9c561ead4a",
            "X-Token-Id": "1cc1c53b-f613-467f-a5c9-20d276405104"
        },
        "status": 200
    }
}

```

webhookイベントフックのネット接続の確認
-----------------------

**エンドポイント** 
<div class="endpoint get">/event\-hooks/\{event\-hook\-id\}/ping</div> 

**応答** 

```json
{
  "source": "kong:event_hooks",
  "event_hooks": {
    "source": "crud",
    "id": "c57340ab-9fed-40fd-bb7e-1cef8d37c2df",
    "on_change": null,
    "event": "consumers",
    "handler": "webhook",
    "created_at": 1627581575,
    "config": {
      "headers": {
        "content-type": "application/json"
      },
      "ssl_verify": false,
      "url": "https://webhook.site/a1b2c3-d4e5-g6h7-i8j9-k1l2m3n4o5p6",
      "secret": null
    },
    "snooze": null
  },
  "event": "ping"
}
```

