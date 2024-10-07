---
title: "カナリアデプロイメント"
content-type: "reference"
---
[高度なロードバランシングを使用](/gateway/latest/how-kong-works/load-balancing/#advanced-load-balancing)すると、目標重量を細かく調整できるため、
スムーズで制御されたカナリアリリースを実現できます。

非常に単純な2つのターゲットの例を使用します。

```bash
# first target at 1000
curl -X POST http://localhost:8001/upstreams/address.v2.service/targets \
    --data "target=192.168.34.17:80"
    --data "weight=1000"
```

```sh
# second target at 0
curl -X POST http://localhost:8001/upstreams/address.v2.service/targets \
    --data "target=192.168.34.18:80"
    --data "weight=0"
```

リクエストを繰り返し、その都度重みを変更すると、トラフィックはゆっくりと他のターゲットにルーティングされます。たとえば、10%に設定します。

```bash
# first target at 900
curl -X POST http://localhost:8001/upstreams/address.v2.service/targets \
    --data "target=192.168.34.17:80"
    --data "weight=900"
```

```sh
# second target at 100
curl -X POST http://localhost:8001/upstreams/address.v2.service/targets \
    --data "target=192.168.34.18:80"
    --data "weight=100"
```

{{site.base_gateway}} Admin API による変更は動的であり、ただちに有効になります。再読み込みや再起動は不要で、処理中のリクエストは破棄されません。

