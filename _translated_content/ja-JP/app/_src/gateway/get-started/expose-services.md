---
title: "Kong Gatewayでサービスを公開"
---
このトピックでは、ルートを使用してサービスを公開する方法を学習します。

スタートガイドのワークフローに従っている場合は、先に進む前に [{{site.base_gateway}} の管理の準備](/gateway/{{page.release}}/get-started//prepare)を完了していることを確認してください。

スタートガイドのワークフローに従っていない場合は、
{{site.base_gateway}} がインストール済みで、起動されていることを確認してください。

サービスおよびルートとは
------------

**Service** and **Route** objects let you expose your services to clients with

{{site.base_gateway}}. When configuring access to your API, you’ll start by specifying a
Service. In {{site.base_gateway}}, a Service is an entity representing an external
upstream API or microservice — for example, a data transformation
microservice, a billing API, and so on.

サービスの主な属性は、サービスがリクエストをリッスンする **URL** です。URLは単一の文字列で指定することも、プロトコル、ホスト、ポート、パスを個別に指定することもできます。

サービスに対してリクエストを開始する前に、ルートを追加する必要があります。ルートは、リクエストが{{site.base_gateway}}に到達した後にサービスに送信される方法（および送信されるかどうか）を決定します。1つのサービスに複数のルートを含めることができます。

After configuring the Service and the Route, you’ll be able to start making
requests through {{site.base_gateway}}.

この図は、サービスを通して、リクエストとレスポンスがバックエンドAPIにルーティングされるフローを示しています。

![サービスとルート](/assets/images/products/gateway/getting-started-guide/route-and-service.png)

サービスの追加
-------

この例では、httpbin APIを指すサービスを作成しています。
httpbinは、リクエストをリクエスト元にレスポンスとして返す「エコー」タイプの公開ウェブサイトです。この視覚化は、どのようにKong GatewayプロキシAPIリクエストの動作を学ぶのに役立ちます。


{{site.base_gateway}}はポート`8001`でRESTful Admin APIを公開します。ゲートウェイの
サービスやルートの追加を含む設定は、Admin APIへのリクエストを通じて
行われます。

```sh
curl -i -X POST http://localhost:8001/services \
  --data name=example_service \
  --data url='http://httpbin.org'
```

サービスが正常に作成されると、201 成功メッセージが表示されます。

サービスのエンドポイントを確認します。

```sh
curl -i http://localhost:8001/services/example_service
```

ルートの追加
------

APIゲートウェイ経由でサービスにアクセスできるようにするには、サービスへのルートを追加する必要があります。

クライアントが要求する必要がある特定のパスを使用して、サービス（`example_service`）のルート（`/mock`）を定義します。ルートをサービスと一致させるには、ホスト、パス、またはメソッドの少なくとも1つを設定する必要があります。

```sh
curl -i -X POST http://localhost:8001/services/example_service/routes \
  --data 'paths[]=/mock' \
  --data name=mocking
```

`201`メッセージは、ルートが正常に作成されたことを示します。

ルートがリクエストをサービスに転送していることを確認します。
------------------------------

By default, {{site.base_gateway}} handles proxy requests on port `8000`. The proxy is often referred to as the data plane.

```sh
curl -i -X GET http://localhost:8000/mock/anything
```

まとめと次の手順
--------

このセクションでは以下を実行しました。

* 名前が`example_service`でURLが`http://httpbin.org`のサービスを追加しました。
* `/mock`という名前のルートを追加しました。
* つまり、HTTPリクエストがポートポート`8000` （プロキシポート）の{{site.base_gateway}}ノードに送信され、、ルート`/mock`に一致する場合、そのリクエストは`http://httpbin.org`に送信されます。
* バックエンド/アップストリームサービスを抽象化し、好きなルートをフロントエンドに設定しました。このルートをクライアントに提供してリクエストを送信できるようになっています。 

次に、[流量制限の適用](/gateway/{{page.release}}/get-started//protect-services/)について学習します。

