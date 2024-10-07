---
title: "Brotli圧縮によるパフォーマンスの向上"
content-type: "reference"
---

{{site.base_gateway}} は、[Nginx ディレクティブ注入メカニズム](/gateway/{{page.release}}/reference/nginx-directives/)を介して [`ngx_brotli`](https://github.com/google/ngx_brotli) モジュールをサポートします。[Brotli](https://github.com/google/brotli) は、パフォーマンスの高いウェブサイトのための圧縮アルゴリズムです。
これは、gzip や deflate などの一般的によく利用されるアルゴリズムよりも圧縮に優れた設計となっています。
アプリケーションの高速化、ページ速度の向上、送信データ量の削減、{{site.base_gateway}} の全体的なパフォーマンスの向上に使用できます。

Brotli 圧縮を有効にする
---------------

Brotli圧縮を有効にするには、`kong.conf`で次のパラメータを設定します。

    nginx_proxy_brotli = "on"
    nginx_proxy_brotli_comp_level = 5
    nginx_proxy_brotli_types = "text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js"

圧縮レベル（`nginx_proxy_brotli_comp_level`）は、処理時間 [\[1\]](#more-information) を損なうことなく、最高のgzip圧縮レベルよりも少量のペイロードを提供するため、バランスの取れたオプションとして4または5に設定することをおすすめします。

その他の情報
------

* [{{site.base_gateway}}でのNginxディレクティブの挿入](/gateway/{{page.release}}/reference/nginx-directives/)
* [Brotli](https://github.com/google/brotli)
* [`ngx_brotli`モジュール](https://github.com/google/ngx_brotli)
* \[1\] [Brotli 圧縮: コンテンツはどのくらい削減されるか？](https://paulcalvano.com/2018-07-25-brotli-compression-how-much-will-it-reduce-your-content/)

