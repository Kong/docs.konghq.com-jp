---
title: "カスタムエンティティの保存"
book: "plugin_dev"
chapter: 7
---
すべてのプラグインがそれを必要とするわけではありませんが、プラグインは、データベースでの構成以上のものを保存する必要があるかもしれません
。その場合、Kongは
プライマリデータストアに加えて、抽象化したものを提供し、カスタムエンティティを保存できるようにします。

[前の章]({{page.book.previous.url}})で説明したように、Kongは
"DAO"と呼ばれ、"DAOファクトリー"と呼ばれるシングルトンで利用できるクラスをとおして
モデルレイヤーと相互作用します。この章では、
独自のエンティティの抽象化を提供する方法を説明します。

モジュール
-----

    kong.plugins.<plugin_name id="sl-md0000000">.daos
    kong.plugins.<plugin_name id="sl-md0000000">.migrations.init
    kong.plugins.<plugin_name id="sl-md0000000">.migrations.000_base_<plugin_name id="sl-md0000000">
    kong.plugins.<plugin_name id="sl-md0000000">.migrations.001_<from-version id="sl-md0000000">_to_<to_version id="sl-md0000000">
    kong.plugins.<plugin_name id="sl-md0000000">.migrations.002_<from-version id="sl-md0000000">_to_<to_version id="sl-md0000000">

移行フォルダの作成
---------

モデルの定義を終えたら、Kongにより実行されエンティティの記録が
保存されるテーブルを作成する、移行モジュールを作成する必要が
あります。

{% if_version lte:3.3.x %}プラグインがCassandraとPostgreSQLの両方をサポートすることを意図している場合は、両方の移行を記述する必要があります。

{% include_cached /md/enterprise/cassandra-deprecation.md length='short' release=page.release %}
{% endif_version %}

プラグインにまだない場合は、`<plugin_name id="sl-md0000000">/migrations`フォルダをプラグインに追加します。`init.lua`ファイルがフォルダ内にない場合は、1つ作成します。
ここで、プラグインのすべての移行が参照されます。

お使いの`migrations/init.lua`ファイルの初回バージョンは単一の移行を指します。

この例では、その移行を`000_base_my_plugin`と呼びます。

```lua
-- `migrations/init.lua`
return {
  "000_base_my_plugin",
}
```

これは、初期移行を含むファイルが`<plugin_name id="sl-md0000000">/migrations/000_base_my_plugin.lua`に存在することを意味します。これがどのように行われるかは、次で説明します。

既存のプラグインに新しい移行を追加
-----------------

プラグインのバージョンがリリースされた後で、変更を導入する必要がある場合が
あります。新しい機能が必要になる場合があります。データベーステーブルの行の変更が必要となる可能性があります。

この場合は、新しい移行ファイルを作成する *必要があります* 。公開した後は既存のマイグレーションファイルを変更 *しないでください* （必要に応じて、より堅牢にすることもできます。たとえば、常に再入可能なマイグレーションを書くようにします）。

移行ファイルに命名するための厳密なルールはありませんが、最初のファイルに`000`、次のファイルに`001`というようにプレフィックスを付けるという規則があります。

前の例に続いて、データベースに変更を加えた新しいバージョンのプラグインをリリースする場合（例えば `foo` というテーブルが必要でした）、`<plugin_name id="sl-md0000000">/migrations/001_100_to_110.lua` というファイルを追加し、migrations init ファイルで次のように参照します（`100` はプラグインの以前のバージョン `1.0.0`、`110` はプラグインが `1.1.0` に移行されるバージョン）。

```lua
-- `<plugin_name id="sl-md0000000">/migrations/init.lua`
return {
  "000_base_my_plugin",
  "001_100_to_110",
}
```

{% if_version gte:3.4.x %}

移行ファイルの構文
---------

移行ファイルは、次の構造を持つテーブルを返すLuaファイルです。

