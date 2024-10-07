---
title: "Kong Gateway"
breadcrumb: "概要"
subtitle: "ハイブリッドクラウドとマルチクラウド向けに構築され、マイクロサービスと分散アーキテクチャに最適化されたAPIゲートウェイ"
description: "Kong Gateway is a lightweight, fast, and flexible cloud-native API gateway. Kong is a reverse proxy that lets you manage, configure, and route requests"
konnect_cta_card: true
---
<blockquote class="note"> <p><strong>{{ site.konnect_product_name }} で 5 分以内にゲートウェイをセットアップしましょう。</strong></p> <p> <a href="/konnect/">{{ site.konnect_product_name }}</a>は、クオリティ、スピード、安全性に優れた方法で最新アプリケーションを構築できるAPIライフサイクルマネジメントプラットフォームです。 </p> <p><a href="https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=install-gateway" class="no-link-icon">無料で始める →</a></p> </blockquote> 

使用開始方法
------

<div class="docs-grid-install max-3"> <a href="/konnect/getting-started/" class="docs-grid-install-block no-description"> <img class="install-icon no-image-expand small" src="/assets/images/icons/kong-gradient.svg" alt=""> <div class="install-text">Konnect の使用を開始する<br> <span class="badge recommended"></span></div> </a> <a href="/gateway/{{page.release}}/get-started/" class="docs-grid-install-block no-description"> <img class="install-icon no-image-expand small" src="/assets/images/icons/third-party/docker.svg" alt=""> <div class="install-text">ローカル環境でDocker の使用を開始する</div> </a> <a href="/gateway/{{page.release}}/install/" class="docs-grid-install-block no-description"> <img class="install-icon no-image-expand small" src="/assets/images/icons/documentation/icn-deployment-color.svg" alt=""> <div class="install-text">プラットフォームにインストールする</div> </a> </div> <div class="docs-grid-install docs-grid-install__bottom max-2"> <a href="/hub/" class="docs-grid-install-block docs-grid-install-block__bottom"> <img class="install-icon no-image-expand small" src="/assets/images/icons/documentation/icn-api-plugins-color.svg" alt=""> <div class="install-block-column"> <div class="install-text">Kong Plugin Hub</div> <div class="install-description">強力なプラグインでゲートウェイを拡張する</div> </div> </a> <a href="/gateway/{{page.release}}/admin-api/" class="docs-grid-install-block docs-grid-install-block__bottom"> <img class="install-icon no-image-expand small" src="/assets/images/icons/documentation/icn-admin-api-color.svg" alt=""> <div class="install-block-column"> <div class="install-text">APIのリファレンスドキュメント</div> <div class="install-description">管理目的で内部 REST API を設定</div> </div> </a> </div> 

