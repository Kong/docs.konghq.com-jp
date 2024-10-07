---
title: "Kong Gatewayのインストール"
book: "kubernetes-install"
chapter: 2
---
このガイドでは、[{{ site.konnect_saas }}](/konnect/gateway-manager/data-plane-nodes/) または [{{ site.kic_product_name }}](/kubernetes-ingress-controller/latest/get-started/) を使用せずに Kubernetes に {{ site.base_gateway }} をデプロイする方法について説明します。

{:.important}
> 
> **デプロイメントの複雑さを軽減するために、新規インストールには{{ site.konnect_saas }}をお勧めします。**
> <br />
> Kongがコントロールプレーンとデータベースを実行します。{{ site.konnect_saas }}は、データプレーンを実行するだけで済み、<a href="https://konghq.com/products/kong-konnect/register?utm_medium=referral&utm_source=docs&utm_campaign=gateway-konnect&utm_content=kubernetes-install">5分以内に開始できます</a>。

これらの手順で、コントロールプレーン（CP）とデータプレーン（DP）を別々にデプロイするように {{ site.base_gateway }} を構成します。これは、推奨のインストール方法です。

前提条件
----

* [`Helm 3`](https://helm.sh/)
* [`kubectl`](https://kubernetes.io/docs/tasks/tools/) v1\.19以降
* 実行中のPostgreSQLデータベース

### Helmのセットアップ

Kongは、 {{ site.base_gateway }}をデプロイするためのHelmチャートを提供します。`charts.konghq.com`リポジトリを追加し、`helm repo update`を実行して、最新バージョンのチャートを確保します。

```bash
helm repo add kong https://charts.konghq.com
helm repo update
```

シークレット
------

### {{ site.ee_product_name }}ライセンス

まず、`kong`の名前空間を作成します。

```bash
kubectl create namespace kong
```

次に、{{site.ee_product_name}} ライセンスシークレットを作成します。

{% navtabs %}
{% navtab Kong Gateway Enterprise Free Mode%}

    kubectl create secret generic kong-enterprise-license --from-literal=license="'{}'" -n kong

{% endnavtab %}
{% navtab Kong Gateway Enterprise Licensed Mode%}
> 
> このコマンドを実行する前に、`license.json` ファイルが含まれているディレクトリにいることを確認してください。

    kubectl create secret generic kong-enterprise-license --from-file=license=license.json -n kong

{% endnavtab %}
{% endnavtabs %}

### 証明書のクラスタリング


{{ site.base_gateway }}は、ハイブリッドモードで実行している場合、mTLSを使用してコントロールプレーン/データプレーンの通信を保護します。

1. OpenSSLを使用してTLS証明書を生成します。

   ```bash
   openssl req -new -x509 -nodes -newkey ec:<(openssl ecparam -name secp384r1) -keyout ./tls.key -out ./tls.crt -days 1095 -subj "/CN=kong_clustering"
   ```

2. 証明書を含む Kubernetes シークレットを作成します。

       kubectl create secret tls kong-cluster-cert --cert=./tls.crt --key=./tls.key -n kong

インストール
------

### コントロールプレーン（CP）

コントロールプレーンにはすべての{{ site.base_gateway }}構成が含まれています。設定はPostgreSQLデータベースに保存されます。

1. `values-cp.yaml`ファイルを作成します。

{% capture values_file %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}
{% include md/gateway/k8s-install-proxy-cp.md release=page.release package='ee' %}
{% endnavtab %}
{% navtab Kong Gateway (OSS) %}
{% include md/gateway/k8s-install-proxy-cp.md release=page.release package='oss' %}
{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ values_file | indent }}

1. *（オプション）* テスト目的でクラスタ内にPostgresデータベースをデプロイする場合は、`values-cp.yaml`の末尾に次のコードを追加します。

   ```yaml
   # This is for testing purposes only
   # DO NOT DO THIS IN PRODUCTION
   # Your cluster needs a way to create PersistentVolumeClaims
   # if this option is enabled
   postgresql:
     enabled: true
     auth:
       password: demo123
   ```

2. `values-cp.yaml`のデータベース接続値を更新します。

   * `env.pg_database`: 使用するデータベース名
   * `env.pg_user`：データベースのユーザー名
   * `env.pg_password`: データベースのパスワード
   * `env.pg_host`: Postgresデータベースのホスト名
   * `env.pg_ssl`：SSLを使用してデータベースに接続します

3. `values-cp.yaml`でKong Managerスーパー管理者パスワードを設定します。

   * `env.password`：Kong Managerのスーパー管理者パスワード

4. `helm install`を実行してリリースを作成します。

   ```bash
   helm install kong-cp kong/kong -n kong --values ./values-cp.yaml
   ```

5. `kubectl get pods -n kong`を実行します。コントロールプレーン（CP）が期待どおりに実行されるようにします。

       NAME                                 READY   STATUS
       kong-cp-kong-7bb77dfdf9-x28xf        1/1     Running

### データプレーン（DP）

{{ site.base_gateway }} データプレーンは、受信トラフィックの処理をします。クラスタリングエンドポイントを使用して、コントロールプレーンからルーティング構成を受け取ります。

1. `values-dp.yaml`ファイルを作成します。

{% capture values_file %}
{% navtabs_ee codeblock %}
{% navtab Kong Gateway %}
{% include md/gateway/k8s-install-proxy-dp.md release=page.release package='ee' %}
{% endnavtab %}
{% navtab Kong Gateway (OSS) %}
{% include md/gateway/k8s-install-proxy-dp.md release=page.release package='oss' %}
{% endnavtab %}
{% endnavtabs_ee %}
{% endcapture %}


{{ values_file | indent }}

1. `helm install`を実行してリリースを作成します。

   ```bash
   helm install kong-dp kong/kong -n kong --values ./values-dp.yaml
   ```

2. `kubectl get pods -n kong`を実行します。データプレーンが期待どおりに実行されることを確実にします。

       NAME                                 READY   STATUS
       kong-dp-kong-5dbcd9f6b9-f2w49        1/1     Running

テスト
---


{{ site.base_gateway }}現在実行中です。テストトラフィックを送信するには、次の操作を試してください。

1. `kong-dp`サービスの`LoadBalancer`アドレスを取得し、`PROXY_IP`環境変数に保存します

   ```bash
   PROXY_IP=$(kubectl get service --namespace kong kong-dp-kong-proxy -o jsonpath='{range .status.loadBalancer.ingress[0]}{@.ip}{@.hostname}{end}')
   ```

2. `$PROXY_IP` に HTTP リクエストを送信します。これには、{{ site.base_gateway }} によって提供される `HTTP 404` が返されます

   ```bash
   curl $PROXY_IP/mock/anything
   ```

3. 別のターミナルで、 `kubectl port-forward`を実行してポート転送を設定し、管理 API にアクセスします。

   ```bash
   kubectl port-forward -n kong service/kong-cp-kong-admin 8001
   ```

4. 模擬サービスとルートの作成

   ```bash
   curl localhost:8001/services -d name=mock  -d url="http://httpbin.org"
   curl localhost:8001/services/mock/routes -d "paths=/mock"
   ```

5. `$PROXY_IP`に対して HTTP 要求を再度行います。今回は{{ site.base_gateway }}リクエストをhttpbinにルーティングします。

   ```bash
   curl $PROXY_IP/mock/anything
   ```

