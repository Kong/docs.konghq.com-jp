---
title: "サポートされているVaultバックエンド"
---
次のVault実装がサポートされています。

|                                                                                           |    ティア     |
|-------------------------------------------------------------------------------------------|------------|
| [AWS Secrets Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/aws-sm/) | Enterprise |

{% if_version gte:3.4.x %}
| [Azure Key Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/azure-key-vaults/) | Enterprise |
{% endif_version %}

| [GCP Secret Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/gcp-sm/)      | Enterprise |
| [HashiCorp Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/hashicorp-vault/) | Enterprise |

無料ティアでは、シークレットは[環境変数](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/env/)に保存される場合があります。

