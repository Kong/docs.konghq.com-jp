クライアントが認証されると、プラグインはアップストリームサービスにプロキシする前にリクエストへいくつかのヘッダーを追加し、以下のコードでコンシューマを識別できるようにします。

* `X-Consumer-ID`: Kong内のコンシューマのID。
* `X-Consumer-Custom-ID`: コンシューマの`custom_id`（設定されている場合）。
* `X-Consumer-Username`: コンシューマの`username`（設定されている場合）。
* `X-Credential-Identifier`: 認証情報の識別子（コンシューマが`anonymous`コンシューマでない場合にのみ）。
* `X-Anonymous-Consumer`: 認証に失敗した場合は`true`に設定され、代わりに`anonymous`コンシューマが設定されます。

この情報を利用して、追加のロジックを実装することができます。`X-Consumer-ID`値を使用してKong Admin APIのクエリを実行し、コンシューマについての詳細な情報を取得できます。

{% if_version lte:2.8.x %}

{:.important}
> 
> **重要** : Kong 2\.1では`X-Credential-Username`は非推奨となり、代わりに`X-Credential-Identifier`が推奨されています。

{% endif_version %}

