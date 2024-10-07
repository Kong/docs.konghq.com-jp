{% if include.desc == "long" %}
{:.important}
> 
> **重要:** 以下の設定は、デフォルトの`admin_listen`設定をオーバーライドして、あらゆるソースからのリクエストをリッスンするため、非本番環境での使用 **のみ** を目的としています。インターネットに直接公開される環境では、これらの設定を使用 **しない** でください。
> <br> 本番環境で「admin\_listen」ポートをインターネットに公開する必要がある場合は、 {% if_version lte:2.8.x %}\[secure it with authentication\]\(/gateway/{{include.release}}/admin\-api/secure\-admin\-api/\).{% endif_version %}{% if_version gte:3.0.x %}\[secure it with authentication\]\(/gateway/{{include.release}}/production/running\-kong/secure\-admin\-api/\).{% endif_version %}  

{% endif %}
{% if include.desc == "short" %}
{:.important}
> 
> **重要** ：本番環境で`admin_listen`ポートをインターネットに公開する必要がある場合は、
> {% if_version lte:2.8.x %}[secure it with authentication](/gateway/{{include.release}}/admin-api/secure-admin-api/).{% endif_version %}{% if_version gte:3.0.x %}[secure it with authentication](/gateway/{{include.release}}/production/running-kong/secure-admin-api/).{% endif_version %}

{% endif %}

