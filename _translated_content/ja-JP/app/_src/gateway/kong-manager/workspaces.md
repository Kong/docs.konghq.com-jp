---
title: "Kong Managerでのワークスペースの設定"
badge: "enterprise"
---
ワークスペースを使用すると、組織は、
同じKongクラスタを共有する管理者チームが、それぞれのグループのエンティティとのみやり取りできるよう
トラフィックをセグメント化できます。ワークスペース内では、
特定のチームに管理者を招待したり、
管理者が利用できるアクションとエンティティの種類をさらに制限するロールと権限で
RBACを適用したりできます。

前提条件
----

* 認証とRBACが[有効](/gateway/{{page.release}}/kong-manager/auth/rbac/)である
* [スーパー管理者](/gateway/{{page.release}}/kong-manager/auth/super-admin/)として、または`/admins`と`/rbac`の読み取りと書き込みのアクセス権を持つユーザーとしてKong Managerにログインしている

デフォルトのワークスペース
-------------

最初のSuper Adminがログインすると、 **default** という名前のワークスペースから
始まります。ここから、デフォルトまたは
その他のワークスペースを管理するために管理者を招待します。

Kong Managerでのワークスペース間の移動
-------------------------

{% if_version lte:3.4.x %}
**概要** ページからワークスペース間を移動するには、
**Vitals** チャートの下に表示される任意のワークスペースをクリックします。{% endif_version %}
{% if_version gte:3.5.x %}
**概要** ページからワークスペース間を移動するには、
ワークスペースのリストに表示される任意のワークスペースをクリックします。
{% endif_version %}

ワークスペースのリストは好みに応じて、カードまたはテーブルとして
レンダリングされます。

ワークスペースの作成
----------

このガイドでは、Kong マネージャー内でワークスペースを作成する方法について説明します。Admin API [`/workspaces/` ルート](/gateway/api/admin-ee/latest/#/Workspaces/create-workspace)を使用してワークスペースを作成することもできます。

1. **スーパー管理者** としてログインします。 **ワークスペース** ページで、 **新しいワークスペース** ボタンをクリックすると、 **ワークスペースの作成** フォームが表示されます。
   新しいワークスペースに名前を付け、色/アイコンを選択します。

   各ワークスペース名は一意である必要があります。
   大文字と小文字を区別しません。例えば、
   ワークスペース "Payments" と "payments" は
   同一のようにに見える2つの異なるワークスペースを作成します。

   ワークスペースの名前をAdmin APIのこれらの主要なAPI名（パス）と同じにしないでください。

       • Admins
       • Certificates
       • Consumers
       • Plugins
       • Portal
       • Routes
       • Services
       • SNIs
       • Upstreams
       • Vitals

2. **新しいワークスペースの作成\]** ボタンをクリックします。アプリケーションを作成すると、
   新しいワークスペースのダッシュボードに移動します。

ワークスペースの編集
----------

1. 編集するワークスペースで、 **ダッシュボード** ページに移動します。

2. **設定** ボタンをクリックします。このボタンをクリックすると、 **\[ワークスペースの編集\]** ページが表示されます。

3. このページで、ワークスペースのアバターとアバターの背景色を編集できます。

4. **ワークスペースの更新** をクリックして保存します。

{% if_version lte:3.4.x %}

ワークスペースの削除
----------

{:.important}
> 
> **注意** ：以下の手順では、 **ワークスペースのすべてのエンティティ** も削除されます。続行する前に、これが本当に実行したい操作であるかどうかを確認してください。

次のいずれかの方法を選択します。

{% navtabs %}
{% navtab Kong Manager %}

1. **Dev Portal** > **Editor** ですべてのファイルを手動で削除します。現時点ではフォルダを削除できませんが、フォルダのすべてのファイルを削除するとフォルダが削除されます。
2. Dev Portalをオフにします。Dev Portal **設定** > **詳細設定** > **オフ** の順に移動します。
3. ワークスペースからすべてのロールを削除します。
   1. **チーム（Teams）** タブに移動します。
   2. **ロール** タブに移動します。
   3. 削除するワークスペースで **表示** をクリックします。
   4. 各ロールのエントリに移動し、 **編集** をクリックします。
   5. エントリーの詳細ページで、 **ロールを削除** をクリックします。確認モーダルが表示されます。もう一度、 **ロールを削除** をクリックします。

4. 削除するワークスペースで、 **\[ダッシュボード\]** ページに移動します。
5. **設定** ボタンをクリックして、 **ワークスペースの編集** ページを開きます。
6. **削除** をクリックします。

{:.note}
> 
> Kong Manager GUI を使用してワークスペースを削除するには、まずワークスペースを空にする必要があります。ワークスペースにエンティティが含まれている場合は、管理 API の指示に従って、ワークスペースと関連するエンティティの両方を削除する必要があります。

