---
title: "導入"
book: "plugin_dev_getstarted"
chapter: 1
---
このチュートリアルでは、{{site.base_gateway}}の新しいカスタムプラグインのすべてのビルド手順を順番に説明します。
完成したプラグインは、APIゲートウェイの通過時にHTTPトラフィックを変換します。また、独自のビジネスニーズを満たすためにカスタムプラグインをビルドする堅実な出発点となります。

このガイドではまず、新しいプラグインプロジェクトをセットアップします。そのために、フォルダ構造を示し、コードファイルコンテンツとベストプラクティスについて説明します。
次に、テストツールとカスタムプラグイン向けの自動化テストのビルド方法について説明します。
続いて、プロジェクトへのプラグイン構成の追加方法を示して、ランタイム動作を変更できるようにします。

このガイドの後の章では、プラグイン内から外部サービスを利用する方法やプラグインのデプロイなど、詳細なトピックに関するガイダンスを提供します。

ガイド内を移動するには、これらのページの下部にあるリンクを使用してください。これらのページのサンプルコードやガイダンスに関して問題が発生した場合は、[GitHubで問題を報告](https://github.com/Kong/docs.konghq.com/issues)してご連絡ください。

{:.note}
> 
> **注** ：必須ではありませんが、[Lua](https://www.lua.org/about.html)言語を理解していると、このガイドの使用に役立ちます。

{:.note}
> 
> **注** ：このガイドは、カスタムプラグインを最初からすばやく構築できるように
> 書かれています。{{site.base_gateway}}を使ったプラグイン開発に関する高度なトピックについては、
> [プラグイン開発セクション](/gateway/{{page.release}}/plugin-development/)のドキュメントの残りの部分を
> 必ず参照してください。
