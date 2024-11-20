---
title: 開発ポータルの概要
content_type: explanation
---

{{site.konnect_short_name}}は、開発者のためのカスタマイズ可能なウェブサイトです。Dev Portalは、開発者がAPIサービスを見つけ、アクセスし、利用するためのカスタマイズ可能なウェブサイトです。Devポータルでは、開発者がAPIドキュメントを閲覧・検索したり、APIエンドポイントをテストしたり、自分の認証情報を管理したりすることができます。 {{site.konnect_short_name}}は、{{site.konnect_short_name}}から管理できる内部APIと外部APIの両方をサポートする柔軟な展開オプションを提供します。

## デブ・ポータルの使用例

どの Dev Portal 構成が使用ケースに最適かを判断するには、以下の表を参考にしてください：

| あなたが望むのは... | そうしたいなら... |
| -------------- | ----------- |
| 社内開発者向けと社外パートナー開発者向けの2つのDev PortalsにAPIを公開する。| [Multi-portal](/konnect/dev-portal/create-dev-portal/) |
| 開発者がAPIキーを取得し、APIを使用できるようにする。 | [アプリ登録とDev Portalを有効にする](/konnect/dev-portal/applications/enable-app-reg/) |
| Dev PortalでどのユーザーがどのAPIを見ることができるかを決める| [RBAC Teamsで異なるAPIと権限を割り当てる](/konnect/api/portal-auth/portal-rbac-guide/#main) |
| 開発ポータルのセルフホストまたは視覚的なカスタマイズ | [セルフホスト](/konnect/dev-portal/customization/self-hosted-portal/) |
| APIのドキュメントを公開する| [API製品ドキュメントの追加と公開](/konnect/dev-portal/publish-service/) |
| SIEM プロバイダーで Dev Portal の認証、承認、アクセスログを追跡する | [Dev Portal audit logs](/konnect/dev-portal/audit-logging/) |

状況に応じたすべてのDev Portal設定オプションに関するガイダンスは、[Dev Portal設定準備ガイド](/konnect/dev-portal/configuration-prep/)を参照してください。

Dev Portalを使用した開発者のセルフサービスに関する詳細については、ユースケースに応じていくつかのドキュメントを提供しています。

{% navtabs %}
{% navtab Konnect admin %}
* [アプリケーション登録の有効化と無効化](/konnect/dev-portal/applications/enable-app-reg/) - アプリケーション登録のアクセス権の付与と取り消しについて説明します

* [開発者のアクセス管理](/konnect/dev-portal/access-and-approval/manage-devs/) -  このドキュメントでは、{{site.konnect_short_name}}管理者が、Devポータルへの開発者のアクセスを管理するために利用できるさまざまなオプションの詳細を説明します。 開発者ポータルでは、管理者が開発者ポータルへのアクセス要求を承認および拒否することができます。

* [アプリケーション登録リクエストの管理](/konnect/dev-portal/access-and-approval/manage-devs/) -  開発者がDev PortalのAPIにアプリケーションを登録したい場合、リクエストを作成する必要があります。  登録リクエストは{{site.konnect_short_name}}内で管理できます. このドキュメントには登録リクエストの管理方法が記載されています。アプリケーションがAPIに自動的に登録できるようにするには、[自動承認](/konnect/dev-portal/create-dev-portal/)を参照してください。
{% endnavtab %}
{% navtab Developers %}
開発者については、[開発者としてアプリケーションを登録・作成する](/konnect/dev-portal/register-and-create-app/) を参照してください。
{% endnavtab %}
{% endnavtabs %}

##  開発者のアプリケーション分析 

アプリケーションのアナリティクスは、{{site.konnect_short_name}}内から見ることができます。Dev Portal。これは、異なるAPIバージョン、ルート、およびメソッドの消費に関する洞察を得るのに役立ちます。

![Contextual developer analytics dashboard](/assets/images/products/konnect/dev-portal/konnect-developer-analytics.png)
> _**Figure 1:** Example developer analytics dashboard_

各アプリケーションには独自のダッシュボードがあり、**リクエスト数**、**平均エラー率**、**p99レイテンシー**のハイレベルなサマリーと、以下のデータポイントのチャートを提供します： 

* バージョン別リクエスト
* P99のバージョン別レイテンシー
* エラーコードの分布 

これらの指標はすべて、最大**30日間**、過去90日間の選択した期間内で見ることができます。


