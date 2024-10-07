---
title: "Kong Dev Portal からのシングルページアプリのホスティング"
badge: "enterprise"
---
Kong Dev Portalはデフォルトのサーバーサイドのレンダリングテーマで出荷されますが、シングルページアプリ（SPA）に交換することも可能です。こちらは、Angular CLIツールを使用したAngular形式の例ですが、わずかな調整を加えるだけで他のJavaScriptフレームワークでも同様に実行できます。

このガイドの基本的なAngularテンプレートの例を表示するには、`kong-portal-templates branch`から[`example/spa-angular`](https://github.com/Kong/kong-portal-templates/tree/example/spa-angular)ブランチにアクセスしてください。

前提条件
----

* ポータルのレガシーがオフになっている
* Dev Portalが有効かつ実行中
* kong\-portal\-cliツールがローカルにインストールされている

SPAの定義
------

シングルページアプリ \(SPA\) は、最初の読み込み時にすべてのHTML、JavaScript、CSSを読み込むウェブサイトです。サーバーから後続のページを読み込む代わりに、JavaScriptを使用してページコンテンツを動的に変更します。ポータルと統合したい既存のSPAがある場合や、さまざまなページでアプリケーションを使用するような体験を実現したい場合は、Dev PortalでSPAを使用することをおすすめします。SPAはサーバーからルーティングを制御し、代わりにクライアント側で処理します。

特定のレイアウトでのみ実行するようにカスタムJavaScriptを追加して、サーバー側のレンダリングを維持することもできます。SPAを実装せずにレイアウトにJavaScriptを追加する方法について[詳細を確認する](/gateway/{{page.release}}/kong-enterprise/dev-portal/customize/adding-javascript-assets/)。

選択
---

カタログおよび仕様ルートは、SPAで処理しないことをおすすめします。認証を使用している場合も、おそらく、どのアカウントページでもサーバー側のレンダリングをそのままにしておく方がよいでしょう。

使用開始する
------

[ポータルテンプレート](https://github.com/Kong/kong-portal-templates)リポジトリをクローン

`workspaces/default` に `router.conf.yaml` というファイルを作成します。このファイルはデフォルトのルーティングを上書きし、JavaScript 経由でルーティングを制御できるようになります。

`router.conf.yaml` yaml ファイルである必要があり、キーは各ルート、値はコンテンツまたは仕様パスです。`/*` これは、`router.conf.yaml` で指定されていないすべてのルートを対象とするキャッチオールワイルドカードであり、コレクションによって設定された、またはヘッドマターで設定されたすべてのデフォルトルーティングを上書きします。

以下の `router.conf.yaml` の例では、kong 関連のすべての機能のルーティングをハードコーディングし、他のすべてのルートについては SPA ルーティングに従います。ファイルの末尾にある `/*` ルートはワイルドカードルートです。この場合、ワイルドカード ルートは、上記のルート宣言で処理されないすべてのリクエストに対して Kong に `content/index.txt` を提供するように指示します。

    /login: content/login.txt
    /logout: content/logout.txt
    /register: content/register.txt
    /settings: content/settings.txt
    /reset-password: content/reset-password.txt
    /account/invalidate-verification: content/account/invalidate-verification.txt
    /account/resend-verification: content/account/resend-verification.txt
    /account/verify: content/account/resend-verification.txt
    /documentation: content/documentation/index.txt
    /documentation/httpbin: specs/httpbin.json
    /documentation/petstore: specs/petstore.yaml
    /*: content/index.txt


### SPA の作成

選択した JavaScript フレームワークで SPA アプリを作成します。この
例では Angular を使用しています。

    ng new

404ルートと、ポータルで保持したいすべてのルート（上記で除外されたサーバーサイドレンダリングで処理されるルートを除く）を必ず含めてください。

ビルドプロセスを実行

Angularの場合：

    ng build

ビルド出力のJSおよびCSSファイルを`workspaces/default/themes/assets/js`内のフォルダーにコピーします。

この例では、`workspaces/default/themes/assets/js/ng`の中にアンギュラービルドを配置します。

### SPAの取り付け

JSをロードするには、それをマウントする必要があります。新しいレイアウトページを作成しましょう。

`workspaces/default/themes/layouts` に `spa.html` というファイルを作成します。

このファイルには、SPAがマウントされるHTML要素と、これを実行するために必要なスクリプトが含まれている必要があります。
参考までに、SPA のビルドステップで作成されたビルドフォルダ内の`index.html`を表示します。

この例では、レイアウト テンプレートのベースとして`layouts/_base.html`を使用します。
そうすることで、<head> 要素はポータルの他のページと同じ方法で処理され、同じCSSとスクリプトが使用されます。

上部のナビゲーションバーと下部のナビゲーションバーが必要な場合は、必ずそれらをレイアウトに含めてください。

以下が結果のレイアウトです。

{% raw %}

    {% layout = "layouts/_base.html" %}
    {-main-}
    
        {(partials/header.html)}
        <div class="page" id="sl-md0000000">
            <app-root id="sl-md0000000"></app-root>
        </div>
        {(partials/footer.html)}
    
    
        <script src="assets/js/ng/runtime-es2015.js" type="module" id="sl-md0000000"></script>
        <script src="assets/js/ng/runtime-es5.js" nomodule defer id="sl-md0000000"></script>
        <script src="assets/js/ng/polyfills-es5.js" nomodule defer id="sl-md0000000"></script>
        <script src="assets/js/ng/polyfills-es2015.js" type="module" id="sl-md0000000"></script>
        <script src="assets/js/ng/styles-es2015.js" type="module" id="sl-md0000000"></script>
        <script src="assets/js/ng/styles-es5.js" nomodule defer id="sl-md0000000"></script>
        <script src="assets/js/ng/vendor-es2015.js" type="module" id="sl-md0000000"></script>
        <script src="assets/js/ng/vendor-es5.js" nomodule defer id="sl-md0000000"></script>
        <script src="assets/js/ng/main-es2015.js" type="module" id="sl-md0000000"></script>
        <script src="assets/js/ng/main-es5.js" nomodule defer id="sl-md0000000"></script>
    
    {-main-}

{% endraw %}

SPAビルドプロセスによってCSSファイルが作成された場合は、 `head.html`部分を編集してCSSファイルを含めます。

### レイアウトの読み込み

独自のレイアウトを使用するために`workspaces/default/content/index.txt`を変更します。ここで設定したタイトルは、JSセットのタイトルが読み込まれるまで表示されるタイトルになります。

{% raw %}

    ---
    layout: spa.html
    title: Home
    ---

{% endraw %}

### ポータルをデプロイ

次に、kong\-portal\-cliツールを使用してポータルをデプロイします。

テンプレートリポジトリのルートフォルダから：

    portal deploy default

