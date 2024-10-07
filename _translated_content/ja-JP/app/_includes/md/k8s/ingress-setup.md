{% unless include.skip_ingress_controller_install %}

1. イングレスコントローラを設定します。

{% include md/k8s/controller-install-tabs.md indent=true service=include.service release=include.release %}

{% endunless %}

1. `values-{{ include.release }}.yaml`で`{{ include.service }}` セクションを設定します。`example.com` をカスタムドメイン名に置き換えます。

{% include md/k8s/ingress-tabs.md indent=true service=include.service type=include.type %}

{% unless include.skip_release %}

1. `helm upgrade`を実行してリリースを作成します。

   ```bash
   helm upgrade kong-{{ include.release }} kong/kong -n kong --values ./values-{{ include.release }}.yaml
   ```

{% endunless %}

{% unless include.skip_dns %}

1. `Ingress` IPアドレスを取得し、イングレスアドレスを指すようにDNSレコードを更新します。DNSを手動で構成することも、[external\-dns](https://github.com/kubernetes-sigs/external-dns)などのツールを使用してDNS構成を自動化することもできます。

   ```bash
   kubectl get ingress -n kong kong-{{ include.release }}-kong-{{ include.service }} -o jsonpath='{.spec.rules[0].host}{": "}{range .status.loadBalancer.ingress[0]}{@.ip}{@.hostname}{end}'
   ```

{% endunless %}

