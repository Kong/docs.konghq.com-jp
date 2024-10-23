---
title: "Kong AI Gateway"
content-type: "explanation"
---
Kong AI Gatewayは、 [{{site.base_gateway}}](/gateway/latest/)の上に構築された強力な機能セットです。
開発者や組織が AI 機能を迅速かつ安全に効果的に導入できるように設計されています。

始め方
---

<div class="docs-grid-install docs-grid-install__bottom max-3"> <a href="/gateway/{{page.release}}/get-started/ai-gateway/" class="docs-grid-install-block docs-grid-install-block__bottom"> <img class="install-icon no-image-expand small" src="/assets/images/icons/documentation/icn-flag.svg" alt=""> <div class="install-block-column"> <div class="install-text">使用開始</div> <div class="install-description">ガイドを使って1分で開始</div> </div> </a> <a href="https://konghq.com/products/kong-ai-gateway#videos" class="docs-grid-install-block docs-grid-install-block__bottom no-link-icon"> <img class="install-icon no-image-expand small" src="/assets/images/icons/documentation/icn-learning.svg" alt=""> <div class="install-block-column"> <div class="install-text">動画</div> <div class="install-description">ビデオチュートリアルを見る</div> </div> </a> <a href="/hub/?category=ai" class="docs-grid-install-block docs-grid-install-block__bottom"> <img class="install-icon no-image-expand small" src="/assets/images/icons/documentation/icn-api-plugins-color.svg" alt=""> <div class="install-block-column"> <div class="install-text">AI プラグイン</div> <div class="install-description">Kong Plugin Hub 上の AI プラグインを確認してください</div> </div> </a> </div> 

Kong AI Gatewayとは？
------------------

複数の AI LLM プロバイダー（オープンソースやセルフホスト型を含む）の急速な出現により、AI テクノロジー業界は断片化しており、その基準や管理が不足しています。これにより、開発者や組織の AI サービスの使用および管理が大幅に複雑化しています。{{site.base_gateway}} の幅広い APIマネジメント機能とプラグイン拡張モデルにより、AI に特化した API マネジメントおよびガバナンスサービスを提供するのに非常に適しています。

AIプロバイダーは標準API仕様に準拠していませんが、AI Gatewayが標準化された APIレイヤーを提供することで、クライアントは同じクライアントコードベースから複数のAIサービスを利用できるようになります。
AI Gatewayは、迅速なエンジニアリングによって、認証情報管理、AI使用の監視、ガバナンス、チューニングのための追加機能を提供します。開発者はコード不要のAIプラグインを使用して、既存のAPIトラフィックを強化し、既存のアプリケーション機能を簡単に強化できます。

AIゲートウェイ機能は、最新の専用プラグインを通じて有効にすることができます。
他の{{site.base_gateway}}プラグインで使用するのと同じモデルを使用します。
既存の[{{site.base_gateway}}プラグイン](/hub/?category=ai)と一緒にデプロイすると、

{{site.base_gateway}}ユーザーは洗練されたAI管理プラットフォームを素早く構築できる
カスタムコードや新しい馴染みのないツールを導入する必要はありません。

![AI Gateway](/assets/images/products/gateway/getting-started-guide/ai-gateway.png)

AI Gatewayの機能
-------------

以下で、AI Gatewayの幅広い機能について説明します。詳細は、Kong [Plugin Hub](/hub/?category=ai)に
あるAI Gatewayプラグインをご覧ください。

### AIプロバイダープロキシ

AI Gatewayのコアとなるのは、プロバイダーに依存しないAPIを介して公開されるさまざまなプロバイダーに、AIリクエストをルーティングする機能です。この正規化されたAPIレイヤーにより、開発者と組織は次のようなさまざまなメリットを得られます。

* クライアントアプリケーションはAIプロバイダーのAPI仕様から保護され、コードの再使用可能性が促進されます
* 一元化されたAIプロバイダーの認証情報管理
* AI Gatewayは、開発者や組織にAIデータとその使用状況のガバナンスと監視を一元的に提供します。
* リクエストルーティングは動的に実行できるため、コスト、使用状況、応答精度などのさまざまなメトリックに基づいてAIの使用を最適化できます。
* AI サービスは、他の{{site.base_gateway}}プラグインによって非 AI API トラフィックを拡張するために使用できます。

