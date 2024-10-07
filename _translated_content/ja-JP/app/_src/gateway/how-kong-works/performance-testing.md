---
title: "Kong Gatewayのパフォーマンステスト"
content_type: "explanation"
---
{:.important}
> 
> **注** : {{site.base_gateway}} 3\.5\.x の時点で、この機能は非推奨です。3\.4\.x 以降ではサポートされていません。

{{site.base_gateway}}コードベースにはパフォーマンステストフレームワークが付属します。このフレームワークを使用すると、Kongの開発者とユーザーは、Kongだけでなく、バンドルまたはカスタムのプラグインのパフォーマンスを評価したり、フレームグラフを作図して、パフォーマンスのボトルネックをデバッグしたりできるようになります。
このフレームワークはKongが処理するリクエストのRPS（毎秒リクエスト数）とレイテンシを収集し、さまざまなワークロードでのパフォーマンスメトリクスを表します。

このフレームワークは、Kongの統合テストスイートの延長として実装されます。

インストール
------

このフレームワークは、[busted](https://github.com/lunarmodules/busted) と Kong の Lua 開発依存関係を使用しています。環境を設定するには、Kong リポジトリ上で `make dev` を実行して、すべての Lua 依存関係をインストールします。

使用方法
----

### 基本的な使用法

KongがインストールされているLua開発依存関係をインストールし、`busted`で各テストファイルを呼び出します。以下は、`docker`ドライバを使用してRPSと遅延テストを実行します。

```shell
PERF_TEST_USE_DAILY_IMAGE=true PERF_TEST_VERSIONS=git:master,git:perf/your-other-branch bin/busted -o gtest spec/04-perf/01-rps/01-simple_spec.lua
```

### Terraformマネージドインスタンス

デフォルトでは、Terraform は `lazy_teardown` の各テスト後にインスタンスをティアダウンしないため、ユーザーは毎回インフラを再構築することなく、複数のテストを連続して実行できます。

この動作は、`PERF_TEST_TEARDOWN_ALL` 環境変数を true に設定することで変更できます。また、ユーザーは以下を実行してインフラを手動でティアダウンすることもできます。

    PERF_TEST_TEARDOWN_ALL=1 PERF_TEST_DRIVER=terraform PERF_TEST_TERRAFORM_PROVIDER=your-provider bin/busted -o gtest -t single spec/04-perf/99-teardown/

### AWS

`terraform`ドライバーと`aws-ec2`をプロバイダーとして使用する場合、環境変数を使用してAWS認証情報を提供できます。
`AWS_PROFILE`変数を保存されているAWS認証情報プロファイルを指すように渡すことや、認証情報を直接使用するために
`AWS_ACCESS_KEY_ID`や`AWS_SECRET_ACCESS_KEY`を渡すことは、一般的です。詳しくは
[Terraform AWSプロバイダードキュメント](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication-and-configuration)
を参照してください。

### チャート

チャートはデフォルトではレンダリングされません。`output` ディレクトリに保存されているすべての
JSONデータにグラフを描画するためのリファレンス実装が
あります。次のコマンドを使用してグラフを描画します。

```shell
cwd=$(pwd)
cd spec/helpers/perf/charts/
pip install -r requirements.txt
for i in $(ls ${cwd}/output/*.data.json); do
  python ./charts.py $i -o "${cwd}/output/"
done
```

ドライバ
----

3つの「ドライバー」は環境タイプ、精度要件、セットアップの複雑さに応じて実装されます。

|   ドライバー   | git コミット間のテスト | バイナリリリース間のテスト | Flame Graph |         未公開バージョンをテスト         |
|-----------|---------------|---------------|-------------|------------------------------|
| Docker    | はい            | はい            |             | 有効（`use_daily_image` = true） |
| Terraform | はい            | はい            | はい          | 有効（`use_daily_image` = true） |

いずれかのドライバーを使用するには、次の Lua 開発依存関係をインストールする必要があります。

* **Docker** DriverはDockerイメージに基づいています。必要な依存関係は少なく、FlameGraphの生成はサポートしません。

* **Terraform** ドライバーはリモートVMまたはベアメタルマシンで実行されます。最も精度が高い反面、操作とセットアップにはTerraformの知識が必要です。

  * [Terraform](https://www.terraform.io/downloads.html)バイナリのインストールが必要です。
  * Git コミット間でテストする場合は、Git バイナリが必要です。Gitコミット間のテストでは、 フレームワークは、現在のディレクトリが Kong のリポジトリであると想定します。実行中のディレクトリを隠し テストが終了したら隠し場所を削除します。DockerまたはTerraformドライバーを使用する場合、 フレームワークは各コミットのベースバージョンを取得し、一致するDockerイメージまたは Kongバイナリパッケージを作成し、ローカルソースコードを内部に配置します。

ワークフロー
------

複数の
テストケースは同じLuaファイルを共有できます。

次のコードスニペットは、ワークロードを定義してKongを負荷テストするための、基本的なワークフローを示しています。

```lua
local perf = require("spec.helpers.perf")

perf.use_defaults()

local versions = { "git:master", "3.0.0" }

for _, version in ipairs(versions) do
  describe("perf test for Kong " .. version .. " #simple #no_plugins", function()
    local bp
    lazy_setup(function()
      local helpers = perf.setup_kong(version)

      bp = helpers.get_db_utils("postgres", {
        "routes",
        "services",
      })

      local upstream_uri = perf.start_worker([[
      location = /test {
        return 200;
      }
      ]])

      local service = bp.services:insert {
        url = upstream_uri .. "/test",
      }

      bp.routes:insert {
        paths = { "/s1-r1" },
        service = service,
        strip_path = true,
      }
    end)

    before_each(function()
      perf.start_kong({
        --kong configs
      })
    end)

    after_each(function()
      perf.stop_kong()
    end)

    lazy_teardown(function()
      perf.teardown()
    end)

    it("#single_route", function()
      local results = {}
      for i=1,3 do
        perf.start_load({
          path = "/s1-r1",
          connections = 1000,
          threads = 5,
          duration = 10,
        })

        ngx.sleep(10)

        local result = assert(perf.wait_result())

        print(("### Result for Kong %s (run %d):\n%s"):format(version, i, result))
        results[i] = result
      end

      print(("### Combined result for Kong %s:\n%s"):format(version, assert(perf.combine_results(results))))

      perf.save_error_log("output/" .. version:gsub("[:/]", "#") .. "-single_route.log")
    end)
  end)
end
```

このテストは、 `bin/busted path/to/this_file_spec.lua`を使用して通常の Busted テストとして実行できます。

他の例は、Kongリポジトリの[spec/04\-perf](https://github.com/Kong/kong/tree/master/spec/04-perf)フォルダをご覧ください。

サンプル出力
------

    ### Test Suite: git:origin/master~10 #simple #hybrid #no_plugins #single_route
    ### Result for Kong git:origin/master~10 (run 1):
    Running 30s test @ http://172.17.0.50:8000/s1-r1
      5 threads and 100 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency     3.53ms    3.35ms  87.02ms   89.65%
        Req/Sec     6.42k     1.33k   10.53k    67.29%
      Latency Distribution
         50%    2.65ms
         75%    4.35ms
         90%    6.99ms
         99%   16.86ms
      960330 requests in 30.10s, 228.96MB read
    Requests/sec:  31904.97
    Transfer/sec:      7.61MB
    ### Result for Kong git:origin/master~10 (run 2):
    Running 30s test @ http://172.17.0.50:8000/s1-r1
      5 threads and 100 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency     4.56ms    6.55ms 236.87ms   91.95%
        Req/Sec     5.79k     1.54k   10.50k    66.18%
      Latency Distribution
         50%    2.82ms
         75%    5.31ms
         90%    9.72ms
         99%   26.33ms
      863963 requests in 30.07s, 206.00MB read
    Requests/sec:  28730.72
    Transfer/sec:      6.85MB
    ### Result for Kong git:origin/master~10 (run 3):
    Running 30s test @ http://172.17.0.50:8000/s1-r1
      5 threads and 100 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency     3.79ms    3.81ms  82.25ms   89.60%
        Req/Sec     6.14k     1.43k   12.59k    66.49%
      Latency Distribution
         50%    2.77ms
         75%    4.69ms
         90%    7.71ms
         99%   18.88ms
      917366 requests in 30.08s, 218.72MB read
    Requests/sec:  30499.09
    Transfer/sec:      7.27MB
    ### Combined result for Kong git:origin/master~10:
    RPS     Avg: 30378.26
    Latency Avg: 3.94ms    Max: 236.87ms
       P90 (ms): 6.99, 9.72, 7.71
       P99 (ms): 16.86, 26.33, 18.88

[spec/04\-perf](https://github.com/Kong/kong/tree/master/spec/04-perf)のサンプルでは、RPSとレイテンシ結果、
Kongのエラーログ、チャート、Flame Graphファイルは、現在のディレクトリの`output`ディレクトリに保存されます。

