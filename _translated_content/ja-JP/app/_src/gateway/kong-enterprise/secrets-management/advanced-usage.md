---
title: "高度なシークレット設定"
---
Vault実装では、さまざまな高度な構成オプションが提供されます。
[Vaultバックエンドの特定の設定パラメータについては、バックエンドリファレンスを参照してください。](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/)

{% if_version lte:3.0.x %}

{:.warning}
> 
> 現在、Kong ManagerはVaultエンティティの構成をサポートしていません。

{% endif_version %}

クエリ引数
-----

クエリ引数を使用して、Vaultバックエンドを構成できます。

たとえば、次のクエリでは、値`SECURE_`を持つ`prefix`というオプションが使用されます。

```bash
{vault://env/secret-config-value?prefix=SECURE_}
```

利用可能な設定オプションの詳細については、
それぞれの[Vaultバックエンドドキュメント](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/)を参照してください。

環境変数
----

`KONG_VAULT_<vault-backend id="sl-md0000000">_<config_opt id="sl-md0000000">`環境変数を使用して、Vaultバックエンドを構成できます。

たとえば、 {{site.base_gateway}}は`KONG_VAULT_ENV_PREFIX`に一致する環境変数を探す場合があります。

```bash
export KONG_VAULT_ENV_PREFIX=SECURE_
```

Vaults エンティティ
-------------

`vaults` エンティティを使用して、Vault バックエンドを構成できます。

Vaultエンティティは、データベースが初期化された後にのみ使用できます。データベースが初期化される *前* に使用された値のシークレットは、Vaultsエンティティを使用できません。

Vaultエンティティを作成します。

```bash
curl -i -X PUT http://HOSTNAME:8001/vaults/env-vault-1  \
  --data name=env \
  --data description='ENV vault for secrets' \
  --data config.prefix=SECRET_
```

結果：

```json
{
    "config": {
        "prefix": "SECRET_"
    },
    "created_at": 1644929952,
    "description": "ENV vault for secrets",
    "id": "684ff5ea-7f65-4377-913b-880857f39251",
    "name": "env",
    "prefix": "env-vault-1",
    "tags": null,
    "updated_at": 1644929952
}
```

構成オプションは、使用する関連[バックエンド](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/)に依存します。

これにより、環境変数とクエリ引数から構成を削除し、リファレンスでエンティティ名を使用できるようになります。

```bash
{vault://env-vault/secret-config-value}
```

Vault CLI
---------

```text
Usage: kong vault COMMAND [OPTIONS]

Vault utilities for {{site.base_gateway}}.

Example usage:
 TEST=hello kong vault get env/test

The available commands are:
  get <reference id="sl-md0000000">  Retrieves a value for <reference id="sl-md0000000">

Options:
 -c,--conf    (optional string)  configuration file
 -p,--prefix  (optional string)  override prefix directory
 --v              verbose
 --vv             debug
```

宣言型構成
-----

{:.note}
> 
> シークレット管理は、decK 1\.16 以降でサポートされています。

deckを使用して、Vaultバックエンドを構成できます。以下に例を示します。

```yaml
vaults:
- config:
    prefix: SECRET_
  description: ENV vault for secrets
  name: env
  prefix: env-vault
```

Vaultの設定と宣言型設定ファイルでのシークレットリファレンスの使用についての詳細は
[decKによるシークレット管理](/deck/latest/guides/vaults/)を参照してください。

共有構成パラメーター
----------

すべてのVaultは次の構成パラメータをサポートしています。

|           パラメータ           | UIフィールド名 |                                                          説明                                                           |
|---------------------------|----------|-----------------------------------------------------------------------------------------------------------------------|
| `vaults.description` *任意* | 説明       | Vaultのオプションの説明です。                                                                                                     |
| `vaults.name`             | 該当なし     | Vaultのタイプ。`env`、`gcp`、`aws`、または`hcv`のいずれかを受け入れます。                                                                     |
| `vaults.prefix`           | プレフィックス  | 参照プレフィックス。このVaultに保存されているシークレットにアクセスするには、このプレフィックスが必要です。`{vault://env-vault/<some-secret id="sl-md0000000">}`はその一例です。 |

ほとんどのVaultは、TTLを使用することでシークレットローテーションもサポートしています。

|             パラメータ             |      フィールド名       |                                                                                                                                                        説明                                                                                                                                                         |
|-------------------------------|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `vaults.config.ttl`           | **TTL**           | キャッシュされた場合の、vaultからのシークレットのTime to Live（秒）。特別な値0は「回転なし」を意味し、これがデフォルトです。ゼロ以外の値を使用する場合は、少なくとも1分にすることをお勧めします。                                                                                                                                                                                                        |
| `vaults.config.neg_ttl`       | **ネガティブTTL**      | Vaultのミス（シークレットではない）の有効期間（秒単位）。ネガティブにキャッシュされたシークレットは、`neg_ttl`に達するまで有効なままです。その後、Kongはシークレットを再度更新しようとします。`neg_ttl`のデフォルト値は0です。これは、ネガティブキャッシュが発生しないことを意味します。                                                                                                                                                        |
| `vaults.config.resurrect_ttl` | **Resurrect TTL** | シークレットが期限切れになった（`config.ttl` が終了した）後もシークレットが使用され続ける時間（秒単位）。これは、Vaultにアクセスできなくなった場合、またはシークレットがVaultから削除されてすぐに置き換えられない場合に役立ちます。どちらの場合も、ゲートウェイは`resurrect_ttl`秒間シークレットの更新を試行し続けます。その後、更新の試みは停止します。Vault に予期しない問題が発生した場合にシームレスな移行を確実に行うために、この構成オプションに十分に高い値を割り当てることをお勧めします。`resurrect_ttl`のデフォルト値は1e8秒で、これは約3年に相当します。 |

{% if_version gte:3.4.x %}
[シークレットローテーションの詳細についてはこちらをご覧ください](/gateway/{{page.release}}/kong-enterprise/secrets-management/secrets-rotation/)。
{% endif_version %}

