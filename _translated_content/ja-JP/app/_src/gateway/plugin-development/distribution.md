---
title: "インストールと配布"
book: "plugin_dev"
chapter: 11
---
Kongのカスタムプラグインは、各Kongノードのファイルシステム内に必要なLuaソースファイルで構成されています。このガイドでは、Kongノードにカスタムプラグインを認識させるための手順をステップバイステップで説明します。

これらの手順は、カスタムプラグインがそれぞれで利用可能になるようにKongクラスタの各ノードに適用される必要があります。

パッケージングソース
----------

通常のパッキング戦略（例：`tar`）を使うことも、LuaRocksパッケージマネージャーを使用して自動的にパッキングを行うこともできます。公式配布パッケージのいずれかを使用する場合は、Kongと一緒にLuaRocksをインストールすることをお勧めします。

LuaRocksを使用する場合は、パッケージの内容を指定する `rockspec`ファイルを作成する
必要があります。例については、 [Kong プラグイン
テンプレート](https://github.com/Kong/kong-plugin)を参照してください。フォーマットの詳細については、LuaRocks
[rockspecs に関するドキュメント](https://github.com/keplerproject/luarocks/wiki/Creating-a-rock)を参照してください。

次のコマンド（プラグインリポジトリから）を使用してロックをパックします。

1. ローカルにインストールします（現在のディレクトリの `.rockspec` に基づきます）。

   ```sh
   luarocks make
   ```

2. インストール済みのロックをパックします。

   {:.important}
   > 
   > **重要：** `luarocks pack`は、インストールされる`zip`ユーティリティに依存します。{{site.base_gateway}}の最近のイメージは厳格化されており、`zip`などのユーティリティは利用できなくなっています。Dockerのカスタムイメージの一部として実行される場合は、このコマンドの実行前に`zip`がインストールされるようにしてください。

   ```sh
   luarocks pack <plugin-name id="sl-md0000000"> <version id="sl-md0000000">
   ```

   プラグインのrockspecが`kong-plugin-my-plugin-0.1.0-1.rockspec` だと仮定すると、上記は次のようになります。

   ```sh
   luarocks pack kong-plugin-my-plugin 0.1.0-1
   ```

LuaRocksの`pack`コマンドが`.rock`ファイルを作成します（これは、ロックをインストールするために必要なものがすべて含まれたzipファイルです）。

LuaRocksを使用しない場合、または使用できない場合は、`tar`を使用して、
プラグインを構成する`.lua`ファイルを`.tar.gz`アーカイブにまとめます。また
ターゲットシステムにLuaRocksがある場合は、 `.rockspec`ファイルも
含めることができます。

このアーカイブの内容は、次のようなものです。

    tree <plugin-name id="sl-md0000000">
    <plugin-name id="sl-md0000000">
    ├── INSTALL.txt
    ├── README.md
    ├── kong
    │   └── plugins
    │       └── <plugin-name id="sl-md0000000">
    │           ├── handler.lua
    │           └── schema.lua
    └── <plugin-name id="sl-md0000000">-<version id="sl-md0000000">.rockspec

プラグインをインストールする
--------------

Kong ノードがカスタムプラグインを使用できるようにするには、カスタムプラグインの Lua ソースはホストのファイルシステムにインストールする必要があります。これを行うには、LuaRocks を使用するか、手動で行うという複数の方法があります。次のいずれかのパスを選択します。

注意：どの方法でプラグインのソースをインストールする場合も、Kongクラスターの各ノードに対して行う必要があります。

### 作成された「ロック」からLuaRocks経由

`.rock`ファイルはローカルまたはリモートサーバーからインストールできる自己完結型パッケージです。

`luarocks`ユーティリティがシステムにインストールされている場合（公式インストールパッケージを使用した場合は、おそらくこれに該当します）、LuaRocksツリー（LuaRocksがLua モジュールをインストールするディレクトリ）に「rock」をインストールできます。

次の手順でインストールできます。

```sh
luarocks install <rock-filename id="sl-md0000000">
```

ファイル名はローカル名、またはサポートされている方法のいずれかになります。例：
`http://myrepository.lan/rocks/my-plugin-0.1.0-1.all.rock`

### ソースアーカイブからLuaRocks経由

`luarocks`ユーティリティがシステムにインストールされている場合（公式インストールパッケージを使用した場合は、おそらくこれに該当します）、LuaRocksツリー（LuaRocksがLuaモジュールをインストールするディレクトリ）にLuaソースをインストールできます。

カレントディレクトリから rockspecファイルがある、展開されたアーカイブのディレクトリに移動することでそれを行うことができます。

```sh
cd <plugin-name id="sl-md0000000">
```

次に以下を実行します。

```sh
luarocks make
```

上記を実行すると、`kong/plugins/<plugin-name id="sl-md0000000">`のLuaソースがシステムのLuaRocksツリーにインストールされます。このツリーにはKongソースがすべて揃っています。

### Dockerfileまたはdocker run（インストールとロード）経由

{{site.base_gateway}}をDockerまたはKubernetesで実行している場合、プラグインは{{site.base_gateway}}コンテナ内にインストールする必要があります。
プラグインのソースコードをコンテナーにコピーまたはマウントします。

{:.note}
> 
> **注：** 公式の{{site.base_gateway}}の画像は、`nobody`ユーザーを実行者として構成されています。カスタム画像を構築する際に、{{site.base_gateway}}画像にファイルをコピーするには、一時的にユーザーを`root`に設定する必要があります。

以下は、{{site.base_gateway}} イメージにプラグインをマウントする方法を示す Dockerfile の例です。

```dockerfile
FROM kong/kong-gateway:latest

# Ensure any patching steps are executed as root user
USER root

# Add custom plugin to the image
COPY example-plugin/kong/plugins/example-plugin /usr/local/share/lua/5.1/kong/plugins/example-plugin
ENV KONG_PLUGINS=bundled,example-plugin

# Ensure kong user is selected for image execution
USER kong

# Run kong
ENTRYPOINT ["/entrypoint.sh"]
EXPOSE 8000 8443 8001 8444
STOPSIGNAL SIGQUIT
HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health
CMD ["kong", "docker-start"]
```

または、 `docker run` コマンドに以下を含めます。

    -v "$custom_plugin_folder:/tmp/custom_plugins/kong" 
    -e "KONG_LUA_PACKAGE_PATH=/tmp/custom_plugins/?.lua;;"
    -e "KONG_PLUGINS=bundled,example-plugin"

### 手動

プラグインのソースをインストールするための、
より保守的な方法は、LuaRocksのツリーを
「汚染」しないようにし、代わりにKongを、それらを含むディレクトリに向けます。

これはKong構成の`lua_package_path`プロパティを微調整することによって行われます。内部では、このプロパティはLua VMの`LUA_PATH`変数のエイリアスです（これに精通している場合）。

これらのプロパティには、Luaソースを検索するための、セミコロンで
区切られたディレクトリのリストが含まれています。Kongの設定ファイルで次のように設定する必要があります：

    lua_package_path = /<path-to-plugin-location id="sl-md0000000">/?.lua;;

この場合：

* `/<path-to-plugin-location id="sl-md0000000">`は、抽出されたアーカイブを含むディレクトリへのパスです。これは、アーカイブの`kong`ディレクトリ上である必要があります。
* `?`は、Kongがプラグインを読み込もうとした場合に`kong.plugins.<plugin-name id="sl-md0000000">`によって置き換えられるプレースホルダーです。変更しないでください。
* `;;`「デフォルトの Lua パス」のプレースホルダー。これは変更しないでください。

たとえば、以下はプラグイン `something`がファイルシステムにあり、ハンドラーファイルが次のディレクトリにある場合の例です。

    /usr/local/custom/kong/plugins/<something id="sl-md0000000">/handler.lua

`kong`ディレクトリの場所は`/usr/local/custom`であるため、適切なパスの設定は以下のようになります。

    lua_package_path = /usr/local/custom/?.lua;;

#### 複数のプラグイン

この方法で2つ以上のカスタムプラグインをインストールする場合は、変数を
変数は次のようなもの次のように設定できます。

    lua_package_path = /path/to/plugin1/?.lua;/path/to/plugin2/?.lua;;

* `;` はディレクトリ間の区切り文字です。
* `;;`こちらも「デフォルトのLuaパス」を意味します。

このプロパティは環境変数を介して設定することもできます。
同等: `KONG_LUA_PACKAGE_PATH`.

プラグインをロードします
------------

1. カスタムプラグインの名前を（各Kongノードの）Kong構成にある`plugins`リストに追加します。

       plugins = bundled,<plugin-name id="sl-md0000000">

   または、バンドルされたプラグインを含めたくない場合は、次のようにします。

       plugins = <plugin-name id="sl-md0000000">

   2 つ以上のカスタムプラグインを使用している場合は、次のように間にコンマを挿入します。

       plugins = bundled,plugin1,plugin2

   または：

       plugins = plugin1,plugin2

   このプロパティは、同等の環境変数 `KONG_PLUGINS` を使用して設定することもできます。
2. Kong クラスター内の各ノードの`plugins`ディレクティブを更新します。

3. Kongを再起動してプラグインを適用します。

       kong restart

   または、Kongを停止せずにプラグインを適用する場合は、以下を使用します。

       kong prepare
       kong reload

プラグインの読み込みの確認
-------------

これで、問題なくKongを起動できるはずです。サービス、ルート、または
コンシューマエンティティのプラグインをどのように有効化/設定すればよいか、
お客様のカスタムプラグインについてご相談ください。

1. プラグインが Kong にロードされていることを確認するには、`debug` ログレベルを使用して Kong を起動します。

       log_level = debug

   または:

       KONG_LOG_LEVEL=debug

2. そのあと、読み込まれる各プラグインに対して以下のログが表示されます。

       [debug] Loading plugin <plugin-name id="sl-md0000000">

プラグインの削除
--------

プラグインを完全に削除するには、3つのステップがあります。

1. Kong サービスまたはルート構成からプラグインを削除します。削除するプラグインがグローバルに、そして、どのサービス、ルート、またはコンシューマにも適用されていないことを確認してください。これは Kong クラスタ全体に対して一度だけ行う必要があり、再起動/再ロードは必要ありません。この手順を実施すると、プラグイン
   が使用されなくなります。しかしながら、プラグインはまだ利用可能であり、
   再度適用することができます。

2. `plugins`ディレクティブ（各Kongノード上）からプラグインを削除します。これは手順1を完了してから行ってください。この手順を実行したあとは、誰もプラグインをKong Service、ルート、コンシューマ、さらにはグローバルに再適用できなくなります。この手順を有効にするには、Kongノードを再起動/リロードする必要があります。

3. プラグインを完全に削除するには、各 Kong ノードからプラグイン関連のファイルを削除してください。ファイルを削除する前に、Kong の再起動/リロードを含む手順 2 を完了していることを確認してください。LuaRocksを使用してプラグインをインストールした場合、 `luarocks remove <plugin-name id="sl-md0000000">` を実行して削除します。

プラグインを配布する
----------

ゲートウェイが稼働しているプラットフォームに応じて、カスタムプラグインの配布にはさまざまな方法があります。

### LuaRocks

これを実行する方法の一つは、Lua モジュール用の
パッケージ マネージャーである [LuaRocks](https://luarocks.org/) を使うことです。このようなモジュールを "ロック"と呼びます。 **モジュールは Kong リポジトリ内に配置する必要はありません** が、Kong のセットアップをそのように維持したいのであれば配置することも可能です。

[rockspec](https://github.com/keplerproject/luarocks/wiki/Creating-a-rock)ファイルでモジュール（およびその永続的依存関係）を定義すると、LuaRocksからプラットフォームにこれらのモジュールをインストールできます。
LuaRocksにモジュールをアップロードして、誰でも利用できるようにすることもできます。

以下は、Lua 表記のモジュールとそれに対応するファイルを定義するために、`builtin` ビルドタイプを使用した [rockspec の例](https://github.com/Kong/kong-plugin/blob/master/kong-plugin-myplugin-0.1.0-1.rockspec)です。

フォーマットの詳細については、LuaRocksの
[rockspecs に関するドキュメント](https://github.com/keplerproject/luarocks/wiki/Creating-a-rock)を参照してください。

### OCIアーティファクト

多くのユーザーは、Docker HubやAmazon ECRなどのOCI準拠のレジストリにアクセスできます。Kong Pluginsは、汎用OCIアーティファクトとしてパッケージ化され、バージョン管理、保存、配布のためにこれらのいずれかにのレジストリにアップロードされます。

プラグインを OCI アーティファクトとして配布する利点は、ユーザーが多数のエコシステムの利点を活用できることです。これには、アーティファクトの構築、プッシュ、プル、そして署名（安全な出所証明のため）に関するツールが含まれます。以下の手順では、Kong カスタムプラグインを OCI アーティファクトとしてパッケージ化、配布、検証するサンプルフローを示しています。

プラグインが開発されたマシンで、または自動化ワークフローの一部として、以下の手順を実行します。

1. 上記の[「ソースのパッケージ」](#packaging-sources)セクションに従ってプラグインをパッケージ化してください。

   ```bash
   tar czf my-plugin.tar.gz ./my-plugin-dir
   ```

2. OSS [Cosign ツール](https://docs.sigstore.dev/system_config/installation/)を使用して、プラグインの署名と検証に使用するキーペアを生成します。

   ```bash
   cosign generate-key-pair
   ```

   プライベート鍵（ `cosign.key` ）は安全に保管する必要があり、プラグインアーティファクトの署名に使用されます。公開鍵（`cosign.pub`）は、ダウンロードしたプラグインをフローの後半で検証するために、対象マシンに配布して使用する必要があります。

   Cosignを使用してアーティファクトに署名および検証を行う、キーを使わないメソッドもあります。詳細情報は
   [ドキュメント](https://docs.sigstore.dev/signing/overview/)に記載されています。
3. OCI準拠のレジストリにログインします。この場合、Docker Hubを使用します。

   ```bash
   cat ~/foo_password.txt | docker login --username foo-user --password-stdin
   ```

4. Cosign を使用して、プラグイン アーティファクトを OCI レジストリにアップロードします。これは、ローカルの Docker イメージをレジストリにプッシュする際に、`docker push <image id="sl-md0000000">` を実行するのと同等です。

   ```bash
   cosign upload blob -f my-plugin.tar.gz docker.io/foo-user/my-plugin
   ```

   `cosign upload`コマンドは、アーティファクトが正常にアップロードされた場合にそのダイジェストを返します。
5. 手順1で生成されたキーペアを使用してアーティファクトに署名します。

   ```bash
   cosign sign --key cosign.key index.docker.io/foo-user/my-plugin@sha256:xxxxxxxxxx
   ```

   コマンドにより、プライベート鍵のパスフレーズの入力が求められる場合があります。また、署名情報が透明性ログであるRekorに永続的に記録されることに同意するかどうかを確認するメッセージが表示される場合があります。Sigstore のツールとフローの詳細情報については、[ドキュメント](https://docs.sigstore.dev/logging/overview/)をご覧ください。

次に、プラグインをインストールする必要があるマシン（ゲートウェイデータプレーンノード）で、以下の手順を実行します
（自動化することもできます）。

6. `cosign.pub`公開鍵が使用可能であることを確認します。プルするプラグインアーティファクトの署名を
   確認します：

   ```bash
   cosign verify --key cosign.pub index.docker.io/foo-user/my-plugin@sha256:xxxxxxxxxx
   ```

   アーティファクトが検証されると、コマンドは成功するはずです。
7. OSS [Crane](https://github.com/google/go-containerregistry/tree/main/cmd/crane)ツールを使用して、プラグインアーティファクトをマシンにプルします。

   ```bash
   crane pull index.docker.io/foo-user/my-plugin@sha256:xxxxxxxxxx my-downloaded-plugin.tar.gz
   ```

   このコマンドはアーティファクトをプルし、作業ディレクトリに保存する必要があります。
8. プラグインを解凍します。ダウンロードした`.tar.gz`ファイルにはマニフェストファイルと別のネストされた`.tar.gz`が含まれています。このネストされたアーカイブにはプラグインディレクトリが含まれています。

   ```bash
   tar xvf my-downloaded-plugin.tar.gz
   tar xvf xxxxxxxxxxxxxxxxxxxxx.tar.gz
   ```

9. 上記の[手動でインストール](#manually)セクションに従って、適切な場所にプラグインディレクトリにコピーします。カスタム`KONG_LUA_PACKAGE_PATH`を設定していない場合、`/usr/local/share/lua/5.1/kong/plugins`にプラグインをコピーします。

10. Kongの構成を更新してカスタムプラグインを読み込むには、`plugins=bundled,my-downloaded-plugin``kong.conf`を構成するか、`KONG_PLUGINS`環境変数を次に設定します。`plugins=bundled,my-downloaded-plugin`

トラブルシューティング
-----------

Kongは、カスタムプラグインの設定ミスにより、以下のような理由で起動に失敗することがあります：

`plugin is in use but not enabled`：別のノードからカスタムプラグインを構成し、そのプラグイン構成がデータベースにあるとしても、起動しようとする現在のノードにその`plugins`ディレクティブが含まれていません。これを解決するには、プラグインの名前をノードの`plugins`ディレクティブに追加します。

`plugin is enabled but not installed`
：プラグインの名前は`plugins`ディレクティブに存在しますが、Kongはファイルシステムから`handler.lua`ソースファイルをを読み込むことができません。解決するには、このプラグインのLuaソースをロードするよう[`lua_package_path`](/gateway/{{page.release}}/reference/configuration/#development-miscellaneous-section)ディレクティブが適切に設定されているようにしてください。

`no configuration schema found for plugin`
: プラグインは`plugins`ディレクティブでインストールされ有効化されますが、Kong はファイルシステムから `schema.lua` ソースファイルを読み込みできません。これを解決するには、`schema.lua` ファイルがプラグインの `handler.lua` ファイルと同じディレクトリに存在することを確認してください。

