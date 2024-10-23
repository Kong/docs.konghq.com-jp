---
title: "Kong Gatewayを安全に起動する"
badge: "enterprise"
---
Admin APIまたはKong Managerを保護するために、スーパー管理者アカウントが必要です。

スーパー管理者は他の管理者を招待する権限を持ち、
ワークスペース内でロール権限に基づいてアクセスを
制限します。

最初のスーパー管理者アカウントは、以下のガイドに従ってデータベース移行中に作成されます。追加できるのは1回のみです。

前提条件
----

[{{site.base_gateway}}をインストールした](/gateway/{{page.release}}/install/)後、
設定ファイルを変更するか、次のプロパティで環境変数を設定します。

{% if_version gte: 3.8.x %}
* RBAC 認証と認証を有効にするには kong.conf: database=postgresとenforce_rbac=onに以下の設定を追加します。
{% endif_version %}
* `enforce_rbac`は、すべてのAdmin APIリクエストに `Kong-Admin-Token` を要求するように強制します。`Kong-Admin-Token`に関連付けられている管理者は、リクエストを成功させるために適切な権限を持っている必要があります。

* Kong Managerを使用する場合は、管理者がログインに使用する認証の種類を選択します。このガイドの目的では、`admin_gui_auth`を`basic-auth`に設定することができます。他のタイプの認証については、[Kong Managerの保護](/gateway/{{page.release}}/kong-manager/auth/)を参照してください。
ベーシック認証を使用して RBAC を構成します。

{% include_cached /md/admin-listen.md desc='long' release=page.release %}

    enforce_rbac = on
    admin_gui_auth = basic-auth
    admin_gui_session_conf = {"secret":"secret","storage":"kong","cookie_secure":false}
    admin_listen = 0.0.0.0:8001, 0.0.0.0:8444 ssl

Kong Managerはバックグラウンドでセッションプラグインを使用します。このプラグイン（`admin_gui_session_conf`で構成）にはシークレットが必要であり、デフォルトで安全に構成されています。

* どのような状況においても、`secret` は手動で文字列に設定する必要があります。
* HTTPS ではなく HTTP を使用する場合は、`cookie_secure` を手動で `false` に設定する必要があります。
{% if_version lte:3.1.x -%}
* Admin API と Kong Manager に異なるドメインを使用する場合は、`cookie_samesite` を `off` に設定する必要があります。
{% endif_version -%}
{% if_version gte:3.2.x -%}
* Admin API と Kong Manager に異なるドメインを使用する場合は、 `cookie_same_site` を `Lax` に設定する必要があります。 {% endif_version %}

これらのプロパティの詳細については、[Kong ManagerのSessionセキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照し、[構成例](/gateway/{{page.release}}/kong-manager/auth/sessions#example-configurations)を確認してください。

手順1
---

スーパー管理者のパスワードを設定します。この環境変数は、データベースの移行が実行される環境に存在している必要があります。

{:.important}
> 
> **重要** : 4 つのティックを含む値 \(たとえば、 `KONG_PASSWORD="a''a'a'a'a"` \) を使用して Kong パスワード \( `KONG_PASSWORD` \) を設定すると、ブートストラップ時に PostgreSQL 構文エラーが発生します。 パスワードに特殊文字を使用しないことで、この問題を回避できます。

    export KONG_PASSWORD=<password-only-you-know id="sl-md0000000">

これにより、Kong Manager へのログインに使用できるユーザー、`kong_admin`、パスワードが自動的に作成されます。このパスワードは、Admin APIリクエストを行うための`Kong-Admin-Token`としても使用できます。

**注：** この方法で作成できるスーパー管理者は1人のみで、データベースの新規インストールでのみ作成できます。移行中に作成されていない場合は、[前提条件](#prerequisites)で設定を元に戻し、[Kong Manager で新しいユーザーをスーパー管理者として招待します](/gateway/{{page.release}}/kong-manager/auth/super-admin/)。

今後の移行では、パスワードが更新されたり、追加のスーパー管理者が作成されたりすることはありません。
スーパー管理者を追加するには、
[Kong Manager で新しいユーザーをスーパー管理者として招待します](/gateway/{{page.release}}/kong-manager/auth//super-admin/)。

ステップ2
-----

次のコマンドを発行して、Kongの移行を実行してデータストアを準備します。

    kong migrations bootstrap [-c /path/to/kong.conf]

ステップ3
-----

Kongを起動します。

    kong start [-c /path/to/kong.conf]

**注：** CLIは構成オプション（`-c /path/to/kong.conf`）を受け入れて、ユーザーが[独自の構成](/gateway/{{page.release}}/reference/configuration/#configuration-loading)を指すことができるようにします。

ステップ4
-----

{{site.base_gateway}}がスーパー管理者として正常に起動したかどうかをテストするには、
Kong ManagerのURLにアクセスしてください。デフォルトでは、ポート`:8002`にあります。

ユーザー名は`kong_admin`で、パスワードは[ステップ 1](#step-1)に
設定されています。

