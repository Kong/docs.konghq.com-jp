---
title: "PostgreSQL TLS の構成"
content_type: "tutorial"
---
TLSを構成すると、 {{site.base_gateway}}とPostgreSQLサーバー間で送信されるすべてのデータが暗号化され、 {{site.base_gateway}}にセキュリティレイヤーが追加されます。このアプローチには、いくつかの長所と短所があります。

* **長所** ：
  * 認証：TLSは{{site.base_gateway}}とPostgreSQLサーバー間のトラフィックを認証できます。これにより、中間者攻撃を防ぐことができます。TLS を使用すると、 {{site.base_gateway}}は正しいPostgreSQLサーバーと通信していることを確認でき、PostgreSQLサーバーは{{site.base_gateway}}またはその他のクライアントが接続を許可されていることを確認できます。
  * 暗号化: {{site.base_gateway}}とPostgreSQLサーバー間で送信されるデータは暗号化されます。この方法により、不正アクセスからデータを保護します。
  * データの整合性：TLS を使用すると、転送中にデータが保護されます。

* **デメリット** ：
  * 複雑さ：安全なTLS接続を構成および維持するには証明書の管理が必要で、{{site.base_gateway}}インスタンスの複雑さが増す可能性があります。
  * パフォーマンス：データの暗号化および復号化では追加のコンピューティングリソースが消費されるため、{{site.base_gateway}}のパフォーマンスに影響する可能性があります。

このガイドでは、転送中のデータを保護するための PostgreSQL 上および {{site.base_gateway}} 上の TLS 構成方法について説明します。

前提条件
----

