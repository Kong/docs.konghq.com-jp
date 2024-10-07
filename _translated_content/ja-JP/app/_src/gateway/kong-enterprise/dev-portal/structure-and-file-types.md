---
title: "開発者ポータルの構造とファイルの種類"
badge: "enterprise"
---
Kong Portalテンプレートは完全に刷新され、カスタマイズが容易になり、コンテンツとレイアウト間の懸念事項がより明確に分離され、Kongライフサイクル/データへのより強力な接続が可能になりました。内部には、`https://github.com/bungle/lua-resty-template`ライブラリ上に構築されたフラットファイルCMSを実装しています。このシステムは、過去に`jekyll`、`kirby cms`、または`vuepress`などのプロジェクトに取り組んだことがある人にとっては馴染みのある使い勝手になっているはずです。

{:.note}
> 
> 注: このガイドに従うには、[kong\-portal\-templates リポジトリ](https://github.com/Kong/kong-portal-templates)をクローンし、 `master` ブランチをチェックアウトするのが最適です。このガイドでは、1 つのワークスペース内で作業していることを前提としています（テンプレートリポジトリには、ワークスペースごとに多数の異なるポータルファイルのセットをホストできます）。ルートから `workspaces/default` ディレクトリに移動して、デフォルトのワークスペースのポータルファイルを表示します。

ディレクトリ構造
--------

`kong-portal-templates` ルートディレクトリから `workspaces/default` に移動して、デフォルトのポータルテンプレートファイルにアクセスします。このディレクトリ内の相対ファイル構造は、ファイル `path` スキーマ属性に直接マップされます。（ここで `content/homepage.txt` は、Kong の `content/homepage.txt` にマップされます\)。

Kong Developer Portalの単一インスタンスを構成する次の各種要素が、`workspaces/default`に表示されています。