{% if_version lte:3.7.x %}
このAI Gatewayのコア機能は、[AI Proxy](/hub/kong-inc/ai-proxy/)プラグインで有効になります。このプラグインは、
、上記で参照した getting started スクリプトでデフォルトで導入されています。
{% endif_version %}
{% if_version gte:3.8.x %}
このコアAIゲートウェイ機能は、[AI Proxy](/hub/kong-inc/ai-proxy/) と[AI Proxy Advanced](/hub/kong-inc/ai-proxy-advanced/) プラグインで有効になります。上記のquickstartスクリプトは、基本的なAIプロキシプラグインを使用します。 ロードバランシングとセマンティックルーティング機能については、AI Proxy Advanced プラグインを参照してください。 
{% endif_version %}

AI Proxyは、次の2種類のLLMリクエストをサポートします。

* *完了* ：AIシステムが1つのプロンプトに基づいてテキスト出力を生成するよう求められるリクエストの一種。 完了はコンフィギュレーション・キー`route_type`と値`llm/v1/completions`を使って設定します。
* *チャット* ：会話型AIインターフェースの一部であるリクエストの一種。`chat`リクエストでは、AIはユーザー入力に対してダイアログ応答を返すことが期待され、AI システムは会話履歴に基づいて応答します。チャットは設定キー`route_type`と値`llm/v1/chat`を使用して設定します。

コアプロキシ動作は、以下がホストするAIプロバイダーをサポートします。

