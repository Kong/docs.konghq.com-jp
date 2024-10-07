---
title: "Key Authentication"
content-type: "tutorial"
book: "get-started"
chapter: 5
---
認証とは、申請者がリソースにアクセスする権限を所有していることを確認するプロセスです。
名前が物語るように、APIゲートウェイ認証では、アップストリームサービスとの間のデータフローが認証されます。


{{site.base_gateway}}には、最も広範に使用されている[APIゲートウェイ認証方法](/hub/#authentication)をサポートするプラグインライブラリがあります。

一般的な認証方法は次のとおりです。

* Key Authentication
* Basic Authentication
* OAuth 2\.0 Authentication
* LDAP Authentication Advanced
* OpenID Connect

認証のメリット
-------

{{site.base_gateway}}が認証を制御している場合、クライアントが正常に認証しないかぎり、リクエストはアップストリームのサービスに届きません。つまり、アップストリームサービスは事前に承認されたリクエストを処理することで、認証のコストから解放され、計算時間 *および* 開発工数が削減されます。


{{site.base_gateway}}はすべての認証試行を可視化し、監視とアラート機能のサポートサービスの可用性とコンプライアンスを構築する能力を提供します。

詳細については、[APIゲートウェイ認証とは](https://konghq.com/learning-center/api-gateway/api-gateway-authentication)を参照してください。

認証を有効化
------

次のチュートリアルでは、{{site.base_gateway}}のさまざまな側面において[Key Authenticationプラグイン](/hub/kong-inc/key-auth/)を有効化する方法について説明します。

API Key Authenticationは、API認証を実施するための一般的な方法です。Key Authenticationでは、
{{site.base_gateway}}を使用してAPI キーを生成し、[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)に関連付けます。このキーは、後続のリクエストを行うときにクライアントが提示する認証シークレットです。{{site.base_gateway}}は、提示されたキーの有効性に基づいてリクエストを承認または拒否します。このプロセスは、グローバルに適用することも、個々の[サービス](/gateway/latest/key-concepts/services/)や[ルート](/gateway/latest/key-concepts/routes/)ルートに適用することもできます。

### 前提条件

この章は、「 *Get Started with Kong* 」シリーズの一部です。最適な効果を得るために、シリーズを最初からご覧になることを
お勧めします。

まずは、導入として[Get Kong](/gateway/latest/get-started/)から始めましょう。ここには、ローカル{{site.base_gateway}}を実行するためのツールの前提条件と手順が含まれています。

ガイドの手順2の[サービスとルート](/gateway/latest/get-started/services-and-routes/)には、このシリーズで使われているモックサービスのインストール方法が記載されています。

これらの手順をまだ完了していない場合は、続行する前に完了してください。

### コンシューマとキーを設定

{{site.base_gateway}}のKey Authenticationは、コンシューマオブジェクトを使用して機能します。キーはコンシューマに割り当てられ、クライアントアプリケーションは、リクエストの中でキーを提示します。

1. **新しいコンシューマを作成します** 

   このチュートリアルでは、ユーザー名`luka`を持つ新しいコンシューマを作成します。

   ```sh
   curl -i -X POST http://localhost:8001/consumers/ \
     --data username=luka
   ```

   コンシューマが作成されたことを示す `201` レスポンスが返されます。
2. **コンシューマにキーを割り当てる** 

   プロビジョニングが完了したら、Admin API を呼び出して、新しいコンシューマーにキーを割り当てます。
   このチュートリアルでは、キーの値を `top-secret-key` に設定します。

   ```sh
   curl -i -X POST http://localhost:8001/consumers/luka/key-auth \
     --data key=top-secret-key
   ```

   キーが作成されたことを示す `201` レスポンスが返されます。

   この例では、キーの内容を明示的に`top-secret-key`に設定しています。
   `key`フィールドを指定しない場合は、 {{site.base_gateway}}によってキー値が生成されます。

   {:.important}
   > 
   > **警告** ：このチュートリアルのために、サンプルのキー値を割り当てました。APIゲートウェイに複雑なキーを
   > 自動生成させることをお勧めします。テスト用または既存のシステムを移行する場合にのみキーを指定します。

### グローバルキー認証

プラグインをグローバルにインストールすると、{{site.base_gateway}}に対する *すべての* プロキシリクエストがKey Authenticationで保護されます。

1. **Key Authenticationを有効化** 

   Key Authenticationプラグインはデフォルトで{{site.base_gateway}}にインストールされており、
   Admin APIのプラグインオブジェクトに`POST`リクエストを送信して有効にすることができます。

   ```sh
   curl -X POST http://localhost:8001/plugins/ \
       --data "name=key-auth"  \
       --data "config.key_names=apikey"
   ```

   プラグインがインストールされたことを示す`201`応答を受け取ります。

   上記のリクエストの`key_names`設定フィールドは、リクエストを認証するときにプラグインがキーを読み取るために検索するフィールドの名前を定義します。プラグインはヘッダー、クエリ文字列パラメータ、リクエスト本文でフィールドを検索します。
2. **認証されていないリクエストの送信** 

   キーを渡さずにサービスにアクセスしてみてください。

   ```sh
   curl -i http://localhost:8000/mock/anything
   ```

   Key Authenticationをグローバルに有効にしたため、不正な応答が届きます。

   ```text
   HTTP/1.1 401 Unauthorized
   ...
   {
       "message": "No API key found in request"
   }
   ```

3. **間違ったキーの送信** 

   不正なキーでサービスにアクセスしてみてください。

   ```sh
   curl -i http://localhost:8000/mock/anything \
     -H 'apikey:bad-key'
   ```

   不正な応答を受信します。

   ```text
   HTTP/1.1 401 Unauthorized
   ...
   {
     "message":"Invalid authentication credentials"
   }
   ```

4. **有効なリクエストを送信します** 

   `apikey` ヘッダーに有効なキーを設定してリクエストを送信します。

   ```sh
   curl -i http://localhost:8000/mock/anything \
     -H 'apikey:top-secret-key'
   ```

   `200 OK`応答が届きます。

### サービスベースのKey Authentication

Key Authenticationプラグインは特定のサービスに対して有効にできます。リクエストは上記と同じですが、`POST` リクエストはサービス URL に送信されます。

```sh
curl -X POST http://localhost:8001/services/example_service/plugins \
  --data name=key-auth
```

### ルートベースのKey Authentication

Key Authenticationプラグインは特定のルートに対して有効にできます。リクエストは上記と同じですが、 `POST`リクエストはルートURLに送信されます。

```sh
curl -X POST http://localhost:8001/routes/example_route/plugins \
  --data name=key-auth
```

（オプション）プラグインを無効化
----------------

この入門ガイドをセクションごとに進行する場合、今後のすべてのリクエストでこのAPIキーを使用する必要があります。キーを指定し続けたくない場合は、先に進む前にプラグインを無効にします。

1. **Key Authentication プラグイン ID を見つけます** 

   ```sh
   curl -X GET http://localhost:8001/plugins/
   ```

   次のスニペットのような、`id`フィールドを含むJSON応答が返されます。

   ```text
   ...
   "id": "2512e48d9-7by0-674c-84b7-00606792f96b"
   ...
   ```

2. **プラグインの無効化** 

   上記で取得したプラグインIDを使用して、インストールされたKey Authenticationプラグインの`enabled`フィールドに`PATCH`を適用します。リクエストは、適切なプラグインIDを置き換えると次のようになります。

   ```sh
   curl -X PATCH http://localhost:8001/plugins/2512e48d9-7by0-674c-84b7-00606792f96b \
     --data enabled=false
   ```

3. **無効な認証のテスト** 

   これで、API キーを指定せずにリクエストを行うことができます。

   ```sh
   curl -i http://localhost:8000/mock/anything
   ```

   次のように表示されます。

   ```text
   HTTP/1.1 200 OK
   ```

