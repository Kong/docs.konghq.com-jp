---
title: "コンシューマ"
concept_type: "参照"
---
コンシューマとは何ですか?
-------------

コンシューマーは通常、 {{site.base_gateway}}によって管理される API を消費または使用するエンティティを指します。コンシューマは、アプリケーション、サービス、またはAPIとやり取りするユーザーです。彼らは必ずしも人間ではないため、 {{site.base_gateway}}は、サービスを「消費する」存在として彼らをコンシューマと呼びます。
{{site.base_gateway}}を使用すると、コンシューマを定義および管理し、アクセス制御ポリシーを適用し、APIの使用状況を監視できます。

APIへのアクセスを制御し、使用状況を追跡し、セキュリティを確保するには、コンシューマが不可欠です。
これらは、Key Authentication、OAuth、またはその他の認証および承認メカニズムによって識別されます。たとえば、サービスまたはルートにBasic Authプラグインを追加すると、コンシューマを識別したり、認証情報が無効な場合はアクセスをブロックしたりできるようになります。

{{site.base_gateway}}と既存のプライマリデータストア間の一貫性を保つには、{{site.base_gateway}}をコンシューマのプライマリデータストアとして使用するか、
既存のデータベースにコンシューマリストをマップすることができます。

プラグインをコンシューマに直接添付することで、レート制限など、コンシューマレベルで特定の制御を管理できます。

{% mermaid %}
flowchart LR

A(Consumer entity)
B(Auth plugin)
C[Upstream service]

Client --> A
subgraph id1[{{site.base_gateway}}]
direction LR
A--Credentials-->B
end

B-->C
{% endmermaid %}

コンシューマ向けのユースケース
---------------

以下は、コンシューマの一般的なユースケースの例です。

|   ユースケース   |                                  説明                                  |
|------------|----------------------------------------------------------------------|
| 認証         | クライアント認証は、コンシューマを設定する主な理由です。認証プラグインを使用している場合は、認証情報を持つコンシューマが必要になります。 |
| コンシューマグループ | 基準セットごとにコンシューマをグループ化し、特定のルールを適用します。                                  |
| 流量制限       | 階層に基づいて特定のコンシューマの流量制限をします。                                           |

コンシューマの管理
---------

{% navtabs %}
{% navtab Kong Admin API %}

