---
title: "RBACの例"
badge: "enterprise"
---
この章では、エンドツーエンドのユースケースを使用して、RBAC の設定方法と実際の動作を順を追って説明します。選択したユースケースは、 **RBAC とワークスペース** を組み合わせて、複雑な階層でチームとユーザーの柔軟な組織を実現する方法を示しています。

ユースケース
------

例として、ある会社に{{site.base_gateway}}クラスタがあり、チームAとチームBの2つのチームで共有されるとします。Kongクラスタはこれらのチーム間で共有されていますが、彼らは、1つのチームでのエンティティの管理が他のチームでの操作を混乱させないような方法で、エンティティをセグメント化できるようにしたいと考えています。[ワークスペースページ](/gateway/{{page.release}}/kong-enterprise/workspaces)に示すように、このようなユースケースはワークスペースで可能です。しかし各チームは、ワークスペースに加えて、ワークスペースへのアクセス制御を適用したいと考えており、これはRBACで実現できます。

要約すると、ワークスペースとRBACは補完関係にあります。ワークスペースはAdmin APIエンティティのセグメンテーションを提供し、RBACはアクセス制御を提供します。

{:.note}
> 
> **注：** このガイドの応答例は、多くの場合、完全な応答の抜粋であり、完全な応答は非常に長くなる可能性があるため、応答の最も関連性の高い部分に焦点を当てています。

最初の RBAC ユーザーのブートストラップ
----------------------

最初のRBACユーザーはスーパー管理者と呼ばれます。

Kong では、スーパー管理者ユーザーを作成したあとに、RBAC を実際に適用し、RBAC を有効にした状態で {{site.base_gateway}} を再起動することを推奨します。

