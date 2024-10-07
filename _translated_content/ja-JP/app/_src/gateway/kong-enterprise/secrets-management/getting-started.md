---
title: "シークレット管理を始める"
---
シークレットは一般的に機密の値であり、アプリケーションにプレーンテキストで表示してはいけません。
こうしたシークレットを安全に保管、取得、ローテーションし、
アクセスを監視する製品が存在します。{{site.base_gateway}}は、シークレットへの参照を設定するメカニズムを提供し、
{{site.base_gateway}}インストールの安全性が向上します。

使用開始
----

次の例では、最も基本的な形式のシークレット管理、つまり環境変数にシークレットを保存する方法を使用します。
この例では、PostgreSQL データベースへのプレーンテキストパスワードを環境変数への参照に置き換えます。

安全なVaultバックエンドにシークレットを保存することもできます。サポートされているVaultバックエンド実装の一覧については、[「バックエンドの概要」](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/)を参照してください。

この例では、参照を使ってPostgreSQLデータベースへのプレーンテキストのパスワードを置き換えます。これを行うには、環境変数を定義し、それにシークレット値を割り当てます。

### リファレンスの定義

環境変数を定義し、シークレット値を割り当てます。

```bash
export MY_SECRET_POSTGRES_PASSWORD="opensesame"
```

次に、{{site.base_gateway}}がこのシークレットを見つけられるように、この環境変数への参照を設定します。これには、Uniform Resource Locator（URL）形式を使用します。

この場合、リファレンスは次のようになります。

```bash
{vault://env/my-secret-postgres-password}
```

この場合、以下の通りです。

* `vault`これがシークレットであることを示すために使用するスキーム（プロトコル）です。
* `env`はバックエンドの名前[（環境変数）](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/env/)で、シークレットをENV変数に格納しているためこれを使用します。
* `my-secret-postgres-password`は、定義した環境変数に対応します。

リファレンスは波括弧で囲まれていることに注意してください。

### 参照の使用

構成オプションを指定するには、環境変数を使用するか、`kong.conf`構成ファイルを直接更新できます。

#### 環境変数

`KONG_PG_PASSWORD`環境変数を次のように設定し、 `my-secret-postgres-password`を独自のパスワードオブジェクト名に調整します。

```bash
KONG_PG_PASSWORD={vault://env/my-secret-postgres-password}
```

#### 構成ファイル

`kong.conf`ファイルにはプロパティ`pg_password` が含まれています。元の値を次の値に置き換え、`my-secret-postgres-password`を独自のパスワードオブジェクト名に調整します。

```bash
pg_password={vault://env/my-secret-postgres-password}
```

{{site.base_gateway}} は起動時に、リファレンスを検出して透過的に解決しようとします。

{:.note}
> 
> 迅速なデバッグやテストを行うには、[Vault 用の CLI](/gateway/{{page.release}}/kong-enterprise/secrets-management/advanced-usage/#vaults-cli) を使用できます。

各Vaultバックエンドの設定オプションの詳細については、[高度な使用法](/gateway/{{page.release}}/kong-enterprise/secrets-management/advanced-usage/)のドキュメントを参照してください。