* **content/** 
  * コンテンツディレクトリには、Kong Dev Portalのサイト構造と各ページ内でレンダリングされる動的コンテンツの両方を決定するファイルが含まれています。

* **スペック** 
  * スペックは、ページに動的コンテンツをレンダリングするのに必要なデータが含まれているという点でコンテンツに似ています。`specs`の場合、ファイルには仕様としてレンダリングされる有効なOASまたはSwaggerが含まれています。

* **themes/** 
  * テーマディレクトリには、ポータルのコンテンツに適用されるさまざまなテーマが含まれています。各テーマには、HTMLテンプレート、アセット、グローバルスタイルをテーマに設定できる構成ファイルが含まれています。

* **portal.conf.yaml** 
  * この構成ファイルは、ポータルがレンダリングに使用するテーマ、ポータルの名前、ログインやログアウトなどのユーザーアクションのリダイレクトパスなどの特別な動作の構成を決定します。

* **router.conf.yaml \(オプション\)** 
  * このオプションの構成ファイルは、ポータルのデフォルトのルーティングシステムをハードコードされた値で上書きします。このオプションは、単一ページのアプリケーションを実装する場合や、静的なサイト構造を設定する場合に役立ちます。

ポータル構成ファイル
----------

### パス

* **形式：** `portal.conf.yaml`
* **ファイル拡張子：** `.yaml`

### 説明

ポータル構成ファイルは、ポータルがレンダリングに使用するテーマ、ポータル名、ログイン/ログアウトなどのユーザーアクションのリダイレクトパスなどの特別な動作の構成を決定します。これは、すべてのポータルのルートに必要です。ポータル構成ファイルは1つしか存在できず、`portal.conf.yaml`と命名され、ルートディレクトリの直接の子である必要があります。

### 例

```yaml
name: Kong Portal
theme:
  name: light-theme
redirect:
  unauthenticated: login
  unauthorized: unauthorized
  login: dashboard
  logout: ''
  pending_approval: ''
  pending_email_verification: ''
collections:
  posts:
    output: true
    route: /:stub/:collection/:name
    layout: post.html
```

* `name`: 
  * **必須** : true
  * **タイプ** :`string`
  * **説明** ：名前属性は、ブラウザタブでポータルのタイトルを設定するなどのメタ情報に使用されます。
  * **例** ：`Kong Portal`

* `theme` 
  * **必須** : true
  * **タイプ** :`object`
  * **説明** ：テーマオブジェクトは、ポータルにレンダリングするテーマやテーマスタイルのオーバーライドを宣言するために使用されます。オーバーライドは必須ではありませんが、使用するテーマは必ず宣言する必要があります。

* `redirect` 
  * **必須** : true
  * **タイプ** :`object`
  * **説明** ：リダイレクトオブジェクトは、特定のアクション後にユーザーをリダイレクトする方法をKongに伝えます。これらの値のいずれかが設定されていない場合、Kongはアクションに基づいたデフォルトのテンプレートを提供します。各キーは実行されるアクション名を表し、値はアプリケーションがユーザーをリダイレクトするルートを表します。

* `collections` 
  * **必須** : false
  * **タイプ** :`object`
  * **説明** ：コレクションは、コンテンツのセットをグループとしてレンダリングできる強力なツールです。コレクションとしてレンダリングされるコンテンツは、レイアウトだけでなく、構成可能なルートパターンも共有します。詳細については、[テンプレートの使用](/gateway/{{page.release}}/kong-enterprise/dev-portal/working-with-templates/)ガイドの「[コレクション](/gateway/{{page.release}}/kong-enterprise/dev-portal/working-with-templates/#collections)」のセクションをご覧ください。

ルーター構成ファイル（オプション）
-----------------

### パス

* **形式：** `router.conf.yaml`
* **ファイル拡張子：** `.yaml`

### 説明

このオプションの構成ファイルは、ポータルのデフォルトのルーティングシステムをハードコードされた値で上書きします。これは、単一ページのアプリケーションを実装する場合や、静的なサイト構造を設定する場合に役立ちます。ルーター構成ファイルは1つしか存在できず、`router.conf.yaml`と命名され、ルートディレクトリの直接の子である必要があります。

### 例

`router.conf.yaml`ファイルには、キーと値のペアが必要です。キーは設定するルートであり、値はそのルートを解決するコンテンツファイルパスである必要があります。ルートはバックスラッシュから始まる必要があります。`/*`は予約済みのルートであり、キャッチオール/ワイルドカードとして機能します。要求されたルートが設定ファイルで明示的に定義されていない場合、ポータルはワイルドカードルートがあればそれを使用します。

```yaml
/*: content/index.txt
/about: content/about/index.txt
/dashboard: content/dashboard.txt
```

コンテンツファイル
---------

### パス

* **形式：** `content/**/*`
* **ファイル拡張子:** `.txt`、`.md`、`.html`、`.yaml`、`.json`

### 説明

コンテンツファイルはポータルサイトの構造を確立し、それに付随するHTMLレイアウトにメタデータと動的コンテンツをレンダリング時に提供します。親ディレクトリが`content/`である限り、コンテンツファイルは必要な数のサブディレクトリにネストできます。

コンテンツファイルは、レンダリング時に現在のページのメタ情報とコンテンツを提供するだけでなく、ブラウザでコンテンツにアクセスできるパスを決定します。

|               コンテンツパス                |                    ポータル URL                    |
|--------------------------------------|------------------------------------------------|
| `content/index.txt`                  | `http://portal_gui_url/`                       |
| `content/about.txt`                  | `http://portal_gui_url/about`                  |
| `content/documentation/index.txt`    | `http://portal_gui_url/documentation`          |
| `content/documentation/spec_one.txt` | `http://portal_gui_url/documentation/spec_one` |

### 内容

    ---
    title: homepage
    layout: homepage.html
    readable_by: [red, blue]
    ---
    
    Welcome to the homepage!

ファイルの内容は、 `headmatter`と`body` 2 つの部分に分けられます。

### ヘッダー情報

サンプルファイルの内容で最初に注目すべき点は、先頭にある2セットの`---`区切り文字です。これらのマーカー内に含まれるテキストは`headmatter`と呼ばれ、常に有効な`yaml`として解析および検証されます。`headmatter`には、ファイルが正常にレンダリングされるために必要な情報と、テンプレート内でアクセスするための情報が含まれています。Kongが有効な`yaml`キーと値のペアを解析し、コンテンツの対応するHTMLテンプレート内で使用できるようになります。レンダリング時にKongにとって特別な意味を持つ、事前に定義された属性がいくつかあります。

* `title`：

  * **必須** : false
  * **タイプ** :`string`
  * **説明文** ：title属性は必須ではありませんが、設定を推奨しています。設定すると、ブラウザタブのページのタイトルが設定されます。
  * **例** ：`homepage`

* `layout`

  * **必須** : true
  * **タイプ** :`string`
  * **説明** :レイアウト属性はコンテンツごとに必要で、ページをレンダリングするためにどのHTMLレイアウトを使用するかを決定します。この属性は、現在のテーマレイアウトディレクトリ \( `themes/<theme-name id="sl-md0000000">/layouts` \) のルートを想定しています。
  * **例** : `bio.html` または `team/about.html`

