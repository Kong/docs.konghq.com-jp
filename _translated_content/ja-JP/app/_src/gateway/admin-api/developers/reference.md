---
title: "開発者向けリファレンス"
badge: "enterprise"
---
Kong Admin APIを通じてDev Portalインスタンスを管理できます。`/developers` APIは次の目的で使用できます。

* 開発者とアプリケーションの管理
* ロールの作成、更新、削除
* 開発者とアプリケーションの認証の管理

{:.note}
> 
> **注：** `/developers` APIはKong Admin APIの一部であり、開発者の一括管理を目的としています。これは、ログインした開発者に関するデータを返すDev Portal API [`/developer`](/dev-portal/api/v1/#/developer/get-developer)エンドポイントとは異なります。

開発者
---

### すべての開発者をリストアップ

すべての開発者のメタデータを取得します。

**エンドポイント** 
<div class="endpoint get">/developers</div> 

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "data": [
        {
            "consumer": {
                "id": "3f0ad41f-39b5-4fd0-ad1e-0b779fcee35c"
            },
            "created_at": 1644615612,
            "email": "some-email@example.com",
            "id": "3c6d4d8c-db86-4236-800b-8911e788425b",
            "meta": "{\"full_name\":\"Barry\"}",
            "roles": [
                "application developer"
            ],
            "status": 0,
            "updated_at": 1644615612
        },
        {
            "consumer": {
                "id": "9dbd764c-9548-4c9f-a4d1-6ea11e32a803"
            },
            "created_at": 1644615583,
            "email": "some-other-email@example.com",
            "id": "5f60930a-ad12-4303-ac5a-59d121ad4942",
            "meta": "{\"full_name\":\"Diana\"}",
            "roles": [
                "application developer"
            ],
            "status": 0,
            "updated_at": 1644615583
        }
    ],
    "next": null,
    "total": 2
}
```

### 開発者アカウントを作成する

新しい開発者アカウントを作成します。

すべての新規開発者のステータスはデフォルトで **承認リクエスト済み（Requested Approval）** に設定されており、いかなるアカウントも使用前にステータスを 
**承認済み（Approved）** に変更する必要があります。

Kong Managerを通じて開発者を承認するか、開発者のステータスを手動で設定して開発者アカウントを作成するときに
承認することもできます。

**エンドポイント** 
<div class="endpoint post">/developers</div> 

**リクエストボディ** 

|                       属性 |                                                                                                  説明                                                                                                   |
|-------------------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                   `meta` | JSON形式のアカウントのメタデータ。Dev Portal設定で定義されたフィールドを受け入れます。<br><br>デフォルトでは、meta属性は`full_name`フィールドを必要とします。必要に応じて、この要件を削除したり、他のフィールドを追加したりすることができます。 <br><br>例：`meta: {"full_name":"<name id="sl-md0000000">"}` |
|         `email`<br> *必須* | 作成する開発者のメールアドレスで、ログインユーザー名として使用されます。                                                                                                                                                                  |
| `password`<br> *セミオプション* | 開発者のパスワードを作成します。ベーシック認証が有効になっている場合は必須です。                                                                                                                                                              |
|      `key`<br> *セミオプション* | 開発者に API キーを割り当てます。Key Authenticationが有効になっている場合に必須です。                                                                                                                                                |
|                     `id` | 開発者エンティティ ID。この値に独自のUUIDを設定することも、省略して{{site.base_gateway}}にUUIDを自動生成させることもできます。                                                                                                                                       |
|                 `status` | アカウントの承認ステータス。指定しない場合、ステータスはデフォルトで `1` に設定され、開発者は自動的に **リクエストされたアクセス** キューに入れられます。<br><br>次のいずれかの整数を受け入れます。<br> • `0` \- 承認済み<br>• `1` \- リクエストされたアクセス<br>• `2` \- 拒否<br>• `3` \- 取り消し            |

リクエストの例:

```sh
curl -i -X POST http://localhost:8001/developers \
  --data email=example@example.com \
  --data meta="{\"full_name\":\"Wally\"}" \
  --data password=mypass \
  --data id=62d17e63-0628-43a3-b936-97b8dcbd366f
```

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumer": {
        "id": "62d17e63-0628-43a3-b936-97b8dcbd366f"
    },
    "created_at": 1644616521,
    "email": "example@example.com",
    "id": "a82f9477-1a24-45b9-827c-23f1c1a7ebb3",
    "meta": "{\"full_name\":\"Wally\"}",
    "status": 0,
    "updated_at": 1644616521
}
```

### 開発者を招待

メールのリストに招待を送信します。
招待メールを送信するには、[SMTP](/gateway/{{page.release}}/kong-enterprise/dev-portal/smtp/)
が有効になっている必要があります。

**エンドポイント** 
<div class="endpoint post">/developers/invite</div> 

**リクエストボディ** 

|       属性 |          説明          |
|---------:|----------------------|
| `emails` | カンマ区切りで配列されたメールアドレス。 |

リクエストの例:

