---
title: "キーリングとデータ暗号化のリファレンス"
badge: "enterprise"
---
キーリングを表示
--------

**エンドポイント** 
<div class="endpoint get">/keyring</div> 

**レスポンス** 

    HTTP 200 OK

```json
{
    "active": "RfsDJ2Ol",
    "ids": [
        "RfsDJ2Ol",
        "xSD219lH"
    ]
}

```

アクティブキーの表示
----------

**エンドポイント** 
<div class="endpoint get">/keyring/active</div> 

**レスポンス** 

    HTTP 200 OK

```json
{
    "id": "RfsDJ2Ol"
}

```

キーリングのエクスポート
------------

*このエンドポイントは、 `cluster`キーリング戦略でのみ使用できます。* 

*エンドポイントでは、`keyring_public_key`および`keyring_private_key`のKong構成値が定義されている必要があります。* 

**エンドポイント** 
<div class="endpoint post">/keyring/export</div> 

**レスポンス** 

    HTTP 200 OK

```json
{
    "data": "<base64 id="sl-md0000000">..."
}
```

エクスポートされたキーリングのインポート
--------------------

*このエンドポイントは、 `cluster`キーリング戦略でのみ使用できます。* 

*エンドポイントでは、`keyring_public_key`および`keyring_private_key`のKong構成値が定義されている必要があります。* 

**エンドポイント** 
<div class="endpoint post">/keyring/import</div> 

**リクエストボディ** 

|   属性   |               説明               |
|--------|--------------------------------|
| `data` | Base64でエンコードされた鍵束のエクスポートマテリアル。 |

**レスポンス** 

    HTTP 201 Created

インポートキー
-------

*このエンドポイントは、 `cluster`キーリング戦略でのみ使用できます。* 

*エンドポイントでは、`keyring_public_key`および`keyring_private_key`のKong構成値が定義されている必要があります。* 

**エンドポイント** 
<div class="endpoint post">/keyring/import/raw</div> 

**リクエストボディ** 

|   属性   |               説明               |
|--------|--------------------------------|
| `id`   | 8バイトのキー識別子。                    |
| `data` | Base64でエンコードされた鍵束のエクスポートマテリアル。 |

**レスポンス** 

    HTTP 201 Created

データベースからのキーリングの回復
-----------------

*このエンドポイントは、 `cluster`キーリング戦略でのみ使用できます。* 

*エンドポイントでは、`keyring_recovery_public_key`のKong構成値が定義されている必要があります。* 

**エンドポイント** 
<div class="endpoint post">/keyring/recover</div> 

**リクエストボディ** 

|           属性           |   説明    |
|------------------------|---------|
| `recovery_private_key` | 秘密鍵の内容。 |

**レスポンス** 

    HTTP 200 OK

```json
{
    "message": "successfully recovered 1 keys",
    "recovered": [
        "RfsDJ2Ol"
    ],
    "not_recovered": [
        "xSD219lH"
    ]
}
```

キーを新規生成
-------

*このエンドポイントは、 `cluster`キーリング戦略でのみ使用できます。* 

**エンドポイント** 
<div class="endpoint post">/keyring/generate</div> 

**レスポンス** 

    HTTP 201 Created

```json
{
    "id": "500pIquV",
    "key": "3I23Ben5m7qKcCA/PK7rnsNeD3kI4IPtA6ki7YjAgKA="
}
```

キーリングからキーを削除
------------

*このエンドポイントは、 `cluster`キーリング戦略でのみ使用できます。* 

**エンドポイント** 
<div class="endpoint post">/keyring/remove</div> 

**リクエストボディ** 

|  属性   |     説明      |
|-------|-------------|
| `key` | 8バイトのキー識別子。 |

**レスポンス** 

    HTTP 204 No Content

キーリングをVault Endpointと同期する
-------------------------

*このエンドポイントは、 `vault`キーリング戦略でのみ使用できます。* 

**エンドポイント** 
<div class="endpoint post">/keyring/vault/sync</div> 

**レスポンス** 

    HTTP 204 No Content

