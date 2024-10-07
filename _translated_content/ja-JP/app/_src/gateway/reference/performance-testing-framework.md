---
title: "パフォーマンステストフレームワークリファレンス"
badge: "oss"
content_type: "reference"
---
{:.important}
> 
> **注** : {{site.base_gateway}} 3\.5\.xの時点で、この機能は非推奨になりました。3\.4\.x以降ではサポートされていません。

### perf.use\_defaults\(\)

*構文：perf.use\_defaults\(\)* 

デフォルトのパラメータを使用します。この関数では次の項目が設定されます。

* `PERF_TEST_DRIVER` 環境変数を検索し、`perf.use_driver`を呼び出します。
* `PERF_TEST_DRIVER` が `terraform` なら `PERF_TEST_TERRAFORM_PROVIDER` を検索します。各プロバイダーに渡すことができる環境変数については `perf.use_driver` を参照してください。
* `perf.set_log_level`を呼び出し、レベルを`"debug"`に設定します。
* 再試行回数を3に設定します。
* `PERF_TEST_USE_DAILY_IMAGE`環境変数を検索し、その変数をドライバーオプションに渡します。

### perf.use\_driver

*構文：perf.use\_driver\(name,options?\)* 

ドライバー名を使用します。ドライバー名は「local」、「docker」、または「terraform」のいずれかである必要があります。追加のドライバーのパラメータは、オプションでLuaテーブルとして指定できます。エラーがある場合はスローします。

DockerおよびTerraformドライバーでは、オプションパラメータに次のキーが含まれている必要があります。

* **use\_daily\_image** : デイリーイメージを使用するかどうかを決めるブール値。 マスターブランチの未リリースのコミットをテストするときに便利です。Enterpriseデイリーイメージをダウンロードするには、有効なDockerログインが必要です。 

Terraformドライバーの通常の設定では、次の任意のキーがオプションで含まれています

* **provider** サービスプロバイダーの名前で、現在は "equinix\-metal" のみです。
* **tfvars** 以下のような、LuaテーブルとしてのTerraform変数。環境変数は、`perf.use_defaults()`が呼び出された場合にのみ使用されます。

|     プロバイダー      |      tfvarsキー       |                   環境変数キー                    |                説明                 |                            デフォルト                            |
|-----------------|---------------------|---------------------------------------------|-----------------------------------|-------------------------------------------------------------|
| equinix\-metal | `metal_project_id`  | PERF\_TEST\_METAL\_PROJECT\_ID          | インスタンスが属するプロジェクトID。               | <required>                                                  |
| equinix\-metal | `metal_auth_token`  | PERF\_TEST\_METAL\_AUTH\_TOKEN          | Equinix認証トークン。                    | <required>                                                  |
| equinix\-metal | `metal_plan`        | PERF\_TEST\_METAL\_PLAN                  | インスタンスサイズ。                        | `"c3.small.x86"`                                            |
| equinix\-metal | `metal_os`          | PERF\_TEST\_METAL\_OS                    | オペレーティングシステムのイメージ名。               | `"ubuntu_20_04"`                                            |
| digitalocean    | `do_project_name`   | PERF\_TEST\_DIGITALOCEAN\_PROJECT\_NAME | インスタンスが属するプロジェクト名。存在しない場合は作成されます。 | `"Benchmark"`                                               |
| digitalocean    | `do_token`          | PERF\_TEST\_DIGITALOCEAN\_TOKEN          | DigitalOcean認証トークン。               | <required>                                                  |
| digitalocean    | `do_size`           | PERF\_TEST\_DIGITALOCEAN\_SIZE           | Dropletのサイズ。                      | `"s-1vcpu-1gb"`                                             |
| digitalocean    | `do_region`         | PERF\_TEST\_DIGITALOCEAN\_REGION         | Dropletをデプロイするリージョン。              | `"sfo3"`                                                    |
| digitalocean    | `do_os`             | PERF\_TEST\_DIGITALOCEAN\_OS             | オペレーティングシステムのイメージ名。               | `"ubuntu-20-04-x64"`                                        |
| aws\-ec2       | `aws_region`        | PERF\_TEST\_AWS\_REGION                  | EC2インスタンスをデプロイするAWSリージョン。         | `"us-east-2"`                                               |
| aws\-ec2       | `ec2_instance_type` | PERF\_TEST\_EC2\_INSTANCE\_TYPE         | EC2インスタンスの種類。                     | `"c4.4xlarge"`                                              |
| aws\-ec2       | `ec2_os`            | PERF\_TEST\_EC2\_OS                      | オペレーティングシステムのイメージパターン。            | `"ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"` |

### perf.set\_log\_level

*構文：perf.set\_log\_level\(level\)* 

ログレベルを設定します。ただし次のいずれかを除きます：`"debug"`、`"info"`、`"notice"`、`"warn"`、`"error"`、`"crit"`。
デフォルトのログレベルは、`"info"`です。ただし、`perf.use_defaults`が呼び出され、`"debug"`に設定されている場合を除きます。

### perf.set\_retry\_count

