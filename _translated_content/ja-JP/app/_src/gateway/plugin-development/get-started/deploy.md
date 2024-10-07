---
title: "プラグインのデプロイ"
book: "plugin_dev_getstarted"
chapter: 6
---
機能的なカスタムプラグインを作成したら、それを使用するために
{{site.base_gateway}}にデプロイする必要があります。カスタムプラグインをデプロイする方法は複数あり、どの方法を選択するかは、特定の{{site.base_gateway}}環境やその他のテクノロジーの選択に大きく依存します。このセクションでは、一般的なデプロイオプションについて説明し、詳細な手順の参考資料を提供します。

前提条件
----

このページは、カスタムプラグインの開発用[導入](/gateway/{{page.release}}/plugin-development/get-started/index)
ガイドの第5章です。これらの手順はガイドの前の章を参照しており、同じ開発者ツールの前提条件が必然的に適用されます。

段階的な手順
------

{{site.base_gateway}}を実行するための非常に一般的な選択肢は、コンテナランタイムシステムの使用です。Kongは、デプロイメントで使用するためにDockerイメージを構築および検証し、Dockerデプロイメントで[詳細な手順](/gateway/{{page.release}}/install/docker/)を提供します。カスタムプラグインを{{site.base_gateway}}コンテナで使用するためのいくつかのオプションをご紹介します。

一般的なオプションの1つは、カスタムプラグインコードを追加して、必要な環境をイメージに直接設定するDockerイメージの構築です。このソリューションでは、デプロイメントパイプラインの構築段階でより多くのステップを実行する必要がありますが、構成としてのデータプレーン（DP）デプロイメントが簡素化され、カスタムコードはデータプレーン（DP）イメージに直接送信されます。

### 次を作成 `Dockerfile`

`my-plugin`プロジェクトのルートに`Dockerfile`という名前の新しいファイルを作成し、次の内容でカスタムイメージを実行します。

```dockerfile
{% if page.release.label == "unreleased" -%}
FROM kong/kong-gateway-dev:latest
{% else -%}
FROM kong/kong-gateway:{{page.release.tag}}
{% endif -%}

# Ensure any patching steps are executed as root user
USER root

# Add custom plugin to the image
COPY ./kong/plugins/my-plugin /usr/local/share/lua/5.1/kong/plugins/my-plugin
ENV KONG_PLUGINS=bundled,my-plugin

# Ensure kong user is selected for image execution
USER kong

# Run kong
ENTRYPOINT ["/entrypoint.sh"]
EXPOSE 8000 8443 8001 8444
STOPSIGNAL SIGQUIT
HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health
CMD ["kong", "docker-start"]
```

### Docker イメージのビルド

Dockerイメージをビルドするときは、識別するためのタグを付けます。{{site.base_gateway}}バージョンとプラグインのバージョンを指定する情報を画像にタグ付けすることをお勧めします。例えば、以下のようにするとカスタムイメージが構築され、バージョンの`my-plugin`の部分が`0.0.1`としてラベル付けされます。

```sh
docker build -t kong-gateway_my-plugin:{{ page.release.tag }}-0.0.1 .
```

### カスタムイメージを実行する

これで、{{site.base_gateway}}クイックスタートスクリプトを使用してカスタムイメージを実行し、プラグインをさらにテストできるようになりました。クイックスタートスクリプトでは、Docker リポジトリ（`-r`）、イメージ（`-i`）、タグ（`-t`）をオーバーライドできるフラグがサポートされています。コマンドの例を次に示します。

```sh
curl -Ls https://get.konghq.com/quickstart | \
  bash -s -- -r "" -i kong-gateway_my-plugin -t {{ page.release.tag }}-0.0.1
```

### デプロイされた`my-plugin`プラグインをテストする

カスタムイメージで{{site.base_gateway}}が実行されるようになったら、プラグインを手動でテストして動作を検証できます。

テストサービスを追加します。

```sh
curl -i -s -X POST http://localhost:8001/services \
    --data name=example_service \
    --data url='http://httpbin.org'
```

カスタムプラグインを`example_service`サービスに関連付けます。

```sh
curl -is -X POST http://localhost:8001/services/example_service/plugins \
    --data 'name=my-plugin'
```

リクエストを送信するために使用できる新しいルートを追加します。

```sh
curl -i -X POST http://localhost:8001/services/example_service/routes \
    --data 'paths[]=/mock' \
    --data name=example_route
```

リクエストをテストルートにプロキシして、`curl`に`-i`フラグの付いたレスポンスヘッダーを表示させることで動作をテストします。

```sh
curl -i http://localhost:8000/mock/anything
```

`curl` は `HTTP/1.1 200 OK` を報告し、ゲートウェイからのレスポンスヘッダーを表示します。ヘッダーのセットには、次を含める必要があります。`X-MyPlugin`

以下に例を示します。

