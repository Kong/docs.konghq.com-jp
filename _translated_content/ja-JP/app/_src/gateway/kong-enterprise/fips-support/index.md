---
title: "Kong GatewayのFIPS 140-2準拠について"
badge: "enterprise"
content_type: "reference"
---
連邦情報処理標準（FIPS）140\-2は、米国国立標準技術研究所によって定義された連邦標準です。暗号モジュールが満たす必要のあるセキュリティ要件を指定したものです。FIPS {{site.base_gateway}}パッケージはFIPS 140\-2に準拠しています。コンプライアンスとは、FIPSモード実行中、{{site.base_gateway}}がFIPS 140\-2承認アルゴリズムのみ使用するが、検証のために製品はNISTテストラボに提出されていないことを意味します。


{{site.ee_product_name}}は、 **Ubuntu 20\.04** {% if_version gte:3.1.x %}、 **Ubuntu 22\.04** {% if_version gte:3.4.x %}、 **Red Hat Enterprise 9** {% endif_version %}、 **Red Hat Enterprise 8** {% endif_version %}用の、FIPS 140\-2準拠のパッケージを提供します。このパッケージは、コア{{site.base_gateway}}製品{% if_version gte:3.2.x %}とすべてのすぐに使用できるプラグイン{% endif_version %}のコンプライアンスを提供します。

このパッケージは、OpenSSL FIPS 3\.0モジュールOpenSSLを使用して、FIPS 140\-2検証済みの暗号化操作を提供します。

{:.note}
> 
> **注意** ：{{site.ee_product_name}}フリーモードで実行している場合、FIPSはサポートされません。

FIPS実装
------

### パスワードのハッシュ化

次の表は、 {{site.base_gateway}}キー導出関数をどのように使用するかを示しています。

|                コンポーネント                |                 通常モード                  |           FIPS モード            |              備考               |
|---------------------------------------|----------------------------------------|-------------------------------|-------------------------------|
| core/rbac                             | bcrypt                                 | <sup>PBKDF2 1</sup>           | OpenSSL 3\.0 FIPSプロバイダーによる準拠 |
| plugins/oauth2 <sup>2</sup>           | Argon2またはbcrypt（`hash_secret=true`の場合） | 無効（`hash_secret`を`true`に設定不可） | OpenSSL 3\.0 FIPSプロバイダーによる準拠 |
| plugins/key\-auth\-enc <sup>3</sup> | SHA1                                   | SHA256                        | SHA1はFIPSモードでは読み取り専用です。       |

{:.note .no-icon}
> 
> **\[1\]** ：{{site.base_gateway}} FIPS 3\.0以降、RBACはパスワード ハッシュ アルゴリズムとしてPBKDF2を使用します。<br><br>
> **\[2\]** ：{{site.base_gateway}} FIPS 3\.1以降、oauth2プラグインは`hash_secret`機能を無効にしているため、ユーザーはこの機能を利用できません。これは、パスワードがデータベースにプレーンテキストで保存されることを意味します。しかし、ユーザーは代わりにシークレット管理やDB暗号化を使用することができます。
> <br><br>
> **\[3\]** ：{{site.base_gateway}} FIPS 3\.1以降、key\-auth\-encはSHA1を使用してDB内のキーの検索を高速化します。{{site.base_gateway}} FIPS 3\.2では、SHA1のサポートは「読み取り専用」です。つまり、DB 内の既存の認証情報は引き続き検証されますが、新しい認証情報はSHA256でハッシュされます。

{:.important}
> 
> **重要** : FIPS モードでの {{site.base_gateway}} 3\.1 から 3\.2 への移行で、key\-auth\-enc プラグインを使用している場合は、既存のすべての key\-auth\-enc 認証情報に [PATCH または POST リクエスト](/hub/kong-inc/key-auth-enc/#create-a-key) を送信して、SHA256 で再ハッシュする必要があります。

### 暗号化アルゴリズムの非暗号化使用

FIPSは、特定の目的ごとに使用する承認済みアルゴリズムを定義しているだけなので、FIPSポリシーは、暗号アルゴリズムの使用を必要な場合のみに明示的にに制限しません。

たとえば、SHA256をメッセージダイジェストアルゴリズムとして使用することは承認されますが、MD5の使用は承認されません。しかし、MD5がアプリケーションにまったく存在できないという意味ではありません。たとえば、FIPS 140\-2承認の[BoringSSL](https://csrc.nist.gov/CSRC/media/projects/cryptographic-module-validation-program/documents/security-policies/140sp3678.pdf)バージョンでは、TLSプロトコルバージョン1\.0および1\.1と併用するとMD5を使用できます。

以下の表では、{{site.base_gateway}}で暗号化以外の目的で暗号化アルゴリズムが使用される場所を説明しています。

|                           コンポーネント                            |        通常モード        |      FIPS モード       |             備考             |
|--------------------------------------------------------------|---------------------|---------------------|----------------------------|
| コア/バランサー                                                     | xxhash32            | xxhash32            | 一意の識別子を生成するために使用します。       |
| コア/バランサー                                                     | crc32               | crc32               | crc32 はメッセージダイジェストではありません。 |
| core/uuid                                                    | Lua 乱数発生器           | Lua 乱数発生器           | RNGは暗号化目的で使用されません。         |
| core/declarative\_config/uuid                               | UUIDv5（名前空間付き SHA1） | UUIDv5（名前空間付き SHA1） | 一意の識別子を生成するために使用されます。      |
| core/declarative\_config/config\_hash と core/hybrid/hashes | MD5                 | MD5                 | 一意の識別子を生成するために使用されます。      |

{% if_version gte:3.5.x %}
| core/kong\_request\_id | rand\(3\) | rand\(3\) | RNGは暗号化目的で使用されません。|
{% endif_version %}

### SSLクライアント

FIPS 140\-2 では SSL サーバーのみが言及されていますが、これは {{site.base_gateway}} FIPS 3\.0 ですでにサポートされています。FIPS 仕様は SSL クライアントを対象としていないため、 {{site.base_gateway}} ではこれらに対する特別な処理は行われません。

これには以下が含まれます。

* Lua を使用して HTTPS と PostgreSQL SSL で通信する
{% if_version lte:3.3.x -%}
* Luaを使用した、HTTPS、PostgreSQL SSL、Cassandra SSLでの通信
{% endif_version -%}
* HTTPSでプロキシするアップストリームの使用