{:.note}
> 
> インストール時に最初のスーパー管理者を作成することが可能です。このオプションを選択した場合は、[RBAC の適用](#enforcing-rbac)に進みます。


{{site.base_gateway}}には、デフォルトのRBACロールのセット、`super-admin`、`admin`、`read-only`が付属しています。これにより、スーパー管理者ユーザーを作成するタスクが簡単になります

1. `super-admin`というRBACユーザーを作成します。

   ```sh
   curl -i -X POST http://localhost:8001/rbac/users \
     -H 'Kong-Admin-Token:secureadmintoken' \
     --data name=super-admin \
     --data user_token=exampletoken
   ```

   レスポンス: 

   ```json
   {
     "user_token": "M8J5A88xKXa7FNKsMbgLMjkm6zI2anOY",
     "id": "da80838d-49f8-40f6-b673-6fff3e2c305b",
     "enabled": true,
     "created_at": 1531009435,
     "updated_at": 1531009435,
     "name": "super-admin"
   }
   ```

   `super-admin` ユーザー名は、既存の `super-admin` ロールと一致するため、`super-admin` ロールに自動的に追加されます。
2. 次のコマンドを使用して確認します。

   ```sh
   curl -i -X GET http://localhost:8001/rbac/users/super-admin/roles
   ```

   レスポンス: 

   ```json
   {
     "roles": [
       {
         "comment": "Full access to all endpoints, across all workspaces",
         "created_at": 1531009724,
         "updated_at": 1531009724,
         "name": "super-admin",
         "id": "b924ac91-e83f-4136-a5a4-4a7ff92594a8"
       }
     ],
     "user": {
       "created_at": 1531009435,
       "updated_at": 1531009724,
       "id": "e6897cc0-0c34-4a9c-9f0b-cc65b4f04d68",
       "name": "super-admin",
       "enabled": true,
       "user_token": "vajeOlkybsn0q0VD9qw9B3nHYOErgY7b8"
     }
   }
   ```

RBACの実施
-------

`super-admin`ユーザーが作成されたので、Kong管理者はRBAC が強制されている

{{site.base_gateway}}を再起動できるようになりました。

    KONG_ENFORCE_RBAC=on kong restart

これはRBACを強制してKongを再起動する方法の
1つです。別の方法としては、 {{site.base_gateway}}設定ファイルを編集し、
再起動します。

次に進む前に、このガイドではスーパー管理者ユーザーを使用していますが、RBACを有効にせずに前に進むことができ、Kong管理者が全体のRBAC階層を設定できることにご留意ください。

ただし、RBACはタスクを柔軟に分離するのに十分な能力があります。
要約すると次のようになります。

* **Kong管理者** ：このユーザーはKongのインフラストラクチャに物理的にアクセスできます。彼らのタスクは、KongクラスターとRBACの初期ユーザーを含むその構成をブートストラップすることです。 
* **RBACスーパー管理者** ：Kong管理者が作成し、RBACユーザー、ロール、権限を管理する役割をします。これらはすべてKong管理者により実行できますが、セキュリティ強化のために責任を分担することをお勧めします。

スーパーアドミンによるチームのワークスペースの作成
-------------------------

スーパーアドミンは、チームAとチームBの2つのチームをセットアップし、それぞれにワークスペースとアドミンを1つ/1人ずつ作成します。

1. `teamA` のワークスペースを作成します。

   ```sh
   curl -i -X POST http://localhost:8001/workspaces \
     --data name=teamA \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "name": "teamA",
     "created_at": 1531014100,
     "updated_at": 1531014100,
     "id": "1412f3a6-4d9b-4b9d-964e-60d8d63a9d46"
   }
   ```

2. `teamB` のワークスペースを作成します。

   ```sh
   curl -i -X POST http://localhost:8001/workspaces \
     --data name=teamB \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "name": "teamB",
     "created_at": 1531014143,
     "updated_at": 1531014143,
     "id": "7dee8c56-c6db-4125-b87a-b508baa33c66"
   }
   ```

スーパー管理者はチームごとに1人の管理者を作成
-----------------------

1. `teamA` の管理者を作成します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users \
     --data name=adminA \
     --data user_token=exampletokenA \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "user_token": "qv1VLIpl8kHj7lC1QOKwRdCMXanqEDii",
     "id": "4d315ff9-8c1a-4844-9ea2-21b16204a154",
     "enabled": true,
     "created_at": 1531015165,
     "updated_at": 1531015165,
     "name": "adminA"
   }
   ```

2. `teamB` の管理者を作成します。

   ```sh
   curl -i -X POST http://localhost:8001/teamB/rbac/users \
     --data name=adminB \
     --data user_token=exampletokenB \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "user_token": "IX5vHVgYqM40tLcctdmzRtHyfxB4ToYv",
     "id": "49641fc0-8c9d-4507-bc7a-2acac8f2903a",
     "enabled": true,
     "created_at": 1531015221,
     "updated_at": 1531015221,
     "name": "adminB"
   }
   ```

3. 両チームに管理者が1人ずつおり、各管理者は対応するワークスペースでのみ表示できます。確認するには、次を実行します。

   ```sh
   curl -i -X GET http://localhost:8001/teamA/rbac/users \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "total": 1,
     "data": [
       {
         "created_at": 1531014784,
         "updated_at": 1531014784,
         "id": "1faaacd1-709f-4762-8c3e-79f268ec8faf",
         "name": "adminA",
         "enabled": true,
         "user_token": "n5bhjgv0speXp4N7rSUzUj8PGnl3F5eG"
       }
     ]
   }
   ```

   同様に、ワークスペース`teamB`には独自のアドミンのみが表示されます。

   ```sh
   curl -i -X GET http://localhost:8001/teamB/rbac/users \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "total": 1,
     "data": [
       {
         "created_at": 1531014805,
         "updated_at": 1531014805,
         "id": "3a829408-c1ee-4764-8222-2d280a5de441",
         "name": "adminB",
         "enabled": true,
         "user_token": "C8b6kTTN10JFyU63ORjmCQwVbvK4maeq"
       }
     ]
   }
   ```

スーパー管理者がチームの管理者ロールを作成
---------------------

これで、スーパー管理者は各チームの RBAC 管理者ユーザーの作成を完了しました。次のタスクは、管理者ユーザーに権限を効果的に付与する管理者ロールの作成です。

管理者ロールは、そのワークスペースに限定されたすべてのAdmin APIにアクセスできる必要があります。

1. リクエストパラメータに細心の注意を払いながら、管理者ロールを設定します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/ \
     --data name=admin \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531016728,
     "updated_at": 1531016728,
     "id": "d40e61ab-8dad-4ef2-a48b-d11379f7b8d1",
     "name": "admin"
   }
   ```