コンシューマを作成するには、[Admin APIの`/consumers`エンドポイント](/gateway/api/admin-ee/latest/#/Consumers)を呼び出します。
次のようにすると、 **example\-consumer** と呼ばれるコンシューマが作成されます。

```sh
curl -i -X POST http://localhost:8001/consumers/ \
  --data username=example-consumer \
  --data custom_id=example-consumer-id \
  --data tags[]=silver-tier
```

{% endnavtab %}
{% navtab Konnect API %}

コンシューマを作成するには、{{site.konnect_short_name}}[コントロールプレーン（CP）設定APIの`/consumers`エンドポイント](/konnect/api/control-plane-configuration/latest/#/Consumers)を呼び出します。
次のようにすると、 **example\-consumer** と呼ばれるコンシューマが作成されます。

```sh
curl -X POST https://{us|eu}.api.konghq.com/v2/control-planes/{controlPlaneId}/core-entities/consumers \
  --data '{
    "custom_id":"example-consumer-id",
    "tags":["silver-tier"],
    "username":"example-consumer"
    }'
```

{% endnavtab %}
{% navtab decK (YAML) %}

次のようにすると、 **example\-consumer** と呼ばれるコンシューマが作成されます。

```yaml
_format_version: "3.0"
consumers:
- custom_id: example-consumer-id
  username: example-consumer
  tags:
    - silver-tier
```

{% endnavtab %}
{% navtab KIC (YAML) %}

次のようにすると、 **example\-consumer** と呼ばれるコンシューマが作成されます。

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
 name: example-consumer
 annotations:
   kubernetes.io/ingress.class: kong
username: example-consumer
```

{% endnavtab %}
{% navtab Kong Manager or Gateway Manager %}

次のようにすると、 **example\-consumer** と呼ばれる新しいコンシューマが作成されます。

1. Kong ManagerまたはGateway Managerで、 **APIゲートウェイ** > **コンシューマ** の順にアクセスします。
2. **新しいコンシューマ** をクリックします。
3. **ユーザー名** と **カスタムID** を入力します。
4. **作成** をクリックします。

{% endnavtab %}
{% endnavtabs %}

### 次の手順

コンシューマごとに認証情報が必要です。各認証方式に固有の手順については、[認証プラグイン](/hub/?category=authentication)を参照してください。プラグインを開き、 **Using this plugin** > **Basic examples** に移動し、目的のツールを選択して指示に従います。

よくある質問
------

<details><summary>認証情報とは何ですか、なぜ必要なのですか?</summary>認証情報は、さまざまな認証メカニズムを介してコンシューマーを認証するために必要です。認証情報の種類は、使用する認証プラグインによって異なります。 <br><br> たとえば、Key AuthenticationプラグインにはAPIキーが必要であり、Basic Authプラグインにはユーザー名とパスワードのペアが必要です。</details> <details><summary>コンシューマとアプリケーションの違いは何ですか？</summary> 

アプリケーションは、開発者に対して{{site.base_gateway}}または{{site.konnect_short_name}}によって管理されるAPIへのアクセス機能を提供するもので、
必要な認証情報を生成するためのKongの管理チームからのやり取りは必要ありません。
<br><br>
コンシューマに関しては、Kongのチームはコンシューマを作り、認証情報を生成して、それをAPIにアクセスする必要がある開発者と共有する必要があります。
アプリケーションは、開発者が必要なAPIの認証情報を自動的に取得し、サブスクライブできるようにするKongの一種のコンシューマと考えることができます。
</details> <details><summary>コンシューマと開発者の違いは何ですか？</summary> 

開発者はDev Portalの実際のユーザーですが、コンシューマは抽象的概念です。
</details> <details><summary>コンシューマとRBACユーザーの違いは何ですか？</summary> 

RBAC ユーザーは Kong Manager のユーザーですが、コンシューマはゲートウェイ自体のユーザー（実ユーザーまたは抽象ユーザー）です。
</details> <details><summary>コンシューマにスコープできるプラグインとは？</summary> 

特定のプラグインは、サービス、ルート、またはグローバルではなく、コンシューマにスコープ設定できます。例えば、
特定のコンシューマに流量制限をかけるには、Rate Limitingプラグインを設定するか、Request Transformer プラグインを使用してそのコンシューマのリクエストを編集できます。
リスト一覧は、<a href="/hub/plugins/compatibility/#scopes">プラグインスコープの互換性リファレンス</a>をご覧ください。
</details> <details><summary>認証プラグインをコンシューマにスコープできますか？</summary> 

いいえ。認証情報を構成すると認証プラグインとコンシューマを関連付けることができます。たとえば、基本的な認証情報を持つコンシューマはBasic Authプラグインを使用します。
ただしそのプラグインは、ルート、サービス、またはグローバルのいずれかコンシューマがアクセス可能なものをスコープとする必要があります。
</details> <details><summary>コンシューマは Kuma/Mesh で使用されますか?</summary> 

いいえ。
</details> <details><summary>decKでコンシューマを管理できますか？</summary> 

はい、decKを使ってコンシューマを管理できますが、コンシューマの数が多い場合は注意してください。
<br><br>
データベース内に多数のコンシューマが存在する場合は、decK を使用してそれらをエクスポートまたは管理しないでください。decK はエンティティ構成を管理するために構築されています。エンドユーザーのデータ用ではありません。
レコード数は簡単に数十万、数百万にまで増える可能性があります。
</details> 

関連リンク
-----

* [認証に関する参考資料](/gateway/latest/kong-plugins/authentication/reference/)
* [コンシューマAPIリファレンス \- {{site.base_gateway}}](/gateway/api/admin-ee/latest/#/Consumers)
* [コンシューマAPIリファレンス \- {{site.konnect_short_name}}](/konnect/api/control-plane-configuration/latest/#/Consumers)
* [コンシューマグループAPI参照](/gateway/api/admin-ee/latest/#/consumer_groups)
* [コンシューマ側で有効化できるプラグイン](/hub/plugins/compatibility/#scopes)

