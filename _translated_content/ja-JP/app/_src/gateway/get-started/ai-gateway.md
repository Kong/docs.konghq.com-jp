---
title: "AI Gatewayを使ってみる"
content-type: "tutorial"
book: "get-started"
chapter: 7
---
[AI Gateway](/gateway/{{page.release}}/ai-gateway/)は{{site.base_gateway}}の標準プラグインモデルを使用して構築されています。AIプラグインはバージョン3\.6\.xでは{{site.base_gateway}}にバンドルされています。これは、[各プラグイン](/hub/?category=ai)の文書化された構成指示に従うことにより、AI Gateway機能をデプロイできることを意味します。

すぐに開始できるように、AI Gatewayとして構成された{{site.base_gateway}}をDockerコンテナにデプロイするタスクを自動化するスクリプトを用意しました。これにより、AIプラグインの完全なスイートを探索する前に、コアAIプロバイダーの機能を評価できます。

クイックスタートスクリプトでは {{site.konnect_product_name}} 上での {{site.base_gateway}} のデプロイがサポートされ、従来のモードでは Docker 上でのデプロイのみがサポートされます。

{:.note}
> 
> **注:**
> このスクリプトを実行すると、ホストされている
> AI プロバイダーとの認証を設定するために使用する、AI プロバイダーの API キーの入力を求められます。これらのキーは {{site.base_gateway}} Docker コンテナにのみ渡され、
> それ以外はホストマシンの外部には送信されません。

1. インタラクティブな`ai`クイックスタートスクリプトを使用して、AI Gatewayを実行します。

   ```sh
   curl -Ls https://get.konghq.com/ai | bash
   ```

2. {{site.konnect_product_name}}またはDockerのみを対象とするデプロイメントを構成し、必要なAIプロバイダーのAPIキーを構成します。

   ```sh
   ...
   Do you want to deploy on Kong Konnect (y/n) y
   ...
   Provide a key for each provider you want to enable and press Enter.
   Not providing a key and pressing Enter will skip configuring that provider.
   
   → Enter API Key for OpenAI    :
   ...
   ```

3. AI Gatewayがデプロイされると、AIプロキシプラグインの動作評価に役立つ情報が表示されます。たとえば、このスクリプトは、OpenAIへのチャットリクエストをルーティングするのに役立つサンプルコマンドを出力します。

   ```sh
   =======================================================
    ⚒️                                 Routing LLM Requests
   =======================================================
   
   Your AI Gateway is ready. You can now route AI requests
   to the configured AI providers. For example to route a
   chat request to OpenAI you can use the following
   curl command:
   
   curl -s -X POST localhost:8000/openai/chat \
    -H "Content-Type: application/json" -d '{
      "messages": [{
      "role": "user",
      "content": "What is Kong Gateway?"
    }] }'
   ```

4. 最後に、スクリプトは自身が生成するデプロイファイルに関する情報を表示します。この情報は、今後の AI Gateway 構成に使用できます。

   ```sh
   =======================================================
    ⚒️                What is next with the Kong AI Gateway
   =======================================================
   
   This script demonstrated the installation and usage
   of only one of the many AI plugins that Kong Gateway
   provides (the 'ai-proxy' plugin).
   
   See the output directory to reference the files
   used during the installation process and modify for
   your production deployment.
   ℹ /tmp/kong/ai-gateway
   ```

{:.note}
> 
> **注：**
> デフォルトでは、ローカルモデルはエンドポイント`http://host.docker.internal:11434`で構成されます。
> これにより、Dockerで実行されている{{site.base_gateway}}ホストマシンに接続できるようになります。

その他の情報
------

* [AI Gatewayの詳細を見る](/gateway/{{page.release}}/ai-gateway/)
* [AI Gatewayプラグイン](/hub/?category=ai)

