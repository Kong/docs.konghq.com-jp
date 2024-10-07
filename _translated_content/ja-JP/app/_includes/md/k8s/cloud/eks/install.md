EKSでイングレスリソースを構成するには、`aws-load-balancer-controller`が[クラスタにインストールされている](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/)必要があります。

インストール後、クラスターが`aws-load-balancer-controller`を実行していることを確認します。

```bash
kubectl get deployments.apps -n kube-system aws-load-balancer-controller
```