```sh
curl -i -X POST http://localhost:8001/developers/invite \
  --data "emails[]=example@example.com" \
  --data "emails[]=example2@example.com"
```

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "error": {
        "count": 0,
        "emails": {}
    },
    "sent": {
        "count": 2,
        "emails": {
            "example@example.com": true,
            "example2@example.com": true
        }
    },
    "smtp_mock": true
}
```

### 開発者を検査

特定の開発者アカウントのメタデータを取得します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}</div> 

|                               属性 |          説明          |
|---------------------------------:|----------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 検査したい開発者のメールまたはUUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumer": {
        "id": "855fe098-3bb3-4041-8a3d-149a9e5462c6"
    },
    "created_at": 1644617754,
    "email": "example@example.com",
    "id": "f94e4fbd-c30b-41d7-b349-13c8f4fc23ca",
    "rbac_user": {
        "id": "7d40540b-4b65-4fa6-b04e-ffe6c2c40b0d"
    },
    "roles": [
        "QA"
    ],
    "status": 0,
    "updated_at": 1644619493
}
```

### 開発者を更新

特定の開発者アカウントの設定またはステータスを更新します。

**エンドポイント** 
<div class="endpoint patch">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}</div> 

|                               属性 |          説明          |
|---------------------------------:|----------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 更新したい開発者のメールまたはUUID。 |

**リクエストボディ** 

|       属性 |                                                                                                  説明                                                                                                   |
|---------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   `meta` | JSON形式のアカウントのメタデータ。Dev Portal設定で定義されたフィールドを受け入れます。<br><br>デフォルトでは、meta属性は`full_name`フィールドを必要とします。必要に応じて、この要件を削除したり、他のフィールドを追加したりすることができます。 <br><br>例：`meta: {"full_name":"<name id="sl-md0000000">"}` |
|  `email` | 作成する開発者のメールアドレスで、ログインユーザー名として使用されます。                                                                                                                                                                  |
| `status` | アカウントの承認ステータス。 <br><br>次のいずれかの整数を受け入れます:<br> • `0` \- 承認済み<br>• `1` \- リクエストされたアクセス<br>• `2` \- 拒否<br>• `3` \- 取り消し                                                                               |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "developer": {
        "consumer": {
            "id": "855fe098-3bb3-4041-8a3d-149a9e5462c6"
        },
        "created_at": 1644617754,
        "email": "example@example.com",
        "id": "f94e4fbd-c30b-41d7-b349-13c8f4fc23ca",
        "rbac_user": {
            "id": "7d40540b-4b65-4fa6-b04e-ffe6c2c40b0d"
        },
        "roles": [
            "QA"
        ],
        "status": 0,
        "updated_at": 1644621148
    }
}
```

### 開発者を削除

特定の開発者アカウントを削除します。

**エンドポイント** 
<div class="endpoint delete">/developers/\{DEVELOPER\_ID\}</div> 

|               属性 |       説明       |
|-----------------:|----------------|
| `{DEVELOPER_ID}` | 削除したい開発者のUUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

開発者メタデータをエクスポート
---------------

開発者のリストをCSV形式で印刷します。
<div class="endpoint get">/developers/export</div> 

**レスポンス** 

    HTTP/1.1 200 OK

```csv
Email, Status
test@example.com,APPROVED
test1@example.com,PENDING
test2@example.com,REVOKED
test3@example.com,REJECTED
```

ロール
---

### すべてのDev Portalロールを一覧表示する

Dev Portal 用に構成されたすべての RBAC 役割を一覧表示します。

デフォルトでは、ロールはありません。すべての Dev Portal ロールはカスタムです。

**エンドポイント** 
<div class="endpoint get">/developers/roles</div> 

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "data": [
        {
            "created_at": 1644615388,
            "id": "525694de-776c-43ad-a940-769295d0c54e",
            "name": "Application Developer"
        },
        {
            "created_at": 1644619493,
            "id": "c8535760-e37f-4578-921f-04e781a3f8f1",
            "name": "QA"
        }
    ],
    "next": null,
    "total": 2
}
```

### Dev Portalロールを作成する

開発者をグループ化するためのRBACロールを作成します。

ロールを作成すると、デフォルトではそのロールには特定の権限はなく、Dev Portal内のすべてのコンテンツにアクセスできます。Kong Managerの **Dev Portal** > **権限** ページ（ `<kong-manager-host:port id="sl-md0000000">/<workspace-name id="sl-md0000000">/portal/permissions/#roles` ）を使用して、ロールの権限を指定します。

**エンドポイント** 
<div class="endpoint post">/developers/roles</div> 

**リクエストボディ** 

|              属性 |   説明    |
|----------------:|---------|
| `name`<br> *必須* | ロールの名前。 |
|       `comment` | 役割の説明。  |

**レスポンス** 

    HTTP/1.1 201 Created

```json
{
    "comment": "Billing services team",
    "created_at": 1644620475,
    "id": "b7794758-394e-46f1-a9a1-f495f2e97a68",
    "name": "Billing",
    "permissions": {}
}
```

### ロールの検査

特定のDev Portal RBACロールを検査します。

**エンドポイント** 
<div class="endpoint get">/developers/roles/\{ROLE\_NAME\|ROLE\_ID\}</div> 

