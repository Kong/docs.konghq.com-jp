---
title: "ブルーグリーンデプロイメント"
content_type: "tutorial"
---
[高度なロードバランシング](/gateway/latest/how-kong-works/load-balancing/#advanced-load-balancing)を使用すると、サービス用ブルーグリーンデプロイメントを簡単に調整できます。
ターゲットインフラを切り替えるには、`host`値を変更するためのサービスの`PATCH`リクエストのみが必要です。

バージョン 1 のアドレスサービスを実行する「ブルー」環境を設定します。

1. アップストリームを作成します。

   ```bash
   curl -X POST http://localhost:8001/upstreams \
       --data "name=address.v1.service"
   ```

2. アップストリームに2つのターゲットを追加します。

   ```sh
   curl -X POST http://localhost:8001/upstreams/address.v1.service/targets \
       --data "target=192.168.34.15:80"
       --data "weight=100"
   curl -X POST http://localhost:8001/upstreams/address.v1.service/targets \
       --data "target=192.168.34.16:80"
       --data "weight=50"
   ```

3. ブルーアップストリームをターゲットとするサービスを作成します。

   ```sh
   curl -X POST http://localhost:8001/services/ \
       --data "name=address-service" \
       --data "host=address.v1.service" \
       --data "path=/address"
   ```

4. 最後に、エントリポイントとしてサービスへルートを追加します。

   ```sh
   curl -X POST http://localhost:8001/services/address-service/routes/ \
       --data "hosts[]=address.mydomain.com"
   ```

ホストヘッダーが`address.mydomain.com`に設定されているリクエストは、{{site.base_gateway}}によって定義された2つのターゲットにプロキシされるようになりました。リクエストの3分の2は`http://192.168.34.15:80/address` （ `weight=100` ）、3分の1は`http://192.168.34.16:80/address` （ `weight=50` ）に送信されます。

アドレスサービスのバージョン2をデプロイする前に、「グリーン」環境を
設定します。

1. アドレスサービスv2用の新しいGreenアップストリームを作成します。

   ```sh
   curl -X POST http://localhost:8001/upstreams \
       --data "name=address.v2.service"
   ```

2. アップストリームにターゲットを追加します。

   ```sh
   curl -X POST http://localhost:8001/upstreams/address.v2.service/targets \
       --data "target=192.168.34.17:80"
       --data "weight=100"
   curl -X POST http://localhost:8001/upstreams/address.v2.service/targets \
       --data "target=192.168.34.18:80"
       --data "weight=100"
   ```

3. ブルーグリーンのスイッチを有効にするには、サービスを更新するだけで済みます。
   サービスをブルーからグリーンのアップストリーム、v1 \-> v2 に切り替えます。

   ```bash
   curl -X PATCH http://localhost:8001/services/address-service \
     --data "host=address.v2.service"
   ```

ホストヘッダーが`address.mydomain.com`に設定された受信リクエストは、Kongによって新しいターゲットにプロキシされるようになりました。リクエストの半分は
`http://192.168.34.17:80/address`（`weight=100`）、残りの半分は`http://192.168.34.18:80/address`（ `weight=100`）に送信されます。

通常通り、{{site.base_gateway}} Admin API による変更は動的であり、ただちに有効になります。再読み込みや再起動は不要で、進行中のリクエストは破棄されません。

