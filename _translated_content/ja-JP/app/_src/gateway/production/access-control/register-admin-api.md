---
title: "管理者の例"
badge: "enterprise"
---
Admin APIを使用して管理者を招待し、登録します。自動化に使用できます。

前提条件
----

次の環境変数を設定します。

```bash
export USERNAME=<username id="sl-md0000000">
export EMAIL=<email id="sl-md0000000">
export WORKSPACE=<workspace id="sl-md0000000">
export HOST=<admin_api_host id="sl-md0000000">
export TOKEN=Kong-Admin-Token:<super_admin_token id="sl-md0000000">
```

以下に例を示します。

```bash
export USERNAME=drogon
export EMAIL=test@test.com
export WORKSPACE=default
export HOST=127.0.0.1:8001
export ADMIN_TOKEN=Kong-Admin-Token:hunter2
```

HTTPieとjqを使用すると効果的です。

管理者を招待する
--------

環境変数を手動で作成するか、`jq`をエコーおよびパイプして、登録URLからトークンを抽出して保存します。

{% navtabs %}
{% navtab Manual method example %}

1. 登録URLにリクエストを送信する

```bash
curl -i http://localhost:8001/$WORKSPACE/admins/$USERNAME?generate_register_url=true \
  -H Kong-Admin-Token:$TOKEN
```

2. レスポンスをコピーして、環境変数としてエクスポートします。たとえば次のようにします。

```bash
export REGISTER_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NDUwNjc0NjUsImlkIjoiM2IyNzY3MzEtNjIxZC00ZjA3LTk3YTQtZjU1NTg0NmJkZjJjIn0.gujRDi2pX_E7u2zuhYBWD4MoPFKe3axMAq-AUcORg2g
```

{% endnavtab %}
{% navtab Programmatic method (requires jq) %}

```bash
REGISTER_TOKEN=$(curl -i -X http://localhost:8001/$WORKSPACE/admins/$USERNAME?generate_register_url=true -H Kong-Admin-Token:$TOKEN | jq .token -r)
```

{% endnavtab %}
{% endnavtabs %}

管理者を登録する
--------

```bash
curl -i -X http://localhost:8001/$WORKSPACE/admins/register \
  --data token=$REGISTER_TOKEN \
  --data username=$USERNAME \
  --data email=$EMAIL \
  --data password="<new_password id="sl-md0000000">"
```