* [OpenAI](https://openai.com/product)
* [Cohere](https://docs.cohere.com/reference/about)
* [Azure](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)
* [Anthropic](https://www.anthropic.com/)

ホスト型AIプロバイダーに加えて、セルフホスト型モデルもサポートされています。ローカルモデルの実行を可能にするツールの例としては、[Ollama](https://ollama.ai/)があります。次のローカルモデルがサポートされています。

* [Mistral](https://mistral.ai/)
* [Llama2](https://llama.meta.com/)

プロキシ動作の変更の詳細については、[AI Proxyプラグインの構成](/hub/kong-inc/ai-proxy/configuration/)を参照してください。

### AI使用ガバナンス

AI技術の導入が加速するなか、開発者と開発組織は一連の新たなリスクベクトルにさらされています。
特に、AIプロバイダーに機密データが漏洩すると、組織と顧客はデータ侵害などのセキュリティリスクに直面することになります。

KongのAI Gatewayは、開発者がAIデータと使用状況を制御できるようにするための追加のプラグインを提供します。これらのプラグインは、AI Proxyプラグインと組み合わせて使用されるため、ユーザーのために安全でAIに特化したエクスペリエンスを構築できます。

#### データガバナンス

AI Gatewayは、送信する AI プロンプトを許可/拒否リスト設定で管理する機能を提供します。プロンプトが拒否されると、クライアントへの 4xx HTTPコード応答が返され、 問題のリクエストの回避が妨げられます。

* [AI Prompt Guard](/hub/kong-inc/ai-prompt-guard) プラグインは正規表現を使用して許可/拒否リストを設定できます。

{% if_version gte:3.8.x %}

* [AI Semantic Prompt Guard](/hub/kong-inc/ai-semantic-prompt-guard)プラグインでは、意味的に類似したプロンプトを使用して許可/拒否リストを設定できます。{% endif_version %}

#### 迅速なエンジニアリング

AIシステムはプロンプトを中心に構築されており、技術を正常に採用するためにはこれらのプロンプトを操作することが重要です。プロンプトエンジニアリングは、AIシステムを導く言語入力を操作する方法論です。AI Gatewayは、デフォルトのプロンプトを設定するか、ゲートウェイを介してパスされるクライアントからのプロンプトを操作することで、簡素化および強化された体験を作成できる一連のプラグインをサポートします。

* [AI Prompt Template](/hub/kong-inc/ai-prompt-template)プラグインを使用すると、管理者は事前設定されたAIプロンプトをユーザーに提供できます。これらのプロンプトには、ユーザーが
  テンプレートを特定のニーズに合わせるために入力する、`{% raw %}{{variable}}{% endraw %}`形式の変数のプレースホルダーが含まれています。この機能は、
  JSON制御文字がエスケープされるように文字列入力をサニタイズすることによる、任意のプロンプト注入を禁止します。

* [AI Prompt Decorator](/hub/kong-inc/ai-prompt-decorator)プラグインは、 `llm/v1/chat`のメッセージの配列を
  発信者のチャット履歴の開始または終了に挿入します。この機能により、発信者は、
  {{site.base_gateway}} で呼び出されたときの大規模言語モデル \(LLM\) がどのように使用されるかについてより複雑なプロンプトを作成しより細かく制御できます。

#### リクエスト変換

KongのAI Gatewayを使用すると、AI技術を使用してその他のAPIトラフィックを拡張することもできます。1つの例として、クライアントに返す前にAI言語翻訳のプロンプトを介してAPIレスポンスをルーティングすることが挙げられます。KongのAI Gatewayは、AI機能をAPIリクエスト処理に組み入れるためのその他のアップストリームAPIサービスと併用できる2つのプラグインを提供します。これらのプラグインは、AI Proxyプラグインとは独立して構成できます。

* [AI Request Transformer](/hub/kong-inc/ai-request-transformer)プラグインは、設定されたLLMサービスを使用して、リクエストをアップストリームに
  プロキシする前に、コンシューマのリクエスト本文を変換し、イントロスペクトします。AI Proxyプラグインの機能を拡張し、すべてのAI Promptプラグインの後に
  実行し、異なるLLMに対するLLMリクエストをイントロスペクトすることができます。変換されたリクエストはバックエンドサービスに送信されます。
  LLMサービスがレスポンスを返すと、これがアップストリームのリクエスト本文として設定されます。

* [AI Response Transformer](/hub/kong-inc/ai-response-transformer)プラグインは、アップストリームからのHTTP応答を、構成済みのLLMサービスを使用してイントロスペクションと変換を完了してからクライアントに返信します。このプラグインはAI Proxyプラグインを補完するもので、別のLLMに照らしながらLLM応答のイントロスペクションを円滑化します。特に重要なのが、LLMからの指示に基づき、応答ヘッダー、応答ステータスコード、応答本文を調整する役割を果たすことであり、調整された応答は、クライアントに返送されます。

{% if_version gte:3.7.x %}

#### 流量制限

{:.badge .enterprise}

また、KongのAI Gatewayを使用すると、LLM APIへのトラフィックの管理も可能になります。KongのAI Gatewayは、AIリクエストトラフィックで流量制限を実装するために使用するAI Rate Limiting Advancedプラグインを提供します。

* [AI Rate Limiting Advanced](/hub/kong-inc/ai-rate-limiting-advanced) プラグインは LLM のレスポンスを調査して、トークンのコストを計算し、LLM バックエンドサービスの流量制限を有効にします。LLM サービスがレスポンスを返すと、これが流量制限を計算するためのコストとして使用されます。 分析の形式に関する詳細は、[AI 分析](/gateway/{{ page.release }}/production/logging/ai-analytics)を参照してください。

#### コンテンツの安全性とモデレーション

{:.badge .enterprise}

KongのAI Gatewayは、コンテンツをモデレートするためのメカニズムを提供します。

* 管理者は[Azure Content Safetyプラグイン](/hub/kong-inc/ai-azure-content-safety/)を使用することで、AI Proxyプラグインが扱うすべてのリクエストに対して[Azure Content Safety](https://azure.microsoft.com/en-us/products/ai-services/ai-content-safety)サービスを通じてイントロスペクションを強制できます。このプラグインではモデレーションカテゴリー別にしきい値を構成可能で、Azure Content Safetyインスタンスから事前構成済みのブロックリストIDの配列セットを指定できます。

{% endif_version %}

{% if_version gte:3.8.x %}

#### Semantic caching

{:.badge .enterprise}

KongのAIゲートウェイでは、セマンティックキャッシュを設定できます。

* The [AI Semantic Cache plugin](/hub/kong-inc/ai-semantic-cache/) allows you to semantically cache responses from LLMs. {% endif_version %}

{% if_version gte:3.7.x %}

### AIオブザーバビリティ

KongのAI Gatewayを使用すると、ロギングとメトリクスといった機能を通じてAIサービスを包括的に観察できるようになります。これらの機能を活用してAIの使用状況、パフォーマンス、コストに関するインサイトを取得することで、AIの運用を最適化し、効率的に監視できるようになります。

#### ログ記録

{:.badge .enterprise}

Kong の AI Gateway は、AI プラグインの標準化されたログ形式を提供し、さまざまなプロバイダー間で AI の使用状況を一貫して追跡および分析できるようにします。

詳細については、 [AI分析](/gateway/{{ page.release }}/production/logging/ai-analytics)を参照してください。{% endif_version %}

{% if_version gte:3.8.x %}

#### メトリクスとPrometheus

{:.badge .enterprise}

KongのAI GatewayではPrometheusとGrafanaを通じてAIメトリクスを開示、可視化できます。AIメトリクスの例として、AIリクエスト数、AIサービスの付随コスト、プロバイダーとモデルあたりのトークン使用量などを挙げることができます。メトリクスはPrometheusサーバーでスクレイピングし、Grafanaダッシュボードを使用して可視化できます。このセットアップではAIの運用状況がリアルタイムで表示されるため、パフォーマンスとコストを効果的に監視できます。

詳細については、「[AIメトリクス](/gateway/{{ page.release }}/production/monitoring/ai-metrics)」を参照してください。

{% endif_version %}

