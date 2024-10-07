---
title: "複数の認証方法の許可"
---
Kong 認証プラグインのデフォルトの動作では、リクエストが他のプラグインを経由して認証されたかどうかに関係なく、すべてのリクエストに対して認証情報が要求されます。認証プラグインで匿名コンシューマを構成すると、クライアントに複数の認証オプションを提供できます。

コンシューマの設定
---------

まず、[サービスを作成して](/gateway/{{page.release}}/admin-api/#service-object)、
次に、3つのコンシューマーを作成します。

1. コンシューマ 1：

   ```bash
   curl -sX POST localhost:8001/consumers \
     -H "Content-Type: application/json" \
     --data '{"username": "anonymous"}'

   ```

   応答: 

   ```json
   {"created_at":1517528237000,"username":"anonymous","id":"d955c0cb-1a6e-4152-9440-414ebb8fee8a"}
   ```

   `anonymous`コンシューマは実際のユーザーに対応しておらず、フォールバックとしてのみ機能します。
2. コンシューマ 2：

   ```sh
   curl -sX POST localhost:8001/consumers \
     -H "Content-Type: application/json" \
     --data '{"username": "medvezhonok"}'
   ```

   応答: 

   ```json
   {"created_at":1517528259000,"username":"medvezhonok","id":"b3c95318-a932-4bb2-9d74-1298a3ffc87c"}
   ```

3. コンシューマ 3：

   ```sh
   curl -sX POST localhost:8001/consumers \
     -H "Content-Type: application/json" \
     --data '{"username": "ezhik"}'
   ```

   応答: 

   ```json
   {"created_at":1517528266000,"username":"ezhik","id":"47e74a17-dc08-4786-a8cf-d8e4f38a5459"}
   ```

認証プラグインを設定する
------------

次に、Key AuthとBasic Authの両方のプラグインをサービスに追加し、先ほど背悪性したコンシューマに匿名のフォールバックを設定します。

1. Key Authプラグインを追加します。

   ```bash
   curl -sX POST localhost:8001/services/example-service/plugins/ \
     -H "Content-Type: application/json" \
     --data '{"name": "key-auth", "config": { "hide_credentials": true, "anonymous": "d955c0cb-1a6e-4152-9440-414ebb8fee8a"} }'
   ```

   応答: 

   ```json
   {"created_at":1517528304000,"config":{"key_in_body":false,"hide_credentials":true,"anonymous":"d955c0cb-1a6e-4152-9440-414ebb8fee8a","run_on_preflight":true,"key_names":["apikey"]},"id":"bb884f7b-4e48-4166-8c80-c858b5a4c357","name":"key-auth","service_id":"a2a168a8-4491-4fe1-9426-cde3b5fcd45b","enabled":true}
   ```

2. Basic Authプラグインを追加します。

   ```sh
   curl -sX POST localhost:8001/services/example-service/plugins/ \
     -H "Content-Type: application/json" \
     --data '{"name": "basic-auth", "config": { "hide_credentials": true, "anonymous": "d955c0cb-1a6e-4152-9440-414ebb8fee8a"} }'
   ```

   応答: 

   ```json
   {"created_at":1517528499000,"config":{"hide_credentials":true,"anonymous":"d955c0cb-1a6e-4152-9440-414ebb8fee8a"},"id":"e5a40543-debe-4225-a879-a54901368e6d","name":"basic-auth","service_id":"a2a168a8-4491-4fe1-9426-cde3b5fcd45b","enabled":true}
   ```

[OpenID Connect](/hub/kong-inc/openid-connect/)を使用する場合は、`anonymous`のみを設定するとそのコンシューマーがマッピングされないため、 `anonymous`とともに`config.consumer_claim`も設定する必要があります。

匿名コンシューマによるテスト
--------------

この時点では、認証されていないリクエストや無効な認証情報を持つリクエストは引き続き許可されます。匿名のコンシューマは許可されており、他の一部のコンシューマに関連付けられた認証情報を渡さないすべてのリクエストに対して適用されます。

1. 以下のように認証情報なしでチェックします。

   ```bash
   curl -s example.com:8000/user-agent
   ```

   応答: 

   ```json
   {"user-agent": "curl/7.58.0"}
   ```

2. nonsenseな認証情報で確認してください。

   ```sh
   curl -s example.com:8000/user-agent?apikey=nonsense
   ```

   応答: 

   ```json
   {"user-agent": "curl/7.58.0"}
   ```

認証情報を構成する
---------

ここで、1つのコンシューマにKey Auth認証情報を追加し、別のコンシューマにBasic Auth認証情報を追加します。

1. Basic Auth認証情報を 1 つのコンシューマーに追加します。

   ```bash
   curl -sX POST localhost:8001/consumers/medvezhonok/basic-auth \
     -H "Content-Type: application/json" \
     --data '{"username": "medvezhonok", "password": "hunter2"}'
   ```

   応答: 

   ```json
   {"created_at":1517528647000,"id":"bb350b87-f0d2-4605-b997-e28a116d8b6d","username":"medvezhonok","password":"f239a0404351d7170201e7f92fa9b3159e47bb01","consumer_id":"b3c95318-a932-4bb2-9d74-1298a3ffc87c"}
   ```

2. Key Auth認証情報を別のコンシューマに追加します。

   ```sh
   curl -sX POST localhost:8001/consumers/ezhik/key-auth \
     -H "Content-Type: application/json" \
     --data '{"key": "hunter3"}'
   ```

   応答: 

   ```json
   {"id":"06412d6e-8d41-47f7-a911-3c821ec98f1b","created_at":1517528730000,"key":"hunter3","consumer_id":"47e74a17-dc08-4786-a8cf-d8e4f38a5459"}
   ```

3. 最後に、匿名のコンシューマに Request Terminator を追加します。

   ```bash
   curl -sX POST localhost:8001/consumers/d955c0cb-1a6e-4152-9440-414ebb8fee8a/plugins/ \
     -H "Content-Type: application/json" \
     --data '{"name": "request-termination", "config": { "status_code": 401, "content_type": "application/json; charset=utf-8", "body": "{\"error\": \"Authentication required\"}"} }'
   ```

   応答: 

   ```json
   {"created_at":1517528791000,"config":{"status_code":401,"content_type":"application\/json; charset=utf-8","body":"{\"error\": \"Authentication required\"}"},"id":"21fc5f6f-363f-4d79-b533-ce26d4478879","name":"request-termination","enabled":true,"consumer_id":"d955c0cb-1a6e-4152-9440-414ebb8fee8a"}
   ```

認証情報の検証
-------

もう一度検証してください。認証情報が欠落しているか無効であるリクエストは拒否されるようになりましたが、どちらの認証メソッドを使用して承認されたリクエストも許可されます。

1. nonsense API キーを使用してテストします。

   ```bash
   curl -s localhost:8000/user-agent?apikey=nonsense
   ```

   応答: 

   ```json
   {"error": "Authentication required"}
   ```

2. APIキーなしでテストします。

   ```sh
   curl -s localhost:8000/user-agent
   ```

   応答: 

   ```json
   {"error": "Authentication required"}
   ```

3. 構成されたAPIキーを使用してテストします。

   ```sh
   curl -s localhost:8000/user-agent?apikey=hunter3
   ```

   応答: 

   ```json
   {"user-agent": "curl/7.58.0"}
   ```

4. 構成されたBasic Auth認証情報を使用してテストします。

   ```sh
   curl -s localhost:8000/user-agent -u medvezhonok:hunter2
   ```

   応答: 

   ```json
   {"user-agent": "curl/7.58.0"}
   ```

