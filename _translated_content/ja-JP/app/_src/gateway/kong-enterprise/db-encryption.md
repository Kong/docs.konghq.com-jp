---
title: "キーリングとデータ暗号化"
badge: "enterprise"
---

{{site.base_gateway}} は、コンシューマのシークレットなどの機密データフィールドをデータベース内に暗号化された形式で保存するメカニズムを提供します。これにより、Kong クラスタでの保存データの暗号化に対するセキュリティ制御を実現します。

この機能により、保存中の機密データフィールドの透過的かつ対称的な暗号化が実現します。 *透過性* とは、有効にすると、データの暗号化/復号化が、データベースへの書き込み直前/データベースからの読み取り直後にKongによってオンザフライで行われるという事実を指します。機密フィールドを含むAdmin APIによって生成された応答では、引き続きデータがプレーンテキストとして表示され、機密フィールドへのアクセスを必要とするKongのランタイム要素（プラグインなど）は、追加の構成を必要とせずに透過的にアクセスします。

使用開始する
------

このドキュメントでは、{{site.base_gateway}}対称データベース暗号化を利用して機密設定データをKongクラスタに保存する方法について、簡単に紹介します。このドキュメントでは、 `cluster`モードでデータベース暗号化を使用する場合に必要なKong構成設定とキー管理ライフサイクル手順について説明します。このモードは外部依存関係を必要としないため、開始するための最も簡単な方法を提供します。

### 管理RSAキーペアの生成

データベース暗号化を有効にする前に、キーリングマテリアルのエクスポートと回復に使用する RSA キーペアを生成することを強くお勧めします。キーリングマテリアルが失われた場合、そのキーリングのキーを使用してデータベースに書き込まれた機密データフィールドを回復することはできません。

RSAキーペアの生成は、`openssl`CLIを使用すると簡単です。

```bash
openssl genrsa -out key.pem 2048
openssl rsa -in key.pem -pubout -out cert.pem
```

このキーペアは、キーリングの回復、エクスポート、再インポートを容易にするために Kong に提供されます。パブリック`cert.pem`とプライベート`key.pem` 、既存のシークレット管理ポリシーに従って安全に保存する必要があります。RSA キーペアの安全な保管の詳細については、このドキュメントの範囲外です。

### データベース暗号化のために Kong を構成

Kongでデータ暗号化を有効にするには、Kongの構成を変更する必要があります。`kong.conf`に次の値を設定します。

    keyring_enabled = on
    keyring_strategy = cluster
    keyring_recovery_public_key = /path/to/generated/cert.pem

または環境変数を設定します。

    export KONG_KEYRING_ENABLED=on
    export KONG_KEYRING_STRATEGY=cluster
    export KONG_KEYRING_RECOVERY_PUBLIC_KEY=/path/to/generated/cert.pem

