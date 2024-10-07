---
title: "コンシューマグループリファレンス"
badge: "enterprise"
---
コンシューマグループを使用して、コンシューマのサブセットのカスタム流量制限構成を管理します。

`consumer_groups`エンドポイントは、[Rate Limiting Advancedプラグイン](/hub/kong-inc/rate-limiting-advanced/)と連携します。
流量制限にコンシューマグループを使用するには、`enforce_consumer_groups`と`consumer_groups`パラメータでプラグインを構成してから、`/consumer_groups`エンドポイントを使用してグループを管理します。

コンシューマグループは、[タグ付けすることも、タグでフィルタリングすることもできます](/gateway/{{page.release}}/admin-api/#tags)。

コンシューマグループの設定と管理に関する詳細と例については、[コンシューマグループの例](/gateway/{{page.release}}/kong-enterprise/consumer-groups)を参照してください。

コンシューマグループの一覧表示
---------------

### すべてのコンシューマグループを一覧表示する

**エンドポイント** 
<div class="endpoint get">/consumer\_groups</div> 

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "data": [
        {
            "created_at": 1557522650,
            "id": "42b022c1-eb3c-4512-badc-1aee8c0f50b5",
            "name": "my_group",
            "tags": null
        },
        {
            "created_at": 1637706162,
            "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
            "name": "my_group2",
            "tags": null
        }
    ],
    "next": null
}
```

### 特定のコンシューマグループを一覧表示する

**エンドポイント** 
<div class="endpoint get">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}</div> 

|                             属性 |          説明           |
|-------------------------------:|-----------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | コンシューマグループの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumer_group": {
        "created_at": 1638917780,
        "id": "be4bcfca-b1df-4fac-83cc-5cf6774bf48e",
        "name": "JL",
        "tags": null
    }
}
```

### コンシューマグループ内のすべてのコンシューマを一覧表示する

**エンドポイント** 
<div class="endpoint get">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}/consumers</div> 

|                             属性 |          説明           |
|-------------------------------:|-----------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | コンシューマグループの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumers": [
        {
            "created_at": 1638918560,
            "id": "288f2bfc-04e2-4ec3-b6f3-40408dff5417",
            "type": 0,
            "username": "BarryAllen",
            "username_lower": "barryallen"
        },
        {
            "created_at": 1638915577,
            "id": "8089a0e6-1d31-4e00-bf51-5b902899b4cb",
            "type": 0,
            "username": "DianaPrince",
            "username_lower": "dianaprince"
        }
    ]
}
```

### あるコンシューマ向けのコンシューマグループの一覧表示

コンシューマが割り当てられているすべてのコンシューマグループを表示します。

**エンドポイント** 
<div class="endpoint get">/consumers/\{CONSUMER\_NAME\|CONSUMER\_ID\}/consumer\_groups</div> 

|                              属性 |        説明         |
|--------------------------------:|-------------------|
| `USERNAME|CONSUMER_ID`<br> *必須* | コンシューマの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumer_groups": [
        {
            "created_at": 1638918476,
            "id": "e2c3f16e-22c7-4ef4-b6e4-ab25c522b339",
            "name": "JL",
            "tags": null,
        }
    ]
}
```

コンシューマグループを作成
-------------

**エンドポイント** 
<div class="endpoint post">/consumer\_groups</div> 

**リクエストボディ** 

|                 属性 |                      説明                       |
|-------------------:|-----------------------------------------------|
|    `name`<br> *必須* | 作成するコンシューマグループの一意の名前。                         |
| `tags`<br> *オプション* | グループ化とフィルタリングのためにコンシューマグループに関連付けられた文字列の任意セット。 |

**レスポンス** 

    HTTP 201 Created

```json
{
  "created_at": 1557522650,
  "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
  "name": "JL",
  "tags": ["tag1", "tag2"]
}
```

**エンドポイント** 
<div class="endpoint put">/consumer\_groups/\{GROUP\_NAME\}</div> 

|                    属性 |                      説明                       |
|----------------------:|-----------------------------------------------|
| `GROUP_NAME`<br> *必須* | 作成するコンシューマグループの一意の名前。                         |
|    `tags`<br> *オプション* | グループ化とフィルタリングのためにコンシューマグループに関連付けられた文字列の任意セット。 |

**レスポンス** 

    HTTP 201 Created

```json
{
  "created_at": 1557522650,
  "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
  "name": "JL",
  "tags": ["tag1", "tag2"]
}
```

グループにコンシューマを追加する
----------------

特定のコンシューマグループにコンシューマを追加します。

コンシューマを複数のグループに追加する場合：

* すべてのグループがRate Limiting Advancedプラグインで許可されている場合は、最初のグループの設定だけが適用されます。
* それ以外の場合は、Rate Limiting Advanced プラグインで指定されたグループがアクティブになります。

**コンシューマエンドポイント** 
<div class="endpoint post">/consumers/\{CONSUMER\_NAME\|CONSUMER\_ID\}/consumer\_groups</div> 

|                                   属性 |        説明         |
|-------------------------------------:|-------------------|
| `CONSUMER_NAME|CONSUMER_ID`<br> *必須* | コンシューマの名前またはUUID。 |

**リクエストボディ** 

|               属性 |            説明             |
|-----------------:|---------------------------|
| `group`<br> *必須* | コンシューマを追加するグループの名前または ID。 |

**レスポンス** 

    HTTP 201 Created

```json
{
    "consumer": {
        "created_at": 1638918560,
        "custom_id": null,
        "id": "288f2bfc-04e2-4ec3-b6f3-40408dff5417",
        "tags": null,
        "type": 0,
        "username": "BarryAllen",
        "username_lower": "barryallen"
    },
    "consumer_groups": [
        {
            "created_at": 1638918476,
            "id": "e2c3f16e-22c7-4ef4-b6e4-ab25c522b339",
            "name": "JL",
            "tags": null
        }
    ]
}
```

