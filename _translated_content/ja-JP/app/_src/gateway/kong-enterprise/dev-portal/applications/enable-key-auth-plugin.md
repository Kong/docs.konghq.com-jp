---
title: "アプリケーション登録のKey Authenticationを有効にする"
badge: "enterprise"
---
Key Authenticationプラグインは、アプリケーション登録プラグインと組み合わせて認証のために使用できます。

キー認証プラグインは、Kong OAuth2 プラグイン用に生成されたものと同じクライアント ID を使用します。
OAuth2 プラグインが有効になっているサービスには、同じクライアント ID 認証情報を使用できます。

前提条件
----

* サービスを作成します。
* サービス上で[アプリケーション登録プラグイン](/gateway/{{page.release}}/kong-enterprise/dev-portal/applications/enable-application-registration/)を有効にします。
* まだ行っていない場合は、サービス用アプリケーションを有効にします。自動承認が有効でない場合、サービス契約はアドミンが承認する必要があります。 
* [最初に作成したデフォルトの認証情報を使用したくない場合は、認証情報を生成してください](#gen-client-id-cred)。

Kong Manager で Key Authentication を有効にする
----------------------------------------

Kong Managerで、アプリケーション登録で使用するKey Authenticationを有効にするサービスにアクセスします。

1. ワークスペースの左側のナビゲーションペインで、 **API Gateway> Services** に移動します。

2. 「サービス」ページで、サービスを選択し、 **表示** をクリックします。

3. サービス（Services） ページのプラグイン（Plugins）ペインで、 **プラグインを追加（Add a Plugin）** をクリックします。

4. 新しいプラグインの追加（Add New Plugin）ページの認証（Authentication ）セクションで、
   **Key Authentication** プラグインを見つけ、 **有効化（Enable）** をクリックします。

   ![Key Authenticationプラグインパネル](/assets/images/products/gateway/dev-portal/key-auth-plugin-panel.png)
5. アプリケーションに応じてフィールドに入力します。この例では、サービスは事前入力されています。次のセクション、[Key Authentication構成パラメータ](#key-auth-params)で説明するパラメータを参照して、フィールドに入力します。

6. **作成** をクリックします。

### Key Authentication構成パラメータ \{\#key\-auth\-params\}

| フォームパラメータ          | 説明                                                                                                                                                                                                                        |
|:-------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Service`          | このプラグイン構成の対象となるサービス。必須。                                                                                                                                                                                                   |
| `Anonymous`        | 認証に失敗した場合に匿名のコンシューマとして使用するオプションの文字列（コンシューマー UID）値。空（デフォルト）の場合、リクエストは `4xx` で失敗します。この値は Kong の内部にあるコンシューマの `id` 属性を参照する必要があり、`custom_id` を参照するのでは **ない** ことに注意してください。                                                      |
| `Hide Credentials` | Upstreamサービスからの認証情報を表示するか非表示にするかを指定します。`true` の場合、プラグインはリクエストから認証情報（つまり、ヘッダー、クエリ文字列、またはキーを含むリクエストボディ）を削除してからプロキシします。デフォルト：`false` 。                                                                                       |
| `Key in Body`      | 有効にすると、プラグインはリクエストボディを読み取り（リクエストにボディが1つあり、そのMIMEタイプがサポートされている場合）、その中でキーを見つけようとします。サポートされているMIMEタイプ：`application/www-form-urlencoded`、`application/json`、`multipart/form-data`。デフォルト：`false`。                              |
| `Key in Header`    | 有効にすると（デフォルト）、プラグインはリクエストヘッダーを読み取り、その中のキーを見つけようとします。デフォルト: true。                                                                                                                                                          |
| `Key in Query`     | 有効にすると（デフォルト）、プラグインはリクエストのクエリパラメータを読み取り、その中のキーを見つけようとします。デフォルト：true。                                                                                                                                                      |
| `Key Names`        | プラグインがキーを探すパラメータ名の配列を記述します。クライアントはそれらのキー名の1つで認証キーを送信する必要があり、プラグインは同じ名前のヘッダー、リクエストボディ、またはクエリ文字列パラメーターから認証情報を読み取ろうとします。キー名には、\[a\-z\]、\[A\-Z\]、\[0\-9\]、\[\_\] アンダースコア、および \[\-\] ハイフンのみを含めることができます。必須。デフォルト：`apikey`。 |
| `Run on Preflight` | プラグインが`OPTIONS`プリフライトリクエストで実行される \(および認証を試行する\) かどうかを示します。デフォルト: `true` 。                                                                                                                                               |

認証情報を生成する \{\#gen\-client\-id\-cred\}
-------------------------------------------

APIキーとして使用するクライアントID認証情報を生成します。複数の認証情報を生成できます。

1. **Dev Portal > My Apps** ページで、アプリケーションの **View** をクリックします。

2. **認証** ペインで、 **認証情報の生成** をクリックします。

   ![アプリケーション認証ペイン](/assets/images/products/gateway/dev-portal/generate-cred-dev-portal.png)

   これで、クライアント IDをAPIキーとして使用してリクエストを行うことができます。

APIキー（クライアント識別子）を使用してリクエストを行う
-----------------------------

認証情報のクライアントIDは、サービスへの認証済みのリクエストを行うためのAPIキーとして使用できます。

**ヒント：** キーリクエストの指示には、アプリケーションのサービス詳細エリアにある情報アイコンから、ユーザーインターフェイス内で直接アクセスすることもできます。 **i** アイコンをクリックして、サービスの詳細ページを開きます。

![サービスペイン](/assets/images/products/gateway/dev-portal/portal-info-modal-key-auth.png)

スクロールして、使用可能なすべての例を表示します。

![サービスの詳細ページ埋め込みキーの使用手順](/assets/images/products/gateway/dev-portal/service-details-key-auth-usage.png)

### リクエスト内のAPIキーの場所について

{% include /md/plugins-hub/api-key-locations.md %}

### キーをクエリー文字列パラメータとしてリクエストを作成する

```bash
curl -X POST {proxy}/{route}?apikey={CLIENT_ID}
```

応答（キーの場所に関係なく、すべての有効なリクエストに対して同じになります）：

```bash
HTTP/1.1 200 OK
...
```

### ヘッダーでキーを使用してリクエストを行う

```bash
curl -X POST {proxy}/{route} \
--header "apikey: {CLIENT_ID}"
```

### ボディでキーを使用してリクエストを行う

```bash
curl -X POST {proxy}/{route} \
--data "apikey:={CLIENT_ID}"
```

{:.note}
> 
> **注：** `key_in_body`パラメータは`true`に設定する必要があります。