```sh
HTTP/1.1 200 OK
Content-Type: application/json
Connection: keep-alive
Content-Length: 529
Access-Control-Allow-Credentials: true
Date: Tue, 12 Mar 2024 14:44:22 GMT
Access-Control-Allow-Origin: *
Server: gunicorn/19.9.0
X-MyPlugin: http://httpbin.org/anything
X-Kong-Upstream-Latency: 97
X-Kong-Proxy-Latency: 1
Via: kong/3.6.1
X-Kong-Request-Id: 8ab8c32c4782536592994514b6dadf55
```

その他のデプロイオプション
-------------

カスタムプラグインのデプロイオプションは、カスタムDockerイメージの構築以外にも次のようなものがあります。

### Kubernetesのデプロイメント

多くのユーザーは、Kubernetes上で{{site.base_gateway}}を実行することを選択します。{{site.base_gateway}} は、Kubernetesに直接デプロイすることも、[{{site.kic_product_name}}](/kubernetes-ingress-controller/latest/)を使用してデプロイすることもできます。どちらの場合でも、Kubernetesへのカスタムプラグインのデプロイは、カスタムプラグインコードをConfigMapまたはSecretのクラスタに追加し、{{site.base_gateway}}プロキシポッドにマウントすることで実行されます。さらに、ポッドは、マウントされたボリュームからカスタムプラグインコードをロードするように設定する必要があります。

Kongは、構成したプラグインに基づいたプロキシポッドが必要とする環境を構成することで、このプロセスを簡素化するHelmチャートを提供します。Helm以外のデプロイメントの場合は、環境を直接変更する必要があります。

[カスタムプラグイン](/kubernetes-ingress-controller/latest/plugins/custom/)のドキュメントページには、
[Kongヘルムチャート](/kubernetes-ingress-controller/latest/install/helm/)
または標準のKubernetesマニフェストを直接使用してカスタムプラグインを
デプロイする手順が記載されています。

### パッケージパスの上書き

{{site.base_gateway}}をベアメタルまたは仮想マシン環境で実行する場合に、Lua VMがパッケージを
探すロケーションを上書きするのは、カスタムプラグインのデプロイのための一般的なストラテジです。
上記と同じファイル構造に従って、ホストマシンにソースファイルを配布し、
`lua_package_path`構成値をこのパスを指すように変更することができます。
この構成は、 `KONG_LUA_PACKAGE_PATH`環境変数を使用して変更することもできます。

このオプションの詳細については、カスタムプラグインの[インストールドキュメント](/gateway/{{page.release}}/plugin-development/distribution/)を参照してください。

{:.note}
> 
> **注** : このストラテジは、ベアメタルまたは仮想マシン環境に加えて、コンテナ化されたシステムでのボリュームマウントにも使用できます。

### LuaRocksパッケージの構築

LuaRocksはLuaモジュール用のパッケージマネージャーです。これにより、Lua モジュールをrockと呼ばれる自己完結型のパッケージとして作成およびインストールできます。 *rock* パッケージを作成するには、パッケージに関するさまざまな情報を指定する *rockspec* ファイルを作成します。`luarocks`ツールを使用して、rockspec ファイルからアーカイブを作成し、それをデータプレーン（DP）に配信して抽出します。

この配布オプションの詳細については、カスタムプラグインのインストールページにある[ソースのパッケージ](/gateway/{{page.release}}/plugin-development/distribution/#packaging-sources)セクションを参照してください。

{{site.konnect_product_name}} のカスタムプラグイン
-----------------

[{{site.konnect_product_name}}](/konnect/)は、サービスとしてのKongの統合APIプラットフォームです。{{site.konnect_short_name}}は、いくつかの制限付きでカスタムプラグインをサポートします。オンプレミスのデプロイでは、ユーザーが{{site.base_gateway}}データプレーン（DP）、コントロールプレーン（CP）、オプションでバッキングデータベースを管理します。{{site.konnect_product_name}}でコントロールプレーンとデータベースが完全に管理されているため、カスタムデータエンティティのサポートとプラグインのAdmin API拡張が制限されます。


{{site.konnect_short_name}} ドキュメントの[プラグインの管理](/konnect/gateway-manager/plugins/)には、{{site.konnect_product_name}} へのカスタムプラグインのデプロイに関する詳細情報が記載されています。

次の手順
----

カスタムプラグインの構築は {{site.base_gateway}} の動作を拡張し、APIゲートウェイレイヤーでビジネスロジックを有効にするための優れた方法です。ただし、カスタムプラグイン開発は最後の手段として使用するのが最適です。{{site.base_gateway}} は、多くの一般的なユースケースをカバーする多数のビルド済みプラグインをサポートしており、その中には、カスタムビジネスロジックプログラミングを、ビルド済みでサポートされているプラグインに直接組み込むことができる[サーバーレス](/hub/?category=serverless)プラグインも含まれています。サポートされているプラグインの完全なカタログについては、Kong [Plugin Hub](/hub/) を参照してください。

{{site.base_gateway}}を実行するための最適な方法は、Kongのサービスとしての統合APIプラットフォーム{{site.konnect_product_name}}を使用することです。{{site.konnect_product_name}}は、[ログイン](https://cloud.konghq.com/login)または[登録](https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=custom-plugins)すればすぐに無料で始められます。

