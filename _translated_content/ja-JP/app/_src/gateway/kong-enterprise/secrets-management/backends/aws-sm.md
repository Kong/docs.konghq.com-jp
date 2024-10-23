---
title: "AWS Secrets Manager"
badge: "enterprise"
---
[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)は複数の方法で設定できます。

{% if_version gte:3.4.x %}
AWS Secrets Managerに保存されているシークレットにアクセスするには、必要なシークレット値を読み取るための十分な権限を持つIAMロールを使用して{{site.base_gateway}}を構成する必要があります。


{{site.base_gateway}}は、次の優先順位に従って、AWS 環境に基づいてIAM ロールの認証情報を自動的に取得できます。

* 環境変数 `AWS_ACCESS_KEY_ID` および `AWS_SECRET_ACCESS_KEY` で定義された認証情報から取得します。
* `AWS_PROFILE`と`AWS_SHARED_CREDENTIALS_FILE`で定義されたプロファイルと認証情報ファイルから取得します。
* ECS[コンテナ認証情報プロバイダー](https://docs.aws.amazon.com/sdkref/latest/guide/feature-container-credentials.html)から取得します。
* [サービスアカウントのEKS IAMロール](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)から取得します。
* EC2 IMDSメタデータから取得します。v1とv2の両方がサポートされています


{{site.base_gateway}}ロールの引き継ぎもサポートしており、これにより、別の IAM ロールを使用して AWS Secrets Manager からシークレットを取得できるようになります。これは、権限の分割とガバナンス、および AWS アカウント間の管理において一般的な方法です。
{% endif_version %}

Vaultオブジェクトをカスタマイズするには、`vaults`
エンティティを{{site.base_gateway}}で構成できます。

AWS Secrets Manager の構成
-----------------------

AWS Secrets ManagerでVaultバックエンドを構成するには、関連するシークレットにアクセスするために必要な IAM ロールを持つように {{site.base_gateway}}を構成する必要があります。

たとえば、AWS環境変数の認証情報でシークレット管理を使用するには、{{site.base_gateway}}データプレーン（DP）で次の環境変数を構成します。

```bash
export AWS_ACCESS_KEY_ID=<access_key_id id="sl-md0000000">
export AWS_SECRET_ACCESS_KEY=<secrets_access_key id="sl-md0000000">
export AWS_SESSION_TOKEN=<token id="sl-md0000000">
export AWS_REGION=<aws-region id="sl-md0000000">
```

デフォルトで参照付きで使用されるリージョンは、コントロールプレーン（CP）で次の環境変数で指定することもできます。

```bash
export KONG_VAULT_AWS_REGION=<aws-region id="sl-md0000000">
```

{% if_version gte:3.4.x %}
さらに、[`assume_role`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) を使用する場合は、{{site.base_gateway}} データプレーン（DP）に次の環境変数があることを確認してください。

```bash
export KONG_VAULT_AWS_ASSUME_ROLE_ARN=<aws_iam_role_arn id="sl-md0000000">
export KONG_VAULT_AWS_ROLE_SESSION_NAME=<aws_assume_role_session_name id="sl-md0000000">
```

Vaultバックエンド構成フィールドは、`kong.conf`ファイルでも構成できます。[Gateway Enterprise構成リファレンス](/gateway/latest/reference/configuration)を参照してください。{% endif_version %}

{:.note}
> 
> **注：** AWS Vaultをバックエンドとして使用する場合、公式のAWS APIが使用するSSL証明書をKongが信頼できるように、`system`を[`lua_ssl_trusted_certificate`構成ディレクティブ](/gateway/{{page.release}}/reference/configuration/#lua_ssl_trusted_certificate)の一部として構成していることを確実にしてください。

### 例

たとえば、 `secret-name`という名前の AWS Secrets Managerシークレットには、複数のキーと値のペアが含まれる場合があります。

```json
{
  "foo": "bar",
  "snip": "snap"
}
```

`secret-name`から上記のシークレットに次のようにアクセスします。

```bash
{vault://aws/secret-name/foo}
{vault://aws/secret-name/snip}
```

複数のバージョンを持つ AWS Secrets Manager シークレットがある場合は、参照でバージョンを指定することで、シークレットの現在のバージョンまたは以前のバージョンにアクセスできます。

次の例では、 `AWSCURRENT` は最新のシークレットバージョンを参照し、 `AWSPREVIOUS` は古いバージョンを参照します。

```bash
# For AWSCURRENT, not specifying version
{vault://aws/secret-name/foo}

# For AWSCURRENT, specifying version == 1
{vault://aws/secret-name/foo#1}

# For AWSPREVIOUS, specifying version == 2
{vault://aws/secret-name/foo#2}
```

{:.note}
> 
> **注意:** スラッシュ記号 \( `/` \) は、AWS SecretsManager のシークレット名に有効な文字です。 スラッシュで始まる、またはスラッシュが2つ続くシークレット名を参照する場合は、名前のスラッシュの1つをURLエンコード形式に変換してください。 例えば

* `/secret/key`という名前のシークレットは次のように参照されます。`{vault://aws/%2Fsecret/key}`
* `secret/path//aaa/key`という名前のシークレットは次のように参照されます。`{vault://aws/secret/path/%2Faaa/key}`
> 
> {{site.base_gateway}} はシークレットリファレンスを有効なURLとして解決しようとするので、URLでエンコードされたスラッシュの代わりにスラッシュを使用すると、予期しないシークレット名が取得されてしまいます。

vaultsエンティティによる構成
-----------------

vaultエンティティは、データベースが初期化されて初めて使用できます。データベースが初期化される *前* に使用された値のシークレットは、vaultsエンティティを使用できません。

{% navtabs %}
{% navtab Admin API %}

```bash
curl -i -X PUT http://localhost:8001/vaults/aws-sm-vault  \
  --data name=aws \
  --data description="Storing secrets in AWS Secrets Manager" \
  --data config.region="us-east-1"
```

結果：

```json
{
    "config": {
        "region": "us-east-1"
    },
    "created_at": 1644942689,
    "description": "Storing secrets in AWS Secrets Manager",
    "id": "2911e119-ee1f-42af-a114-67061c3831e5",
    "name": "aws",
    "prefix": "aws-sm-vault",
    "tags": null,
    "updated_at": 1644942689
}
```

{% endnavtab %}
{% navtab Declarative configuration %}

{:.note}
> 
> シークレット管理のサポートは、decK 1\.16で追加されました。

次のスニペットを宣言型設定ファイルに追加します。

```yaml
_format_version: "3.0"
vaults:
- config:
    region: us-east-1
  description: Storing secrets in AWS Secrets Manager
  name: aws
  prefix: aws-sm-vault
```

{% endnavtab %}
{% endnavtabs %}

Vaultエンティティが配置されると、シークレットを参照できるようになります。これにより、`AWS_REGION`環境変数を削除できます。

```bash
{vault://aws-sm-vault/secret-name/foo}
{vault://aws-sm-vault/secret-name/snip}
```

各リージョンのシークレット
-------------

複数のエンティティを作成して、異なるリージョンにシークレットを保持することができます。

```bash
curl -X PUT http://localhost:8001/vaults/aws-eu-central-vault -d name=aws -d config.region="eu-central-1"
```

```bash
curl -X PUT http://localhost:8001/vaults/aws-us-west-vault -d name=aws -d config.region="us-west-1"
```

これにより、さまざまなリージョンからシークレットを取得できます。

```bash
{vault://aws-eu-central-vault/secret-name/foo}
{vault://aws-us-west-vault/secret-name/snip}
```

Vault構成オプション
------------

サポートツールのいずれかから、次の構成オプションを使用して`vaults`エンティティを構成します。

* Admin API
* 宣言型構成
{% if_version gte:3.1.x -%}
* Kong Manager
* {{site.konnect_short_name}} {% endif_version %}

{{site.base_gateway}}のAWS Secrets Manager vaultの構成オプション：

|         パラメータ          |    フィールド名    |           説明           |
|------------------------|--------------|------------------------|
| `vaults.config.region` | **AWSリージョン** | Vaultが配置されているAWSリージョン。 |

{% if_version gte:3.4.x -%}
`vaults.config.endpoint_url` \| **AWS Secrets ManagerのエンドポイントURL** \| AWS Secrets ManagerサービスのエンドポイントURL。 指定されていない場合、vaultで使用される値は公式の AWS Secrets Managerサービス URL（ `https://secretsmanager.{region}.amazonaws.com` ）になります。完全な URL（ `http/https` スキームを含む）を指定して、エンドポイントをオーバーライドできます。`vaults.config.assume_role_arn` \| **Assume AWS IAM role ARN** \| AWS Secrets Managerサービスとして引き受けるターゲットIAMロールARN。指定した場合、Vaultバックエンドは、現在のランタイムのIAMロールに基づいて追加のロールを実行します。ロールを引き受ける機能を使用していない場合は、この値を指定しないでください。`vaults.config.role_session_name` \| **Assume AWS IAM role ARN** \| ロールを引き受けるために使用されるロールSession名。デフォルト値は`KongVault`です。{% endif_version -%}

{% if_version gte:3.8.x -%}
`vaults.config.sts_endpoint_url` | **AWS STS エンドポイントURL** | AWS VaultでIAMに使用されるカスタムSTSエンドポイントURL。エンドポイントを上書きするために、完全な URL (`http/https` スキームを含む) を指定することができます。`https://sts.amazonaws.com` または `https://sts.amazonaws.com` に設定されている `AWS_STS_REGIONAL_ENDPOINTS` が `regional` に設定されている場合、 `https://sts.amazonaws.com` または`https://sts.amazonaws.com.<0>.amazonaws.com` に設定されている `regional`。
{% endif_version -%}

`vaults.config.ttl` \| **TTL** \| キャッシュされた場合の、vaultからのシークレットのTime\-to\-live（秒単位）。特別な値0は「回転なし」を意味し、これはデフォルトです。ゼロ以外の値を使用する場合は、少なくとも1分にすることをおすすめします。`vaults.config.neg_ttl` \| **ネガティブTTL** \| vaultミス（シークレットなし）のTime\-to\-live（秒単位）。ネガティブキャッシュされたシークレットは、`neg_ttl`に達するまで有効です。その後、Kongはシークレットの更新を試行します。`neg_ttl`のデフォルト値は0で、ネガティブキャッシュは発生しないことを意味します。`vaults.config.resurrect_ttl` \| **復活TTL** \| 期限切れになった後にシークレットが使用状態になり続ける時間（秒単位）（`config.ttl`を停止点として使用）。これは、vaultにアクセスできなくなった場合や、シークレットがVaultから削除されてすぐに置き換えられない場合に役立ちます。どちらの場合も、ゲートウェイは`resurrect_ttl`秒間シークレットの更新を試み続けます。その後、更新の試みは停止します。vaultに予期しない問題が発生した場合でもシームレスな移行が行われるように、この構成オプションに十分に高い値を割り当てることをおすすめします。`resurrect_ttl`のデフォルト値は1^e8秒で、これは約3年に相当します。

一般的なオプション：

|               パラメータ               | フィールド名  |                                                             説明                                                              |
|-----------------------------------|---------|-----------------------------------------------------------------------------------------------------------------------------|
| `vaults.description` <br> *オプション* | **説明**  | Vaultの説明（オプション）。                                                                                                            |
| `vaults.name`                     | **名前**  | Vaultのタイプ。`env`、`gcp`、`aws`、または`hcv`のいずれかを受け入れます。AWS Secrets Manager に`aws`を設定します。                                          |
| `vaults.prefix`                   | **接頭辞** | 参照プレフィックス（接頭辞）。このVaultに保存されているシークレットにアクセスするには、この接頭辞が必要です。たとえば、`{vault://aws-sm-vault/<some-secret id="sl-md0000000">}`などです。 |