|                             属性 |         説明         |
|-------------------------------:|--------------------|
| `{ROLE_NAME|ROLE_ID}`<br> *必須* | 検査するロールの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "comment": "Billing services team",
    "created_at": 1644620475,
    "id": "b7794758-394e-46f1-a9a1-f495f2e97a68",
    "name": "Billing",
    "permissions": {}
}
```

### ロールの更新

特定のDev Portal RBACロールをアップデートします。

**エンドポイント** 
<div class="endpoint patch">/developers/roles/\{ROLE\_NAME\|ROLE\_ID\}</div> 

|                             属性 |         説明         |
|-------------------------------:|--------------------|
| `{ROLE_NAME|ROLE_ID}`<br> *必須* | 検査するロールの名前またはUUID。 |

**リクエストボディ** 

|        属性 |   説明    |
|----------:|---------|
|    `name` | ロールの名前。 |
| `comment` | 役割の説明。  |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "comment": "Billing services team",
    "created_at": 1644620475,
    "id": "b7794758-394e-46f1-a9a1-f495f2e97a68",
    "name": "Billing",
    "permissions": {}
}
```

### ロールを削除する

特定の Dev Portal RBAC 役割を削除します。

**エンドポイント** 
<div class="endpoint delete">/developers/roles/\{ROLE\_NAME\|ROLE\_ID\}</div> 

|                             属性 |         説明         |
|-------------------------------:|--------------------|
| `{ROLE_NAME|ROLE_ID}`<br> *必須* | 検査するロールの名前またはUUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

アプリケーション
--------

アプリケーションはアプリケーションレベルの認証を介して{{site.base_gateway}}のサービスを利用します。開発者、つまりデベロッパーポータルにログインするペルソナは、Dev Portalでアプリケーションを作成し管理します。

管理者はまず[アプリケーション登録の有効化](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration/)手順を完了させて、開発者がサービスをアプリケーションと関連付けられるようにする必要があります。

### 開発者向けのすべてのアプリケーションを一覧表示する

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "data": [
        {
            "consumer": {
                "id": "6e31bf1e-dbcb-4a31-bac9-a192fa24f088"
            },
            "created_at": 1644963627,
            "developer": {
                "id": "5f60930a-ad12-4303-ac5a-59d121ad4942"
            },
            "id": "5ff48aaf-3951-4c99-a636-3b682081705c",
            "name": "example_app",
            "redirect_uri": "http://httpbin.org",
            "updated_at": 1644963627
        },
        {
            "consumer": {
                "id": "2390fd02-bbcd-48f1-b32f-89c262fa68a8"
            },
            "created_at": 1644963657,
            "developer": {
                "id": "5f60930a-ad12-4303-ac5a-59d121ad4942"
            },
            "id": "94e0f633-e8fd-4647-a0cd-4c3015ff2722",
            "name": "example_app2",
            "redirect_uri": "http://httpbin.org",
            "updated_at": 1644963657
        }
    ],
    "next": null,
    "total": 2
}

```

### アプリケーションの検査

アプリケーションIDを使用して特定のアプリケーションを検査します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumer": {
        "id": "d5ea9628-f359-4b96-9f76-61562aaf4300"
    },
    "created_at": 1644965555,
    "custom_id": "billing",
    "developer": {
        "id": "5f60930a-ad12-4303-ac5a-59d121ad4942"
    },
    "id": "ca0d62bd-4616-4b87-b947-43e33e5418f0",
    "name": "testapp",
    "redirect_uri": "http://httpbin.org",
    "updated_at": 1644965555
}
```

### アプリケーションの作成

特定の開発者向けの新しいアプリケーションを作成します。

**エンドポイント** 
<div class="endpoint post">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |

**リクエストボディ** 

|                  属性 |                                                                説明                                                                 |
|--------------------:|-----------------------------------------------------------------------------------------------------------------------------------|
|         `name` *必須* | アプリケーションの名前。                                                                                                                      |
| `redirect_uri` *必須* | アプリケーションのURI。                                                                                                                     |
|         `custom_id` | アプリケーションのカスタム名。`custom_id`は、アプリケーションにリンクされたコンシューマエンティティに保存されます。これは、アプリケーションの `openid-connect` 認証プラグインを設定する際のOIDCクレームマッピングに使用できます。 |

**レスポンス** 

    HTTP/1.1 201 Created

```json
{
    "consumer": {
        "id": "d5ea9628-f359-4b96-9f76-61562aaf4300"
    },
    "created_at": 1644965555,
    "custom_id": "billing-app",
    "developer": {
        "id": "5f60930a-ad12-4303-ac5a-59d121ad4942"
    },
    "id": "ca0d62bd-4616-4b87-b947-43e33e5418f0",
    "name": "testapp",
    "redirect_uri": "http://httpbin.org",
    "updated_at": 1644965555
}
```

### アプリケーションを更新

アプリケーション ID を使用して特定のアプリケーションを更新します。

**エンドポイント** 
<div class="endpoint patch">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**リクエストボディ** 

