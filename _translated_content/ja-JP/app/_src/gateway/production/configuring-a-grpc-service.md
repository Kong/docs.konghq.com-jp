---
title: "gRPCサービスの設定"
content_type: "how-to"
---
{:.note}
> 
> 注：このガイドは、gRPCに精通していることを前提としています。アップストリームREST APIを
> 使用するKongのセットアップ方法については、[サービス設定のガイド](/gateway/{{page.release}}/get-started/services-and-routes)を参照してください。

gRPC プロキシは Kong でネイティブにサポートされています。このガイドでは、gRPC サービスを管理するための Kong の構成方法を説明します。このガイドでは、[`grpcurl`](https://github.com/fullstorydev/grpcurl) と [`grpcbin`](https://github.com/moul/grpcbin) を使用します。これらは、それぞれ gRPC クライアントと gRPC サービスを提供します。

このガイドでは、次の2つの例を設定します。

* 一致するすべてのgRPCトラフィックをアップストリームの gRPCサービスにプロキシする単一のキャッチオールルートを含む、単一のgRPCサービスとルート。
* 複数のルートを持つ単一のgRPCサービス。gRPCメソッドごとにルートを使用する方法を示します。

Kong では、gRPC サポートは HTTP/2 フレーミングを介した gRPC を想定しています。[HTTP/2プロキシリスナー](/gateway/{{page.release}}/reference/configuration/#proxy_listen)が
少なくとも1つあることを確認してください。

以下の例では、Kongが実行され、ポート9080でHTTP/2プロキシリクエストをリッスンしていることを前提としています。

単一のgRPCサービスとルート
---------------

1. gRPCサービスを作成するには、次のリクエストを発行します。たとえば、gRPCサーバーが`localhost`をリッスンしている場合は、ポート`15002`：

   ```bash
   curl -XPOST localhost:8001/services \
     --data name=grpc \
     --data protocol=grpc \
     --data host=localhost \
     --data port=15002
   ```

2. 次のリクエストを発行して、gRPCルートを作成します。

   ```bash
   curl -XPOST localhost:8001/services/grpc/routes \
     --data protocols=grpc \
     --data name=catch-all \
     --data paths=/
   ```

3. [`grpcurl`](https://github.com/fullstorydev/grpcurl)コマンドラインクライアントを使用して、次のgRPCリクエストを発行します。

   ```bash
   grpcurl -v -d '{"greeting": "Kong!"}' \
     -plaintext localhost:9080 hello.HelloService.SayHello
   ```

   レスポンスは次のようになります。

       Resolved method descriptor:
       rpc SayHello ( .hello.HelloRequest ) returns ( .hello.HelloResponse );
       
       Request metadata to send:
       (empty)
       
       Response headers received:
       content-type: application/grpc
       date: Tue, 16 Jul 2019 21:37:36 GMT
       server: openresty/1.15.8.1
       via: kong/1.2.1
       x-kong-proxy-latency: 0
       x-kong-upstream-latency: 0
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response trailers received:
       (empty)
       Sent 1 request and received 1 response

`via`や`x-kong-proxy-latency`のようなKong応答ヘッダーが、応答に挿入されているのがわかります。

複数のルートを持つ単一のgRPCサービス
--------------------

前の例を基にして、各gRPCメソッドにさらにいくつかのルートを作成しましょう。

この例では、gRPC `HelloService`サービスは、[プロトコル バッファー ファイル](https://raw.githubusercontent.com/moul/pb/master/hello/hello.proto)で確認できるようないくつかの異なる
メソッドを公開します。

1. `SayHello` メソッドと `LotsOfReplies` メソッドの個別のルートを作成します。

   1. `SayHello`へのルートを作成します。

      ```bash
      curl -X POST localhost:8001/services/grpc/routes \
        --data protocols=grpc \
        --data paths=/hello.HelloService/SayHello \
        --data name=say-hello
      ```

   2. `LotsOfReplies`へのルートを作成します。

      ```bash
      curl -X POST localhost:8001/services/grpc/routes \
        --data protocols=grpc \
        --data paths=/hello.HelloService/LotsOfReplies \
        --data name=lots-of-replies
      ```

   この設定では、`SayHello`メソッドへのgRPCリクエストは、最初のルートに一致し、`LotsOfReplies`へのリクエストは後のルートにルーティングされます。

{% if_version gte:3.2.x %}

1. `kong.conf`で、[`allow_debug_header: on`](/gateway/{{page.release}}/reference/configuration/#allow_debug_header)を設定します。
   {% endif_version %}

2. `SayHello`メソッドにgRPCリクエストを発行します。

   ```bash
   grpcurl -v -d '{"greeting": "Kong!"}' \
     -H 'kong-debug: 1' -plaintext \
     localhost:9080 hello.HelloService.SayHello
   ```

   この例ではヘッダー`kong-debug`が送信されたことで、Kongが
   応答ヘッダー内にデバッグ情報を挿入することに注意してください。

   応答は次のようになります。

       Resolved method descriptor:
       rpc SayHello ( .hello.HelloRequest ) returns ( .hello.HelloResponse );
       
       Request metadata to send:
       kong-debug: 1
       
       Response headers received:
       content-type: application/grpc
       date: Tue, 16 Jul 2019 21:57:00 GMT
       kong-route-id: 390ef3d1-d092-4401-99ca-0b4e42453d97
       kong-service-id: d82736b7-a4fd-4530-b575-c68d94c3493a
       kong-service-name: s1
       server: openresty/1.15.8.1
       via: kong/1.2.1
       x-kong-proxy-latency: 0
       x-kong-upstream-latency: 0
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response trailers received:
       (empty)
       Sent 1 request and received 1 response

   ルートIDは、最初に作成したルートを参照する必要があることに注意してください。
3. `LotsOfReplies` gRPCメソッドにも同様にリクエストを発行してみましょう。

   ```bash
   grpcurl -v -d '{"greeting": "Kong!"}' \
     -H 'kong-debug: 1' -plaintext \
     localhost:9080 hello.HelloService.LotsOfReplies
   ```

   応答は次のようになります。

       Resolved method descriptor:
       rpc LotsOfReplies ( .hello.HelloRequest ) returns ( stream .hello.HelloResponse );
       
       Request metadata to send:
       kong-debug: 1
       
       Response headers received:
       content-type: application/grpc
       date: Tue, 30 Jul 2019 22:21:40 GMT
       kong-route-id: 133659bb-7e88-4ac5-b177-bc04b3974c87
       kong-service-id: 31a87674-f984-4f75-8abc-85da478e204f
       kong-service-name: grpc
       server: openresty/1.15.8.1
       via: kong/1.2.1
       x-kong-proxy-latency: 14
       x-kong-upstream-latency: 0
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response contents:
       {
         "reply": "hello Kong!"
       }
       
       Response trailers received:
       (empty)
       Sent 1 request and received 10 responses

   `kong-route-id`
   応答ヘッダーに異なる値が指定され、このページに作成された2番目のルートを指していることに注意してください。

{:.note}
> 
> **注：**
> 一部のgRPCクライアント（通常はCLIクライアント）は[「gRPCリフレクションリクエスト」](https://github.com/grpc/grpc/blob/master/doc/server_reflection_tutorial.md)
> を発行し、これがサーバーでのエクスポート方法とその方法の呼び出し方を決定する手段となります。
> これらのリクエストには特定のパスがあり、たとえば、`/grpc.reflection.v1alpha.ServerReflection/ServerReflectionInfo`は有効なリフレクションパスです。
> 他のプロキシリクエスト同様、Kongはこれらのリクエストのルーティング方法を認識する必要があります。
> 前述の例では、キャッチオールルートにルーティングされます。このパスはパスが `/`と、あらゆるパスと一致します。
> gRPCリフレクションリクエストに一定するルートがない場合、Kongは通例どおり、`404 Not Found`応答を返します。

プラグインの有効化
---------

gRPCで[File Log](/hub/kong-inc/file-log)プラグインを試してみましょう。

1. `SayHello` ルートで File Log プラグインを有効にするために、次のリクエストを発行します。

   ```bash
   curl -X POST localhost:8001/routes/say-hello/plugins \
     --data name=file-log \
     --data config.path=grpc-say-hello.log
   ```

2. ログの出力に従って、gRPCリクエストが`SayHello`に対して行われます。

       tail -f grpc-say-hello.log
       {"latencies":{"request":8,"kong":5,"proxy":3},"service":{"host":"localhost","created_at":1564527408,"connect_timeout":60000,"id":"74a95d95-fbe4-4ddb-a448-b8faf07ece4c","protocol":"grpc","name":"grpc","read_timeout":60000,"port":15002,"updated_at":1564527408,"write_timeout":60000,"retries":5},"request":{"querystring":{},"size":"46","uri":"\/hello.HelloService\/SayHello","url":"http:\/\/localhost:9080\/hello.HelloService\/SayHello","headers":{"host":"localhost:9080","content-type":"application\/grpc","kong-debug":"1","user-agent":"grpc-go\/1.20.0-dev","te":"trailers"},"method":"POST"},"client_ip":"127.0.0.1","tries":[{"balancer_latency":0,"port":15002,"balancer_start":1564527732522,"ip":"127.0.0.1"}],"response":{"headers":{"kong-route-id":"e49f2df9-3e8e-4bdb-8ce6-2c505eac4ab6","content-type":"application\/grpc","connection":"close","kong-service-name":"grpc","kong-service-id":"74a95d95-fbe4-4ddb-a448-b8faf07ece4c","kong-route-name":"say-hello","via":"kong\/1.2.1","x-kong-proxy-latency":"5","x-kong-upstream-latency":"3"},"status":200,"size":"298"},"route":{"id":"e49f2df9-3e8e-4bdb-8ce6-2c505eac4ab6","updated_at":1564527431,"protocols":["grpc"],"created_at":1564527431,"service":{"id":"74a95d95-fbe4-4ddb-a448-b8faf07ece4c"},"name":"say-hello","preserve_host":false,"regex_priority":0,"strip_path":false,"paths":["\/hello.HelloService\/SayHello"],"https_redirect_status_code":426},"started_at":1564527732516}
       {"latencies":{"request":3,"kong":1,"proxy":1},"service":{"host":"localhost","created_at":1564527408,"connect_timeout":60000,"id":"74a95d95-fbe4-4ddb-a448-b8faf07ece4c","protocol":"grpc","name":"grpc","read_timeout":60000,"port":15002,"updated_at":1564527408,"write_timeout":60000,"retries":5},"request":{"querystring":{},"size":"46","uri":"\/hello.HelloService\/SayHello","url":"http:\/\/localhost:9080\/hello.HelloService\/SayHello","headers":{"host":"localhost:9080","content-type":"application\/grpc","kong-debug":"1","user-agent":"grpc-go\/1.20.0-dev","te":"trailers"},"method":"POST"},"client_ip":"127.0.0.1","tries":[{"balancer_latency":0,"port":15002,"balancer_start":1564527733555,"ip":"127.0.0.1"}],"response":{"headers":{"kong-route-id":"e49f2df9-3e8e-4bdb-8ce6-2c505eac4ab6","content-type":"application\/grpc","connection":"close","kong-service-name":"grpc","kong-service-id":"74a95d95-fbe4-4ddb-a448-b8faf07ece4c","kong-route-name":"say-hello","via":"kong\/1.2.1","x-kong-proxy-latency":"1","x-kong-upstream-latency":"1"},"status":200,"size":"298"},"route":{"id":"e49f2df9-3e8e-4bdb-8ce6-2c505eac4ab6","updated_at":1564527431,"protocols":["grpc"],"created_at":1564527431,"service":{"id":"74a95d95-fbe4-4ddb-a448-b8faf07ece4c"},"name":"say-hello","preserve_host":false,"regex_priority":0,"strip_path":false,"paths":["\/hello.HelloService\/SayHello"],"https_redirect_status_code":426},"started_at":1564527733554}