2. ロールエンドポイントの権限を作成します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/admin/endpoints/ \
     --data 'endpoint=*' \
     --data 'workspace=teamA' \
     --data 'actions=*' \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "total": 1,
     "data": [
       {
         "endpoint": "*",
         "created_at": 1531017322,
         "updated_at": 1531017322,
         "role_id": "d40e61ab-8dad-4ef2-a48b-d11379f7b8d1",
         "actions": [
           "delete",
           "create",
           "update",
           "read"
         ],
         "negative": false,
         "workspace": "teamA"
       }
     ]
   }
   ```

3. ワークスペースの admin ロールに `adminA` ユーザーを追加します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users/adminA/roles/ \
     --data roles=admin \
     -H 'Kong-Admin-Token:vajeOlkbsn0q0VD9qw9B3nHYOErgY7b8'
   ```

   レスポンス: 

   ```json
   {
     "roles": [
         {
             "created_at": 1685551877,
             "updated_at": 1531014805,
             "id": "42809ada-650c-4575-b0a0-d464a64ffb70",
             "name": "admin",
             "ws_id": "9dc7adbb-9b64-4121-bf76-653cf5871bc2"
         }
     ],
     "user": {
         "comment": "null",
         "created_at": 1685552809,
         "updated_at": 1531014805,
         "enabled": true,
         "id": "bca4e390-fbbf-4a46-b55d-f4642efc14bb",
         "name": "adminA",
         "user_token": "$2b$09$oLyKTIDuKriPZ.SD5wYtxeMclGYNDn4udJkQG0NGx/Aq3j9j/tWsa",
         "user_token_ident": "0ebb5"
           }
   }
   ```

4. チームAの管理者ユーザーはチームを管理できるようになりました。それを検証するために、チームAの管理者ユーザートークンを使用してチームBのRBACユーザーを一覧表示してみましょう。

   ```sh
   curl -i -X GET http://localhost:8001/teamB/rbac/users \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   エンドポイントにアクセスできないことに注意してください。

   ```json
   {
     "message": "Invalid RBAC credentials"
   }
   ```

5. 次に、チームAのワークスペースでRBACユーザーを一覧表示してみます。

   ```sh
   curl -i -X GET http://localhost:8001/teamA/rbac/users \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "total": 1,
     "data": [
       {
         "created_at": 1531014784,
         "updated_at": 1531014784,
         "id": "1faaacd1-709f-4762-8c3e-79f268ec8faf",
         "name": "adminA",
         "enabled": true,
         "user_token": "n5bhjgv0speXp4N7rSUzUj8PGnl3F5eG"
       }
     ]
   }
   ```

同じ手順をチームBでも繰り返す場合、似通ったセットアップ（アドミンロール1，アドミンユーザー1名、いずれもこのチームのワークスペースに限定）になります。

これでスーパー管理者の仕事は完了です。個々のチーム管理者は、チームのユーザーとエンティティを設定できるようになりました。

チーム管理者が通常のユーザーを作成
-----------------

この時点から、チーム管理者はプロセスを推進できます。次の手順では、チームAやBに所属するエンジニアなど、チームユーザーを作成します。では、管理者Aのユーザートークンを使用して、それを実行しましょう。

通常ユーザーを作成するには、作成前にそのユーザー用のロールが利用可能になっている必要があります。このロールには、RBACとワークスペースを除くすべてのAdmin APIエンドポイントに対する権限が必要です。通常のユーザーはこれらのエンドポイントにアクセスする必要はありません。アクセスが必要な場合、管理者はそれらを個別に付与できます。

### ユーザーロールの作成

1. チーム管理者として、通常の`users`ロールを作成します.

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/ \
     --data name=users \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531020346,
     "updated_at": 1531020346,
     "id": "9846b92c-6820-4741-ac31-425b3d6abc5b",
     "name": "users"
   }
   ```

