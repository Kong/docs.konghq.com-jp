RBACトークン認証のみを使用する場合、Kongロールへのサービスディレクトリのマッピングは有効になりません。サービスディレクトリのマッピングでCLIアクセスを使用する必要がある場合は、Kong Managerがブラウザセッションの保護に使用するものと同じ認証メカニズムを使用できます。

#### ユーザーセッションを認証する

承認されたLDAPユーザーの認証情報を使用して、安全なCookieセッションを取得します。

```sh
$ curl -c /tmp/cookie http://localhost:8001/auth \
-H 'Kong-Admin-User: <ldap_username id="sl-md0000000">' \
--user <ldap_username id="sl-md0000000">:<ldap_password id="sl-md0000000">
```

これで、Cookie は `/tmp/cookie` に保存され、将来のリクエストで読み取ることができます。

```sh
$ curl -c /tmp/cookie -b /tmp/cookie http://localhost:8001/consumers \
-H 'Kong-Admin-User: <ldap_username id="sl-md0000000">'
```

Kong Managerはブラウザアプリケーションであるため、HTTP応答に`Set-Cookie`ヘッダーが含まれている場合、このヘッダーは将来のリクエストに自動的に添付されます。[cURLのCookieエンジン](https://ec.haxx.se/http/cookies/index.html)または[HTTPieセッション](https://httpie.org/docs/0.9.7#sessions)の使用に役立つのはそのためです。セッションを保存したくない場合は、`Set-Cookie`ヘッダーの値を`/auth`応答から直接コピーして、以降のリクエストで使用できます。

