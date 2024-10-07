---
title: "導入"
book: "plugin_dev"
chapter: 1
---
カスタムプラグインを構築する前に、どのように{{site.base_gateway}}が構築されているか、
どのようにNginxと統合されているか、どのように高性能な[Lua](https://www.lua.org/about.html)言語が
使用されているかを理解するのは重要です。

Nginx では [lua\-nginx\-module](https://github.com/openresty/lua-nginx-module) を使って Lua の開発が可能です。このモジュールで Nginx をコンパイルする代わりに、Kong は `lua-nginx-module` を含む [OpenResty](https://openresty.org/) と一緒に配布されます。OpenResty は Nginx のフォーク *ではなく* 、その機能を拡張したモジュールのバンドルです。


{{site.base_gateway}} は、一般的に *プラグイン* と呼ばれるLua モジュールをロードして実行するために
設計された Lua アプリケーションです。{{site.base_gateway}}は、
SDK、データベース抽象化、移行などの幅広いプラグイン環境を提供します。

プラグインは、リクエスト/レスポンスオブジェクトまたはネットワークストリームと対話して任意のロジックを実装するためのLuaモジュールで構成されています。{{site.base_gateway}} は、プラグイン、{{site.base_gateway}} コア、およびその他のコンポーネント間の相互作用を容易にするために使用されるLua関数のセットである **Plugin Development Kit** （PDK）を提供します

このガイドでは、プラグインの構造、プラグインで拡張できる機能、プラグインの配布とインストールの方法について詳しく説明します。PDK の完全なリファレンスについては、[プラグイン開発キット](/gateway/{{page.release}}/plugin-development/pdk)リファレンスを参照してください。

