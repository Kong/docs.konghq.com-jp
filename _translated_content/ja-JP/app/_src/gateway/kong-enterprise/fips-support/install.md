---
title: "FIPS準拠パッケージのインストールと構成"
badge: "enterprise"
content_type: "how-to"
---
このハウツーガイドでは、{{site.base_gateway}} FIPS準拠のパッケージのインストールと構成方法について説明します。このガイドの手順に従うと、FIPSモードが有効になったFIPS準拠の{{site.base_gateway}}が作成されます。

{{site.base_gateway}} FIPS準拠パッケージのインストール
------------------------

{% navtabs %}
{% navtab Ubuntu %}

FIPS 準拠の Ubuntu 20\.04 パッケージは、`kong-enterprise-edition-fips` という明確な名前のパッケージを使用してインストールできます。パッケージをインストールするには、次の手順に従います。

1. Kong APTリポジトリを設定します。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/gpg.{{ gpg_key }}.key" |  gpg --dearmor >> /usr/share/keyrings/kong-gateway-{{ page.major_minor_version }}-archive-keyring.gpg
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.deb.txt?distro=ubuntu&codename=$(lsb_release -sc)" > /etc/apt/sources.list.d/kong-gateway-{{ page.major_minor_version }}.list
   ```

2. リポジトリを次のとおり更新します。

   ```bash
   sudo apt-get update
   ```

3. {{site.base_gateway}} FIPSパッケージをインストールします。

   ```sh
   apt install -y kong-enterprise-edition-fips={{page.versions.ee}}
   ```

{% endnavtab %}
{% navtab RHEL %}

FIPS準拠のRed Hat 8パッケージは、`kong-enterprise-edition-fips`という明確な名前のパッケージを使用してインストールできます。パッケージをインストールするには、次の手順に従います。

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

FIPSの構成
-------

FIPS モードで起動するには、 {{site.base_gateway}}を起動する前に、 `kong.conf`構成ファイルで次の構成プロパティを`on`に設定します。

    fips = on # fips mode is enabled, causing incompatible ciphers to be disabled

この構成は、環境変数を使用して設定することもできます。

```bash
export KONG_FIPS=on
```

FIPS モードで{{site.base_gateway}} 3\.1 から 3\.2 に移行し、key\-auth\-enc プラグインを使用している場合は、既存のすべての key\-auth\-enc 認証情報に[PATCH または POST リクエスト](/hub/kong-inc/key-auth-enc/#create-a-key)を送信して、SHA256 で再ハッシュする必要があります。

{:.important .no-icon}
> 
> 非FIPSモードからFIPSモードへの移行、およびその逆の移行はサポートされていません。

