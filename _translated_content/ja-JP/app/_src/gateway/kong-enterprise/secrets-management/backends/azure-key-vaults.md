---
title: "Azure Key Vault"
badge: "enterprise"
content_type: "how-to"
---
{{site.base_gateway}}の実装の現在のバージョンでは、2つの方法で[Azure Key Vaults](https://azure.microsoft.com/en-us/products/key-vault)の構成をサポートしています。

* 環境変数
* マネージドID認証

{:.note}
> 
> {{site.base_gateway}} は `Secrets` Azure Vault タイプのみをサポートします。`Keys` Vault および `Certificates` Vault タイプはサポートされていません。

Azure Key Vaultの構成
------------------


{{site.base_gateway}} はキーを使用して自動的に[Azure Key Vault API](https://learn.microsoft.com/en-us/rest/api/keyvault/)を認証し、アクセス権をグラントします。

次の値を指定する必要があります。

* Azure ActiveDirectory テナント ID
* AzureクライアントID
* `vault_URI`
* Azureクライアントシークレット：この値は環境変数としてのみ構成できます。

{{site.base_gateway}}を起動する前に、環境変数を使用してこれらの値を構成できます。

```bash
export KONG_VAULT_AZURE_VAULT_URI=https://my-vault.vault.azure.com
export KONG_VAULT_AZURE_TENANT_ID=tenant_id
export KONG_VAULT_AZURE_CLIENT_ID=client_id
export AZURE_CLIENT_SECRET=client_secret
```

{:.note}
> 
> `Instance Managed Identity Token`では、環境変数を設定する必要はありません。

### 例：

`secret-name`という名前で[シークレット](https://learn.microsoft.com/en-us/azure/key-vault/general/about-keys-secrets-certificates)を使用するには、1 つ以上のプロパティを持つ JSON オブジェクトを Azure Key Vault に作成します。

```json
{
  "foo": "bar",
  "snip": "snap"
}
```

Azure AD テナント ID、クライアント ID、`vault_uri`、およびクライアントシークレットを指定する必要があります。
{{site.base_gateway}} を起動する前に、環境変数を使用してこれらの値を構成できます。

```bash
export KONG_VAULT_AZURE_VAULT_URI=https://my-vault.vault.azure.com
export KONG_VAULT_AZURE_TENANT_ID=tenant_id
export KONG_VAULT_AZURE_CLIENT_ID=client_id
export AZURE_CLIENT_SECRET=client_secret
```

```bash
{vault://azure/secret-name/foo}
{vault://azure/secret-name/snip}
```

あるいは、vaultsエンティティを介してvaultを構成することもできます。

vaultsエンティティによる構成
-----------------

データベースが初期化されると、vaultエンティティを作成し、
プロバイダーと必要な Azure Key Vault 情報をカプセル化します。

{% navtabs %}
{% navtab Admin API %}

```bash
curl -i -X PUT http://localhost:8001/vaults/azure-key-vault \
  --data name=azure \
  --data description="Storing secrets in Azure Key Vault (Secrets)" \
  --data config.type="secrets" \
  --data config.location="us-east" \
  --data config.vault_uri="http://my-vault-uri.azure.com"
```

結果：

```json
{
    "config": {
        "client_id": null,
        "credentials_prefix": "AZURE",
        "vault_uri": "http://my-vault-uri.azure.com",
        "location": "us-east",
        "neg_ttl": null,
        "resurrect_ttl": null,
        "tenant_id": null,
        "ttl": null,
        "type": "secrets",
        "vault_uri": null
    },
    "created_at": 1696235611,
    "description": "Storing secrets in Azure Key Vault (Secrets)",
    "id": "7c9287c1-2cbc-406b-a013-843fe54dc0b6",
    "name": "azure",
    "prefix": "azure-key-vault",
    "tags": null,
    "updated_at": 1696235611
}
```

{% endnavtab %}
{% navtab Declarative configuration %}

{:.note}
> 
> シークレット管理は、decK 1\.16以降でのみサポートされます。

次のスニペットを宣言型構成ファイルに追加します。

```yaml
_format_version: "3.0"
vaults:
- config:
    type: secrets
    vault_uri: http://my-vault-uri.azure.com
    location: us-east
  description: Storing secrets in Azure Key Vaults
  name: azure
  prefix: azure-key-vault
```

{% endnavtab %}
{% endnavtabs %}

Vaultエンティティを配置すると、それを通じてAzureシークレットを参照できます。

```bash
{vault://azure-key-vault/secret-name/foo}
{vault://azure-key-vault/secret-name/snip}
```

Vaultエンティティの設定オプション
-------------------

サポートツールのいずれかから、次の構成オプションを使用して Vault エンティティを構成します。

* Kong Manager
* {{site.konnect_short_name}}
* Admin API
* 宣言型構成

{{site.base_gateway}}にAzure Key Vaultのオプションを構成します。

|             パラメータ             |      フィールド名       |                                                                                                                                              説明                                                                                                                                               |
|-------------------------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `vaults.config.vault_uri`     | **Vault URI**     | Vaultに到達できるURI。この値は、 **Azure Key Vaultダッシュボード** のVault URIエントリの下にあります。                                                                                                                                                                                                                        |
| `vaults.config.client_id`     | **クライアント ID**     | 登録済みアプリケーションのクライアントID。Azure Dashboardにアクセスし、 **アプリ登録** を選択してクライアントIDを確認します。                                                                                                                                                                                                                   |
| `vaults.config.tenant_id`     | **テナントID**        | `DirectoryId` と `TenantId`はどちらも、ActiveDirectory テナントを表す GUID に相当します。つまり、" テナント ID " は " ディレクトリ ID " です。コンテキストに応じて、両方の用語が Microsoft のドキュメントおよび製品で使用される場合があります。                                                                                                                                 |
| `vaults.config.location`      | **Location**      | 各 Azure 地域には 1 つ以上のリージョンが含まれ、個別のデータ所在地とコンプライアンスの要件を満たしています。                                                                                                                                                                                                                                   |
| `vaults.config.type`          | **タイプ**           | Azure Key Vaultを使用すると、Microsoft Azureアプリケーションおよびユーザーは、キー、シークレット、証明書など、さまざまな種類のシークレットデータまたはキーデータを保存および使用できるようになります。Kongは現在、`secrets`のみをサポートしています。                                                                                                                                              |
| `vaults.config.ttl`           | **TTL**           | キャッシュされた場合の、vaultからのシークレットのTime\-to\-live（秒）。特別な値0は「ローテーションなし」を意味し、デフォルトです。ゼロ以外の値を使用する場合は、少なくとも1分単位にすることをお勧めします。                                                                                                                                                                            |
| `vaults.config.neg_ttl`       | **ネガティブTTL**      | Vaultのミス（シークレットが存在しない）のTime to Live（秒単位）。ネガティブキャッシュされたシークレットは、`neg_ttl`に達するまで有効です。その後、Kongはシークレットのリフレッシュを試行します。`neg_ttl`のデフォルト値は0で、ネガティブキャッシュは発生しないことを意味します。                                                                                                                                  |
| `vaults.config.resurrect_ttl` | **Resurrect TTL** | シークレットが期限切れになった（`config.ttl`が終了した）後もシークレットが使用され続ける時間（秒単位）。これは、Vaultにアクセスできなくなった場合や、シークレットがVaultから削除されてすぐに置き換えられない場合に役立ちます。どちらの場合も、ゲートウェイは`resurrect_ttl`秒間シークレットの更新を試行し続けます。その後、更新は停止します。Vaultに予期しない問題が発生した場合にシームレスな移行を確実に行うには、この構成オプションに高い値を設定します。`resurrect_ttl`のデフォルト値である1^e8秒は、約3年に相当します。 |

一般的なオプション：

|              パラメータ               |   フィールド名   |                                                            説明                                                            |
|----------------------------------|------------|--------------------------------------------------------------------------------------------------------------------------|
| `vaults.description`<br> *オプション* | **説明**     | Vault の説明（オプション）。                                                                                                        |
| `vaults.name`                    | **名前**     | Vaultのタイプ。`env`、 `gcp`、 `azure`、 `aws`、または`hcv`のいずれかを受け入れます。Azure Key Vaults に`azure`を設定します。                             |
| `vaults.prefix`                  | **Prefix** | 参照プレフィックス。このVaultに保存されているシークレットにアクセスするには、このプレフィックスが必要です。`{vault://gcp-sm-vault/<some-secret id="sl-md0000000">}`はその一例です。 |

