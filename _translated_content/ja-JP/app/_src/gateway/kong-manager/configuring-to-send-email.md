---
title: "Kong Managerがメールを送信するように設定"
badge: "enterprise"
---
**スーパー管理者** は、Kong Managerに登録するよう他の **管理者** を招待できます。また、 **管理者は** 「パスワードを忘れた場合」機能を使用してパスワードをリセットできます。どちらのワークフローもユーザーとの通信にメールが使用されます。

Kong Managerからのメールには`kong.conf`を通じた次の構成が必要です。

* [`admin_emails_from`](/gateway/{{page.release}}/reference/configuration/#admin_emails_from)
* [`admin_emails_reply_to`](/gateway/{{page.release}}/reference/configuration/#admin_emails_reply_to)
* [`admin_invitation_expiry`](/gateway/{{page.release}}/reference/configuration/#admin_invitation_expiry)

{{site.base_gateway}} をハイブリッド デプロイモードで実行している場合、これらの管理者 SMTP 設定はコントロールプレーン（CP）に適用されます。

Kongは、構成で設定された電子メールアドレスの有効性をチェックしません。SMTP設定が正しく構成されていない場合、たとえば存在しない電子メールアドレスを指している場合、Kong Managerはエラーメッセージを表示 *しません* 。

{% if_version lte:3.4.x %}
SMTP に関する追加情報については、Kong Manager と Dev Portal で共有される[一般的な SMTP 構成](/gateway/{{page.release}}/reference/configuration/#general-smtp-configuration)を参照してください。
{% endif_version %}
{% if_version gte:3.5.x %}
SMTP に関する追加情報については、[一般的な SMTP 構成](/gateway/{{page.release}}/reference/configuration/#general-smtp-configuration)を参照してください。
{% endif_version %}

