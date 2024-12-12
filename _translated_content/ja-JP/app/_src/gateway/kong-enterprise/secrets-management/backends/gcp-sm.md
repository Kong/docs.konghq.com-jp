---
title: "GCP Secret Manager"
badge: "enterprise"
content-type: "how-to"
---
{{site.base_gateway}}の実装の現在のバージョンでは、2つの方法で[GCP Secret Manager](https://cloud.google.com/secret-manager/)の構成をサポートしています。

* 環境変数
* ワークロードID

GCP Secret Managerを構成する
-----------------------

GCP Secret Managerを構成するには、`GCP_SERVICE_ACCOUNT`環境変数を[サービスアカウントの認証情報](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)を参照するJSONドキュメントに設定する必要があります。

```bash
export GCP_SERVICE_ACCOUNT=$(cat gcp-project-c61f2411f321.json)
```


{{site.base_gateway}}はキーを使用して自動的にGCP APIを認証し、アクセス権を付与します。

GKE クラスタで、GCP Secret Manager を [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) と一緒に使用するには、ポッドの仕様を更新して、サービスアカウントをポッドに関連付けます。構成情報については、[Workload Identity の構成に関するドキュメント](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity#authenticating_to)を参照してください。

{:.note}
> 
> **注：** 
> 
> * Workload Identity では、 `GCP_SERVICE_ACCOUNT`を設定する必要はありません。
> * GCP Vault をバックエンドとして使用する場合は、公式の GCP API が使用する SSL 証明書を Kong が信頼できるように、[`lua_ssl_trusted_certificate`](/gateway/{{page.release}}/reference/configuration/#lua_ssl_trusted_certificate) 構成ディレクティブの一部として `system` が構成されていることを確認してください。

### 例:

`secret-name`という名前でGCP Secret Managerの[シークレット](https://cloud.google.com/secret-manager/docs/reference/rest/v1/projects.secrets)を使用するには、以下のように1つ以上のプロパティを含むJSONオブジェクトをGCPで作成します。

```json
{
  "foo": "bar",
  "snip": "snap"
}
```

以下のように、シークレットの個々のリソースを参照できるようになりました。

```bash
{vault://gcp/secret-name/foo?project_id=project_id}
{vault://gcp/secret-name/snip?project_id=project_id}
```

プロバイダー（`gcp`）とGCPプロジェクトID（`project_id`）の両方を指定する必要があります。{{site.base_gateway}}を起動する前に、環境変数を使用してプロジェクトIDを構成できます。

```bash
export KONG_VAULT_GCP_PROJECT_ID=project_id
```

その後、参照でそれを繰り返す必要はありません。

```bash
{vault://gcp/secret-name/foo}
{vault://gcp/secret-name/snip}
```

Vaults エンティティによる構成
------------------

データベースが初期化されると、プロバイダーとGCPプロジェクトIDをカプセル化するVaultエンティティを作成できます。

{% navtabs %}
{% navtab Admin API %}

```bash
curl -i -X PUT http://HOSTNAME:8001/vaults/gcp-sm-vault \
  --data name=gcp \
  --data description="Storing secrets in GCP Secret Manager" \
  --data config.project_id="project_id"
```

結果

```json
{
    "config": {
        "project_id": "project_id"
    },
    "created_at": 1657874961,
    "description": "Storing secrets in GCP Secret Manager",
    "id": "90e200be-cf84-4ce9-a1d6-a41c75c79f31",
    "name": "gcp",
    "prefix": "gcp-sm-vault",
    "tags": null,
    "updated_at": 1657874961
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
    project_id: project_id
  description: Storing secrets in GCP Secret Manager
  name: gcp
  prefix: gcp-sm-vault
```

{% endnavtab %}
{% endnavtabs %}

Vaultエンティティを配置すると、それを通じてGCPシークレットを参照できます。

```bash
{vault://gcp-sm-vault/secret-name/foo}
{vault://gcp-sm-vault/secret-name/snip}
```

Vaultエンティティの設定オプション
-------------------

サポートツールのいずれかから、次の構成オプションを使用して Vault エンティティを構成します。

* Admin API
* 宣言型構成
{% if_version gte:3.1.x -%}
* Kong Manager
* {{site.konnect_short_name}} {% endif_version %}

{{site.base_gateway}} の GCP Secret Manager vault の構成オプション:

|             パラメータ             |        フィールド名         |                                                                                                                                                        説明                                                                                                                                                         |
|-------------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `vaults.config.project_id`    | **Google Project ID** | Google API ConsoleのプロジェクトID。Google API Consoleにアクセスし、プロジェクトリストで **すべてのプロジェクトを管理** を選択し、プロジェクトIDを確認します。                                                                                                                                                                                                            |
| `vaults.config.ttl`           | **TTL**               | キャッシュされた場合の、vaultからのシークレットのTime to Live（秒）。特別な値0は「ローテーションなし」を意味し、これはデフォルトです。ゼロ以外の値を使用する場合は、少なくとも1分にすることをお勧めします。                                                                                                                                                                                                        |
| `vaults.config.neg_ttl`       | **ネガティブ TTL**         | Vault のミス（シークレットが存在しない）の Time to Live（秒単位）。ネガティブキャッシュされたシークレットは、`neg_ttl`に達するまで有効です。その後、Kong はシークレットのリフレッシュを試行します。`neg_ttl` のデフォルト値は 0 で、ネガティブキャッシュは発生しないことを意味します。                                                                                                                                                |
| `vaults.config.resurrect_ttl` | **Resurrect TTL**     | シークレットが期限切れになった（`config.ttl` が終了した）後もシークレットが使用され続ける時間（秒単位）。これは、Vaultにアクセスできなくなった場合、またはシークレットがVaultから削除されてすぐに置き換えられない場合に役立ちます。どちらの場合も、ゲートウェイは`resurrect_ttl`秒間シークレットの更新を試行し続けます。その後、更新の試みは停止します。Vault に予期しない問題が発生した場合にシームレスな移行を確実に行うために、この構成オプションに十分に高い値を割り当てることをお勧めします。`resurrect_ttl`のデフォルト値は1e8秒で、これは約3年に相当します。 |

一般的なオプション:

|               パラメータ               |   フィールド名   |                                                              説明                                                              |
|-----------------------------------|------------|------------------------------------------------------------------------------------------------------------------------------|
| `vaults.description` <br> *オプション* | **説明**     | Vaultの説明（オプション）。                                                                                                             |
| `vaults.name`                     | **名前**     | Vaultのタイプ。`env`、`gcp`、`aws`、または`hcv`のいずれかを受け入れます。GCP Secret Managerに`gcp`を設定します。                                             |
| `vaults.prefix`                   | **Prefix** | 参照プレフィックス（接頭辞）。このボールトに保存されているシークレットにアクセスするには、この接頭辞が必要です。たとえば、 `{vault://gcp-sm-vault/<some-secret id="sl-md0000000">}` などです。 |

