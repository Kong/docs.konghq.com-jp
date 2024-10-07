---
title: "Dev Portalを有効にする"
badge: "enterprise"
---
{{site.base_gateway}}をデータベース（従来モードまたはハイブリッドモード）で実行している場合は、Dev Portalを使用できます。

Dev Portal はワークスペースに紐付けされています。各ワークスペースには個別の Dev Portal インスタンスがあります。

Dev Portalを有効にすると、次のURLが公開されます。

* ワークスペースのDev PortalのURL。 たとえば、`default`ワークスペースの場合、URLは`http://localhost:8003/default`になります。
* Dev Portalファイルエンドポイント：`http://localhost:8001/files`
* 公開Dev Portalファイル API：`http://localhost:8004/files`

Dev Portal を有効にするには、まず[ライセンスをデプロイする](/gateway/{{page.release}}/licenses/deploy/)必要があります。

{% navtabs %}
{% navtab Docker %}

1. Docker コンテナで、ポータル URL を設定し、`KONG_PORTAL` を `on` に設定します。

   ```sh
   echo "KONG_PORTAL_GUI_HOST=localhost:8003 KONG_PORTAL=on kong reload exit" \
     | docker exec -i kong-container-name /bin/sh
   ```

   `kong-container-name` を {{site.base_gateway}} コンテナに置き換えます。

   {:.note}
   > 
   > `KONG_PORTAL_GUI_HOST`の`HOSTNAME`はプロトコルの前に配置しないでください。たとえば、 `http://`。
2. ワークスペースのDev Portalを有効にします。

   ```sh
   curl -i -X PATCH http://localhost:8001/workspaces/default \
     --data "config.portal=true"
   ```

3. `KONG_PORTAL_GUI_HOST`変数で指定されたURLを使用してワークスペースのDev Portalにアクセスします。

   ```sh
   http://localhost:8003/default
   ```

{% endnavtab %}
{% navtab Linux (kong.conf) %}

1. Dev Portal を有効にするには、Kong 構成ファイル（[`kong.conf`](/gateway/{{page.release}}/production/kong-conf/)）に次のプロパティを設定する必要があります。

       portal = on

   この値を有効にするには、{{site.base_gateway}} を再起動します。

       kong reload

2. 次のいずれかの方法を使用して、ワークスペースのDev Portalを有効にします。

{% capture enable-portal %}
{% navtabs %}
{% navtab Kong Manager %}

1. Kong Managerのワークスペースに移動します。
2. **Dev Portal** のメニューセクションで、 **概要** をクリックします。
3. ボタンをクリックして **開発者ポータルを有効** にします。

{% endnavtab %}
{% navtab Admin API %}

```bash
curl -i -X PATCH http://localhost:8001/workspaces/default \
  --data "config.portal=true"
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}


{{ enable-portal | indent | replace: " </code>", "</code>"}}

{% endnavtab %}
{% endnavtabs %}

