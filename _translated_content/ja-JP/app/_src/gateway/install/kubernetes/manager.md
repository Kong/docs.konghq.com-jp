---
title: "Kong Managerのインストール"
book: "kubernetes-install"
chapter: 4
---
Kong Managerは、{{ site.base_gateway }}のグラフィカルユーザーインターフェース（GUI）です。内部的にはKong Admin APIを使用して{{ site.base_gateway }}を管理および制御します。

{:.important}
> 
> Kong Managerを使用するには、ローカルマシンからKongのAdmin APIにHTTP経由でアクセスできる必要があります。

前提条件
----

* [{{ site.base_gateway }}がインストール済み](/gateway/{{ page.release }}/install/kubernetes/proxy/)
* KongのAdmin APIはローカルマシンからHTTP経由でアクセスできます。

インストール
------

Kong Managerは、Admin APIと同じノードから提供されます。Kong Managerを有効にするには、`values-cp.yaml`ファイルに次の変更を加えます。

1. `env`キーの下に、`admin_gui_url`、`admin_gui_api_url`、`admin_gui_session_conf`を設定します。

   ```yaml
   env:
     admin_gui_url: http://manager.example.com
     admin_gui_api_url: http://admin.example.com
     # Change the secret and set cookie_secure to true if using a HTTPS endpoint
     admin_gui_session_conf: '{"secret":"secret","storage":"kong","cookie_secure":false}'
   ```

2. 構成内の`example.com`をドメインに置き換えてください。

3. `enterprise`キーでKong Manager認証を有効にします。

   ```yaml
   enterprise:
     rbac:
       enabled: true
       admin_gui_auth: basic-auth
   ```

{% include md/k8s/ingress-setup.md service="manager" release="cp" type="private" skip_ingress_controller_install=true %}

テスト
---

Kong Managerのログインページを表示するには、ウェブブラウザで`env.admin_gui_url`のURLにアクセスします。デフォルトのユーザー名は「`kong_admin`」で、パスワードは前のステップで{{ site.base_gateway }}コントロールプレーン（CP）をインストールしたときに、`env.password`に設定した値です。

トラブルシューティング
-----------

### Kong Manager にログインできません

Kong をインストールする前に、 `values-cp.yaml`に`env.password`が設定されていることを確認してください。{{ site.base_gateway }}は、設定されていない場合はランダムな管理者パスワードを生成します。このパスワードは回復できないため、新しい管理者パスワードを設定するにはKongを再インストールする必要があります。

### 私のログイン認証情報は何ですか?

Kong のスーパー管理者のユーザー名は `kong_admin` で、パスワードは `values-cp.yaml` の `env.password` に設定された値です。

### Kong Managerが白い画面を表示する

`env.admin_gui_api_url`が`values-cp.yaml` に正しく設定されていることを確認してください。

