---
title: "FIPS 140-2準拠のプラグイン"
badge: "enterprise"
content_type: "reference"
---
このリファレンスには、FIPS 140\-2に準拠している{{site.base_gateway}}プラグインがリストされ、準拠を維持する方法に関する追加の詳細が提供されます。

|     プラグイン     | サブコンポーネントのコンプライアンス（該当する場合） | FIPS準拠 |               備考               |
|---------------|----------------------------|--------|--------------------------------|
| jwe\-decrypt | 対象外                        | はい     | OpenSSL 3\.0 FIPS プロバイダー経由で準拠 |

{% if_version gte: 3.2.x %}
| openid\-connect | すべて | はい | OpenSSL 3\.0 FIPSプロバイダー経由で対応 |
| jwt\-signer | すべて | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| key\-auth\-enc | 該当なし | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
{% endif_version %}

| hmac\-auth | 非該当 | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| ldap\-auth\-advanced | 非該当 | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| proxy\-cache | 非該当 | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| proxy\-cache\-advanced | 非該当 | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| graphql\-proxy\-cache\-advanced | 非該当 | Yes | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| mtls\-auth | 非該当 | Yes | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| oauth2 | 非該当 | Yes | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| basic\-auth | 非該当 | Yes | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| saml | 非該当 | はい | OpenSSL 3\.0 FIPSプロバイダー経由で準拠 |
| jwt | 非該当 | はい | OpenSSL 3\.0 FIPS プロバイダー経由で準拠 |
| Kong Inc.の他のすべてのプラグイン | 非該当 | 非該当 | 暗号化操作は不要 |

