---
title: "AWS Secrets Manager を使用して Kong Gateway データベースの認証情報を保護する"
content_type: "how-to"
badge: "enterprise"
description: "{{{site.base_gateway}}を維持する方法AWS Secrets ManagerとVault統合を使用してデータベース認証情報を保護します。"
---
アプリケーションシークレットには、パスワード、キー、証明書、トークンなど、保護すべき機密データが含まれます。[{{site.base_gateway}}](/gateway/{{page.release}}/) は[シークレット管理](/gateway/{{page.release}}/kong-enterprise/secrets-management/)をサポートしています。これにより、さまざまな安全なバックエンドシステムにシークレットを保存し、偶発的な漏洩から保護することができます。

従来、{{site.base_gateway}}は外部データベースに接続するために静的認証情報で構成されていました。このガイドでは、従来のファイルまたは環境変数をベースにしたソリューションを使用する代わりに、{{site.base_gateway}}を構成して[AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access.html)を使用し、データベース認証情報を安全に読み取る方法を説明します。

このガイドでは、PostgreSQL と {{site.base_gateway}} を Docker 上でローカルに実行します。AWS Secrets Manager でシークレットを作成し、値を安全に読み取るために vault リファレンスを使用して {{site.base_gateway}} をデプロイします。

### 前提条件

* [AWSアカウント](https://aws.amazon.com/)。AWS Secrets Managerサービスへのアクセスを許可するには、アカウントに適切なIAM権限が必要です。権限ポリシーの例については、[AWS Secrets Managerのドキュメント](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access_examples.html)を参照してください。さらに次の権限も必要です。

  * `secretsmanager:CreateSecret`
  * `secretsmanager:PutSecretValue`
  * `secretsmanager:GetSecretValue`

* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)がインストールされ、構成されています。これは、{{site.base_gateway}}がSecrets Managerサービスに接続するために使用する方法であるため、[AWS CLI環境変数](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)を使用してゲートウェイ環境を構成できる必要があります。

