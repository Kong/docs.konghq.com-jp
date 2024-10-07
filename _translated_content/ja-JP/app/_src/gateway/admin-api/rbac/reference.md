---
title: "RBACリファレンス"
badge: "enterprise"
---
Kong {{site.base_gateway}}のRBAC機能はKongの
[Admin API](/gateway/{{page.release}}/admin-api/)または[Kong Manager](/gateway/{{page.release}}/kong-manager/auth/rbac/)経由で構成できます。

RBACに関与する基本的なエンティティは次の4つです。

* **ユーザー** ：システムとやり取りするエンティティ。ゼロ、1つ、または複数のロールに関連付けることができます。 例：ユーザー`bob`はトークン`1234`を所有しています。
* **ロール** ：権限セット \( `role_endpoint`および`role_entity` ）。名前があり、0、1またはそれ以上の権限に関連付けられている。例：ユーザーbobは、`developer`のロールに関連付けられている。
* **role\_endpoint** : 有効または無効のセット（ `negative`を参照） パラメータ）アクション（ `read` 、 `create` 、 `update` 、 `delete` ） `endpoint`.例: ロール`developer`には 1 つの role\_endpoint: `read & write`エンドポイントにあります`/routes`
* **role\_entity** : 有効または無効の（`negative` パラメータを参照）アクションのセット（`read`、`create`、`update`、`delete`） `entity`。例: ロール`developer`には、エンティティ`283fccff-2d4f-49a9-8730-dc8b71ec2245`に対する1つのrole\_entity: `read & write & delete`があります。

ユーザーを追加
-------

**エンドポイント** 
<div class="endpoint post">/rbac/users</div> 

**リクエストボディ** 

|       属性        |                           説明                           |
|-----------------|--------------------------------------------------------|
| `name`          | RBACユーザー名。                                             |
| `user_token`    | Admin APIに提示される認証トークン。値はハッシュ化され、プレーンテキストで取得することはできません。 |
| `enabled`<br>任意 | ユーザーを有効または無効にするフラグです。デフォルトでは、ユーザーは有効になっています。           |
| `comment`<br>任意 | RBACユーザーオブジェクトを説明する文字列。                                |

**レスポンス** 

    HTTP 201 Created

```json
{
  "comment": null,
  "created_at": 1557522650,
  "enabled": true,
  "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
  "name": "doc_knight",
  "user_token": "$2b$09$Za30VKAAAmyoB9zF2PNEF.9hgKcN2BdKkptPMCubPK/Ps08lzZjYG",
  "user_token_ident": "4d870"
}
```

*** ** * ** ***

ユーザーの取得
-------

**エンドポイント** 
<div class="endpoint get">/rbac/users/\{name\_or\_id\}</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "created_at": 1557522650,
  "enabled": true,
  "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
  "name": "doc_lord",
  "user_token": "$2b$09$Za30VKAAAmyoB9zF2PNEF.9hgKcN2BdKkptPMCubPK/Ps08lzZjYG",
  "user_token_ident": "4d870"
}
```

*** ** * ** ***

ユーザーを一覧化
--------

**エンドポイント** 
<div class="endpoint get">/rbac/users/</div> 

**レスポンス** 

    HTTP 200 OK

```json
{
  "data": [
    {
      "comment": null,
      "created_at": 1557512629,
      "enabled": true,
      "id": "f035f120-a95e-4327-b2ae-8fa264601d75",
      "name": "doc_lord",
      "user_token": "$2b$09$TIMneYcTosdG9WbzRsqcweAS2zote8g6I8HqXAtbFHR1pds2ymsh6",
      "user_token_ident": "88ea3"
    },
    {
      "comment": null,
      "created_at": 1557522650,
      "enabled": true,
      "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
      "name": "doc_knight",
      "user_token": "$2b$09$Za30VKAAAmyoB9zF2PNEF.9hgKcN2BdKkptPMCubPK/Ps08lzZjYG",
      "user_token_ident": "4d870"
    }
  ],
  "next": null
}
```

⚠️ **注意** ： **管理者** に関連付けられた **RBACユーザー** は、 **`GET`** `/rbac/users`に表記され *ません* 。代わりに、[**`GET`** `/admins`](/gateway/{{page.release}}/admin-api/admins/reference/#list-admins)を使用して、すべての **管理者** が表記されます。

*** ** * ** ***

ユーザーを更新する
---------

**エンドポイント** 
<div class="endpoint patch">/rbac/users/\{name\_or\_id\}</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**リクエストボディ** 

|         属性         |                         説明                         |
|--------------------|----------------------------------------------------|
| `user_token`<br>任意 | Admin APIに提示される認証トークン。この値が存在しない場合、トークンは自動的に生成されます。 |
| `enabled`<br>任意    | ユーザーを有効または無効にするフラグです。デフォルトでは、ユーザーは有効になっています。       |
| `comment`<br>任意    | RBACユーザーオブジェクトを説明する文字列。                            |

**レスポンス** 

    HTTP 200 OK

```json
{
  "comment": "this comment came from a patch request",
  "created_at": 1557522650,
  "enabled": true,
  "id": "fa6881b2-f49f-4007-9475-577cd21d34f4",
  "name": "donut_lord",
  "user_token": "$2b$09$Za30VKAAAmyoB9zF2PNEF.9hgKcN2BdKkptPMCubPK/Ps08lzZjYG",
  "user_token_ident": "4d870"
}
```

*** ** * ** ***

ユーザーを削除
-------

**エンドポイント** 
<div class="endpoint delete">/rbac/users/\{name\_or\_id\}</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**レスポンス** 

    HTTP 204 No Content

*** ** * ** ***

ロールを追加する
--------

**エンドポイント** 
<div class="endpoint post">/rbac/roles</div> 

|       属性        |           説明            |
|-----------------|-------------------------|
| `name`          | RBAC ロール名。              |
| `comment`<br>任意 | RBACユーザーオブジェクトを説明する文字列。 |

**レスポンス** 

    HTTP 201 Created

```json
{
  "comment": null,
  "created_at": 1557532241,
  "id": "b5c5cfd4-3330-4796-9b7b-6026e91e3ad6",
  "is_default": false,
  "name": "service_reader"
}
```

*** ** * ** ***

ロールを取得
------

エンドポイント
<div class="endpoint get">/rbac/roles/\{name\_or\_id\}</div> 

|      属性      |         説明         |
|--------------|--------------------|
| `name_or_id` | RBAC ロール名または UUID。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "created_at": 1557532241,
  "id": "b5c5cfd4-3330-4796-9b7b-6026e91e3ad6",
  "is_default": false,
  "name": "service_reader"
}
```

