---
title: "代替OpenAPIレンダラー"
badge: "enterprise"
---
Kong Managerを介してDev Portal [Editor](/gateway/{{page.release}}/kong-enterprise/dev-portal/using-the-editor/)を使用し、Dev Portal でのAPI仕様のレンダリング方法をカスタマイズします。

これを行うには、次のようにカスタムコードを追加して構成を更新します。

1. Kong Manager で、左側のサイドバーから Dev Portal Editor に移動します。

2. **テーマ** > **ベース** > **レイアウト** > **システム** > **`spec.renderer.html`** の順に移動します。

3. `spec.renderer.html`で、現在のコンテンツすべてをこの[spec\-renderer.html](https://github.com/Kong/docs.konghq.com/blob/{{ site.git_branch }}/app/code-snippets/_spec-renderer.html)ファイルの内容に置き換えます。これにより、[Stoplight](https://meta.stoplight.io/docs/platform/ZG9jOjIwNjk2MQ-welcome-to-the-stoplight-docs)と[Redoc](https://github.com/Redocly/redoc)レイアウトのオプションが追加されます。

4. `theme.conf.yaml`に、パラメータ`spec_render_type`と値`stoplight`または`redoc`を追加します。以下に例を示します。

   ```yaml
   spec_render_type: "stoplight"
   ```

5. Dev Portal をリフレッシュして、変更が有効になっていることを確認します。