2. すべてのAdmin APIに対する権限を作成します。これには、 `\*`に対する肯定的な権限が必要です。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/users/endpoints/ \
     --data 'endpoint=*' \
     --data 'workspace=teamA' \
     --data 'actions=*' \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "endpoint": "*",
     "created_at": 1531020573,
     "updated_at": 1531020573,
     "role_id": "9846b92c-6820-4741-ac31-425b3d6abc5b",
     "actions": [
       "delete",
       "create",
       "update",
       "read"
     ],
     "negative": false,
     "workspace": "teamA"
   }
   ```

3. 次に、RBACと否定的な権限を持つワークスペースを除外します。

   RBACエンドポイントをフィルタリングします。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/users/endpoints/ \
     --data 'endpoint=/rbac/*' \
     --data 'workspace=teamA' \
     --data 'actions=*' \
     --data 'negative=true' \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "endpoint": "/rbac/*",
     "created_at": 1531020744,
     "updated_at": 1531020744,
     "role_id": "9846b92c-6820-4741-ac31-425b3d6abc5b",
     "actions": [
       "delete",
       "create",
       "update",
       "read"
     ],
     "negative": true,
     "workspace": "teamA"
   }
   ```

   ワークスペースのエンドポイントをフィルタリングします。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/users/endpoints/ \
     --data 'endpoint=/workspaces/*' \
     --data 'workspace=teamA' \
     --data 'actions=*' \
     --data 'negative=true' \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "endpoint": "/workspaces/*",
     "created_at": 1531020778,
     "updated_at": 1531020778,
     "role_id": "9846b92c-6820-4741-ac31-425b3d6abc5b",
     "actions": [
       "delete",
       "create",
       "update",
       "read"
     ],
     "negative": true,
     "workspace": "teamA"
   }
   ```

{:.important}
> 
> **重要** : [権限のワイルドカード](#wildcards-in-permissions)セクションで説明したように、`*` の意味は一般的なグロ部と同じではありません。そのため、`/rbac/*` または `/workspaces/*` は、すべての RBAC およびワークスペースのエンドポイントと一致するわけではありません。たとえば、すべての RBAC API をカバーするには、次のエンドポイントの権限を定義する必要があります。
> 
> * `/rbac/*`
> * `/rbac/*/*`
> * `/rbac/*/*/*`
> * `/rbac/*/*/*/*`
> * `/rbac/*/*/*/*/*`

### チームにメンバーを追加する

チームAに、`foogineer` と `bargineer` の新しいメンバーが2人加わりました。管理者Aは、RBACユーザーを作成し、Kongへのアクセス権を付与することで、彼らをチームに迎えます。

1. `foogineer`を作成。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users \
     --data name=foogineer \
     --data user_token=exampletokenfoo \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531019797,
     "updated_at": 1531019797,
     "id": "0b4111da-2827-4767-8651-a327f7a559e9",
     "name": "foogineer",
     "enabled": true,
     "user_token": "dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI"
   }
   ```

