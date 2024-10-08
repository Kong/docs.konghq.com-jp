---
title: "開発者の管理"
badge: "enterprise"
---
開発者ステータス
--------

ステータスは、開発者の状態と、Dev PortalおよびAPIへのアクセス権を表します。

* **承認済み** 
  * Dev Portalにアクセス可能な開発者。承認済み開発者は、認証情報を作成し、それらの認証情報を許可する **すべての** APIにアクセスできます。 

* **リクエスト** 
  * アクセスをリクエスト済みで、承認待ちの開発者。

* **拒否済み** 
  * Kongの管理者によってリクエストが拒否された開発者。

* **Revoked** 
  * 以前は Dev Portal のアクセス権を有していたが、アクセス権を取り消された開発者。

![開発者の管理](https://konghq.com/wp-content/uploads/2018/05/gui-developer-tabs.png)

開発者を承認
------

Dev Portalへのアクセスをリクエストした開発者は、
**リクエストされたアクセスタブ** 下に表示されます。このタブから、表の行のアクションの開発者を *受け入れる* か *拒否する* かを選択できます。アクションを選択すると、
対応するタブが更新されます。

承認済み開発者の表示
----------

現在承認されている開発者をすべて表示するには、 **承認済み** タブをクリックします。ここから、特定の開発者の *取り消し* または *削除* を行えます。さらに、このビューを使用すると、 **開発者にメールを送信** `mailto`リンクから、開発者にメールを送ることもできます。詳細については、[開発者へのメール送信](#emailing-developers)を参照してください。

取り消された開発者を表示
------------

現在無効となっている開発者をすべて表示するには、 **無効化済み（Revoked）** タブをクリックします。ここから、開発者を *再承認する* か *削除するか* を選択できます。

### 却下された開発者の表示

現在拒否されている開発者をすべて表示するには、 **拒否** タブをクリックします。拒否された開発者は、Dev Portalで登録フローを完了しましたが、 **アクセス要求** タブで拒否されました。このタブから開発者を *承認* または *削除* することができます。

開発者にメールを送信する
------------

### 開発者を登録に招待する

1人または複数の開発者を招待する方法は次のとおりです。

1. **開発者を招待** をクリックします。
2. ポップアップモーダルを使用して、複数のメールアドレスをカンマで区切って入力します。
3. すべてのメールが追加されたら、 **招待** をクリックします。デフォルトのメールクライアントで 事前に入力されたメッセージが作成され、Dev Portalの登録ページへの リンクが記載されます。

プライバシー保護のため、各開発者にはデフォルトで BCC が送信されます。メッセージを編集するか、そのまま送信するかを選択できます。

![開発者を招待](https://konghq.com/wp-content/uploads/2018/05/invite-developers.png)

開発者管理プロパティリファレンス
----------------

開発者管理プロパティに関する包括的なドキュメントについては、[デフォルトのDev Portal認証](/gateway/{{page.release}}/reference/configuration/#default-developer-portal-authentication-section)を参照してください。

