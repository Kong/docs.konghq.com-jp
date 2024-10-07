---
title: "管理者リファレンス"
badge: "enterprise"
---
リスト管理者
------

**エンドポイント** 
<div class="endpoint get">/admins</div> 

**レスポンス** 

    HTTP 200 OK

```json
{
  "data": [{
    "created_at": 1556638385,
    "id": "665b4070-541f-48bf-82c1-53030babaa81",
    "updated_at": 1556638385,
    "status": 4,
    "username": "test-admin",
    "email": "test@test.com",
    "rbac_token_enabled": true
  }, {
    "created_at": 1556563122,
    "id": "a93ff120-9e6c-4198-b47e-f779104c7eac",
    "updated_at": 1556563122,
    "status": 0,
    "username": "kong_admin",
    "rbac_token_enabled": false
  }],
  "next": null
}
```

すべての管理者に対してクエリを実行するには、 `/admins` エンドポイントにパラメータ `all_workspaces=true` を追加します。

レスポンスの`status`フィールドは、管理者が招待を承諾したかどうかを示します。

| コード |  ステータス  |
|-----|---------|
| 0   | 承認済み    |
| 1   | 保留中     |
| 2   | 拒否済み    |
| 3   | Revoked |
| 4   | 招待済み    |
| 5   | 未確認     |

管理者を招待
------

**エンドポイント** 
<div class="endpoint post">/admins</div> 

|          属性          |                          説明                           |
|----------------------|-------------------------------------------------------|
| `email`              | **管理者** のメールアドレス                                      |
| `username`           | **管理者** のユーザー名                                        |
| `custom_id`<br>任意    | **管理者** のカスタムID                                       |
| `rbac_token_enabled` | **管理者** がRBAC トークンを使用およびリセットできるようにします。デフォルトでは`true`です |

**レスポンス** 

    HTTP 200 OK

```json
{
  "admin": {
    "created_at": 1556638641,
    "id": "8f0a742f-07f3-49e0-90d7-4fc7eea7e6a4",
    "updated_at": 1556638641,
    "status": 4,
    "username": "test-case-3",
    "email": "test3@test.com",
    "rbac_token_enabled": true
  }
}
```

管理者の認証情報を登録する
-------------

**エンドポイント** 
<div class="endpoint post">/admins/register</div> 

|     属性     |        説明         |
|------------|-------------------|
| `token`    | 認証トークン            |
| `username` | **管理者** のユーザー名    |
| `email`    | **管理者** のメールアドレス  |
| `password` | **管理者** の新しいパスワード |

**レスポンス** 

    HTTP 201 Created

パスワードリセットのメールを管理者に送信する
----------------------

**エンドポイント** 
<div class="endpoint post">/admins/password\_resets</div> 

|   属性    |        説明        |
|---------|------------------|
| `email` | **管理者** のメールアドレス |

**レスポンス** 

    HTTP 201 Created

管理者のパスワードをリセットする
----------------

**エンドポイント** 
<div class="endpoint patch">/admins/password\_resets</div> 

|     属性     |        説明         |
|------------|-------------------|
| `email`    | **管理者** のメールアドレス  |
| `password` | **管理者** の新しいパスワード |
| `token`    | 認証トークン            |

**レスポンス** 

    HTTP 200 OK

管理者の取得
------

**エンドポイント** 
<div class="endpoint get">/admins/\{name\_or\_id\}</div> 

|              属性               |               説明                |
|-------------------------------|---------------------------------|
| `name_or_id`                  | **管理者** のユーザー名またはID             |
| `generate_register_url`<br>任意 | `true` **管理者** 用の一意の登録URLを返します。 |

**注：** 

* `generate_register_url`がURLを生成するのは、 **管理者** の の招待ステータスが 4（「招待済み」）の場合のみです。
* `generate_register_url`は、リクエストが行われるたびに特定の **管理者** の以前の登録URLを上書きします。

**レスポンス** 

    HTTP 200 OK

```json
{
  "created_at": 1556638385,
  "id": "665b4070-541f-48bf-82c1-53030babaa81",
  "updated_at": 1556638385,
  "status": 4,
  "username": "test-admin",
  "email": "test@test.com",
  "rbac_token_enabled": true
}
```

管理者の更新
------

**エンドポイント** 
<div class="endpoint patch">/admins/\{name\_or\_id\}</div> 