```lua
-- `<plugin_name id="sl-md0000000">/migrations/000_base_my_plugin.lua`
return {
  postgres = {
    up = [[
      CREATE TABLE IF NOT EXISTS "my_plugin_table" (
        "id"           UUID                         PRIMARY KEY,
        "created_at"   TIMESTAMP WITHOUT TIME ZONE,
        "col1"         TEXT
      );

      DO $$
      BEGIN
        CREATE INDEX IF NOT EXISTS "my_plugin_table_col1"
                                ON "my_plugin_table" ("col1");
      EXCEPTION WHEN UNDEFINED_COLUMN THEN
        -- Do nothing, accept existing state
      END$$;
    ]],
  }
}

-- `<plugin_name id="sl-md0000000">/migrations/001_100_to_110.lua`
return {
  postgres = {
    up = [[
      DO $$
      BEGIN
        ALTER TABLE IF EXISTS ONLY "my_plugin_table" ADD "cache_key" TEXT UNIQUE;
      EXCEPTION WHEN DUPLICATE_COLUMN THEN
        -- Do nothing, accept existing state
      END;
    $$;
    ]],
    teardown = function(connector, helpers)
      assert(connector:connect_migrations())
      assert(connector:query([[
        DO $$
        BEGIN
          ALTER TABLE IF EXISTS ONLY "my_plugin_table" DROP "col1";
        EXCEPTION WHEN UNDEFINED_COLUMN THEN
          -- Do nothing, accept existing state
        END$$;
      ]])
    end,
  }
}
```

各ストラテジセクションには、 `up`と`teardown` 2 つの部分があります。

* `up`は、生のSQLステートメントのオプション文字列です。これらのステートメントは、
  `kong migrations up`が実行されたときに実行されます。

  新しいテーブルの作成や新規レコードの追加といったあらゆる非破壊的な操作は、`up`
  セクションで実行することをお勧めします。
* `teardown` はオプションの Lua 関数であり、`connector` パラメータを受け取ります。コネクタ
  は `query` メソッドを呼び出して SQL クエリを実行できます。ティアダウンは 
  `kong migrations finish` によって引き起こされます。

  データの削除や行タイプの変更、新規データの挿入といった破壊的な操作は、`teardown`
  セクションで実行することをお勧めします。

すべての SQLステートメントは、可能な限り再入可能になるように記述する必要があります。たとえば、 `DROP TABLE`の代わりに`DROP TABLE IF EXISTS`を使用し、`CREATE INDEX`の代わりに`CREATE INDEX IF NOT EXIST`を使用するなど。移行が何らかの理由で失敗した場合、問題解決での最初の試行は、単純に移行のやりなおしとなることが考えられます。

{:.important}
> 
> `schema`で`unique`制約を使用している場合は、PostgreSQLの移行でこの制約を設定する必要があります。

