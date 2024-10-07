---
title: "Debian に Kong Gateway をインストールする"
---
{{site.base_gateway}} ソフトウェアは、[Kong ソフトウェアライセンス契約](https://konghq.com/kongsoftwarelicense)で管理されています。
{{site.ce_product_name}} は、[Apache 2\.0 ライセンス](https://github.com/Kong/kong/blob/master/LICENSE)でライセンスされています。

{% include_cached /md/gateway/install-traditional-mode.md %}

前提条件
----

* [サポートされているシステム](/gateway/{{page.release}}/support-policy/#supported-versions)で、ルートまたは[ルートと同等](/gateway/{{page.release}}/production/running-kong/kong-user/)のアクセス権を持つもの。
* 次のツールがインストールされています。
  * [`curl`](https://curl.se/)
  * [`lsb-release`](https://packages.debian.org/lsb-release)
  * [`apt-transport-https`](https://packages.debian.org/apt-transport-https)（APTリポジトリをインストールする場合のみ）

* （Enterpriseのみ）Kongの`license.json`ファイル。

{% if_version gte:3.2.x %}
{:.note}
> 
> **注：** 

* {{site.base_gateway}}は、[AWS Gravitonプロセッサ](https://aws.amazon.com/ec2/graviton/)上の実行をサポートします。AWS GravitonがサポートされているすべてのAWSリージョンで実行できます。
* 2023年7月、Kongはパッケージホスティングが、download.konghq.comから [{{ site.links.download }}]({{ site.links.download }})に移行することを発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。 {% endif_version %}

{% if_version lte:3.2.x %}
{:.note}
> 
> **注:** 2023年7月、Kongはパッケージホスティングが download.konghq.comから[{{ site.links.download }}]({{ site.links.download }})に移行することを発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。{% endif_version %}

パッケージのインストール
------------

インストールパッケージをダウンロードするか、APTリポジトリを使用して {{site.base_gateway}} をインストールできます。

次の手順では、パッケージ **のみ** をインストールし、データストアはインストールしません。インストール後にセットアップしてください。

{% navtabs %}
{% navtab Package %}

コマンドラインからDebianに{{site.base_gateway}}をインストールします。

1. Kongパッケージをダウンロードします。

{% capture download_package %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
curl -Lo kong-enterprise-edition-{{page.versions.ee}}.deb "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/debian/pool/bullseye/main/k/ko/kong-enterprise-edition_{{page.versions.ee}}/kong-enterprise-edition_{{page.versions.ee}}_$(dpkg --print-architecture).deb"
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
curl -Lo kong-{{page.versions.ce}}.deb "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/debian/pool/bullseye/main/k/ko/kong_{{page.versions.ce}}/kong_{{page.versions.ce}}_$(dpkg --print-architecture).deb"
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ download_package | indent | replace: " </code>", "</code>" }}

2. 以下のようにパッケージをインストールします。

{% capture install_package %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
sudo apt install -y ./kong-enterprise-edition-{{page.versions.ee}}.deb
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
sudo apt install -y ./kong-{{page.versions.ce}}.deb
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ install_package | indent | replace: " </code>", "</code>" }}

{% endnavtab %}
{% navtab APT repository %}

コマンドラインからAPTリポジトリをインストールします。

{% assign gpg_key = site.data.installation.gateway[page.major_minor_version].gpg_key  %}
{% unless gpg_key %}
{% assign gpg_key = site.data.installation.gateway.legacy.gpg_key  %}
{% endunless %}

1. Kong APTリポジトリをダウンロードします。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/gpg.{{ gpg_key }}.key" |  gpg --dearmor | sudo tee /usr/share/keyrings/kong-gateway-{{ page.major_minor_version }}-archive-keyring.gpg > /dev/null
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.deb.txt?distro=debian&codename=$(lsb_release -sc)" | sudo tee /etc/apt/sources.list.d/kong-gateway-{{ page.major_minor_version }}.list > /dev/null
   ```

2. リポジトリを更新します。 

   ```bash
   sudo apt-get update
   ```

3. Kongをインストールします。

{% capture install_from_repo %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
sudo apt install -y kong-enterprise-edition={{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
sudo apt install -y kong={{page.versions.ce}}
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ install_from_repo | indent | replace: " </code>", "</code>" }}

{% endnavtab %}
{% endnavtabs %}

### 次の手順

{{site.base_gateway}}を開始する前に、[データストアを設定](/gateway/{{page.release}}/install/post-install/set-up-data-store/)し、データストアを参照して`kong.conf.default`構成プロパティファイルを更新します。

目的の環境に応じて、次のガイドも参照してください。

* オプション：[Enterpriseライセンスを追加します](/gateway/{{ page.release }}/licenses/deploy/)
{% if_version gte:3.4.x -%}
* Kong Managerを有効にします。
  * [Kong Manager Enterprise](/gateway/{{ page.release }}/kong-manager/enable/)
  * [Kong Manager OSS](/gateway/{{ page.release }}/kong-manager-oss/)
{% endif_version -%}
{% if_version lte:3.3.x -%}

* [Kong Managerを有効にする](/gateway/{{ page.release }}/kong-manager/enable/) {% endif_version %}

{{site.base_gateway}}を最大限に活用する方法については、{{site.base_gateway}}のシリーズの[使用開始](/gateway/{{ page.release }}/get-started/)ガイドも参照してください。

パッケージのアンインストール
--------------

{{site.base_gateway}}を停止します。

    kong stop

{% navtabs_ee %}
{% navtab Kong Gateway %}
パッケージをアンインストールするには、次のコマンドを実行します。

    sudo apt remove kong-enterprise-edition

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}
パッケージをアンインストールするには、次のコマンドを実行します。

    sudo apt remove kong

{% endnavtab %}
{% endnavtabs_ee %}

