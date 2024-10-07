---
title: "JavaScript でプラグインを書く"
content-type: "explanation"
---
JavaScript言語の
{{site.base_gateway}}サポートは、[JavaScript PDK](https://github.com/Kong/kong-js-pdk)によって提供されます。ライブラリは{{site.base_gateway}}のJavaScriptバインディングのランタイムを提供するプラグインサーバーを提供します。

TypeScriptは以下の方法でもサポートされています。

* TypeScriptでプラグインを開発する場合、PDKには、型チェックを可能にするPDK関数の型定義が含まれています。
* TypeScriptで記述されたプラグインは、 {{site.base_gateway}}に直接ロードしてトランスパイルできます。

インストール
------

[JavaScript PDK](https://github.com/Kong/kong-js-pdk)は、`npm`を使用してインストールできます。プラグインサーバーバイナリをインストールするには、次のコマンドを実行します。

    npm install kong-pdk -g

開発
---

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

* `Plugin`属性は、このプラグインを実装するクラスを定義します。
* `Schema`は、プラグインの構成スキーマを定義します。
* `Version`と`Priority`の変数には、バージョン番号と実行の優先順位が設定されます。

**注** : [このリポジトリ](https://github.com/Kong/kong-js-pdk/tree/master/examples)には、JavaScript でビルドしたプラグインの例が含まれています。

フェーズハンドラー
---------

実行するカスタムロジックを実装できます。
リクエスト処理ライフサイクルのさまざまなポイント。アクセスフェーズの
カスタム JavaScript コードを実行するには、 `access`という名前の関数を定義します。

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

同じ関数シグネチャを使用して、以下のフェーズ中にカスタムロジックを実装できます。

* `certificate`
* `rewrite`
* `access`
* `response`
* `preread`
* `log`

`response`ハンドラが存在すると、プロキシでのバッファリングモードが自動的に有効になります。

PDK機能
-----

Kong は、ネットワークベースのプロセス間通信を通じて PDK と対話します。
各関数は[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)インスタンスを
返します。読みやすさを向上させるために、フェーズ ハンドラーで`async` / `await`キーワードを使用できます。

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

あるいは、`then`メソッドを使用してプロミスを解決します。

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

プラグインの依存関係
----------

プラグインサーバーを使用する場合、プラグインのソースコードを格納するディレクトリに`node_modules`ディレクトリも含まれている限り、プラグインに追加の依存関係を設定できます。

プラグインが`/usr/local/kong/js-plugins`に保存されていると仮定すると、追加の依存関係は
`/usr/local/kong/js-plugins/package.json`で定義されます。

開発者は、これらの依存関係をローカルに`/usr/local/kong/js-plugins/node_modules`にインストールするために、 `/usr/local/kong/js-plugins`の下で
`npm install`を実行する必要もあります。

Node.jsバージョンとアーキテクチャでプラグインサーバーを実行するものは、プラグインディレクトリで
`npm install`を実行するものと一致する必要があります。

TypeScriptプラグインを実行する場合、`kong-pdk`は `package.json`の依存関係として定義する必要があります。

```json
{
  "name": "ts_hello",
  "version": "0.1.0",
  "description": "hello TS plugin from kong",
  "main": "ts_hello.ts",
  "dependencies": {
    "kong-pdk": "^0.5.5"
  }
}
```

次に、TypeScriptファイルに`kong-pdk`をインポートします。

```javascript
import kong from "kong-pdk/kong";
```

### テスト

JavaScript PDK は、 [`jest`](https://jestjs.io/)を使用してプラグイン コードをテストするためのモック フレームワークを提供します。

`jest`を開発依存関係としてインストールしてから、`package.json`に`test`スクリプトを追加します。

    npm install jest --save-dev

この `package.json` には、次のような情報が含まれています。

    {
      "scripts": {
        "test": "jest"
      },
      "devDependencies": {
        "jest": "^26.6.3",
        "kong-pdk": "^0.3.2"
      }
    }

次のようにして、npmでテストを実行します。

    npm test

[このリポジトリ](https://github.com/Kong/kong-js-pdk/tree/master/examples)には
`jest`を使用してテストを記述する例が含まれています。

構成例
---

必要な依存関係をインストールして、システムを準備します。Debian/Ubuntuベースのシステムの場合。

    apt update
    apt install -y nodejs npm
    npm install -g kong-pdk

プラグインコードと`package.json`ファイルを`/usr/local/kong/js-plugins`にコピーして、次を実行します。

    cd /usr/local/kong/js-plugins/ 
    npm install

`kong.conf`[構成ファイル](/gateway/latest/production/kong-conf/)を使用してプラグインを読み込むには、既存の{{site.base_gateway}}プロパティをプラグインの側面にマッピングする必要があります。`kong.conf` 内でプラグインをロードする例を次に示します。

    pluginserver_names = js
    
    pluginserver_js_start_cmd = /usr/local/bin/kong-js-pluginserver -v --plugins-directory /usr/local/kong/js-plugins
    pluginserver_js_query_cmd = /usr/local/bin/kong-js-pluginserver --plugins-directory /usr/local/kong/js-plugins --dump-all-plugins
    
    pluginserver_js_socket = /usr/local/kong/js_pluginserver.sock

その他の情報
------

* [PDKリファレンス](/gateway/latest/plugin-development/pdk/)
* [コンテナを使用したプラグイン](/gateway/latest/plugin-development/pluginserver/plugins-kubernetes/)
* [Pythonでプラグインを開発](/gateway/latest/plugin-development/pluginserver/python/)
* [Goを使用したプラグインの開発](/gateway/latest/plugin-development/pluginserver/go/)

