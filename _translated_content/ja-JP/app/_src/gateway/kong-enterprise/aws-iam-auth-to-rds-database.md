---
title: "AWS IAMを使用してKong Gateway Amazon RDSデータベースを認証する"
badge: "enterprise"
content_type: "how-to"
---
{{site.base_gateway}} 3\.3\.x以降では、AWS ID/アクセス管理（IAM）認証を使用して、{{site.base_gateway}}に使用するAWS RDSデータベースに接続できます。このページでは、この機能を使用してデータベース構成とデータベース接続を保護する方法について説明します。

この機能を有効にすると、データベースインスタンスに接続するときにパスワードを使用する必要がなくなります。代わりに、一時的な認証トークンを使用します。AWS IAM は認証を外部で管理するため、データベースにはユーザー認証情報は保存されません。{{site.base_gateway}} のデータベースに AWS RDS を使用する場合は、実行中のクラスタでこの機能を有効にできます。これにより、 {{site.base_gateway}}（`pg_password`）と RDS データベース側の両方にデータベースユーザーの認証情報を保存する必要がなくなります。

AWS IAM認証の制限
------------

AWS IAM 認証には、いくつかの制限もあります。この機能を本番環境で使用する前に、以下をそれぞれ確認してください。

* 従来の{{site.base_gateway}}クラスタまたは単一の従来ノードの場合、{{site.base_gateway}}で1秒あたり200件未満の新しいIAMデータベース認証が必要な場合にのみ、IAMデータベース認証を使用します。1秒あたりの接続数を増やすと、スロットリングが発生する可能性があります。認証は、接続が正常に確立された後の各接続の初期化部分でのみ行われます。後続のクエリと通信は認証されません。データベース上の接続確立のTPSをチェックして、この制限が発生していないことを確認します。従来のクラスタでは、各ノードがデータベースへの接続を確立する必要があるため、この制限が発生する可能性が高くなります。詳細については、Amazon RDSユーザーガイドの[IAMデータベース認証に関する推奨事項](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html#UsingWithRDS.IAMDBAuth.ConnectionsPerSecond)を参照してください。
* AWS IAM認証を有効にするには、データベースへのSSL接続が必要です。これを行うには、RDSクラスタを正しく構成し、{{site.base_gateway}}側で正しいSSL関連の構成を提供する必要があります。以前にSSLを使用していなかった場合は、SSLを有効にすると性能オーバーヘッドも発生します。現在、TLSv1\.3はAWS RDSではサポートされていません。

<!-- -->
* Postgres RDSはmTLSをサポートしていないため、AWS IAM認証が有効になっている場合は、{{site.base_gateway}}とPostgres RDSデータベース間でmTLSを有効にすることはできません。
* {{site.base_gateway}}を起動した後に、AWS認証情報に使用する環境変数の値を変更することは **できません** 。

その他の推奨事項と制限事項については、Amazon RDS ユーザーガイドの「[MariaDB、MySQL、および PostgreSQL の IAM データベース認証](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)」を参照してください。

前提条件
----

AWS IAM認証を有効にする前に、AWS RDSデータベースと{{site.base_gateway}}が使用するAWS IAMロールを構成する必要があります。

* **データベースインスタンスでIAMデータベース認証を有効にします。** 詳細については、『Amazon RDSユーザーガイド』の「[IAMデータベース認証の有効化と無効化](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.Enabling.html)」を参照してください。

* **{{site.base_gateway}}インスタンスに IAM ロールを割り当てます。** {{site.base_gateway}} は、IAM ロールに使用する AWS 認証情報を自動的に検出して取得できます。

  * EC2 環境を使用する場合は、 [EC2 IAM ロール](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)を使用します。

  * ECSクラスタを使用する場合は、 [ECSタスクIAMロール](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)を使用します。

  * EKS クラスタを使用する場合は、割り当てられたロールに注釈を付けることができるKubernetesサービスアカウントを構成し、 [`serviceaccount`で定義された IAMロール](https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html)を使用するようにポッドを構成します。

    `serviceaccount`で定義されたIAMロールを使用するには、AWS STSサービスへのリクエストが必要です。そのため、ポッド内のKongインスタンスがAWS STSサービスエンドポイントにアクセスできることも確認する必要があります。

    STSリージョンエンドポイントを使用している場合は、`AWS_STS_REGIONAL_ENDPOINTS`が環境変数で定義されていることを確実にしてください。
  * {{site.base_gateway}}をローカルで実行する場合は、環境変数を使用します。（`AWS_ACCESS_KEY_ID`と`AWS_SECRET_ACCESS_KEY`を使用したアクセスキーと秘密鍵の組み合わせ、またはプロフィールと認証情報ファイルの組み合わせ（使用：`AWS_PROFILE`と`AWS_SHARED_CREDENTIALS_FILE`

  {:.warning}
  > 
  > **警告：** AWS認証情報の提供に使用した環境変数の値は、{{site.base_gateway}}の起動後には変更 **できません** 。すべての変更は無視されます。{% if_version gte:3.8.x %}
  * ロールを引き受ける場合は、Kongが使用する元のIAMロールにターゲットIAMロールのロールを引き受けるための正しい権限があること、およびターゲットIAMロールにIAM認証を使用してデータベースに接続するための正しい権限があることを確認します。 {% endif_version %}

* **IAM ポリシーを {{site.base_gateway}} IAM ロールに割り当てます** 。詳細については、Amazon RDS ドキュメントの「[IAM データベースアクセス用の IAM ポリシーの作成と使用](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.IAMPolicy.html)」を参照してください。

* **RDSでデータベースアカウントを作成していることを確認** してください。詳細については、Amazon RDSドキュメントの[「PostgreSQL での IAM 認証の使用」](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.DBAccounts.html#UsingWithRDS.IAMDBAuth.DBAccounts.PostgreSQL)を参照してください。

  {:.note}
  > 
  > **注:** 
  > * `rds_iam`ロールに割り当てられたデータベースユーザーは、IAMデータベース認証のみを使用できます。
  > * データベースを作成し、作成したデータベースユーザーに適切な権限を付与してください。詳細については、「[データベースの使用](/gateway/latest/install/linux/debian/#using-a-database)」を参照してください。

AWS IAM 認証の有効化
--------------

環境変数または{{site.base_gateway}}設定ファイルを使用して、AWS IAM 認証を有効にできます。この機能を読み取り専用モードと読み取り/書き込みモードの両方で有効にすることも、読み取り専用モードのみで有効にすることもできます。

{:.note}
> 
> **注意：** AWS IAM認証が有効になっている場合、 {{site.base_gateway}}は関連するパスワード設定を無視します。読み取り専用モードでのみ認証を有効にしても、読み取り/書き込み関連の構成には影響しないため、 `pg_user`と`pg_password`正常に機能します。

AWS IAM 認証を有効にする前に、 `kong.conf`ファイルで次の操作を行う必要があります。

* `pg_password`または`pg_ro_password`を削除します。
* `pg_user` または `pg_ro_user` が、IAM ポリシーで定義し、Postgres RDS データベースに作成したユーザー名と一致していることを確認します。

### 環境変数によるAWS IAM認証の有効化

読み取り/書き込みモードおよび読み取り専用モードでAWS IAM認証を有効にするには、`KONG_PG_IAM_AUTH`環境変数を`on`に設定します。

```bash
KONG_PG_IAM_AUTH=on
```

読み取り専用モードでAWS IAM認証を有効にするには、次のように設定します。

```bash
KONG_PG_IAM_AUTH=off # This line can be omitted because off is the default value
KONG_PG_RO_IAM_AUTH=on
```

{% if_version gte:3.8.x %}
[ロールを引き受ける](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)場合は、次の環境変数も設定します。

```bash
# For read-write connections
KONG_PG_IAM_AUTH_ASSUME_ROLE_ARN=<role_arn id="sl-md0000000">
KONG_PG_IAM_AUTH_ROLE_SESSION_NAME=<role_session_name id="sl-md0000000">

# For read-only connections, if you need a different role than for read-write
KONG_PG_RO_IAM_AUTH_ASSUME_ROLE_ARN=<role_arn id="sl-md0000000">
KONG_PG_RO_IAM_AUTH_ROLE_SESSION_NAME=<role_session_name id="sl-md0000000">
```

{% endif_version %}

### 構成ファイルで AWS IAM 認証を有効化

[`kong.conf`ファイル](/gateway/{{page.release}}/production/kong-conf/)には`pg_iam_auth`プロパティと`pg_ro_iam_auth`プロパティが含まれています。
環境変数と同様に、読み取り接続と書き込み接続の両方で IAM 認証を有効にする場合、または RDS Postgres データベースへの読み取り専用接続のみを有効にする場合は、それに応じて`on`に設定できます。

読み取り/書き込みモードでAWS IAM認証を有効にするには、 `pg_iam_auth`を`on`に設定します。

```text
pg_iam_auth=on
```

読み取り専用モードでAWS IAM認証を有効にするには、 `pg_ro_iam_auth`を`on`に設定します。

```text
pg_ro_iam_auth=on
```

{% if_version gte:3.8.x %}
[ロールを引き受ける](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)場合は、次の構成パラメータも設定します。

```bash
# For read-write connections
pg_iam_auth_assume_role_arn=<role_arn id="sl-md0000000">
pg_iam_auth_role_session_name=<role_session_name id="sl-md0000000">

# For read-only connections, if you need a different role than for read-write
pg_ro_iam_auth_assume_role_arn=<role_arn id="sl-md0000000">
pg_ro_iam_auth_role_session_name=<role_session_name id="sl-md0000000">
```

{% endif_version %}

{:.note}
> 
> **注：** 構成ファイルでAWS IAM認証を有効にする場合、移行コマンドの実行時に機能プロパティで構成ファイルを指定する必要があります。`kong migrations bootstrap -c /path/to/kong.conf`はその一例です。