{% endnavtab %}
{% navtab Admin API %}

{% if_version lte:3.3.x %}

1. ワークスペースに関連付けられているすべての Dev Portal ファイルを削除します。

   ```bash
   curl -i -X DELETE http://localhost:8001/{WORKSPACE_NAME}/files
   ```

2. ワークスペースの Dev Portal をオフにします。

   ```bash
   curl -X PATCH http://localhost:8001/workspaces/{WORKSPACE_NAME}  \
    --data "config.portal=false"
   ```

3. ワークスペースから[各ロール](/gateway/{{page.release}}/admin-api/rbac/reference/#delete-a-role)を削除します。

   ```bash
   curl -i -X DELETE http://localhost:8001/{WORKSPACE_NAME}/rbac/roles/{ROLE_NAME|ROLE_ID}
   ```

4. Kong Admin APIに`DELETE`リクエストを送信します。

   ```sh
   curl -i -X DELETE http://localhost/workspaces/{WORKSPACE_NAME|WORKSPACE_ID}
   ```

   ワークスペースにデータがある場合、削除は失敗します。

   成功した場合は、次の応答が表示されます。

       HTTP 204 No Content

{% endif_version %}

{% if_version gte:3.4.x %}
ワークスペースと、[ワークスペースに関連付けられているすべてのエンティティ](/gateway/api/admin-ee/latest/#/Workspaces/delete-workspace)を削除します。

```bash
curl -i -X DELETE http://localhost:8001/{WORKSPACE_NAME}?cascade=true
```

成功した場合は、次の応答が表示されます。

    HTTP 204 No Content

{% endif_version %}

{% endnavtab %}
{% navtab Portal CLI %}

1. ワークスペースに関連付けられているすべての Dev Portal ファイルを削除します。

   ```sh
   portal wipe WORKSPACE_NAME
   ```

2. ワークスペースの Dev Portal をオフにします。

   ```sh
   portal disable WORKSPACE_NAME
   ```

3. ワークスペースから各ロールを削除します。この手順は
   ポータル CLIを使用して完了することができないため、 *Kong Manager* または *Admin API* タブに切り替えて
   手順3を完了します。

4. ワークスペースを削除します。この手順は
   ポータル CLIを使用して完了することができないため、 *Kong Manager* または *Admin API* タブに切り替えて
   手順4を完了します。
   {% endnavtab %}
   {% endnavtabs %}
   {% endif_version %}

{% if_version gte:3.5.x %}

ワークスペースの削除
----------

{:.important}
> 
> **注意** ：以下の手順では、 **ワークスペースのすべてのエンティティ** も削除されます。続行する前に、これが本当に実行したい操作であるかどうかを確認してください。

次のいずれかの方法を選択します。

{% navtabs %}
{% navtab Kong Manager %}
Kong Manager を使用して、次の操作を実行します。

1. **ワークスペース** に移動し、削除するワークスペースを選択します。

2. **設定（Settings）** をクリックし、 **削除（Delete）** をクリックします。

3. **\[ワークスペースの削除\]** ダイアログで、ワークスペースの名前を入力し、 **\[確認：関連付けられているすべてのリソースを削除\]** を選択して、 **\[削除\]** をクリックします。

これにより、ワークスペース自体だけでなく、ワークスペースに関連付けられているすべてのエンティティ（チーム、ロール、サービス、ルートなど）が自動的に削除されます。

{% endnavtab %}
{% navtab Admin API %}
ワークスペースと、[ワークスペースに関連付けられているすべてのエンティティ](/gateway/api/admin-ee/latest/#/Workspaces/delete-workspace)を削除します。

```bash
curl -i -X DELETE http://localhost:8001/{WORKSPACE_NAME}?cascade=true
```

成功した場合は、次の応答が表示されます。

    HTTP 204 No Content

{% endnavtab %}
{% endnavtabs %}
{% endif_version %}

ワークスペースへのアクセス
-------------

あるロールに、ワークスペース内のエンドポイント全体にアクセスする権限がない場合、そのロールに割り当てられた管理者は関連するナビゲーションリンクを参照することができません。

アクセスを設定するには以下の操作を行います。

1. Kong Managerを開きます。
2. **セキュリティセクション** の **管理者** リンクをクリックします。

サイドバーが折りたたまれている場合は、
**セキュリティバッジアイコン** 上にカーソルを合わせて
**管理者** をクリックします。

管理者ページには、現在の管理者とロールの一覧が表示されます。新しいワークスペースに固有の4つのデフォルトロールが表示されており、このページからワークスペースに固有の新しいロールを割り当てることができます。

管理者とロールの詳細については、 [Kong ManagerのRBAC](/gateway/{{page.release}}/kong-manager/auth/rbac/)を参照してください。

