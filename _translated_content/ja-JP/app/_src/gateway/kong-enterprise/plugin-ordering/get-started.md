---
title: "動的プラグインの注文を開始する"
badge: "enterprise"
content_type: "how-to"
---
[動的プラグインの順序付け](/gateway/{{page.release}}/kong-enterprise/plugin-ordering/)の一般的なユースケースをいくつかご紹介します。

認証前の流量制限
--------

Kongが認証をリクエストする *前に* 、サービスとルートに対するリクエストの量を制限するとしましょう。
この依存関係は、トークン`before`で記述できます。

次の例は、認証メソッドが[Key Authentication](/hub/kong-inc/key-auth/)プラグイン
である[Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)プラグインを
使用しています。

{% navtabs %}
{% navtab Admin API %}

ポート`8001`でAdmin APIを呼出し、`rate-limiting`プラグインを有効にして、`key-auth`の前に実行するように構成します。

```sh
curl -i -X POST http://localhost:8001/plugins \
  --data name=rate-limiting \
  --data config.minute=5 \
  --data config.policy=local \
  --data config.limit_by=ip \
  --data ordering.before.access=key-auth
```

{% endnavtab %}
{% navtab Kubernetes %}

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
  name: limit-before-key-auth
  labels:
    global: "true"
  annotations:
    kubernetes.io/ingress.class: "kong"
config:
  minute: 5
  policy: local
  limit_by: ip
plugin: rate-limiting
ordering:
  before:
    access:
    - key-auth
```

{% endnavtab %}
{% navtab decK (YAML) %}

1. 新しい `plugins` セクションを `kong.yaml` ファイルの一番下に追加します。
   `rate-limiting`を有効化し、プラグインを`key-auth`の前に実行するように設定します。

   ```yaml
   plugins:
   - name: rate-limiting
     config:
       minute: 5
       policy: local
       limit_by: ip
     ordering:
       before:
         access:
           - key-auth
   ```

   ファイルは以下のようになります。

   ```yaml
   _format_version: "3.0"
   services:
   - host: httpbin.org
     name: example_service
     port: 80
     protocol: http
     routes:
     - name: mocking
       paths:
       - /mock
       strip_path: true
   plugins:
   - name: rate-limiting
     config:
       minute: 5
       policy: local
       limit_by: ip
     ordering:
       before:
         access:
           - key-auth
   ```

   このプラグインはグローバルに適用されるため、流量制限はワークスペース内のすべてのサービスとルートを含むすべてのリクエストに適用されます。

   既存のサービス、ルート、またはコンシューマにこのプラグインセクションを貼り付けると、流量制限はその特定のエンティティのみに適用されます。

   {:.note}
   > 
   > **注** ：デフォルトでは、プラグインの`enabled`は`true`に設定されています。`enabled: false`を設定することでいつでもプラグインを無効にすることができます。
2. 設定を同期します：

   ```bash
   deck gateway sync kong.yaml
   ```

{% endnavtab %}
{% navtab Kong Manager UI %}

{:.note}
> 
> **注:** プラグインの動的順序付けに対するKong Managerのサポートは、{{site.base_gateway}} 3\.1\.x以降で利用できます。

1. Kong Managerで、 **デフォルト** のワークスペースを開きます。

2. メニューから **プラグイン** を開き、 **プラグインのインストール** をクリックします。

3. **Rate Limiting** プラグインを見つけて、 **有効にする（Enable）** をクリックします。

4. プラグインを **グローバル** として適用します。これにより、流量制限はワークスペース内のすべてのサービスとルートを含むすべてのリクエストに適用されます。

5. 次のフィールドのみに、次のパラメータを入力します。

   1. config.minute：`5`
   2. config.policy：`local`
   3. config.limit\_by:`ip`

   上記のフィールド以外にも、デフォルト値が設定されているフィールドが存在する場合があります。この例では、残りのフィールドはそのままにしておきます。
6. **インストール** をクリックします。

7. **Rate Limiting** プラグインページで、 **順序付け** タブをクリックします。

8. **注文の追加** をクリックします。

9. **アクセス前** で、 **プラグインの追加** をクリックしてください。

10. **プラグイン** ドロップダウンメニューから **Key Auth** を選択します。

11. **更新** をクリックします。

流量制限プラグインは、{{site.base_gateway}}リクエスト認証 *前* にデフォルトのワークスペースですべてのサービスとルートに対してリクエスト量を制限するようになりました。

{% endnavtab %}
{% endnavtabs %}

リクエスト変換後の認証
-----------

次の例は、[認証前に流量制限](#rate-limiting-before-authentication)を行う場合に似ています。

たとえば、最初にリクエストの変換を行い、変換した *後に* 認証をリクエストしたい場合があります。この依存関係は、トークン `after` で記述できます。

[Request Transformer](/hub/kong-inc/request-transformer/)
プラグインの順序を変更する代わりに、認証プラグイン
の順序を変更できます（この例では、[Basic Authentication](/hub/kong-inc/basic-auth/) を使用）。

{% navtabs %}
{% navtab Admin API %}

ポート`8001`でAdmin APIを呼び出し、
`basic-auth`プラグインを`request-transformer`の後に実行するように構成して有効にします。

```sh
curl -i -X POST http://localhost:8001/plugins \
  --data name=basic-auth \
  --data ordering.after.access=request-transformer
