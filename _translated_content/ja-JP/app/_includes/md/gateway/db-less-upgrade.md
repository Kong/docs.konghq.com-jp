DBレスモードでは、独立した各{{site.base_gateway}}ノードは、永続的なデータベースストレージを使用せずに宣言型{{site.base_gateway}}構成データのコピーをメモリにロードするため、一部のノードの障害が他のノードに広がることはありません。

このモードでのデプロイメントでは、[ローリングアップグレード](/gateway/{{include.release}}/upgrade/rolling-upgrade/)ストラテジを使用する必要があります。
`deck gateway validate` または `kong config parse` コマンドを使用して、バージョン Y の宣言型 YAML コンテンツの有効性を解析できます。

アップグレードを開始する前に、現在の`kong.yaml`ファイルをバックアップする必要があります。

