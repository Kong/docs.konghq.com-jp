---
title: "ワークスペースのリファレンス"
badge: "enterprise"
workspace_body: "属性名 | 説明\n---:| ---\n  name  | ワークスペース名。\n"
---

{{site.base_gateway}}のワークスペース機能はKongの
Admin APIを通じて構成できます。

ワークスペースオブジェクトは、IDと名前を持つワークスペースエンティティを記述します。

ワークスペースのユースケースと構成例については、 [ワークスペースの例](/gateway/{{page.release}}/kong-enterprise/workspaces/)を参照してください。

ワークスペースの追加
----------

**エンドポイント** 
<div class="endpoint post">/workspaces/</div> 

**リクエストボディ** 


{{ page.workspace_body }}

**応答** 

    HTTP 201 Created

```json
{
  "comment": null,
  "config": {
    "meta": null,
    "portal": false,
    "portal_access_request_email": null,
    "portal_approved_email": null,
    "portal_auth": null,
    "portal_auth_conf": null,
    "portal_auto_approve": null,
    "portal_cors_origins": null,
    "portal_developer_meta_fields": "[{\"label\":\"Full Name\",\"title\":\"full_name\",\"validator\":{\"required\":true,\"type\":\"string\"}}]",
    "portal_emails_from": null,
    "portal_emails_reply_to": null,
    "portal_invite_email": null,
    "portal_reset_email": null,
    "portal_reset_success_email": null,
    "portal_token_exp": null
  },
  "created_at": 1557441226,
  "id": "c663cca5-c6f6-474a-ae44-01f62aba16a9",
  "meta": {
    "color": null,
    "thumbnail": null
  },
  "name": "green-team"
}
```

ワークスペースを一覧化
-----------

**エンドポイント** 
<div class="endpoint get">/workspaces/</div> 

**応答** 

    HTTP 200 OK

```json
{
  "data": [
    {
      "comment": null,
      "config": {
        "meta": null,
        "portal": false,
        "portal_access_request_email": null,
        "portal_approved_email": null,
        "portal_auth": null,
        "portal_auth_conf": null,
        "portal_auto_approve": null,
        "portal_cors_origins": null,
        "portal_developer_meta_fields": "[{\"label\":\"Full Name\",\"title\":\"full_name\",\"validator\":{\"required\":true,\"type\":\"string\"}}]",
        "portal_emails_from": null,
        "portal_emails_reply_to": null,
        "portal_invite_email": null,
        "portal_reset_email": null,
        "portal_reset_success_email": null,
        "portal_token_exp": null
      },
      "created_at": 1557419951,
      "id": "00000000-0000-0000-0000-000000000000",
      "meta": {
        "color": null,
        "thumbnail": null
      },
      "name": "default"
    },
    {
      "comment": null,
      "config": {
        "meta": null,
        "portal": false,
        "portal_access_request_email": null,
        "portal_approved_email": null,
        "portal_auth": null,
        "portal_auth_conf": null,
        "portal_auto_approve": null,
        "portal_cors_origins": null,
        "portal_developer_meta_fields": "[{\"label\":\"Full Name\",\"title\":\"full_name\",\"validator\":{\"required\":true,\"type\":\"string\"}}]",
        "portal_emails_from": null,
        "portal_emails_reply_to": null,
        "portal_invite_email": null,
        "portal_reset_email": null,
        "portal_reset_success_email": null,
        "portal_token_exp": null
      },
      "created_at": 1557441226,
      "id": "c663cca5-c6f6-474a-ae44-01f62aba16a9",
      "meta": {
        "color": null,
        "thumbnail": null
      },
      "name": "green-team"
    }
  ],
  "next": null
}
```

ワークスペースの更新または作成
---------------

**エンドポイント** 
<div class="endpoint put">/workspaces/\{id\}</div> 

|                       属性 |               説明               |
|-------------------------:|--------------------------------|
| `id`<br> **conditional** | **ワークスペース** 固有のID（置き換える場合）\*。 |

* `PUT` エンドポイントの動作は次のとおりです。リクエストペイロードにエンティティのプライマリキー（ワークスペースの `id`）が含まれて **いない** 場合、エンティティは指定されたペイロードで作成されます。リクエストペイロードにエンティティのプライマリキーが含まれて **いる** 場合、ペイロードは、提供されたプライマリキーで指定されたエンティティを「置換」します。プライマリキーが既存のエンティティのもので **ない** 場合は、`404 NOT FOUND` が返されます。

**リクエストボディ** 

|     属性 |    説明     |
|-------:|-----------|
| `name` | ワークスペース名。 |

**応答** 

エンティティを作成する場合：

    HTTP 201 Created

エンティティを置き換える場合：

    HTTP 200 OK

```json
{
  "comment": null,
  "config": {
    "meta": null,
    "portal": false,
    "portal_access_request_email": null,
    "portal_approved_email": null,
    "portal_auth": null,
    "portal_auth_conf": null,
    "portal_auto_approve": null,
    "portal_cors_origins": null,
    "portal_developer_meta_fields": "[{\"label\":\"Full Name\",\"title\":\"full_name\",\"validator\":{\"required\":true,\"type\":\"string\"}}]",
    "portal_emails_from": null,
    "portal_emails_reply_to": null,
    "portal_invite_email": null,
    "portal_reset_email": null,
    "portal_reset_success_email": null,
    "portal_token_exp": null
  },
  "created_at": 1557504202,
  "id": "c663cca5-c6f6-474a-ae44-01f62aba16a9",
  "meta": {
    "color": null,
    "thumbnail": null
  },
  "name": "rocket-team"
}
```

