---
title: "外部プラグインのパフォーマンス"
content_type: "reference"
---
実装の詳細に応じて、Goプラグインは複数のCPUコアを使用して、マルチコアシステムで最高のパフォーマンスを発揮できます。JavaScriptプラグインは現在シングルコアのみで、専用のプラグインサーバーサポートはありません。Pythonプラグインは専用のプラグインサーバーを使用して、ワークロードを複数のCPUコアに分散することもできます。

PDK関数の呼び出しがローカルプロセスで処理されるLuaプラグインとは異なり、
外部プラグインでPDK関数を呼び出すことはプロセス間通信を意味するため、
比較的高価な操作となります。外部プラグインでPDK関数を呼び出すにはコストがかかるため、外部プラグインを使用するKongのパフォーマンスは、
各リクエストにおけるIPC（プロセス間通信）の呼び出し回数に大きく影響します。

次のグラフは、パフォーマンスとリクエストごとのIPC呼び出し数の
相関関係を示しています。RPSとレイテンシの数値はハードウェアに依存しており、
混乱を避けるために排除されています
<center><img title="RPS" src="/assets/images/products/plugins/external-plugins/rps.png"/></center> <center><img title="レイテンシ" src="/assets/images/products/plugins/external-plugins/latency.png"/></center>

