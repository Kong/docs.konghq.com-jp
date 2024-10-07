---
title: "シークレット管理"
---
シークレットとは、APIゲートウェイの操作に必要な機密情報です。シークレットはプラグインで使用される可能性のあるコア{{site.base_gateway}}構成の一部、またはゲートウェイによって処理されるAPIに関連する構成の一部である可能性があります。

{{site.base_gateway}}で使用される最も一般的なシークレットのタイプには以下のものが含まれます。

* PostgreSQL および Redis で使用するデータストアのユーザー名とパスワード
* プライベートX.509証明書
* API キー
* 一般的に認証、ハッシュ化、署名、暗号化に使用される機密性の高いプラグイン構成フィールド。


{{site.base_gateway}} では、特定の値を vault に保存します。
機密情報をシークレットとして保存することで、`kong.conf`、
宣言型構成ファイル、ログ、または Kong Manager UI など、プラットフォーム全体でプレーンテキストで
表示されないようにします。代わりに、
`vault` リファレンスを使用して各シークレットを参照することができます。

たとえば、次のリファレンスは、環境変数`MY_SECRET_POSTGRES_PASSWORD`を解決します。

    {vault://env/my-secret-postgres-password}

上記の方法でシークレット管理が一元化されます。

参照可能な値
------

シークレットリファレンスは文字列値を指します。他のデータタイプは現在サポートされていません。

Vaultバックエンドでは、関連するシークレットをオブジェクト内に複数保存することができますが、参照が指すのは常に文字列値に変換されるキーです。
たとえば、次の参照があるとします。

    {vault://hcv/pg/username}

HashiCorp Vault内の`pg`と呼ばれるシークレットオブジェクトを指し、次の値を返す可能性があります。

```json
{
  "username": "john",
  "password": "doe"
}
```

Kongはペイロードを受信し、`{vault://hcv/pg/username}`の秘密参照の`"john"`の
`"username"`値を抽出します。

識別子「`pg/username`」を単一の値とするシークレットがある場合、参照に`/`をサフィックスとして追加し、Vault APIに適切に送信されるようにします。

    {vault://hcv/pg/username/}

シークレットとして保存できるもの
----------------

ほとんどの[Kongの構成](/gateway/{{page.release}}/reference/configuration/)の値は、[`pg_user`](/gateway/{{page.release}}/reference/configuration/#postgres-settings)や[`pg_password`](/gateway/{{page.release}}/reference/configuration/#postgres-settings)などのシークレットとして保存できます。

{% if_version gte:3.1.x %}

デフォルトの証明書をVaultに保存することもできます。例：

```bash
SSL_CERT=$(cat cluster.crt) \
SSL_CERT_KEY=$(cat cluster.key) \
KONG_SSL_CERT={vault://env/ssl-cert} \
KONG_SSL_CERT_KEY={vault://env/ssl-cert-key} \
kong prepare
```

{% endif_version %}

{% if_version lte:3.0.x %}

{:.note}
> 
> **制限事項：** {{site.base_gateway}}では現在、証明書キーの内容をVaultまたはファイルパスを使用する`kong.conf`設定の環境変数に格納することはサポートされていません。たとえば、 [ssl\_cert\_key](/gateway/{{page.release}}/reference/configuration/#ssl_cert_key)は、参照として保存できない証明書キー`file path`を構成します。

{% endif_version %}

[Kongライセンス](/gateway/{{page.release}}/licenses/)は通常
`KONG_LICENSE_DATA`環境変数で構成され、シークレットとして保存可能です。

Kong Admin API の[証明書オブジェクト](/gateway/{{page.release}}/admin-api/#certificate-object)はシークレットとして保存できます。

### 参照可能なプラグインフィールド

一部のプラグインには、Vaultバックエンドにシークレットとして保存できる
フィールドがあります。これらのフィールドは`referenceable`とラベル付けされています。

以下のプラグインは特定のフィールドに対して Vault リファレンスをサポートしています。
各フィールドの詳細については、各プラグインのドキュメントを参照してください。

{% referenceable_fields_table %}

サポートされているバックエンド
---------------


{{site.base_gateway}}は以下の保管庫バックエンドをサポートしています：

* 環境変数
* AWS Secrets Manager
* GCP Secret Manager
* Azure Key Vault
* HashiCorp Vault

各オプションの詳細については、[バックエンドの概要](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/)を参照してください。

使用開始
----

シークレット管理の詳細については、次のトピックを参照してください。

* [シークレット管理の使用方法](/gateway/{{page.release}}/kong-enterprise/secrets-management/getting-started/)
{% if_version gte:3.4.x -%}
* [シークレットローテーション](/gateway/{{page.release}}/kong-enterprise/secrets-management/secrets-rotation/)
{% endif_version -%}
* [バックエンドの概要](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/)
* [参照形式](/gateway/{{page.release}}/kong-enterprise/secrets-management/reference-format/)
* [高度な使用法](/gateway/{{page.release}}/kong-enterprise/secrets-management/advanced-usage/)

