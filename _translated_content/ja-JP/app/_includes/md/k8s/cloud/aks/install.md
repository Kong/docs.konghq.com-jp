AKSでイングレスリソースを構成するには、`application-gateway-kubernetes-ingress`が[クラスタにインストールされている](https://azure.github.io/application-gateway-kubernetes-ingress/setup/install/)必要があります。

インストール後、クラスタが`ingress-appgw-deployment`を実行していることを確認します。

```bash
kubectl get deployments.apps -n kube-system ingress-appgw-deployment
```

