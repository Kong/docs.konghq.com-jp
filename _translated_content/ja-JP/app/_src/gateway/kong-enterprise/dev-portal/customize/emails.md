---
title: "ポータルメールのカスタマイズ"
badge: "enterprise"
---
Kong Dev Portal から送信される電子メールのメッセージと外観を管理できます。
編集可能なメールテンプレートは、ポータルレンダリング用のコンテンツファイルと同様のファイルとして読み込まれます。
メールファイルは、他のファイルと同じようにレンダリング用に、エディターまたはPortal CLIツールで管理できます。この機能はレガシーポータルモードではサポートされて **いません** 。

電子メール テンプレートが読み込まれていない場合、Kong は{{site.base_gateway}} 1\.3\.0\.0 と同じ電子メールにフォールバックします。
レガシーではないポータルを新しいワークスペースで有効にすると、デフォルトの編集可能なメールテンプレートが読み込まれます。
既存の非レガシー ポータルの場合、編集可能な電子メール テンプレートを手動で読み込む必要があります。

メール固有の値は、ポータルレイアウトやパーシャルのテンプレートと同様に機能するトークンでテンプレート化されます。すべてのトークンがすべてのメールでサポートされるわけではありません。

前提条件
----

* Kong Dev Portalは **レガシーモード** では実行されていません。
* Kong Dev Portal が有効かつ実行中
* [必要なメールがkongで有効になっている](/gateway/{{page.release}}/kong-enterprise/dev-portal/smtp/#portal_invite_email)
* CLI ツールを使用する場合、kong\-portal\-cli ツール 1\.1 以降がローカルにインストールされており、git がインストールされている

メールファイルを理解する
------------

ポータルテンプレートは、HTML、マークダウン、[`tokens`](#token-descriptions) を組み合わせて使用します。

次の例は`emails/approved-access.txt`テンプレートです。

{% raw %}

```yaml
---
layout: emails/email_base.html

subject: Dev Portal access approved {{portal.url}}
heading: Hello {{email.developer_name}}!
---
You have been approved to access {{portal.url}}.
<br id="sl-md0000000">
Please visit <a href="{{portal.url}}/login" id="sl-md0000000">{{portal.url}}/login</a> to login.
```

{% endraw %}

他のコンテンツファイルと同様に、これらのファイルは2つの`---`の間にヘッドマターがあります。レイアウトは`layout`属性で設定されます。これは、このメールがレンダリングするレイアウトファイルです。

* `subject`はメールの件名を設定します。
* `heading`はオプションの値で、デフォルトではh3としてレンダリングされます

メールの本文はHTMLコンテンツです。メールで許可されるトークンについては、以下の表を参照してください。この場合、ポータルURLにアクセスするために{% raw %}`{{portal.url}}`{% endraw %}が使用されます。

サポートされているメールとトークン
-----------------

{% raw %}
|パス	|サポートされているトークン	|必要なトークン	|説明|
|\-\-\- 	|\-\-\-	|\-\-\-	|\-\-\-	|
|emails/invite.txt	| `{{portal.gui_url}}` `{{email.developer_email}}` | `{{portal.gui_url}}` |マネージャーからポータルに招待された開発者へメールを送信	|
|\-\-\-	|\-\-\-	|\-\-\-	|\-\-\-	|
|emails/request\-access.txt	|`{{portal.gui_url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}` `{{email.admin_url}}`	|`{{portal.gui_url}}` `{{email.developer_email}}`	|開発者がポータルにサインアップしたときに、開発者を承認するために管理者に送信されるメール	|
|emails/approved\-access.txt	 |`{{portal.url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}`	|`{{portal.gui_url}}`	|アカウントが承認されたときに開発者に送信されるメール |
|emails/password\-reset.txt	|`{{portal.url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}` `{{email.token}}` `{{email.token_exp}}` `{{email.reset_url}}`	|`{{portal.url}}` `{{email.token}}`または`{{email.reset_url}}`|パスワードのリセットが要求されたときに開発者に送信されるメール（basic\-authのみ） |
|emails/password\-reset\-success.txt	|`{{portal.url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}`	|`{{portal.url}}`	|パスワードのリセットが成功したときに開発者に送信されるメール（basic\-auth のみ）|
|emails/account\-verification.txt	|`{{portal.url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}` `{{email.token}}` `{{email.verify_url}}` `{{email.invalidate_url}}`	|`{{portal.url}}` `{{email.token}}`または`{{email.verify_url}}`と`{{email.invalidate_url}}`の両方	| `portal_email_verification` がオンのときに開発者に送信されるメール（basic\-auth のみ）	|
|emails/account\-verification\-approved.txt	 |`{{portal.url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}`	|`{{portal.url}}`	|`portal_email_verification` がオンで、開発者がメールを確認し、開発者が管理者によって承認された場合/自動承認がオンの場合、開発者に送信されるメール（基本認証のみ）	 |
|emails/account\-verification\-pending.txt	|`{{portal.url}}` `{{email.developer_email}}` `{{email.developer_name}}` `{{email.developer_meta.*}}`	|`{{portal.url}}`	|`portal_email_verification` がオンで、開発者がメールを確認し、開発者がまだ管理者によって承認されていない場合に開発者に送信されるメール（basic\-authのみ）	|
{% endraw %}

トークンの説明
-------

{% raw %}
|トークン	|説明	|
|\-\-\- |\-\-\- |
| `{{portal.url}}`	|ワークスペースの開発ポータルURL	|
|\-\-\-	|\-\-\-	|
| `{{email.developer_email}}`	|開発者のメール	|
| `{{email.developer_name}}`	|開発者のフルネーム。この値はデフォルトで登録の一部として収集されます。 メタフィールドがfull\_nameを含まないように編集された場合、これはemailにフォールバックします	|
| `{{email.developer_meta.*}}`	|開発者のメタフィールド。これらの値は登録の一部として収集されます。これらは、登録前に構成する必要があります。値が存在しないかオプションで空白の場合は、空の文字列として表示されます。`developer_meta`構成でフィールドが指定されていない場合は、置換されずにそのまま表示されます。例：`{{email.developer_meta.preferred_name}}`|
| `{{email.admin_url}}`	|Kong Manager URL	|
| `{{email.reset_url}}` |パスワードをリセットするためのDev Portalの完全なURL（パスワードリセットのデフォルトパスを想定）	|
| `{{email.token_exp}}`	|パスワードリセットトークン/URLが有効である、電子メール送信からの時間を示す、人間が判読できる文字列。	|
| `{{email.verify_url}}`	|アカウント確認へのリンク（アカウント確認のデフォルトパスを想定）	|
| `{{email.invalidate_url}}`	|アカウント検証リクエストを無効にするリンク（アカウント検証のデフォルトパスを想定）	|
| `{{email.token}}`	| `{{portal.url}}`組み合わせて使用して、パスワードのリセットとアカウントの検証/無効化のためのURL文字列を手動で構築できます。パスワードリセットまたはアカウント認証用のカスタムパスが設定されていない限り、 **使用はお勧めしません** 。	|
{% endraw %}

メールテンプレートの編集
------------

Dev Portalがアクティブになると、デフォルトのメールテンプレートが Kong Dev Portalのファイルシステムに自動的に読み込まれます。これらのテンプレートは、 **ポータルエディタ** または **ポータルCLI** ツールを使用して、Kong Managerで編集できるようになりました。 **注：** 1\.3\.0\.1以前のバージョンの{{site.base_gateway}}で開始されたDev Portalを使用している場合は、メールテンプレートをファイルシステムに手動で読み込む必要があります。[既存のDev Portalにメールテンプレートを読み込む](#loading-email-templates-on-existing-dev-portals)の手順に従ってください。

### ポータルエディタを介した編集

メールテンプレートは、Kong Dev Portalファイルシステム内の他のファイルとともに、ポータルエディタで編集できるようになりました。これらのファイルは、次の方法で表示や編集できます。

1. Kong Managerにログインし、編集が必要なDev Portalがあるワークスペースに移動します。
2. **Dev Portal** のサイドバーから **エディタ** を選択します。

メールテンプレートは、ポータルエディターのサイドバーの **メール** セクションにあります。

### Portal CLI ツールを使った編集

1. \[https://github.com/Kong/kong\-portal\-templates\] の master ブランチをクローンし、そのディレクトリに移動します。
2. 保持したいカスタマイゼーションや権限の変更がある場合は、`portal fetch <workspacename id="sl-md0000000">`を実行します。これにより修正がローカルでプルされます。
3. 変更を加えた後、 `portal deploy <workspacename id="sl-md0000000">`を実行してすべてのファイルをデプロイします。

メールの外観の編集
---------

メールの外観を編集するために、メールが使用するレイアウトファイルを変更できます。
デフォルトでは、メールは `emails/email_base.html` レイアウトを使用するように設定されています。このファイルがテーマ内に存在しない場合（デフォルトのテーマは`base`）、ハードコードされたフォールバックが使用されます。

このレイアウトファイルのレイアウトは、ポータルエディタで編集することも、Portal CLIツールを使用してローカルで編集することもできます。メール本文を編集する手順に従い、レイアウトファイルを変更します。

メールに応じて異なる外観にしたい場合は、メールファイルごとに異なるレイアウトファイルを設定できます。

デフォルトのメールレイアウトは、次のようになります。

{% raw %}

```html

<!doctype id="sl-md0000000"     >
<html id="sl-md0000000">
  <head id="sl-md0000000">
  </head>
  <body id="sl-md0000000">
    <img       src="{{portal.url}}/{{theme.images.logo}}"       alt="{{portal.name}}"       style="max-height: 32px; max-width: 200px;" id="sl-md0000000">
    <h3 id="sl-md0000000">{{page.heading}}</h3>
    <p id="sl-md0000000">
      {*page.body*}
    </p>
  </body>
</html>
```

{% endraw %}

`img`タグは、Managerの外観タブで設定できるロゴを読み込みます。ロゴを表示したくない場合は、 `<img id="sl-md0000000">`タグを削除します。ロゴに異なるサイズを設定する場合は、インラインスタイル属性を変更できます。
> 
> 注：ポータルが公開URLからアクセスできるように設定されていない場合（たとえば、ローカルホストでポータルをテストしている場合）、画像をプリフェッチする多くのメールクライアントでは、ロゴが表示されません。

このファイルのHTMLを変更することで、電子メールの外観を変更できます。たとえば、すべてのメールに表示されるフッターを追加したい場合は、`<p id="sl-md0000000">`タグの下に追加してください

サポートする予定のメールクライアントの HTML サポートの制限に注意してください。

既存の Dev Portal でのメールテンプレートの読み込み
-------------------------------

**注:** これは、 {{site.base_gateway}} 1\.3\.0 で作成された既存の開発ポータルにのみ必要です。1\.3\.0\.1以降で作成された新しいポータルには、手動で削除しない限り、これらのファイルはすでにロードされています。

編集可能な電子メールテンプレートは、エディターまたは`kong-portal-cli`ツールを使用して読み込むことができます。

### ポータルエディタを介したメールテンプレートの読み込み

1. Kong Managerにログインし、編集するワークスペースに移動します。サイドバーの **Editor** をクリックしてください。

2. **新規ファイル\+** をクリックします。

3. コンテンツタイプ`email`を選択します。

4. 上記のサポートされているパスのいずれかを入力します。

5. **ファイルを作成** をクリックすると、そのメールに有効なデフォルトのメールテンプレートが生成されます。

6. 編集するメールに対してこれを行います。

   注：デフォルトでは、作成するこれらのメールにはレイアウトキー（`layout: emails/email_base.html` ）が設定されます。
   このレイアウトファイルを作成しない場合、すべてのメールは既定のレイアウトにフォールバックします。メール本文に加えてメールの外観をカスタマイズする場合は、このレイアウトを作成する必要があります。

メールレイアウトを作成する方法:

1. **新規ファイル\+** をクリックします。
2. コンテンツタイプ`theme`を選択します。
3. レイアウトのパス、`themes/<theme_name id="sl-md0000000">/layouts/email_base.html`を入力します。 デフォルトのテーマ名はbaseです。1\.3\.0\.1では、新しいポータルに読み込まれるレイアウトは次のとおりです（これにより、マネージャーの「外観」タブで設定できるロゴ画像が追加されます）。

{% raw %}

```html

<!doctype id="sl-md0000000"     >
<html id="sl-md0000000">
  <head id="sl-md0000000">
  </head>
  <body id="sl-md0000000">
    <img       src="{{portal.url}}/{{theme.images.logo}}"       alt="{{portal.name}}"       style="max-height: 32px; max-width: 200px;" id="sl-md0000000">
    <h3 id="sl-md0000000">{{page.heading}}</h3>
    <p id="sl-md0000000">
      {*page.body*}
    </p>
  </body>
</html>
```

{% endraw %}

メールレイアウトのカスタマイズの詳細については、以下のセクションを参照してください。

### ポータル CLI ツールを使用して電子メール テンプレートをロードする

1. [ポータルテンプレートマスターブランチ](https://github.com/Kong/kong-portal-templates)のクローンを作成し、クローンしたフォルダに移動します。

2. 保持したいカカスタマイゼーションや権限の変更がある場合：

   1. `portal fetch <workspacename id="sl-md0000000">`を実行します。これにより、変更がローカルに取り込まれます。

   2. ポータルテンプレートマスターブランチに統合し、メールを
      含めた変更を1\.3\.0\.1に摘要します。

3. `portal deploy <workspacename id="sl-md0000000">`を実行します。これにより、すべてのファイルがデプロイされます。