{{site.base_gateway}}で実行できる機能の詳細については、[機能](#features)を参照してください。

{{ site.base_gateway }}の紹介
--------


{{site.base_gateway}}は軽量、高速、柔軟なクラウドネイティブAPIゲートウェイです。APIゲートウェイは、リクエストの管理、設定、APIへのルーティングを可能にするリバースプロキシです。


{{site.base_gateway}} runs in front of any RESTful API and can be extended through
modules and plugins. It's designed to run on decentralized architectures, including
hybrid\-cloud and multi\-cloud deployments.

{{site.base_gateway}}を使用すると、次の目的を実現できます。

* ワークフローの自動化と最新の GitOps プラクティスを活用する
* アプリケーション/サービスの分散化と、マイクロサービスへの移行
* Create a thriving API developer ecosystem
* API関連の異常と脅威をプロアクティブに特定
* API/サービスを保護および管理し、組織全体でAPIの可視性を向上させます。 

<blockquote class="note no-icon" id="nurture-signup"> <p>追加のサポートが必要ですか？あなただけの無料トレーニングと厳選されたコンテンツ：</p> <form action="https://go.konghq.com/l/392112/2022-09-19/cfr97r" method="post"> <input class="button" name="email" placeholder="you@yourcompany.com" /> <button class="button" type="submit">Sign up now</button> </form> </blockquote> 

{{site.base_gateway}}の拡張
---------


{{site.base_gateway}}はNginxで実行されているLuaアプリケーションです。{{site.base_gateway}}は[OpenResty](https://openresty.org/)と一緒に配布されており、これは[lua\-nginx\-module](https://github.com/openresty/lua-nginx-module)を拡張するモジュールのバンドルです。

これにより、プラグインを実行時に有効にして実行できるモジュラーアーキテクチャの基礎が確立されます。その核となるのは、
{{site.base_gateway}}がデータベースの抽象化、ルーティング、プラグイン管理を実装することです。プラグインは別のコードベースに存在し、リクエストライフサイクルのどこにでも挿入で、すべて数行のコードで完了します。

Kongは、Gatewayデプロイメントで使用できる多くの[プラグイン](#kong-gateway-plugins)を提供しています。独自のカスタムプラグインを作成することもできます。詳細については、[プラグイン開発ガイド](/gateway/{{page.release}}/plugin-development)、[PDKリファレンス](/gateway/{{page.release}}/plugin-development/pdk/)、および他の言語（[JavaScript](/gateway/{{page.release}}/plugin-development/pluginserver/javascript)、[Go](/gateway/{{page.release}}/plugin-development/pluginserver/go)、[Python](/gateway/{{page.release}}/plugin-development/pluginserver/python/)）でのプラグイン作成に関するガイドを参照してください。

Packages and modes
------------------

{{site.base_gateway}}をデプロイするには、{{ site.konnect_saas }}での管理と自己管理の2つの方法があります。初めて{{site.base_gateway}}を試す場合は、[{{ site.konnect_saas }}](https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=gateway-mode-overview)から始めることをおすすめします。

### {{site.konnect_short_name}}


{{site.konnect_short_name}} provides the easiest way to get started
with {{site.base_gateway}}.
The global control plane is hosted in the cloud by Kong, and you manage the
individual data plane nodes within your preferred network environment.


{{site.konnect_short_name}}の価格パッケージは2通りあります。

* **Plus** : Our self\-serve pay\-as\-you\-go pricing model, giving you access to the
  
{{site.konnect_short_name}} platform in its entirety while offering the flexibility
  to only pay for the services your organization uses.

* **Enterprise** : With an Enterprise subscription,
  you have access to the entire {{ site.konnect_saas }} suite and:

  * 24x7x365 technical support
  * お客様の環境に合わせた専用ソリューションを作成するプロフェッショナルサービス

詳細については、[料金ページ](https://konghq.com/pricing)をご覧ください。

![{{site.konnect_short_name}}の{{site.base_gateway}}の紹介](/assets/images/products/konnect/gateway-manager/konnect-control-planes-example.png)
> 
> *図1: {{site.konnect_short_name}} コントロールプレーン（CP）に接続された {{site.base_gateway}} データプレーン（DP）の図。*
> <br>
> *API クライアントからのリクエストはゲートウェイのデータプレーン（DP）へ流れ、コントロールプレーン（CP）の構成に基づきプロキシによって変更および管理されます。そして、アップストリームサービスへ転送されます。* 

### 自己管理型


{{site.base_gateway}} は、オープンソース（OSS）とEnterpriseの2つのパッケージで利用できます。

**{{site.ce_product_name}}** ：基本的なAPIゲートウェイ機能とオープンソースプラグインを含む
オープンソースパッケージ機能。Kongの[Admin API](#kong-admin-api){% if_version gte:3.4.x %}、[Kong Manager
オープンソース](/gateway/{{page.release}}/kong-manager/)、{% endif_version %}または[宣言型構成](#deck)を使用して、オープンソースのゲートウェイを管理できます。

**{{site.ee_product_name}}** （[無料モードまたはEnterpriseモード](https://konghq.com/pricing)で利用可能）：KongのAPIゲートウェイ機能が追加されたもの。

* <span class="badge free"></span> **フリーモード** では、このパッケージは、[Kong Manager](#kong-manager)を基本的なオープンソース機能に追加します。
* <span class="badge enterprise"></span> With an **Enterprise** subscription, it also includes:
  {% if_version lte:3.4.x -%}
  * [Dev Portal](#kong-dev-portal)
  * [Vitals](#kong-vitals)
  {% endif_version -%}
  * [RBAC](/gateway/api/admin-ee/latest/#/rbac/get-rbac-users)
  * [Enterprise プラグイン](/hub/)

{{site.ee_product_name}}は無料モードまたはEnterpriseモードで、Kongの[Admin API](#kong-admin-api)、[宣言型構成](#deck)、または[Kong Manager](#kong-manager)を使用して管理できます。

![{{site.base_gateway}}の紹介](/assets/images/products/gateway/kong-gateway-features.png)
> 
> *図2：{{site.base_gateway}}の主な機能の図。{{site.ce_product_name}}は基本機能を備えています。一方、{{site.ee_product_name}}は高度なプロキシ機能を備えたオープンソース基盤上に構築されています。*
> <br>
> *リクエストはAPIクライアントからゲートウェイに流れ、ゲートウェイ構成に基づきプロキシによって変更、管理され、アップストリームサービスに転送されます。*

特徴
---

{% include_cached feature-table.html config=site.data.tables.features.gateway %}

### Kong Admin API

[Kong Admin API](/gateway/{{page.release}}/admin-api)は、サービス、ルート、プラグイン、コンシューマなどのGatewayエンティティの管理と構成用のRESTfulインターフェースを提供します。Gatewayに対して実行できるすべてのタスクは、Kong Admin APIを使用して自動化できます。

### Kong Manager
{:.badge .free}

{:.note}
> 
> **Note** : If you are running Kong in traditional mode, increased traffic could
> lead to potential performance with Kong Proxy.
> Server\-side sorting and filtering large quantities of entities will also cause increased CPU usage in both Kong CP and database.

[Kong Manager](/gateway/{{page.release}}/kong-manager/) は {{site.base_gateway}} のグラフィカルユーザーインターフェイス（GUI）です。内部的には Kong Admin API を使用して {{site.base_gateway}} を管理および制御します。

Kong Managerでできることを以下にいくつか挙げます。

* 新しいルートとサービスの作成
* Activate or deactivate plugins with a couple of clicks
* チーム、サービス、プラグイン、コンシューマ管理、その他すべてを望みどおりにグループ化します。
{% if_version lte:3.4.x -%}
* パフォーマンスの監視：クラスタ全体、ワークスペースレベルで、または オブジェクトレベルの正常性を直感的でカスタマイズ可能なダッシュボードを使って視覚化 {% endif_version %}

{% if_version lte:3.4.x %}

### Kong Dev Portal
{:.badge .enterprise}

[Kong Dev Portal](/gateway/{{page.release}}/kong-enterprise/dev-portal/)は、新しい開発者のオンボード、APIドキュメントの生成、カスタムページの作成、APIバージョンの管理、開発者のアクセスの保護に使用されます。

### Kong Vitals
{:.badge .enterprise}

[Kong Vitals](/gateway/{{page.release}}/kong-enterprise/analytics/)は、{{site.base_gateway}}ノードのヘルスとパフォーマンスに関する有用なメトリクスと、プロキシされたAPIの使用状況に関するメトリクスを提供します。バイタルサインを視覚的に監視し、リアルタイムで異常を特定し、視覚的なAPI分析を使用して、APIとゲートウェイのパフォーマンスを正確に確認し、主要な統計にアクセスできます。Kong VitalsはKong Manager UIの一部です。
{% endif_version %}

### Kubernetes


{{site.base_gateway}} は、カスタム[イングレスコントローラー](/kubernetes-ingress-controller/)、Helm チャート、および演算子を使用して Kubernetes 上でネイティブに実行できます。Kubernetes イングレスコントローラーは、Kubernetes クラスタで実行されているアプリケーション（デプロイメント、ReplicaSets など）から、クラスタの外部で実行されているクライアントアプリケーションに Kubernetes サービスを公開するプロキシです。イングレスコントローラーの目的は、Kubernetes クラスタへのすべての受信トラフィックを一元的に制御することです。

### {{site.base_gateway}}のプラグイン

[{{site.base_gateway}} プラグイン](/hub/)は、APIとマイクロサービスをより適切に管理するための高度な機能を提供します。{{site.base_gateway}} プラグインは、最も困難なユースケースに対応するターンキー機能を備えており、最大限の制御を保証し、不要なオーバーヘッドを最小限に抑えます。Kong Manager または Admin API を通じて{{site.base_gateway}}プラグインを有効にすることで、認証、流量制限、変換などの機能を有効にします。

ツール
---

Kongは、{{site.base_gateway}}で使用できるAPIライフサイクルマネジメントツールも提供します。

### Insomnia

[Insomnia](https://docs.insomnia.rest)は、すべてのRESTおよびGraphQLサービスに対して仕様優先の開発を可能にします。Insomniaを使用すると、組織は自動テスト、Git直接同期、すべての応答タイプの検査を使用して、設計とテストのワークフローを加速できます。あらゆる規模のチームがInsomniaを使用して開発速度を上げ、デプロイメントのリスクを軽減し、コラボレーションを強化できます。

### decK

[decK](/deck/)は、宣言的な方法で{{site.base_gateway}}の構成を管理するのに役立ちます。つまり、開発者はサービス、ルート、プラグインなど、{{site.base_gateway}}または
{{site.konnect_short_name}}の望ましい状態を定義し、Kong Admin APIのように各ステップを手動で実行する必要がなく、decKに実装を任せることができます。

Get started with {{site.base_gateway}}
-----------------------

[{{site.base_gateway}}をダウンロードしてインストールします](/gateway/{{page.release}}/install/)。
テストするには、オープンソースパッケージを選択するか、{{site.ee_product_name}}を無料モードで実行してKong Managerを試してみるかの、いずれかを選ぶことができます。

After installation, get started with the introductory [quickstart guide](/gateway/{{page.release}}/get-started/).

### {{site.konnect_short_name}}で試す

[{{site.konnect_product_name}}](/konnect/)は{{site.base_gateway}}インスタンスを管理できます。この設定では、Kongがコントロールプレーン（CP）をホストし、あなたは独自のデータプレーンをホストします。

There are a few ways to test out the Gateway's Enterprise features:

* [{{site.konnect_product_name}}](https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=gateway-overview)にサインアップします。
* [Kong Academy]({{site.links.learn}})の学習ラボを参照してください。
* Enterprise機能をローカルで評価することをご希望の場合は、[デモをリクエスト](https://konghq.com/get-started/#request-demo)してください。Kong担当者が詳細についてご連絡し、開始のお手伝いをいたします。

サポートポリシー
--------

Kongは、製品のバージョン管理に構造化されたアプローチを採用しています。

{{site.ee_product_name}}と
{{site.mesh_product_name}}の最新バージョンのサポート情報については、[バージョンのサポートポリシー](/gateway/{{page.release}}/support-policy/)を参照してください。