*** ** * ** ***

ロールの一覧表示
--------

**エンドポイント** 
<div class="endpoint get">/rbac/roles</div> 

**レスポンス** 

    HTTP 200 OK

```json
{
  "data": [
    {
      "comment": "Full access to all endpoints, across all workspaces—except RBAC Admin API",
      "created_at": 1557506249,
      "id": "38a03d47-faae-4366-b430-f6c10aee5029",
      "name": "admin"
    },
    {
      "comment": "Read access to all endpoints, across all workspaces",
      "created_at": 1557506249,
      "id": "4141675c-8beb-41a5-aa04-6258ab2d2f7f",
      "name": "read-only"
    },
    {
      "comment": "Full access to all endpoints, across all workspaces",
      "created_at": 1557506249,
      "id": "888117e0-f2b3-404d-823b-dee595423505",
      "name": "super-admin"
    },
    {
      "comment": null,
      "created_at": 1557532241,
      "id": "b5c5cfd4-3330-4796-9b7b-6026e91e3ad6",
      "name": "doc_lord"
    }
  ],
  "next": null
}
```

*** ** * ** ***

ロールの更新または作成
-----------

**エンドポイント** 
<div class="endpoint put">/rbac/roles/\{name\_or\_id\}</div> 

**リクエストボディ** 