* `readable_by`

  * **必須** : false
  * **タイプ** : `array` \(複数\)、 `string` \(単数\)
  * **説明** : オプションの`readable_by`属性は、どの開発者がコンテンツにアクセスできるかを決定します。上記の例の場合、「red」または「blue」のrbacロールを持つ開発者だけが`/homepage`ルートにアクセスできます。
  * **例** ：`[red, blue]`（複数）、 `red`（単数）

* `route`

  * **必須** : false
  * **タイプ** :`string`
  * **説明** ：このオプション属性はKongがコンテンツに割り当てる生成ルートを上書きし、ここに含まれるルートで置き換えます。
  * **例** ：`route: /example/dog`は、自動生成された`/homepage`ルートの代わりに、上のサンプルページを`<url id="sl-md0000000">/example/dog`にレンダリングします。

* `output`

  * **必須** : false
  * **タイプ** :`boolean`
  * **デフォルト** : `true`
  * **説明** ：このオプションの属性はデフォルトで `true`で、コンテンツの一部をレンダリングするかどうかを決定します。この値が`false`に設定されている場合、ルートまたはページは作成されません。

* `stub`

  * **必須** : false
  * **タイプ** :`string`
  * **説明** : カスタム ルーティングを決定するために `collection` 構成によって使用されます。コレクションの詳細については、 *テンプレートの使用* ガイドのコレクションセクションをご覧ください。

#### 本文

ヘッドマターの下に位置する情報は、コンテンツ本体を表しています。本文の内容は自由形式で、ファイル パスに含まれるファイル拡張子によって解析されます。上記の例の場合、ファイルは `.txt` であり、テンプレート内でそのまま使用できます。

仕様ファイル
------

### パス

* **形式：** `specs/**/*`
* **ファイル拡張子：** `.yaml` 、`.json`

### 説明

仕様は、ページのレンダリングに必要な動的データと、ユーザーが`headmatter`として提供したいメタデータを提供するという点で、`content`ファイルと似ています。これらがポータルに提供される形式は、以下の例に示すように、 `content`ファイルとは異なります。
> 
> スペックフォルダ構造はフラットに保つことをお勧めします。仕様ファイルは有効な OAS または Swagger の `.yaml`/`.yml` または `.json` ファイルである必要があります。

### 内容

    swagger: "2.0"
    info:
      version: 1.0.0
      title: Swagger Petstore
      license:
        name: MIT
    host: petstore.swagger.io
    basePath: /v1
    x-headmatter
      - key1: val1
      - key2: val2
    ...

仕様ファイルの内容自体は、有効な OAS または Swagger 仕様である必要があります。
ヘッドマターを仕様に組み込むには、`x-headmatter` キーを
スペックオブジェクトのルートへ含めます。たとえば、次のような場合に便利です。
`x-headmatter.layout` で独自のレンダラーテンプレートを提供するか、仕様のデフォルトルートを`x-headmatter.route` 経由で上書きしたい場合です
。

例：

    swagger: "2.0"
    info:
      version: 1.0.0
      title: Swagger Petstore
      license:
        name: MIT
    host: petstore.swagger.io
    basePath: /v1
    x-headmatter:
      layout: custom/my-spec-renderer.html         <- Custom Layout
      route: my-special-route/myfirstspec          <- Custom Route
    ...

仕様はコレクションです。つまり、その `layout` と `route` はファイル自体ではなく、ポータル構成によって決定されます。使用は、デフォルトでは`system/spec-renderer.html` レイアウトで、ルートパターン `/documentation/:name`（`:name`は特定のスペックファイルの名前）の下にレンダリングされます。つまり、パスが`specs/myfirstspec.json`の仕様は、ポータルでは`/documentation/myfirstspec`としてレンダリングされます。   
ハードコードされたスペックコレクションの設定を上書きしたい場合は、`portal.conf.yaml` に独自の構成を含めることができます。詳細については、 `Working with Templates`ガイドの「コレクション」のセクションをご覧ください。

[ポータルファイル API](/dev-portal/api/v1/) を使用して、コンテンツ、仕様、テーマファイルを `POST`、`GET`、`PATCH`、`DELETE` することもできます。

テーマファイル
-------

### テーマディレクトリ構造

テーマディレクトリには、ポータルテーマのさまざまなインスタンスが含まれており、それぞれがHTML/CSS/JSを介して開発者ポータルのルックアンドフィールを決定します。レンダリング時に使用されるテーマは、 `portal.conf.yaml`内の`theme.name`を設定することによって決まります。`theme.name`を`best-theme`に設定すると、ポータルは`themes/best-theme/**`の下にあるテーマファイルを読み込みます。

各テーマファイルは、複数のフォルダで構成されています。

