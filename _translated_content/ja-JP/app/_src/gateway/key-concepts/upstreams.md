---
title: "アップストリーム"
concept_type: "説明"
---
アップストリームとは、{{site.base_gateway}}がリクエストを転送するAPI、アプリケーション、またはマイクロサービスを指します。{{site.base_gateway}}では、アップストリームオブジェクトは仮想ホスト名を表し、ヘルスチェック、サーキットブレーク、複数のサービスにわたる受信リクエストのロードバランシングに使用できます。

アップストリームとサービスのやり取り
------------------

ホストではなくアップストリームを指すように[サービス](/gateway/{{ page.release }}/key-concepts/services/)を構成できます。
たとえば、 `example_service`というサービスと`example_upstream`というアップストリームがある場合、ホストを指定する代わりに、 `example_service`を`example_upstream`に指すことができます。`example_upstream`アップストリームは、2 つの異なるターゲット、`httpbin.org`と`httpbun.com`を指すことができます。実際の環境では、アップストリームは複数のシステムで実行されている同じサービスを指します。

この設定により、アップストリームターゲット間で[ロードバランシング](/gateway/{{ page.release }}/how-kong-works/load-balancing)が可能になります。
たとえば、アプリケーションが2つの異なるサーバーまたはアップストリームターゲットにデプロイされる場合、{{site.base_gateway}}は両方のサーバーに負荷を分散する必要があります。
その結果、一方のサーバー（前例の`httpbin.org`）を利用できない場合は、その問題を自動的に検知して、すべてのトラフィックが動作しているサーバー（`httpbun.com`）にルーティングされるようになります。

アップストリーム構成
----------

次の方法を使用して、 {{site.base_gateway}}のサービスにアップストリームを追加できます。

* Kong Managerの使用
* Admin APIの使用
* decK（YAML）の使用

アップストリームの設定方法の詳細については、[ロードバランシングの設定](/gateway/{{ page.release }}/get-started/load-balancing/)を参照してください。