|             属性 |        説明         |
|---------------:|-------------------|
|         `name` | アプリケーションの名前。      |
| `redirect_uri` | アプリケーションのURI。     |
|    `custom_id` | アプリケーションのカスタム識別子。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "consumer": {
        "id": "6e31bf1e-dbcb-4a31-bac9-a192fa24f088"
    },
    "created_at": 1644963627,
    "developer": {
        "id": "5f60930a-ad12-4303-ac5a-59d121ad4942"
    },
    "id": "5ff48aaf-3951-4c99-a636-3b682081705c",
    "name": "ExampleApp",
    "redirect_uri": "http://httpbin.org",
    "updated_at": 1645575611
}
```

### アプリケーションを削除する

アプリケーションIDを使用して特定のアプリケーションを削除します。

**エンドポイント** 
<div class="endpoint delete">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**レスポンス** 

    HTTP/1.1 204 No Content

### アプリケーションのすべてのインスタンスの表示

{{site.base_gateway}}のサービスに接続されているすべてのアプリケーションインスタンスを表示します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/application\_instances</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "data": [
        {
            "application": {
                "consumer": {
                    "id": "a4e8f835-e5bb-498c-8b6a-73a21c82776d"
                },
                "created_at": 1644965487,
                "developer": {
                    "consumer": {
                        "id": "9dbd764c-9548-4c9f-a4d1-6ea11e32a803"
                    },
                    "created_at": 1644615583,
                    "email": "example@example.com",
                    "id": "5f60930a-ad12-4303-ac5a-59d121ad4942",
                    "meta": "{\"full_name\":\"Wally\"}",
                    "rbac_user": {
                        "id": "431cae60-5d60-40be-8636-1b349a70a88d"
                    },
                    "status": 0,
                    "updated_at": 1644615874
                },
                "id": "645682ae-0be6-420a-bcf3-0e711a391546",
                "name": "testapp",
                "redirect_uri": "http://httpbin.org",
                "updated_at": 1644965487
            },
            "composite_id": "645682ae-0be6-420a-bcf3-0e711a391546_212a758a-810b-4226-9175-b1b44eecebec",
            "created_at": 1644968368,
            "id": "13fdfd32-8412-4479-b04f-a83d155e5de5",
            "service": {
                "id": "212a758a-810b-4226-9175-b1b44eecebec"
            },
            "status": 0,
            "suspended": false,
            "updated_at": 1644968411
        }
    ],
    "next": null,
    "total": 1
}

```

### アプリケーションインスタンスの作成

{{site.base_gateway}}のサービスにアプリケーションを接続します。

**エンドポイント** 
<div class="endpoint post">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/application\_instances</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**リクエストボディ** 

|           属性 |           説明            |
|-------------:|-------------------------|
| `service.id` | アプリケーションを接続するサービスのUUID。 |

**レスポンス** 

    HTTP/1.1 201 Created

```json
{
    "application": {
        "id": "5ff48aaf-3951-4c99-a636-3b682081705c"
    },
    "composite_id": "5ff48aaf-3951-4c99-a636-3b682081705c_212a758a-810b-4226-9175-b1b44eecebec",
    "created_at": 1645570372,
    "id": "50193ee0-372a-4694-874c-90ffbc0ae522",
    "service": {
        "id": "212a758a-810b-4226-9175-b1b44eecebec"
    },
    "status": 1,
    "suspended": false,
    "updated_at": 1645570372
}
```

### アプリケーションインスタンスを検査

アプリケーションの特定のインスタンスに関する情報を取得します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/application\_instances/\{APPLICATION\_INSTANCE\_ID\}</div> 

