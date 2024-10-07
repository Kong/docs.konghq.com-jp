---
title: "Kong Dev Portal での JavaScript アセットの追加と使用"
badge: "enterprise"
---
Kong Dev Portal には、Vue、React、jQuery がすでにロードされた状態で出荷されます。
これらのライブラリを利用してカスタム・インタラクティブ・ウェブページを書き込んだり、
追加のJavaScriptを読み込んだりできます。
> 
> 注: このガイドは、サーバー側のルーティングを変更せずに JavaScript アセットを追加/使用するためのものです。[SPA の詳細については、Dev Portal をご覧ください](/gateway/{{page.release}}/kong-enterprise/dev-portal/customize/single-page-app/)。

前提条件
----

* ポータルのレガシーがオフになっている
* Kong Dev Portal が有効になっていて実行中である
* [kong\-portal\-cliツール](/gateway/{{page.release}}/kong-enterprise/dev-portal/cli/)がローカルにインストールされている

JS アセットの追加
----------
> 
> 警告：互換性の問題により、`layouts/system/spec-render.html`レイアウトではReact 15以外のReactバージョンは使用しないでください。デフォルトのベーステーマに含まれているReactバージョンの使用をお勧めします。

JavaScriptアセットを追加するには、以下の手順を実行します。

1. [kong\-portal\-templates](https://github.com/Kong/kong-portal-templates)リポジトリをクローンします。
2. JavaScriptファイルを`themes/base/js`フォルダに追加します。
3. [kong\-portal\-cli\-tool](/gateway/{{page.release}}/kong-enterprise/dev-portal/cli/)を使用してデプロイします。

JS アセットを読み込み
------------

これらのスクリプトが読み込まれる`partials/theme/required-scripts.html`を含む任意のレイアウト/パーシャルで、既存のVueとjQueryを利用できます。

デフォルトでは、Reactは`layouts/system/spec-render.html`でのみロードされます。

すべてのページでReactまたはカスタムJavaScriptアセットを読み込む場合は、`themes/partial/foot.html`を編集できます。

{% raw %}

    {% layout = "layouts/_base.html" %}
    
    {-main-}
      {(partials/header.html)}
    
      <div class="page" id="sl-md0000000">
        {* blocks.content *}
      </div>
    
      {(partials/footer.html)}
      <script src="assets/js/third-party/react.min.js" id="sl-md0000000"></script>
    
    {-main-}

{% endraw %}

または、必要に応じて、各コンテンツページの特定のレイアウトに必要なスクリプトを読み込むこともできます。