|       属性        |           説明            |
|-----------------|-------------------------|
| `name`          | RBAC ロール名。              |
| `comment`<br>任意 | RBACユーザーオブジェクトを説明する文字列。 |

`PUT` エンドポイントの動作は次のとおりです。リクエストのペイロードにエンティティのプライマリキーが **含まれている** 場合、ペイロードは指定されたプライマリキーでエンティティを「置き換え」ます。プライマリキーが既存のエンティティのもの **ではない** 場合、エンティティは指定されたペイロードで作成されます。

**レスポンス** 

エンティティを作成する場合:

    HTTP 201 Created

エンティティを置き換える場合：

    HTTP 200 OK

```json
{
  "comment": "the best",
  "created_at": 1557532566,
  "id": "b5c5cfd4-3330-4796-9b7b-6026e91e3ad6",
  "is_default": false,
  "name": "doc_lord"
}
```

ロールの更新
------

**エンドポイント** 
<div class="endpoint patch">/rbac/roles/\{name\_or\_id\}</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBAC ロールまたは UUID。 |

**リクエストボディ** 

|       属性        |           説明           |
|-----------------|------------------------|
| `comment`<br>任意 | RBACロールオブジェクトを説明する文字列。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "comment": "comment from patch request",
  "created_at": 1557532566,
  "id": "b5c5cfd4-3330-4796-9b7b-6026e91e3ad6",
  "is_default": false,
  "name": "service_reader"
}
```

*** ** * ** ***

ロールの削除
------

**エンドポイント** 
<div class="endpoint delete">/rbac/roles/\{name\_or\_id\}</div> 

|   属性   |     説明     |
|--------|------------|
| `name` | RBAC ロール名。 |

**レスポンス** 

    HTTP 204 No Content

*** ** * ** ***

ロールエンドポイントのアクセス許可を追加する
----------------------

**エンドポイント** 
<div class="endpoint post">/rbac/roles/\{name\_or\_id\}/endpoints</div> 

|      属性      |     説明     |
|--------------|------------|
| `name_or_id` | RBAC ロール名。 |

**リクエストボディ** 

|       属性        |                                             説明                                             |
|-----------------|--------------------------------------------------------------------------------------------|
| `workspace`     | エンドポイントに関連付けられたワークスペース。defaultの権限がデフォルトになります。「\*」の特別な値は、 **すべて** のワークスペースが影響を受けることを意味します。 |
| `endpoint`      | この権限に関連付けられているエンドポイント。                                                                     |
| `negative`      | これが「正」の場合は、このエンドポイントに結び付けられた権限に関連するアクションを明示的に禁止します。デフォルトでは、この値は「偽」です。                      |
| `actions`       | この権限に関連付けられた1つ以上のアクション。カンマ区切りの文字列（読み取り、作成、更新、削除）です                                         |
| `comment`<br>任意 | RBAC権限オブジェクトを説明する文字列。                                                                      |

`endpoint`は、関連するエンドポイントのパスでなければなりません。完全一致にすることも、`*`で表されるワイルドカードを含めることもできます。

* 完全一致：

  * /services/
  * /services/foo

* ワイルドカード：

  * /services/\*
  * /services/\*/plugins

`*`を置き換える場所はスラッシュ間のちょうど1セグメント（またはパスの
終端）です。

ワイルドカードはネストできることに注意してください（`/rbac/*`、`/rbac/*/*`、`/rbac/*/*/*` は `/rbac/` 配下のすべてのパスを指します）

**レスポンス** 

    HTTP 201 Created

```json
{
  "actions": [
    "delete",
    "create",
    "update",
    "read"
  ],
  "created_at": 1557764505,
  "endpoint": "/consumers",
  "negative": false,
  "role": {
    "id": "23df9f20-e7cc-4da4-bc89-d3a08f976e50"
  },
  "workspace": "default"
}
```

*** ** * ** ***

ロールエンドポイントの権限を取得
----------------

**エンドポイント** 
<div class="endpoint get">/rbac/roles/\{name\_or\_id\}/endpoints/\{workspace\_name\_or\_id\}/\{endpoint\}</div> 

|           属性           |           説明           |
|------------------------|------------------------|
| `name_or_id`           | RBAC ロール名または UUID。     |
| `workspace_name_or_id` | ワークスペース名またはUUID。       |
| `endpoint`             | この権限に関連付けられているエンドポイント。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "actions": [
    "delete",
    "create",
    "update",
    "read"
  ],
  "created_at": 1557764505,
  "endpoint": "/consumers",
  "negative": false,
  "role": {
    "id": "23df9f20-e7cc-4da4-bc89-d3a08f976e50"
  },
  "workspace": "default"
}
```

