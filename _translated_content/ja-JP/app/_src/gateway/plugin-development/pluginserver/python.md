---
title: "Pythonでプラグインを書く"
content-type: "explanation"
---
Python PDK
----------

Pythonプラグイン開発の
{{site.base_gateway}}サポートは、[kong\-python\-pdk](https://github.com/Kong/kong-python-pdk)ライブラリによって提供されます。
ライブラリは、プラグインサーバーと{{site.base_gateway}}の
インターフェイスとなるKong固有の関数を提供します。

インストール
------

プラグインサーバーと PDK をグローバルにインストールするには、 `pip`を使用します。

    pip3 install kong-pdk

開発
---

{{site.base_gateway}}Pythonプラグインの実装には次の属性があります。

```python
Schema = (
    { "message": { "type": "string" } },
)
version = '0.1.0'
priority = 0
class Plugin(object):
  pass
```

* `Plugin`という名前のクラスは、このプラグインを実装するクラスを定義します。
* `Schema`という名のディクショナリで、プラグインに期待される値とデータ型を定義します。
* バージョン番号とそれぞれの実行の優先順位を定義する変数`version`と`priority`。

{:.note}
> 
> **注** ：[このリポジトリ](https://github.com/Kong/kong-python-pdk/tree/master/examples)には、サンプルのPythonプラグインと[APIリファレンス](https://kong.github.io/kong-python-pdk/py-modindex.html)が含まれています。

フェーズハンドラ
--------

リクエスト処理ライフサイクルのさまざまなポイントで実行されるカスタムロジックを実装できます。アクセスフェーズ中にカスタムコードを実行するには、 `access` という名前の関数を定義します。

```python
class Plugin(object):
    def __init__(self, config):
        self.config = config
    def access(self, kong):
      pass
```

同じ関数シグネチャを使用して、以下のフェーズでカスタムロジックを実装できます。

* `certificate`
* `rewrite`
* `access`
* `response`
* `preread`
* `log`

`response`ハンドラが存在すると、プロキシでのバッファリングモードが自動的に有効になります。

{:.note}
> 
> **注：** フェーズハンドラメソッドの定義には、位置引数が必要です。以下の例では、位置引数は`kong`と呼ばれます。位置引数はPDK関数のルートオブジェクトとして使用できます。つまり、この引数を使用して `kong.log.info`や`kong.request.get_header`などの特定のPDK関数を呼び出すことができます。

### 型ヒント

[タイプヒント](https://www.python.org/dev/peps/pep-0484/)のサポートが利用可能です。タイプヒン
とIDEのオートコンプリートトを使用するには、フェーズハンドラー関数に `kong` パラメータを追加します。

```python
import kong_pdk.pdk.kong as kong
class Plugin(object):
    def __init__(self, config):
        self.config = config
    def access(self, kong: kong.kong):
        host, err = kong.request.get_header("host")
```

{:.important}
> 
> **警告:** `kong_pdk.pdk.kong`モジュール内のクラスと関数は型ヒントにのみ使用されるため、直接使用することはできません。PDK 関数を呼び出すには、フェーズハンドラーのパラメータとして渡される`kong`オブジェクトを使用します。PDK関数をフェーズハンドラーの外部で呼び出す場合は、外部コードに `kong` オブジェクトも渡す必要があることを覚えておいてください。

`Plugin`クラス以外でPDK関数を使用する例を次に示します。

```python
import kong_pdk.pdk.kong as kong

def example_access_phase(kong: kong.kong):
    host_header, err = kong.request.get_header("host")
    kong.log.info(host_header)

class Plugin(object):
    def __init__(self, config):
        self.config = config
    def access(self, kong: kong.kong):
        example_access_phase(kong)
```

組み込みサーバー
--------

組み込みサーバーを使用するには、次のコードを使用します。

```python
if __name__ == "__main__":
    from kong_pdk.cli import start_dedicated_server
    start_dedicated_server("py-hello", Plugin, version, priority)
```

`start_dedicated_server` の最初の引数はプラグイン名を定義するもので、すべての言語で一意にする必要があります。

構成例
---

`kong.conf`[構成ファイル](/gateway/latest/production/kong-conf/)を使用してプラグインを読み込むには、既存の{{site.base_gateway}}プロパティをプラグインの側面にマッピングする必要があります。`kong.conf`内でプラグインを読み込む例を以下に示します。

    pluginserver_names = my-plugin,other-one
    pluginserver_my_plugin_socket = /usr/local/kong/my-plugin.socket
    pluginserver_my_plugin_start_cmd = /path/to/my-plugin.py
    pluginserver_my_plugin_query_cmd = /path/to/my-plugin.py --dump
    pluginserver_other_one_socket = /usr/local/kong/other-one.socket
    pluginserver_other_one_start_cmd = /path/to/other-one.py
    pluginserver_other_one_query_cmd = /path/to/other-one.py -dump

ソケットと開始コマンドの設定はデフォルトと一致しているため省略できます。

    pluginserver_names = my-plugin,other-one
    pluginserver_my_plugin_query_cmd = /path/to/my-plugin --dump
    pluginserver_other_one_query_cmd = /path/to/other-one --dump

詳細ログを開く場合は、以下のように `-v`引数を`start`コマンドラインに渡します。

    pluginserver_my_plugin_start_cmd = /path/to/my-plugin.py -v

並行性モデル
------

Pythonプラグインサーバーと組み込みサーバーは、同時実行をサポートします。デフォルトでは、サーバーはマルチスレッドモードで起動します。

ワークロードが IO 集約型の場合は、`kong.conf`で`-g`フラグを`start_cmd`に渡すことで[Gevent](http://www.gevent.org/)モデルを使用できます。ワークロードが CPU を集中的に使用する場合は、`kong.conf`で`-m`フラグを`start_cmd`に渡して、マルチプロセッシングモデルを検討してください。

詳細情報
----

* [PDK リファレンス](/gateway/latest/plugin-development/pdk/)
* [コンテナを使用したプラグイン](/gateway/latest/plugin-development/pluginserver/plugins-kubernetes/)
* [Goを使用したプラグインの開発](/gateway/latest/plugin-development/pluginserver/go/)
* [JavaScriptを使用したプラグインの開発](/gateway/latest/plugin-development/pluginserver/javascript/)