Kongクラスタのすべてのノードでは、通常、`keyring_enabled`と`keyring_strategy`で同じ構成値を共有します。
生成された秘密鍵`key.pem`は、Kongの構成または起動に必要ありません。キーリング管理で秘密鍵を使用する方法の詳細については、「[障害復旧](#disaster-recovery)」セクションを参照してください。

### Kongの起動

クラスタ内のすべてのKongノードを構成したら、各ノードを起動します。

```bash
kong start
```

Kongノードで暗号化が有効になっていると、起動時にクラスターキーリングの状態がチェックされます。キーリングにキーが存在しないことが検出された場合は、自動的にキーが生成されます。このプロセスにより、暗号化/復号化処理を直ちに開始することができます。

他のすべてのノードでは、生成されたキーは数秒以内に自動的に配布されます。

暗号化キーはメモリ内に *のみ* 保持され、Kongプロセスの再起動（例： `kong restart`の実行）後は保持されません。この制限があるため、初期化後にキーリングを[回復](#recover-the-keyring)または[インポート](#import-the-keyring)する必要があります。回復またはインポートを行わない場合、クラスタ内のすべてのKongノードが同時に再起動すると、キーリングで書き込まれた機密フィールドはすべて回復できなくなります。キーマテリアルは、Kongのソフトリロード後（ `kong reload`）も保持されます。

### クラスタキーリングを確認

キーリングを有効にしてKongを起動したら、以下のようにキーリングの内容を確認します。

```bash
curl -s localhost:8001/keyring | jq
```

レスポンス：

```json
{
  "ids": [
    "LaW1urRQ"
  ],
  "active": "LaW1urRQ"
}
```

この例では、値 `LaW1urRQ` はキーの *ID* であり、キーマテリアル自体ではないことに注意してください。

### 暗号化ルーチンの実行

ベーシック認証の認証情報を使用してコンシューマを作成します。この時点で、ベーシック認証の認証情報の `password` フィールドは、データベースに書き込まれる前に対称的に暗号化されます（ベーシック認証プラグインによってハッシュ化されるのに加えて、これはキーリング暗号化が有効であるかどうかに関係なくプラグインによって行われます）。

```bash
curl -s localhost:8001/consumers -d username=bob | jq
```

レスポンス：

```json
{
  "custom_id": null,
  "created_at": 1576518610,
  "id": "6375b5fd-9c95-4822-b2dd-80ffbccb7ec9",
  "tags": null,
  "username": "bob",
  "type": 0
}
```

```bash
curl -s localhost:8001/consumers/bob/basic-auth -d username=bob -d password=supersecretpassword | jq
```

レスポンス：

```json
{
  "created_at": 1576518704,
  "consumer": {
    "id": "6375b5fd-9c95-4822-b2dd-80ffbccb7ec9"
  },
  "id": "fc46ce48-c1d6-4078-9f51-5a777350a8a2",
  "password": "da61c0083b6d19ef3db2490d0da96a71572da0fa",
  "username": "bob"
}
```

API 応答で返される`password`フィールドは暗号化された値ではないことに注意してください。データベース暗号化により、Admin APIで生成された応答は変更されません。これにより、内部で保存時の暗号化セキュリティを確保しながら、既存のワークフローを同じ形式で継続できます。これを確認するには、以下のようにデータベースに保存されている値を調べます。

    kong=# select id,password from basicauth_credentials;
                      id                  |                                                         password
    --------------------------------------+---------------------------------------------------------------------------------------------------------------------------
     fc46ce48-c1d6-4078-9f51-5a777350a8a2 | $ke$1$-LaW1urRQ-0f5b1ee8ddeefca1a1d75125-53f158a5f619133a2113692a7057e2b91fa947321de4480d452dd42c36bc9ef8aa6499cd429db6d7
    (1 row)

また、認証情報が作成された後にそれを読み込み直しても、期待どおりに動作することを確認することもできます。

```bash
curl -s localhost:8001/consumers/bob/basic-auth/fc46ce48-c1d6-4078-9f51-5a777350a8a2 | jq
```

レスポンス：

```json
{
  "created_at": 1576518704,
  "consumer": {
    "id": "6375b5fd-9c95-4822-b2dd-80ffbccb7ec9"
  },
  "id": "fc46ce48-c1d6-4078-9f51-5a777350a8a2",
  "password": "da61c0083b6d19ef3db2490d0da96a71572da0fa",
  "username": "bob"
}
```

### キーマテリアルのローテーション

上述したとおり、Kongはキーリング上に複数のキーが同時に併存できるようにすることで、キーのローテーションをサポートします。これにより、1つのキーで記述されたデータフィールドを読み返すことができると同時に、新しい暗号化キーを書き込み操作に使用できます。キーをローテーションするには、キーリングに新しいキーをインポートまたは生成して、それをアクティブとマークするだけで済みます。任意のキーマテリアルを`/keyring/import`マテリアルからインポートすることもKongで`/keyring/generate`エンドポイントから新しいキーを生成することもできます。

```bash
curl -XPOST -s localhost:8001/keyring/generate
```

レスポンス：

```json
{
  "key": "t6NWgbj3g9cbNVC3/D6oZ2Md1Br5gWtRrqb1T2FZy44=",
  "id": "8zgITLQh"
}
```

便宜上、このエンドポイントの呼び出しからは生のキーマテリアルが返されることに注意してください。

キーリングに新しいキーが追加されたら、キーのIDをアクティブにします。

```bash
curl -s localhost:8001/keyring/activate -d key=8zgITLQh
```

Kong は、現在の`active`キーを使用して新しい機密データフィールドを書き込み、そのキーがキーリング内にある場合は、以前のキーを使用してデータベースに以前に書き込まれたフィールドを読み取ることができます。Kong は、データベースからフィールドを復号化するときに使用する適切なキーを自動的に選択します。

この時点で、 `/keyring/export`Admin APIエンドポイントを介してキーリングのバックアップを再度作成することをお勧めします。

### 暗号化データフィールドのローテーション

キーリングが更新されると、既存の暗号化フィールドは新しいアクティブなキーを使用するように変更できます。このローテーションを実現するには、対象のエンティティに PATCH リクエストを送信します。

```bash
curl -XPATCH -s localhost:8001/consumers/bob/basic-auth/fc46ce48-c1d6-4078-9f51-5a777350a8a2 -d password=adifferentsecretpassword | jq
```

レスポンス：

```json
{
  "created_at": 1576518704,
  "consumer": {
    "id": "6375b5fd-9c95-4822-b2dd-80ffbccb7ec9"
  },
  "id": "fc46ce48-c1d6-4078-9f51-5a777350a8a2",
  "password": "cc7274e94e41f3e00c238ff8712d1a83693f2a89",
  "username": "bob"
}
```

繰り返しになりますが、暗号化の動作は Admin API のレスポンスには透過的であることに注意してください。内部的には、Kong は読み取り専用の暗号化キーを使用してエンティティの `password` フィールドを読み取ります。PATCH プロセスの一環として、Kong は `password` フィールドを新しい`active` キーを使用してデータベースに書き戻します。このプロセスを確認するには、データベースの生のコンテンツを調べます。

    kong=# select id,password from basicauth_credentials;
                      id                  |                                                         password
    --------------------------------------+---------------------------------------------------------------------------------------------------------------------------
     fc46ce48-c1d6-4078-9f51-5a777350a8a2 | $ke$1$-8zgITLQh-b8d28531252241e6b95907e4-0768a9a4baaa2c777d9406d4e3098d813be55ae82c4c849182b06b9c5954704c6290c9e677bcd693
    (1 row)

パスワードデータBLOBないのキー識別子が更新されたことにご留意ください。このローテーションメカニズムにより、組織はキー管理ローテーションとリテンションポリシーに関する特定のコンプライアンスニーズを満たすことができます。

現在、暗号化されたフィールドをローテーションするには、暗号化されたフィールドを直接書き込み操作する必要があります。Kongの将来のリリースには、このプロセスを自動化するヘルパーAPI操作が含まれる可能性があります。

### 欠落しているキーリングマテリアル動作の調査

試しに、クラスタ内のすべてのKongノードを停止してから1つのKongノードを再起動します。ただし、キーリングマテリアルはインポートしません。暗号化フィールドがあるエンティティを読み取ろうとする動作は、次のように変更されます。

```bash
time curl -s localhost:8001/consumers/bob/basic-auth/fc46ce48-c1d6-4078-9f51-5a777350a8a2
```

レスポンス：

    {"message":"An unexpected error occurred"}
    real	0m24.811s
    user	0m0.017s
    sys	0m0.006s

Kongノードが再起動すると、キーリングのブロードキャストリクエストが送信されますが、クラスタ内に他のノードが存在しない（または他のノードが使用可能なキーリングを持っていない）ため、Kongは認証情報の`password`フィールドを復号化できません。Kongには最終的に一貫性のあるクラスタモデルがあるため、キーリングのリクエストを複数回試行し、別のクラスタノードからのレスポンスを聞く際に遅延が発生します。そのため、リクエストを完了するまでに数秒かかり、最終的に失敗します。

障害復旧
----

`cluster`ストラテジは2通りの災害復旧ストラテジをサポートしています。

### 自動バックアップと手動リカバリ

キーリングマテリアルの自動バックアップを使用するには、 `keyring_recovery_public_key` を設定します。

    keyring_recovery_public_key = /path/to/generated/cert.pem

または環境変数を設定します。

    export KONG_KEYRING_RECOVERY_PUBLIC_KEY=/path/to/generated/cert.pem

#### キーリングの回復

キーリングマテリアルは、データベースの `keyring_recovery_public_key` の Kong 構成値で定義された公開 RSA キーで暗号化されます。対応する秘密鍵を使用して、データベース内のキーリングマテリアルを復号化できます。

```bash
curl -X POST localhost:8001/keyring/recover -F recovery_private_key=@/path/to/generated/key.pem
```

レスポンス：

```json
{
  "message": "successfully recovered 1 keys",
    "recovered": [
        "RfsDJ2Ol"
    ],
    "not_recovered": [
        "xSD219lH"
    ]
}
```

レスポンスには、正常に回復されたキーと回復できなかったキーのリストが含まれます。Kongエラーログには、キーを回復できなかった詳細な理由が含まれます。

### 手動エクスポートと手動インポート

**このメソッドの使用は Kong 3\.0 では非推奨であり、将来のリリースでは削除される予定です。** 

    keyring_public_key = /path/to/generated/cert.pem
    keyring_private_key = /path/to/generated/key.pem

または環境変数を設定します。

    export KONG_KEYRING_PUBLIC_KEY=/path/to/generated/cert.pem
    export KONG_KEYRING_PRIVATE_KEY=/path/to/generated/key.pem

管理 RSA キーペアはバックアップとリカバリのプロセスにのみ使用されるため、すべてのノードに管理 RSA キーペアを提供する必要はありません。通常のデータベースの読み取りや書き込み操作を行うために存在する必要はありません。

Kongワーカープロセスが実行されているユーザーは、キーリングをインポートおよびエクスポートできるようにするために、公開鍵と秘密鍵への読み取りアクセス権を持っている必要があります。このユーザーは、 `nginx_user` Kong構成オプションで定義されています。これらのファイルへのアクセスを制限することをお勧めします。

```bash
chown <nginx_user id="sl-md0000000">:<nginx_user id="sl-md0000000"> /path/to/generated/cert.pem /path/to/generated/key.pem
chmod 400 /path/to/generated/cert.pem /path/to/generated/key.pem
```

テスト時には、kong.conf で`keyring_blob_path`設定するか、環境変数を使用して、 `KONG_KEYRING_BLOB_PATH`をせっていし、
既知のキーをダンプするパスを指定します。ダンプされたキーは
`keyring_public_key` Kong構成値で定義された公開RSAキーで暗号化され
Kong の起動時に自動的に読み込まれます。

#### キーリングのエクスポート

キーリングをエクスポートします。エクスポートされたマテリアルは、停電が発生した場合や、以前に削除されたキーを復元するためにクラスタに再インポートできます。

```bash
curl -XPOST -s localhost:8001/keyring/export | jq
```

レスポンス：

```json
{
  "data": "eyJrIjoiV1JZeTdubDlYeFZpR3VVQWtWTXBcL0JiVW1jMWZrWHluc0dKd3N4M1c0MlIxWE5XM05lZ05sdFdIVmJ1d0ZnaVZSTnFSdmM1WERscGY3b0NIZ1ZDQ3JvTFJ4czFnRURhOXpJT0tVV0prM2lhd0VLMHpKTXdwRDd5ZjV2VFYzQTY0Y2UxcVl1emJoSTI4VUZ1ZExRZWljVjd2T3BYblVvU3dOY3IzblhJQWhyWlcxc1grWXE3aHM1RzhLRXY2OWlRamJBTXAwbHZmTWNFWWxTOW9NUjdnSm5xZWlST0J1Q09iMm5tSXg0Qk1uaTJGalZzQzBtd2R2dmJyYWxYa3VLYXhpRWZvQm9EODk3MEtVcDYzY05lWGdJclpjang4YmJDV1lDRHlEVmExdGt5c0g1TjBJM0hTNDRQK1dyT2JkcElCUk5vSVZVNis1QWdcLzdZM290RUdzN1E9PSIsImQiOiIxWEZJOXZKQ05CTW5uVTB5c0hQenVjSG5nc2c5UURxQmcxZ3g1VVYxNWNlOEVTTlZXTmthYm8zdlUzS2VRTURcL0RUYXdzZCtJWHB5SllBTkRtanZNcytqU2lrVTFiRkpyMEVcLzBSRlg2emJrT0oybTR2bXlxdVE9PSIsIm4iOiJUUmRLK01Qajh6MkdHTmtyIn0="
}

```

生成される応答は、キーリングを含む不透明なblobです。このblobを暗号化するためにランダムに生成される対称鍵は、`keyring_public_key`Kong構成値を介して定義されるRSA公開鍵で暗号化されます。

エクスポートされたキーリングは、災害復旧目的のために安全な場所に保管する必要があります。災害復旧プロセスに使用される前に、修正したり複合化したりするようには設計されていません。

#### キーリングのインポート

Kongを再起動し、以前にエクスポートしたキーリングを再度インポートします。

```bash
kong restart
```

```bash
curl localhost:8001/keyring/import -d data=<exported data id="sl-md0000000">
```

この操作では、`keyring_private_key` がキーリングを最初にエクスポートするときに使用された公開鍵に関連付けられたプライベートRSAキーを指している必要があります。これが完了すると、暗号化/復号化のためにキーリングを必要とするAdmin API操作が確認できるようになります。

```bash
curl -s localhost:8001/consumers/bob/basic-auth/fc46ce48-c1d6-4078-9f51-5a777350a8a2 | jq
```

レスポンス：

```json
{
  "created_at": 1576518704,
  "consumer": {
    "id": "6375b5fd-9c95-4822-b2dd-80ffbccb7ec9"
  },
  "id": "fc46ce48-c1d6-4078-9f51-5a777350a8a2",
  "password": "da61c0083b6d19ef3db2490d0da96a71572da0fa",
  "username": "bob"
}
```

実装の詳細
-----

### 暗号化/復号化


{{site.base_gateway}}はGCMモードで256ビットAES暗号化を使用します。各暗号化ルーチン実行の暗号化nonce値は、カーネルCSPRNG（`/dev/urandom`）から導出されます。Kongで使用されるAESルーチンは、{{site.base_gateway}}パッケージにバンドルされているOpenSSLライブラリによって提供されます。

### キーの生成とライフサイクル


{{site.base_gateway}}のキーリング処理メカニズムでは、一度に複数のキーを任意のKongノードに併存させることができます。各キーはデータベースから暗号化されたフィールドの読み取りにしようできますが、暗号化されたフィールドをデータストアに書き戻すために使用できるキーは一度に1つのみです。このプロセスで確立されるキーのローテーションメカニズムでは、新しいキーリングマテリアルが導入されても、古い方のキーはしばらく維持され以前に暗号化されたフィールドをローテーションできます。

Kong はカーネル CSPRNG を通して、`/keyring/generate` Admin API エンドポイントで生成したキーリングマテリアルを取得します。Kong は、すべての Kong ワーカープロセスがアクセスする共有メモリゾーンにキーリングマテリアルを保存します。メモリのページング操作の一環としてキーマテリアルがディスクに書き込まれないように、Kong を実行しているシステムではスワップを無効にすることをお勧めします。

`cluster` モードで動作している場合、キーリングマテリアルは Kong クラスタ内のすべてのノード間で自動的に伝達されます。Kong ノードには直接的なピアツーピア通信という概念がないため、基盤となるデータストアはメッセージを送信するための通信チャネルとして機能します。Kong ノードが起動すると、一時的な RSA キーペアが生成されます。ノードの公開鍵は、クラスタ内の他のすべてのアクティブノードに伝播されます。アクティブノードは、キーリングマテリアルを要求するメッセージを検知すると、提示された公開鍵でメモリ内のキーリングマテリアルを暗号化し、そのペイロードを基盤となるデータストアが提供する中央のメッセージングチャネル経由で送り返します。このプロセスにより、クラスタ内の各ノードはキーリングマテリアルを新しいノードにブロードキャストできるため、プレーンテキストのままでキーリングマテリアルをネットワーク経由で送信することを避けられます。このモデルでは、クラスタ内で少なくとも 1 つのノードが常に稼働している必要があります。すべてのノードに障害が発生した場合は、障害復旧時にキーリングを 1 つのノードに手動で再インポートする必要があります。

### 暗号化されたフィールド

キーリングモジュールは、保存時に次のフィールドを暗号化します。

* `key` `certificate`オブジェクトのフィールド（TLS証明書の秘密鍵に対応）。
* プラグイン内の特定の設定パラメータとプラグイン関連の認証 オブジェクト。どのプラグインフィールドが暗号化されているかについては、 [各プラグインのドキュメント](/hub/)を参照してください。

Vaultの統合
--------

Kongのキーリングメカニズムは、キーリングの保存とバージョン管理のために[HashiCorp Vault](https://www.vaultproject.io/)と直接統合できます。このモデルでは、Kongノードは、クラスター全体にキーリングマテリアルを生成して配布するのではなく、Vault KVシークレットエンジンからキーリングマテリアルを直接読み取ります。

キーリングの保存に Vault を使用するように Kong を構成するには、`keyring_strategy` 構成値を `vault` に設定します。Vault を活用するには、Vault アクセス用に、ホスト、マウントポイント、およびトークンまたは [Kubernetes サービスアカウント ロール](https://developer.hashicorp.com/vault/docs/auth/kubernetes)を定義する必要もあります。詳細については、[構成リファレンスを](/gateway/latest/reference/configuration/#keyring_vault_kube_api_token_file)参照してください。

### キー形式

Kongは[Vault KVシークレットエンジン](https://www.vaultproject.io/docs/secrets/kv/kv-v2.html)のバージョン2を活用します。このプロセスにより、`cluster`キーリングストラテジが提供するものと同じバージョン管理およびキーローテーションメカニズムが可能になります。KVシークレットエントリの各バージョンには、`id`フィールドと`key`フィールドの両方が含まれていなければなりません。次がその例です。

```json
{
  "key": "t6NWgbj3g9cbNVC3/D6oZ2Md1Br5gWtRrqb1T2FZy44=",
  "id": "8zgITLQh"
}
```

すべてのVault KVシークレットが一貫して消費されるように、基礎となる対称キーはシークレット値の`key`コンポーネントのSHA256として派生します。他のシステムからの対称暗号化キーを拡張または再利用する場合は、この派生に注意してください。また、Kongはマテリアルをキーリングにインポートする前にコンテンツをハッシュ化するため、この派生は`key`フィールドに任意のデータを含めることができることも意味します。

新しいキーを提供するには、構成されたパスのVaultシークレットエンジンに新しいバージョンを追加します。Vaultシークレットの`current_version`は、キーリング内のアクティブなキーを示します。詳細については、[KV v2のドキュメント](https://www.vaultproject.io/docs/secrets/kv/kv-v2.html)を参照してください。

### Vaultの権限

Vaultと通信するには、 {{site.base_gateway}}に次のいずれかを指定する必要があります。

* Vaultトークン。
* 実行中のポッドのKubernetesサービスアカウントに適用されるVaultロール。

トークンまたはKubernetes認証ロールは、キーリングシークレットが保存されているパスで`read`と`list`アクションを許可するポリシーに関連付ける必要があります。KongはキーリングマテリアルをVaultクラスタに書き込みません。

### キーリングの同期

Kongはプロセスの開始時に、Vaultからキーリングマテリアルを読み取ります。Vault KVストアに変更が加えられる変更は、Kongが`/keyring/vault/sync` Admin APIエンドポイントを介してVaultと同期されるまで、Kongノードに反映されません。この結果、登録と読み取り操作は一度のみ実行され、Kongは低いTTLでVaultトークンを受信できます。

### ハイブリッドモードのキーリング

キーリングはデータベースのデータを暗号化するので、データベースなしで動作してコントロールプレーンからデータを取得するKongデータプレーンノード上のデータは暗号化されません。ただし、Kong 構成内で`declarative_config_encryption_mode`オンにしたり、シークレット管理を使用して機密データをデータプレーンノードに安全に保存したりすることができます。

