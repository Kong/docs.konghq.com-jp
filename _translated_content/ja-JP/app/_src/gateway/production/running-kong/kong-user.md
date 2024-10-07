---
title: "非ルートユーザーとしてのKongの実行"
---
GNU/Linuxシステムに{{site.base_gateway}}をインストールした後、Kongを`root`ではなく組み込みの`kong`ユーザーおよびグループとして実行するよう構成できます。これにより、Nginxマスタープロセスとワーカープロセスが組み込みの`kong`ユーザーおよびグループとして実行され、[`nginx_user`](/gateway/{{page.release}}/reference/configuration/#nginx_user)構成プロパティの設定はすべて上書きされます。または、Kongをカスタムの非ルートユーザーとして実行することも可能です。

{:.important}
> 
> **重要：** Nginxが特定のアクションを実行するには（例：特権ポート80でリッスンする）、Nginxマスタープロセスを`root`として実行する必要があります。<br><br>
> Kongを`kong`ユーザーおよびグループとして実行するとより高いセキュリティが得られますが、この決定を行う前にシステムとネットワーク管理評価を実施する必要があります。これを行わない場合、オペレーティングシステムで特権システムコール実行する権限が不十分なため、Kongノードが使用できなくなる可能性があります
> 。

前提条件
----


{{site.ee_product_name}}は、次のいずれかのLinuxディストリビューションにインストールされています。

* [Amazon Linux 1 または 2](/gateway/{{page.release}}/install/linux/amazon-linux/)
* [RHEL](/gateway/{{page.release}}/install/linux/rhel/)
* [Ubuntu](/gateway/{{page.release}}/install/linux/ubuntu/)

組み込みKongユーザーとして{{site.base_gateway}}を実行
-----------------------

{{site.base_gateway}} が `APT` や `YUM` などのパッケージ管理システムとともにインストールされると、デフォルトの `kong` ユーザーとデフォルトの `kong` グループが作成されます。パッケージによってインストールされたすべてのファイルは、`kong` ユーザーとグループによって所有されます。

1. 組み込みの`kong`ユーザーに切り替えます。

   ```sh
   su kong
   ```

2. Kongを起動します。

   ```sh
   kong start
   ```

{{site.base_gateway}}をカスタムの非ルートユーザーとして実行します。
----------------------------

または、Kongをカスタムの非ルートユーザーとして実行することも可能です。{{site.base_gateway}}パッケージによってインストールされたすべてのファイルは`kong`グループが所有しているため、そのグループに属するユーザーには`kong`ユーザーと同じ操作を実行する権限が付与される必要があります。

1. ユーザーを`kong`グループに追加します

   ```sh
   sudo usermod -aG kong your-user
   ```

2. Kongを起動します。

   ```sh
   kong start
   ```

