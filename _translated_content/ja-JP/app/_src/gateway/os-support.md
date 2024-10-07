---
title: "サポートされているオペレーティングシステム"
---
{% assign os_versions = site.data.tables.os_support %}

サポートされるOS
---------


{{site.base_gateway}}は現在、次のLinuxオペレーティングシステムをサポートしています。
<table> <thead> <th>OSバージョン</th> <th style="text-align: center">最初のEnterprise Gatewayバージョン</th> <th style="text-align: center">最初のオープンソース Gateway バージョン</th> </thead> <tbody> {% for item in os_versions %} {% if item.status == "active" %} <tr> <td> 
          {{ item.name }} </td> <td style="text-align: center"> {% if item.enterprise == true and item.enterprise_versions.first_version != null %} 
            {{ item.enterprise_versions.first_version }} {% else %} はサポート対象外 {% endif %} </td> <td style="text-align: center"> {% if item.oss == true and item.oss_versions.first_version != null %} 
            {{ item.oss_versions.first_version }} {% else %} はサポート対象外 {% endif %} </td> </tr> {% endif %} {% endfor %} </tbody> </table> 

非推奨
---

以下のバージョンはサポート終了（EoL）となり、Kong ではサポートされません。
<table> <thead> <th>OSバージョン</th> <th>サポート終了</th> <th style="text-align: center">最後のEnterprise Gatewayバージョン</th> <th style="text-align: center">最後のオープンソースGatewayバージョン</th> </thead> <tbody> {% for item in os_versions %} {% if item.status == "deprecated" %} <tr> <td> 
          {{ item.name }} </td> <td> <a href="{{ item.eol_link }}">{{ item.deprecation_date }}</a> </td> <td style="text-align: center"> {% if item.enterprise == true %} 
            {{ item.enterprise_versions.last_version }} {% else %} 適用不可 {% endif %} </td> <td style="text-align: center"> {% if item.oss == true %} 
            {{ item.oss_versions.last_version }} {% else %} 適用不可 {% endif %} </td> </tr> {% endif %} {% endfor %} </tbody> </table>

