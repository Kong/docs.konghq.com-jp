---
title: "RBACユーザーの作成"
badge: "enterprise"
---
管理者とRBACユーザー
------------

|          | Admin API | Kong Manager |
|----------|-----------|--------------|
| 管理者      | ✔️        | ✔️           |
| RBACユーザー | ✔️        | X            |

RBACユーザーは{{site.base_gateway}}Admin APIにアクセスできます。ロールに割り当てられた権限は、さまざまなAdmin APIオブジェクトで実行できるアクションのタイプを定義します。

[管理者は](/gateway/{{page.release}}/kong-manager/auth/)、RBACユーザーと同様に、 {{site.base_gateway}}Admin APIにアクセスできます。管理者はKong Managerにログインすることもできます。RBACユーザーと同様に、管理者のロールによって実行できるアクションの種類が決まりますが、Kong Managerのインターフェイスとビジュアライゼーションも利用できます。

たとえば、自動化プロセスの一部としてマシンに対して{{site.base_gateway}}の *サービス アカウント* を作成する場合は、RBACユーザーが適切です。

{{site.base_gateway}} *の個人アカウント* を作成する場合は、Kong Managerにもアクセスできるため、管理者が望ましい場合があります。

前提条件
----

* 認証とRBACが[有効](/gateway/{{page.release}}/kong-manager/auth/rbac/enable/)になっている
* [スーパー管理者権限](/gateway/{{page.release}}/kong-manager/auth/super-admin/)があるか、または `/admins` および `/rbac` への読み取りと書き込みアクセス権を持つユーザーがいます。

Kong ManagerでRBACユーザーを追加する
--------------------------

1. ダッシュボードで **\[チーム\]** タブをクリックします。

2. **\[チーム\]** ページで、 **\[RBACユーザー\]** タブをクリックします。

3. ドロップダウンメニューを使用して、新しいユーザーがアクセスできる **ワークスペース** を選択します。

   {:.note}
   > 
   > **注：** **デフォルトのワークスペース** はグローバルです。つまり、デフォルトへのアクセス権を持つRBACユーザーは、他のすべてのワークスペースのエンティティにアクセスできます。このワークスペースの割当は、管理アカウントや監査アカウントには役立ちますが、特定のチームのメンバーには役立ちません。
4. **新規ユーザー追加** ボタンをクリックして、登録フォームを開きます。

5. **新規ユーザー追加** 登録フォームに入力してください。

   * RBAC ユーザーの名前は、2 人のユーザーが異なるワークスペースに存在する場合でもグローバルに一意である必要があり、管理者アカウントと同じ名前にすることはできません。これらの命名規則は、OIDC、LDAP、またはその他の外部の ID およびアクセス管理方法を使用する場合に重要です。
   * RBAC ユーザーアカウントはデフォルトで有効になっています。RBACユーザーアカウントを無効な状態で開始し、後で有効にする場合は、 **有効** ボックスのチェックを外します。

6. **ロールの追加/編集** ボタンをクリックします。新しいRBACユーザーに必要なロール（複数可）を選択します。

   * RBACユーザーにロールが割り当てられていない場合、どのオブジェクトにもアクセスする権限がありません。
   * RBACユーザーのロールの割り当ては、必要に応じて後で変更できます。
   * ロールは、手順 3 で選択した 1 つのワークスペースにのみ属することができます。
   * RBACユーザーに複数のワークスペース内のオブジェクトへのアクセス権を付与するには、手順3を参照してください。

7. **Create User** をクリックすると、ユーザー登録が完了します。

   ページは自動的に **チーム（Teams）** ページにリダイレクトされ、新しいユーザーがリストされます。