2. `foogineer` を `users` のロールに追加します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users/foogineer/roles \
     --data roles=users \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "roles": [
       {
         "comment": "Default user role generated for foogineer",
         "created_at": 1531019797,
         "updated_at": 1531019797,
         "id": "125c4212-b882-432d-a323-9cbe38b1d0df",
         "name": "foogineer"
       },
       {
         "created_at": 1531020346,
         "updated_at": 1531020346,
         "id": "9846b92c-6820-4741-ac31-425b3d6abc5b",
         "name": "users"
       }
     ],
     "user": {
       "created_at": 1531019797,
       "updated_at": 1531019797,
       "id": "0b4111da-2827-4767-8651-a327f7a559e9",
       "name": "foogineer",
       "enabled": true,
       "user_token": "dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI"
     }
   }
   ```

3. `bargineer`を作成。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users \
     --data name=bargineer \
     --data user_token=exampletokenbar \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531019797,
     "updated_at": 1531019797,
     "id": "e8926efa-11b4-43a3-9a28-767c05d8e9d8",
     "name": "bargineer",
     "enabled": true,
     "user_token": "dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI"
   }
   ```

4. `bargineer` を `users` のロールに追加します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users/foogineer/roles \
     --data roles=users \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "roles": [
       {
         "comment": "Default user role generated for bargineer",
         "created_at": 1531019797,
         "updated_at": 1531019797,
         "id": "125c4212-b882-432d-a323-9cbe38b1d0df",
         "name": "bargineer"
       },
       {
         "created_at": 1531020346,
         "updated_at": 1531020346,
         "id": "9846b92c-6820-4741-ac31-425b3d6abc5b",
         "name": "users"
       }
     ],
     "user": {
       "created_at": 1531019797,
       "updated_at": 1531019797,
       "id": "e8926efa-11b4-43a3-9a28-767c05d8e9d8",
       "name": "bargineer",
       "enabled": true,
       "user_token": "dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI"
     }
   }
   ```

通常のチームユーザーはトークンを使用します
---------------------

`foogineer`と`bargineer`は、チームAのアドミンからRBACユーザートークンを取得し、チームAのワークスペースの範囲内で{{site.base_gateway}}
を探索できるようになっています。
これを検証してみましょう。

1. `foogineer` として、ワークスペースを一覧表示してみてください。

   ```sh
   curl -i -X GET http://localhost:8001/teamA/workspaces/ \
     -H 'Kong-Admin-Token:dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI'
   ```

   レスポンス: 

   ```json
   {
     "message": "foogineer, you do not have permissions to read this resource"
   }
   ```

2. いくつかのプラグイン、たとえば`key-auth`を有効にします。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/plugins \
     --data name=key-auth \
     -H 'Kong-Admin-Token:dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531021732,
     "updated_at": 1531021732,
     "config": {
       "key_in_body": false,
       "run_on_preflight": true,
       "anonymous": "",
       "hide_credentials": false,
       "key_names": [
         "apikey"
       ]
     },
     "id": "cdc85ef0-804b-4f92-aafd-3ff58512e445",
     "enabled": true,
     "name": "key-auth"
   }
   ```

3. 現在有効なプラグインを一覧表示します。

   ```sh
   curl -i -X GET http://localhost:8001/teamA/plugins \
     -H 'Kong-Admin-Token:dNeYvYAwvjOJdoReVJZXF8vLBXQioKkI'
   ```

   レスポンス: 

   ```json
   {
     "total": 1,
     "data": [
       {
         "created_at": 1531021732,
         "updated_at": 1531021732,
         "config": {
           "key_in_body": false,
           "run_on_preflight": true,
           "anonymous": "",
           "hide_credentials": false,
           "key_names": [
             "apikey"
           ]
         },
         "id": "cdc85ef0-804b-4f92-aafd-3ff58512e445",
         "name": "key-auth",
         "enabled": true
       }
     ]
   }
   ```

これで、実際のシナリオを使ったRBACとワークスペースの能力を示すユースケースチュートリアルを終了します。

エンティティレベルのRBAC
--------------

エンドポイントの権限に加えて、 {{site.base_gateway}} の RBAC はエンティティレベルの権限もサポートしています。これは、特定のエンティティは一意の ID で識別され、ロール内でアクセスを許可または拒否できることを意味します。

