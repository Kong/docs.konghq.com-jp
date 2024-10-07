---
title: "サービスとルートの設定"
---
{{site.base_gateway}} の使用を開始する前に、正しくインストールされ、管理する準備ができていることを確認します。

始める前に
-----

このセクションを開始する前に、次のことを確認してください。

* {{site.base_gateway}} がインストールされ、実行されています。
* Kong Admin APIポートは、 適切なポート/IP/DNS設定をリッスンしています。
* 宣言型構成を使用して{{site.base_gateway}}を構成する場合は、 [decK](/deck/latest/installation)がインストールされています。

このガイドでは、{{site.base_gateway}} のインスタンスは `localhost` を使用して参照されます。`localhost` を使用していない場合は、`localhost` をコントロールプレーン（CP）インスタンスのホスト名に置き換えてください。

{{site.base_gateway}}設定の確認
----------

cURLまたは HTTPieを使用して、ゲートウェイのAdmin APIにリクエストを送信できることを確認します。

ターミナルウィンドウで次のコマンドを実行して、現在の構成を表示します。

```bash
curl -i -X GET http://localhost:8001
```

（オプション） コントロールプレーンとデータプレーンの接続の確認
--------------------------------

{{site.base_gateway}} を[ハイブリッドモード](/gateway/{{page.release}}/production/deployment-topologies/hybrid-mode/)で実行している場合は、このガイドのすべてのタスクをコントロールプレーン（CP）から実行する必要があります。クラスタステータス CLI を使用して、すべての構成がコントロールプレーン（CP）からデータプレーン（DP）にプッシュされていることを確認できます。

コントロールプレーン（CP）から以下を実行します。

```bash
curl -i -X GET http://localhost:8001/clustering/data-planes
```

出力には、クラスタ内の接続されているすべてのデータプレーン（DP）インスタンスが表示されます。

```json
{
    "data": [
        {
            "config_hash": "a9a166c59873245db8f1a747ba9a80a7",
            "hostname": "data-plane-2",
            "id": "ed58ac85-dba6-4946-999d-e8b5071607d4",
            "ip": "192.168.10.3",
            "last_seen": 1580623199,
            "status": "connected"
        },
        {
            "config_hash": "a9a166c59873245db8f1a747ba9a80a7",
            "hostname": "data-plane-1",
            "id": "ed58ac85-dba6-4946-999d-e8b5071607d4",
            "ip": "192.168.10.4",
            "last_seen": 1580623200,
            "status": "connected"
        }
    ],
    "next": null
}
```

まとめと次のステップ
----------

このセクションでは、

{{site.base_gateway}}の管理方法とその構成にアクセスする方法について学びました。次に、
[{{site.base_gateway}} を利用したサービスの公開](/gateway/{{page.release}}/get-started//expose-services/)について学びましょう。