**コンシューマグループエンドポイント** 
<div class="endpoint post">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}/consumers</div> 

|                             属性 |          説明           |
|-------------------------------:|-----------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | コンシューマグループの名前またはUUID。 |

**リクエストボディ** 

|                  属性 |             説明              |
|--------------------:|-----------------------------|
| `consumer`<br> *必須* | このグループに追加するコンシューマの名前または ID。 |

**レスポンス** 

    HTTP 201 Created

```json
{
  "consumer_group": {
  "created_at": 1638915521,
  "id": "8a4bba3c-7f82-45f0-8121-ed4d2847c4a4",
  "name": "JL"
  },
  "consumers": [
    {
        "created_at": 1638915577,
        "id": "8089a0e6-1d31-4e00-bf51-5b902899b4cb",
        "type": 0,
        "username": "DianaPrince",
        "username_lower": "dianaprince"
    }
  ]
}
```

コンシューマグループを削除
-------------

コンシューマグループを削除すると、そのグループからすべてのコンシューマが削除されます。コンシューマは削除 **されません** 。

**エンドポイント** 
<div class="endpoint delete">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}</div> 

|                             属性 |             説明             |
|-------------------------------:|----------------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | 削除するコンシューマグループの名前または UUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

コンシューマの削除
---------

### コンシューマをすべてのグループから削除する

**エンドポイント** 
<div class="endpoint delete">/consumers/\{CONSUMER\_NAME\|CONSUMER\_ID\}/consumer\_groups</div> 

|                                   属性 |                説明                |
|-------------------------------------:|----------------------------------|
| `CONSUMER_NAME|CONSUMER_ID`<br> *必須* | すべてのグループから削除するコンシューマの名前または UUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

### コンシューマグループからのコンシューマの削除

**コンシューマエンドポイント** 
<div class="endpoint delete">/consumers/\{CONSUMER\_NAME\|CONSUMER\_ID\}/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}</div> 

|                                   属性 |                 説明                 |
|-------------------------------------:|------------------------------------|
| `CONSUMER_NAME|CONSUMER_ID`<br> *必須* | 削除するコンシューマの名前または UUID。             |
|       `GROUP_NAME|GROUP_ID`<br> *必須* | コンシューマを削除するコンシューマ グループの名前または UUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

**コンシューマグループエンドポイント** 
<div class="endpoint delete">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}/consumers/\{CONSUMER\_NAME\|CONSUMER\_ID\}</div> 

|                                   属性 |                 説明                 |
|-------------------------------------:|------------------------------------|
|       `GROUP_NAME|GROUP_ID`<br> *必須* | コンシューマを削除するコンシューマ グループの名前または UUID。 |
| `CONSUMER_NAME|CONSUMER_ID`<br> *必須* | 削除するコンシューマの名前または UUID。             |

**レスポンス** 

    HTTP/1.1 204 No Content

### コンシューマグループからすべてのコンシューマを削除

<div class="endpoint delete">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}/consumers</div> 

|                             属性 |                  説明                  |
|-------------------------------:|--------------------------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | すべてのコンシューマを削除するコンシューマグループの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

コンシューマグループの流量制限の設定
------------------

コンシューマグループのカスタム流量制限設定を定義します。このエンドポイントは、Rate Limiting Advancedプラグインの設定を上書きします。
<div class="endpoint put">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}/overrides/plugins/rate\-limiting\-advanced</div> 

|                             属性 |            説明             |
|-------------------------------:|---------------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | 設定するコンシューマグループの名前またはUUID。 |

**リクエストボディ** 

|                                          属性 |                                                                                   説明                                                                                   |
|--------------------------------------------:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     `config.limit`<br> *必須* | 適用する1つ以上のウィンドウあたりのリクエスト制限の配列。指定されたウィンドウ制限とサイズの数が一致している必要があります。                                                                                                         |
|               `config.window_size`<br> *必須* | 制限を適用する1つ以上のウィンドウサイズの配列（秒単位で定義）。指定されたウィンドウ制限とサイズの数が一致している必要があります。                                                                                                      |
|            `config.window_type`<br> *オプション* | 時間ウィンドウのタイプを`sliding`（デフォルト）または`fixed`に設定します。                                                                                                                          |
| `config.retry_after_jitter_max`<br> *オプション* | すべてのクライアントが同時に戻ってくるのを防ぐために、拒否されたリクエストの `Retry-After` ヘッダー（ステータス= `429`）に追加されるジッター（ランダム遅延）の上限（秒単位）。ジッターの下限は`0`です。この場合、 `Retry-After`ヘッダーは`RateLimit-Reset`ヘッダーと等しくなります。 |

**レスポンス** 

    HTTP/1.1 201 Created

```json
{
    "config": {
        "limit": [
            10
        ],
        "retry_after_jitter_max": 0,
        "window_size": [
            10
        ],
        "window_type": "sliding"
    },
    "group": "test-group",
    "plugin": "rate-limiting-advanced"
}
```

コンシューマグループの設定の削除
----------------

コンシューマグループのカスタム流量制限設定を削除します。
<div class="endpoint delete">/consumer\_groups/\{GROUP\_NAME\|GROUP\_ID\}/overrides/plugins/rate\-limiting\-advanced</div> 

|                             属性 |            説明             |
|-------------------------------:|---------------------------|
| `GROUP_NAME|GROUP_ID`<br> *必須* | 設定するコンシューマグループの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

