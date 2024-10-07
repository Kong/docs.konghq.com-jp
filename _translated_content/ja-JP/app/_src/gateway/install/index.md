---
title: "インストールオプション"
subtitle: "Kong Gatewayは、低需要でパフォーマンスの高いAPIゲートウェイです。Kong Gateway は Konnect を使用してセットアップすることも、さまざまな自己管理システムにインストールすることもできます。"
disable_image_expand: true
---
{% if_version lte:3.3.x %}
{% include install.html config=site.data.tables.install_options %}
{% endif_version %}

{% if_version gte:3.4.x %}
<blockquote class="note"> <p><strong>{{ site.konnect_product_name }}で5分以内にゲートウェイをセットアップしましょう。</strong></p> <p> <a href="/konnect/">{{ site.konnect_product_name }}</a>は、最新のアプリケーションをより良く、より速く、より安全に構築できる API ライフサイクル管理プラットフォームです。 </p> <p><a href="https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=install-gateway" class="no-link-icon">無料で始める →</a></p> </blockquote> 

または、独自の自己管理システムに{{site.base_gateway}}を設定することもできます。

{% include install.html config=site.data.tables.install_options_34x %}

{% endif_version %}