ワークスペースの取得
----------

**エンドポイント** 
<div class="endpoint get">/workspaces/\{name or id\}</div> 

|                      属性 |              説明               |
|------------------------:|-------------------------------|
| `name or id`<br> **必須** | 取得するワークスペースの一意の識別子 **または** 名前 |

**応答** 

    HTTP 200 OK

```json
{
  "config": {
    "portal": false,
    "portal_developer_meta_fields": "[{\"label\":\"Full Name\",\"title\":\"full_name\",\"validator\":{\"required\":true,\"type\":\"string\"}}]"
  },
  "created_at": 1557504202,
    "id": "c663cca5-c6f6-474a-ae44-01f62aba16a9",
    "meta": { },
  "name": "rocket-team"
}
```

ワークスペースのメタデータを取得する
------------------

**エンドポイント** 
<div class="endpoint get">/workspaces/\{name or id\}/meta</div> 

|                      属性 |              説明               |
|------------------------:|-------------------------------|
| `name or id`<br> **必須** | 取得するワークスペースの一意の識別子 **または** 名前 |

**応答** 

    HTTP 200 OK

```json
{
  "counts": {
    "acls": 1,
    "basicauth_credentials": 1,
    "consumers": 1234,
    "files": 41,
    "hmacauth_credentials": 1,
    "jwt_secrets": 1,
    "keyauth_credentials": 1,
    "oauth2_authorization_codes": 1,
    "oauth2_credentials": 1,
    "oauth2_tokens": 1,
    "plugins": 5,
    "rbac_roles": 3,
    "rbac_users": 12,
    "routes": 15,
    "services": 2,
    "ssl_certificates": 1,
    "ssl_servers_names": 1,
    "targets": 1,
    "upstreams": 1
  }
}
```

ワークスペースの削除
----------

**エンドポイント** 
<div class="endpoint delete">/workspaces/\{name or id\}</div> 

{% if_version lte:3.3.x %}

|                      属性 |               説明               |
|------------------------:|--------------------------------|
| `name or id`<br> **必須** | 削除するワークスペースの一意の識別子 **または** 名前。 |

{:.note}
> 
> **注：** ワークスペース自体を削除する前に、ワークスペース内のすべてのエンティティを削除する必要があります。

{% endif_version %}

{% if_version gte:3.4.x %}
属性 \| 説明
\-\-\-:\| \-\-\-
`name or id`<br> **必須** \| 削除するワークスペースの一意の識別子 **または** 名前
`cascade` \| `cascade`オプションを使用すると、ワークスペースとそのすべてのエンティティを1つのリクエストで削除できます。

**応答** 

    HTTP 204 No Content

連鎖的削除を実行します。通常、ワークスペースを削除するには、まずそのエンティティを削除する必要があります。 `cascade`オプションを使用すると、1つのリクエストでワークスペースとそのすべてのエンティティを削除できます。

    DELETE /workspaces/{name or id}?cascade=true

{% endif_version %}

ワークスペースの更新
----------

**エンドポイント** 
<div class="endpoint patch">/workspaces/\{name or id\}</div> 

|                      属性 |                 説明                 |
|------------------------:|------------------------------------|
| `name or id`<br> **必須** | パッチを適用するワークスペースの一意な識別子 **または** 名前。 |

**リクエストボディ** 

|        属性 |       説明        |
|----------:|-----------------|
| `comment` | ワークスペースを説明する文字列 |

`PATCH` エンドポイントの動作により、ワークスペースの名前変更ができません。

**応答** 

    HTTP 200 OK

```json
{
  "comment": "this is a sample comment in the patch request",
  "config": {
    "meta": null,
    "portal": false,
    "portal_access_request_email": null,
    "portal_approved_email": null,
    "portal_auth": null,
    "portal_auth_conf": null,
    "portal_auto_approve": null,
    "portal_cors_origins": null,
    "portal_developer_meta_fields": "[{\"label\":\"Full Name\",\"title\":\"full_name\",\"validator\":{\"required\":true,\"type\":\"string\"}}]",
    "portal_emails_from": null,
    "portal_emails_reply_to": null,
    "portal_invite_email": null,
    "portal_reset_email": null,
    "portal_reset_success_email": null,
    "portal_token_exp": null
  },
  "created_at": 1557509909,
  "id": "c543d2c8-d297-4c9c-adf5-cd64212868fd",
  "meta": {
    "color": null,
    "thumbnail": null
  },
  "name": "green-team"
}
```

ワークスペース内のエンドポイントにアクセスする
-----------------------

**エンドポイント** 
<div class="endpoint">/\{name or id\}/\{endpoint\}</div> 

|                      属性 |               説明                |
|------------------------:|---------------------------------|
| `name or id`<br> **必須** | 対象とするワークスペースの一意の識別子 **または** 名前。 |

任意のリクエストメソッドで使用できます。

エンティティエンドポイントの前にワークスペース名またはID接頭辞を追加して、指定されたワークスペース内のエンティティをターゲットにします。

```sh
http://localhost:8001/<WORKSPACE_NAME|WORKSPACE_ID>/<endpoint id="sl-md0000000">
```

たとえば、ワークスペース`SRE`の`services`をターゲットにするには、次のようにします。

```sh
http://localhost:8001/SRE/services
```

*** ** * ** ***

