---
title: "イベントフックの例"
badge: "enterprise"
---
{% include_cached /md/enterprise/event-hooks-intro.md %}

Webフック
------

webhookイベントフックは、イベントデータをペイロードとして、指定されたURLにJSON POSTリクエストを送信します。この例では、webhookのテストに役立つサイト[https://webhook.site](https://webhook.site)を使用します。

webhookイベントフックを作成する方法は次のとおりです。

1. ウェブブラウザで[https://webhook.site](https://webhook.site)に移動してURLを生成します。

2. **一意の URL** の横にある **クリップボードにコピー** を選択します。

3. 次のHTTPリクエストを使用して、`consumers`イベント（イベントのためにイベントフックがリッスンするKongエンティティ）上、`crud`ソース上（ログをトリガーするアクション）、および
   手順2でコピーしたURLにwebhookイベントフックを作成します。

   ```sh
   curl -i -X POST http://{HOSTNAME}:8001/event-hooks \
     -d source=crud \
     -d event=consumers \
     -d handler=webhook \
     -d config.url={WEBHOOK_URL}
   ```

4. 手順2のURLに移動します。`ping`タイプのPOSTリクエストが表示され、このwebhookの作成について
   webhookエンドポイントに通知されます。

5. Kong Manager または Kong Admin API で、任意のワークスペースからコンシューマを追加します。

{% capture the_code %}
{% navtabs %}
{% navtab Kong Manager %}

1. ワークスペースを選択します。
2. 左のナビゲーションで **コンシューマ** を選択します。
3. **新しいコンシューマ** ボタンを選択します。
4. **ユーザー名** を入力します。
5. （オプション） **カスタム ID** と **任意のタグ** を入力します。
6. **作成** ボタンを選択します。

{% endnavtab %}
{% navtab Admin API %}

Kong Admin API のインスタンスに次の HTTP リクエストを送信して、コンシューマ Ada Lovelace を作成します。

    curl -i -X POST http://{HOSTNAME}:8001/consumers \
      -d username="Ada Lovelace"

{% endnavtab %}
{% endnavtabs %}

{% endcapture %}

{{ the_code | indent }}

1. [https://webhook.site](https://webhook.site) ページから URL を確認します。
   ペイロードに新しいコンシューマのデータを含むエントリが表示されます。

   ```json
   {
   "source": "crud",
   "entity": {
     "created_at": 1627581878,
     "type": 0,
     "username": "Ada Lovelace",
     "id": "0fd2319f-13ea-4582-a448-8d11893026a8"
   },
   "event": "consumers",
   "operation": "create",
   "schema": "consumers"
   }
   ```

カスタムwebhook
-----------

カスタム webhook イベントフックは完全にカスタマイズ可能なリクエストです。カスタムWebhookは、
サービスとの直接統合を構築するのに便利です。カスタム Webhook は完全に構成可能であるため、より複雑な構成になります。
カスタム Webhook は、構成可能な本文、フォームペイロード、およびヘッダーでの Lua テンプレートをサポートします。テンプレートに使用できるフィールドの
リストについては、[ソース](/gateway/api/admin-ee/latest/#/Event-hooks/get-event-hooks-sources)エンドポイントを参照してください。

次の例では、新しい管理者が{{site.base_gateway}}に招待されるたびにSlackにメッセージが送信されます。Slackでは[着信ウェブフック](https://slack.com/help/articles/115005265063-Incoming-webhooks-for-Slack#set-up-incoming-webhooks)が許可されており、これを使用して、Kongのイベントフック機能との統合を構築できます。

カスタム webhook イベントフックを作成するには以下のようにします。

1. [Slackでアプリを作成します。](https://api.slack.com/apps?new_app=1)

2. 新しいアプリの設定で受信ウェブフックを有効にします。

3. **ワークスペースに新しいWebhookを追加** を選択し、通知を受け取るチャネルを選択して、 **許可** を選択します。

4. **Webhook URL** （例： `https://hooks.slack.com/services/foo/bar/baz`）をコピーします。

5. `admins`イベント（event hookがイベントをリッスンするKongエンティティ）と`crud`ソース（ロギングをトリガーするアクション）にwebhook event hookを作成します。

   次の HTTP 要求を使用して、ペイロードを「`{% raw %}"Admin account \`{{ entity.username }}\` {{ operation }}d; e\-mail address set to \`{{ entity.email }}\`"{% endraw %}\`」のようにフォーマットします。

   ```sh
   curl -i -X POST http://{HOSTNAME}:8001/event-hooks \
     -d source=crud \
     -d event=admins \
     -d handler=webhook-custom \
     -d config.method=POST \
     -d config.url={WEBHOOK_URL} \
     -d config.headers.content-type="application/json" \
     -d config.payload.text={% raw %}"Admin account \`{{ entity.username }}\` {{ operation}}d; email address set to \`{{ entity.email }}\`"{% endraw %}
   ```

6. RBACを有効にします。

{% capture anInclude %}
{% include_cached /md/enterprise/turn-on-rbac.md %}
{% endcapture %}

{{ anInclude | indent }}

7. Kong Manager または Kong Admin API を使用して管理者を招待します。

{% capture the_code2 %}
{% navtabs %}
{% navtab Kong Manager %}

1. Kong Managerにアクセスするか、すでに開いている場合はページを再読み込みすると、ログイン画面が表示されます。
2. 組み込みのスーパー管理者アカウント`kong_admin`とそのパスワードでKong Managerにログインします。これは、インストール中に移行を実行したときに使用した初期`KONG_PASSWORD`です。
3. **チーム（Teams）> 管理者（Admins）** タブから、 **管理者を招待（Invite Admin）** をクリックします。
4. 新しい管理者の **メールアドレス** と **ユーザー名** を入力します。
5. 招待を送信するには、 **管理者を招待** をクリックします。入門ガイドのこの時点では、SMTPがまだ設定されていない可能性が高いため、Eメールは送信されません。

{% endnavtab %}
{% navtab Admin API %}

Kong Admin APIのインスタンスに次のHTTPリクエストを送信して、管理者Arya Starkを作成します。

{:.note}
> 
> **注意:** `{KONG_ADMIN_PASSWORD`\} を`kong_admin`パスワードに置き換えてください。これは
> インストール中に移行を実行したときに使用した最初の`KONG_PASSWORD`です。

```sh
curl -i -X POST http://{HOSTNAME}:8001/admins \
-d username="Arya Stark" \
-d email=arya@gameofthrones.com \
-H Kong-Admin-Token:{KONG_ADMIN_PASSWORD}
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ the_code2 | indent }}

その後、お客様が`config.payload.text`を含むメッセージを選択したSlackチャンネル内で、
メッセージを受信します。

ログ
---

ログイベントフックは、指定されたイベントとペイロードの内容を{{site.base_gateway}}ログに記録します。

ログイベントフックを作成するには:

1. `consumers`イベント \(イベントフックがイベントをリッスンする Kong エンティティ\)と、
   `crud`ソース \(ログ記録をトリガーするアクション\) に、以下の HTTP リクエストを使用してログイベントフックを作成します。

   ```sh
   curl -i -X POST http://{HOSTNAME}:8001/event-hooks \
     -d source=crud \
     -d event=consumers \
     -d handler=log
   ```

2. Kong Manager または Kong Admin API で、任意のワークスペースからコンシューマを追加します。

{% capture the_code3 %}
{% navtabs %}
{% navtab Kong Manager %}

1. ワークスペースを選択します。
2. 左のナビゲーションで **コンシューマ** を選択します。
3. **新しいコンシューマ** ボタンを選択します。
4. **ユーザー名** を入力します。
5. （オプション） **カスタム ID** と **任意のタグ** を入力します。
6. **作成** ボタンを選択します。

{% endnavtab %}
{% navtab Admin API %}

Kong Admin APIのインスタンスに次のHTTPリクエストを送信して、コンシューマであるElizabeth Bennetを作成します。

```sh
curl -i -X POST http://{HOSTNAME}:8001/consumers \
-d username="Elizabeth Bennet"
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ the_code3 | indent }}

3. 通常は、`/usr/local/kong/logs/error.log`からアクセスできるKongのエラーログのペイロードに、新しいコンシューマのデータが入ったエントリが表示されます。

   ```log
   172.19.0.1 - - [29/Jul/2021:15:57:15 +0000] "POST /consumers HTTP/1.1" 409 147 "-" "HTTPie/2.4.0"
   2021/07/29 15:57:26 [notice] 68854#0: *819021 +--------------------------------------------------+, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |[kong] event_hooks.lua:?:452 "log callback: " { "consumers", "crud", {|, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |    entity = {                                    |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |      created_at = 1627574246,                    |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |      id = "4757bd6b-8d54-4b08-bf24-01e346a9323e",|, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |      type = 0,                                   |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |      username = "Elizabeth Bennet"               |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |    },                                            |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |    operation = "create",                         |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |    schema = "consumers"                          |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 |  }, 68854 }                                      |, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   2021/07/29 15:57:26 [notice] 68854#0: *819021 +--------------------------------------------------+, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001

   ```

Lambda
------

ラムダイベントフックを使用すると、Luaコードで完全にカスタムロジックを記述でき、さまざまなKongイベントに組み込むことができます。次の例では、コンシューマが変更になるたびに、条件付きでカスタムフォーマットを使用してログエントリを書き込みます。

{:.important}
> 
> ラムダのイベントフックタイプは非常に強力です。完全にカスタムのロジックを書いて、どんなユースケースにも対応できます。
> ただし、[デフォルトではサンドボックスによって制限されています](/gateway/{{ page.release }}/reference/configuration/#untrusted_lua)。この
> サンドボックスはユーザーの安全を守るために設置されています。誤って安全でないライブラリやオブジェクトを誤ってサンドボックスに追加し、
> {{site.base_gateway}}をセキュリティの脆弱性にさらしてしまうのは簡単だからです。サンドボックス設定を変更する際は注意してください。

Lambda イベントフックを作成するには以下のようにします。

1. Luaスクリプトを作成してラムダイベントフックに読み込み、ホームディレクトリの`lambda.lua`という名前のファイルに保存します。

   ```lua
   return function (data, event, source, pid)
     local user = data.entity.username
     error("Event hook on consumer " .. user .. "")
   end
   ```

2. 次のHTTPリクエストを使用して、 `consumers`イベント（イベントフックがイベントをリッスンするKongエンティティ）と`crud`ソース（ログ記録をトリガーするアクション）にラムダイベントフックを作成します。

   ```sh
   curl -i -X POST http://{HOSTNAME}:8001/event-hooks \
     -d source=crud \
     -d event=consumers \
     -d handler=lambda \
     -F config.functions='return function (data, event, source, pid) local user = data.entity.username error("Event hook on consumer " .. user .. "")end'
   ```

3. Kong ManagerまたはKong Admin APIで、任意のワークスペースにコンシューマを追加します。

{% capture the_code4 %}
{% navtabs %}
{% navtab Kong Manager %}

1. ワークスペースを選択します。
2. 左のナビゲーションで **コンシューマ** を選択します。
3. **新しいコンシューマ** ボタンを選択します。
4. **ユーザー名** を入力します。
5. （オプション） **カスタム ID** と **任意のタグ** を入力します。
6. **作成** ボタンを選択します。

{% endnavtab %}
{% navtab Admin API %}

Kong Admin APIのインスタンスに次のHTTPリクエストを送信して、コンシューマLois Lane を作成します。

```sh
curl -i -X POST http://{HOSTNAME}:8001/consumers \
  -d username="Lois Lane"
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ the_code4 | indent }}

3. Kongのエラーログに「コンシューマLois Laneのevent hook」というエントリが表示され、通常、`/usr/local/kong/logs/error.log`からアクセスできます。

   ```log
   2021/07/29 21:52:54 [error] 114#0: *153047 [kong] event_hooks.lua:190 [string "return function (data, event, source, pid)..."]:3: Event hook on consumer Lois Lane, context: ngx.timer, client: 172.19.0.1, server: 0.0.0.0:8001
   ```

