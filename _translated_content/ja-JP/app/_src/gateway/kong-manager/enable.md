---
title: "Kong Managerを有効にする"
---
{{site.base_gateway}}をデータベース（従来モード
またはハイブリッドモード）を使用して実行している場合は、Kong Managerで{{site.base_gateway}}のグラフィカルユーザーインターフェース（GUI）を
有効にできます。

{% navtabs %}
{% navtab Docker %}

1. （[`kong.conf`](/gateway/{{page.release}}/production/kong-conf/)）構成ファイルの [`KONG_ADMIN_GUI_PATH`](/gateway/{{page.release}}/reference/configuration/#admin_gui_path) および [`KONG_ADMIN_GUI_URL`](/gateway/{{page.release}}/reference/configuration/#admin_gui_url) プロパティにシステムの DNS または IP アドレスを設定し、次に、設定を有効にするために {{site.base_gateway}} を再起動します。以下に例を示します。

   ```bash
   docker exec -i <kong_container_id id="sl-md0000000"> /bin/sh -c "export KONG_ADMIN_GUI_PATH='/'; export KONG_ADMIN_GUI_URL='http://localhost:8002/manager'; kong reload; exit"
   ```

   `KONG_CONTAINER_ID`をDockerコンテナのIDに置き換えます。
2. `KONG_ADMIN_GUI_PATH`で指定したパス、またはデフォルトのURL`http://localhost:8002/workspaces`にあるポート`8002`でKong Managerにアクセスします。

{% endnavtab %}
{% navtab Linux (kong.conf) %}

1. `kong.conf`構成ファイルの[`admin_gui_url`](/gateway/{{page.release}}/reference/configuration/#admin_gui_url)プロパティをシステムのDNS、IPアドレスに更新します。例：

       admin_gui_path = /manager
       admin_gui_url = http://localhost:8002/manager

   この設定は、オペレーティングシステム（OS）ホストに到達できるネットワーク パスに解決する必要があります。
2. 次のコマンドを使用して{{site.base_gateway}}を再起動し、設定を有効にします。

   ```bash
   kong restart -c {PATH_TO_KONG.CONF_FILE}
   ```

3. `admin_gui_path`で指定したパスのポート`8002`でKong Managerにアクセスします。

{% endnavtab %}
{% endnavtabs %}

次の手順
----

* [{{site.base_gateway}}の管理を開始する](/gateway/{{page.release}}/kong-manager/get-started/services-and-routes/)
* [Kong Managerの認証を設定する](/gateway/{{page.release}}/kong-manager/auth/)
* [{{site.base_gateway}}リソースへのロールベースのアクセス制御を設定する](/gateway/{{page.release}}/kong-manager/auth/rbac/)