* **assets/** 
  * assets ディレクトリには、レイアウト/パーシャルがレンダリング時にリファレンスする静的アセットが含まれています。CSS、JS、フォント、画像ファイルが含まれます。

* **layouts/** 
  * レイアウトディレクトリには、headmatterの`layout`属性を介して`content`参照されるHTMLページテンプレートが含まれています （`content`セクション参照）。

* **partials/** 
  * パーシャルディレクトリには、レイアウトによって参照されるHTMLパーシャルが含まれています。レガシーポータルでレイアウトとパーシャルが相互作用していた方法を比較できます。

* **theme.conf.yaml** 
  * この設定ファイルは、CSS変数として参照するためにテンプレートで使用できる色とフォントのデフォルトを設定します。また、Kong Managerの外観ページで使用できるオプションも決定します。

### テーマアセット

#### パス

* **形式：** `theme/*/assets/**/*`

#### 説明

assetフォルダには、テンプレートが参照するCSS/JS/フォント/画像が含まれています。

テンプレートからアセットファイルにアクセスするには、Kongが選択したテーマのルートからのパスを想定していることに注意してください。

|                      アセットパス                      |                                        HREF要素                                        |
|--------------------------------------------------|--------------------------------------------------------------------------------------|
| `themes/light-theme/assets/images/image1.jpeg`   | `<img src="assets/images/image1" id="sl-md0000000">`                                 |
| `themes/light-theme/assets/js/my-script.js`      | `<script src="assets/js/my-script.js" id="sl-md0000000"></script>`                   |
| `themes/light-theme/assets/styles/my-styles.css` | `<link href="assets/styles/normalize.min.css" rel="stylesheet" id="sl-md0000000" />` |
> 
> 注：`theme/*/assets/`ディレクトリにアップロードされる画像ファイルは、SVG、テキスト文字列、または`base64`でエンコードされている必要があります。 `base64`画像は、提供時にデコードされます。

### テーマレイアウト

#### パス

* **形式：** `theme/*/layouts/**/*`
* **ファイル拡張子：** `.html`

#### 説明

レイアウトは、レンダリングするページの HTML スケルトンとして機能します。レイアウト ディレクトリ内の各ファイルは、`html` ファイル タイプであることが必要です。それらは通常の `html` として存在することも、ポータルのテンプレート構文を介してパーシャルおよび親レイアウトをリファレンスすることもできます。レイアウトは、`content` で設定された `headmatter` 属性と `body` 属性にもアクセスできます。

次の例は、典型的なレイアウトがどのようになるかを示しています。

{% raw %}

```html
<div class="homepage" id="sl-md0000000">
  {(partials/header.html)}       <- syntax for calling a partial within a template
  <div class="page" id="sl-md0000000">
    <div class="row" id="sl-md0000000">
      <h1 id="sl-md0000000">{{page.title}}</h1>    <- 'title' retrieved from page headmatter
    </div>
    <div class="row" id="sl-md0000000">
      <p id="sl-md0000000">{{page.body}}</p>       <- 'body' retrieved from page body
    </div>
  </div>
  {(partials/footer.html)}
</div>
```

{% endraw %}

この例で使用されているテンプレート構文の詳細については、[テンプレートガイド](/gateway/{{page.release}}/kong-enterprise/dev-portal/working-with-templates/)を参照してください。

### テーマのパーシャル

#### パス

* **形式：** `theme/*/partials/**/*`
* **ファイル拡張子：** `.html`

#### 説明

パーシャルはレイアウトとよく似ています。構文は同じで、内部で他のパーシャルを呼び出すことができ、レンダリング時に同じデータ/ヘルパーにアクセスできます。パーシャルとレイアウトの違いは、レイアウトはページを構築するためにパーシャルを呼び出しますが、パーシャルはレイアウトを呼び出せないということです。

以下の例は、上の例で参照された`header.html`パーシャルを示しています。

{% raw %}

```html
<header id="sl-md0000000">
  <div class="row" id="sl-md0000000">
    <div class="column" id="sl-md0000000">
      <img src="{{page.logo}}" id="sl-md0000000">      <- can access the same page data the parent layout
    </div>
    <div class="column" id="sl-md0000000">
      {(partials/header_nav.html)}   <- partials can call other partials
    </div>
  </div>
</header>
```

{% endraw %}

### テーマ構成ファイル

#### パス

* **形式：** `theme/*/theme.conf.yaml`
* **ファイル拡張子：** `.yaml`

#### 説明

テーマ設定ファイルは、レンダリング時にテーマがテンプレート/CSSで使用できる色/フォント/画像の値を決定します。これは、すべてのテーマのルートに必要です。テーマ設定ファイルは1つだけです。`theme.conf.yaml` という名前で、テーマのルートディレクトリの直接の子でなければなりません。

