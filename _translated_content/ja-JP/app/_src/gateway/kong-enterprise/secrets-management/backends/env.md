---
title: "環境変数 Vault"
badge: "free"
content-type: "how-to"
---
ビルド時に注入できるシークレットの環境変装への保存は一般的な方法です。

環境変数による構成
---------

環境変数でシークレットを定義します。

```bash
export MY_SECRET_VALUE=EXAMPLE_VALUE
```

これで、次のシークレットを参照できます。

```text
{vault://env/my-secret-value}
```

複数のシークレットを単一の環境変数で保存したい場合は、フラットな `json` 
文字列を定義することもできます。ネストされた `json` はサポートされていません。

```bash
export PG_CREDS='{"username":"user", "password":"pass"}'
```

これにより、シークレットを個別に参照できます。

```text
{vault://env/pg-creds/username}
{vault://env/pg-creds/password}
```

{:.note}
> 
> Helm を使用して環境変数を追加する場合は、渡される変数に`kong-`が追加されていることを確認してください。

vaultsエンティティによる構成
-----------------

{:.note}
> 
> Vault エンティティは、データベースが初期化された後にのみ使用できます。データベースが初期化される *前に* 使用された値のシークレットは、Vaultsエンティティを使用できません。

{% navtabs %}
{% navtab Admin API %}

```bash
curl -i -X PUT http://HOSTNAME:8001/vaults/my-env-vault \
  --data name=env \
  --data description="Store secrets in environment variables"
```

結果：

```json
{
    "config": {
        "prefix": null
    },
    "created_at": 1644942689,
    "description": "Store secrets in environment variables",
    "id": "2911e119-ee1f-42af-a114-67061c3831e5",
    "name": "env",
    "prefix": "my-env-vault",
    "tags": null,
    "updated_at": 1644942689
}
```

{% endnavtab %}
{% navtab Declarative configuration %}

{:.note}
> 
> シークレット管理は、decK 1\.16以降でサポートされています。

次のスニペットを宣言型構成ファイルに追加します。

```yaml
_format_version: "3.0"
vaults:
- config:
    prefix: null
  description: Store secrets in environment variables
  name: env
  prefix: my-env-vault
```

{% endnavtab %}
{% endnavtabs %}

エンティティを配置すると、次のようにシークレットを参照できます。

```bash
{vault://my-env-vault/my-secret-value}
```

Vault構成オプション
------------

サポートされているツールのいずれかを使用してVaultsエンティティを設定するには、以下の設定オプションを使用します。

* Admin API
* 宣言型構成
{% if_version gte:3.1.x -%}
* Kong Manager
* {{site.konnect_short_name}} {% endif_version %}

{{site.base_gateway}}の環境変数Vaultの構成オプション:

|         パラメータ          |                                フィールド名                                |        説明        |
|------------------------|----------------------------------------------------------------------|------------------|
| `vaults.config.prefix` | **config\-prefix** \(Kong Manager\) <br> **環境変数接頭辞** \({{site.konnect_short_name}}\) | 値が格納される環境変数の接頭辞。 |

一般的なオプション：

|              パラメータ               |   フィールド名    |                                                              説明                                                              |
|----------------------------------|-------------|------------------------------------------------------------------------------------------------------------------------------|
| `vaults.description`<br> *オプション* | **説明**      | Vaultのオプションの説明です。                                                                                                            |
| `vaults.name`                    | **名前**      | Vaultのタイプ。`env`、 `gcp` 、 `aws`、または`hcv`のいずれかを受け入れます。環境変数vaultに`env`を設定します。                                                   |
| `vaults.prefix`                  | **プレフィックス** | 参照プレフィックス（接頭辞）。このボールトに保存されているシークレットにアクセスするには、この接頭辞が必要です。たとえば、 `{vault://my-env-vault/<some-secret id="sl-md0000000">}` などです。 |