*構文：perf.set\_retry\_count\(count\)* 

各「ドライバ」操作の再試行時間を設定します。デフォルトでは、すべての操作が3回再試行されます。

### perf.setup

*構文: helpers = perf.setup\(\)* 

{{site.base_gateway}}を除く基本環境を準備し、 `spec.helpers`モジュールを返します。エラーがある場合は、エラーをスローします。

このフレームワークでは、`spec.helpers`モジュールをインポートする前に、環境変数がいくつか設定されます。
返されるヘルパーは、通常の`spec.helpers`モジュールです。ユーザーは統合テストで同じパターンを使用して、Kongにエンティティを設定できます。
現在、DBレスモードは実装されていません。

### perf.setup\_kong

*構文：helpers = perf.setup\_kong\(version\)* 

基本環境を準備し、`version`を使用して{{site.base_gateway}}をインストールしてから、 `spec.helpers`モジュールを返します。
エラーがある場合は、エラーをスローします。この関数を呼び出すと、`perf.setup()`も呼び出されます。

Gitハッシュまたはタグを選択するには、バージョンとして`"git:<hash id="sl-md0000000">"`を使用します。そうでなければ、フレームワークは
`version`をリリースバージョンとして扱い、バイナリパッケージをKongの配布ウェブサイトからダウンロードします。

このフレームワークでは、`spec.helpers`モジュールをインポートする前に、環境変数がいくつか設定されます。
返されるヘルパーは、通常の`spec.helpers`モジュールです。ユーザーは統合テストで同じパターンを使用して、Kongにエンティティを設定できます。
現在、DBレスモードは実装されていません。

### perf.start\_worker

