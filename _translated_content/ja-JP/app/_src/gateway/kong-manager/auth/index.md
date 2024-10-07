---
title: "Kong Managerでの認証と承認"
breadcrumb: "概要"
badge: "enterprise"
---
Kong Managerを使用すると、管理者アカウントを持つユーザーはサービス、プラグイン、コンシューマなどのKongエンティティにアクセスできます。

認証の構成
-----


{{site.base_gateway}}には、Kong Managerを保護するために使用できる認証プラグインがパッケージ化されています。Kong Manager *のみ* の認証プラグインを有効にするには、 [`kong.conf`](/gateway/{{page.release}}/production/kong-conf/)で次のプロパティを設定する必要があります。

* `enforce_rbac` の設定を `on`
* `admin_gui_auth`を希望の認証タイプに設定します（例：`basic-auth`）。
* セッションシークレットを使用した `admin_gui_session_conf` の構成
* オプションで、認証タイプに合わせてカスタム構成で`admin_gui_auth_conf`を構成します

構成の詳細は、認証タイプによって異なります。

Kong Managerは現在、次の認証プラグインをサポートしています。

* [Basic Auth](/gateway/{{page.release}}/kong-manager/auth/basic/)
* [OIDC](/gateway/{{page.release}}/kong-manager/auth/oidc/mapping/)
* [LDAP](/gateway/{{page.release}}/kong-manager/auth/ldap/configure/)

上記の認証プラグインに加えて、
[セッションプラグイン](/gateway/{{page.release}}/kong-manager/auth/sessions/)
RBAC が有効な場合に必要です。クライアントのリクエストを認証し、セッション情報を維持するためにHTTPクッキーを送信します。

{:.note}
> 
> **注：** Kong Managerが2FA、MFA、OTP、CAPTCHA、またはreCAPTCHAを直接提供するわけではありませんが、
> [OIDC認証](/gateway/{{page.release}}/kong-manager/auth/oidc/configure/)を使用するようにKong Managerを構成すると、OIDCプロバイダーを介して二次的認証を提供できます。

**セッションプラグイン** （`admin_gui_session_conf`で構成）にはシークレットが必要であり、デフォルトで安全に構成されています。

* どのような状況においても、`secret` は手動で文字列に設定する必要があります。
* HTTPSではなくHTTPを使用する場合は、 `cookie_secure`を手動で`false`に設定する必要があります。{% if_version lte:3.1.x -%}
* Admin APIとKong Managerに異なるドメインを使用する場合、`cookie_samesite`を`off`に設定する必要があります。
{% endif_version -%}
{% if_version gte:3.2.x -%}
* Admin API と Kong Manager に異なるドメインを使用する場合は、`cookie_same_site` を `Lax` に設定する必要があります。 {% endif_version %}

これらのプロパティの詳細については、[Kong ManagerのSessionセキュリティ](/gateway/{{page.release}}/kong-manager/auth/sessions/#session-security)を参照し、[構成例](/gateway/{{page.release}}/kong-manager/auth/sessions/#example-configurations)を確認してください。

ロールとワークスペースによるアクセス制御
--------------------

多くの組織には、厳格なセキュリティ要件があります。たとえば、組織には、ある管理者によるミスや悪意のある行為によってシステム停止が発生しないように、管理者の職務を分離する機能が必要です。{{site.base_gateway}} 、顧客が管理環境を安全に保護できるようにするためのさまざまなセキュリティ機能を提供します。

[ワークスペース](/gateway/{{page.release}}/kong-manager/workspaces/)を使用すると、組織はオブジェクトと管理者を名前空間に分割できます。セグメンテーションにより、同じ {{site.base_gateway}} クラスターを共有する管理者 **チーム** は、特定のオブジェクトを操作する **役割** を担うことができます。たとえば、あるチーム \(チーム A\) が特定のサービスの管理を担当し、別のチーム \(チーム B\) が別のサービスの管理を担当する場合があります。チームには、特定のワークスペース内で管理タスクを実行するために必要な役割のみを付与する必要があります。


{{site.base_gateway}}は、[ロールベースのアクセス制御（RBAC）](/gateway/{{page.release}}/kong-manager/auth/rbac/)を通じてこれらすべてを実行します。Kong ManagerまたはAdmin APIのどちらを使用しているかに関係なく、すべての管理者に特定のロールを与えることができ、これにより特定のワークスペース内の管理権限の範囲を制御および制限できます。

Kong Managerのユーザータイプ：

* [管理者](/gateway/{{page.release}}/kong-manager/auth/rbac/add-admin/)：管理者はワークスペースに属し、一連の権限を持つロールを少なくとも1つ持つ必要があります。
  ロールを *持たない* 管理者がワークスペースにいても、何かを表示したり操作することはできません。管理者は、ユーザーとそのロールを含むワークスペース内のエンティティを管理できます。

* [スーパー管理者](/gateway/{{page.release}}/kong-manager/auth/super-admin/): 以下の権限を持つ特殊な管理者。

  * すべてのワークスペースを管理
  * 権限をさらにカスタマイズする
  * まったく新しい役割を作成する
  * 管理者を招待または非アクティブ化する
  * 管理者ロールの割り当てまたは取り消し

* [RBACユーザー](/gateway/{{page.release}}/kong-manager/auth/rbac/add-user/)：
  管理者権限を持たないRBACユーザー。
  彼らには{{site.base_gateway}}を管理する権限がありますが、チーム、グループ、または
  ユーザー権限の調整はできません。

管理者には、明確に定義された権限を持つ役割が割り当てられます。
Kong Managerでは、権限を制限すると、
アプリケーションインターフェースとナビゲーションの視認性も制限します。

