---
title: "式のパフォーマンスの最適化"
content-type: "reference"
---
APIトラフィックのプロキシでは、パフォーマンスがきわめて重要です。このガイドでは、ルーティングエンジンのパフォーマンスを最大限に引き出すために
記述する式を最適化する方法を説明します。

ルート数
----

### ルートの一致優先順位

式ルートは、常に定義された `priority` の降順で評価されます。したがって、マッチする可能性の高いルートを、マッチする頻度の低いルートより先に（優先順位を高くして）配置すると便利です。

次の例は、一致する可能性があるかどうかに基づいて2つのルートに優先順位を付ける方法を示しています。

例ルート1：

    expression: http.path == "/likely/matched/request/path"
    priority: 100

サンプルルート2：

    expression: http.path == "/unlikely/matched/request/path"
    priority: 50

また、式言語の論理組み合わせ機能を活用することで、`Route`エンティティの数を減らすのが最善です。

### ルートの組み合わせ

複数のルートが同じ`Service`と`Plugin`の構成を使用する結果になる場合、それらは`||`論理演算子または演算子で一つの式`Route`に結合する必要があります。ルートを1つの式にまとめることで、作成される `Route` オブジェクトの数が減り、パフォーマンスが向上します。

例ルート1：

    service: example-service
    expression: http.path == "/hello"

サンプルルート2：

    service: example-service
    expression: http.path == "/world"

これら2つのルートは次のように組み合わせることもできます。

    service: example-service
    expression: http.path == "/hello" || http.path == "/world"

正規表現の使用法
--------

正規表現（regex）はきわめて複雑な条件に基づいて文字列を照合するために使用できる協力ツールです。
残念ながらこの特質は、実行時の評価コストが上昇し、最適化が困難になる要因にもなります。
そのため、正規表現を使用せずに照合パフォーマンスを大幅に改善するためによく使用されるシナリオがいくつか考案されています。

リクエストパスの完全一致（プレフィックス以外の一致）を実行する場合、正規表現でなく、`==`演算子を使用します。

**より高速なパフォーマンスの例：** 

    http.path == "/foo/bar"

**パフォーマンスの低下の例：** 

    http.path ~ r#"^/foo/bar$"#

末尾に `/` オプションのスラッシュを付けて完全一致を実行すると、正規表現を書きたくなります。ただし、これは式言語ではまったく不要です。

**より高速なパフォーマンスの例：** 

    http.path == "/foo/bar" || http.path == "/foo/bar/"

**パフォーマンスの低下の例：** 

    http.path ~ r#"^/foo/?$"#
