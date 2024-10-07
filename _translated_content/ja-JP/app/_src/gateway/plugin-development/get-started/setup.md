---
title: "プラグインプロジェクトの設定"
book: "plugin_dev_getstarted"
chapter: 2
---
以下の手順は、最初のビルド、テスト、
[Lua](http://www.lua.org/)を使用した
{{site.base_gateway}}カスタム プラグイン実行を迅速に行うために記述されています。このセクションの残りのページでは、
プラグインの開発とベストプラクティスに関連する
その他の高度なトピックについて詳しく説明します。

前提条件
----

このガイドを完了するには、次の開発ツールが必要です。

* [Docker](https://docs.docker.com/get-docker/)（またはDockerと同等のもの）は{{site.base_gateway}}の実行とコードのテストに使用されます
* [curl](https://curl.se/)はWebリソースをダウンロードするために使用されます。
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)は、ホスト マシンにソフトウェアをインストールおよび更新するために使用されます。

段階的な手順
------

この章では、まず単純な {{site.base_gateway}} カスタムプラグイン プロジェクトを作成します。必要なフォルダとファイルを作成し、次に簡単な Lua コードを書いて基本的な機能を持つプラグインを作成します。

### 新しいプラグインリポジトリを初期化する

まず、ターミナルを開き、ディレクトリをソースコードを格納する場所に変更します。

プラグイン用の新しいフォルダを作成し、その中に移動します。

```sh
mkdir -p my-plugin && \
  cd my-plugin
```

次に、プラグインフォルダ構造を作成します。

{:.note}
> 
> **重要：** このガイドで紹介されている特定にツリー構造とファイル名は、プラグインの開発と実行が{{site.base_gateway}}で正常に機能するために重要です。このガイドのこれらの名前から逸脱しないでください。

```sh
mkdir -p kong/plugins/my-plugin && \
  mkdir -p spec/my-plugin
```

プラグインは、以下の個別ファイル内で定義された[Luaモジュール](http://www.lua.org/manual/5.1/manual.html#5.3)で構成されています。

まず、機能するプラグインに最低限必要なモジュールである空の`handler.lua`ファイルと`schema.lua`ファイルを作成します。

```sh
touch kong/plugins/my-plugin/handler.lua
touch kong/plugins/my-plugin/schema.lua
```

これで、新しいプラグインの基本構造ができました。これらのモジュールのコードの作成方法を見てみましょう。

### スキーマモジュールを初期化

`schema.lua`ファイルは、プラグインの設定データモデルを定義します。以下は、有効なプラグインに必要な最小構造です。

次のコードを`schema.lua`ファイルに追加します。

```lua
local PLUGIN_NAME = "my-plugin"

local schema = {
  name = PLUGIN_NAME,
  fields = {
    { config = {
        type = "record",
        fields = {
        },
      },
    },
  },
}

return schema
```

これにより、プラグインの構成用の空の基本テーブルが作成されます。
このガイドの後半では、実行時にプラグインを構成するための構成可能な値をテーブルに追加します。

次に、プラグインのハンドラコードを追加します。

### ハンドラモジュールを初期化

`handler.lua` モジュールには、新しいプラグインのコアロジックが含まれています。
まず、`handler.lua` ファイルに以下の Lua コードを記述します。

```lua
local MyPluginHandler = {
    PRIORITY = 1000,
    VERSION = "0.0.1",
}

return MyPluginHandler
```

このコードは、有効なプラグインの必須フィールドのセットを指定する[Luaテーブル](https://www.lua.org/pil/2.5.html)を定義します。

* `PRIORITY`フィールドは、他の 読み込まれたプラグインに 対して相対的に実行されるタイミングを決定する、プラグインの静的な[実行順序](/gateway/{{page.release}}/plugin-development/custom-logic/#plugins-execution-order)を設定します。
* `VERSION`フィールドはこのプラグインのバージョンを設定し、 `major.minor.revision`形式に従う必要があります。

これで有効なプラグインができましたが、現在は何も実行しません。

次に、プラグインにロジックを追加し、それを検証する方法を学びます。

### ハンドラロジックを追加する

プラグインロジックは、HTTP リクエスト、TCP ストリーム、および {{site.base_gateway}} 自体のライフサイクルにおけるいくつかの重要なポイントで実行されるように定義されています。

`handler.lua`モジュール内で、よく知られた名前が付いた[関数](/gateway/{{page.release}}/plugin-development/custom-logic/#available-contexts)をプラグインテーブルに追加して、プラグインロジックが実行される必要のあるポイントを示すことができます。

この例では、アップストリームサービスから応答を受信し、クライアントに返す前に実行される`response`関数を追加します。

レスポンスをクライアントに返す前に、レスポンスにヘッダーを追加しましょう。`return MyPluginHandler`ステートメントの前の`handler.lua`ファイルに、以下のファンクション実装を追加します。

```lua
function MyPluginHandler:response(conf)
    kong.response.set_header("X-MyPlugin", "response")
end
```

`kong.response`モジュールは[Kong PDK](/gateway/{{page.release}}/plugin-development/pdk/)に、クライアントに
返されるレスポンスを操作するための関数を提供します。上記のコードは、
`X-MyPlugin`という名前と`response`の値を持つすべてのレスポンスに、新たなヘッダーを設定します。

完全な`handler.lua`ファイルのリストは次のようになります。

```lua
local MyPluginHandler = {
  PRIORITY = 1000,
  VERSION = "0.0.1",
}

function MyPluginHandler:response(conf)
    kong.response.set_header("X-MyPlugin", "response")
end

return MyPluginHandler
```

{:.important}
> 
> **重要:** Kong PDK は、カスタムプラグイン開発のための安定したインターフェイスと一連の関数を提供します。PDK の一部 *ではない* {{site.base_gateway}} コードベースのモジュールを使用しないことが重要です。これらのモジュールは、安定したインターフェイスまたは動作を提供する保証はなく、プラグインコードで使用すると予期しない動作を引き起こす可能性があります。

次の手順
----

この段階で、{{site.base_gateway}}用の機能的なプラグインができました。ただし、どの開発プロジェクトもテストなしでは不完全です。次の章では、テストツールをインストールし、自動テストルーチンを作成する方法を学習します。