|                               属性 |         説明          |
|---------------------------------:|---------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。    |
|               `{APPLICATION_ID}` | アプリケーションのUUID。      |
|      `{APPLICATION_INSTANCE_ID}` | アプリケーションインスタンスUUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "application": {
        "id": "645682ae-0be6-420a-bcf3-0e711a391546"
    },
    "composite_id": "645682ae-0be6-420a-bcf3-0e711a391546_212a758a-810b-4226-9175-b1b44eecebec",
    "created_at": 1644968368,
    "id": "13fdfd32-8412-4479-b04f-a83d155e5de5",
    "service": {
        "id": "212a758a-810b-4226-9175-b1b44eecebec"
    },
    "status": 0,
    "suspended": false,
    "updated_at": 1644968411
}
```

### アプリケーションインスタンスの更新

アプリケーションの特定のインスタンスを更新します。

**エンドポイント** 
<div class="endpoint patch">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/application\_instances/\{APPLICATION\_INSTANCE\_ID\}</div> 

|                               属性 |         説明          |
|---------------------------------:|---------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。    |
|               `{APPLICATION_ID}` | アプリケーションのUUID。      |
|      `{APPLICATION_INSTANCE_ID}` | アプリケーションインスタンスUUID。 |

**リクエストボディ** 

|           属性 |           説明            |
|-------------:|-------------------------|
| `service.id` | アプリケーションを接続するサービスのUUID。 |

**レスポンス** 

    HTTP/1.1 201 OK

```json
{
    "application": {
        "id": "645682ae-0be6-420a-bcf3-0e711a391546"
    },
    "composite_id": "645682ae-0be6-420a-bcf3-0e711a391546_212a758a-810b-4226-9175-b1b44eecebec",
    "created_at": 1644968368,
    "id": "13fdfd32-8412-4479-b04f-a83d155e5de5",
    "service": {
        "id": "212a758a-810b-4226-9175-b1b44eecebec"
    },
    "status": 0,
    "suspended": false,
    "updated_at": 1644968411
}
```

### アプリケーションインスタンスを削除

アプリケーションをサービスから切断します。

**エンドポイント** 
<div class="endpoint delete">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/application\_instances/\{APPLICATION\_INSTANCE\_ID\}</div> 

|                               属性 |         説明          |
|---------------------------------:|---------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。    |
|               `{APPLICATION_ID}` | アプリケーションのUUID。      |
|      `{APPLICATION_INSTANCE_ID}` | アプリケーションインスタンスUUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

プラグイン
-----

### すべてのプラグインをの一覧表示

開発者向けのすべてのプラグインを一覧表示します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/plugins</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |

**レスポンス** 

    HTTP/1.1 200 OK

正確な応答は、この開発者が構成したプラグインによって異なります。
たとえば、`proxy-cache`プラグインが有効になっている場合は、以下の応答が返されます。

```json
{
    "config": {
        "cache_control": false,
        "cache_ttl": 300,
        "content_type": [
            "text/plain",
            "application/json"
        ],
        "memory": {
            "dictionary_name": "kong_db_cache"
        },
        "request_method": [
            "GET",
            "HEAD"
        ],
        "response_code": [
            200,
            301,
            404
        ],
        "strategy": "memory"
    },
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645564279,
    "enabled": true,
    "id": "d5f70fe4-58e6-4153-8569-07a1f901e751",
    "name": "proxy-cache",
    "protocols": [
        "grpc",
        "grpcs",
        "http",
        "https"
    ]
}
```

### プラグインを適用する

プラグインを構成して開発者に適用します。

**エンドポイント** 
<div class="endpoint post">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/plugins</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |

**リクエストボディ** 

|           属性 |                                        説明                                        |
|-------------:|----------------------------------------------------------------------------------|
|  `name` *必須* | 有効にするプラグインの名前。コンシューマで有効にできる任意のプラグインを使用できます。                                      |
| プラグイン構成フィールド | 有効にするプラグインの構成パラメータ。特定のプラグインのパラメータを見つけるには、[Plugin Hub](/hub/)のプラグインのリファレンスを参照します。 |

リクエストの例:

```sh
curl -i -X POST http://localhost:8001/developers/example@example.com/plugins \
  --data name=proxy-cache \
  --data config.strategy=memory
```

**レスポンス** 

    HTTP/1.1 201 Created

正確な応答はプラグインによって異なります。たとえば`proxy-cache`の場合は、次のようになります。

```json
{
    "config": {
        "cache_control": false,
        "cache_ttl": 300,
        "content_type": [
            "text/plain",
            "application/json"
        ],
        "memory": {
            "dictionary_name": "kong_db_cache"
        },
        "request_method": [
            "GET",
            "HEAD"
        ],
        "response_code": [
            200,
            301,
            404
        ],
        "strategy": "memory"
    },
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645564279,
    "enabled": true,
    "id": "d5f70fe4-58e6-4153-8569-07a1f901e751",
    "name": "proxy-cache",
    "protocols": [
        "grpc",
        "grpcs",
        "http",
        "https"
    ]
}
```

### 開発者プラグインを調査する

開発者に適用されているプラグインを検査します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/plugins/\{PLUGIN\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|                    `{PLUGIN_ID}` | 検査するプラグインのUUID。  |

**レスポンス** 

    HTTP/1.1 200 OK

正確な応答はプラグインによって異なります。たとえば`proxy-cache`の場合は、次のようになります。

```json
{
    "config": {
        "cache_control": false,
        "cache_ttl": 300,
        "content_type": [
            "text/plain",
            "application/json"
        ],
        "memory": {
            "dictionary_name": "kong_db_cache"
        },
        "request_method": [
            "GET",
            "HEAD"
        ],
        "response_code": [
            200,
            301,
            404
        ],
        "strategy": "memory"
    },
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645564279,
    "enabled": true,
    "id": "d5f70fe4-58e6-4153-8569-07a1f901e751",
    "name": "proxy-cache",
    "protocols": [
        "grpc",
        "grpcs",
        "http",
        "https"
    ]
}
```

### 開発者プラグインの更新

開発者に適用されているプラグインを編集します。

**エンドポイント** 
<div class="endpoint patch">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/plugins/\{PLUGIN\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|                    `{PLUGIN_ID}` | 更新するプラグインのUUID。  |

**リクエストボディ** 

|           属性 |                                          説明                                          |
|-------------:|--------------------------------------------------------------------------------------|
| プラグイン構成フィールド | 編集しているプラグインの構成パラメータ。特定のプラグインのパラメータを見つけるには、[Plugin Hub](/hub/)のプラグインのリファレンスを参照してください。 |

`proxy-cache`プラグインのインスタンスを更新するリクエストの例：

```sh
http PATCH :8001/developers/91a1fb59-d90f-4f07-b609-4c2a1e3d847e/plugins/d5f70fe4-58e6-4153-8569-07a1f901e751 \
  config.response_code:='[200, 301, 400, 404]'
