---
title: "PostgreSQL TLSのトラブルシューティング"
content_type: "reference"
---
PostgreSQL
----------

### SSL接続を受け入れることができませんでした

次のいずれかのエラーメッセージが表示される場合は、SSLバージョンの不一致の問題が発生しています。

```sh
LOG:  could not accept SSL connection: wrong version number
HINT:  This may indicate that the client does not support any SSL protocol version between TLSv1.1 and TLSv1.1.
```

```sh
LOG:  could not accept SSL connection: unsupported protocol
HINT:  This may indicate that the client does not support any SSL protocol version between TLSv1.2 and TLSv1.3.
```

この場合、クライアントはサーバーがサポートしていないバージョンの TLS プロトコルを使用してサーバーに接続しようとしています。PostgreSQL がサポートするバージョンを確認し、クライアント側の TLS のバージョンを調整してください。

`pg_config --configure`コマンドを実行すると、TLSバージョンを確認できます。

KongはTLSv1\.2以上を推奨しています。下位バージョンは非推奨であり、安全とは見なされません。

### サーバー証明書ファイルを読見込むことができない

このエラーメッセージが表示される場合:

```sh
FATAL:  could not load server certificate file "server.crt": No such file or directory
```

サーバーが証明書ファイルを見つけられません。デフォルトでは、証明書ファイルの名前は`server.crt`となり、サーバーのデータディレクトリに配置されます。他の名前も使用できますが、 `ssl_cert_file`で明示的に指定する必要があります。

### 秘密鍵ファイルにアクセスできませんでした

このエラーメッセージが表示される場合:

    FATAL:  could not access private key file "server.key": No such file or directory

サーバーが秘密鍵ファイルを見つけられません。デフォルトでは、秘密鍵ファイルの名前は `server.key` で、サーバーのデータディレクトリに配置する必要があります。別の命名規則を使用した場合は、 `postgresql.conf`ファイルの`ssl_key_file`パラメータを使用して明示的に設定する必要があります。

### 間違った秘密鍵の権限

```sh
FATAL:  private key file "/certs/example.com.key" has group or world access
DETAIL:  File must have permissions u=rw (0600) or less if owned by the database user, or permissions u=rw,g=r (0640) or less if owned by root.
```

このエラーが発生した場合、秘密鍵の権限が正しくありません。エラーメッセージの情報を使用して、秘密鍵に正しい権限を適用します。

### SSL証明書の検証に失敗した

```sh
LOG:  could not accept SSL connection: tlsv1 alert unknown ca
```

クライアントがPostgreSQLサーバー証明書の検証に失敗しました。この問題を解決するには、証明書が{{site.base_gateway}}に適用されている証明書と一致していることを確認してください。

### 証明書の検証に失敗しました

```sh
LOG:  could not accept SSL connection: certificate verify failed
```

PostgreSQLサーバーはクライアント証明書の検証に失敗しました。この問題を解決するには、`postgres.conf`の`ssl_ca_file`パラメータで指定されたサーバーがCAによって信頼されていることを確認します。

### 証明書の認証に失敗しました`user`

```sh
LOG:  provided user name (kong) and authenticated user name (foo@example.com) do not match
FATAL:  certificate authentication failed for user "kong"
```

このエラーメッセージは、クライアント証明書がデータベースで指定されたユーザー名と一致しない場合に表示されます。

### 接続には有効なクライアント証明書が必要です

```sh
FATAL:  connection requires a valid client certificate
```

このエラーは、PostgreSQLサーバーが認証にクライアント証明書を必要とするが、クライアント（この場合は{{site.base_gateway}}）が有効な証明書を提供できなかった場合に発生します。このタイプのエラーの場合は、`pg_ssl_cert`と`pg_ssl_cert_key`が`kong.conf`で正しく設定されているかどうかを確認してください。

{{site.base_gateway}}でのTLSのトラブルシューティング
----------------------

### プロトコルのバージョンが一致しません

次のいずれかのエラーメッセージが表示された場合、

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: tlsv1 alert protocol version
```

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: unsupported protocol
```

サーバーとクライアントのバージョンが同期していません。PostgreSQLサーバーが使用しているバージョンを確認し、それに応じて{{site.base_gateway}}バージョンを調整します。

### サーバーはSSL接続をサポートしていません

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: the server does not support SSL connections
```

このエラーは、PostgreSQLでSSLがサポートされていない場合に発生します。[設定手順](/gateway/{{page.release}}/production/networking/configure-postgres-tls/)に従ってPostgreSQLを構成します。

### 証明書の検証に失敗しました

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: certificate verify failed
```

このエラーは、{{site.base_gateway}}がPostgreSQLサーバー証明書の検証に失敗した場合に起こります。PostgreSQLサーバーに信頼できる証明書が設定されており、対応するCAチェーンが`kong.conf`内の`lua_ssl_trusted_certificate`に正しく設定されていることを確認します。

### 接続には有効なクライアント証明書が必要です

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: FATAL: connection requires a valid client certificate
```


{{site.base_gateway}}認証にクライアント証明書を必要としますが、クライアントは有効な証明書を提供できませんでした。

このエラーは、PostgreSQL サーバーが認証にクライアント証明書が必要とされていながら、 {{site.base_gateway}}が有効な証明書を提供できない場合に発生します。このタイプのエラーの場合は、 `pg_ssl_cert`と`pg_ssl_cert_key`が`kong.conf`で正しく設定されているかどうかを確認してください。

### TLSv1アラート不明CA

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: tlsv1 alert unknown ca
```

PostgreSQLサーバーはクライアント証明書の検証に失敗しました。`postgres.conf`内の`ssl_ca_file`に設定されているCAによって信頼されているクライアント証明書を使用します。

### ユーザーの証明書認証に失敗した

```sh
Error: [PostgreSQL error] failed to retrieve PostgreSQL server_version_num: FATAL: certificate authentication failed for user "kong"
```

クライアント証明書のコモンネームが、データベースに設定されているユーザー名と一致しません。[PostgreSQL構成手順](/gateway/latest/production/networking/configure-postgres-tls/)の説明に従って、 `pg_ident.conf`にユーザー名マッピングを追加します。

