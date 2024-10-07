---
nav_title: "概要"
---
ユーザー名とパスワード保護があるサービスまたはルートにBasic Authenticationを追加します。`Proxy-Authorization`と`Authorization`ヘッダーの有効な認証情報のプラグインチェック（この順序で）。

使用法
---

プラグインを使用するには、まず、コンシューマを作成して1つ以上の認証情報を関連付ける必要があります。コンシューマは、アップストリームサービスを利用する開発者またはアプリケーションを表します。

### コンシューマを作成

既存の[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)オブジェクトへの認証情報を関連付ける必要があります。コンシューマは多数の認証情報を持つことができます。

{% navtabs %}
{% navtab With a Database %}
コンシューマを作成するには、次のリクエストを実行します。

```bash
curl -d "username={USER123}&custom_id={SOME_CUSTOM_ID}" http://localhost:8001/consumers/
```

{% endnavtab %}
{% navtab Without a Database %}
宣言型構成ファイルには、1つ以上のコンシューマが必要です。これらは、`consumers:` YAMLセクションを使用して作成できます。

```yaml
consumers:
- username: {USER123}
  custom_id: {SOME_CUSTOM_ID}
```

{% endnavtab %}
{% endnavtabs %}

コンシューマパラメータ：

|         パラメータ         |                                    説明                                     |
|-----------------------|---------------------------------------------------------------------------|
| `username` *セミオプション*  | コンシューマのユーザー名。このフィールドか`custom_id`のいずれかを指定する必要があります。                        |
| `custom_id` *セミオプション* | コンシューマを別のデータベースにマップするために使用されるカスタム識別子。このフィールドか`username`のいずれかを指定する必要があります。 |

このサービスで[ACL](/plugins/acl/)プラグインと許可リストも使用している場合、新しいコンシューマーを許可されたグループに追加する必要があります。詳細については、[ACL：コンシューマの関連付け](/plugins/acl/#associating-consumers)を参照してください。

### 認証情報を作成

{% navtabs %}
{% navtab With a Database %}
次のHTTPリクエストを行うことで、新しいユーザー名/パスワード認証情報をプロビジョニングできます。

```bash
curl -X POST http://localhost:8001/consumers/{CONSUMER}/basic-auth \
  --data "username=Aladdin" \
  --data "password=OpenSesame"
```

{% endnavtab %}
{% navtab Without a Database %}

`basicauth_credentials` YAMLエントリを使用して、宣言型構成ファイルに認証情報を追加できます。

```yaml
basicauth_credentials:
- consumer: {CONSUMER}
  username: Aladdin
  password: OpenSesame
```

{% endnavtab %}
{% endnavtabs %}

コンシューマ認証情報パラメータ:

| フィールド/パラメータ |                                                      説明                                                      |
|-------------|--------------------------------------------------------------------------------------------------------------|
| `consumer`  | 認証情報を関連付けるための[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)エンティティの`id`または`username`プロパティ。 |
| `username`  | ベーシック認証の認証情報で使用するユーザー名。                                                                                      |
| `password`  | ベーシック認証の認証情報で使用するパスワード。                                                                                      |

### 認証情報の使用

認証ヘッダーはbase64でエンコードされている必要があります。たとえば、認証情報でユーザー名として`Aladdin`、パスワードとして`OpenSesame`が使用される場合、フィールド値は`Aladdin:OpenSesame`のbase64エンコーディングまたは`QWxhZGRpbjpPcGVuU2VzYW1l`となります。

`Authorization` \(または`Proxy-Authorization` \) ヘッダーは次のように表示される必要があります。

    Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l

ヘッダーを使用してリクエストを行います。

```bash
curl http://kong:8000/{PATH_MATCHING_CONFIGURED_ROUTE} \
  -H 'Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l'
```

gRPCクライアントもサポートされています。

```bash
grpcurl -H 'Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l' ...
```

### アップストリームヘッダー

{% include_cached /md/plugins-hub/upstream-headers.md %}

### basic\-auth認証情報を介したページ番号付け

次のリクエストを使用して、すべてのコンシューマのbasic\-auth認証情報を開始ページ番号を付けることができます。

{% navtabs codeblock %}
{% navtab Request %}

```bash
curl -X GET http://localhost:8001/basic-auths
```

{% endnavtab %}
{% navtab Response %}

```json
{
    "total": 3,
    "data": [
        {
            "created_at": 1511379926000,
            "id": "805520f6-842b-419f-8a12-d1de8a30b29f",
            "password": "37b1af03d3860acf40bd9c681aa3ef3f543e49fe",
            "username": "baz",
            "consumer": { "id": "5e52251c-54b9-4c10-9605-b9b499aedb47" }
        },
        {
            "created_at": 1511379863000,
            "id": "8edfe5c7-3151-4d92-971f-3faa5e6c5d7e",
            "password": "451b06c564a06ce60874d0ea2f542fa8ed26317e",
            "username": "foo",
            "consumer": { "id": "89a41fef-3b40-4bb0-b5af-33da57a7ffcf" }
        },
        {
            "created_at": 1511379877000,
            "id": "f11cb0ea-eacf-4a6b-baea-a0e0b519a990",
            "password": "451b06c564a06ce60874d0ea2f542fa8ed26317e",
            "username": "foobar",
            "consumer": { "id": "89a41fef-3b40-4bb0-b5af-33da57a7ffcf" }
        }
    ]
}
```

{% endnavtab %}
{% endnavtabs %}

次のエンドポイントを使用して、コンシューマ別にリストをフィルタリングできます。

{% navtabs codeblock %}
{% navtab Request %}

```bash
curl -X GET http://localhost:8001/consumers/{USERNAME_OR_ID}/basic-auth
```

{% endnavtab %}
{% navtab Response %}

```json
{
    "total": 1,
    "data": [
        {
            "created_at": 1511379863000,
            "id": "8edfe5c7-3151-4d92-971f-3faa5e6c5d7e",
            "password": "451b06c564a06ce60874d0ea2f542fa8ed26317e",
            "username": "foo",
            "consumer": { "id": "89a41fef-3b40-4bb0-b5af-33da57a7ffcf" }
        }
    ]
}
```

{% endnavtab %}
{% endnavtabs %}

`username`または`id` : 認証情報が一覧表示される必要のあるコンシューマのユーザー名またはID。

### 認証情報に関連付けられているコンシューマを取得

次のリクエストを使用して、basic\-auth認証情報に関連する[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を取得するのは可能です。

{% navtabs codeblock %}
{% navtab Request %}

```bash
curl -X GET http://localhost:8001/basic-auths/{USERNAME_OR_ID}/consumer
```

{% endnavtab %}
{% navtab Response %}

```json
{
   "created_at":1507936639000,
   "username":"foo",
   "id":"c0d92ba9-8306-482a-b60d-0cfdd2f0e880"
}
```

{% endnavtab %}
{% endnavtabs %}

`username or id`：関連付けられた[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を取得するbasic\-auth認証情報の`id`または`username`プロパティ。ここで許可される`username`は、コンシューマの`username`プロパティでは **ない** ことにご留意ください。

