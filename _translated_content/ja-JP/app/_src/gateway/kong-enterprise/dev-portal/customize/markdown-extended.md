---
title: "Markdown レンダリングモジュール"
badge: "enterprise"
---
Kong Dev Portalは、[テンプレート](/gateway/{{page.release}}/kong-enterprise/dev-portal/working-with-templates/)の代わりに使用できる[Githubフレーバーのマークダウン](https://github.github.com/gfm/)（GFM）をサポートしています。テンプレートのHTMLレイアウトとパーシャルを作成する代わりに、改善されたマークダウンレンダリングモジュールでカスタムCSSを使用できます。マークダウンのサポートが拡張されたため、特にテーブルのレンダリングが大幅に向上します。

前提条件
----

* Kong Managerへのアクセス
* Dev Portalが有効かつ実行中

ドキュメント内のマークダウンレンダリングモジュールの指定
----------------------------

1. Dev Portalドキュメント用のマークダウンファイルを作成します。

2. `layout`パラメータを使用してマークダウンモジュールを呼び出します。

3. 使用する`.css`ファイルを指定します。表示されているとおり、デフォルトの Githubである`.css`を使用することも、独自のカスタム`.css`を指定することもできます。

   * すべてのマークダウンCSSクラスの前には`.markdown-body`を追加する必要があります。これは、マークダウンスタイルがポータルの他の領域に及ばないようにするためです。
   * `.css`ファイルは現在のテーマの`/assets`ディレクトリに配置する必要があります。
   * レンダラーは、 `.css`パスが`/assets`ディレクトリから開始すると想定します。以下の例を参照してください。
   * `markdown.css` の外部にある他の Dev Portal CSS で定義されている他のクラスは、レンダリングされたマークダウンを汚染する可能性があります。特定のスタイルの設定を解除するには、`/assets/style/markdown-fixes.css` ファイルを使用します。

### 例

`/content/markdown-example.md`

    ---
    layout: system/markdown.html
    css: assets/style/markdown.css
    ---

