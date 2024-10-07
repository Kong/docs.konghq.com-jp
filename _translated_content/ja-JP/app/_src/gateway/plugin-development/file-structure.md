---
title: "ファイル構成"
book: "plugin_dev"
chapter: 3
---
{:.note}
> 
> **注** ：この章は、[Lua](http://www.lua.org/)に精通していることを前提と
> しています。

お使いのプラグインは[Lua
モジュール](http://www.lua.org/manual/5.1/manual.html#5.3)のセットとして考えます。本章で説明されている各ファイルは、別個のモジュールとみなされます。
モジュールの名前が次の規則に従う場合、Kongはお使いのプラグインのモジュールを検出し、読み込みます。

    kong.plugins.<plugin_name id="sl-md0000000">.<module_name id="sl-md0000000">

モジュールが [package.path](http://www.lua.org/manual/5.1/manual.html#pdf-package.path)
変数を介して到達可能である必要があります。これは、必要に応じて
[`lua_package_path`](/gateway/{{page.release}}/reference/configuration/#lua_package_path)
の構成プロパティを使用して微調整できます。ただし、プラグインのインストールは Kong がネイティブに統合する [LuaRocks](https://luarocks.org/) を通して行うのが好ましいです。
LuaRocks でインストールされたプラグインについては、このガイドの後半で詳しく説明します。

プラグインのモジュールを探す必要があることを Kong に認識させるには、構成ファイルの[プラグイン](/gateway/{{page.release}}/reference/configuration/#plugins)プロパティ（コンマ区切りのリスト）に追加する必要があります。以下に例を示します。

```yaml
plugins = bundled,my-custom-plugin # your plugin name here
```

または、バンドルされているプラグインを一切ロードしたくない場合は、次のようにします。

```yaml
plugins = my-custom-plugin # your plugin name here
```

ここでKongは、次の名前空間からいくつかのLuaモジュールを読み込もうとします。

    kong.plugins.my-custom-plugin.<module_name id="sl-md0000000">

これらのモジュールの一部は必須であり（例：`handler.lua`）、
任意のものもあります。また、プラグインがその他の追加機能を実装できるようにします
（例：`api.lua`を使用してAdmin APIエンドポイントを拡張する、など）。

では、実装できるモジュールとそれらの目的について具体的に説明します。

基本的なプラグインモジュール
--------------

最も純粋な形式では、プラグインは次の2つの必須モジュールで構成されます。

    simple-plugin
    ├── handler.lua
    └── schema.lua

* [handler.lua](https://github.com/Kong/kong/blob/master/kong/plugins/basic-auth/handler.lua): プラグインのコア。これは実装すべきインターフェイスであり、各関数はリクエストや接続のライフサイクルの中で適切なタイミングで実行されます。
* [schema.lua](https://github.com/Kong/kong/blob/master/kong/plugins/basic-auth/schema.lua) : プラグインはユーザーによって入力されたいくつかの構成を保持する必要があるでしょう。このモジュールはその構成の *スキーマ* を保持し、それにルールを定義するため、ユーザーは有効な構成値のみを入力できます。

高度なプラグインモジュール
-------------

一部のプラグインは、データベースに独自のテーブルを保持したり、Admin APIでエンドポイントを公開したりするなど、Kong とより深く統合する必要があります。これらはそれぞれ、プラグインに新しいモジュールを追加することで実現できます。プラグインがすべてのオプションモジュールを実装している場合、プラグインの構造は次のようになります。

    complete-plugin
    ├── api.lua
    ├── daos.lua
    ├── handler.lua
    ├── migrations
    │   ├── init.lua
    │   └── 000_base_complete_plugin.lua
    └── schema.lua

実装可能なモジュール、実装例へのリンク、
そして使用目的を短く説明した完全なリストは次のとおりです。このガイドでは
それぞれを理解できるように詳しく説明します。

| モジュール名                                                                                            | 必須  |                                                                                                 説明                                                                                                 |
|:--------------------------------------------------------------------------------------------------|-----|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [api.lua](https://github.com/Kong/kong/blob/master/kong/plugins/prometheus/api.lua)               | いいえ | プラグインで処理するカスタムエンティティとやり取りするために Admin API で使用可能なエンドポイントのリストを定義します。                                                                                                                                  |
| [daos.lua](https://github.com/Kong/kong/blob/master/kong/plugins/basic-auth/daos.lua)             | いいえ | プラグインに必要なカスタムエンティティを抽象化し、データストアに保存する DAO（Database Access Objects）のリストを定義します。                                                                                                                       |
| [handler.lua](https://github.com/Kong/kong/blob/master/kong/plugins/basic-auth/handler.lua)       | はい  | 実装するインターフェイス。各関数は、リクエスト/接続のライフサイクル内の必要な時点でKongによって実行されます。                                                                                                                                          |
| [migrations/\*.lua](https://github.com/Kong/kong/tree/master/kong/plugins/basic-auth/migrations) | いいえ | データベースの移行（例：テーブルの作成）。移行が必要なのは、お使いのプラグインでカスタムエンティティがデータベースに保存され、[daos.lua](https://github.com/Kong/kong/blob/master/kong/plugins/basic-auth/daos.lua)で定義されたいずれかのDAOを介してそれらエンティティを操作する必要がある場合に限られます。 |
| [schema.lua](https://github.com/Kong/kong/blob/master/kong/plugins/basic-auth/schema.lua)         | はい  | プラグインの構成のスキーマを保持し、ユーザーが有効な構成値のみを入力できるようにします。                                                                                                                                                       |

[Key\-Auth プラグイン](/hub/kong-inc/key-auth/)は、このファイル構造を持つプラグインの例です。
詳細については、[Key\-Auth ソースコード](https://github.com/Kong/kong/tree/master/kong/plugins/key-auth)を参照してください。

