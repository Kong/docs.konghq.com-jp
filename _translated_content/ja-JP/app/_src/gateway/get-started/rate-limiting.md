---
title: "Rate Limiting"
content-type: "tutorial"
book: "get-started"
chapter: 3
---
流量制限は、アップストリームサービスに送信されるリクエストレートを制御します。DoS攻撃を防止し、Webスクレイピングやその他の過剰使用を制限するために使用できます。流量制限を使用しない場合、クライアントはアップストリームのサービスに無制限にアクセスできますが、可用性に悪影響を及ぼす可能性があります。

Rate Limiting プラグイン
-------------------


{{site.base_gateway}}は、[Rate Limitingプラグイン](/hub/kong-inc/rate-limiting/)を使用してクライアントに流量制限を課します。
流量制限を有効にすると、クライアントは設定可能な期間内に実行できるリクエストの数が制限されます。プラグインはクライアントを[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)として識別するか、リクエストのクライアントIPアドレスによって識別することをサポートします。

{:.note}
> 
> このチュートリアルでは、[Rate Limiting](/hub/kong-inc/rate-limiting/) <span class="badge free"></span> プラグインを使用します。また、
> [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced)<span class="badge enterprise"></span>
> プラグインも使用可能です。上級バージョンでは、スライディングウィンドウアルゴリズム
> のサポートや高度な Redis サポートなど、より高いパフォーマンスのための追加機能が提供されます。

流量制限の管理
-------

次のチュートリアルでは、 {{site.base_gateway}}のさまざまな側面における流量制限の管理について説明します。

### 前提条件

この章は、「 *Get Started with Kong* 」シリーズの一部です。最適な効果を得るために、シリーズを最初からご覧になることをお勧めします。

まずは、導入として [Get Kong](/gateway/latest/get-started/) から始めましょう。ここには、ローカル {{site.base_gateway}} を実行するための前提のツールと手順が含まれています。

ガイドの手順2の[サービスとルート](/gateway/latest/get-started/services-and-routes/)には、このシリーズで使われているモックサービスのインストール方法が記載されています。

これらの手順をまだ完了していない場合は、続行する前に完了してください。

### グローバル流量制限

プラグインをグローバルにインストールすると、{{site.base_gateway}}への *すべて* のプロキシリクエストが流量制限の実行対象となります。

1. **流量制限を有効にする** 

   The rate limiting plugin is installed by default on {{site.base_gateway}}, and can be enabled
   by sending a `POST` request to the [plugins](/gateway/latest/admin-api/#add-plugin) object on the Admin API:

   ```sh
   curl -i -X POST http://localhost:8001/plugins \
     --data name=rate-limiting \
     --data config.minute=5 \
     --data config.policy=local
   ```

   このコマンドは、すべてのルートおよびサービスに対して、クライアントIPアドレスごとに 1 分あたり最大 5 件のリクエストを課すよう{{site.base_gateway}}に指示しました。

   `policy` 構成により、{{site.base_gateway}} 制限を取得し、増分させる場所が決まります。詳細については、すべての[プラグイン設定リファレンス](/hub/kong-inc/rate-limiting/#configuration)を参照してください。

   次のような識別情報を含む、新しいプラグイン構成を持つ応答が表示されます。

   ```text
   ...
   "id": "fc559a2d-ac80-4be8-8e43-cb705524be7f",
   "name": "rate-limiting",
   "enabled": true
   ...
   ```

2. **検証** 

   流量制限を設定したら、設定された制限時間内に許可される数よりも多くのリクエストを送信することで、
   正しく構成され機能していると確認できます。

{% capture global_instructions %}
{% navtabs %}
{% navtab Command Line %}

Run the following command to quickly send 6 mock requests:

```sh
for _ in {1..6}; do curl -s -i localhost:8000/mock/anything; echo; sleep 1; done
```

{% endnavtab %}
{% navtab Web browser %}

ブラウザで[http://localhost:8000/mock/anything](http://localhost:8000/mock/anything)を開き、1 分以内にページを6回更新します。

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ global_instructions | indent }}

6 回目のリクエストの後、429「API レート制限を超えました」エラーを受け取ります。

    {
       "message": "API rate limit exceeded"
    }

### サービスレベル流量制限

Rate Limitingプラグインは特定のサービスに対して有効にできます。リクエストは上記と同じです。
ただし、サービス URL に投稿されます。

```sh
curl -X POST http://localhost:8001/services/example_service/plugins \
   --data "name=rate-limiting" \
   --data config.minute=5 \
   --data config.policy=local
```

### ルートレベルの流量制限

Rate Limitingプラグインは特定のルートに対して有効にすることができます。リクエストは上記と同じですが、
ルートURLに投稿されます。

```sh
curl -X POST http://localhost:8001/routes/example_route/plugins \
   --data "name=rate-limiting" \
   --data config.minute=5 \
   --data config.policy=local
```

### コンシューマレベルの流量制限

[コンシューマ](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)とは{{site.base_gateway}}では、サービスのユーザーを定義する抽象的概念です。
コンシューマレベルの流量制限はコンシューマごとのリクエスト流量を制限するために使用できます。

1. **コンシューマを作成する** 

   コンシューマは、Admin APIの[コンシューマオブジェクト](/gateway/api/admin-ee/latest/#/Consumers/list-consumer/)を使用して作成されます。

   ```sh
   curl -X POST http://localhost:8001/consumers/ \
     --data username=jsmith
   ```

2. **コンシューマの流量制限を有効にする** 

   Using the consumer id, enable rate limiting for all routes and services for
   the `jsmith` consumer.

   ```sh
   curl -X POST http://localhost:8001/plugins \
      --data "name=rate-limiting" \
      --data "consumer.username=jsmith" \
      --data "config.second=5"
   ```

詳細な流量制限
-------

大規模な本番シナリオでは、効果的な流量制限には高度な技術が必要になる場合があります。上記の基本的な流量制限プラグインでは、固定時間ウィンドウに対する制限のみを定義できます。固定時間ウィンドウは多くの場合において十分ですが、デメリットもあります。

* 固定ウィンドウの境時間近辺にリクエストが殺到すると、トラフィックの急増中にウィンドウカウンターがリセットされるためリソースの逼迫につながる場合があります。 
* 複数のクライアントアプリケーションが、リクエストを再開するため 固定時間枠がリセットされるのを待っている可能性があります。固定ウィンドウがリセットされると、複数のクライアントによって システムにリクエストが殺到し、上流のサービスに多大な影響を与える可能性があります。

[Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)<span class="badge enterprise"></span>プラグインは、流量制限プラグインの拡張バージョンです。高度プラグインでは追加の制限アルゴリズム機能の使用が可能になり、基本的なプラグインと比較して優れたパフォーマンスが実現します。高度な流量制限アルゴリズムの詳細については、[Kong APIで拡張可能な流量制限アルゴリズムを設計する方法](https://konghq.com/blog/how-to-design-a-scalable-rate-limiting-algorithm)を参照してください。。

