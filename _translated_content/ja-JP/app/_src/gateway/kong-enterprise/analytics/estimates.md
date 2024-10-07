---
title: "PostgreSQLでのVitalsストレージの推定"
badge: "enterprise"
---
Vitalsデータは、 {{site.base_gateway}}ノード統計と要求応答コードの2つのカテゴリに分類できます。

{{site.base_gateway}} ノードの統計
------------

これらのタイプのメトリクスには、プロキシレイテンシー、アップストリームレイテンシー、およびキャッシュヒット/ミスがあります。{{site.base_gateway}} ノードの統計は以下のようなテーブルに保存されます。

* `vitals_stats_seconds_timestamp` Kongが毎 *秒* 実行するたびに新しい行を1行保存します
* `vitals_stats_minutes` Kongが毎 *分* 実行されるたびに新しい行を1行保存します。
* `vitals_stats_days` Kongが毎 *日* 実行されるたびに新しい行を1行保存します。


{{site.base_gateway}} ノード統計は、ワークスペース、サービス、ルートなどの特定の {{site.base_gateway}} エンティティに関連付けられていません。これらは、クラスタの状態を時間で表すように設計されています。つまり、{{site.base_gateway}} がトラフィックをルーティングしているかアイドル状態にあるかに関係なく、テーブルには新しい行が追加されます。

テーブルは無限に拡大することはなく、以下の期間データを保持します。

* `vitals_stats_seconds_timestamp` データを 1 時間（3600 行）保持します。
* `vitals_stats_minutes` データを25時間保持（90000 行）します。
* `vitals_stats_days` データを2年間（730行）保持します

リクエスト応答コード
----------

要求応答コードは、異なる理由に従って他のテーブル グループに保存されます。 このグループ内のテーブルは同じ構造を共有します（`entity_id` 、 `at` 、 `duration` 、 `status_code` 、 `count` ）。

* `vitals_code_classes_by_workspace`
* `vitals_code_classes_by_cluster`
* `vitals_codes_by_route`

このテーブルにはエンティティ固有の情報が保存されていないため、`entity_id`は`vitals_code_classes_by_cluster`に存在しません。
`vitals_code_classes_by_workspace`テーブルでは、`entity_id`は`workspace_id`です。`vitals_codes_by_route`テーブルでは、`entity_id`は`service_id`と`route_id`です。

`at`はタイムスタンプです。行が表す期間の開始をログに記録し、`duration`はその期間の継続時間です。

`status_code`および`count`は、行で表される期間内に観測されたHTTPステータスコード（200、300、400、500）の数です。

{{site.base_gateway}} ノードの統計テーブルは時間とともにしか増加しませんが、ステータスコードテーブルには {{site.base_gateway}} がトラフィックをプロキシする場合にのみ新しい行があり、新しい行の数はトラフィック自体によって異なります。

例：
---

まだトラフィックをプロキシしていない、完全に新しい{{site.base_gateway}}を検討してください。{{site.base_gateway}}ノード統計テーブルには行がありますが、ステータスコードテーブルにはありません。

{{site.base_gateway}}が最初のリクエストを`t`でプロキシし、ステータスコード200を返す場合、以下の行が追加されます。

```bash
vitals_codes_by_cluster
[second(t), 1, 200, 1]
[minute(t), 60, 200, 1] 
[day(t), 84600, 200, 1]
```

秒、分、日の内容は、次のように整えられます。

* `second(t)` は `t` を秒単位に切り詰めます。たとえば、`second(2021-01-01 20:21:30)` は `2021-01-01 20:21:30` になります。
* `minute(t)`は、分単位で切り詰められた`t`です。たとえば、 `minute(2021-01-01 20:21:30.234)` `2021-01-01 20:21:00`になります。
* `day(t)` は `t` を日単位に切り詰めます。たとえば、`day(2021-01-01 20:21:30.234)` は `2021-01-01 00:00:00` になります。

```bash
vitals_codes_by_workspace
[workspace_id, second(t), 1, 200, 1]
[workspace_id, minute(t), 60, 200, 1]
[workspace_id, day(t), 84600, 200, 1]

vitals_codes_by_route
[service_id, route_id, second(t), 1, 200, 1]
[service_id, route_id, minute(t), 60, 200, 1]
[service_id, route_id, day(t), 84600, 200, 1]
```

いくつかのシナリオで新しいリクエストがプロキシされると何が起こるかを考えてみましょう。

### 行が挿入されないシナリオ

同じ`t`で同じリクエストを再度実行し、やはり 200 を受け取った場合、新しい行は挿入されません。

この場合、既存の行のcount列が適宜増加します。