RBACは、`enforce_rbac`設定ディレクティブ、またはその`KONG_ENFORCE_RBAC`環境変数に対応するディレクティブで[強制さ](#enforcing-rbac)れます。ディレクティブは列挙型で、次の可能な値があります。

* `on`：エンドポイントレベルのアクセス制御を適用します
* `entity`:エンティティレベルのアクセス制御 **のみ** 適用します
* `both`： **エンドポイントレベルとエンティティレベルの両方のアクセス制御** を適用します。
* `off`: RBACの強制を無効にします

`entity`または`both`に設定すると、Kongはエンティティレベルのアクセス制御を強制します。
ただし、エンドポイントレベルのアクセス制御同様、強制を有効にする前に、権限をブートストラップする必要があります。

### エンティティレベルの権限の作成

チームAに、新しい臨時のチームメンバーが1人加わったところです。`qux`。チームAの管理者である管理者Aは、
すでに`qux`RBACユーザーを作成しています。

次に管理者は、`qux`がチームAのワークスペース内のエンティティに対して持つアクセスを制限する必要があり、
ユーザーにいくつかのエンティティへの読み取りアクセス権のみを与えます。そのために、管理者は
エンティティレベルのRBACを使用します。

1. 管理者Aとして、一時ユーザ`qux`のロールを作成します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles \
     --data name=qux-role \
     -H Kong-Admin-Token:exampletokenA
   ```

   レスポンス: 

   ```json
   {
     "name": "qux-role",
     "created_at": 1531065975,
     "updated_at": 1531065975,
     "id": "ffe93269-7993-4308-965e-0286d0bc87b9"
   }
   ```

2. ユーザーに、サービスとルートの2つのエンティティへの読み取りアクセス権をグラントします。各エンティティをそのIDで参照します。

   たとえば、ID `3ed24101-19a7-4a0b-a10f-2f47bcd4ff43`の`service1`と ID `d25afc46-dc59-48b2-b04f-d3ebe19f6d4b`の`route1`を使用します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/admin/entities \
     --data entity_id=3ed24101-19a7-4a0b-a10f-2f47bcd4ff43 \
     --data entity_type=services \
     --data actions=read \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531066684,
     "updated_at": 1531066684,
     "role_id": "ffe93269-7993-4308-965e-0286d0bc87b9",
     "entity_id": "3ed24101-19a7-4a0b-a10f-2f47bcd4ff43",
     "negative": false,
     "entity_type": "services",
     "actions": [
       "read"
     ]
   }
   ```

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/roles/qux-role/entities \
     --data entity_id=d25afc46-dc59-48b2-b04f-d3ebe19f6d4b \
     --data entity_type=routes \
     --data actions=read \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "created_at": 1531066684,
     "updated_at": 1531066684,
     "role_id": "ffe93269-7993-4308-965e-0286d0bc87b9",
     "entity_id": "d25afc46-dc59-48b2-b04f-d3ebe19f6d4b",
     "negative": false,
     "entity_type": "routes",
     "actions": [
       "read"
     ]
   }
   ```

3. ユーザー`qux`を`qux-role`に追加します。

   ```sh
   curl -i -X POST http://localhost:8001/teamA/rbac/users/qux/roles \
     --data roles=qux-role \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "roles": [
       {
         "comment": "Default user role generated for qux",
         "created_at": 1531065373,
         "updated_at": 1531065373,
         "name": "qux",
         "id": "31614171-4174-42b4-9fae-43c9ce14830f"
       },
       {
         "created_at": 1531065975,
         "updated_at": 1531065975,
         "name": "qux-role",
         "id": "ffe93269-7993-4308-965e-0286d0bc87b9"
       }
     ],
     "user": {
       "created_at": 1531065373,
       "created_at": 1531065373,
       "id": "4d87bf78-5824-4756-b0d0-ceaa9bd9b2d5",
       "name": "qux",
       "enabled": true,
       "user_token": "sUnv6uBehM91amYRNWESsgX3HzqoBnR5"
     }
   }
   ```

4. `qux`に対して正しい権限がリストされていることを確認してください。

   ```sh
   curl -i -X GET http://localhost:8001/teamA/rbac/users/qux/permissions \
     -H 'Kong-Admin-Token:exampletokenA'
   ```

   レスポンス: 

   ```json
   {
     "entities": {
       "d25afc46-dc59-48b2-b04f-d3ebe19f6d4b": {
         "actions": [
           "read"
         ],
         "negative": false
       },
       "3ed24101-19a7-4a0b-a10f-2f47bcd4ff43": {
         "actions": [
           "read"
         ],
         "negative": false
       }
     },
     "endpoints": {}
   }
   ```

   `qux`にはエンティティ権限が2つ必要ですが、エンドポイント権限は不要です。

アドミンAが`qux`設定を完了し、`qux`
がユーザートークンを使用してKongのAdmin APIから2つのエンティティを読み取ることができるようになりました。

アドミンAが[エンティティレベルの強制も有効化](#enforcing-rbac)しているとしましょう。
`qux`には **エンドポイントレベルの権限がない** ことに注意します。エンドポイントとエンティティの両レベルで強制が有効になっている場合、エンドポイントレベルの確認はエンティティレベルの確認の前に実行されるため、`qux`はエンティティを読み取ることができません。

1. `qux`として、すべてのRBACユーザーを一覧表示してみます。

   ```sh
   curl -i -X GET http://localhost:8001/teamA/rbac/users/ \
     -H Kong-Admin-Token:sUnv6uBehM91amYRNWESsgX3HzqoBnR5
   ```

   レスポンス: 

   ```json
   {
     "message": "qux, you do not have permissions to read this resource"
   }
   ```

2. `qux`として、`service1`にアクセスしてみます。

       curl -i -X GET http://localhost:8001/teamA/services/service1 \
         -H Kong-Admin-Token:sUnv6uBehM91amYRNWESsgX3HzqoBnR5

   レスポンス: 

   ```json
   {
     "host": "httpbin.org",
     "created_at": 1531066074,
     "updated_at": 1531066074,
     "connect_timeout": 60000,
     "id": "3ed24101-19a7-4a0b-a10f-2f47bcd4ff43",
     "protocol": "http",
     "name": "service1",
     "read_timeout": 60000,
     "port": 80,
     "path": null,
     "updated_at": 1531066074,
     "retries": 5,
     "write_timeout": 60000
   }
   ```

権限のワイルドカード
----------

RBAC は、権限の多くの側面でワイルドカード \( `*` \) の使用をサポートしています。

### エンドポイント権限の作成

`/rbac/roles/:role/endpoints` を介してエンドポイントの権限を作成するには、以下のパラメータを渡す必要があります。これらはすべて `*` 文字で置き換えることができます。

* `endpoint`: `*` **任意のエンドポイント** に一致します
* `workspace`: `*`は **任意のワークスペース** に一致します
* `actions`：`*`は、 **読み取り、更新、作成、削除など、すべてのアクション** を評価します

**特別な場合** ：`endpoint`は、単一の`*`に加えて、`/`間のURLセグメントを置き換え、エンドポイント自体の中で`*`も受け入れます。例えば、
以下はすべて有効なエンドポイントです。

* `/rbac/*`：ここで、`*`は可能なセグメントを置き換えます（例：`/rbac/users`および`/rbac/roles`
* `/services/*/plugins`:`*`は任意のサービス名またはIDに一致します

{:.note}
> 
> **注** `*` は一般的なシェルのようなグロブパターンでは **ありません** 。

`workspace`が省略された場合、現在のリクエストのワークスペースがデフォルトになります。たとえば、
`/teamA/roles/admin/endpoints` で作成されたロールエンドポイント権限は
ワークスペース `teamA` にスコープされています。

### エンティティの権限の作成

`/rbac/roles/:role/entities`経由で作成されたエンティティの権限の場合、以下のパラメータは`*`文字を受け入れます。

* `entity_id`：`*`は **任意のエンティティID** に一致します

エンティティレベルRBACにネストされたエンティティ
--------------------------

エンティティレベルのRBACを有効にすると、特定のコレクションのすべてのエンティティを一覧表示するエンドポイントは、ユーザーがアクセスできるエンティティのみを一覧表示します。

上記の例では、ユーザー`qux`がすべてのルートを一覧化した場合、ルートは、
ワークスペース内でさらに入手できる場合でも、レスポンス内でアクセス
できるエンティティのみを入手します。

```sh
curl -i -X GET http://localhost:8001/teamA/routes \
  -H 'Kong-Admin-Token:sUnv6uBehM91amYRNWESsgX3HzqoBnR5'
