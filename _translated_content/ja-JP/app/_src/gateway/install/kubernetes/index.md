---
title: "概要"
book: "kubernetes-install"
chapter: 1
---

{{ site.base_gateway }}は、いくつかの異なるモードでKubernetesにインストールできます。

* [Kubernetesリソースと{{ site.kic_product_name }}を使用して構成された DBレスモード](/kubernetes-ingress-controller/latest/get-started/)
* [{{ site.konnect_saas }}と統合されたデータプレーン（DP）](/konnect/gateway-manager/data-plane-nodes/)
* Kubernetes クラスタ内のコントロールプレーン（CP）とデータプレーン（DP）のハイブリッドモード
* 各ノードがPostgreSQLデータベースに接続する従来モード

このドキュメントでは、Kubernetes クラスタ内でハイブリッドモードで {{ site.base_gateway }} をデプロイする手順を説明します。

サポートされているバージョン
--------------


{{ site.base_gateway }} は、サポートされているすべてのバージョンの Kubernetes および Red Hat OpenShift と互換性があります。

{% include_cached md/kic/support.md %}

