---
title: "サードパーティツール"
toc: false
---
これらは{{site.base_gateway}}の日常業務で使用されるサービスです。これらのサービスの使用はオプションである場合もあれば、特定のプラグインによって必須となる場合もあります。

特に明記されていない限り、Kongは任意の最新サードパーティツールの2バージョンと、利用可能な場合は現在の管理バージョンをサポートします。

{:.note}
> 
> 以下のサードパーティツールにはバージョン番号がありません。これらのツールはマネージドサービスであり、Kongは現在リリースされているバージョンとの互換性を提供します。

{% navtabs %}
{% if_version gte: 3.8.x %}
{% navtab 3.8 %}
{% include_cached gateway-support-third-party.html data=site.data.tables.support.gateway.versions.38 %}
{% endnavtab %}
{% endif_version %}
{% if_version gte: 3.7.x %}
{% navtab 3.7 %}
{% include_cached gateway-support-third-party.html data=site.data.tables.support.gateway.versions.37 %}
{% endnavtab %}
{% endif_version %}
{% if_version gte: 3.6.x %}
{% navtab 3.6 %}
{% include_cached gateway-support-third-party.html data=site.data.tables.support.gateway.versions.36 %}
{% endnavtab %}
{% endif_version %}
{% navtab 3.5 %}
{% include_cached gateway-support-third-party.html data=site.data.tables.support.gateway.versions.35 %}
{% endnavtab %}
{% navtab 3.4 LTS %}
{% include_cached gateway-support-third-party.html data=site.data.tables.support.gateway.versions.34 %}
{% endnavtab %}
{% navtab 2.8 LTS %}
{% include_cached gateway-support-third-party.html data=site.data.tables.support.gateway.versions.28 %}
{% endnavtab %}
{% endnavtabs %}

