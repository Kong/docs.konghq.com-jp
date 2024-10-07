---
title: "AI 監査ログのリファレンス"
content_type: "reference"
---
Kong AI Gatewayは、AIプラグイン用の標準化されたログ形式を提供し、分析イベントの発行を可能にし、さまざまなプロバイダーにわたるAI使用状況分析の集計を容易にします。

ログ形式
----

各AIプラグインは、トークンのセットを返します。

すべてのログエントリには、次の属性が含まれます。

```json
"ai": {
    "payload": { "request": "[$optional_payload_request_]" },
    "[$plugin_name_1]": {
      "payload": { "response": "[$optional_payload_response]" },
      "usage": {
        "prompt_token": 28,
        "total_tokens": 48,
        "completion_token": 20,
        "cost": 0.0038
      },
      "meta": {
        "request_model": "command",
        "provider_name": "cohere",
        "response_model": "command",
        "plugin_id": "546c3856-24b3-469a-bd6c-f6083babd2cd"
      }
    },
    "[$plugin_name_2]": {
      "payload": { "response": "[$optional_payload_response]" },
      "usage": {
        "prompt_token": 89,
        "total_tokens": 145,
        "completion_token": 56,
        "cost": 0.0012
      },
      "meta": {
        "request_model": "gpt-35-turbo",
        "provider_name": "azure",
        "response_model": "gpt-35-turbo",
        "plugin_id": "5df193be-47a3-4f1b-8c37-37e31af0568b"
      }
    }
  }

```

### ログの詳細

各ログエントリには、以下の詳細が含まれます。

|                   プロパティ                    |            説明             |
|--------------------------------------------|---------------------------|
| `ai.payload.request`                       | リクエストペイロード。               |
| `ai.[$plugin_name].payload.response`       | 応答ペイロード。                  |
| `ai.[$plugin_name].usage.prompt_token`     | プロンプトに使用されたトークンの数。        |
| `ai.[$plugin_name].usage.completion_token` | 完了に使用されたトークンの数。           |
| `ai.[$plugin_name].usage.total_tokens`     | 使用されたトークンの合計数。            |
| `ai.[$plugin_name].usage.cost`             | リクエストの合計コスト（入力コストと出力コスト）。 |
| `ai.[$plugin_name].meta.request_model`     | AI リクエストに使用されるモデル。        |
| `ai.[$plugin_name].meta.provider_name`     | AIサービスプロバイダーの名前。          |
| `ai.[$plugin_name].meta.response_model`    | AIレスポンスに使用されるモデル。         |
| `ai.[$plugin_name].meta.plugin_id`         | プラグインの一意の識別子。             |

