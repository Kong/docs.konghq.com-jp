---
nav_title: "概要"
---
CORSプラグインを使用すると、オリジン間リソース共有（CORS）をサービスまたはルートに追加できます。

CORSの制限事項
---------

クライアントがブラウザの場合、このプラグインにはプリフライト`OPTIONS`リクエストのカスタム`Host`ヘッダーの指定を防止する[CORS指定](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)の制限が原因で発生する既知の問題があります。

この制限が原因で、このプラグインは`paths`設定で構成されたルートのみで機能します。CORSプラグインは、カスタムDNS（`hosts`プロパティ）を使用して解決されたルートには機能しません。

ルートに`paths`を構成する方法については、[プロキシ参照](/gateway/latest/reference/proxy)を参照してください。

