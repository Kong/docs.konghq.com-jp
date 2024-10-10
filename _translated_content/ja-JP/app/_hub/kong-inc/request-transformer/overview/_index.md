---
nav_title: "概要"
---
KongのRequest Transformerプラグインなら、アップストリームサーバーに到達する前にリクエストを簡単に変換できます。
これらの変換は単純なものから複雑なものまで様々で、正規表現を使って受信リクエストの一部と一致させ、その一致した文字列を変数に保存し、柔軟なテンプレートを使って変換されたリクエストにその文字列を置換ます。

その他のリクエスト変換機能については、
[Request Transformer Advancedプラグイン](/hub/kong-inc/request-transformer-advanced/)をご覧ください。
高度なプラグインを使用すると、リクエスト本文で許可されるパラメータのリストを制限することもできます。

{:.note}
> 
> **注** ：

* 値に`,`（カンマ）が含まれている場合、リストのカンマ区切り形式は使用できません。代わりに配列表記を使用する必要があります。
* `X-Forwarded-*`フィールドは、Nginxがアップストリームにクライアントの詳細を知らせるために書いた非標準のヘッダーフィールドで、このプラグインでは上書きできません。これらのヘッダーフィールドを上書きする必要がある場合は、 [ポストファンクション・プラグイン](/hub/kong-inc/post-function/how-to/)をご覧ください。

実行順序
----

このプラグインは、次の順序で応答変換を実行します。

remove → rename → replace → add → append

Request Transformerプラグインの使用開始
-----------------------------

* [構成リファレンス](/hub/kong-inc/request-transformer/configuration/)
* [基本的な構成例](/hub/kong-inc/request-transformer/how-to/basic-example/)
* [プラグインの使用方法](/hub/kong-inc/request-transformer/how-to/)
* [{{site.base_gateway}}を使用したHTTPリクエストへの属性の追加](/hub/kong-inc/request-transformer/how-to/add-body-value/)
* [テンプレートを値として使用する](/hub/kong-inc/request-transformer/how-to/templates/)

