---
title: "Kong GatewayをUbuntuにインストールする"
---
{{site.base_gateway}} ソフトウェアは、[Kong ソフトウェアライセンス契約](https://konghq.com/kongsoftwarelicense)で管理されています。Kongは、[Apache 2\.0ライセンス](https://github.com/Kong/kong/blob/master/LICENSE)で
ライセンスされています。

{% include_cached /md/gateway/install-traditional-mode.md %}

前提条件
----

* [サポートされているシステム](/gateway/{{page.release}}/support-policy/#supported-versions)で、ルートまたは[ルートと同等](/gateway/{{page.release}}/production/running-kong/kong-user/)のアクセス権を持つもの。
* （Enterpriseのみ）Kongの`license.json`ファイル

必要なものがすべて揃ったら、インストールパスを選択します。

* [クイックスタート](#installation): {{site.base_gateway}}パッケージとPostgreSQLデータベースのスクリプトをインストールします
* [高度なインストール](#advanced-installation): インストールするピースを自分で選択する

{% if_version gte:3.2.x %}
{:.note}
> 
> **注：** 

* {{site.base_gateway}}は、[AWS Gravitonプロセッサ](https://aws.amazon.com/ec2/graviton/)での実行をサポートします。AWS GravitonがサポートされているすべてのAWSリージョンで実行できます。
* 2023年7月、Kongはパッケージホスティングが、download.konghq.comから[{{ site.links.download }}]({{ site.links.download }})に移行することを発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。 {% endif_version %}

{% if_version lte:3.2.x %}
{:.note}
> 
> **注:** 2023年7月、Kongはパッケージホスティングが download.konghq.comから[{{ site.links.download }}]({{ site.links.download }})に移行することを発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。
> {% endif_version %}

インストール
------

{{ site.base_gateway }} を使い始めるための最も速い方法は、インストールスクリプトを使用することです。

{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
bash <(curl -sS https://get.konghq.com/install) -v {{ page.versions.ee }}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
bash <(curl -sS https://get.konghq.com/install) -p kong -v {{ page.versions.ce }}
```

{% endnavtab %}
{% endnavtabs_ee %}

このスクリプトはオペレーティングシステムを検出し、正しいパッケージを自動的にインストールします。また、PostgreSQLデータベースもインストールされ、 {{ site.base_gateway }}がブートストラップされます。

{{site.base_gateway}}パッケージのみをインストールする場合は、 [「パッケージのインストール」](#package-install)セクションを参照してください。

### インストール状態の確認

スクリプトが完了したら、同じターミナルウィンドウで次のコマンドを実行します。

```bash
curl -i http://localhost:8001
```

`200`ステータスコードが表示されます。

### 次の手順

{{ site.base_gateway }}が稼働し始めたら、次の操作を実行できます。

* 任意: [エンタープライズライセンスを追加します](/gateway/{{ page.release }}/licenses/deploy/)
{% if_version gte:3.4.x -%}
* Kong Manager を有効にします。
  * [Kong Manager Enterprise](/gateway/{{ page.release }}/kong-manager/enable/)
  * [Kong Manager OSS](/gateway/{{ page.release }}/kong-manager-oss/)
{% endif_version -%}
{% if_version lte:3.3.x -%}

* [Kong Managerを有効化](/gateway/{{ page.release }}/kong-manager/enable/)
{% endif_version -%}
{% if_version lte:3.4.x -%}
* [Dev Portalを有効にします](/gateway/{{ page.release }}/kong-enterprise/dev-portal/enable/)
{% endif_version -%}
* [サービスとルートの作成](/gateway/{{ page.release }}/get-started/services-and-routes/)

高度なインストール
---------

### パッケージのインストール

インストールパッケージをダウンロードするか、APTリポジトリを使用して {{site.base_gateway}} をインストールできます。

次の手順では、パッケージ **のみ** をインストールし、データストアはインストールしません。
インストール後にセットアップしてください。

{% navtabs %}
{% navtab Package %}

コマンド行からUbuntuに{{site.base_gateway}}をインストールします。

1. Kongパッケージをダウンロードします。

   {% if_version gte:3.4.x %}
   現在、 {{ site.base_gateway }} Ubuntu Focalと Jammy用にパッケージ化されています。次のコマンドは、 `jammy`を実行していることを前提としています。
   別のリリースを使用している場合は、以下のコマンドで`jammy`を`$(lsb_release -sc)`またはリリース名に置き換えます。リリース名を確認するには`lsb_release -sc`を実行してください。
   {% endif_version %}
   {% if_version lte:3.3.x %}
   現在、 {{ site.base_gateway }} Ubuntu Bionic、Focal、Jammy用にパッケージ化されています。次のコマンドは、 `jammy`を実行していることを前提としています。
   別のリリースを使用している場合は、以下のコマンドで`jammy`を`$(lsb_release -sc)`またはリリース名に置き換えます。リリース名を確認するには`lsb_release -sc`を実行してください。
   {% endif_version %}

{% assign ubuntu_flavor = "jammy" %}
{% if page.release == "3.0.x" %}
{% assign ubuntu_flavor = "bionic" %}
{% endif %}

{% capture download_package %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
curl -Lo kong-enterprise-edition-{{page.versions.ee}}.deb "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/ubuntu/pool/{{ ubuntu_flavor }}/main/k/ko/kong-enterprise-edition_{{page.versions.ee}}/kong-enterprise-edition_{{page.versions.ee}}_$(dpkg --print-architecture).deb"
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
curl -Lo kong-{{page.versions.ce}}.deb "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/ubuntu/pool/{{ ubuntu_flavor }}/main/k/ko/kong_{{page.versions.ce}}/kong_{{page.versions.ce}}_$(dpkg --print-architecture).deb"
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ download_package | indent | replace: " </code>", "</code>" }}

2. パッケージをインストールします。

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

コマンドラインから APT リポジトリをインストールします。

{% assign gpg_key = site.data.installation.gateway[page.major_minor_version].gpg_key  %}
{% unless gpg_key %}
{% assign gpg_key = site.data.installation.gateway.legacy.gpg_key  %}
{% endunless %}

1. Kong APTリポジトリを設定します。

   {% if_version gte:3.4.x %}
   現在、 {{ site.base_gateway }} Ubuntu Focalと Jammy用にパッケージ化されています。次のコマンドは、 `jammy`を実行していることを前提としています。
   別のリリースを使用している場合は、以下のコマンドで`jammy`を`$(lsb_release -sc)`またはリリース名に置き換えます。リリース名を確認するには`lsb_release -sc`を実行してください。
   {% endif_version %}
   {% if_version lte:3.3.x %}
   現在、 {{ site.base_gateway }} Ubuntu Bionic、Focal、Jammy用にパッケージ化されています。次のコマンドは、 `jammy`を実行していることを前提としています。
   別のリリースを使用している場合は、以下のコマンドで`jammy`を`$(lsb_release -sc)`またはリリース名に置き換えます。リリース名を確認するには`lsb_release -sc`を実行してください。
   {% endif_version %}

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/gpg.{{ gpg_key }}.key" |  gpg --dearmor | sudo tee /usr/share/keyrings/kong-gateway-{{ page.major_minor_version }}-archive-keyring.gpg > /dev/null
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.deb.txt?distro=ubuntu&codename={{ ubuntu_flavor }}" | sudo tee /etc/apt/sources.list.d/kong-gateway-{{ page.major_minor_version }}.list > /dev/null
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
sudo apt-get install -y kong-enterprise-edition={{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
sudo apt-get install -y kong={{page.versions.ce}}
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ install_from_repo | indent | replace: " </code>", "</code>" }}

4. 任意: パッケージに `hold` のマークを付けて、誤ったアップグレードを防止します。 {% capture optional %} {% navtabs_ee %} {% navtab Kong Gateway %}

```bash
sudo apt-mark hold kong-enterprise-edition
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
sudo apt-mark hold kong
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}

{{ optional | indent | replace: " </code>", "</code>" }}

{% endnavtab %}
{% endnavtabs %}

### 次の手順

{{site.base_gateway}}を開始する前に、[データストアを設定](/gateway/{{page.release}}/install/post-install/set-up-data-store/)し、データストアを参照して`kong.conf.default`構成プロパティファイルを更新します。

目的の環境に応じて、次のガイドも参照してください。

* 任意: [エンタープライズライセンスを追加します](/gateway/{{ page.release }}/licenses/deploy/)
{% if_version gte:3.4.x -%}
* Kong Manager を有効にします。
  * [Kong Manager Enterprise](/gateway/{{ page.release }}/kong-manager/enable/)
  * [Kong Manager OSS](/gateway/{{ page.release }}/kong-manager-oss/)
{% endif_version -%}
{% if_version lte:3.3.x -%}

* [Kong Manager を有効にします](/gateway/{{ page.release }}/kong-manager/enable/) {% endif_version %}

{{site.base_gateway}}のシリーズもチェックしてみてください
[入門](/gateway/{{ page.release }}/get-started/)ガイドで方法を学ぶ
{{site.base_gateway}}を最大限に活用しましょう。

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

