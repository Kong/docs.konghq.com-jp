---
title: "RHELへのKong Gatewayのインストール"
---
{{site.base_gateway}} ソフトウェアには、[Kong ソフトウェア ライセンス アグリーメント](https://konghq.com/kongsoftwarelicense)が適用されます。Kongは、[Apache 2\.0 ライセンス](https://github.com/Kong/kong/blob/master/LICENSE)でライセンスされています。

{% include_cached /md/gateway/install-traditional-mode.md %}

前提条件
----

* [サポートされているシステム](/gateway/{{page.release}}/support-policy/#supported-versions)で、ルートまたは[ルートと同等](/gateway/{{page.release}}/production/running-kong/kong-user/)のアクセス権を持つもの。
* （エンタープライズのみ）Kongの`license.json`ファイル

{:.note}
> 
> **注：** 2023年7月に、Kongはパッケージホスティングがdownload.konghq.comから[{{ site.links.download }}]({{ site.links.download }})に移行すると発表しました。詳細については、こちらの[ブログ記事](https://konghq.com/blog/product-releases/changes-to-kong-package-hosting)をご覧ください。

パッケージのインストール
------------

インストールパッケージをダウンロードするか、yumリポジトリを使用して{{site.base_gateway}}をインストールできます。

次の手順では、パッケージ **のみ** をインストールし、データストアはインストールしません。
インストール後にセットアップしてください。

{% navtabs %}
{% navtab Package %}

コマンドラインからRHELに{{site.base_gateway}}をインストールします。

1. Kong パッケージをダウンロードします。

{% capture download_package %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}

```bash
curl -Lo kong-enterprise-edition-{{page.versions.ee}}.rpm $(rpm --eval {{ site.links.direct }}/gateway-{{ page.major_minor_version }}/rpm/el/%{rhel}/%{_arch}/kong-enterprise-edition-{{page.versions.ee}}.el%{rhel}.%{_arch}.rpm)
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
curl -Lo kong-{{page.versions.ce}}.rpm $(rpm --eval {{ site.links.direct }}/gateway-{{ page.major_minor_version }}/rpm/el/%{rhel}/%{_arch}/kong-{{page.versions.ce}}.el%{rhel}.%{_arch}.rpm)
```

{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ download_package | indent | replace: " </code>", "</code>" }}

2. `yum`または`rpm`を使用してパッケージをインストールします。

   `rpm` インストール方法を使用する場合、パッケージには {{site.base_gateway}} *のみ* が含まれます。パッケージに依存関係は含まれていません。

{% capture install_package %}
{% navtabs %}
{% navtab yum %}
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
{% endnavtab %}
{% navtab rpm %}

{:.important}
> 
> `rpm` メソッドはオープンソース パッケージでのみ使用できます。`kong-enterprise-edition` パッケージの場合は、`yum` を使用します。

```bash
rpm -iv kong-{{page.versions.ce}}.rpm
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}


{{ install_package | indent | replace: " </code>", "</code>" }}

    Installing directly using `rpm` is suitable for Red Hat's [Universal Base Image](https://developers.redhat.com/blog/2020/03/24/red-hat-universal-base-images-for-docker-users) "minimal" variant. You will need to install Kong's dependencies separately via `microdnf`.

{% endnavtab %}
{% navtab YUM repository %}

コマンドラインから YUM リポジトリをインストールします。

1. Kong YUMリポジトリをダウンロードします。

   ```bash
   curl -1sLf "{{ site.links.direct }}/gateway-{{ page.major_minor_version }}/config.rpm.txt?distro=el&codename=$(rpm --eval '%{rhel}')" | sudo tee /etc/yum.repos.d/kong-gateway-{{ page.major_minor_version }}.repo
   sudo yum -q makecache -y --disablerepo='*' --enablerepo='kong-gateway-{{ page.major_minor_version }}'
   ```

2. Kongをインストールします。
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

### 次のステップ

{{site.base_gateway}}を開始する前に、[データストアを設定](/gateway/{{page.release}}/install/post-install/set-up-data-store/)し、データストアを参照して`kong.conf.default`構成プロパティファイルを更新します。

目的の環境に応じて、次のガイドも参照してください。

* [Enterpriseライセンスを適用する](/gateway/{{page.release}}/licenses/deploy/)
{% if_version gte:3.4.x -%}
* Kong Manager を有効にします。
  * [Kong Manager Enterprise](/gateway/{{ page.release }}/kong-manager/enable/)
  * [Kong Manager OSS](/gateway/{{ page.release }}/kong-manager-oss/)
{% endif_version -%}
{% if_version lte:3.3.x -%}

* [Kong Managerを有効にする](/gateway/{{ page.release }}/kong-manager/enable/) {% endif_version %}

{{site.base_gateway}}のシリーズもチェックしてみてください
[入門](/gateway/{{ page.release }}/get-started/)ガイドで方法を学ぶ
{{site.base_gateway}}を最大限に活用しましょう。

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

