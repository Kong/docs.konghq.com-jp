---
title: "テンプレートの使用"
badge: "enterprise"
---
Kong Portalは`lua-resty-template`テンプレートライブラリ上に構築されており、[https://github.com/bungle/lua\-resty\-template](https://github.com/bungle/lua-resty-template)で確認できます。ライブラリの基本的な使い方については後述します。何が実現できるかについての詳細な情報は、ソースドキュメントを参照してください。

構文
---

***\(lua\-resty\-templates ドキュメントからの抜粋\)*** 

テンプレートでは次のタグを使用できます。
{% raw %}

* `{{expression}}`、式の結果を書き込みます（HTMLエスケープ処理された）

* `{*expression*}`, 式の結果を記述します

* `{% lua code %}`は、Luaコードを実行します

* `{(path-to-partial)}`、パスで`partial`ファイルを含める場合、ファイルのコンテキストを指定することもできます`{(partials/header.html, { message = "Hello, World" } )}`

* `{-block-}...{-block-}``{-block-}` の内容を、（このケースでは）キー `block` を持つ `blocks` テーブルに格納された値にラップします。[ブロックの使用](https://github.com/bungle/lua-resty-template#using-blocks)を参照してください。定義済みのブロック名（`verbatim` および `raw`）をそのまま使用しないでください。

* `{-verbatim-}...{-verbatim-}`、`{-raw-}...{-raw-}`は定義済みのブロックであり、その内部は`lua-resty-template`によって処理されず、内容がそのまま出力されます。

* `{# comments #}``{#`から`#}`までのすべての要素はコメントアウト（出力も実行も不可）状態とみなされます。
  {% endraw %}

カスタムプロパティの表示
------------

OpenAPI仕様では、カスタムプロパティを使用できます。Dev Portalでカスタムプロパティを公開するには、`spec-renderer.html`ファイルの`showExtensions`を`true`に変更します。デフォルトでは、`showExtensions`は`false`です。

部分的
---

パーシャルは、レイアウトが参照できる HTML のスニペットです。パーシャルはそのレイアウトがアクセスできるデータと同じすべてのデータにアクセスでき、他のパーシャルを呼び出すこともできます。コードをパーシャルに分割すると、大きなページを整理しやすくなるだけでなく、さまざまなレイアウトで共通のページ要素を共有できます。

### content/index.txt

{% raw %}

    ---
    layout: index.html
    title: Partials
    header_logo: assets/images/example.jpeg
    header_nav_items:
      about:
        href: /about
      guides:
        href: /guides
    hero_title: Partials Info
    hero_description: Partials are wicked sick!
    ---

{% endraw %}

### layouts/index.html

{% raw %}

```html
{(partials/header.html)}
<div class="content" id="sl-md0000000">
  {(partials/hero.html)}
</div>
{(partials/footer.html)}
```

{% endraw %}

### partials/header.html

{% raw %}

```html
<header class="row" id="sl-md0000000">
  <div class="column" id="sl-md0000000">
    <img src="{{page.header_logo}}" id="sl-md0000000"/>
  </div>
  <div class="column" id="sl-md0000000">
    {(partials/header_nav.html)}
  </div>
</header>
```

{% endraw %}

### partials/header\_nav.html

{% raw %}

```html
<ul id="sl-md0000000">
  {% for title, href in each(page.header_nav_items) do %}
    <li id="sl-md0000000"><a href="{{href}}" id="sl-md0000000">{{title}}</a></li>
  {% end %}
</ul>
```

{% endraw %}

### partials/hero.html

{% raw %}

```html
<h1 id="sl-md0000000">{{page.hero_title}}</h1>
<p id="sl-md0000000">{{page.hero_description}}</p>
```

{% endraw %}

### partials/hero.html

{% raw %}

```html
<footer id="sl-md0000000">
  <p id="sl-md0000000">footer</p>
</footer>
```

{% endraw %}

出力：

{% raw %}

```html
<header class="row" id="sl-md0000000">
  <div class="column" id="sl-md0000000">
    <img src="assets/images/example.jpeg" id="sl-md0000000"/>
  </div>
  <div class="column" id="sl-md0000000">
    <ul id="sl-md0000000">
      <li id="sl-md0000000"><a href="/about" id="sl-md0000000">about</a></li>
      <li id="sl-md0000000"><a href="/guieds" id="sl-md0000000">guides</a></li>
    </ul>
  </div>
</header>
<h1 id="sl-md0000000">Partials Info</h1>
<p id="sl-md0000000">Partials are wicked sick!</p>
<footer id="sl-md0000000">
  <p id="sl-md0000000">footer</p>
</footer>
```

{% endraw %}

ブロック
----

ブロックを使用すると、ビューまたは部分を別のテンプレートに埋め込むことができます。ブロックは、異なるテンプレートで共通のラッパーを共有する場合に特に便利です。

以下の例では、コンテンツ ファイルが`wrapper.html`ではなく`index.html`を参照していることに注意してください。

### content/index.txt

{% raw %}

```markdown
---
layout: index.html
title: Blocks
description: Blocks are the future!
---
```

{% endraw %}

### layouts/index.html

{% raw %}

```html
{% layout = "layouts/wrapper.html" %}    <- syntax declaring where to find the block

{-main-}                                 <- delimiter describing what content renders in block
<div class="content" id="sl-md0000000">
  <h1 id="sl-md0000000">{{page.title}}</h1>
  <p id="sl-md0000000">{{page.description}}<p id="sl-md0000000">
</div>
{-main-}
```

{% endraw %}

### layouts/wrapper.html

{% raw %}

```html
<!doctype id="sl-md0000000"     >
<html id="sl-md0000000">
  <head id="sl-md0000000">
    <title id="sl-md0000000">Testing lua-resty-template blocks</title>
  </head>
  <body id="sl-md0000000">
    <header id="sl-md0000000">
      <p id="sl-md0000000">header</p>
    </header>
    {*main*}                 <- syntax indicating where to place the block
    <footer id="sl-md0000000">
      <p id="sl-md0000000">footer</p>
    </footer>
  </body>
</html>
```

{% endraw %}

出力：

{% raw %}

```html
<!doctype id="sl-md0000000"     >
<html id="sl-md0000000">
  <head id="sl-md0000000">
    <title id="sl-md0000000">Testing lua-resty-template blocks</title>
  </head>
  <body id="sl-md0000000">
    <header id="sl-md0000000">
      <p id="sl-md0000000">header</p>
    </header>
    <div class="content" id="sl-md0000000">
      <h1 id="sl-md0000000">Blocks</h1>
      <p id="sl-md0000000">Blocks are the future!<p id="sl-md0000000">
    </div>
    <footer id="sl-md0000000">
      <p id="sl-md0000000">footer</p>
    </footer>
  </body>
</html>
```

{% endraw %}

コレクション
------

コレクションは、コンテンツのセットをグループとしてレンダリングできる強力なツールです。コレクションとしてレンダリングされるコンテンツは、構成可能なルートパターンとレイアウトを共有します。コレクションはポータル `portal.conf.yaml` ファイルで構成します。

以下の例は、個々の `posts` で構成される基本的な `blog` コレクションをレンダリングするのに必要なすべての設定/ファイルを示しています。

#### portal.conf.yaml

{% raw %}

    name: Kong Portal
    theme:
      name: base
    collections:
      posts:
        output: true
        route: /:stub/:collection/:name
        layout: post.html

{% endraw %}

上記を見ると、`collections`オブジェクトが宣言されたことがわかります。このオブジェクトは個別コレクションで構成されています。この例では、`posts`という名前のコレクションを構成しています。レンダラーは`content`フォルダ内の`_posts`というルートディレクトリでレンダリングする個々のページを検索します。`animals`という名前の別のコレクション構成を作成した場合、レンダラーは`_animals`というディレクトリでレンダリングするコンテンツファイルを検索します。

各構成項目は、いくつかのパーツで構成されています。

* `output` 
  * **必須** ：false
  * **タイプ** ：`boolean`
  * **説明** ：このオプション属性は、コレクションをレンダリングするかどうかを決定します。`false`に設定すると、コレクションの仮想ルートは作成されません。

* `route` 
  * **必須** : true
  * **タイプ** ：`string`
  * **デフォルト** ：`none`
  * **説明** ：`route` 属性は必須で、どのパターンからコレクションルートを生成するかをレンダラーに伝えます。コレクションルートには、各コレクションメンバーを一意に識別する有効な動的名前空間が必ず1つ以上含まれている必要があります。 
    * ルート宣言内の`:`で始まる名前空間はすべて動的であると見なされます。
    * Kongでは、特定の動的名前空間のみが有効と認識されます。 
      * `:title`: 名前空間を、headmatterで宣言されたコンテンツ`title`に置き換えます。
      * `:name`：名前空間をコンテンツのファイル名に置き換えます。
      * `:collection`: 名前空間を現在のコレクションの名前に置き換えます。
      * `:stub`：各コンテンツのheadmatterのnamespaceを`headmatter.stub`の値で置き換えます。

* `layout` 
  * **必須** ：true
    * **タイプ** ：`string`
    * **説明** : `layout` 属性は、コレクションがレンダリングに使用する HTML レイアウトを決定します。パスルートは、現在のテーマ `layouts` ディレクトリ内からアクセスされます。

### content/\_posts/post1\.md

{% raw %}

    ---
    title: Post One
    stub: blog
    ---
    
    This is my first post!

{% endraw %}

### content/\_posts/post2\.md

{% raw %}

    ---
    title: Post Two
    stub: blog
    ---
    
    This is my second post!

{% endraw %}

### themes/base/layouts/post.html

{% raw %}

```html
<h1 id="sl-md0000000">{{ page.title }}</h1>
<p id="sl-md0000000">{* page.body *}</p>
```

{% endraw %}

出力：

`<kong_portal_gui_url id="sl-md0000000">/blog/posts/post1`より：

```html
<h1 id="sl-md0000000">Post One</h1>
<p id="sl-md0000000">This is my first post!</p>
```

`<kong_portal_gui_url id="sl-md0000000">/blog/posts/post2`より：

```html
<h1 id="sl-md0000000">Post Two</h1>
<p id="sl-md0000000">This is my second post!</p>
```

Kong テンプレートヘルパー \- Lua API
---------------------------

Kong テンプレートヘルパーは、レンダリング時にポータルデータへのアクセスを提供し、Kong への強力な統合を実現するオブジェクトのコレクションです。

グローバル:

* [`l`](#lkey-fallback) \- 最初のバージョンのロケールヘルパーで、現在アクティブなページから値を取得します。
* [`each`](#eachlist_or_table) \- リストまたはテーブルを反復処理するためによく使用されるヘルパー。
* [`print`](#printany) \- リスト/テーブルを印刷するためによく使用されるヘルパー。
* [`markdown`](#printany) \- リスト/テーブルを印刷するためによく使用されるヘルパー。
* [`json_decode`](#json_decode) \- JSON を Lua テーブルにデコードします。
* [`json_encode`](#json_encode) \- LuaテーブルをJSONにエンコードします。

オブジェクト：

* [`portal`](#portal) \- ポータルオブジェクトは、現在アクセスされているのワークスペースポータルを指します。
* [`page`](#page) \- ページオブジェクトは、現在アクティブなページとそのコンテンツを参照します。
* [`user`](#user) \- ユーザーオブジェクトはKongポータルに現在ログインしてアクセスしている開発者を表します。
* [`theme`](#theme) \- テーマオブジェクトは、現在アクティブなテーマとその変数を表します。
* [`tbl`](#tbl) = テーブルヘルパーメソッド。例: `map`、`filter`、`find`、`sort`.
* [`str`](#str) = 文字列ヘルパーメソッド。例：`lower`、`upper`、`reverse`、`endswith`など
* [`helpers`](#helpers) \- ヘルパー関数は、一般的なタスクを簡素化したり、Kong Portal メソッドへの簡単なショートカットを提供したりします。

用語/定義：

* `list` \- Luaでの通称である配列（`[1, 2, 3]`）は、テーブルのようなオブジェクト（`{1, 2, 3}`）です。Luaリストのインデックスは、`0`ではなく`1`から始まります。値には配列表記（`list[1]`）でアクセスできます。
* `table` \- LuaではオブジェクトまたはHashMap（`{1: 2}`）ともよく呼ばれ、（`{1 = 2}`）のように記述されます。値には配列またはドット表記（`table.one or table["one"]`）でアクセスできます。

### l\(key, fallback\)

現在アクティブなページから現在の翻訳をキーで返します。

#### 戻り値の型

{% raw %}

```lua
string
```

{% endraw %}

#### 使用法

`content/en/example.txt`を使用：

{% raw %}

```yaml
---
layout: example.html

locale:
  title: Welcome to {{portal.name}}
  slogan: The best developer portal ever created.
---
```

{% endraw %}

`content/es/example.txt`を使用：

{% raw %}

```yaml
---
layout: example.html

locale:
  title: Bienvenido a {{portal.name}}
  slogan: El mejor portal para desarrolladores jamás creado.
---
```

{% endraw %}

`layouts/example.html`を使用：

{% raw %}

```lua
<h1 id="sl-md0000000">{* l("title", "Welcome to" .. portal.name) *}</h1>
<p id="sl-md0000000">{* l("slogan", "My amazing developer portal!") *}</p>
<p id="sl-md0000000">{* l("powered_by", "Powered by Kong.") *}</p>
```

{% endraw %}

出力：

`en/example`の場合：

{% raw %}

```html
<h1 id="sl-md0000000">Welcome to Kong Portal</h1>
<p id="sl-md0000000">The best developer portal ever created.</p>
<p id="sl-md0000000">Powered by Kong.</p>
```

{% endraw %}

`es/example`の場合：

{% raw %}

```html
<h1 id="sl-md0000000">Bienvenido a Kong Portal</h1>
<p id="sl-md0000000">El mejor portal para desarrolladores jamás creado.</p>
<p id="sl-md0000000">Powered by Kong.</p>
```

{% endraw %}

#### 注記

* `l(...)` は、 `page` オブジェクトからのヘルパーです。`page.l`経由でもアクセス可能です。ただし、 `page.l`テンプレート補間をサポートしていません \(たとえば、 `{{portal.name}}`は機能しません\)。

### each\(list\_or\_table\)

渡される引数のタイプに応じて適切な反復子を返します。

#### 戻り値の型

```lua
Iterator
```

#### 使用法

テンプレート（リスト）：

{% raw %}

```lua
{% for index, value in each(table) do %}
<ul id="sl-md0000000">
  <li id="sl-md0000000">Index: {{index}}</li>
  <li id="sl-md0000000">Value: {{ print(value) }}</li>
</ul>
{% end %}
```

{% endraw %}

テンプレート（テーブル）：

{% raw %}

```lua
{% for key, value in each(table) do %}
<ul id="sl-md0000000">
  <li id="sl-md0000000">Key: {{key}}</li>
  <li id="sl-md0000000">Value: {{ print(value) }}</li>
</ul>
{% end %}
```

{% endraw %}

### print\(any\)

入力値の出力を文字列として返します。

#### 戻り値の型

```lua
string
```

#### 使用法

テンプレート（テーブル）：

{% raw %}

```lua
<pre id="sl-md0000000">{{print(page)}}</pre>
```

{% endraw %}

### markdown（文字列）

引数として渡されたマークダウン文字列からHTMLを返します。文字列引数が有効なマークダウンでない場合、関数は文字列をそのまま返します。正しくレンダリングするには、ヘルパーを未加工の`{* *}`区切り文字とともに使用する必要があります。

#### 戻り値の型

```lua
string
```

#### 使用法

テンプレート（引数としての文字列）:

{% raw %}

```lua
<pre id="sl-md0000000">{* markdown("##This is Markdown") *}</pre>
```

{% endraw %}

テンプレート（引数としてのコンテンツ値）:

{% raw %}

```lua
<pre id="sl-md0000000">{* markdown(page.description) *}</pre>
```

{% endraw %}

### json\_encode\(object\)

JSONは引数として渡されたLuaテーブルをエンコードします

#### 戻り値の型

```lua
string
```

#### 使用法

テンプレート：

{% raw %}

```lua
<pre id="sl-md0000000">{{ json_encode({ dog = cat }) }}</pre>
```

{% endraw %}

### json\_decode\(string\)

JSON 文字列引数を Lua テーブルにデコードします

#### 戻り値の型

```lua
table
```

#### 使用法

テンプレート：

{% raw %}

```lua
<pre id="sl-md0000000">{{ print(json_encode('{"dog": "cat"}')) }}</pre>
```

{% endraw %}

### ポータル

`portal`では現在のポータルに関連するデータにアクセスできます。たとえば、ポータルの構成、コンテンツ、仕様、レイアウトなどのデータです。

* [`portal.workspace`](#portalworkspace)
* [`portal.url`](#portalurl)
* [`portal.api_url`](#portalapi_url)
* [`portal.auth`](#portalauth)
* [`portal.specs`](#portalspecs)
* [`portal.specs_by_tag`](#portalspecs_by_tag)
* [`portal.developer_meta_fields`](#portaldeveloper_meta_fields)

次のようにして、`portal`オブジェクトで現在のワークスペースポータル構成に直接アクセスできます。

```lua
portal[config_key] or portal.config_key
```

たとえば、 `portal.auth` はポータルの構成値です。構成値のリストは、 `kong.conf`のポータルセクションを読むことで確認できます。

#### kong.confから

ポータルには `portal_` で始まる設定値のみが表示され、`portal_` プレフィックスを削除することでアクセスできます。

一部の設定値は変更またはカスタマイズされています。これらのカスタマイズは、[ポータルメンバー](#portal-members)セクションに記載されています。

##### portal.workspace

現在のポータルのワークスペースを返します。

##### 戻り値の型

```lua
string
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{portal.workspace}}
```

{% endraw %}

出力：

```html
default
```

#### portal.url

現在のポータルのURLをワークスペースと共に返します。

##### 戻り値の型

```lua
string
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{portal.url}}
```

{% endraw %}

出力：

```html
http://127.0.0.1:8003/default
```

#### portal.api\_url

追加された現在のワークスペースで`portal_api_url`の設定値を返します。

##### 戻り値の型

```lua
string or nil
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{portal.api_url}}
```

{% endraw %}

`portal_api_url = http://127.0.0.1:8004` の場合の出力結果:

```html
http://127.0.0.1:8004/default
```

#### portal.auth

現在のポータルの認証タイプを返します。

##### 戻り値の型

```lua
string
```

##### 使用法

**値の印刷** 

入力：

{% raw %}

```hbs

{{portal.auth}}
```

{% endraw %}

`portal_auth = basic-auth` の場合の出力結果:

```html
basic-auth
```

**認証が有効になっているかどうかを確認** 

入力：

{% raw %}

```hbs
{% if portal.auth then %}
  Authentication is enabled!
{% end %}
```

{% endraw %}

`portal_auth = basic-auth` の場合の出力結果:

```html
Authentication is enabled!
```

#### portal.specs

現在のポータル内に含まれる仕様ファイルの配列を返します。

##### 戻り値の型

```lua
array
```

##### 使用法

**コンテンツ値の表示** 

テンプレート：

{% raw %}

```hbs
<pre id="sl-md0000000">{{ print(portal.specs) }}</pre>
```

{% endraw %}

出力：

```lua
{
  {
    "path" = "content/example1_spec.json",
    "content" = "..."
  },
  {
    "path" = "content/documentation/example1_spec.json",
    "content" = "..."
  },
  ...
}
```

**値のループ** 

テンプレート：

{% raw %}

```hbs
{% for _, spec in each(portal.specs) %}
  <li id="sl-md0000000">{{spec.path}}</li>
{% end %}
```

{% endraw %}

出力：

```hbs
  <li id="sl-md0000000">content/example1_spec.json</li>
  <li id="sl-md0000000">content/documentation/example1_spec.json</li>
```

**パスでフィルタリング** 

テンプレート：

{% raw %}

```hbs
{% for _, spec in each(helpers.filter_by_path(portal.specs, "content/documentation")) %}
  <li id="sl-md0000000">{{spec.path}}</li>
{% end %}
```

{% endraw %}

出力：

```hbs
  <li id="sl-md0000000">content/documentation/example1_spec.json</li>
```

#### portal.developer\_meta\_fields

開発者を登録するためにKongで利用可能/必要な開発者メタフィールドの配列を返します。

#### 戻り値の型

```lua
array
```

##### 使用法

**値の印刷** 

テンプレート：

{% raw %}

```hbs

{{ print(portal.developer_meta_fields) }}
```

{% endraw %}

出力：

{% raw %}

```lua
{
  {
    label    = "Full Name",
    name     = "full_name",
    type     = "text",
    required = true,
  },
  ...
}
```

{% endraw %}

**値のループ** 

テンプレート：

{% raw %}

```hbs
{% for i, field in each(portal.developer_meta_fields) do %}
<ul id="sl-md0000000">
  <li id="sl-md0000000">Label: {{field.label}}</li>
  <li id="sl-md0000000">Name: {{field.name}}</li>
  <li id="sl-md0000000">Type: {{field.type}}</li>
  <li id="sl-md0000000">Required: {{field.required}}</li>
</ul>
{% end %}
```

{% endraw %}

出力：

```html
<ul id="sl-md0000000">
  <li id="sl-md0000000">Label: Full Name</li>
  <li id="sl-md0000000">Name: full_name</li>
  <li id="sl-md0000000">Type: text</li>
  <li id="sl-md0000000">Required: true</li>
</ul>
...
```

### ページ

`page`は、ページのURL、パス、ブレッドクラムなどを含む、現在のページに関連するデータにアクセスを提供します。

* [`page.route`](#pageroute)
* [`page.url`](#pageurl)
* [`page.breadcrumbs`](#pagebreadcrumbs)
* [`page.body`](#pagebody)

新しいコンテンツ ページを作成するときに、キーと値を定義できます。ここでは、それらの値にアクセスする方法とその他の興味深い点について学習します。

`page`オブジェクトに直接定義してキー値にアクセスするには、次のように指定します。

{% raw %}

```lua
page[key_name] or page.key_name
```

{% endraw %}

このように、ネストしたキーにアクセスすることもできます。

```lua
page.key_name.nested_key
```

{% raw %}
ここで注意が必要です。出力エラーを回避するには、次に示すように、`nested_key` にアクセスする前に `key_name` が存在することを確認してください。

```hbs

{{page.key_name and page.key_name.nested_key}}
```

{% endraw %}

#### page.route

現在のページのルート/パスを返します。

##### 戻り値の型

```lua
string
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{page.route}}
```

{% endraw %}

出力（指定URLは`http://127.0.0.1:8003/default/guides/getting-started`）。

```html
guides/getting-started
```

#### page.url

現在のページの URL を返します。

##### 戻り値の型

```lua
string
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{page.url}}
```

{% endraw %}

出力（指定URLは`http://127.0.0.1:8003/default/guides/getting-started`）。

```html
http://127.0.0.1:8003/default/guides/getting-started
```

#### page.breadcrumbs

現在のページの階層リンクコレクションを返します。

##### 戻り値の型

```lua
table[]
```

##### アイテムのプロパティ

* `item.path` \- アイテムへの完全パス、フォワードスラッシュの接頭辞なし。
* `item.display_name` \- フォーマットされた名前。
* `item.is_first` \- これはリストの最初のアイテムですか？
* `item.is_last` \- これはリストの最後のアイテムですか？

##### 使用法

テンプレート：

{% raw %}

```hbs
<div id="sl-md0000000 breadcrumbs">
  <a href="" id="sl-md0000000">Home</a>
  {% for i, crumb in each(page.breadcrumbs) do %}
    {% if crumb.is_last then %}
      / {{ crumb.display_name }}
    {% else %}
      / <a href="{{crumb.path}}" id="sl-md0000000">{{ crumb.display_name }}</a>
    {% end %}
  {% end %}
</div>
```

{% endraw %}

#### page.body

現在のページの本文を文字列として返します。ルートのコンテンツファイルに `.md` または `.markdown` の拡張子が付いている場合、本文はマークダウンからHTMLに解析されます。

##### 戻り値の型

```lua
string
```

##### .txt、.json、.yaml、.ymlテンプレートの使用法

`index.txt`:

```hbs
This is text content.
```

テンプレート：
{% raw %}

```hbs
<h1 id="sl-md0000000">This is a title</h1>
<p id="sl-md0000000">{{ page.body) }}</p>
```

{% endraw %}

出力：

    > # This is a title
    > This is text content.

##### 使用法：.md、.markdown テンプレート

テンプレート（markdown）：
テンプレート内のマークダウンをレンダリングするには、未加工の区切り記号 `{* *}` を使用してください。

`index.txt`

```hbs
# This is a title
This is text content.
```

テンプレート：

```hbs
{* page.body *}
```

出力：

    > # This is a title
    > This is text content.

### ユーザー

`user`は、現在認証されているユーザーに関連するデータへのアクセスを許可します。ユーザーオブジェクトは、`KONG_PORTAL_AUTH`が有効な場合にのみ適用されます。

* [`user.is_authenticated`](#useris_authenticated)
* [`user.has_role`](#userhas_role)
* [`user.get`](#userget)

#### user.is\_authenticated

現在のユーザーの認証ステータスを `boolean` 値で返します。

##### 戻り値の型

```lua
boolean
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{print(user.is_authenticated)}}
```

{% endraw %}

出力：

```html
true
```

#### user.has\_role

ユーザーに引数として指定されたロールがある場合は`true`を返します。

##### 戻り値の型

```lua
boolean
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{print(user.has_role("blue"))}}
```

{% endraw %}

出力：

```html
true
```

#### user.get

開発者属性を引数に受け取り、存在すれば値を返します。

##### 戻り値の型

```lua
any
```

##### 使用法

テンプレート：

{% raw %}

```hbs

{{user.get("email")}}

{{print(user.get("meta"))}}
```

{% endraw %}

出力：

```html
example123@konghq.com
{ "full_name" = "example" }
```

### テーマ

`theme`オブジェクトは、`theme.conf.yaml`ファイルに設定された値を公開します。さらに、`portal.conf.yaml`に含まれる変数のオーバーライドも含まれます。

* [`theme.colors`](#usercolors)
* [`theme.color`](#usercolor)
* [`theme.fonts`](#userfonts)
* [`theme.font`](#userfont)

#### theme.colors

色変数のテーブルとその値をキーと値のペアで返します。

##### 戻り値の型

```lua
table
```

##### 使用法

`theme.conf.yaml`:

```yaml
name: Kong
colors:
  primary:
    value: '#FFFFFF'
    description: 'Primary Color'
  secondary:
    value: '#000000'
    description: 'Secondary Color'
  tertiary:
    value: '#1DBAC2'
    description: 'Tertiary Color'
```

テンプレート：

{% raw %}

```lua
{% for k,v in each(theme.colors) do %}
  <p id="sl-md0000000">{{k}}: {{v}}</p>
{% end %}
```

{% endraw %}

出力：

```html
<p id="sl-md0000000">primary: #FFFFFF</p>
<p id="sl-md0000000">secondary: #000000</p>
<p id="sl-md0000000">tertiary: #1DBAC2</p>
```

#### theme.colors

説明

string引数でcolor varを取り、値を返します。

##### 戻り値の型

```lua
string
```

##### 使用法

`theme.conf.yaml`:

```yaml
name: Kong
colors:
  primary:
    value: '#FFFFFF'
    description: 'Primary Color'
  secondary:
    value: '#000000'
    description: 'Secondary Color'
  tertiary:
    value: '#1DBAC2'
    description: 'Tertiary Color'
```

テンプレート：

{% raw %}

```lua
<p id="sl-md0000000">primary: {{theme.color("primary")}}</p>
<p id="sl-md0000000">secondary: {{theme.color("secondary")}}</p>
<p id="sl-md0000000">tertiary: {{theme.color("tertiary")}}</p>
```

{% endraw %}

出力：

```html
<p id="sl-md0000000">primary: #FFFFFF</p>
<p id="sl-md0000000">secondary: #000000</p>
<p id="sl-md0000000">tertiary: #1DBAC2</p>
```

#### theme.fonts

フォント変数のテーブルとその値をキーと値のペアで返します。

##### 戻り値の型

```lua
table
```

##### 使用法

`theme.conf.yaml`:

```yaml
name: Kong
fonts:
  base: Roboto
  code: Roboto Mono
  headings: Lato
```

テンプレート：

{% raw %}

```lua
{% for k,v in each(theme.fonts) do %}
  <p id="sl-md0000000">{{k}}: {{v}}</p>
{% end %}
```

{% endraw %}

出力：

```html
<p id="sl-md0000000">base: Roboto</p>
<p id="sl-md0000000">code: Roboto Mono</p>
<p id="sl-md0000000">headings: Lato</p>
```

#### theme.font

文字列引数でフォント名を取り、値を返します。

##### 戻り値の型

```lua
string
```

##### 使用法

`theme.conf.yaml`:

```yaml
name: Kong
fonts:
  base: Roboto
  code: Roboto Mono
  headings: Lato
```

テンプレート：

{% raw %}

```lua
<p id="sl-md0000000">base: {{theme.font("base")}}</p>
<p id="sl-md0000000">code: {{theme.font("code")}}</p>
<p id="sl-md0000000">headings: {{theme.font("headings")}}</p>
```

{% endraw %}

出力：

```html
<p id="sl-md0000000">base: #FFFFFF</p>
<p id="sl-md0000000">code: #000000</p>
<p id="sl-md0000000">headings: #1DBAC2</p>
```

### str

便利な文字列ヘルパーメソッドを含むテーブル。

#### 使用法

`.upper()`例：

{% raw %}

```lua
<pre id="sl-md0000000">{{ str.upper("dog") }}</pre>
```

{% endraw %}

#### メソッド

##### str.[byte](https://www.gammon.com.au/scripts/doc.php?lua=string.byte)

##### str.[char](https://www.gammon.com.au/scripts/doc.php?lua=string.char)

##### str.[dump](https://www.gammon.com.au/scripts/doc.php?lua=string.dump)

##### str[.find](https://www.gammon.com.au/scripts/doc.php?lua=string.find)

##### str.[format](https://www.gammon.com.au/scripts/doc.php?lua=string.format)

##### str. [gfind](https://www.gammon.com.au/scripts/doc.php?lua=string.gfind)

##### str.[gmatch](https://www.gammon.com.au/scripts/doc.php?lua=string.gmatch)

##### str.[gsub](https://www.gammon.com.au/scripts/doc.php?lua=string.gsub)

##### str.[len](https://www.gammon.com.au/scripts/doc.php?lua=string.len)

##### str[.lower](https://www.gammon.com.au/scripts/doc.php?lua=string.lower)

##### str.[match](https://www.gammon.com.au/scripts/doc.php?lua=string.match)

##### str.[rep](https://www.gammon.com.au/scripts/doc.php?lua=string.rep)

##### str.[reverse](https://www.gammon.com.au/scripts/doc.php?lua=string.reverse)

##### str.[sub](https://www.gammon.com.au/scripts/doc.php?lua=string.sub)

##### str[.upper](https://www.gammon.com.au/scripts/doc.php?lua=string.upper)

##### str. [isalpha](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#isalpha)

##### str.[isdigit](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#isdigit)

##### str.[isalnum](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#isalnum)

##### str.[isspace](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#isspace)

##### str.[islower](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#islower)

##### str.[isupper](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#isupper)

##### str[.startswith](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#startswith)

##### str[.endswith](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#endswith)

##### str.[join](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#join)

##### str.[splitlines](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#splitlines)

##### str[.split](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#split)

##### str.[expandtabs](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#expandtabs)

##### str.[lfind](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#lfind)

##### str.[rfind](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#rfind)

##### str.[replace](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#replace)

##### str.[count](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#count)

##### str. [ljust](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#ljust)

##### str.[rjust](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#rjust)

##### str.[center](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#center)

##### str.[lstrip](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#lstrip)

##### str. [rstrip](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#rstrip)

##### str.[strip](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#strip)

##### str.[splitv](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#splitv)

##### str.[partition](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#partition)

##### str.[rpartition](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#rpartition)

##### [str.at](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#at)

##### str.[lines](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#lines)

##### str.[title](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#title)

##### str.[shorten](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#shorten)

##### str.[quote\_string](https://stevedonovan.github.io/Penlight/api/libraries/pl.stringx.html#quote_string)

### tbl

便利なテーブルヘルパーメソッドを含むテーブル

#### 使用法

`.map()` 例:
{% raw %}

```lua
{% tbl.map({"dog", "cat"}, function(item) %}
  {% if item ~= "dog" then %}
    {% return true %}
  {% end %}
{% end) %}
```

{% endraw %}

#### メソッド

##### tbl.[getn](https://www.gammon.com.au/scripts/doc.php?lua=table.getn)

##### tbl.[setn](https://www.gammon.com.au/scripts/doc.php?lua=table.setn)

##### tbl.[maxn](https://www.gammon.com.au/scripts/doc.php?lua=table.maxn)

##### tbl[.insert](https://www.gammon.com.au/scripts/doc.php?lua=table.insert)

##### tbl.[remove](https://www.gammon.com.au/scripts/doc.php?lua=table.remove)

##### tbl.[concat](https://www.gammon.com.au/scripts/doc.php?lua=table.concat)

##### tbl.[map](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#map)

##### tbl.[foreach](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#foreach)

##### tbl.[foreachi](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#foreachi)

##### tbl.[sort](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#sort)

##### tbl.[sortv](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#sortv)

##### tbl.[filter](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#filter)

##### tbl.[size](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#size)

##### tbl.[index\_by](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#index_by)

##### tbl.[transform](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#transform)

##### tbl.[range](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#range)

##### tbl.[reduce](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#reduce)

##### tbl.[index\_map](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#index_map)

##### tbl.[makeset](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#makeset)

##### tbl.[union](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#union)

##### tbl.[intersection](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#intersection)

##### tbl.[count\_map](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#count_map)

##### tbl.[set](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#set)

##### tbl.[new](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#new)

##### tbl.[clear](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#clear)

##### tbl.[removevalues](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#removevalues)

##### tbl.[readonly](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#readonly)

##### tbl.[update](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#update)

##### tbl.[copy](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#copy)

##### tbl.[deepcopy](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#deepcopy)

##### tbl.[icopy](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#icopy)

##### tbl.[move](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#move)

##### tbl[.insertvalues](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#insertvalues)

##### tbl.[deepcompare](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#deepcompare)

##### tbl.[compare](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#compare)

##### tbl.[compare\_no\_order](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#compare_no_order)

##### tbl.[find](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#find)

##### tbl.[find\_if](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#find_if)

##### tbl.[search](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#search)

##### tbl.[keys](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#keys)

##### tbl.[values](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#values)

##### tbl.[sub](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#sub)

##### tbl.[merge](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#merge)

##### tbl.[difference](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#difference)

##### tbl.[zip](https://stevedonovan.github.io/Penlight/api/libraries/pl.tablex.html#zip)

