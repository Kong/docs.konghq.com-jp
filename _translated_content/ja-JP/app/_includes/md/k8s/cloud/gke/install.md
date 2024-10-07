GKEバージョン1\.18以降を実行するクラスタは、`Ingress`リソースの作成に応じてロードバランサーを自動的にプロビジョニングします。

GKEでは、Kongのデプロイメントが正常とマークされるためには `BackendConfig` リソースを作成する必要があります。

1. ヘルスチェックを構成するには、`BackendConfig`[リソース](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#interpreted_hc)を作成します。

   ```yaml
   echo "apiVersion: cloud.google.com/v1
   kind: BackendConfig
   metadata:
     name: kong-hc
     namespace: kong
   spec:
     healthCheck:
       checkIntervalSec: 15
       port: 8100
       type: HTTP
       requestPath: /status" | kubectl apply -f -
   ```

2. この `BackendConfig` は、次の中にある `annotations` キーを使用して `{{ include.service }}` サービスにアタッチされます`values-{{ include.release }}.yaml`

{:.important}
> 
> GKEは、`Ingress`定義ごとにロードバランサを1つプロビジョニングします。このガイドに従うと、複数のロードバランサが作成されます。