* [Docker](https://docs.docker.com/get-docker/) がインストールされている。

* テストとしてゲートウェイにリクエストを送信するにはシステム上に[`curl`](https://curl.se/)が必要です。ほとんどのシステムには`curl`がプリインストールされています。

### AWS Secrets Managerを使用するために{{site.base_gateway}}を構成

1. {{site.base_gateway}}とデータベースが通信するための Docker ネットワークを作成します。

   ```sh
   docker network create kong-net
   ```

2. データベースを構成して実行します。

   ```sh
   docker run -d --name kong-database \
     --network=kong-net \
     -p 5432:5432 \
     -e "POSTGRES_USER=admin" \
     -e "POSTGRES_PASSWORD=password" \
     postgres:9.6
   ```

   上記で使用されたユーザー名とパスワードはPostgreSQLマスター認証情報であり、データベースを使用して{{site.base_gateway}}を認証するために使用するユーザー名とパスワードでは *ありません* 。

   {:.note}
   > 
   > **注:** 本番環境へのデプロイでは、AWS Secrets Manager との直接統合をサポートする [AWS RDS for PostgreSQL](https://aws.amazon.com/rds/postgresql/) を使用できます。
3. {{site.base_gateway}} のデータベース用のユーザー名とパスワードを作成し
   このガイドで使用する環境変数に保存します。

   ```sh
   KONG_PG_USER=kong
   KONG_PG_PASSWORD=KongFTW
   ```

4. PostgreSQL サーバーコンテナ内に {{site.base_gateway}} データベースユーザーを作成します。

   ```sh
   docker exec -it kong-database psql -U admin -c \
     "CREATE USER ${KONG_PG_USER} WITH PASSWORD '${KONG_PG_PASSWORD}'"
   ```

   次のように表示されます。

   ```sh
   CREATE ROLE
   ```

5. PostgreSQLコンテナ内に`kong`という名前のデータベースを作成します。

   ```sh
   docker exec -it kong-database psql -U admin -c "CREATE DATABASE kong OWNER ${KONG_PG_USER};"
   ```

   次のように表示されます。

   ```sh
   CREATE DATABASE
   ```

6. 新しいAWSシークレットを作成します。

   ```sh
   aws secretsmanager create-secret --name kong-gateway-database \
     --description "Kong GW Database credentials"
   ```

7. 上記で割り当てられた変数のユーザー名とパスワードを使用して、シークレット値を更新します。
   後でシークレット値を更新する場合は、次のコマンドを使用します。

   ```sh
   aws secretsmanager put-secret-value --secret-id kong-gateway-database \
     --secret-string '{"pg_user":"'${KONG_PG_USER}'","pg_password":"'${KONG_PG_PASSWORD}'"}'
   ```

8. {{site.base_gateway}} を起動する前に、次のコマンドを実行してデータベースの移行を実施します。

   {:.note}
   > 
   > **注：** 現在、`kong migrations`ツールはシークレット管理をサポートしていないため、この手順は従来の{{site.base_gateway}}構成オプションを使用して行う必要があります。この例では、環境を介してシークレットをDockerに渡しています。

   ```sh
   docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=$KONG_PG_USER" \
     -e "KONG_PG_PASSWORD=$KONG_PG_PASSWORD" \
     kong/kong-gateway:latest kong migrations bootstrap
   ```

9. データベースのユーザー名とパスワードとして参照できる値を使用するように構成された{{site.base_gateway}}を起動します。{{site.base_gateway}}にAWS Secrets Managerへの接続を許可するには、環境変数を使用してIAMセキュリティ認証情報を指定する必要があります。

   標準の `KONG_` [環境変数名](/gateway/{{page.release}}/reference/configuration/#environment-variables)を使用してデータベース認証情報を指定しますが、静的な値を指定する代わりに[リファレンス値](/gateway/{{page.release}}/kong-enterprise/secrets-management/reference-format/)を使用します。

   形式は、`{vault://aws/kong-gateway-database/pg_user}`のようになります。この例の参照形式には、バックエンドのVaultタイプとして`aws`が含まれ、`kong-gateway-database`は以前作成したシークレット名と一致します。また、`pg_user`はシークレット値で参照するJSONフィールド名です。

   詳細については、[AWS Secrets Managerのドキュメント](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/aws-sm/)をご覧ください。

   現在の環境で `AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_REGION`、および `AWS_SESSION_TOKEN` を設定していると仮定して、次のように{{site.base_gateway}}を開始します。

   ```sh
   docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "AWS_ACCESS_KEY_ID" \
     -e "AWS_SECRET_ACCESS_KEY" \
     -e "AWS_REGION" \
     -e "AWS_SESSION_TOKEN" \
     -e "KONG_PG_USER={vault://aws/kong-gateway-database/pg_user}" \
     -e "KONG_PG_PASSWORD={vault://aws/kong-gateway-database/pg_password}" \
     kong/kong-gateway:{{page.release}}
   ```

   しばらくすると、{{site.base_gateway}} 実行されるはずです。これは、Admin API で確認できます。

   ```sh
   curl -s localhost:8001
   ```

   {{site.base_gateway}}からのさまざまな情報と共にJSON応答を受信します。

[シークレット管理のドキュメント](/gateway/{{page.release}}/kong-enterprise/secrets-management/)には、
利用可能なバックエンドと構成の詳細に関する詳細情報が含まれています。

### 詳細情報

* {{site.base_gateway}}が統合できる、サポートされているVaultバックエンドについては、次のドキュメントを参照してください。 
  * [環境変数Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/env/)
  * [AWS Secrets Manager](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/aws-sm/)
  * [HashiCorp Vault](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/hashicorp-vault/)
  * [Googleシークレット管理](/gateway/{{page.release}}/kong-enterprise/secrets-management/backends/gcp-sm/)

* {{site.base_gateway}}のセキュリティ対策の詳細については、[Kongを安全に起動する](/gateway/{{page.release}}/production/access-control/start-securely/)を参照してください

