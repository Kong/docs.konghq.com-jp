---
title: "動的ログレベル更新"
content_type: "reference"
---
Admin APIの`/debug`エンドポイントを使用すると、{{site.base_gateway}}を再起動せずに、{{site.base_gateway}}のログレベルを動的に変更できます。このエンドポイントのセットは[RBAC](/gateway/api/admin-ee/latest/#/rbac/post-rbac-roles-name_or_id-endpoints)を使用して保護でき、ログレベルの変更は[監査ログ](/gateway/{{page.release}}/kong-enterprise/audit-log/)に反映されます。

ログレベルの変更は、新しく生成されたワーカーを含む、従来のクラスタ内のすべてのNGINXワーカーノードに伝播されます。

{:.note}
> 
> **注：** 
> 
> * `debug`エンドポイントは、ハイブリッドモードのデータプレーン（DP）では機能 **しません** 。このエンドポイントは[コントロールプレーン（CP）ノード](#manage-new-nodes-in-the-cluster)にのみ使用できます。
> * 本番環境でログレベルを `debug` に変更すると、ディスクがすぐにいっぱいになる可能性があります。デバッグログの取得が終了したら、`notice` などの上位レベルに戻すか、リクエストクエリ文字列で `timeout` パラメータを使用してください。 **動的に設定されるログレベルのデフォルトのタイムアウトは 60 秒です** 。

現在のログレベルを表示
-----------

個々のノードのログレベルを表示するには、目的の`node`をパスパラメータとしてパスする`GET`リクエストを発行します。

```bash
curl --request GET \
  --url http://localhost:8001/debug/node/log-level/
```

適切な権限がある場合、このリクエストは現在のログレベルに関する情報を返します。

```json
{
    "message": "log level: notice"
}
```

{:.important}
> 
> データプレーン（DP）またはDBレスノードのログレベルを変更することはできません。

個別の{{site.base_gateway}}ノード向けログレベルの変更
---------------------

個々のノードのログレベルを変更するには、目的の`node`と[`log-level`](/gateway/{{page.release}}/production/logging/log-reference/)をパスパラメータとしてパスする`PUT`リクエストを発行します。

```bash
curl --request PUT \
  --url http://localhost:8001/debug/node/log-level/notice
```

適切な権限があり、リクエストに成功した場合は、`200`応答コードと次の応答ボディが返されます。

```json
{
	"message": "log level changed"
}
```

{{site.base_gateway}}クラスタのログレベルを変更
------------------

クラスタのすべてのノードのログレベルを変更するには、パスパラメータとして指定された目的の[`log-level`](/gateway/{{page.release}}/production/logging/log-reference/)で`PUT`リクエストを発行します。

```bash
curl --request PUT \
  --url http://localhost:8001/debug/cluster/log-level/notice
```

適切な権限があり、リクエストに成功した場合は、`200`応答コードと次の応答ボディが返されます。

```json
{
	"message": "log level changed"
}
```

### クラスタ内の新しいノードの管理

クラスタに追加された新しいノードのログレベルがクラスタ内の他のノードと同期していることを確認するには、[`kong.conf`](/gateway/{{page.release}}/reference/configuration/#log_level) の `log_level` エントリを `KONG_LOG_LEVEL` に変更してください。この設定により、新しいノードは既存のすべてのノードと同じログレベルでクラスタに参加できます。

すべてのコントロールプレーン（CP）{{site.base_gateway}}ノードのログレベルを変更
------------------------------------

クラスタ内のコントロールプレーン（CP）のログレベルを変更するには、パスパラメータとして指定された目的の[`log-level`](/gateway/{{page.release}}/production/logging/log-reference/)で`PUT`リクエストを発行します。

```bash
curl --request PUT \
  --url http://localhost:8001/debug/cluster/control-planes-nodes/log-level/notice
```

適切な権限があり、リクエストに成功した場合は、`200`応答コードと次の応答ボディが返されます。

```json
{
	"message": "log level changed"
}
```

