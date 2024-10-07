---
title: "Kong Gateway (OSS) から Kong Gateway Enterprise への移行"
toc: true
---
`kong migrations`コマンドを実行して、 {{site.ce_product_name}}インストールから{{site.ee_product_name}}に移行します。

{{site.ee_product_name}}バージョンのうち、移行先として選択できるのは同じ{{site.ce_product_name}}バージョンをサポートするものに限定されます。

前提条件
----

{:.warning}
> 
> **警告：** この操作は元に戻すことができないので、
   {{site.ce_product_name}}から
> {{site.ee_product_name}}に移行する前に、プロダクションデータをバックアップすることを強くお勧めします。

{{page.release}} より前のバージョンの {{site.ce_product_name}} を実行している場合は、[{{site.ce_product_name}} {{page.release}} にアップグレード](/gateway/{{page.release}}/upgrade/)してから、
{{site.ce_product_name}} を {{site.ee_product_name}} {{page.release}} に移行してください。

移行手順
----

次の手順では、移行プロセスについて説明します。

1. {{site.ee_product_name}} 
{{page.release}}パッケージを[ダウンロード](/gateway/{{page.release}}/install/)し、
{{site.ce_product_name}}ノードと同じデータストアを指すように構成します。移行コマンドは、保留中の移行についてデータストアが最新出あることを期待します。

   ```shell
   kong migrations up [-c configuration_file]
   kong migrations finish [-c configuration_file]
   ```

2. すべてのエンティティが
   {{site.ee_product_name}} ノードで利用可能になっていることを確認してください。

