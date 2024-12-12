---
title: "シークレットローテーション"
---
シークレットのローテーションとはシークレットを更新するプロセスです。シークレットを定期的にローテーションするのは好ましい慣行と考えられています。
シークレットローテーションが重要な理由を以下にいくつか挙げます。

* 漏洩したシークレットの影響を軽減する
* ブルートフォース攻撃に対するレジリエンスの強化
* セキュリティ規制の順守
* 職務の分離の維持
* 進化する脅威への適応
* 内部脅威の影響を軽減する

Kongはリクエストを処理するためおよび他のシステムとクライアントと統合するために、多種のシークレットを保持する必要があります。

* TLS証明書
* 認証情報とAPIキー（データベース、IDプロバイダー、その他のサービスへのアクセスに使用）
* 暗号鍵（デジタル署名と暗号化）

シークレットをローテーションする方法は主に2つあります。

* TTLを使用して定期的にローテーションします（例：1日に1回新規TLS証明書を確認）
* 失敗時にローテーションします（たとえば、データベース認証に失敗した場合、シークレットが更新されているかどうかを確認して、もう一度やり直してください）

Kong は、シークレットのローテーションについて両方の方法をサポートしています。失敗時のローテーションにはコードを書く必要があるため、その点において Kong でのサポート（現時点で Postgres の認証情報）は限られています。失敗時にシークレットをローテーションに使用できる実験的な Kong PDK API（[kong.vault.try](/gateway/{{page.release}}/plugin-development/pdk/kong.vault/#kongvaulttrycallback-options)）が用意されています。

TTL を使用したシークレットの定期的なローテーション
---------------------------

{% if_version lte: 3.3.x %}
Kong はバックグラウンドで *1 分ごとに* 自動的にシークレットをローテーションします。有効期限が近づいているシークレットをローテーションします。TTLは、Vault（すべてのシークレットの場合）またはシークレット参照（単一のシークレットの場合）で設定できます。デフォルトでは、Kongはシークレットをローテーションしないため、自動ローテーションを有効にする場合は必ずTTLを構成してください。
{% endif_version %}
{% if_version gte: 3.4.x %}
Kong はバックグラウンドで *1 分ごとに* 自動的にシークレットをローテーションします。これにより、シークレットのローテーションプロセスがプロキシから切り離されます。これにより、次のような結果になります。

* 1分間に1回の更新レートというのは、TTLが30秒のシークレットが、最も不利なシナリオでは更新に最大60秒かかる可能性があることを意味します。
* さらに、特定のシークレットのTTLと`resurrect_ttl`の合計が60秒未満の場合、シークレットは正しく更新または復活されません。 {% endif_version %}

TTLを使用したローテーションは、以下を含むKongがサポートするほとんどのVaultで機能します。

* [AWS Secrets Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/aws-sm/)
* [GCP Secret Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/gcp-sm/)
* [HashiCorp Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/hashicorp-vault/)

TTLを使用してローテーションする場合、通常は同じシークレットの2つのバージョンを同時に有効にしておくと便利です。
つまり、シークレットのローテーション中に次の手順が発生します。

1. 新しいシークレット \(またはシークレットバージョン\) が作成されると、有効なシークレットが3つになります。
2. 3 つのシークレットすべてが機能することが確認されています。
3. 最も古いシークレット（またはシークレットバージョン）が削除/取り消されるか無効になり、2つの有効なシークレットが残ります。

### TTLを使用してAWS Secrets Managerシークレットのローテーションを構成する

デフォルトの [AWS Secrets Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/aws-sm/)
vault TTLは `kong.conf` または環境変数（値は秒単位）を使用して設定できます。

```bash
KONG_VAULT_AWS_TTL=300
KONG_VAULT_AWS_NEG_TTL=60
KONG_VAULT_AWS_RESURRECT_TTL=300
```

すべてのAWSシークレットの参照は、デフォルトでこれらの設定を継承します。例：

```bash
{vault://aws/my-secret-name/foo}
```

参照を使用してTTLを直接上書きまたは設定することもできます。

```bash
{vault://aws/my-secret-name/foo?ttl=600&neg_ttl=30&resurrect_ttl=600}
```

また、さまざまな種類のシークレット用に複数のvaultを作成することもできます。
そして次の例のように、シークレットタイプ別にTTLを設定します。

```bash
curl -i -X PUT http://HOSTNAME:8001/vaults/aws-certs  \
  --data name=aws \
  --data description="Storing secrets in AWS Secrets Manager" \
  --data config.region="us-east-1" \
  --data config.ttl=21600
```

証明書の使用時に以下を参照できるようになりました。

```bash
{vault://aws-certs/certs/web-site}
```

`aws-certs` vaultで参照されるシークレット （この場合は証明書）は、同じ6時間のTTLを共有し、有効期限が切れる1分前にローテーションします。

### TTLを使用してGCP Secret Managerシークレットのローテーションを構成する

デフォルトの[GCP Secret Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/gcp-sm/)
vault TTLは、`kong.conf`または環境変数（値は秒単位）を使用して構成できます。

```bash
KONG_VAULT_GCP_TTL=300
KONG_VAULT_GCP_NEG_TTL=60
KONG_VAULT_GCP_RESURRECT_TTL=300
```

すべての GCP シークレット参照は、デフォルトでこれらの設定を継承します。例：

```bash
{vault://gcp/my-secret-name/foo}
```

参照を使用してTTLを直接上書きまたは設定することもできます。

```bash
{vault://gcp/my-secret-name/foo?ttl=600&neg_ttl=30&resurrect_ttl=600}
```

また、さまざまな種類のシークレット用に複数のvaultを作成することもできます。
そして次の例のように、シークレットタイプ別にTTLを設定します。

```bash
curl -i -X PUT http://HOSTNAME:8001/vaults/gcp-certs  \
  --data name=gcp \
  --data description="Storing secrets in GCP Secret Manager" \
  --data config.project_id="my_project_id-1" \
  --data config.ttl=21600
```

証明書の使用時に以下を参照できるようになりました。

```bash
{vault://gcp-certs/certs/web-site}
```

`gcp-certs` vaultで参照されるシークレット （この場合は証明書）は、同じ6時間のTTLを共有し、有効期限が切れる1分前にローテーションします。

### TTLを使用したHashiCorp Vaultシークレットのローテーションの設定

デフォルトの[HashiCorp Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/hashicorp-vault/)
TTL は`kong.conf`または環境変数を通じて設定できます \(値は秒単位です\)。

```bash
KONG_VAULT_HCV_TTL=300
KONG_VAULT_HCV_NEG_TTL=60
KONG_VAULT_HCV_RESURRECT_TTL=300
```

すべてのHCVシークレットの参照は、デフォルトでこれらの設定を継承します。例：

```bash
{vault://hcv/my-secret-name/foo}
```

参照を使用してTTLを直接上書きまたは設定することもできます。

```bash
{vault://hcv/my-secret-name/foo?ttl=600&neg_ttl=30&resurrect_ttl=600}
```

また、さまざまな種類のシークレット用に複数のvaultを作成することもできます。
そして次の例のように、シークレットタイプ別にTTLを設定します。

```bash
curl -i -X PUT http://HOSTNAME:8001/vaults/hcv-certs  \
  --data name=hcv \
  --data description="Storing secrets in HashiCorp Vault" \
  --data config.token="<my-token id="sl-md0000000">" \
  --data config.ttl=21600
```

これで、次の方法で証明書を参照できます。

```bash
{vault://hcv-certs/certs/web-site}
```

`hcv-certs` vaultで参照されるシークレット （この場合は証明書）は、同じ6時間のTTLを共有し、有効期限が切れる1分前にローテーションします。

