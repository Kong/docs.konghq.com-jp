---
title: "ヘルスチェックプローブ"
content_type: "tutorial"
---
このチュートリアルでは、{{site.base_gateway}}がユーザーのリクエストに対応する準備ができているかどうかを確実に判断できるノードのreadinessエンドポイントの使用方法を説明します。

Readinessチェックエンドポイントは、{{site.base_gateway}}が準備できたら`200 OK`を返し、準備が整っていない場合は`503 Service Temporarily Unavailable`を返します。これは、Kongインスタンスの準備状況を監視する必要があるロードバランサーやその他のツールに役立ちます。Kongの準備ができていない場合、エンドポイントは準備できていない理由を`message`フィールドで返します。これは、ユーザーがノードの準備が整っていると予測しているのに、まだ準備ができていない状況をデバッグするのに役立ちます。

{:.note}
> 
> **注：** Readinessエンドポイントは、ノードのステータスの詳細な情報を返しません。

ヘルスチェックの種類
----------

{{site.base_gateway}} ノードごとに、2 つの異なるヘルスチェック（「プローブ」とも呼ばれる）があります。

* **Liveness** : Kongが実行されている場合、`/status`エンドポイントは`200 OK`ステータスで応答します。Kongが実行されていない場合、リクエストは`500 Internal Server Error`で失敗するか、応答しません。GETリクエストを送信して、{{site.base_gateway}}インスタンスの稼働状態を確認できます。

  ```sh
  # Replace localhost:8100 with the appropriate host and port for
  # your Status API server
  
  curl -i http://localhost:8100/status
  ```

* **準備状況** ：Kongが有効な構成を正常に読み込み、トラフィックをプロキシする準備ができている場合、 `/status/ready`エンドポイントは`200 OK`ステータスで応答します。Kongがトラフィックをプロキシする準備ができていない場合、リクエストは`500 Internal Server Error`で失敗するか、応答がありません。GETリクエストを送信して、{{site.base_gateway}}インスタンスの準備状況を確認できます。

  ```sh
  # Replace localhost:8100 with the appropriate host and port for
  # your Status API server
  
  curl -i http://localhost:8100/status/ready
  ```

{{site.base_gateway}}のこれら2種類のヘルスチェックは、[Kubernetesがヘルスチェックプローブを定義する](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)方法に基づいてモデル化されています。

トラフィックを送信する前に、コンポーネント（ロードバランサー）が readiness ヘルスチェックを実行することを強くお勧めします。これにより、{{site.base_gateway}} ノードは正常に起動しただけでなく、構成のロードも完了し、プロキシトラフィックを受信できるようになります。

liveness ヘルスチェックは、readiness ヘルスチェックよりも先に200 OKのレスポンスを返すことがあります。
{{site.base_gateway}} が実行中であっても、完全な構成をまだロードしている可能性があります。これは、生存状態ではあるものの準備状態ではないことを意味します。
コンポーネントが liveness プローブのみを監視して {{site.base_gateway}} にトラフィックを送信するタイミングを決定する場合、{{site.base_gateway}} がトラフィックをプロキシする準備が整うまでのわずかな時間の間に、リクエストが `404 Not Found` のレスポンスを返されることがあります。
特に本番環境では、liveness プローブよりも readiness プローブを使用することをお勧めします。

ノードのReadinessエンドポイントの理解
-----------------------

手順に進む前に、ノードのreadinessエンドポイントの目的と、これがKongインスタンスの準備具合を決定する方法について理解しておくことが重要です。エンドポイントはノードのタイプによって動作が異なります。

{% navtabs %}
{% navtab Traditional mode %}

[従来モード](/gateway/{{page.release}}/production/deployment-topologies/traditional/)で次の条件がすべて満たされる場合、エンドポイントは`200 OK`を返します。

1. データベースへの接続が成功しました
2. すべてのKongワーカーはリクエストをルーティングする準備ができています
3. すべてのルートとサービスは、そのプラグインでリクエストを処理する準備が整っていること

{% endnavtab %}
{% navtab Hybrid mode (data plane role) or DB-less mode %}

[ハイブリッドモード](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/)（`data_plane`ロール）または[DB\-lessモード](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config/)では、次の条件が満たされるとエンドポイントは`200 OK`を返します。

1. Kong は有効で空でない構成（ `kong.yaml` ）をロードしました
2. すべてのKongワーカーはリクエストをルーティングする準備ができています
3. すべてのルートとサービスは、そのプラグインでリクエストを処理する準備が整っていること

{% endnavtab %}
{% navtab Hybrid mode (control plane role) %}

[ハイブリッドモード](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/)（`control_plane`ロール）では、次の条件が満たされると、このエンドポイントは`200 OK`を返します。