* [OpenSSL](https://www.openssl.org/)
* PostgreSQLでTLSサポートが有効になりました。 
  * 事前にパッケージ化されたディストリビューションをインストールする場合、または公式のDockerイメージを使用する場合、TLSサポートはすでに含まれています。
  * ソースからPostgreSQLをコンパイルしている場合、ビルド時に`--with-ssl=openssl`フラグを使用してTLSサポートを有効にする必要があります。この場合、OpenSSL開発パッケージのインストールも必要になります。

PostgreSQLのセットアップ
-----------------

TLSを有効にしてPostgreSQLサーバーを設定するには、 `postgresql.conf`ファイルで次のパラメータを設定します。

```bash
ssl = on #This parameter tells PostgreSQL to start with `ssl`. 
ssl_cert_file = '/path/to/server.crt' # If this parameter isn't specified, the cert file must be named `server.crt` in the server's data directory. 
ssl_key_file = '/path/to/server.key' # This is the default directory, other locations and parameters can be specified. 
```

サーバーは上記設定で、TLSを使用してクライアントとセキュアな通信チャネルを確立できます。クライアントがTLSをサポートする場合、サーバーはTLSハンドシェイクを続行し、セキュアな接続を確立します。クライアントがSSLまたはTLSをサポートしていない場合、システムはセキュアでない接続にフォールバックします。

PostgreSQL 構成で参照されるキーファイルには適切な権限が必要です。このファイルの所有者がデータベースユーザーである場合は、`u=rw (0600)` に設定する必要があります。次のコマンドを使用してこれを設定します。

`chmod 0600 /path/to/server.key`

このファイルの所有者が root ユーザーである場合、ファイルの権限は `u=rw,g=r (0640)` である必要があります。次のコマンドを使用してこれを設定します。

`chmod 0640 /path/to/server.key`.

中間認証局が発行した証明書も使用できますが、`server.crt` の最初の証明書はサーバーの証明書である必要があり、かつ、サーバーの秘密鍵と一致する必要があります。

### mTLS

Mutual Transport Layer Security \(mTLS\) は、標準TLSプロトコルの上に追加のセキュリティ層を提供するプロトコルです。mTLS では、TLSハンドシェイク中にクライアントとサーバーの両方が相互に認証する必要があります。PostgreSQLでmTLS認証を使用する必要がある場合は、mTLS固有のパラメータを設定する必要があります。

```bash
ssl_ca_file = '/path/to/ca/chain.crt'
```

このパラメータは、信頼できる証明機関を含むファイルに設定する必要があります。デフォルトでは、証明書が存在する場合にのみ、構成された証明機関に対して検証されます。ログイン時にクライアント証明書を適用するには、 `cert`認証と`clientcert`認証の 2 つの方法があります。

#### `cert`の認証方法

`cert`認証メソッドでは、SSLクライアント証明書を使用して認証を実行します。このメソッドでは、[`pg_hba.conf`](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)の`hostssl`行を編集します。

`hostssl all all all cert`

`cert` 認証方法では、証明書が有効である必要があり、証明書内のコモンネーム（`cn`）がユーザー名または要求されたデータベースの適用可能なマッピングと一致することも確認します。

ユーザー名マッピングを使用すると、`cn` をデータベース内のユーザー名と異なるものにできます。ユーザー名のマッピングの詳細については、PostgreSQL 公式の[ユーザー名マッピングドキュメント](https://www.postgresql.org/docs/current/auth-username-maps.html)を参照してください。

#### `clientcert`認証オプション

`clientcert`認証オプションでは、 `hostssl`エントリの任意の認証方法とクライアント証明書の検証を組み合わせることができます。これにより、サーバーは証明書が有効であることを強制し、証明書内の`cn`がユーザー名または適切なマッピングと一致することを確認できます。

`clientcert`認証を使用するには、 `pg_hba.conf`ファイルの`hostssl`行を次のように設定します。

`hostssl all all all trust clientcert=verify-full`

この設定により、PostgreSQLはすべてのクライアントがSSL/TLS経由でサーバーに接続できるようにし、サーバーはパスワードやその他の認証形式を求めることなくクライアントを信頼できるようになります。サーバーはクライアント証明書を検証し、証明書が有効であること、および証明書上の`cn`データベース内の正しいマッピングと一致していることを確認します。

{:.note}
> 
> どちらの方法でも、パラメータ`ssl_crl_file`が設定されている場合は、クライアントがサーバーに接続するときに証明書失効リスト（CRL）もチェックされます。CRLは、無効な証明書を取り消すために使用されます。

{{site.base_gateway}} TLS構成
------------

{{site.base_gateway}}でTLSを構成するには、以下の設定を`kong.conf`構成ファイルに追加します。

```bash
pg_ssl = on
pg_ssl_required = on
pg_ssl_verify = on
pg_ssl_version = tlsv1_2
lua_ssl_trusted_certificate = system,/path/to/ca/chain.crt
lua_ssl_verify_depth = 1
```

これらのパラメータの詳細については、{{site.base_gateway}} [PostgreSQL 設定ドキュメント](/gateway/{{page.release}}/reference/configuration/#postgres-settings)、および [`lua_ssl_trusted_certificate`](/gateway/{{page.release}}/reference/configuration/#lua_ssl_trusted_certificate) と [`lua_ssl_verify_depth`](/gateway/{{page.release}}/reference/configuration/#lua_ssl_verify_depth) の特定のドキュメントを参照してください。

### {{site.base_gateway}} mTLS構成

mTLS が必要な場合、 {{site.base_gateway}}ハンドシェイクプロセス中にPostgreSQLサーバーに証明書を提供する必要があります。これは`kong.conf`に以下の設定を追加することで実現できます。

```bash
pg_ssl_cert = '/path/to/client.crt'
pg_ssl_cert_key = '/path/to/client.key'
```

* **`pg_ssl_cert`** PostgreSQL接続用のPEMで暗号化されたクライアントTLS証明書への絶対パス。この値が設定されている場合にのみ、PostgreSQLに対するTLS相互認証（mTLS）が有効になります。

* **`pg_ssl_cert_key`** `pg_ssl_cert`が設定されている場合、PostgreSQL接続用のPEMでエンコードされたクライアントTLS秘密鍵への絶対パスです。

クライアント証明書は、指定された認証局のいずれかによって **信頼されている** 必要があります。認証局は、PostgreSQL設定ファイル `postgres.conf`の `ssl_ca_file` パラメータで設定します。

証明書の作成
------

証明書は、デフォルト設定を使用してOpenSSLで作成できます。

### 中間CAチェーン

中間CAチェーンは、サーバーまたはクライアント証明書がルートCAによって直接発行されたのではなく、中間CAによって発行されたことを意味します。

ターミナルから、次の手順に従ってルートCA秘密鍵を生成し、ルートCA証明書に自己署名します。

1. ルートCA証明書のキーペアを生成します。

   ```sh
   openssl req -new -x509 -utf8 -nodes -subj "/CN=root.yourdomain.com" -config /etc/ssl/openssl.cnf -extensions v3_ca -days 3650 -keyout root.key -out root.crt
   ```

2. 中間の秘密鍵と証明書を生成します。

   1. CAの秘密鍵と証明書署名リクエスト（CSR）を生成します。

      ```sh
      openssl req -new -utf8 -nodes -subj "/CN=intermediate.yourdomain.com" -config /etc/ssl/openssl.cnf -keyout intermediate.key -out intermediate.csr
      ```

   2. 秘密鍵の権限を変更します。 

      ```sh
      chmod og-rwx intermediate.key
      ```

   3. ルートCAによって署名された中間CA証明書を作成します。

      ```sh
      openssl x509 -req -extfile /etc/ssl/openssl.cnf -extensions v3_ca -in intermediate.csr -days 1825 -CA root.crt -CAkey root.key -CAcreateserial -out intermediate.crt
      ```

3. サーバーの秘密鍵と証明書を生成します。

   1. サーバーの秘密鍵と証明書署名要求（CSR）を生成します。

      ```sh
      openssl req -new -utf8 -nodes -subj "/CN=dbhost.yourdomain.com" -config /etc/ssl/openssl.cnf -keyout server.key -out server.csr
      ```

   2. 秘密鍵の権限を変更します。 

      ```sh
      chmod og-rwx server.key
      ```

   3. 中間CAによって署名されたサーバー証明書を作成します。 

      ```sh
      openssl x509 -req -in server.csr -days 365 -CA intermediate.crt -CAkey intermediate.key -CAcreateserial -out server.crt
      ```

4. クライアントの秘密キーと証明書を生成します。

   1. クライアントの秘密鍵と証明書署名リクエスト（CSR）を生成します。

      ```sh
      openssl req -new -utf8 -nodes -subj "/CN=kong" -config /etc/ssl/openssl.cnf -keyout client.key -out client.csr
      ```

   2. 秘密鍵の権限を変更します。 

      ```sh
      chmod og-rwx client.key
      ```

   3. 中間CAによって署名されたクライアント証明書を作成します。 

      ```sh
      openssl x509 -req -in client.csr -days 365 -CA intermediate.crt -CAkey intermediate.key -CAcreateserial -out client.crt
      ```

5. 以下の手順で証明書を追加します。

   1. CA チェーンを完成させます。

      ```sh
      cat intermediate.crt > chain.crt
      cat root.crt >> chain.crt
      ```

   2. **任意** : `intermediate.crt`を`server.crt`に追加します。

      ```sh
      cat intermediate.crt >> server.crt
      ```

      この手順は通常、サーバーが中間 CA を含む証明書チェーンを使用するように構成されている場合に実行されます。サーバーが証明書チェーンを使用するように構成されていない場合、または中間 CA がすでに `server.crt` ファイルに含まれている場合、この手順を実行する必要はありません。
   3. **任意** : `intermediate.crt`を`client.crt`に追加します。

      ```sh
      cat intermediate.crt >> client.crt
      ```

      この手順は通常、クライアントが中間CAを含む証明書チェーンを使用するように構成されている場合に実行されます。クライアントが証明書チェーンを使用するように構成されていない場合、または中間CAがすでに`client.crt`ファイルに含まれている場合、この手順を実行するは必要ありません。

### 中間CAなしのチェーン

場合によっては、証明書チェーンは中間CAではなくルート証明機関によって直接署名されることがあります。このタイプのチェーンは、証明書がよく知られた信頼できるルートCAによって発行され、中間CAによって提供されるセキュリティと信頼のレベルが不要な状況で使用される場合があります。

{:.important}
> 
> **重要：** 中間CAを使用しない場合、チェーンが短くなり、通常中間CAが提供する追加のセキュリティ層が存在しないため、中間CAを使用する場合よりも安全性が低下する可能性があります。

1. ルートCA証明書のキーペアを生成します。

   ```sh
   openssl req -new -x509 -utf8 -nodes -subj "/CN=root.yourdomain.com" -config /etc/ssl/openssl.cnf -extensions v3_ca -days 3650 -keyout root.key -out root.crt
   ```

2. サーバーの秘密鍵と証明書を生成します。

   1. サーバーの秘密鍵と証明書署名要求（CSR）を生成します。

      ```sh
      openssl req -new -utf8 -nodes -subj "/CN=dbhost.yourdomain.com" -config /etc/ssl/openssl.cnf -keyout server.key -out server.csr
      ```

   2. 秘密鍵の権限を変更します。 

      ```sh
      chmod og-rwx server.key
      ```

   3. 中間CAによって署名されたサーバー証明書を作成します。 

      ```sh
      openssl x509 -req -in server.csr -days 365 -CA root.crt -CAkey root.key -CAcreateserial -out server.crt
      ```

3. クライアントの秘密キーと証明書を生成します。

   1. クライアントの秘密鍵と証明書署名リクエスト（CSR）を生成します。

      ```sh
      openssl req -new -utf8 -nodes -subj "/CN=kong" -config /etc/ssl/openssl.cnf -keyout client.key -out client.csr
      ```

   2. 秘密鍵の権限を変更します。 

      ```sh
      chmod og-rwx client.key
      ```

   3. 中間CAによって署名されたクライアント証明書を作成します。 

      ```sh
      openssl x509 -req -in client.csr -days 365 -CA root.crt -CAkey root.key -CAcreateserial -out client.crt
      ```

### テスト用の自己署名証明書

自己署名証明書は、テストに **のみ** 使用してください。

1. サーバーキーと自己署名証明書を生成します。

   ```sh
   openssl req -new -x509 -utf8 -nodes -subj "/CN=dbhost.yourdomain.com" -config /etc/ssl/openssl.cnf -days 365 -keyout server.key -out server.crt
   ```

2. 秘密鍵の権限を変更します。

   ```sh
   chmod og-rwx server.key
   ```

3. クライアントキーと自己署名証明書を生成します。

   ```sh
   openssl req -new -x509 -utf8 -nodes -subj "/CN=kong" -config /etc/ssl/openssl.cnf -days 365 -keyout client.key -out client.crt
   ```

4. 秘密鍵の権限を変更します。

   ```sh
   chmod og-rwx client.key
   ```

自己署名証明書を使用する場合、 `ssl_ca_file`パラメータに`client.crt`を指定し、 `lua_ssl_trusted_certificate`パラメータに`server.crt`を指定する必要があります。

