---
title: "AI 監査ログのリファレンス"
content_type: "reference"
---
Kong AI Gatewayは、AIプラグイン用の標準化されたログ形式を提供し、分析イベントの発行を可能にし、さまざまなプロバイダーにわたるAI使用状況分析の集計を容易にします。

ログ形式
----

各AIプラグインは、トークンのセットを返します。

すべてのログエントリには、次の属性が含まれます。

{% if_version lte:3.7.x %}

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

{% endif_version %}
{% if_version gte:3.8.x %}

```json
"ai": {
    "payload": { "request": "[$optional_payload_request_]" },
    "[$plugin_name_1]": {
      "payload": { "response": "[$optional_payload_response]" },
      "usage": {
        "prompt_token": 28,
        "total_tokens": 48,
        "completion_token": 20,
        "cost": 0.0038,
        "time_per_token": 133
      },
      "meta": {
        "request_model": "command",
        "provider_name": "cohere",
        "response_model": "command",
        "plugin_id": "546c3856-24b3-469a-bd6c-f6083babd2cd",
        "llm_latency": 2670
      }
    },
    "[$plugin_name_2]": {
      "payload": { "response": "[$optional_payload_response]" },
      "usage": {
        "prompt_token": 89,
        "total_tokens": 145,
        "completion_token": 56,
        "cost": 0.0012,
        "time_per_token": 87
      },
      "meta": {
        "request_model": "gpt-35-turbo",
        "provider_name": "azure",
        "response_model": "gpt-35-turbo",
        "plugin_id": "5df193be-47a3-4f1b-8c37-37e31af0568b",
        "llm_latency": 4927
      }
    }
  }
```

{% endif_version %}

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

{% if_version gte:3.8.x %}
\| `ai.[$plugin_name].usage.time_per_token` \| ミリ秒単位で出力トークンを生成する平均時間。 \|
{% endif_version %}

\| `ai.[$plugin_name].meta.request_model` \| AIリクエストに使用されるモデル。\|
\| `ai.[$plugin_name].meta.provider_name` \|  AI サービスプロバイダーの名前。 \|
\| `ai.[$plugin_name].meta.response_model` \| AI応答に使用されるモデル。 \|
\| `ai.[$plugin_name].meta.plugin_id` \| プラグインの識別子. \|

{% if_version gte:3.8.x %}
\| `ai.[$plugin_name].meta.llm_latency` \| 時間、ミリ秒単位で、LLMプロバイダが完全な応答を生成するのに要しました \|
\| `ai.[$plugin_name].cache.cache_status` \| キャッシュの状態。 これはヒット、ミス、バイパス、または更新することができます. \|
\| `ai.[$plugin_name].cache.fetch_latency` \| 時間、ミリ秒単位でキャッシュ応答を返します。 \|
\| `ai.[$plugin_name].cache.embeddings_provider` \| セマンティックキャッシュでは、埋め込みの生成に使用されるプロバイダです。 \|
\| `ai.[$plugin_name].cache.embeddings_model` \|  セマンティックキャッシュでは、埋め込みを生成するために使用されるモデルです。 \|
\| `ai.[$plugin_name].cache.embeddings_latency` \|  セマンティックキャッシュの場合、埋め込みを生成するのにかかる時間です \|
{% endif_version %}

{% if_version gte:3.8.x %}

### Caches logging

If you're using the [AI Semantic Cache plugin](/hub/kong-inc/), logging will include some additional details about caching:

```json
"ai": {
    "payload": { "request": "[$optional_payload_request_]" },
    "[$plugin_name_1]": {
      "payload": { "response": "[$optional_payload_response]" },
      "usage": {
        "prompt_token": 28,
        "total_tokens": 48,
        "completion_token": 20,
        "cost": 0.0038,
        "time_per_token": 133
      },
      "meta": {
        "request_model": "command",
        "provider_name": "cohere",
        "response_model": "command",
        "plugin_id": "546c3856-24b3-469a-bd6c-f6083babd2cd",
        "llm_latency": 2670
      },
      "cache": {
        "cache_status": "Hit",
        "fetch_latency": 21
      }
    },
    "[$plugin_name_2]": {
      "payload": { "response": "[$optional_payload_response]" },
      "usage": {
        "prompt_token": 89,
        "total_tokens": 145,
        "completion_token": 56,
        "cost": 0.0012,
      },
      "meta": {
        "request_model": "gpt-35-turbo",
        "provider_name": "azure",
        "response_model": "gpt-35-turbo",
        "plugin_id": "5df193be-47a3-4f1b-8c37-37e31af0568b",
      },
      "cache": {
        "cache_status": "Hit",
        "fetch_latency": 444,
        "embeddings_provider": "openai",
        "embeddings_model": "text-embedding-3-small",
        "embeddings_latency": 424
      }
    }
  }
```

{:.note}
> 
> **注意:**
> キャッシュ応答を返す場合、 `time_per_token` と `llm_latency` は省略されます。
> キャッシュ応答はセマンティックキャッシュまたは正確なキャッシュとして返すことができます。 セマンティックキャッシュとして返される場合、埋め込みプロバイダ、埋め込みモデル、埋め込みレイテンシーなどの追加の詳細が含まれます
> {% endif_version %}

