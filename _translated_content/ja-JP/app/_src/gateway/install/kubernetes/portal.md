---
title: "Kong Developer Portal"
book: "kubernetes-install"
chapter: 5
---
このガイドでは、 {{ site.ee_product_name  }}に付属するセルフホスト型ポータルであるKong Developer Portalをデプロイする方法を説明します。新たに開発者ポータルを展開する場合は、<a href="https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=kubernetes-install">{{ site.konnect_saas }} Developer Portal</a>の使用をお勧めします。

前提条件
----

* [{{ site.base_gateway }} がインストール済み](/gateway/{{ page.release }}/install/kubernetes/proxy/)
* KongのAdmin APIはローカルマシンからHTTP経由でアクセスできます。

インストール
------

Kong Developer Portalは、API仕様とコンテンツを管理するためにポータルAPIのデータベースにアクセスする必要があるため、 `kong-cp`デプロイメントの一部としてデプロイされます。

1. 次の値を既存の`values-cp.yaml`ファイルにマージして`values-cp.yaml`ファイルを更新し、Dev Portal を有効にします。

   ```yaml
   env:
     # Portal configuration
     # CHANGE THESE VALUES
     portal_gui_protocol: http
     portal_gui_host: portal.example.com
     portal_api_url: http://portalapi.example.com
     portal_session_conf: '{"cookie_name": "portal_session", "secret": "PORTAL_SUPER_SECRET", "storage": "kong", "cookie_secure": false, "cookie_domain":".example.com"}'
   
   # Enable Developer Portal
   enterprise:
     portal:
       enabled: true
   ```

2. `values-cp.yaml`のポータルホスト名の値を更新します。

   * `env.portal_gui_protocol`：GUIで使用するプロトコル（HTTPまたはHTTPS）
   * `env.portal_gui_host`: ポータルへのアクセスに使用するホスト名
   * `env.portal_api_url`: Dev Portalデータのパブリックアクセスが可能なAPI URL
   * `env.portal_session_conf`: 次の値を更新します。`secret` と `cookie_domain`

{% include md/k8s/ingress-setup.md service="portal" release="cp" type="public" skip_release=true skip_ingress_controller_install=true skip_dns=true %}

{% include md/k8s/ingress-setup.md service="portalapi" release="cp" type="public" skip_ingress_controller_install=true skip_dns=true %}

1. `Ingress` IP アドレスを取得し、イングレスアドレスを指すように DNS レコードを更新します。DNS を手動で構成することも、 [external\-dns](https://github.com/kubernetes-sigs/external-dns)などのツールを使用して DNS 構成を自動化することもできます。

   `portal`と`portalapi`はいずれも同じ`LoadBalancer`を使用してアクセスできます。次のコマンドによって返されるエンドポイントを使用して、`portal.example.com`と`portalapi.example.com`の両方を構成します。

   ```bash
   kubectl get ingress -n kong kong-cp-kong-portal -o jsonpath='{range .status.loadBalancer.ingress[0]}{@.ip}{@.hostname}{end}'
   ```

Dev Portal を有効化
---------------

新しい{{ site.base_gateway }}インストールでは、Kong Developer Portalはデフォルトで有効になっていません。Admin APIを使用してポータルを有効にします。ドメインを変更し、インストールに一致するように`Kong-Admin-Token`の値を変更することを確認してください。

```bash
curl -X PATCH http://admin.example.com/default/workspaces/default -d config.portal=true -H 'Kong-Admin-User: kong_admin' -H 'Kong-Admin-Token: YOUR_PASSWORD'
```

テスト
---

ブラウザで[http://portal.example.com/](http://portal.example.com/)にアクセスします。3 つのサンプル OpenAPI 仕様を含む"Built with Kong"というページが表示されます。

トラブルシューティング
-----------

### ログインできません

画面が更新されたのにDev Portalにログインしていない場合は、`values-cp.yaml`に`cookie_domain`が正しく設定されているか確認してください。デフォルトでは、CookieはポータルAPIドメインに設定されます。ポータルが、ポータルAPIから認証Cookieを読み取れるようにするには、`cookie_domain`が必要です。

