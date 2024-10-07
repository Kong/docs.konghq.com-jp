---
title: "Kong GatewayをAmazon Linuxにインストールする"
---
{{site.base_gateway}}ソフトウェアは、[Kongソフトウェア
ライセンス契約](https://konghq.com/kongsoftwarelicense)で管理されています。
Kongは、[Apache 2\.0ライセンス](https://github.com/Kong/kong/blob/master/LICENSE)で
ライセンスされています。

{% include_cached /md/gateway/install-traditional-mode.md %}

前提条件
----

* ルートまたは[ルートと同等](/gateway/{{page.release}}/production/running-kong/kong-user/)のアクセス権を持つ、[サポートされているシステム。](/gateway/{{page.release}}/support-policy/#supported-versions)
* （Enterpriseのみ）Kongの`license.json`ファイル。

{% if_version gte:3.2.x %}
{:.note}
> 
> **注：** 

* {{site.base_gateway}} は、[AWS Graviton プロセッサ](https://aws.amazon.com/ec2/graviton/)での実行をサポートします。AWS Graviton がサポートされているすべての AWS リージョンで実行できます。
* 2023年7月、Kongはパッケージホスティングが、download.konghq.comから [{{ site.links.download }}]({{ site.links.download }})に移行することを発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。{% endif_version %}

{% if_version lte:3.2.x %}
{:.note}
> 
> **注:** 2023年7月、Kongはパッケージホスティングが download.konghq.com から [{{ site.links.download }}]({{ site.links.download }})に移行することを発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。
> {% endif_version %}

パッケージのインストール
------------

インストールパッケージをダウンロードするか、yum リポジトリを使用して {{site.base_gateway}} をインストールできます。

次の手順では、パッケージ **のみ** をインストールし、データストアはインストールしません。インストール後データストアをセットアップする必要があります。

{% navtabs %}
{% navtab Package %}
コマンドラインからAmazon Linuxに{{site.base_gateway}}をインストールします。

1. Kongパッケージをダウンロードします。

{% capture download_package %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
curl -Lo kong-enterprise-edition-{{page.versions.ee}}.rpm $(rpm --eval {{ site.links.direct }}/gateway-{{ page.major_minor_version }}/rpm/amzn/%{amzn}/%{_arch}/kong-enterprise-edition-{{page.versions.ee}}.aws.%{_arch}.rpm)
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
curl -Lo kong-{{page.versions.ce}}.rpm $(rpm --eval {{ site.links.direct }}/gateway-{{ page.major_minor_version }}/rpm/amzn/%{amzn}/%{_arch}/kong-{{page.versions.ce}}.aws.%{_arch}.rpm)
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
sudo yum install -y kong-enterprise-edition-{{page.versions.ee}}.rpm
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
sudo yum install -y kong-{{page.versions.ce}}.rpm
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ install_package | indent | replace: " </code>", "</code>" }}

{% endnavtab %}
{% navtab YUM repository %}

コマンドラインからYUMリポジトリをインストールします。

1. Kong YUMリポジトリをダウンロードします。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.rpm.txt?distro=amzn&codename=$(rpm --eval '%{amzn}')" | sudo tee /etc/yum.repos.d/kong-gateway-{{ page.major_minor_version }}.repo > /dev/null
   sudo yum -q makecache -y --disablerepo='*' --enablerepo='kong-gateway-{{ page.major_minor_version }}'
   ```

2. Kong をインストールします。

{% capture install_from_repo %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
sudo yum install -y kong-enterprise-edition-{{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
sudo yum install -y kong-{{page.versions.ce}}
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

パッケージをアンインストール
--------------

{{site.base_gateway}}を停止します。

    kong stop

{% navtabs_ee %}
{% navtab Kong Gateway %}
パッケージをアンインストールするには、次のコマンドを実行します。

    sudo yum remove kong-enterprise-edition

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}
パッケージをアンインストールするには、次のコマンドを実行します。

    sudo yum remove kong

{% endnavtab %}
{% endnavtabs_ee %}