```

レスポンス: 

```json
{
  "next": null,
  "data": [
    {
      "created_at": 1531066253,
      "updated_at": 1531066253,
      "id": "d25afc46-dc59-48b2-b04f-d3ebe19f6d4b",
      "hosts": null,
      "preserve_host": false,
      "regex_priority": 0,
      "service": {
        "id": "3ed24101-19a7-4a0b-a10f-2f47bcd4ff43"
      },
      "paths": [
        "/anything"
      ],
      "methods": null,
      "strip_path": false,
      "protocols": [
        "http",
        "https"
      ]
    }
  ]
}
```

一部のKongエンドポイントは、レスポンスに`total`フィールドを持ちます。エンティティレベルのRBAC
を有効にすると、エンティティのグローバルカウントは表示されますが、ユーザーが
アクセスできるエンティティのみが表示されます。

たとえば、チームAに複数のプラグインが設定されているが、 `qux`がアクセスできるのはそのうちの1つだけの場合、
`/teamA/plugins`への GET リクエストの予想される出力は次のとおりです。

```sh
curl -i -X GET http://localhost:8001/teamA/plugins \
  -H 'Kong-Admin-Token:sUnv6uBehM91amYRNWESsgX3HzqoBnR5'
```

レスポンス: 

```json
{
  "total": 2,
  "data": [
    {
      "created_at": 1531070344,
      "updated_at": 1531070344,
      "config": {
        "key_in_body": false,
        "run_on_preflight": true,
        "anonymous": "",
        "hide_credentials": false,
        "key_names": [
          "apikey"
        ]
      },
      "id": "8813dd0b-3e9d-4bcf-8a10-3112654f86e7",
      "name": "key-auth",
      "enabled": true
    }
  ]
}
```

`total`フィールドは2ですが、 `qux`応答でエンティティを1つしか取得していないことに注意してください。

### エンティティレベルのRBACでのエンティティの作成

エンティティレベルのRBACは、個々の既存のエンティティへのアクセス制御を提供するため、新しいエンティティの作成には適用されません。そのため、エンドポイントレベルの権限を設定して適用する必要があります。

たとえば、エンドポイントレベル権限が実行されていない場合、`qux`は新しいエンティティを作成し、作成したエンティティにすべてのアクションを実行する権限を自動的に得ることができます。

```sh
curl -i -X POST http://localhost:8001/teamA/routes \
  --data paths[]=/anything \
  --data service.id=3ed24101-19a7-4a0b-a10f-2f47bcd4ff43 \
  --data strip_path=false \
  -H 'Kong-Admin-Token:sUnv6uBehM91amYRNWESsgX3HzqoBnR5'
```

レスポンス: 

```json
{
  "created_at": 1531070828,
  "updated_at": 1531070828,
  "strip_path": false,
  "hosts": null,
  "preserve_host": false,
  "regex_priority": 0,
  "paths": [
    "/anything"
  ],
  "service": {
    "id": "3ed24101-19a7-4a0b-a10f-2f47bcd4ff43"
  },
  "methods": null,
  "protocols": [
    "http",
    "https"
  ],
  "id": "6ee76f74-3c96-46a9-ae48-72df0717d244"
}
```

