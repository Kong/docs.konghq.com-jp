---
title: "Kong Gateway環境変数"
content-type: "how-to"
---
環境変数
----


{{site.base_gateway}}は、環境変数を使用して完全に構成されます。

[`kong.conf`で定義されているすべてのパラメータ](/gateway/{{page.release}}/reference/configuration/)は、環境変数を介して管理できます。`kong.conf`からプロパティを読み込む場合、{{site.base_gateway}}は既存の環境変数を最初にチェックします。

環境変数を使用して設定を上書きするには、
環境変数の前に`KONG_`という接頭辞を付けて設定の名前を宣言します。

たとえば、 `log_level`パラメータをオーバーライドするには、次のようにします。

    log_level = debug # in kong.conf

`KONG_LOG_LEVEL`を環境変数として設定します。

```bash
export KONG_LOG_LEVEL=error
```

詳細情報
----

* [使用方法：`kong.conf`](/gateway/{{page.release}}/production/kong-conf/)
* [KongでAPIとウェブサイトを提供する方法](/gateway/{{page.release}}/production/website-api-serving/)
* [構成パラメータの参照](/gateway/{{page.release}}/reference/configuration/)

