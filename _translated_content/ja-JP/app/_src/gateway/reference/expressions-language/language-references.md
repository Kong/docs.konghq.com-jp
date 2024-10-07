---
title: "言語リファレンス"
content-type: "reference"
---
このリファレンスでは、式ルーターに使用される式言語構造の各部分について説明します。

述部
---

述部は、次の形式をとる式コードの基本単位です。

    http.path ^= "/foo/bar"

この述部の例は、次の構造に従っています。

* `http.path`: フィールド
* `^=`：演算子
* `"/foo/bar"`: 定数値

型システム
-----

式言語は厳密に型指定されています。このような操作が実際の
フィールドと定数の型に対して意味がある場合にのみ、操作が実行されます。

実行時の型変換は、明示的または黙示的にもサポートされていません。指定された
フィールドと定数で演算子を実行できない場合、型はルートが解析され、
エラーが返される時点で常に認識されています。

現在、次のタイプの式の言語をサポートしています。

|    型     |                                       説明                                       | フィールド型 | 定数型 |
|----------|--------------------------------------------------------------------------------|--------|-----|
| `String` | 常に有効なUTF\-8の文字列値。                                                             | ✅      | ✅   |
| `IpCidr` | CIDR形式のIPアドレスの範囲。IPv4またはIPv6のいずれかです。                                           | ❌      | ✅   |
| `IpAddr` | 単一のIPアドレス。IPv4またはIPv6のいずれかです。                                                  | ✅      | ✅   |
| `Int`    | 64ビットの符号付き整数。                                                                  | ✅      | ✅   |
| `Regex`  | Rust`regex`クレートによって指定された[構文](https://docs.rs/regex/latest/regex/#syntax)の正規表現。 | ❌      | ✅   |

さらに、式は`Array`という複合型もサポートしています。配列型は`Type[]`と記述されます。
たとえば、`String[]`や`Int[]`です。現在、配列はフィールド値にのみ存在できます。これらは、
1つのフィールドが複数の値を含む場合に使用されます。たとえば、 `http.headers.x`や`http.queries.x`です。

### ストリング

文字列は有効なUTF\-8シーケンスで、`"content"`のような文字列リテラルで定義可能です。
次のエスケープシーケンスがサポートされています。

| エスケープシーケンス |     説明      |
|------------|-------------|
| `\n`      | 改行文字        |
| `\r`      | キャリッジリターン文字 |
| `\t`      | 水平タブ文字      |
| `\\`     | `\`文字      |
| `\"`      | `"`文字       |

さらに、式は`r#"content"#`のような生のリテラル文字列をサポートします。
この機能は、正規表現を書きたいときにエスケープを繰り返すのが面倒になった場合に
便利です。

たとえば、正規表現`~` 演算子を使って `http.path` と `/\d+\-\d+` を一致させたい場合、述部はリテラル文字列を使用して次のように記述されます。

    http.path ~ "/\\d+\\-\\d+"

生の文字列リテラルを使用すると、次のように記述できます。

    http.path ~ r#"/\d+\-\d+"#

### IpCidr

`IpCidr`はクラスレスドメイン間ルーティング（CIDR）形式で幅広いIPアドレスを表します。

以下はIPv4の例です。

    net.src.ip in 192.168.1.0/24

以下はIPv6の例です。

    net.src.ip in fd00::/8

式パーサーは、ホスト部分にゼロ以外のビットが含まれるCIDRリテラルを拒否します。つまり、作成者の意図が不明瞭であるため、 `192.168.0.1/24`パーサーチェックに合格しません。

### IPアドレス

`IpAddr`IPv4ドット10進表記、
または標準のIPv6アドレス形式で単一のIPアドレスを表します。

以下はIPv4の例です。

    net.src.ip == 192.168.1.1

以下はIPv6の例です。

    net.src.ip == fd00::1

### Int

式には整数型が1つだけあります。すべての整数は符号付き64ビット整数です。整数
リテラルは `12345`、`-12345`、または `0xab12ff` などの16進形式や、
`0751` のようなオクテット形式で書くことができます。

### 正規表現

正規表現は`String`リテラルとして記述されますが、`~`正規表現演算子が存在すると解析され
[Rust `regex`クレート構文に従って有効性を確認されます](https://docs.rs/regex/latest/regex/#syntax)。
例えば、以下の述部では、定数は`Regex`として解析されます。

    http.path ~ r#"/foo/bar/.+"#

演算子
---

式言語は、さまざまなデータ型に対して実行できる豊富な演算子をサポートしています。

|      演算子       |    名前     |                     説明                     |
|----------------|-----------|--------------------------------------------|
| `==`           | イコール      | フィールド値と定数値が等しい                             |
| `!=`           | 等しくない     | フィールド値と定数値が等しくない                           |
| `~`            | 正規表現の一致   | フィールド値が正規表現に一致する                           |
| `^=`           | 接頭辞の一致    | フィールド値が定数値で始まる                             |
| `=^`           | 接尾辞が一致    | フィールド値が定数値で終わる                             |
| `>=`           | より大きいか等しい | フィールド値が定数値と等しいかそれ以上                        |
| `>`            | より大きい     | フィールド値が定数値より大きい                            |
| `<=`           | 以下        | フィールド値が定数値以下である                            |
| `<`            | より小さい     | フィールド値が定数値より小さい                            |
| `in`           | 内         | フィールド値が定数値内にある                             |
| `not in`       | 含まれていない   | フィールド値が定数値内にない                             |
| `contains`     | 含む        | フィールド値に定数値が含まれる                            |
| `&&`           | および       | 左辺と右辺の **両方の** 式が評価される場合は`true`を返す`true`   |
| `||`           | または       | 左辺と右辺の **いずれかの** 式が評価される場合は`true`を返す`true` |
| `(Expression)` | 括弧        | 最初に評価される式をグループ化する                          |

{% if_version gte:3.4.x %}
\| `!` \| Not \| 括弧で囲まれた式の結果を否定します。 **注意：** `!`演算子は、 `!(foo == 1)`のような括弧で囲まれた式でのみ使用できます。 `! foo == 1`のような単なる述語では使用 **できません** 。
{% endif_version %}

### 詳細な説明

### 含む、含まない

これらの演算子は `IpAddr` と `IpCidr` タイプとともに使用され、IPリストのチェックを効率的に行います。
たとえば、 `net.src.ip in 192.168.0.0/24` `net.src.ip`の値が
`192.168.0.0/24`の範囲内にある場合にのみ`true`を返します。

### 含む

この演算子は、文字列が別の文字列内に存在するかどうかを確認するために使用されます。
たとえば、 `foo`が`http.path`内のどこかで見つかった場合、 `http.path contains "foo"` は`true`を返します。
これは、たとえば`/foo` 、 `/abc/foo`、または`/xfooy`のような`http.path`と一致します。

### 型と演算子のセマンティクス

各演算子で許可されるフィールドタイプと定数タイプの組み合わせは次のとおりです。
次の表では、行は述語の左側（LHS）に表示されるフィールドタイプを表しています。
一方、列は述語の右側（RHS）に表示される定数値タイプを表します。

| フィールド（LHS）/定数（RHS）型 |              `String`              |   `IpCidr`    | `IpAddr` |            `Int`            | `Regex` | `Expression` |
|---------------------|------------------------------------|---------------|----------|-----------------------------|---------|--------------|
| `String`            | `==`、`!=`、`~`、`^=`、`=^`、`contains` | ❌             | ❌        | ❌                           | `~`     | ❌            |
| `IpAddr`            | ❌                                  | `in`,`not in` | `==`     | ❌                           | ❌       | ❌            |
| `Int`               | ❌                                  | ❌             | ❌        | `==`、`!=`、`>=`、`>`、`<=`、`<` | ❌       | ❌            |
| `Expression`        | ❌                                  | ❌             | ❌        | ❌                           | ❌       | `&&`,`||`    |

{:.note}
> 
> **注：** 

* `~`演算子は、 `String ~ String`および`String ~ Regex`の両方をサポートすると説明されています。 実際には、 `Regex`定数値は右側に`String`としてのみ記述できます。 `~`演算子が存在する場合、文字列値は正規表現として扱われます。 `~`演算子を使用した場合でも、[上記の`String`エスケープルール](#string)は適用され、 [`Regex`セクション](#regex)で説明されているように、 `~`演算子には生のリテラル文字列を使用する方が簡単です。
* `~`演算子は、入力の先頭に正規表現を自動的に固定しません。 つまり、 `http.path ~ r#"/foo/\d"#`は`/foo/1`または`/some/thing/foo/1`のようなパスと一致する可能性があります。 文字列の先頭からマッチさせる（正規表現を固定する）場合は、`^` メタ文字を使用して手動で指定してください。たとえば、 `http.path ~ r#"^/foo/\d"#`などです。
* IPアドレス関連の比較を `==`、`in`、`not in` で行うときは、 アドレスタイプの異なるファミリーと定数値を指定すると、実行時、述部は常に `false`を返します。
