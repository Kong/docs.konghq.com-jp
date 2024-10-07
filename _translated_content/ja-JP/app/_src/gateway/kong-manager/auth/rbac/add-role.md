---
title: "ロールと権限を追加する"
badge: "enterprise"
---
ロールを使用すると、同じ権限セットを論理的にグループ化して管理者に適用することが容易になります。権限は、個々のアクションやエンドポイントに至るまで、詳細にカスタマイズできます。


{{site.base_gateway}}には、標準的なユースケースのデフォルトの役割が含まれています。追加のスーパー管理者を招待し、エンドポイントのみを`read`できる管理者を招待します。

このガイドでは、ユニークなユースケースのKong
Manager にカスタムロールを作成する方法について説明します。別の方法として、スーパー管理者がロールを
Admin APIで作成したい場合、[`/rbac/roles`](/gateway/api/admin-ee/latest/#/rbac/post-rbac-roles)を使用して
実行することが可能です。新しいロールに権限を追加するには、
エンドポイントには[`/rbac/roles/{name_or_id}/endpoints`](/gateway/api/admin-ee/latest/#/rbac/post-rbac-roles-name_or_id-endpoints)
、または特定のエンティティに対して
[`/rbac/roles/{name_or_id}/entities`](/gateway/api/admin-ee/latest/#/rbac/post-rbac-roles-name_or_id-entities)
を使用します。

前提条件
----

* 認証とRBACが[有効](/gateway/{{page.release}}/kong-manager/auth/rbac/enable/)になっている
* [スーパー管理者権限](/gateway/{{page.release}}/kong-manager/auth/super-admin/)があるか、`/admins`と`/rbac`の読み取りと書き込みアクセス権を持つユーザーがいる

ロールと権限を追加
---------

1. **管理者** ページから、 **ロールを追加** ボタンをクリックします。

2. **ロールの追加（Add Role）** フォームで、付与したい **権限（Permissions）** に応じて **ロール（Role）** に名前を付けます。

   権限の理由を説明する簡単なコメントやロールの概要を含めておくと、後で参照する際に役立つ場合があります。
3. **\[権限を追加\]** ボタンをクリックして、フォームに記入してください。
   適切なチェックボックスをオンにしてエンドポイントの権限を追加します。

4. **ロールに権限を追加** をクリックして、フォームに列挙される権限を表示します。

5. 特定のエンドポイントへのアクセスを禁止するには、再び **権限を追加する（Add Permission）** をクリックし、 **無効化** のチェックボックスを使用します。

6. フォームを送信して、管理者ページに新しいロールを表示します
   。

