---
title: "Dev Portal CLI"
badge: "enterprise"
---
Kong Dev Portal CLI は、コマンドラインから Dev Portal を管理するために使用されます。Kong Dev Portal CLI は、[clipanion](https://github.com/arcanis/clipanion)を使用して構築されています。

これは次世代のTypeScriptベースのDev Portal CLI です。このプロジェクトの目的は、初期同期スクリプトよりも高品質のCLI ツールを作成することです。

インストール
------

    > npm install -g kong-portal-cli

使用方法
----

最も簡単な開始方法は、マスターブランチの[ポータル・テンプレート・リポジトリ](https://github.com/Kong/kong-portal-templates)のクローンを
ローカルに作成することです。

次に、お使いの設定に合うように `workspaces/default/cli.conf.yaml` を編集してワークスペース `kong_admin_url` と `kong_admin_token` を設定します。

`workspaces/default/cli.conf.yaml`の設定値は、環境変数の
`KONG_ADMIN_URL`と`KONG_ADMIN_TOKEN`を使用して上書きすることも可能です。

Kongが実行中で、ポータルがオンになっていることを確認してください。

これで、テンプレートリポジトリのルートフォルダから、次のコマンドを実行できます。

`portal [-h,--help] [--config PATH] [-v,--verbose] <command id="sl-md0000000"> <workspace id="sl-md0000000">`

`<command id="sl-md0000000">`は次のいずれかです。

* `config` 指定された `workspace` のポータルの構成をローカルで出力または変更します。
* `deploy` 指定されたワークスペースのアップストリームでローカルに行われた変更をデプロイします。
* `disable` 指定したワークスペースのポータルを無効にします。
* `enable` 指定したワークスペースのポータルを有効にします。
* `fetch` 指定されたワークスペースからコンテンツとテーマを取得します。
* `wipe` 上流のワークスペースからすべてのコンテンツとテーマを削除します

ここで、 `<workspace id="sl-md0000000">`操作対象となるディレクトリ/ワークスペースのペアを示します。

環境変数を設定して、 `workspaces/default/cli.conf.yaml`ファイル内の値を上書きすることができます。

```sh
KONG_ADMIN_URL=<kong_admin_base_url id="sl-md0000000"> \
KONG_ADMIN_TOKEN=<kong_admin_token id="sl-md0000000"> \
portal [-h,--help] [--config PATH] [-v,--verbose] <command id="sl-md0000000"> <workspace id="sl-md0000000">
```

`<kong_admin_base_url id="sl-md0000000">`がある場所は開発者ポータルを管理するKong Admin APIのロケーションであり、 `<kong_admin_token id="sl-md0000000">`はポータル管理のためのアクセス権を持つRBACトークンです。

#### コマンド: `deploy`

* `-W`または`--watch`を追加して、変更をリアクティブにします。
* ローカルに存在しないアップストリームのファイルを削除しないようにするには、`-P`または`--preserve`を追加します。
* SSL検証を無効にして自己署名証明書を使用するには、`-D`または`--disable-ssl-verification`を追加します。
* `/specs`ディレクトリを無視するには、 `-I`または`--ignore-specs`を追加します。

#### コマンド: `fetch`

* `-K`または`--keep-encode`を追加して、バイナリアセットをbase64でエンコードされた文字列としてローカルに保持します。
* SSL検証を無効にして自己署名証明書を使用するには、`-D`または`--disable-ssl-verification`を追加します。
* `/specs`ディレクトリを無視するには、 `-I`または`--ignore-specs`を追加します。

#### コマンド: `wipe`

* SSL検証を無効にして自己署名証明書を使用するには、`-D`または`--disable-ssl-verification`を追加します。
* `/specs`ディレクトリを無視するには、 `-I`または`--ignore-specs`を追加します。

#### `enable`と`disable`

* SSL検証を無効にして自己署名証明書を使用するには、`-D`または`--disable-ssl-verification`を追加します。