*構文：`upstream_uri` = perf.start\_worker\(`nginx_conf_blob`, `port_count`?）* 

指定された`nginx_conf_blob`を使用してアップストリーム （Nginx/OpenResty）を開始します。{{site.base_gateway}}のビューからアクセス可能なアップストリームURIを返します。エラーがある場合はスローします。

`port_count`が設定されていて1より大きい場合、フレームワークは複数のポートでアップストリームを開始し、それらをテーブルで返します。

### perf.start\_kong

*構文：perf.start\_kong\(`kong_configs`?, `driver_configs`?\)* 

{{site.base_gateway}}を`kong_configs`の構成でLuaテーブルとして開始します。エラーがある場合はスローします。

`driver_configs`は、次のキーを含む可能性のあるLuaテーブルです。

* `name`：異なる {{site.base_gateway}} インスタンスを区別する文字列。1つの{{site.base_gateway}}インスタンスのみが起動される場合は必要ありません。これは、ある{{site.base_gateway}}インスタンスから別のインスタンスにアクセスするためのDNS名として使用できます。
* `ports`：公開ポートを設定するためのテーブル。このテーブルを尊重するのは、`docker`ドライバーのみです。

### perf.stop\_kong

*構文: perf.stop\_kong\(\)* 

すべてのKongインスタンスを停止します。エラーがある場合はスローします。

### perf.teardown

*構文：perf.teardown\(full?\)* 

ティアダウンを行います。エラーがある場合はスローします。Terraformドライバーでは、fullをtrueに設定するとすべてのインフラを終了させますが、
デフォルトでは、連続した実行を高速化するためだけにクリーンアップを行います。

### perf.start\_stapxx

*構文：perf.start\_stapxx\(`stapxx_file_name`, `arg`?, `driver_configs`?\)* 

[available stapxx scripts](https://github.com/Kong/stapxx/tree/kong/samples)
にある`stapxx_file_name`
と追加のCLI引数でStap\+\+スクリプトを開始します。該当する場合は、エラーがスローされます。

この関数は、 `SystemTap`モジュールが完全に準備され、カーネルに挿入されるまでテストの実行をブロックします。
これは、`perf.start_load`の前に呼び出す必要があります。

`driver_configs` は、次のキーを含むLuaテーブルです。

* `name`：プローブする個々の{{site.base_gateway}}インスタンスを区別するための文字列。

### perf.start\_load

*構文：perf.start\_load\(options?\)* 

`wrk`を使用してKongへの負荷の送信を開始します。エラーがある場合はスローします。Optionsは、次のものを含む可能性のあるLuaテーブルです。

* **path** 文字列のリクエストパス。デフォルトは`/`です。
* **uri** ベースURI例外パス。デフォルトは`http://kong-ip:kong-port/`です。
* **connections** 接続数。デフォルトの上限は1000です。
* **threads** スレッド数のリクエスト。デフォルトの上限は5です。
* **duration** パフォーマンステストの所要時間を秒単位で表します。デフォルトは10です。
* **script** `wrk`スクリプトの内容を文字列として指定します。デフォルトはnilです。
* **wrk2** `wrk`の代わりに`wrk2`を使用するためのブール値です。デフォルトはfalseです。
* **rate** 送信する毎秒リクエスト数。`wrk2`を使用する場合は必要ですが、`wrk`を使用する場合は無効です。
* **kong\_name** 負荷の送信先として{{site.base_gateway}}を起動するときに`driver_confs`によって指定された`name` 。これが指定されていない場合、フレームワークはプロキシポートを公開する最初の {{site.base_gateway}} インスタンスを選択します。

### perf.get\_admin\_uri

*構文：perf.get\_admin\_uri\(kong\_name?\)* 

Admin APIのURLを返します。URLにアクセスできるのは
ロードジェネレーターのみであり、一般に公開されていません。フレームワークは
管理ポートを公開する最初の
{{site.base_gateway}}インスタンスを選択します。{{site.base_gateway}}の起動時に`driver_confs`で指定された`name`にマッチする{{site.base_gateway}}インスタンスを選択するように`kong_name`を設定する必要があります。

### perf.wait\_result

*構文: result = perf.wait\_result\(options?\)* 

ロードテストが終了するまで待機し、結果を文字列として返します。エラーがある場合はスローします。

現在、この関数は無期限、または`wrk`と Stap\+\+プロセスの両方が終了するまで待機します。

### perf.combine\_results

*構文: `combined_result` = perf.combine\_results\(results, suite?）* 

`perf.wait_result`によって返された複数の結果を計算し、その平均と最小値/最大値を返します。
エラーがある場合はスローします。`suite`文字列はチャートに表示するテストスイートを識別し、
X軸ラベルとして使用されます。スイートが指定されていない場合、フレームワークはテスト名を使用します。

同じファイルの結果が相関している場合、ユーザーは `suite` を数値に設定して次のオプションを追加することで、
トレンドライン付きの折れ線グラフを描くことができます。

```lua
local charts = require "spec.helpers.perf.charts"
charts.options({
  suite_sequential = true,
  xaxis_title = "Upstreams count",
})
```

このオプションが設定されていない場合、チャートは同じテストファイル内の結果を独立したテストとして扱い、棒グラフを描画します。

### perf.generate\_flamegraph

*構文：perf.generate\_flamegraph\(path,title?\)* 

タイトル付きのFlameGraphが生成され、パスに保存されます。該当する場合は、エラーがスローされます。

### perf.save\_error\_log

*構文：perf.save\_error\_log\(path\)* 

Kongのエラーログをパスに保存します。エラーがある場合はスローします。

### perf.enable\_charts

*構文: perf.enable\_charts\(on?\)* 

チャートの生成を有効にし、エラーがある場合はスローします。

デフォルトでは、チャートは有効になっています。チャートのデータは`perf.combine_results`によって供給されます。
チャートを作成するために生成されたJSONの結果は、`output`ディレクトリに保存されます。

### perf.save\_pgdump

*構文：perf.save\_pgdump\(path\)* 

Blueprintを使用してKongをセットアップした後、PostgreSQLデータベースのダンプをパスに保存します。エラーがある場合はスローします。

### perf.load\_pgdump

*構文: perf.load\_pgdump\(path, `dont_patch_service`\)* 

パスからpgdumpをロードし、Kongデータベースをロードされたデータに置き換えます。この関数は
`dont_patch_service`がfalseに設定されていない限り、アップストリームへのサービスアドレスもパッチします。
これは`perf.start_upstream`の後に呼び出す必要があります。エラーがある場合はスローします。

カスタマイズ
------

### 新しいテストスイートの追加

すべてのテストはKongリポジトリの`spec/04-perf/01-rps`と`spec/04-perf/02-flamegraph`に保存されます。
いずれかのディレクトリの下に新しいファイルを追加し、テストの説明に`#tags`を追加します。

### Terraformで新しいプロバイダーを追加する

ユーザーは、Terraformによってサポートされている限り、ほとんどの主要サービスプロバイダーでTerraformドライバーを使用できます。
フレームワークとTerraformモジュールの間で次の契約が結ばれます。

Terraformのファイルは `spec/fixtures/perf/terraform/<provider id="sl-md0000000">` に保存されています。

3つのインスタンスがプロビジョニングされており、1つはKongの実行用、もう1つはPostgreSQLデータベースの実行用、そして最後の1つはアップストリームとロード（ワーカー）を実行するためのものです。ファイアウォールは、3つのインスタンス間の双方向トラフィックおよび公共のインターネットからのトラフィックを許可します。どちらのインスタンスもUbuntu 20\.04/focalを実行します。

両方のインスタンスにログインするためのSSHキーが
`spec/fixtures/perf/terraform/<provider id="sl-md0000000">/id_rsa`に存在するか、あるいは作成されます。ログインしたユーザーにはルート権限またはsudo権限があります。

以下はTerraform出力の変数です。

* **kong\-ip** ：KongノードのパブリックIP。
* **kong\-internal\-ip** ：Kongノードの内部IP（利用できない場合は、`kong-ip`を指定）。
* **worker\-ip** ：ワーカーノードのパブリックIP。
* **worker\-internal\-ip** : ワーカーノードの内部IP（利用できない場合は、 `worker-ip`を指定します）。
* **db\-ip** ：データベースノードのパブリックIP。
* **db\-internal\-ip** : データベースノードの内部IP（利用できない場合は、 `db-ip`を指定します）。