```

{% endnavtab %}
{% navtab Kubernetes %}

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
  name: auth-after-transform
  labels:
    global: "true"
  annotations:
    kubernetes.io/ingress.class: "kong"
plugin: basic-auth
ordering:
  after:
    access:
    - request-transformer
```

{% endnavtab %}
{% navtab decK (YAML) %}

1. 新しい`plugins`セクションを`kong.yaml`ファイルの一番下に追加します。
   `basic-auth`を有効にし、プラグインを`request-transformer`の後に実行するように設定します。

   ```yaml
   plugins:
   - name: basic-auth
     config: {}
     ordering:
       after:
         access:
           - request-transformer
   ```

   ファイルは以下のようになります。

   ```yaml
   _format_version: "3.0"
   services:
   - host: httpbin.org
     name: example_service
     port: 80
     protocol: http
     routes:
     - name: mocking
       paths:
       - /mock
       strip_path: true
   plugins:
   - name: basic-auth
     config: {}
     ordering:
       after:
         access:
           - request-transformer
   ```

   {:.note}
   > 
   > **注** ：デフォルトでは、プラグインの`enabled`は`true`に設定されています。`enabled: false`を設定することでいつでもプラグインを無効にすることができます。
2. 設定を同期します：

   ```bash
   deck gateway sync kong.yaml
   ```

{% endnavtab %}
{% navtab Kong Manager UI %}

{:.note}
> 
> **注:** プラグインの動的順序付けに対するKong Managerのサポートは、{{site.base_gateway}} 3\.1\.x以降で利用できます。

1. Kong Managerで、 **デフォルト** のワークスペースを開きます。
2. メニューから **プラグイン** を開き、 **プラグインのインストール** をクリックします。
3. **Basic Authentication** プラグインを見つけて、 **有効にする** をクリックします。
4. プラグインを **グローバル** として適用します。これにより、流量制限はワークスペース内のすべてのサービスとルートを含むすべてのリクエストに適用されます。
5. **インストール** をクリックします。
6. **ベーシック認証** プラグインページから、 **順序付け** タブをクリックします。
7. **注文の追加** をクリックします。
8. **アクセス後** に、 **プラグインの追加** をクリックします。
9. **プラグイン1** ドロップダウンメニューから、 **Request Transformer** を選択します。
10. **更新** をクリックします。

ベーシック認証プラグインは、リクエストが変換された後に認証をリクエストするようになりました。
{% endnavtab %}
{% endnavtabs %}