```

**レスポンス** 

    HTTP/1.1 200 OK

正確な応答はプラグインによって異なります。たとえば、
`proxy-cache`の受け入れられる応答コードを変更して`400`を追加する場合:

```json
{
    "config": {
        "cache_control": false,
        "cache_ttl": 300,
        "content_type": [
            "text/plain",
            "application/json"
        ],
        "memory": {
            "dictionary_name": "kong_db_cache"
        },
        "request_method": [
            "GET",
            "HEAD"
        ],
        "response_code": [
            200,
            301,
            400,
            404
        ],
        "strategy": "memory"
    },
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645564279,
    "enabled": true,
    "id": "d5f70fe4-58e6-4153-8569-07a1f901e751",
    "name": "proxy-cache",
    "protocols": [
        "grpc",
        "grpcs",
        "http",
        "https"
    ]
}
```

### 開発者プラグインを削除する

開発者から認証プラグインを削除します。

**エンドポイント** 
<div class="endpoint delete">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/plugins/\{PLUGIN\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|                    `{PLUGIN_ID}` | 削除するプラグインのUUID。  |

**レスポンス** 

    HTTP/1.1 204 No Content

開発者認証
-----

開発者は、次のいずれかの認証方法を使用してDev Portalに認証できます。

* [`basic-auth`](/hub/kong-inc/basic-auth/)
* [`oauth2`](/hub/kong-inc/oauth2/)
* [`hmac-auth`](/hub/kong-inc/hmac-auth/)
* [`jwt`](/hub/kong-inc/jwt/)
* [`key-auth`](/hub/kong-inc/key-auth/)
* [`openid-connect`](/hub/kong-inc/openid-connect/)

これらの各メソッドは、関連する {{site.base_gateway}}
認証プラグインのインスタンスを構成します。

開発者がアプリケーションを管理できるようにするには、Dev Portalである種の認証を
使用し、各開発者にDev Portalへのアクセスを付与する必要があります。
認証が無効で、Dev PortalのURLが公開されている場合、アプリケーションの
登録はできません。

### すべての認証情報を取得する

特定の種類の認証プラグインに対して構成されたすべてのDev Portal認証情報のリストを取得します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/credentials/\{PLUGIN\_NAME\}</div> 

|                               属性 |                                                                   説明                                                                    |
|---------------------------------:|-----------------------------------------------------------------------------------------------------------------------------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。                                                                                                                        |
|                  `{PLUGIN_NAME}` | サポートされている認証プラグインの名前。次のいずれかになります。<br> • `basic-auth`<br> • `oauth2`<br> • `hmac-auth`<br> • `jwt`<br> • `key-auth`<br> •`openid-connect` |

**レスポンス** 

    HTTP/1.1 200 OK

正確な応答はプラグインによって異なります。たとえば、`key-auth`認証情報が2つある場合は以下の応答が返されます。

```json
{
    "data": [
        {
            "consumer": {
                "id": "bc504999-b3e5-4b08-8544-86962c969335"
            },
            "created_at": 1645566606,
            "id": "1a86d7eb-53ce-4a29-b573-a545e602fc05",
            "key": "testing"
        },
        {
            "consumer": {
                "id": "bc504999-b3e5-4b08-8544-86962c969335"
            },
            "created_at": 1645567499,
            "id": "1b0ba8f4-dd9f-4f68-8108-c81952e05b8e",
            "key": "wpJqvsTFRjgQstL16pQTYKtlHfkzdJpb"
        }
    ],
    "next": null,
    "total": 2
}
```

### 認証情報を作成

Dev Portal の認証情報を作成します。作成できる認証情報は、Dev Portalで現在有効になっている認証のタイプに依存します。

**エンドポイント** 
<div class="endpoint post">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/credentials/\{PLUGIN\_NAME\}</div> 

|                               属性 |                                                                   説明                                                                    |
|---------------------------------:|-----------------------------------------------------------------------------------------------------------------------------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。                                                                                                                        |
|                  `{PLUGIN_NAME}` | サポートされている認証プラグインの名前。次のいずれかになります。<br> • `basic-auth`<br> • `oauth2`<br> • `hmac-auth`<br> • `jwt`<br> • `key-auth`<br> •`openid-connect` |

**リクエストボディ** 

|           属性 |                                                                                                                                                                        説明                                                                                                                                                                         |
|-------------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| プラグイン構成フィールド | 構成しているプラグインの認証資格情報。特定のプラグインのパラメータを見つけるには、プラグインのリファレンスドキュメントを参照してください。<br> • [`basic-auth`](/hub/kong-inc/basic-auth/)<br> • [`oauth2`](/hub/kong-inc/oauth2/)<br> • [`hmac-auth`](/hub/kong-inc/hmac-auth/)<br> • [`jwt`](/hub/kong-inc/jwt/)<br> • [`key-auth`](/hub/kong-inc/key-auth/)<br> • [`openid-connect`](/hub/kong-inc/openid-connect/) |

`key-auth`の認証情報を作成するリクエストの例：

```sh
curl -i -X POST http://localhost:8001/developers/91a1fb59-d90f-4f07-b609-4c2a1e3d847e/credentials/key-auth
```

**レスポンス** 

    HTTP/1.1 201 Created

正確な応答はプラグインによって異なります。たとえば、`key-auth`の認証情報を追加すると、次のようになります。

```json
{
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645567499,
    "id": "1b0ba8f4-dd9f-4f68-8108-c81952e05b8e",
    "key": "wpJqvsTFRjgQstL16pQTYKtlHfkzdJpb"
}