|          属性          |                          説明                           |
|----------------------|-------------------------------------------------------|
| `name_or_id`         | **管理者** の現在のユーザー名またはカスタムID                            |
| `email`<br>任意        | **管理者** の新しいメールアドレス                                   |
| `username`<br>任意     | **管理者** の新しいユーザー名                                     |
| `custom_id`<br>任意    | **管理** 者の新しいカスタムID                                    |
| `rbac_token_enabled` | **管理者** がRBAC トークンを使用およびリセットできるようにします。デフォルトでは`true`です |

**レスポンス** 

    HTTP 200 OK

```json
{
  "created_at": 1556638385,
  "id": "665b4070-541f-48bf-82c1-53030babaa81",
  "updated_at": 1556639017,
  "status": 4,
  "username": "test-renamed",
  "email": "test@test.com"
  "rbac_token_enabled": true
}
```

管理者を削除する
--------

**エンドポイント** 
<div class="endpoint delete">/admins/\{name\_or\_id\}</div> 

|      属性      |         説明          |
|--------------|---------------------|
| `name_or_id` | **管理者** のユーザー名またはID |

**レスポンス** 

    HTTP 204 No Content

管理者の役割を一覧表示する
-------------

**エンドポイント** 
<div class="endpoint get">/admins/\{name\_or\_id\}/roles</div> 

|      属性      |         説明          |
|--------------|---------------------|
| `name_or_id` | **管理者** のユーザー名またはID |

**レスポンス** 

    HTTP 200 OK

```json
{
  "roles": [{
    "comment": "Read access to all endpoints, across all workspaces",
    "created_at": 1556563122,
    "id": "7574eb1d-c9fa-46a9-bd3a-3f1b4b196287",
    "name": "read-only",
    "is_default": false
  }, {
    "comment": "Full access to all endpoints, across all workspaces—except RBAC Admin API",
    "created_at": 1556563122,
    "id": "7fdea5c8-2bfa-4aa9-9c21-7bb9e607186d",
    "name": "admin",
    "is_default": false
  }]
}
```

管理者の役割を作成または更新する
----------------

**エンドポイント** 
<div class="endpoint post">/admins/\{name\_or\_id\}/roles</div> 

|      属性      |                    説明                    |
|--------------|------------------------------------------|
| `name_or_id` | **管理者** の現在のユーザー名またはID                   |
| `roles`      | （カンマ区切り） **管理者** のために作成または更新するロールの名前の文字列 |

**レスポンス** 

    HTTP 201 OK

```json
{
  "roles": [{
    "comment": "Read access to all endpoints, across all workspaces",
    "created_at": 1556563122,
    "id": "7574eb1d-c9fa-46a9-bd3a-3f1b4b196287",
    "name": "read-only",
    "is_default": false
  }, {
    "comment": "Full access to all endpoints, across all workspaces—except RBAC Admin API",
    "created_at": 1556563122,
    "id": "7fdea5c8-2bfa-4aa9-9c21-7bb9e607186d",
    "name": "admin",
    "is_default": false
  }, {
    "comment": "Full access to all endpoints, across all workspaces",
    "created_at": 1556563122,
    "id": "99bd8d18-f5b6-410e-aefe-d75f4252f13c",
    "name": "super-admin",
    "is_default": false
  }]
}
```

管理者の役割の削除
---------

**エンドポイント** 
<div class="endpoint delete">/admins/\{name\_or\_id\}/roles</div> 

|      属性      |               説明                |
|--------------|---------------------------------|
| `name_or_id` | **管理者** の現在のユーザー名またはカスタムID      |
| `roles`      | **管理者** から削除するロール名の（カンマ区切りの）文字列 |

**レスポンス** 

    HTTP 204 No Content

管理者のワークスペースの一覧表示
----------------

**エンドポイント** 
<div class="endpoint get">/admins/\{name\_or\_id\}/workspaces</div> 

|      属性      |         説明          |
|--------------|---------------------|
| `name_or_id` | **管理者** のユーザー名またはID |

**レスポンス** 

    HTTP 200 OK

```json
[{
  "created_at": 1556563122,
  "config": {
    "portal": true,
    "portal_auto_approve": true
  },
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "default",
  "meta": {}
}, {
  "created_at": 1556570807,
  "config": {
    "portal": true
  },
  "id": "57b3ce24-6d29-427f-af13-15bd60430e56",
  "name": "sdfgsdfg",
  "meta": {
    "color": "#3894f0"
  }
}]
```

