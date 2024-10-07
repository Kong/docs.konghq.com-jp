---
title: "Admin APIを構成する"
book: "kubernetes-install"
chapter: 3
---

{{ site.base_gateway }}は現在Kubernetes上で実行されています。Admin API は`NodePort`サービスであるため、一般には公開されていません。プロキシサービスは、パブリックアドレスを提供する`LoadBalancer`です。

`kubectl port-forward`を使用せずにAdmin APIにアクセスできるようにするには、選択したクラウドに内部ロードバランサーを作成します。このプロセスは、[Kong Manager]({{ page.book.next.url }})を使用して構成を表示または編集するために必要です。

`values-cp.yaml`ファイルを以下のイングレス構成で更新します。

{% include md/k8s/ingress-setup.md service="admin" release="cp" type="private" %}