```

### 認証情報を検査する

IDを使用して特定のDev Portal認証情報に関する情報を取得します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/credentials/\{PLUGIN\_NAME\}/\{CREDENTIAL\_ID\}</div> 

|                               属性 |                                                                   説明                                                                    |
|---------------------------------:|-----------------------------------------------------------------------------------------------------------------------------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。                                                                                                                        |
|                  `{PLUGIN_NAME}` | サポートされている認証プラグインの名前。次のいずれかになります。<br> • `basic-auth`<br> • `oauth2`<br> • `hmac-auth`<br> • `jwt`<br> • `key-auth`<br> •`openid-connect` |
|                `{CREDENTIAL_ID}` | プラグインの認証情報のUUID。                                                                                                                        |

**レスポンス** 

    HTTP/1.1 200 OK

正確なレスポンスはプラグインによって異なります。たとえば、`key-auth` 
認証情報を調査する場合、次のようになります。

```json
{
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645567499,
    "id": "1b0ba8f4-dd9f-4f68-8108-c81952e05b8e",
    "key": "wpJqvsTFRjgQstL16pQTYKtlHfkzdJpb"
}
```

### 認証情報の更新

特定のDev Portal認証情報を更新します。

**エンドポイント** 
<div class="endpoint patch">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/credentials/\{PLUGIN\_NAME\}/\{CREDENTIAL\_ID\}</div> 

|                               属性 |                                                                    説明                                                                    |
|---------------------------------:|------------------------------------------------------------------------------------------------------------------------------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。                                                                                                                         |
|                  `{PLUGIN_NAME}` | サポートされている認証プラグインの名前。次のいずれかになります。<br> • `basic-auth`<br> • `oauth2`<br> • `hmac-auth`<br> • `jwt`<br> • `key-auth`<br> • `openid-connect` |
|                `{CREDENTIAL_ID}` | プラグインの認証情報のUUID。                                                                                                                         |

**リクエストボディ** 

|           属性 |                                                                                                                                                                        説明                                                                                                                                                                         |
|-------------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| プラグイン構成フィールド | 構成しているプラグインの認証資格情報。特定のプラグインのパラメータを見つけるには、プラグインのリファレンスドキュメントを参照してください。<br> • [`basic-auth`](/hub/kong-inc/basic-auth/)<br> • [`oauth2`](/hub/kong-inc/oauth2/)<br> • [`hmac-auth`](/hub/kong-inc/hmac-auth/)<br> • [`jwt`](/hub/kong-inc/jwt/)<br> • [`key-auth`](/hub/kong-inc/key-auth/)<br> • [`openid-connect`](/hub/kong-inc/openid-connect/) |

`key-auth`キーを更新するリクエストの例:

    http PATCH :8001/developers/91a1fb59-d90f-4f07-b609-4c2a1e3d847e/credentials/key-auth/1b0ba8f4-dd9f-4f68-8108-c81952e05b8e \
      key=apikey

**レスポンス** 

    HTTP/1.1 200 OK

正確なレスポンスはプラグインによって異なります。たとえば、UUID の代わりにカスタム キーを使用するために `key-auth` 
認証情報を更新すると、次のようになります。

```json
{
    "consumer": {
        "id": "bc504999-b3e5-4b08-8544-86962c969335"
    },
    "created_at": 1645567499,
    "id": "1b0ba8f4-dd9f-4f68-8108-c81952e05b8e",
    "key": "apikey"
}
```

### 認証情報を削除する

**エンドポイント** 
<div class="endpoint delete">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/credentials/\{PLUGIN\_NAME\}/\{CREDENTIAL\_ID\}</div> 

|                               属性 |                                                                   説明                                                                    |
|---------------------------------:|-----------------------------------------------------------------------------------------------------------------------------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。                                                                                                                        |
|                  `{PLUGIN_NAME}` | サポートされている認証プラグインの名前。次のいずれかになります。<br> • `basic-auth`<br> • `oauth2`<br> • `hmac-auth`<br> • `jwt`<br> • `key-auth`<br> •`openid-connect` |
|                `{CREDENTIAL_ID}` | プラグインの認証情報のUUID。                                                                                                                        |

**レスポンス** 

    HTTP/1.1 204 No Content

アプリケーション認証
----------

アプリケーション登録を有効にすると、[認証ストラテジ](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/auth-provider-strategy)が必要になります。デフォルトでは、このストラテジは`kong-oauth2`であり、 `kong.conf`で設定されます。

    portal_app_auth = kong-oauth2

デフォルトのストラテジを選択する場合、次のAPIを使用してアプリケーションの認証を構成できます。
`external-oauth2`ストラテジを選択する場合、[IdPを通じて認証を管理](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/auth-provider-strategy)します。

### アプリケーションのすべての認証情報を検査する

特定の種類の認証プラグインのすべてのアプリケーション認証情報のリストを取得します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/credentials/oauth2</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "data": [
        {
            "client_id": "3psqvdiojUKoJudqCh1U5P5mvR9k3zFF",
            "client_secret": "HUbYpUw8sNrEZGqyCs19HmRo08l9shC0",
            "client_type": "confidential",
            "consumer": {
                "id": "6e31bf1e-dbcb-4a31-bac9-a192fa24f088"
            },
            "created_at": 1644963627,
            "hash_secret": false,
            "id": "e22760bc-c6d4-4572-9814-132825286618",
            "name": "example_app",
            "redirect_uris": [
                "http://httpbin.org"
            ]
        }
    ],
    "next": null,
    "total": 1
}
```