*** ** * ** ***

ロールエンドポイントの権限を一覧表示する
--------------------

**エンドポイント** 
<div class="endpoint get">/rbac/roles/\{role\_name\_or\_id\}/endpoints</div> 

|        属性         |         説明         |
|-------------------|--------------------|
| `role_name_or_id` | RBAC ロール名または UUID。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "data": [
    {
      "actions": [
        "delete",
        "create",
        "update",
        "read"
      ],
      "created_at": 1557764505,
      "endpoint": "/consumers",
      "negative": false,
      "role": {
        "id": "23df9f20-e7cc-4da4-bc89-d3a08f976e50"
      },
      "workspace": "default"
    },
    {
      "actions": [
        "read"
      ],
      "created_at": 1557764438,
      "endpoint": "/services",
      "negative": false,
      "role": {
        "id": "23df9f20-e7cc-4da4-bc89-d3a08f976e50"
      },
      "workspace": "default"
    }
  ]
}
```

*** ** * ** ***

ロールエンドポイント権限の更新
---------------

**エンドポイント** 
<div class="endpoint patch">/rbac/roles/\{name\_or\_id\}/endpoints/\{workspace\_name\_or\_id\}/\{endpoint\}</div> 

|           属性           |           説明           |
|------------------------|------------------------|
| `name_or_id`           | RBAC ロール名または UUID。     |
| `workspace_name_or_id` | ワークスペース名またはUUID。       |
| `endpoint`             | この権限に関連付けられているエンドポイント。 |

**リクエストボディ** 

|     属性     |                                説明                                 |
|------------|-------------------------------------------------------------------|
| `negative` | true の場合、このリソースに紐づいた権限に関連するアクションを明示的に禁止します。デフォルトでは、この値は false です。 |
| `actions`  | この権限に関連付けられた1つ以上のアクション。                                           |

**レスポンス** 

    HTTP 200 OK

```json
{
  "actions": [
    "delete",
    "create",
    "update",
    "read"
  ],
  "created_at": 1557764438,
  "endpoint": "/services",
  "negative": false,
  "role": {
    "id": "23df9f20-e7cc-4da4-bc89-d3a08f976e50"
  },
  "workspace": "default"
}
```

*** ** * ** ***

ロールエンドポイント権限の削除
---------------

**エンドポイント** 
<div class="endpoint delete">/rbac/roles/\{name\_or\_id\}/endpoints/\{workspace\_name\_or\_id\}/\{endpoint\}</div> 

|           属性           |           説明           |
|------------------------|------------------------|
| `name_or_id`           | RBAC ロール名または UUID。     |
| `workspace_name_or_id` | ワークスペース名またはUUID。       |
| `endpoint`             | この権限に関連付けられているエンドポイント。 |

**レスポンス** 

    HTTP 204 No Content

*** ** * ** ***

ロールエンティティ権限を追加
--------------

**エンドポイント** 
<div class="endpoint post">/rbac/roles/\{name\_or\_id\}/entities</div> 

|      属性      |         説明         |
|--------------|--------------------|
| `name_or_id` | RBAC ロール名または UUID。 |

**リクエストボディ** 

|       属性        |                                説明                                 |
|-----------------|-------------------------------------------------------------------|
| `negative`      | true の場合、このリソースに紐づいた権限に関連するアクションを明示的に禁止します。デフォルトでは、この値は false です。 |
| `entity_id`     | この権限に関連付けられているエンティティのID。                                          |
| `entity_type`   | 特定の `entity_id`のエンティティのタイプ。                                       |
| `actions`       | この権限に関連付けられた1つ以上のアクション。                                           |
| `comment`<br>任意 | RBAC アクセス許可オブジェクトを説明する文字列                                         |

`entity_id` は、Kong 内のエンティティのIDでなければなりません。ワークスペースの ID が与えられた場合、権限はそのワークスペース内のすべてのエンティティに適用されます将来そのワークスペースに属するエンティティは、同じ権限を取得します。ワイルドカード `*` は、そのシステム内の **すべてのエンティティ** と解釈されます。

**レスポンス** 

    HTTP 201 Created

```json
{
  "actions": [
    "delete",
    "create",
    "read"
  ],
  "created_at": 1557771505,
  "entity_id": "*",
  "entity_type": "wildcard",
  "negative": false,
  "role": {
    "id": "bba049fa-bf7e-40ef-8e89-553dda292e99"
  }
}
```

*** ** * ** ***

ロールエンティティの権限を取得
---------------

**エンドポイント** 
<div class="endpoint get">/rbac/roles/\{name\_or\_id\}/entities/\{entity\_id\}</div> 

|      属性      |            説明            |
|--------------|--------------------------|
| `name_or_id` | RBAC権限名またはUUID。          |
| `entity_id`  | この権限に関連付けられているエンティティのID。 |

**レスポンス** 

    HTTP 200 Ok

```json
{
  "actions": [
    "delete",
    "create",
    "read"
  ],
  "created_at": 1557771505,
  "entity_id": "*",
  "entity_type": "wildcard",
  "negative": false,
  "role": {
    "id": "bba049fa-bf7e-40ef-8e89-553dda292e99"
  }
}
```

*** ** * ** ***

エンティティのアクセス許可を一覧表示する
--------------------

**エンドポイント** 
<div class="endpoint get">/rbac/roles/\{name\_or\_id\}/entities</div> 

|      属性      |       説明        |
|--------------|-----------------|
| `name_or_id` | RBAC権限名またはUUID。 |

**レスポンス** 

    HTTP 200 Ok

```json
{
  "data": [
    {
      "actions": [
        "delete",
        "create",
        "read"
      ],
      "created_at": 1557771505,
      "entity_id": "*",
      "entity_type": "wildcard",
      "negative": false,
      "role": {
        "id": "bba049fa-bf7e-40ef-8e89-553dda292e99"
      }
    }
  ]
}
```

*** ** * ** ***

エンティティの権限を更新する
--------------

**エンドポイント** 
<div class="endpoint patch">/rbac/roles/\{name\_or\_id\}/entities/\{entity\_id\}</div> 

|      属性      |         説明         |
|--------------|--------------------|
| `name_or_id` | RBAC ロール名または UUID。 |
| `entity_id`  | エンティティ名または UUID。   |

**リクエストボディ** 

|     属性     |                                説明                                 |
|------------|-------------------------------------------------------------------|
| `negative` | true の場合、このリソースに紐づいた権限に関連するアクションを明示的に禁止します。デフォルトでは、この値は false です。 |
| `actions`  | この権限に関連付けられた1つ以上のアクション。                                           |

**レスポンス** 

    HTTP 200 OK

```json
{
  "actions": [
    "update"
  ],
  "created_at": 1557771505,
  "entity_id": "*",
  "entity_type": "wildcard",
  "negative": false,
  "role": {
    "id": "bba049fa-bf7e-40ef-8e89-553dda292e99"
  }
}
```

*** ** * ** ***

エンティティのアクセス許可を削除する
------------------

**エンドポイント** 
<div class="endpoint delete">/rbac/roles/\{name\_or\_id\}/entities/\{entity\_id\}</div> 

|      属性      |         説明         |
|--------------|--------------------|
| `name_or_id` | RBAC ロール名または UUID。 |
| `entity_id`  | エンティティ名または UUID。   |

**レスポンス** 

    HTTP 204 No Content

*** ** * ** ***

ロール権限を一覧表示する
------------

**エンドポイント** 
<div class="endpoint get">/rbac/roles/\{name\_or\_id\}/permissions/</div> 

|      属性      |         説明         |
|--------------|--------------------|
| `name_or_id` | RBAC ロール名または UUID。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "endpoints": {
    "*": {
      "*": {
        "actions": [
          "delete",
          "create",
          "update",
          "read"
        ],
        "negative": false
      },
      "/*/rbac/*": {
        "actions": [
          "delete",
          "create",
          "update",
          "read"
        ],
        "negative": true
      }
    }
  },
  "entities": {}
}
```

