---
title: "テーマファイル"
content-type: "reference"
---
テーマファイル
-------

### テーマのディレクトリ構造

テーマファイルを使用すると、HTML、CSS、JavaScript を使用して Dev Portal の外観と操作性を決定できます。テーマディレクトリには、ポータルテーマのさまざまなインスタンスが含まれています。レンダリング中に使用されるテーマは、`portal.conf.yaml` ファイルの `theme.name` を設定することによって決まります。たとえば、 `theme.name`を`best-theme`に設定すると、Dev Portal はテーマファイルを`themes/best-theme/**`に読み込みます。

テーマファイルは、以下のディレクトリで構成されています。

* **assets/** ：レンダリング中にレイアウトとパーシャルが参照する静的アセットが含まれます。CSS、JS、フォント、画像ファイルが含まれます。
* **layouts/** : `content`が`layout`属性を介して参照するHTMLページテンプレートが含まれています`headmatter`
* **partials/** ：レイアウトによって参照されるHTMLパーシャルが含まれます。
* **theme.conf.yaml** : テンプレートでCSS変数として使用できる色とフォントのデフォルトを設定します。また、Kong Managerの外観ページで使用できるオプションも決定します。

テーマアセット
-------

### パス

* **形式：** `theme/*/assets/**/*`

### 説明

アセットディレクトリには、テンプレートで参照できるCSS、JavaScript、フォント、画像が含まれます。

テンプレートからアセットファイルにアクセスするには、{{site.base_gateway}}が選択したテーマのルートからのパスを想定していることに注意してください。

|                      アセットパス                      |                                        HREF要素                                        |
|--------------------------------------------------|--------------------------------------------------------------------------------------|
| `themes/light-theme/assets/images/image1.jpeg`   | `<img src="assets/images/image1" id="sl-md0000000">`                                 |
| `themes/light-theme/assets/js/my-script.js`      | `<script src="assets/js/my-script.js" id="sl-md0000000"></script>`                   |
| `themes/light-theme/assets/styles/my-styles.css` | `<link href="assets/styles/normalize.min.css" rel="stylesheet" id="sl-md0000000" />` |

{:.note}
> 
> **注意:** `theme/*/assets/`ディレクトリにアップロードされる画像ファイルは、 `svg`テキスト文字列または`base64`エンコードのいずれかである必要があります。`base64` 画像は提供時にデコードされます。

テーマレイアウト
--------

### パス

* **形式：** `theme/*/layouts/**/*`
* **ファイル拡張子：** `.html`

### 説明

レイアウトは、レンダリングするページの HTMLスケルトンとして機能します。レイアウトディレクトリ内の各ファイルには、 `html`ファイルタイプが必要です。これらは、バニラ`html`として存在することも、ポータルのテンプレート構文を介して部分レイアウトと親レイアウトを参照することもできます。レイアウトは、 `content`で設定された`headmatter`属性と`body`属性にもアクセスできます。

この例は、典型的なレイアウトがどのようなものかを示しています：

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

テーマのパーシャル
---------

### パス

* **形式：** `theme/*/partials/**/*`
* **ファイル拡張子：** `.html`

### 説明

パーシャルはレイアウトとよく似ています。構文は同じで、内部で他のパーシャルを呼び出すことができ、レンダリング中に同じデータ/ヘルパーにアクセスできます。レイアウトではページを構築するためにパーシャルが必要ですが、パーシャルはレイアウトを呼び出せないという点で、パーシャルはレイアウトとは異なります。

この例は、前の例で参照した`header.html`部分を示しています。

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

テーマ構成ファイル
---------

### パス

* **形式：** `theme/*/theme.conf.yaml`
* **ファイル拡張子：** `.yaml`

### 説明

テーマ設定ファイルは、テーマがレンダリング中にテンプレートやCSSで使用できる色、フォント、画像の値を決定します。これは、すべてのテーマのルートに必要です。テーマ設定ファイルは 1 つだけです。名前は`theme.conf.yaml`で、テーマのルートディレクトリの直接の子である必要があります。