### アプリケーションの認証情報の作成

アプリケーションの OAuth2 認証資格を作成します。
このリクエストは、 [OAuth2 プラグイン](/hub/kong-inc/oauth2/)のインスタンスを構成します。

**エンドポイント** 
<div class="endpoint post">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/credentials/oauth2</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |

**リクエストボディ** 

|              属性 |                                                             説明                                                              |
|----------------:|-----------------------------------------------------------------------------------------------------------------------------|
|     `client_id` | オプションで独自の`client_id`を設定することもできます。提供されない場合は、プラグインが生成します。                                                                     |
| `client_secret` | オプションで独自の`client_secret`を設定することもできます。提供されない場合は、プラグインが生成します。                                                                 |
| `redirect_uris` | 1つ以上のURLを含むアプリ内の配列で、ユーザーが認証後に送信されます（[RFC 6742 Section 3\.1\.2](https://tools.ietf.org/html/rfc6749#section-3.1.2)）。       |
|   `hash_secret` | `client_secret`フィールドがハッシュ形式で保存されるかどうかを示すブールフラグ。既存のプラグインインスタンスで有効にすると、クライアントシークレットは最初に使用したときにその場でハッシュされます。<br>デフォルト： `false` |

`oauth2` の認証情報を作成するリクエストの例は以下のとおりです。

```sh
curl -i -X POST http://localhost:8001/developers/5f60930a-ad12-4303-ac5a-59d121ad4942/applications/5ff48aaf-3951-4c99-a636-3b682081705c/credentials/oauth2 \ 
  --data client_id=myclient \
  --data client_secret=mysecret
```

**レスポンス** 

    HTTP/1.1 201 Created

```json
{
    "client_id": "t2vTlxxV1ANi9jFq2U1rlR1yp03Mi6uf",
    "client_secret": "EdKxXDD97LSe45lG5G5f6CBn5G9gXcp4",
    "client_type": "confidential",
    "consumer": {
        "id": "6e31bf1e-dbcb-4a31-bac9-a192fa24f088"
    },
    "created_at": 1645569439,
    "hash_secret": false,
    "id": "2dc8ee2a-ecfd-4ca5-a9c1-db371480e2cf",
    "name": "example_app",
    "redirect_uris": [
        "http://httpbin.org"
    ]
}
```

### アプリケーションの認証情報を検査

アプリケーションのOAuth2認証の認証情報を検査します。

**エンドポイント** 
<div class="endpoint get">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/credentials/oauth2/\{CREDENTIAL\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|               `{APPLICATION_ID}` | アプリケーションのUUID。   |
|                `{CREDENTIAL_ID}` | 認証情報のUUID。       |

**レスポンス** 

    HTTP/1.1 200 OK

```json
{
    "client_id": "3psqvdiojUKoJudqCh1U5P5mvR9k3zFF",
    "client_secret": "HUbYpUw8sNrEZGqyCs19HmRo08l9shC0",
    "client_type": "confidential",
    "consumer": {
        "id": "6e31bf1e-dbcb-4a31-bac9-a192fa24f088"
    },
    "created_at": 1644963627,
    "hash_secret": false,
    "id": "e22760bc-c6d4-4572-9814-132825286618",
    "name": "example_app",
    "redirect_uris": [
        "http://httpbin.org"
    ]
}
```

### アプリケーションからの認証情報の削除

アプリケーションの OAuth2 認証の認証情報を削除します。

**エンドポイント** 
<div class="endpoint delete">/developers/\{DEVELOPER\_EMAIL\|DEVELOPER\_ID\}/applications/\{APPLICATION\_ID\}/credentials/oauth2/\{CREDENTIAL\_ID\}</div> 

|                               属性 |        説明        |
|---------------------------------:|------------------|
| `{DEVELOPER_EMAIL|DEVELOPER_ID}` | 開発者のメールまたは UUID。 |
|                `{CREDENTIAL_ID}` | プラグインの認証情報のUUID。 |

**レスポンス** 

    HTTP/1.1 204 No Content

