{{ site.kic_product_name }} を有効にするには、`values-{{ include.release }}.yaml` ファイルで `ingressController.enabled` を `true` に設定します。イングレスコントローラーを有効にするときは、`env.publish_service` を設定して、管理された `Ingress` リソースのアドレスフィールドに {{ site.kic_product_name }} が入力されるようにします。

{{ site.kic_product_name }}と{{ site.base_gateway }} Admin API間の通信を有効にするには、 `ingressController.env.kong_admin_token`を`env.password`に保存されている値に設定する必要があります。

```yaml
ingressController:
  enabled: true
  env:
    publish_service: kong/kong-dp-kong-proxy
    kong_admin_token: kong_admin_password
```