実際の例を確認するには、[Key\-Auth プラグインの移行](https://github.com/Kong/kong/tree/master/kong/plugins/key-auth/migrations/)を参照してください。

{% endif_version %}

{% if_version lte:3.3.x %}

移行ファイルの構文
---------

Kongのコア移行はPostgresとCassandraの両方をサポートしますが、カスタムプラグインを選択して両方またはいずれかのみをサポートすることができます。

移行ファイルは、次の構造を持つテーブルを返すLuaファイルです。

```lua
-- `<plugin_name id="sl-md0000000">/migrations/000_base_my_plugin.lua`
return {
  postgres = {
    up = [[
      CREATE TABLE IF NOT EXISTS "my_plugin_table" (
        "id"           UUID                         PRIMARY KEY,
        "created_at"   TIMESTAMP WITHOUT TIME ZONE,
        "col1"         TEXT
      );

      DO $$
      BEGIN
        CREATE INDEX IF NOT EXISTS "my_plugin_table_col1"
                                ON "my_plugin_table" ("col1");
      EXCEPTION WHEN UNDEFINED_COLUMN THEN
        -- Do nothing, accept existing state
      END$$;
    ]],
  },

  cassandra = {
    up = [[
      CREATE TABLE IF NOT EXISTS my_plugin_table (
        id          uuid PRIMARY KEY,
        created_at  timestamp,
        col1        text
      );

      CREATE INDEX IF NOT EXISTS ON my_plugin_table (col1);
    ]],
  }
}

-- `<plugin_name id="sl-md0000000">/migrations/001_100_to_110.lua`
return {
  postgres = {
    up = [[
      DO $$
      BEGIN
        ALTER TABLE IF EXISTS ONLY "my_plugin_table" ADD "cache_key" TEXT UNIQUE;
      EXCEPTION WHEN DUPLICATE_COLUMN THEN
        -- Do nothing, accept existing state
      END;
    $$;
    ]],
    teardown = function(connector, helpers)
      assert(connector:connect_migrations())
      assert(connector:query([[
        DO $$
        BEGIN
          ALTER TABLE IF EXISTS ONLY "my_plugin_table" DROP "col1";
        EXCEPTION WHEN UNDEFINED_COLUMN THEN
          -- Do nothing, accept existing state
        END$$;
      ]])
    end,
  },

  cassandra = {
    up = [[
      ALTER TABLE my_plugin_table ADD cache_key text;
      CREATE INDEX IF NOT EXISTS ON my_plugin_table (cache_key);
    ]],
    teardown = function(connector, helpers)
      assert(connector:connect_migrations())
      assert(connector:query("ALTER TABLE my_plugin_table DROP col1"))
    end,
  }
}
```

プラグインがPostgresまたはCassandraのみをサポートしている場合、1つのストラテジのセクションのみが
必要となります。各ストラテジのセクションには、`up`と`teardown`の2つの部分があります。

* `up` は、生のSQL/CQLステートメントのオプションの文字列です。これらのステートメントは、`kong migrations up`が実行されたときに実行されます。
* `teardown` はオプションの Lua 関数であり、`connector` パラメータを受け取ります。このようなコネクタ は `query` メソッドを呼び出して SQL/CQL クエリを実行できます。ティアダウンは以下によって引き起こされます `kong migrations finish`

新しいテーブルの作成や、新しいレコード追加などのすべての非破壊操作は、 `up` セクションで行い、破壊操作（データの削除、行タイプの変更、新しいデータの挿入など）
は `teardown` セクションで行うことが推奨されています。

どちらの場合も、すべてのSQL/CQLステートメントは可能な限り再入可能な形で記述することが
推奨されます。`DROP TABLE`の代わりに`DROP TABLE IF EXISTS`、
`CREATE INDEX`の代わりに`CREATE INDEX IF NOT EXIST`など。移行が何らかの理由で
失敗した場合、問題解決での最初の試行は、単純に移行のやりなおしとなることが
考えられます。

PostgreSQLはサポートしていますがCassandraは「NOT NULL」、「UNIQUE」、「FOREIGN KEY」などの制約をサポートしておらず、Kongはモデルのスキーマを定義するときにそのような機能を提供します。このスキーマはPostgreSQLとCassandraで同じであるため、純粋なSQLスキーマとCassandraで動作するスキーマをトレードオフする可能性があることに注意してください。

{:.important}
> 
> `schema`で`unique`制約を使用している場合、KongはCassandraではこれを強制しますが、PostgreSQLの場合は、移行でこの制約を設定する必要があります。

実際の例を確認するには、[Key\-Auth プラグインの移行](https://github.com/Kong/kong/tree/master/kong/plugins/key-auth/migrations/)を参照してください。

{% endif_version %}

スキーマを定義する
---------

カスタムプラグインでカスタムエンティティを使用する最初のステップは、1つまたは複数の
*スキーマ* を定義することです。

スキーマはエンティティを記述するLuaテーブルです。たとえば、エンティティのさまざまなフィールドの名前やタイプは何か、などの構造情報があります。
これは、[プラグイン設定]({{page.book.chapters.plugin-configuration}})を説明するフィールドに似ています。
プラグイン設定スキーマと比較すると、カスタムエンティティスキーマには
追加のメタデータが必要です（たとえば、どのフィールドがエンティティのプライマリキーを構成するかなど）。

スキーマは、次の名前のモジュールで定義します。

    kong.plugins.<plugin_name id="sl-md0000000">.daos

つまり、プラグインフォルダ内に `<plugin_name id="sl-md0000000">/daos.lua` というファイルがあるはずです。`daos.lua` ファイルは、1 つ以上のスキーマを含むテーブルを返します。以下に例を示します。

```lua
-- daos.lua
local typedefs = require "kong.db.schema.typedefs"


return {
  -- this plugin only results in one custom DAO, named `keyauth_credentials`:
  {
    name                  = "keyauth_credentials", -- the actual table in the database
    endpoint_key          = "key",
    primary_key           = { "id" },
    cache_key             = { "key" },
    generate_admin_api    = true,
    admin_api_name        = "key-auths",
    admin_api_nested_name = "key-auth",
    fields = {
      {
        -- a value to be inserted by the DAO itself
        -- (think of serial id and the uniqueness of such required here)
        id = typedefs.uuid,
      },
      {
        -- also interted by the DAO itself
        created_at = typedefs.auto_timestamp_s,
      },
      {
        -- a foreign key to a consumer's id
        consumer = {
          type      = "foreign",
          reference = "consumers",
          default   = ngx.null,
          on_delete = "cascade",
        },
      },
      {
        -- a unique API key
        key = {
          type      = "string",
          required  = false,
          unique    = true,
          auto      = true,
        },
      },
    },
  },
}
```

この例の`daos.lua`ファイルでは、 `keyauth_credentials`と呼ばれる単一のスキーマを導入しています。

以下は、最上位プロパティの説明です。
<table> <tbody> <tr><th>名称</th><th>タイプ</th><th>説明</th></tr> <tr> <td><code translate="no">name</code></td> <td><code translate="no">string</code>（必須）</td> <td>DAO名（<code translate="no">kong.db.\[name\]</code>）の決定に使用されます。</td> </tr> <tr> <td><code translate="no">primary_key</code></td> <td><code translate="no">table</code>（必須）</td> <td> エンティティのプライマリキーを形成するフィールド名。 スキーマは、ほとんどのKongコアエンティティが<code translate="no">id</code>という名前のUUIDを使用している場合でも、複合キーをサポートします。 {% if_version lte:3.3.x %} Cassandraを使用していて複合キーが必要な場合は、同じ フィールドをパーティションキーとして使用します。 {% endif_version %} </td> </tr> <tr> <td><code translate="no">endpoint_key</code></td> <td><code translate="no">string</code>\(オプション\)</td> <td> Admin APIで代替識別子として使用されるフィールドの名前。上の例では<code translate="no">key</code>が endpoint\_key です。つまり、<code translate="no">id = 123</code> と <code translate="no">key = "foo"</code> を持つ認証情報は、<code translate="no">/keyauth_credentials/123</code> と <code translate="no">/keyauth_credentials/foo</code> の両方として参照できます。</td> </tr> <tr> <td><code translate="no">cache_key</code></td> <td><code translate="no">table</code>\(オプション\)</td> <td><code translate="no">cache_key</code>の生成に使用されるフィールドの名前が含まれ、Kongのキャッシュ内のエンティティを明確に識別する必要がある文字列です。通常、例の<code translate="no">key</code>のような一意のフィールドが適切な候補となります。その他の場合には、複数のフィールドを組み合わせることが望ましいです。</td> </tr> <tr> <td><code translate="no">generate_admin_api</code></td> <td><code translate="no">boolean</code>\(オプション\)</td> <td>エンティティの管理apiを自動生成するかどうか。デフォルトでは、admin apiはカスタムのものを含むすべてのdaoに対して生成されます。dao用に完全にカスタマイズされた管理apiを作成する場合、またはdaoの自動生成を完全に無効にする場合は、このオプションを <code translate="no">false</code> に設定します。</td> </tr> <tr> <td><code translate="no">admin_api_name</code></td> <td><code translate="no">boolean</code>\(オプション\)</td> <td> <code translate="no">generate_admin_api</code>が有効になっている場合、admin api 自動ジェネレーターは、自動生成された管理apiのコレクションURLを取得するために<code translate="no">name</code>を使用します。コレクションURLに<code translate="no">name</code>とは異なる別の名前を付けたい場合もあるでしょう。例えば、DAO <code translate="no">keyauth_credentials</code>においては、実際に、自動生成ツールがこのDAOのためのエンドポイントを、別の、より分かりやすいURL名<code translate="no">key-auths</code>で生成してくれることを期待していました。（例：<code translate="no">http://<KONG_ADMIN>/keyauth_credentials</code>ではなく<code translate="no">http://<KONG_ADMIN>/key-auths</code>など）。 </td> </tr> <tr> <td><code translate="no">admin_api_nested_name</code></td> <td><code translate="no">boolean</code>\(オプション\)</td> <td><code translate="no">admin_api_name</code>と同様に、<code translate="no">admin_api_nested_name</code>は、admin api 自動ジェネレータがネストされたコンテキストで作成する dao の名前を指定します。このパラメーターは、<code translate="no">name</code>や<code translate="no">admin_api_name</code>に満足できない場合にのみ使用する必要があります。レガシー上の理由から、Kongには<code translate="no">key-auth</code>が複数形の<code translate="no">http://<KONG_ADMIN>/key-auths</code>に従わない<code translate="no">http://<KONG_ADMIN>/consumers/john/key-auth</code>のようなURLがあります。<code translate="no">admin_api_nested_name</code> では、このような場合に別の名前を指定できます。</td> </tr> <tr> <td><code translate="no">fields</code></td> <td><code translate="no">table</code></td> <td> 各フィールド定義は単一のキーを持つテーブルで、これはフィールド名です。テーブルの値は、 フィールドの<em>属性</em>を含むサブテーブルを含んでおり、その一部については後述します。 </td> </tr> </tbody> </table> 

多くのフィールド属性は、 *検証ルール* をエンコードします。DAOを使用してエンティティを挿入または更新しようとすると、これらの検証がチェックされ、指定された入力がそれらに適合しない場合はエラーが返されます。

`typedefs` 変数（`kong.db.schema.typedefs` を要求することで取得）は、プライマリキーの最も一般的な型である `typedefs.uuid` や `created_at` フィールドの `typedefs.auto_timestamp_s` など、多くの便利な型定義と別名を含むテーブルです。これは、フィールドを定義するときに広く使用されます。

以下は、使用可能ないくつかのフィールド属性の説明の抜粋です。
<table> <tbody> <tr><th>属性名</th><th>タイプ</th><th>説明</th></tr> <tr> <td><code translate="no">type</code></td> <td><code translate="no">string</code></td> <td>スキーマは、<code translate="no">"string"</code>、<code translate="no">"integer"</code>、<code translate="no">"number"</code>、<code translate="no">"boolean"</code> のスカラー型をサポートします。<code translate="no">"array"</code>、<code translate="no">"record"</code>、<code translate="no">"set"</code> などの複合型もサポートされています。<br><br> 

    In addition to these values, the <code id="sl-md0000000">type</code> attribute can also take the special <code id="sl-md0000000">"foreign"</code> value,
    which denotes a foreign relationship.<br id="sl-md0000000"><br id="sl-md0000000">
    
    Each field will need to be backed by database fields of appropriately similar types, created via migrations.<br id="sl-md0000000"><br id="sl-md0000000">
    
    <code id="sl-md0000000">type</code> is the only required attribute for all field definitions.

</td> </tr> <tr> <td><code translate="no">default</code></td> <td><code translate="no">any</code>（<code translate="no">type</code>属性との一致）</td> <td> 値が指定されていない場合に、フィールドを挿入しようとしたときにフィールドが持つ値を指定します。デフォルト値は常にLua経由で設定され、基盤となるデータベースによって設定されることはありません。したがって、移行のフィールドにデフォルト値を設定することはお勧めしません</td> </tr> <tr> <td><code translate="no">required</code></td> <td><code translate="no">boolean</code></td> <td> フィールド上で<code translate="no">true</code>を設定する際に、特定のフィールドに値がないエンティティを挿入しようとするとエラーが発生します（該当フィールドにデフォルト値がある場合を除く）。</td> </tr> <tr> <td><code translate="no">unique</code></td> <td><code translate="no">boolean</code></td> <td> <p>フィールドが<code translate="no">true</code> に設定される場合にデータベースにエンティティを挿入しようとすると、エラーがスローされますが、別のエンティティで既にそのフィールドに指定済みの値があります。</p> <p>この属性は、PostgreSQLを使用するときに、マイグレーションでフィールドを <code translate="no">UNIQUE</code> として宣言することでバックアップする<em>必要があります</em>。{% if_version lte:3.3.x %} Cassandra ストラテジは、挿入を試みる前にLuaでチェックを行うため、特別な処理は必要ありません。{% endif_version %} </p> </td> </tr> <tr> <td><code translate="no">auto</code></td> <td><code translate="no">boolean</code></td> <td> <code translate="no">auto</code> が <code translate="no">true</code> に設定されているフィールドに値を指定せずにエンティティを挿入しようとすると、 <br><br> <ul> <li><code translate="no">type == "uuid"</code>の場合、フィールドはランダムなUUIDを値として受け取ります。</li> <li><code translate="no">type == "string"</code>の場合、フィールドはランダムな文字列を受け入れます。</li> <li>フィールド名が<code translate="no">created_at</code>または<code translate="no">updated_at</code> である場合、このフィールドを適宜挿入または更新する際、現在時刻が入ります。</li> </ul> </td> </tr> <tr> <td><code translate="no">reference</code></td> <td><code translate="no">string</code></td> <td>タイプが<code translate="no">foreign</code>のフィールドには必須です。指定する文字列は、<em>必ず</em>既存スキーマの名前にし、外部キーはこの名前を「指す」ことになります。つまり、スキーマBにスキーマAを指す外部キーがある場合は、Bの前にAが読み込まれる必要があります。 </td> </tr> <tr> <td><code translate="no">on_delete</code></td> <td><code translate="no">string</code></td> <td>オプションで、<code translate="no">foreign</code>タイプのフィールド専用です。これは、参照されたエンティティが削除された場合に、外部キーによってリンクされたエンティティに何が起こるべきかを指示します。これには3つの潜在的な値があります：<br><br> 

    <ul id="sl-md0000000">
      <li id="sl-md0000000"><code id="sl-md0000000">"cascade"</code>: When the linked entity is deleted, all the dependent entities must also be deleted.</li>
      <li id="sl-md0000000"><code id="sl-md0000000">"null"</code>: When the linked entity is deleted, all the dependent entities will have their foreign key
      field set to <code id="sl-md0000000">null</code>.</li>
      <li id="sl-md0000000"><code id="sl-md0000000">"restrict"</code>: Attempting to delete an entity with linked entities will result in an error.</li>
    </ul>
    
    <br id="sl-md0000000"><br id="sl-md0000000">
    {% if_version lte:3.3.x %}
    In Cassandra this is handled with pure Lua code, but in PostgreSQL it will be necessary to declare the references
    as <code id="sl-md0000000">ON DELETE CASCADE/NULL/RESTRICT</code> in a migration.
    {% endif_version %}
    {% if_version gte:3.4.x %}
    In PostgreSQL, you must declare the references
    as <code id="sl-md0000000">ON DELETE CASCADE/NULL/RESTRICT</code> in a migration.
    {% endif_version %}

</td> </tr> </tbody> </table> 

スキーマの詳細については、以下を参照してください。

* [typedefs.lua](https://github.com/Kong/kong/blob/release/3.0.x/kong/db/schema/typedefs.lua)のソースコードで、デフォルトで何が提供されているかを把握します。
* [コアスキーマ](https://github.com/Kong/kong/tree/release/3.0.x/kong/db/schema/entities)では、ここで説明されていない他のフィールド属性の例を確認できます。
* [組み込みプラグイン用のすべての `daos.lua` ファイル](https://github.com/search?utf8=%E2%9C%93&q=repo%3Akong%2Fkong+path%3A%2Fkong%2Fplugins+filename%3Adaos.lua)、特にこのガイドで例として使用された[key\-auth ファイル](https://github.com/Kong/kong/blob/release/3.0.x/kong/plugins/key-auth/daos.lua)。

カスタムDAO
-------

スキーマは、データベースとやり取りするために直接使用されるわけではありません。代わりに、有効なスキーマごとにDAOが構築されます。DAOはそれがラップするスキーマの名前を取り、
`kong.db`インターフェースを通じてアクセス可能です。

上記の例のスキーマでは、生成されたDAOは`kong.db.keyauth_credentials`経由のプラグインで利用可能になります。

### エンティティを選択します

```lua
local entity, err, err_t = kong.db.<name id="sl-md0000000">:select(primary_key)
```

データベース内のエンティティを見つけて返そうとします。次の3つのことが起こり得ます。

* エンティティが見つかりました。この場合、通常のLuaテーブルとして返されます。
* エラーが発生します。たとえば、データベースとの接続が失われるなどです。その場合、 最初に返される値は`nil`で、2番目はエラーを説明する文字列、 最後は表形式の同じエラーになります。
* エラーは発生しませんが、エンティティが見つかりません。その後、関数はエラーなしで`nil`を返します。

使用例：

```lua
local entity, err = kong.db.keyauth_credentials:select({
  id = "c77c50d2-5947-4904-9f37-fa36182a71a9"
})

if err then
  kong.log.err("Error when inserting keyauth credential: " .. err)
  return nil
end

if not entity then
  kong.log.err("Could not find credential.")
  return nil
end
```

### すべてのエンティティを反復処理

```lua
for entity, err on kong.db.<name id="sl-md0000000">:each(entities_per_page) do
  if err then
    ...
  end
  ...
end
```

このメソッドは、リクエストページ分割を行うことで、データベース内のすべてのエンティティを効率的に
反復処理します。`entities_per_page`パラメータ（`100`のデフォルト）は、
ページごとに返されるエンティティ数を制御します。

反復ごとに、新しい `entity` が返されます。エラーがある場合は、`err` 変数にエラーが格納されます。推奨する反復方法は、最初に `err` をチェックし、それ以外の場合は `entity` が存在すると仮定することです。

使用例：

```lua
for credential, err on kong.db.keyauth_credentials:each(1000) do
  if err then
    kong.log.err("Error when iterating over keyauth credentials: " .. err)
    return nil
  end

  kong.log("id: " .. credential.id)
end
```

この例では、認証情報を 1000 アイテムごとのページで繰り返し処理し、エラーが発生しない限り
IDを記録します。

### エンティティの挿入

```lua
local entity, err, err_t = kong.db.<name id="sl-md0000000">:insert(<values id="sl-md0000000">)
```

データベースにエンティティを挿入し、挿入されたエンティティのコピー、または、`nil`、エラーメッセージ（文字列）とテーブルフォームのエラーを説明するテーブルを返します。

挿入が成功すると、返されるエンティティには、
`default`と`auto` によって提供される追加の値が含まれます。

次の例では、`keyauth_credentials` DAOを使用して、特定のコンシューマに認証情報を挿入し、`key`を`"secret"`
に設定しています。外部キーの参照用構文に注目してください。

```lua
local entity, err = kong.db.keyauth_credentials:insert({
  consumer = { id = "c77c50d2-5947-4904-9f37-fa36182a71a9" },
  key = "secret",
})

if not entity then
  kong.log.err("Error when inserting keyauth credential: " .. err)
  return nil
end
```

返されたエンティティ（エラーが発生しなかったと仮定して）には、`id`と`created_at`のような`auto`入力されたフィールドが含まれます。

### エンティティを更新

```lua
local entity, err, err_t = kong.db.<name id="sl-md0000000">:update(primary_key, <values id="sl-md0000000">)
```

指定された主キーと値のセットを使用して既存のエンティティが見つかる場合に、そのエンティティを更新します。

返されるエンティティは、更新が行われた後のエンティティ、または `nil` \+ エラーメッセージ \+ エラーテーブルになります。

次の例では、認証情報の ID を指定して、既存の認証情報の `key` フィールドを変更します。

```lua
local entity, err = kong.db.keyauth_credentials:update(
  { id = "2b6a2022-770a-49df-874d-11e2bf2634f5" },
  { key = "updated_secret" },
)

if not entity then
  kong.log.err("Error when updating keyauth credential: " .. err)
  return nil
end
```

プライマリキーを指定するための構文が、外部キーを指定するために使用する構文と似ていることに注意してください。

### エンティティのアップサート

```lua
local entity, err, err_t = kong.db.<name id="sl-md0000000">:upsert(primary_key, <values id="sl-md0000000">)
```

`upsert``insert`と`update`が混在している。

* 提供された`primary_key`既存のエンティティを識別すると、 `update`のように機能します。
* 提供された`primary_key`が既存のエンティティを識別しない場合は、次のように動作します。`insert`

次のコードがあるとします。

```lua
local entity, err = kong.db.keyauth_credentials:upsert(
  { id = "2b6a2022-770a-49df-874d-11e2bf2634f5" },
  { consumer = { id = "a96145fb-d71e-4c88-8a5a-2c8b1947534c" } }
)

if not entity then
  kong.log.err("Error when upserting keyauth credential: " .. err)
  return nil
end
```

次の 2 つのことが起こり得ます。

* IDが `2b6a2022-770a-49df-874d-11e2bf2634f5` の認証情報が存在する場合、このコードはそのコンシューマを指定されたものに設定しようとします。
* 認証情報が存在しない場合、このコードは、指定された id とコンシューマを使用して新しい認証情報の作成を試行します。

### エンティティの削除

```lua
local ok, err, err_t = kong.db.<name id="sl-md0000000">:delete(primary_key)
```

`primary_key` で識別されるエンティティの削除を試行します。このメソッドを呼び出した後にエンティティが *存在しない* 場合は `true` を返し、エラーが検出された場合は `nil` \+ error \+ errorテーブルを返します。

`delete`の呼び出しは、 *呼び出す前に* エンティティが不在であれば、正常に実行されることに注意してください。
その理由はパフォーマンスにあり、削除前に読み込み操作を実行することをできるだけ回避するためです。
このチェックは手動で実行する必要があり、`delete`を呼び出す前に`select`を使用してチェックします。

例：

```lua
local ok, err = kong.db.keyauth_credentials:delete({
  id = "2b6a2022-770a-49df-874d-11e2bf2634f5"
})

if not ok then
  kong.log.err("Error when deleting keyauth credential: " .. err)
  return nil
end
```

カスタムエンティティのキャッシュ
----------------

リクエスト/応答ごとにカスタムエンティティが必要になることがあります。
その結果、データストアのクエリが毎回発生します。データストアをクエリするとレイテンシが増加し、リクエスト/応答が遅くなるため、これは非常に非効率的です。
結果、データストアの負荷が増加し、データストアのパフォーマンスそのものや、
ほかのKongノードにまで影響を及ぼす可能性があります。

すべてのリクエスト/レスポンスにカスタムエンティティが必要な場合は、
Kong が提供するメモリ内キャッシュ API を活用してメモリ内にキャッシュするのが良い方法です。

次の章では、カスタムエンティティのキャッシュと、それらがデータストアで変更された場合に無効にする方法について説明します：[カスタムエンティティのキャッシュ]({{page.book.next.url}})。