ロールにユーザーを追加
-----------

**エンドポイント** 
<div class="endpoint post">/rbac/users/\{name\_or\_id\}/roles</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**リクエストボディ** 

|   属性    |              説明              |
|---------|------------------------------|
| `roles` | ユーザーに割り当てるロール名のコンマで区切られたリスト。 |

**レスポンス** 

    HTTP 201 Created

```json
{
  "roles": [
    {
      "created_at": 1557772263,
      "id": "aae80073-095f-4553-ba9a-bee5ed3b8b91",
      "name": "doc-knight"
    }
  ],
  "user": {
    "comment": null,
    "created_at": 1557772232,
    "enabled": true,
    "id": "b65ca712-7ceb-4114-87f4-5c310492582c",
    "name": "gruce-wayne",
    "user_token": "$2b$09$gZnMKK/mm/d2rAXN7gL63uL43mjdX/62iwMqdyCQwLyC0af3ce/1K",
    "user_token_ident": "88ea3"
  }
}
```

*** ** * ** ***

ユーザーのロールを表記
-----------

**エンドポイント** 
<div class="endpoint get">/rbac/users/\{name\_or\_id\}/roles</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**レスポンス** 

    HTTP 200 OK

```json

{
  "roles": [
    {
      "comment": "Read access to all endpoints, across all workspaces",
      "created_at": 1557765500,
      "id": "a1c810ee-8366-4654-ba0c-963ffb9ccf2e",
      "name": "read-only"
    },
    {
      "created_at": 1557772263,
      "id": "aae80073-095f-4553-ba9a-bee5ed3b8b91",
      "name": "doc-knight"
    }
  ],
  "user": {
    "comment": null,
    "created_at": 1557772232,
    "enabled": true,
    "id": "b65ca712-7ceb-4114-87f4-5c310492582c",
    "name": "gruce-wayne",
    "user_token": "$2b$09$gZnMKK/mm/d2rAXN7gL63uL43mjdX/62iwMqdyCQwLyC0af3ce/1K",
    "user_token_ident": "88ea3"
  }
}
```

*** ** * ** ***

ユーザーからロールを削除する
--------------

**エンドポイント** 
<div class="endpoint delete">/rbac/users/\{name\_or\_id\}/roles</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**リクエストボディ** 

|   属性    |              説明              |
|---------|------------------------------|
| `roles` | ユーザーに割り当てるロール名のコンマで区切られたリスト。 |

**レスポンス** 

    HTTP 204 No Content

*** ** * ** ***

ユーザーの権限を一覧表示する
--------------

**エンドポイント** 
<div class="endpoint get">/rbac/users/\{name\_or\_id\}/permissions</div> 

|      属性      |        説明         |
|--------------|-------------------|
| `name_or_id` | RBACユーザー名またはUUID。 |

**レスポンス** 

    HTTP 200 OK

```json
{
  "endpoints": {
    "*": {
      "*": {
        "actions": [
          "read"
        ],
        "negative": false
      }
    }
  },
  "entities": {}
}

```