```bash
vitals_codes_by_cluster
[second(t), 1, 200, 2]
[minute(t), 60, 200, 2]
[day(t), 84600, 200, 2]

vitals_codes_by_workspace
[workspace_id, second(t), 1, 200, 2]
[workspace_id, minute(t), 60, 200, 2]
[workspace_id, day(t), 84600, 200, 2]

vitals_codes_by_route
[service_id, route_id, second(t), 1, 200, 2]
[service_id, route_id, minute(t), 60, 200, 2]
[service_id, route_id, day(t), 84600, 200, 2]
```

### 新しい行が挿入されるシナリオ

最後のリクエストが 500 ステータスコードを受信した場合、新しい行が挿入されます。

```bash
vitals_codes_by_cluster
[second(t), 1, 200, 1]
[minute(t), 60, 200, 1]
[day(t), 84600, 200, 1]

[second(t), 1, 500, 1]
[minute(t), 60, 500, 1]
[day(t), 84600, 500, 1]

vitals_codes_by_workspace
[workspace_id, second(t), 1, 200, 1]
[workspace_id, minute(t), 60, 200, 1]
[workspace_id, day(t), 84600, 200, 1]

[workspace_id, second(t), 1, 500, 1]
[workspace_id, minute(t), 60, 500, 1]
[workspace_id, day(t), 84600, 500, 1]

vitals_codes_by_route
[service_id, route_id, second(t), 1, 200, 1]
[service_id, route_id, minute(t), 60, 200, 1]
[service_id, route_id, day(t), 84600, 200, 1]

[service_id, route_id, second(t), 1, 500, 1]
[service_id, route_id, minute(t), 60, 500, 1]
[service_id, route_id, day(t), 84600, 500, 1]
```

### 2行目が挿入されるシナリオ

`t + 5s`で、`minute(t)==minute(t + 5s)`、{{site.base_gateway}}が同じリクエストをプロキシし、200を返すと仮定します。`t`と`t + 5s`の両方の`minute()`と`day()`は同じなので、分と日の行を更新するだけで済みます。`second()` は2つのインスタンスで異なるため、各テーブルに新たに2番目の行を挿入する必要があります。

```bash
vitals_codes_by_cluster
[second(t), 1, 200, 1]
[second(t + 5s), 1, 200, 1]
[minute(t), 60, 200, 2]
[day(t), 84600, 200, 2]

vitals_codes_by_workspace
[workspace_id, second(t), 1, 200, 1]
[workspace_id, second(t + 5s), 1, 200, 1]
[workspace_id, minute(t), 60, 200, 2]
[workspace_id, day(t), 84600, 200, 2]

vitals_codes_by_route
[service_id, route_id, second(t), 1, 200, 1]
[service_id, route_id, second(t + 5s), 1, 200, 1]
[service_id, route_id, minute(t), 60, 200, 2]
[service_id, route_id, day(t), 84600, 200, 2]
```

要約すると、ステータスコード テーブルの行数は、次のものに直接関係しています。

* {{site.base_gateway}}のプロキシされたリクエストで観測されたステータスコードの数
* これらのリクエストに関与した{{site.base_gateway}}個のエンティティの数
* プロキシされたリクエストの一定のフロー

シナリオの行数を推定する際に、次の特性を持つ{{site.base_gateway}}クラスタを検討します。

* ステータスコードの可能な 5 つのグループ（1xx、2xx、3xx、4xx、および 5xx）すべてを返すリクエストの一定のフロー。
* 1つのワークスペース、1つのサービス、1つのルート

24時間のトラフィックの後、ステータスコードテーブルには以下の行数が含まれます。

|          ステータスコードテーブル名          | 日数 |  分   |   秒   |  合計   |
|---------------------------------|----|------|-------|-------|
| **`vitals_codes_by_cluster`**   | 5  | 7200 | 18000 | 25200 |
| **`vitals_codes_by_workspace`** | 5  | 7200 | 18000 | 25200 |
| **`vitals_codes_by_route`**     | 5  | 7200 | 18000 | 25200 |

ここで注意すべき点は、5つのグループのステータスコードすべてが24時間のトラフィックで観察されたことを前提としていることです。そのため数量は5倍になっています。

このベースラインシナリオでは、トラフィックに含まれる {{site.base_gateway}} エンティティ（ワークスペースとルート）の数が増えたときに何が起こるかを簡単に計算できます。テーブル `vitals_codes_by_workspace` と `vitals_codes_by_route` の行番号は、それぞれワークスペースとルートの増加に伴って変化します。

上記の{{site.base_gateway}}クラスターを拡張して、それぞれ 1 つのルートを持つ 10 個のワークスペース \(合計 10 個のルート\) を作成し、トラフィックを 24 時間プロキシして 5 つのステータスコードすべてを返す場合、 `vitals_codes_by_workspace`と`vitals_codes_by_route`には 252,000 行が含まれます。