1. データベースへの接続が成功しました

{% endnavtab %}
{% endnavtabs %}

ノードのreadinessエンドポイントの有効化
------------------------

ノードのreadinessエンドポイントを使用するには、[`status_listen`](/gateway/latest/reference/configuration/#status_listen) 設定パラメータでStatus APIサーバー（デフォルトでは無効）を有効にしていることを確認してください。

例`kong.conf` :

```conf
status_listen = 0.0.0.0:8100
```

{:.note}
> 
> **注：** Readinessプローブは、スタンドアロン、コントロールプレーン（CP）、データプレーン（DP）の各ノードを含むクラスタ内のすべてのノードで使用されます。クラスタ内のノードを1つのみチェックするだけでは不十分です。

ノードの readiness エンドポイントの使用
-------------------------

ノードの readiness エンドポイントを有効にしたら、GET リクエストを送信して {{site.base_gateway}} インスタンスの readiness 状態を確認できます。

```sh
# Replace localhost:8100 with the appropriate host and port for
# your Status API server

curl -i http://localhost:8100/status/ready
```

レスポンスコードが`200`の場合、 {{site.base_gateway}}インスタンスはリクエストを処理する準備ができています。

```http
HTTP/1.1 200 OK
Date: Thu, 04 May 2023 22:00:52 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 19
X-Kong-Admin-Latency: 3
Server: kong/3.3.0

{
  "message": "ready"
}
```

応答コードが`503`の場合、 {{site.base_gateway}}インスタンスは正常ではないか、またはリクエストを処理する準備ができていません。

```http
HTTP/1.1 503 Service Temporarily Unavailable
Date: Thu, 04 May 2023 22:01:11 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 43
X-Kong-Admin-Latency: 3
Server: kong/3.3.0

{
  "message": "failed to connect to database"
}
```

```http
HTTP/1.1 503 Service Temporarily Unavailable
Date: Thu, 04 May 2023 22:06:58 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Length: 70
X-Kong-Admin-Latency: 16
Server: kong/3.3.0

{
  "message": "no configuration available (empty configuration present)"
}
```

### Kubernetesでのreadinessプローブの使用

Kubernetes または Helm を使用している場合は、新しいノードの readiness エンドポイントを使用するように readiness プローブ構成を更新する必要がある場合があります。構成ファイルの `readinessProbe` セクションを次のように変更します。

```yaml
readinessProbe:
    httpGet:
        path: /status/ready
        # Make sure to replace the port number with the one you
        # configured for the Status API Server.
        port: 8100
    initialDelaySeconds: 10
    periodSeconds: 5
```

{:.note}
> 
> **注意:** `initialDelaySeconds`を設定しないと、構成を完全に読み込むのに少し時間がかかるため、{{site.base_gateway}}がクラッシュループに陥る可能性があります。遅延する時間は、構成のサイズによって異なります。

### バージョン 3\.2 以前で準備状況チェックを使用する

`/status/ready` エンドポイントはバージョン3\.3で追加されたため、以前のバージョンでは、この組み込みの準備エンドポイントのメリットを享受できません。これらのバージョンでは、次の回避策をお勧めします。

1. この目的用に一意に設定されたパスを使用して、{{site.base_gateway}}に新しいルートを構成します。このルートにはサービスは必要ありません。

   たとえば、パス`/health/ready`を使用できます。
2. HTTP 200ステータスコードでそのルートに対するリクエストに返答するようにRequest Terminationプラグインを構成します。

{:.note}
> 
> **注：** この回避策では、ヘルスチェックリクエストを送信するポートは、ステータスAPIポートではなくプロキシポート（デフォルトでは8000 & 8443）です。

ヘルスチェックの対象外
-----------

ヘルスチェックプローブでは、以下の点は考慮されません。

* {{site.base_gateway}}が最適に機能しているかどうか
* 何らかの理由で {{site.base_gateway}} が断続的な障害を引き起こしている場合
* {{site.base_gateway}}が DNS、クラウドプロバイダーの停止、ネットワーク障害などのサードパーティシステムによりエラーをスローしている場合
* アップストリームサービスがエラーを出したり、応答が遅すぎる場合

関連項目
----

Kong および関連トピックの詳細については、以下のリソースを参照してください。

* [ヘルスチェックとモニタリングの概要](/gateway/latest/production/monitoring/)
* [Kong Admin APIドキュメント](/gateway/latest/admin-api/)
* [{{site.kic_product_name}}を使い始める](/kubernetes-ingress-controller/latest/deployment/overview/)
* [Kong Helmチャート](https://github.com/Kong/charts/tree/main/charts/kong)
* [Kongハイブリッドモード](/gateway/latest/production/deployment-topologies/hybrid-mode/)

