バグ修正のガイドライン
-----------

残念ながら、すべてのソフトウェアにはバグがつきものです。Kongは、次のように構造化されたプロトコルを通じてバグを修正しようとしています。

* 深刻なセキュリティの脆弱性は、最優先で処理されます。脆弱性の報告方法を含む、当社のセキュリティ脆弱性の報告と是正プロセスについては、[こちら](/gateway/latest/production/security-update-process/)をご覧ください。

* 本番環境の停止や実質的な動作不能\(壊滅的なパフォーマンスの低下など\)につながるバグは、優先度の高いバグ修正を通じて修正され、現在サポートされているすべてのソフトウェアのメジャーバージョンの最新メジャー/マイナーバージョンリリースのパッチリリースで提供され、バグの重大度と影響に基づいてKongの裁量で他のバージョンに移植される場合もあります。

* サポートされているバージョンからより新しいバージョンへのアップグレードを妨げるバグは、優先度の高いバグ修正によって修正され、現在サポートされているソフトウェアのすべてのメジャー バージョンの最新のメジャー/マイナー バージョン リリースで提供され、バグの重大度と影響に基づいて Kong の裁量で他のバージョンに移植される場合もあります。

* その他のバグや機能のリクエストについては、バグの影響度に応じて Kong の裁量で重大度が評価され、修正や機能強化がバージョンに適用されます。通常、この種の修正や機能強化は、最新のメジャーバージョンの中の最新のマイナーバージョンにのみ適用されます。

プラチナ以上のサブスクリプションをお持ちのお客様は、上記以外の修正をリクエストすることができ、Kong が独自の裁量でそれを評価します。

非推奨ガイドライン
---------

当社では、製品の進化の一環として、随時、製品の機能や機能を廃止（つまり、削除または中止）することがあります。

当社は、重要な機能の削除または段階的な廃止について、少なくとも6か月前にお客様に通知することを目指しています。セキュリティ上または法律上の理由で変更が必要な場合は、通知を少なくするか、または通知しないことがありますが、そのような状況はまれです。該当する場合は、ドキュメント、製品更新メール、または製品内通知で通知することがあります。

重要な機能を非推奨にすることを発表した後は、通常、その機能を拡張または強化することはありません。

追加条件
----

* 上記は概要のみであり、Kongの[サポートおよびメンテナンスポリシー](https://konghq.com/supportandmaintenancepolicy)によって制限されます。
* 上記は、Kong標準ソフトウェアビルドにのみ適用されます。

