---
title: "Goでプラグインを書く"
content-type: "explanation"
---
Goプラグインの開発
----------


{{site.base_gateway}}は、{{site.base_gateway}}にGoバインディングを提供するライブラリ、[Go PDK](https://pkg.go.dev/github.com/Kong/go-pdk)でGo言語をサポートします。

概要
---

Goで{{site.base_gateway}}プラグインを書くには、以下を行います。

1. 構成を保持する`structure`タイプを定義します。
2. 構造のインスタンスを作成する `New()` 関数を書き込みます。
3. その構造にフェーズを処理するメソッドを追加します。
4. `go-pdk/server`サブライブラリを含めます。
5. `server.StartServer(New, Version, Priority)`を呼び出す `main()` 関数を追加します。
6. `go build`で実行可能ファイルとしてコンパイルします。

{:.note}
> 
> **注** ：[プラグインの例](https://github.com/Kong/go-pdk/tree/master/examples)については、Go PDKリポジトリを参照してください。

構成
---

### `Struct`

作成するプラグインには、データストアまたはAdmin APIから受信した設定データを処理する方法が必要です。受信データのスキーマを作成するには、`struct`を使用します。

```go
type MyConfig struct {
    Path   string
    Reopen bool
}
```

このプラグインは設定データを処理するため、`encoding/json` パッケージを使ってエンコーディングを制御したいと思うでしょう。
大文字で始まる Go フィールドはエクスポートでき、 `encoding/json`パッケージを含む現在のパッケージの外部からアクセスできるようになります。
データストア内のフィールドに別の名前を付けたい場合は、`struct` のフィールドにタグを追加してください。

```go
type MyConfig struct {
    Path   string `json:"my_file_path"`
    Reopen bool   `json:"reopen"`
}
```

### `New()`コンストラクタ

プラグインは`New`という関数で定義する必要があります。この関数は、 `MyConfig`構造体をインスタンス化し、それを`interface`として返す必要があります。

```go
func New() interface{} {
    return &MyConfig{}
}
```

### `main()`関数

各プラグインはスタンドアロンの実行可能ファイルとしてコンパイルされます。`github.com/Kong/go-pdk`をインポート
リストに含め、`main()`関数を追加します。

```go
func main () {
  server.StartServer(New, Version, Priority)
}
```

実行可能ファイルはパスのどこかに配置することができます（たとえば、`/usr/local/bin`）。共通の`-h`フラグは使用方法のヘルプメッセージを表示します。

```sh
my-plugin -h
```

出力:

    Usage of my-plugin:
      -dump
            Dump info about plugins
      -help
            Show usage info
      -kong-prefix string
            Kong prefix path (specified by the -p argument commonly used in the Kong CLI) (default "/usr/local/kong")

引数なしでプラグインを実行すると、`kong-prefix`の中にソケットファイルと実行ファイルの名前が作成され、末尾に`.socket` が追加されます。たとえば、実行ファイルが`my-plugin`の場合、ソケットファイルは`/usr/local/kong/my-plugin.socket` です。

### フェーズハンドラ

{{site.base_gateway}} Luaプラグインでは、リクエスト処理ライフサイクルのさまざまなポイントで実行されるカスタムロジックを実装できます。たとえば、アクセスフェーズでカスタムGoコードを実行するには、次の関数シグネチャを持つ`Access`という名前の関数を作成します。

```go
func (conf *MyConfig) Access (kong *pdk.PDK) {
  ...
}
```

同じ関数シグネチャを使用して、以下のフェーズでカスタムロジックを実装できます。

* `Certificate`
* `Rewrite`
* `Access`
* `Response`
* `Preread`
* `Log`

`Response` ハンドラが存在すると、プロキシでのバッファリングモードが自動的に有効になります。

### バージョンと優先度

プラグインコード内で次の定数を宣言することで、バージョン番号と実行の優先順位を定義できます。

```go
const Version = "1.0.0"
const Priority = 1
```


{{site.base_gateway}}は、優先順位の高いものから低いものへとプラグインを実行します。

構成例
---

`kong.conf`[構成ファイル](/gateway/latest/production/kong-conf/)を使用してプラグインを読み込むには、既存の{{site.base_gateway}}プロパティをプラグインの側面にマッピングする必要があります。`kong.conf`内でプラグインを読み込む2つの例を以下に示します。

    pluginserver_names = my-plugin,other-one
    
    pluginserver_my_plugin_socket = /usr/local/kong/my-plugin.socket
    pluginserver_my_plugin_start_cmd = /usr/local/bin/my-plugin
    pluginserver_my_plugin_query_cmd = /usr/local/bin/my-plugin -dump
    
    pluginserver_other_one_socket = /usr/local/kong/other-one.socket
    pluginserver_other_one_start_cmd = /usr/local/bin/other-one
    pluginserver_other_one_query_cmd = /usr/local/bin/other-one -dump


ソケットと開始コマンドの設定はデフォルトと一致しているため省略できます。

    pluginserver_names = my-plugin,other-one
    pluginserver_my_plugin_query_cmd = /usr/local/bin/my-plugin -dump
    pluginserver_other_one_query_cmd = /usr/local/bin/other-one -dump

その他の情報
------

* [PDKに関する参考資料](/gateway/latest/plugin-development/pdk/)
* [コンテナ付きプラグイン](/gateway/latest/plugin-development/pluginserver/plugins-kubernetes/)
* [Pythonでプラグインを開発する](/gateway/latest/plugin-development/pluginserver/python/)
* [JavaScriptを使用したプラグインの開発](/gateway/latest/plugin-development/pluginserver/javascript/)

