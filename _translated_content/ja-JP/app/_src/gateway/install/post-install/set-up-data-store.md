---
title: "データストアの設定"
content_type: "how-to"
---

{{site.base_gateway}}には、`/etc/kong/kong.conf.default`にあるデフォルトの構成プロパティファイルが付属しています。
この構成ファイルは、起動時に{{site.base_gateway}}の構成プロパティを
設定するために使用されます。


{{site.base_gateway}} には、
{{site.base_gateway}} のすべての構成済みエンティティの構成プロパティを保存するための2つのオプションがあります。1 つはデータベース （PostgreSQL）を使用する方法で、もう 1つは DB レスモードとも呼ばれる YAML 宣言型構成ファイルを使用する方法です。

{{site.base_gateway}}を開始する前に、データストアを参照して、`kong.conf.default`構成プロパティファイルを更新する必要があります。

{:.note}
> 
> **注** : 次のデータストアの設定は、データストアなしで {{site.base_gateway}} をインストールした場合にのみ必要です。
> 当社が提供する Docker、Kubernetes、またはクイックスタートのいずれかのガイドに従った場合は、データストアがすでに構成されているはずです。

データストアの設定
---------

`kong.conf.default`ファイルにリストされているデフォルトのプロパティを変更して{{site.base_gateway}}を構成するには、ファイルのコピーを作成して、名前を変更し（たとえば`kong.conf`）、更新して同じロケーションに保存します。

次に、次のいずれかのオプションを選択してデータストアを設定します。

* [データベースの使用](#using-a-database)
* [YAML 宣言型構成ファイルの使用](#using-a-yaml-declarative-config-file)（DB レス）

### データベースの使用

`kong.conf.default` ファイルを使用して {{site.base_gateway}} を構成し、データベースに接続できるようにします。
関連するすべての構成パラメータについては、[構成プロパティリファレンス](/gateway/{{ page.release }}/reference/configuration/#datastore-section)のデータストアのセクションを参照してください。

{% if_version gte:2.7.x lte:3.3.x %}

{% include_cached /md/enterprise/cassandra-deprecation.md length='short' release=page.release %}

{% endif_version %}

{{site.base_gateway}} を開始する前に、 [PostgreSQL](http://www.postgresql.org/)データベースとユーザーをプロビジョニングします。

```sql
CREATE USER kong WITH PASSWORD 'super_secret'; CREATE DATABASE kong OWNER kong;
```

次の{{site.base_gateway}}マイグレーションのいずれかを実行します。

{% navtabs %}
{% navtab Kong Gateway Enterprise %}

Enterprise環境では`kong migrations`コマンドを使用して、スーパー管理者ユーザーのパスワードをシード値で設定することを強くお勧めします。
設定しておくと、後で必要になったときに、RBAC（ロールベースのアクセス制御）を使用できます。

希望するパスワードを含む環境変数を作成し、そのパスワードを安全な場所に保存します。

```bash
KONG_PASSWORD=<password id="sl-md0000000"> kong migrations bootstrap -c <kong_declarative_config_path id="sl-md0000000">
```

{:.important}
> 
> **重要** : Kong パスワード（`KONG_PASSWORD`）に 4 つのティックを含む値を設定すると
> （たとえば、`KONG_PASSWORD="a''a'a'a'a"`）ブートストラップ時に PostgreSQL 構文エラーを引き起こします。
> パスワードに特殊文字を使用しないことで、この問題を回避できます。

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

Enterpriseを使用していない場合は、以下を実行します。

```bash
kong migrations bootstrap -c <kong_declarative_config_path id="sl-md0000000">
```

{% endnavtab %}
{% endnavtabs %}

PostgreSQLの古いバージョンでは、デフォルトで`ident`認証が使用されますが、新しいバージョン（PSQL 10以降）では
`scram-sha-256`が使用されます。`kong`ユーザーがローカルでデータベースと通信できるようにするには、
PostgreSQL構成ファイルを変更して、認証方法を`md5`に変更します。

### YAML宣言型構成ファイルの使用

データベースを使用する代わりに、すべての{{site.base_gateway}}の構成プロパティを、YAML宣言型構成ファイル内で
構成されたエンティティに保存することもできます（DB\-lessモードとも呼ばれます）。

`kong.yml`ファイルを作成し、 `kong.conf`構成ファイルを更新して、 `kong.yml`ファイルへのファイルパスを含めます。

1. 現在のフォルダに `kong.yml` 宣言型構成ファイルを生成します。

   ```bash
   kong config init
   ```

   生成された`kong.yml`ファイルには、そのファイルを使用して{{site.base_gateway}}を構成する手順が含まれています。
2. `kong.conf`で、`database`オプションを`off`に設定し、`declarative_config`ディレクティブを
   `kong.yml`へのパスに設定します。

       database = off
       declarative_config = <kong_declarative_config_path id="sl-md0000000">

{{site.base_gateway}} を起動
----------

{% include_cached /md/gateway/root-user-note.md release=page.release %}

次のコマンドを使用して{{site.base_gateway}}を起動します。

    kong start -c <path_to_kong.conf_file id="sl-md0000000">

インストール状態の確認
-----------

すべてがうまくいけば、 {{site.base_gateway}}が実行中であることを通知するメッセージ \(`Kong started`\) が表示されます。

Admin APIを使用して確認することもできます。

    curl -i http://localhost:8001

`200`ステータスコードが表示されます。

関連情報と次の手順
---------

目的の環境に応じて、次のガイドを参照してください。

* [Enterpriseライセンスの追加](/gateway/{{ page.release }}/licenses/deploy)
{% if_version gte:3.4.x -%}
* Kong Managerを有効にします。
  * [Kong Manager Enterprise](/gateway/{{ page.release }}/kong-manager/enable/)
  * [Kong Manager OSS](/gateway/{{ page.release }}/kong-manager-oss/)
{% endif_version -%}
{% if_version lte:3.3.x -%}

* [Kong Managerを有効化](/gateway/{{ page.release }}/kong-manager/enable/)
{% endif_version -%}
* [デフォルトのポート参照](/gateway/{{page.release}}/production/networking/default-ports/)

{{site.base_gateway}}のシリーズもチェックしてみてください
[入門](/gateway/{{page.release}}/get-started/)ガイドで方法を学ぶ
{{site.base_gateway}}を最大限に活用しましょう。

