---
title: "FIPS 140-2"
badge: "enterprise"
content_type: "reference"
---
連邦情報処理標準（FIPS）140\-2 は、米国国立標準技術研究所によって定義された連邦標準です。暗号モジュールが満たす必要のあるセキュリティ要件を指定したものです。FIPS {{site.base_gateway}}パッケージは FIPS 140\-2に準拠しています。コンプライアンスとは、ソフトウェアが FIPS 140\-2 のすべての規則を満たしているが、検証のために NISTテストラボに提出されていないことを意味します。


{{site.ee_product_name}}は、 **Ubuntu 20\.04** {% if_version gte:3.1.x %}、 **Ubuntu 22\.04** {% if_version gte:3.4.x %}、 **Red Hat Enterprise 9** {% endif_version %}、 **Red Hat Enterprise 8** {% endif_version %}用の、FIPS 140\-2準拠のパッケージを提供します。このパッケージを使用すると、{{site.base_gateway}}のコア製品{% if_version gte:3.2.x %}とすぐに使用できるすべてのプラグイン{% endif_version %}はコンプライアンス要件を満たすことができます。

このパッケージは、OpenSSL FIPS 3\.0モジュールOpenSSLを使用して、FIPS 140\-2検証済みの暗号化操作を提供します。

{% if_version eq:3.0.x %}

{{site.base_gateway}} FIPS 準拠の Ubuntu パッケージのインストール
-----------------------------------

FIPS 準拠の Ubuntu 20\.04およびUbuntu 22\.04パッケージは、 `kong-enterprise-edition-fips`という名前のパッケージを使用してインストールできます。パッケージをインストールするには、次の手順に従います。

1. Kong APTリポジトリを設定します。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/gpg.{{ gpg_key }}.key" |  gpg --dearmor >> /usr/share/keyrings/kong-gateway-{{ page.major_minor_version }}-archive-keyring.gpg
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.deb.txt?distro=ubuntu&codename=$(lsb_release -sc)" > /etc/apt/sources.list.d/kong-gateway-{{ page.major_minor_version }}.list
   ```

2. リポジトリを更新します。

   ```bash
   sudo apt-get update
   ```

3. {{site.base_gateway}} FIPSパッケージをインストールします。

   ```sh
   apt install kong-enterprise-edition-fips
   ```

{% endif_version %}

{% if_version gte:3.1.x %}

{{site.base_gateway}} FIPS準拠パッケージのインストール
-------------------------

{% navtabs %}
{% navtab Ubuntu %}

FIPS 準拠の Ubuntu 20\.04 パッケージは、`kong-enterprise-edition-fips` という明確な名前のパッケージを使用してインストールできます。パッケージをインストールするには、次の手順に従います。

1. Kong APTリポジトリを設定します。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/gpg.{{ gpg_key }}.key" |  gpg --dearmor >> /usr/share/keyrings/kong-gateway-{{ page.major_minor_version }}-archive-keyring.gpg
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.deb.txt?distro=ubuntu&codename=$(lsb_release -sc)" > /etc/apt/sources.list.d/kong-gateway-{{ page.major_minor_version }}.list
   ```

2. リポジトリを更新します。

   ```bash
   sudo apt-get update
   ```

3. {{site.base_gateway}} FIPSパッケージをインストールします。

   ```sh
   apt install -y kong-enterprise-edition-fips={{page.versions.ee}}
   ```

{% endnavtab %}
{% navtab RHEL %}

FIPS準拠のRed Hat 8パッケージは、 `kong-enterprise-edition-fips`と明記されたパッケージを使用してインストールできます。パッケージをインストールするには、次の手順に従います。

{% navtabs %}
{% navtab Package %}

1. FIPSパッケージをダウンロードします。

   ```sh
   curl -Lo kong-enterprise-edition-fips-{{page.versions.ee}}.rpm $(rpm --eval {{ site.links.direct }}/gateway-{{ page.major_minor_version }}/rpm/el/%{rhel}/x86_64/kong-enterprise-edition-fips-{{page.versions.ee}}.el%{rhel}.x86_64.rpm)
   ```

2. {{site.base_gateway}} FIPSパッケージをインストールします。

   ```sh
   yum install kong-enterprise-edition-fips-{{page.versions.ee}}
   ```

{% endnavtab %}
{% navtab Yum repo %}

1. Kong Yumリポジトリを設定します。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.rpm.txt?distro=el&codename=$(rpm --eval '%{rhel}')" | sudo tee /etc/yum.repos.d/kong-gateway-{{ page.major_minor_version }}.repo
   sudo yum -q makecache -y --disablerepo='*' --enablerepo='kong-gateway-{{ page.major_minor_version }}'
   ```

2. {{site.base_gateway}} FIPSパッケージをインストールします。

   ```sh
   yum install kong-enterprise-edition-fips-{{page.versions.ee}}
   ```

{% endnavtab %}
{% endnavtabs %}

{% endnavtab %}
{% endnavtabs %}
{% endif_version %}

FIPSの構成
-------

FIPSモードを開始するには、{{site.base_gateway}}を開始する前に、`kong.conf`構成ファイルで以下の変数を`on`に設定します。

    fips = on # fips mode is enabled, causing incompatible ciphers to be disabled

以下のように、環境変数を1つ使用することもできます。

```bash
export KONG_FIPS=on
```

{:.important .no-icon}
> 
> 非 FIPSモードからFIPSモードへの移行、およびその逆の移行はサポートされていません。

