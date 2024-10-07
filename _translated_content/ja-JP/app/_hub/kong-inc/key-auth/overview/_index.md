---
nav_title: "概要"
---
このプラグインを使用すると、サービスまたはルートにKey Authenticationを追加できます。コンシューマは、APIキーをクエリ文字列パラメータ、ヘッダー、またはリクエストボディに追加してリクエストを認証できます。

このプラグインは、[アプリケーション登録](/hub/kong-inc/application-registration/)プラグインと組み合わせて認証に使用できます。

このプラグインはエンタープライズバージョンである[Key Authentication Encrypted](/hub/kong-inc/key-auth-enc/)でも利用でき、キーを暗号化することができます。キーはAPIゲートウェイデータストアに保存時に暗号化されます。

大文字と小文字の区別
----------

それぞれの使用によると、HTTPヘッダー名は大文字と小文字は *区別されません* が、HTTPクエリ文字列パラメータ名は大文字と小文字が *区別されます* 。Kongは設計どおりにこれらの仕様に従います。つまり、リクエストヘッダーフィールドの検索とクエリ文字列の検索では`key_names`構成値は処理が異なります。ベストプラクティスとして、管理者は認証キーがリクエストヘッダーに送信されることを期待する場合、大文字と小文字を区別する`key_names`値を定義しないことを奨励します。

これが適用されると、有効な認証情報を持つユーザーはこのサービスにアクセスできます。特定の権限を持つユーザーの使用を制限するには、[ACL](/plugins/acl/)プラグイン（ここでは説明されていません）を追加して許可されるユーザーと拒否されるユーザーのグループを作成できます。

使用法
---

### コンシューマを作成

既存の[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)オブジェクトへの認証情報を関連付ける必要があります。コンシューマは多数の認証情報を持つことができます。

{% navtabs %}
{% navtab With a database %}

コンシューマを作成するには、以下のリクエストを実行します。

```bash
curl -d "username=user123&custom_id=SOME_CUSTOM_ID" http://localhost:8001/consumers/
```

{% endnavtab %}

{% navtab Without a database %}

宣言型構成ファイルには、1つ以上のコンシューマが必要です。これらは`consumers:` yamlセクションで作成できます。

```yaml
consumers:
- username: user123
  custom_id: SOME_CUSTOM_ID
```

{% endnavtab %}
{% endnavtabs %}

どちらの場合も、パラメーターは以下のとおりです。

|           パラメータ           |                                    説明                                     |
|---------------------------|---------------------------------------------------------------------------|
| `username`<br> *セミオプション*  | コンシューマのユーザー名。このフィールドか`custom_id`のいずれかを指定する必要があります。                        |
| `custom_id`<br> *セミオプション* | コンシューマを別のデータベースにマップするために使用されるカスタム識別子。このフィールドか`username`のいずれかを指定する必要があります。 |

