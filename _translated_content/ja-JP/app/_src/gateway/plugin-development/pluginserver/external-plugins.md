---
title: "他の言語のプラグイン"
---
はじめに
----

外部プラグインは、{{site.base_gateway}}自体とは別のプロセスで実行されるプラグインで、適切なプラグインサーバーが利用可能なすべてのプログラミング言語での使用が可能になります。

各プラグインサーバーは1つ以上のプラグインをホストし、Unix ソケットを経由してメインの {{site.base_gateway}} プロセスと通信します。その設定がされている場合、{{site.base_gateway}} はこれらのプロセスを管理して、必要に応じて開始、再起動、停止できます。


{{site.base_gateway}}現在、各Kong PDKsと以下の言語を
サポートしています。

* Go言語：[go\-pdk](https://github.com/Kong/go-pdk)
* JavaScript言語: [kong\-js\-pdk](https://github.com/Kong/kong-js-pdk)
* Python 言語: [kong\-python\-pdk](https://github.com/Kong/kong-python-pdk)

{{site.base_gateway}}プラグインサーバーの構成
-----------------

`pluginserver_names`プロパティは、コンマで区切られた名前のリストです。
各プラグインサーバープロセスごとに1つ指定します。これらの名前は各プロセスのプロパティをグループ化したり、
ログエントリに注釈を追加したりするために使用されます。

名前ごとに、他のプロパティを定義できます。

|                       プロパティ                       |           説明            |                       デフォルト                       |
|---------------------------------------------------|-------------------------|---------------------------------------------------|
| `pluginserver_<name id="sl-md0000000">_socket`    | Unixソケットパス              | `/usr/local/kong/<name id="sl-md0000000">.socket` |
| `pluginserver_<name id="sl-md0000000">_start_cmd` | プラグインサーバープロセスを開始するコマンド  | `/usr/local/bin/<name id="sl-md0000000">`         |
| `pluginserver_<name id="sl-md0000000">_query_cmd` | 利用可能なプラグインの情報をダンプするコマンド | `/usr/local/bin/query_<name id="sl-md0000000">`   |

たとえば、GoとPythonのプラグインを次のように設定できます（
`pypluginserver.py`と呼ばれる仮想Pythonプラグインサーバーを想定します）。

    pluginserver_names = go,python
    
    pluginserver_go_socket = /usr/local/kong/go_pluginserver.sock
    pluginserver_go_start_cmd = /usr/local/bin/go-plugin -kong-prefix /usr/local/kong
    pluginserver_go_query_cmd = /usr/local/bin/go-plugin -dump -kong-prefix /usr/local/kong
    
    pluginserver_python_socket = /usr/local/kong/python_pluginserver.sock
    pluginserver_python_start_cmd = /usr/local/bin/kong-python-pluginserver
    pluginserver_python_query_cmd = /usr/local/bin/kong-python-pluginserver --dump-all-plugins

これらのプラグインを有効にするには、各プラグイン名を`plugins`構成に追加します。各言語でこれらのhelloプラグインがあると仮定します
：

    plugins = bundled, go-hello, js-hello, py-hello

{:.note}
> 
> **注：** `pluginserver_XXX_start_cmd`および`pluginserver_XXX_query_cmd`コマンドは、
> 制限されたデフォルトの`PATH`変数を使用します。ほとんどの場合、代わりに完全な実行可能パスを指定する
> 必要があります。

### レガシー構成


{{site.base_gateway}}バージョン 2\.0\.x から 2\.2\.x では、Go 外部プラグインと
異なる構成スタイルを使用する単一のプラグインサーバーのみをサポートしていました。{{site.base_gateway}}バージョン 2\.3 以降では、
古いスタイルが認識され、内部で新しいスタイルに変換されます。

{:.important}
> 
> レガシー構成は{{site.base_gateway}}バージョン2\.8\.0では非推奨となり、{{site.base_gateway}}バージョン3\.0\.0で削除される予定です。

プロパティ`pluginserver_names`が定義されていない場合は、レガシープロパティ
`go_plugins_dir`と`go_pluginserver_exe`は新しいスタイルに透過的にマッピングされます。

|         プロパティ         |               説明               |              デフォルト               |
|-----------------------|--------------------------------|----------------------------------|
| `go_plugins_dir`      | Goプラグインのディレクトリ                 | `off`、つまりGoプラグインを無効にする           |
| `go_pluginserver_exe` | go\-pluginserver 実行可能ファイルへのパス | `/usr/local/bin/go-pluginserver` |

注：

* レガシースタイルの使用はお勧めしません。レガシースタイルは、{{site.base_gateway}} バージョン2\.8\.0で非推奨となり、 {{site.base_gateway}}バージョン3\.0\.0で削除される予定です。
* 従来の構成では、複数のプラグインサーバーを使用することはできません。
* [go\-pluginserver](https://github.com/Kong/go-pluginserver)バージョン 0\.5\.0 では、古いスタイルの構成が必要です。
* 新しい構成スタイルには、[go\-pluginserver](https://github.com/Kong/go-pluginserver)のv0\.6\.0が必要です。

#### レガシーから組み込みサーバースタイルへの更新

組み込みサーバーモードでは、プラグイン自体がプラグインサーバーとして機能します。
そのため、外部の `go-pluginserver` はもう必要ありません。

次の手順に従って、レガシー構成を埋め込みプラグインサーバースタイル構成に更新します。

1. `server.StartServer(New, Version, Priority)`を呼び出す`main()`関数を追加します。
2. Kong構成ファイルまたは環境変数で`go_plugins_dir`と`go_pluginserver_exe`の各プロパティが設定されていないことを確認します。 
3. [{{site.base_gateway}}プラグインサーバーの構成](#kong-gateway-plugin-server-configuration)に従って設定を行います。

必要な更新の例については、[go\-plugins](https://github.com/Kong/go-plugins/tree/v0.5.0)リポジトリを確認してください。`-lm`サフィックスを持つプラグインは従来の方法に対応し、サフィックスのないプラグインは組み込みプラグインサーバー方式に対応します。

Goプラグインの開発
----------


{{site.base_gateway}}は、[go\-pdk](https://github.com/Kong/go-pdk)を使用してGo言語をサポートします。これは、[PDK](/gateway/{{page.release}}/plugin-development/pdk/) の{{site.base_gateway}}機能にアクセスするためのGo関数を提供するライブラリです。

注：

{{site.base_gateway}}バージョン2\.3では、複数のプラグインサーバーを使用できます。特に
マイクロサービスとしてのプラグインを有効にする、単一プラグインサーバーの書込みができるようになりました
。このサポートのために、[go\-pdk](https://github.com/Kong/go-pdk)パッケージのバージョンv0\.6\.0
にはオプションのプラグインサーバーが含まれています。詳細については、[組み込みサーバー](#embedded-server)
を参照してください。

### 開発

Goで{{site.base_gateway}}プラグインを書くには、以下を行います。

1. 構成を保持する構造タイプを定義します。
2. 構造のインスタンスを作成する `New()` 関数を書き込みます。
3. その構造にフェーズを処理するメソッドを追加します。
4. `go-pdk/server`サブライブラリを含めます。
5. `server.StartServer(New, Version, Priority)`を呼び出す`main()`関数を追加します。
6. `go build`で実行可能ファイルとしてコンパイルします。

**注** ：Goプラグインの例については、[このリポジトリ](https://github.com/Kong/go-plugins)を参照してください。

#### 1\. 設定構造

Luaで書かれたプラグインは、データストアまたはAdmin APIから取得する
構成データの読み込み方法または確認方法を指定するスキーマを定義します。Goは
静的型付け言語であり、すべての仕様は次の
構成構造の定義によって処理されます。

```go
type MyConfig struct {
    Path   string
    Reopen bool
}
```

パブリックフィールド（大文字で始まるフィールド）には構成データが入力されます。データストアで別の名前をつけたい場合は、`encoding/json`パッケージで定義されているフィールドタグを追加します。

```go
type MyConfig struct {
    Path   string `json:"my_file_path"`
    Reopen bool   `json:"reopen"`
}
```

#### 2\. New\(\) コンストラクタ

プラグインは、このタイプのインスタンスを作成し、`interface{}` として戻る
`New` という関数を定義する必要があります。ほとんどの場合、以下のようになります。

```go
func New() interface{} {
    return &MyConfig{}
}
```

構造にさらにフィールドを追加すると、それらは渡されますが、
構成インスタンスの有効期間や数量についての
保証はありません。

#### 3\. main\(\) 関数

各プラグインはスタンドアロンの実行可能ファイルとしてコンパイルされます。`github.com/Kong/go-pdk/server`をインポートリストに含め、`main()`関数を追加します。

```go
func main () {
  server.StartServer(New, Version, Priority)
}
```

`main()`関数はファイルの先頭に
`package main`行が必要です。

次に、標準なGo buildにより実行ファイルが作成されます。余分な`go-pluginserver`、プラグインの読み込み、コンパイラ、ライブラリ、環境互換性の問題はありません。

結果として得られる実行可能ファイルは、パス内のどこかに置くことができます（たとえば、`/usr/local/bin`）。共通の `-h` フラグは使用方法のヘルプメッセージを表示します。

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

引数なしで実行すると、`kong-prefix`の中にソケットファイルと
実行ファイルの名前が作成され、末尾に`.socket` が追加されます。たとえば、
実行ファイルが`my-plugin`の場合、
デフォルトでは`/usr/local/kong/my-plugin.socket`です。

#### 4\. フェーズハンドラー

{{site.base_gateway}} Luaプラグインと同様に、リクエスト処理ライフサイクルのさまざまなポイントで実行されるカスタムロジックを実装できます。たとえば、アクセスフェーズでカスタムGoコードを実行するには、 `Access`という名前の関数を定義します。

```go
func (conf *MyConfig) Access (kong *pdk.PDK) {
  ...
}
```

カスタムロジックを実装できるフェーズは以下のとおりです。期待される関数シグネチャはすべてに対して同じです。

* `Certificate`
* `Rewrite`
* `Access`
* `Response`
* `Preread`
* `Log`

Lua プラグインと同様に、`Response` ハンドラがあると、プロキシでのバッファリングモードが自動的に有効になります。

#### 4\. バージョンと優先度

{{site.base_gateway}}Luaプラグインと同様に、プラグインコードに次の行を追加することで、バージョン番号と
実行の優先順位を定義できます。

```go
const Version = "1.0.0"
const Priority = 1
```


{{site.base_gateway}}は優先順位の高いものから低いものへとプラグインを実行します。

#### 構成例

`my-plugin`と`other-one`と呼ばれる、2つのスタンドアロンプラグインです。

    pluginserver_names = my-plugin,other-one
    
    pluginserver_my_plugin_socket = /usr/local/kong/my-plugin.socket
    pluginserver_my_plugin_start_cmd = /usr/local/bin/my-plugin
    pluginserver_my_plugin_query_cmd = /usr/local/bin/my-plugin -dump
    
    pluginserver_other_one_socket = /usr/local/kong/other-one.socket
    pluginserver_other_one_start_cmd = /usr/local/bin/other-one
    pluginserver_other_one_query_cmd = /usr/local/bin/other-one -dump


ソケットと起動コマンドの設定はデフォルトと
一致しているので、省略することができます。

    pluginserver_names = my-plugin,other-one
    pluginserver_my_plugin_query_cmd = /usr/local/bin/my-plugin -dump
    pluginserver_other_one_query_cmd = /usr/local/bin/other-one -dump

JavaScriptプラグインの開発
------------------

JavaScript言語の
{{site.base_gateway}}サポートは、[kong\-js\-pdk](https://github.com/Kong/kong-js-pdk)によって提供されます。
このライブラリは、JavaScriptプラグインのランタイムを提供するプラグインサーバーと、
[PDK](/gateway/{{page.release}}/plugin-development/pdk/)の{{site.base_gateway}}機能にアクセスするための関数を提供します。

TypeScriptは以下の方法でもサポートされています。

* TypeScript でプラグインを開発する場合、[kong\-js\-pdk](https://github.com/Kong/kong-js-pdk) には、型チェックを可能にするPDK関数の型定義が含まれています。
* TypeScript で記述されたプラグインは直接ロードして、その場でトランスパイルすることができます。

### 構成例

[kong\-python\-pdk](https://github.com/Kong/kong-js-pdk)は`npm`を使用してインストールできます。プラグインサーバーバイナリをインストールするには、次のコマンドを実行します。

    npm install kong-pdk -g

プラグインが`/usr/local/kong/js-plugins`に保存されているとしましょう。

    pluginserver_names = js
    pluginserver_js_socket = /usr/local/kong/js_pluginserver.sock
    pluginserver_js_start_cmd = /usr/local/bin/kong-js-pluginserver --plugins-directory /usr/local/kong/js-plugins
    pluginserver_js_query_cmd = /usr/local/bin/kong-js-pluginserver --plugins-directory /usr/local/kong/js-plugins --dump-all-plugins

### 開発

ローカル開発ディレクトリに[kong\-js\-pdk](https://github.com/Kong/kong-js-pdk)をインストールします。

    npm install kong-pdk --save

有効なJavaScriptプラグインの実装では、通常、次のオブジェクトをエクスポートします。

```javascript
module.exports = {
  Plugin: KongPlugin,
  Schema: [
    { message: { type: "string" } },
  ],
  Version: '0.1.0',
  Priority: 0,
}
```

`Plugin` 属性は、このプラグインを実装するクラスを定義します。`Schema` はプラグインの構成スキーマを定義し、Lua プラグインであるため、同じ構文を共有します。`Version` と `Priority` は、それぞれバージョン番号と実行の優先度を定義します。

**注** ：JavaScriptやTypeScriptのプラグインの例については、[このリポジトリ](https://github.com/Kong/kong-js-pdk/tree/master/examples)を参照してください。

#### 1\. フェーズハンドラ

{{site.base_gateway}} Luaプラグインと同様に、カスタムロジックを実装して、リクエスト処理のライフサイクルのさまざまなポイントで実行できます。たとえば、カスタムJavaScriptコードをアクセスフェーズで実行するには、`access`という名前の関数を定義します。

```javascript
class KongPlugin {
  constructor(config) {
    this.config = config
  }
  async access(kong) {
    // ...
  }
}
```

カスタムロジックを実装できるフェーズは以下のとおりです。期待される関数シグネチャはすべてに対して同じです。

* `certificate`
* `rewrite`
* `access`
* `response`
* `preread`
* `log`

Lua プラグインと同様に、`response` ハンドラがあると、プロキシでのバッファリングモードが自動的に有効になります。

#### 2\. PDKの機能

[kong\-js\-pdk](https://github.com/Kong/kong-js-pdk) は、ネットワークベースの IPC \(プロセス間通信\) を通じて Kong の PDK 機能を呼び出します。
つまり、各関数はプロミスインスタンスを返します。
読みやすくするために、フェーズハンドラーで `async`/`await` キーワードを使用すると便利です。

```javascript
class KongPlugin {
  constructor(config) {
    this.config = config
  }
  async access(kong) {
    let host = await kong.request.getHeader("host")
    // do something to host
  }
}
```

または、従来の方法でPromiseを消費します。

```javascript
class KongPlugin {
  constructor(config) {
    this.config = config
  }
  async access(kong) {
    kong.request.getHeader("host")
      .then((host) => {
        // do something to host
      })
  }
}
```

#### 3\. プラグインの依存関係

プラグインサーバーを使用する場合、プラグインのソースコードを格納するディレクトリに`node_modules`ディレクトリも含まれている限り、プラグインに追加の依存関係を設定できます。

プラグインが`/usr/local/kong/js-plugins`に保存されていると仮定すると、追加の依存関係は
`/usr/local/kong/js-plugins/package.json`で定義されます。開発者も
`/usr/local/kong/js-plugins`の下で`npm install`を実行して、それらの依存関係をローカルの
`/usr/local/kong/js-plugins/node_modules`にインストールします。

この場合は、プラグインサーバーを実行するノードのバージョンとアーキテクチャ、
pluginsディレクトリで`npm install`を実行するものが一致しなければならないことに注意してください。例えばmacOSで
`npm install`を実行し、作業ディレクトリをLinuxコンテナにマウントすると、壊れるかもしれません。

### テスト

[kong\-js\-pdk](https://github.com/Kong/kong-js-pdk)は`jest`を通じて、プラグインコードの正確性をテストする模擬フレームワークを提供します。

開発依存関係として`jest`をインストールし、 `package.json`に`test`スクリプトを追加します。

    npm install jest --save-dev

`package.json`には、次のようなコンテンツがあります。

    {
      "scripts": {
        "test": "jest"
      },
      "devDependencies": {
        "jest": "^26.6.3",
        "kong-pdk": "^0.3.2"
      }
    }

次のようにして、npm でテストを実行します。

    npm test

**注** : `jest`を使用してテストを記述する方法の例については、[このリポジトリ](https://github.com/Kong/kong-js-pdk/tree/master/examples)を確認してください。

Python プラグインの開発
---------------

Python 言語の
{{site.base_gateway}}サポートは[kong\-python\-pdk](https://github.com/Kong/kong-python-pdk)によって提供されます。
このライブラリは、Pythonプラグインのランタイムを提供するプラグインサーバーと、
[PDK](/gateway/{{page.release}}/plugin-development/pdk/)の{{site.base_gateway}}機能にアクセスするための関数を提供します。

### 構成例

[kong\-python\-pdk](https://github.com/Kong/kong-python-pdk)は`pip`を使用してインストールできます。プラグインサーバーバイナリとPDKをグローバルにインストールするには、次のコマンドを使用します。

    pip3 install kong-pdk

プラグインが`/usr/local/kong/python-plugins`に保存されているとしましょう。

    pluginserver_names = python
    pluginserver_python_socket = /usr/local/kong/python_pluginserver.sock
    pluginserver_python_start_cmd = /usr/local/bin/kong-python-pluginserver --plugins-directory /usr/local/kong/python-plugins
    pluginserver_python_query_cmd = /usr/local/bin/kong-python-pluginserver --plugins-directory /usr/local/kong/python-plugins --dump-all-plugins

### 開発

有効なPythonプラグインの実装には次の属性があります。

```python
Schema = (
    { "message": { "type": "string" } },
)
version = '0.1.0'
priority = 0
class Plugin(object):
  pass
```

`Plugin`という名前のクラスは、このプラグインを実装するクラスを定義します。`Schema`はプラグインの構成スキーマを定義し、Luaプラグインと同じ構文を共有します。`version`と`priority`は、
それぞれバージョン番号と実行の優先度を定義します。

**注** ：たとえばPythonプラグインや[API リファレンス](https://kong.github.io/kong-python-pdk/py-modindex.html)
など、[このリポジトリ](https://github.com/Kong/kong-python-pdk/tree/master/examples)を確認してください

#### 1\. フェーズハンドラ

{{site.base_gateway}} Luaプラグインと同様に、リクエスト処理ライフサイクルのさまざまなポイントで実行されるカスタムロジックを実装できます。たとえば、アクセスフェーズでカスタムGoコードを実行するには、 `access`という名前の関数を定義します。

```python
class Plugin(object):
    def __init__(self, config):
        self.config = config
    def access(self, kong):
      pass
```

カスタムロジックを実装できるフェーズは以下のとおりです。期待される関数シグネチャはすべてに対して同じです。

* `certificate`
* `rewrite`
* `access`
* `response`
* `preread`
* `log`

Lua プラグインと同様に、`response` ハンドラがあると、プロキシでのバッファリングモードが自動的に有効になります。

#### 2\. ヒントを入力する

[kong\-python\-pdk](https://github.com/Kong/kong-python-pdk)は、Python 3\.xで[タイプヒント](https://www.python.org/dev/peps/pep-0484/)をサポートしています。タイプ静的解析を使用してIDEで自動補完するには、ユーザーはフェーズハンドラで`kong`パラメータに注釈を追加できます。

```python
import kong_pdk.pdk.kong as kong
class Plugin(object):
    def __init__(self, config):
        self.config = config
    def access(self, kong: kong.kong):
        host, err = kong.request.get_header("host")
```

### 組み込みサーバー

各プラグインはマイクロサービスにすることができます。組み込みサーバーを使用するには、次のコードを使用します。

```python
if __name__ == "__main__":
    from kong_pdk.cli import start_dedicated_server
    start_dedicated_server("py-hello", Plugin, version, priority)
```

`start_dedicated_server` の最初の引数はプラグイン名を定義するもので、すべての言語で一意にする必要があることに注意してください。

#### 構成例

`my-plugin`と`other-one`と呼ばれる、2つのスタンドアロンプラグインです。

    pluginserver_names = my-plugin,other-one
    pluginserver_my_plugin_socket = /usr/local/kong/my-plugin.socket
    pluginserver_my_plugin_start_cmd = /path/to/my-plugin.py
    pluginserver_my_plugin_query_cmd = /path/to/my-plugin.py --dump
    pluginserver_other_one_socket = /usr/local/kong/other-one.socket
    pluginserver_other_one_start_cmd = /path/to/other-one.py
    pluginserver_other_one_query_cmd = /path/to/other-one.py -dump

ソケットと起動コマンドの設定はデフォルトと
一致しているので、省略することができます。

    pluginserver_names = my-plugin,other-one
    pluginserver_my_plugin_query_cmd = /path/to/my-plugin --dump
    pluginserver_other_one_query_cmd = /path/to/other-one --dump

### 並行モデル

Pythonプラグインサーバーと組み込みサーバーは、複数の並行モデルをサポートします。デフォルトでは、サーバーはマルチスレッドモードで起動します。

ワークロードがIO集中型の場合、`-g`をプラグインサーバーstart\_cmdに追加してgeventモデルを検討します。ワークロードがCPUを大量に消費する場合は、`-m`をプラグインサーバーのstart\_cmdに追加してマルチプロセッシングモデルを検討します。

外部プラグインのパフォーマンス
---------------

実装の詳細に応じて、Goプラグインは複数のCPUコアを使用することができます。
そのため、マルチコア システムで最高のパフォーマンスを発揮します。JavaScriptプラグインは、現在のところ
シングルコアのみで、専用のプラグインサーバーはサポートされていません。
Pythonプラグインは専用のプラグインサーバーを使用して、ワークロードを
複数のCPUコアに分散することもできます。

PDK関数の呼び出しがローカルプロセスで処理されるLuaプラグインとは異なり、
外部プラグインでPDK関数を呼び出すことはプロセス間通信を意味するため、
比較的高価な操作となります。外部プラグインでPDK関数を呼び出すにはコストがかかるため、外部プラグインを使用するKongのパフォーマンスは、
各リクエストにおけるIPC（プロセス間通信）の呼び出し回数に大きく影響します。

次のグラフは、パフォーマンスとリクエストごとのIPC呼び出し数の
相関関係を示しています。RPSとレイテンシの数値はハードウェアに依存しており、
混乱を避けるために排除されています。
<center><img title="RPS" src="/assets/images/products/plugins/external-plugins/rps.png"/></center> <center><img title="レイテンシ" src="/assets/images/products/plugins/external-plugins/latency.png"/></center> 

コンテナとKubernetesで外部プラグインを使用する
----------------------------

外部プラグインサーバーを必要とするプラグインを使用するには、プラグインサーバーとプラグイン自体の両方を {{ site.base_gateway }} コンテナ内にインストールする必要があります。

Golangで書かれたプラグインの場合は、ビルダーコンテナの組み込みサーバーモードで外部プラグインをビルドしてください。
そして、バイナリアーティファクトを {{ site.base_gateway }} コンテナにコピーまたはマウントします。
JavaScriptで書かれたプラグインの場合は、まずNodeと`npm`をインストールし、次に`npm`を使用して`kong-pdk`をインストールし、最後に
プラグインのソースコードを{{ site.base_gateway }}コンテナーにコピーまたはマウントします。
Python で書かれたプラグインの場合は、Python と`pip`をインストールします。次に `pip` を使って `kong-pdk` をインストールします。最後に、
プラグインのソースコードを{{ site.base_gateway }}コンテナーにコピーまたはマウントします。

イメージまたはコンテナを作成した後に {{ site.base_gateway }} を構成する方法については、
前のセクションを参照してください。

{:.note}
> 
> **注：** 公式の{{ site.base_gateway }}の画像は、`nobody`ユーザーを実行者として構成されています。カスタム画像を構築する際に、{{ site.base_gateway }}画像にファイルをコピーするには、一時的にユーザーを`root`に設定する必要があります。

```dockerfile
FROM kong
USER root
# Example for GO:
COPY your-go-plugin /usr/local/bin/your-go-plugin
# Example for JavaScript:
RUN apk update && apk add nodejs npm && npm install -g kong-pdk
COPY you-js-plugin /path/to/your/js-plugins/you-js-plugin
# Example for Python
# PYTHONWARNINGS=ignore is needed to build gevent on Python 3.9
RUN apk update && \
    apk add python3 py3-pip python3-dev musl-dev libffi-dev gcc g++ file make && \
    PYTHONWARNINGS=ignore pip3 install kong-pdk
COPY you-py-plugin /path/to/your/py-plugins/you-py-plugin
# reset back the defaults
USER kong
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 8000 8443 8001 8444
STOPSIGNAL SIGQUIT
HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health
CMD ["kong", "docker-start"]
```

*** ** * ** ***

