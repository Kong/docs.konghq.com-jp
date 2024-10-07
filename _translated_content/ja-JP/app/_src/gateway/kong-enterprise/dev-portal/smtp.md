---
title: "Dev PortalのSMTP設定"
badge: "enterprise"
---
Dev Portal では、メールの変数を通して SMTP 構成を有効にします。この変数を使用して Dev Portal は Kong 管理者と開発者にメールを送信します。

SMTP 構成プロパティに関する包括的なドキュメントについては、 [「デフォルトのポータル SMTP 構成」](/gateway/{{page.release}}/reference/configuration/#default-portal-smtp-configuration-section)を参照してください。

これらの設定は、Dev Portal `Settings / Email`タブの`Kong Manager`で変更するか、次のコマンドを実行して変更できます。

```bash
curl http://localhost:8001/workspaces/<workspace_name id="sl-md0000000"> \
  --data "config.<property_name id="sl-md0000000">=off"
```

手動で修正されていない場合、Dev PortalはKong構成ファイルで定義されたデフォルトの値を使用します。

Dev Portalのメールコンテンツとスタイルは、[テンプレートファイル](/gateway/{{page.release}}/kong-enterprise/dev-portal/customize/emails/)でカスタマイズできます。