このサービスで[ACL](/plugins/acl/)プラグインと許可リストも使用している場合、新しいコンシューマを許可されたグループに追加する必要があります。詳細については、[ACL：コンシューマの関連付け](/plugins/acl/#associating-consumers)を参照してください。

匿名アクセスを構成する方法の詳細については、 [匿名アクセス](/gateway/latest/kong-plugins/authentication/reference/#anonymous-access)を参照してください。

### 複数認証


{{site.base_gateway}}は特定のサービスに対して複数の認証プラグインをサポートしており、異なるクライアントが異なる認証方法を使用して特定のサービスまたはルートにアクセスできるようにします。詳細については、 [複数認証](/gateway/latest/kong-plugins/authentication/reference/#multiple-authentication)を参照してください。

### キーの作成

{:.important}
> 
> **注** ：APIゲートウェイにキーを自動生成させることをお勧めします。既存のシステムを{{site.base_gateway}}に移行する場合にのみ、自分で指定してください。
> {{site.base_gateway}}への移行をコンシューマにとって透過的にするために、キーを再利用する必要があります。

{% navtabs %}
{% navtab With a database %}

次のHTTPリクエストを実行して、新しい認証情報をプロビジョニングします。

```bash
curl -X POST http://localhost:8001/consumers/{USERNAME_OR_ID}/key-auth
```

応答：

    HTTP/1.1 201 Created
    {
      "key": "5SRmk6gLnTy1SyQ1Cl9GzoRXJbjYGGbZ",
      "created_at": 1655412883,
      "tags": null,
      "ttl": null,
      "id": "0103844a-0b40-42c8-aa71-64e98e2e525f",
      "consumer": {
        "id": "d9e37c7d-3261-4b7b-81a6-a94bc203a0ca"
      }
    }

{% endnavtab %}
{% navtab Without a database %}

`keyauth_credentials` YAMLエントリを使用して、宣言型構成ファイルに認証情報を追加します。

```yaml
keyauth_credentials:
- consumer: {USERNAME_OR_ID}
  ttl: 5000
  tags:
    - example_tag
  key: example_apikey
```

{% endnavtab %}
{% endnavtabs %}

どちらの場合も、フィールド/パラメータは次のように機能します。

|    フィールド/パラメータ     |                                                           説明                                                           |
|--------------------|------------------------------------------------------------------------------------------------------------------------|
| `{USERNAME_OR_ID}` | 認証情報を関連付けるための[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)エンティティの`id`または`username`プロパティ。           |
| `ttl`<br> *オプション*  | キーが有効になる秒数。欠落しているかゼロに設定されている場合、キーの`ttl`は無制限になります。 存在する場合、値は0 ～ 100000000の整数である必要があります。現在、DBレスモードやハイブリッドモードとは互換性がありません。 |
| `tags`<br> *オプション* | オプションで、タグのリストを`key`に割り当てることができます。                                                                                      |
| `key`<br> *オプション*  | オプションで、クライアントを認証するために独自の一意の`key`を設定できます。見つからない場合は、プラグインが生成します。                                                         |

### キーでリクエストする

クエリ文字列パラメータとしてキーを指定してリクエストします：

```bash
curl http://localhost:8000/{proxy path}?apikey={some_key}
```

**注：** `key_in_query` プラグインパラメータは `true`（デフォルト）に設定する必要があります。

ヘッダーにキーを使用してリクエストを行います。

```bash
curl http://localhost:8000/{proxy path} \
    -H 'apikey: {some_key}'
```

**注：** `key_in_header` プラグインパラメータは `true`（デフォルト）に設定する必要があります。

本文にキーを使用してリクエストを行います。

```bash
curl http://localhost:8000/{proxy path} \
    --data 'apikey={some_key}'
```

**注：** `key_in_body`プラグインパラメータは`true`に設定する必要があります（デフォルトは`false`です）。

gRPCクライアントもサポートされています。

```bash
grpcurl -H 'apikey: {some_key}' ...
```

### リクエスト内のAPIキーの場所について

{% include /md/plugins-hub/api-key-locations.md %}

### キーの場所を無効にする

この例では、クエリパラメータでのキーの使用を無効にします。

```bash
curl -X POST http://localhost:8001/routes/{NAME_OR_ID}/plugins \
    --data "name=key-auth"  \
    --data "config.key_names=apikey" \
    --data "config.key_in_query=false"
```

### キーの削除

APIキーを削除するには、次のリクエストを行います。

```bash
curl -X DELETE http://localhost:8001/consumers/{USERNAME_OR_ID}/key-auth/{ID}
```

応答：

```bash
HTTP/1.1 204 No Content
```

* `USERNAME_OR_ID`：認証情報を関連付けるための[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)エンティティの`id`または`username`プロパティ。
* `ID`：キー認証情報オブジェクトの `id`属性。

### アップストリームヘッダー

{% include_cached /md/plugins-hub/upstream-headers.md %}

### キーにページ番号を付ける

次のリクエストを実行して、すべてのコンシューマのAPIキーにページ番号を付けます。

```bash
curl -X GET http://localhost:8001/key-auths
```

応答：

```json
...
{
   "data":[
      {
         "id":"17ab4e95-9598-424f-a99a-ffa9f413a821",
         "created_at":1507941267000,
         "key":"Qslaip2ruiwcusuSUdhXPv4SORZrfj4L",
         "consumer": { "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" }
      },
      {
         "id":"6cb76501-c970-4e12-97c6-3afbbba3b454",
         "created_at":1507936652000,
         "key":"nCztu5Jrz18YAWmkwOGJkQe9T8lB99l4",
         "consumer": { "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" }
      },
      {
         "id":"b1d87b08-7eb6-4320-8069-efd85a4a8d89",
         "created_at":1507941307000,
         "key":"26WUW1VEsmwT1ORBFsJmLHZLDNAxh09l",
         "consumer": { "id": "3c2c8fc1-7245-4fbb-b48b-e5947e1ce941" }
      }
   ]
   "next":null,
}
```

別のエンドポイントを使用して、コンシューマ別にリストをフィルタリングします。

```bash
curl -X GET http://localhost:8001/consumers/{USERNAME_OR_ID}/key-auth
```

上記は、コンシューマに関連付けられた`username`または`id`プロパティを置き換えます。

応答：

```json
...
{
    "data": [
       {
         "id":"6cb76501-c970-4e12-97c6-3afbbba3b454",
         "created_at":1507936652000,
         "key":"nCztu5Jrz18YAWmkwOGJkQe9T8lB99l4",
         "consumer": { "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" }
       }
    ]
    "next":null,
}
```

### キーに関連付けられたコンシューマを取得する

次のリクエストを実行して、APIキーに関連付けられた[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を取得します。

```bash
curl -X GET http://localhost:8001/key-auths/{KEY_OR_ID}/consumer
```

上記は、Key Authenticationエンティティに関連付けられた`key`または`id`プロパティのいずれかを置き換えます。

応答：

```json
{
   "created_at":1507936639000,
   "username":"foo",
   "id":"c0d92ba9-8306-482a-b60d-0cfdd2f0e880"
}
```

### リクエスト動作マトリックス

次の表は、さまざまなシナリオにおける{{site.base_gateway}}の動作を示しています。

|           説明            | アップストリームサービスにプロキシされている | 応答ステータスコード |
|-------------------------|------------------------|------------|
| リクエストに有効なAPIキーがあります。    | はい                     | 200        |
| APIキーが提供されていません。        | いいえ                    | 401        |
| APIキーが{{site.base_gateway}}に認識されていません。 | いいえ                    | 401        |
| ランタイムエラーが発生しました。        | いいえ                    | 500        |

