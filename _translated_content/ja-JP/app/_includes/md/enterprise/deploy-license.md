ライセンスファイルは、次のいずれかの方法でデプロイできます。

|                     メソッド                      |                 サポートされているデプロイの種類                 |
|-----------------------------------------------|--------------------------------------------------|
| `/licenses` Admin APIエンドポイント                  | • 従来のデータベースを利用したデプロイメント <br> • ハイブリッドモードのデプロイメント |
| ノードファイルシステム上のファイル <br>\(`license.json`\)    | \- 従来のデータベースを利用したデプロイメント <br>\- DBレスモード        |
| フルライセンスを含む環境変数<br>（`KONG_LICENSE_DATA`）       | \- 従来のデータベースを利用したデプロイメント <br>\- DBレスモード        |
| ライセンスファイルへのパスを含む環境変数<br>（`KONG_LICENSE_PATH`） | \- 従来のデータベースを利用したデプロイメント <br>\- DBレスモード        |

推奨される方法は、Admin APIを使用することです。

{:.important}
> 
> **重要:** 
> 
> * コントロールプレーンの `/license` エンドポイントを使用してライセンスをデプロイすると、コントロールプレーンは接続されたデータプレーンにライセンスを自動的に伝達します。
> * `KONG_LICENSE_DATA`または`KONG_LICENSE_PATH`環境変数を使用してライセンスをデプロイすると、コントロールプレーンはライセンスをデータプレーンノードに伝達 **しません** 。各データプレーンノードにライセンスを追加する **必要があり** 、各ノードはライセンスで起動する **必要があります** 。ノードの起動後にライセンスを追加することはできません。
> 
> ハイブリッドモードのデプロイでは、この方法の使用はお勧めしません。

{{ include.heading }} 前提条件

* Kongから `license.json` ファイルを受け取りました。
* {{site.base_gateway}} がインストールされています。

{{ include.heading }}ライセンスをデプロイする

{% navtabs %}
{% navtab Admin API %}

Kong Admin APIを使用して、ライセンスをいかなるデータベースベースまたは
ハイブリッド モードのデプロイメントにも配布することができます。。ほとんどのデプロイメントでこの方法を使用することをお勧めします。

ハイブリッドモードでは、コントロールプレーン（CP）にライセンスを適用します。コントロールプレーン
（CP）はライセンスをデータプレーン（DP）ノードに配布します。これは、ライセンスをデータ
プレーン（DP）に自動的に適用する唯一の方法です。

ライセンスデータが有効なJSONと見なされるためには、まっすぐの引用符が
含まれている必要があります（`'`と`"`で、`’`や`“`ではありません）。

{:.note}
> 
> **注：** 以下のライセンスのペイロードは一例です。以下の形式を使用する必要がありますが、内容は各自で提供してください。

{% navtabs %}
{% navtab Add new license %}
新しいライセンスを追加するには、{{site.base_gateway}}インスタンスに提供された`license.json`ライセンスの内容を`POST`します。

```bash
curl -i -X POST http://localhost:8001/licenses \
  -d payload='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-20","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b","version":"1"}}'
```

結果：

```json
{
  "created_at": 1500508800,
  "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
  "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-20\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"6985968131533a967fcc721244a979948b1066967f1e9cd65dbd8eeabe060fc32d894a2945f5e4a03c1cd2198c74e058ac63d28b045c2f1fcec95877bd790e1b\",\"version\":\"1\"}}",
  "updated_at": 1500508800
}
```

{% endnavtab %}
{% navtab Update existing license %}
既存のライセンスIDに対して`PATCH`リクエストを使ってライセンスを更新します。

1. 更新したいライセンスを探し、出力からIDをコピーします。

   ```bash
   curl -i -X GET http://localhost:8001/licenses
   ```

2. ライセンスIDに `PATCH` リクエストを送信します。

   ```bash
   curl -i -X PATCH http://localhost:8001/licenses/30b4edb7-0847-4f65-af90-efbed8b0161f \
     payload='{"license":{"payload":{"admin_seats":"1","customer":"Example Company, Inc","dataplanes":"1","license_creation_date":"2017-07-20","license_expiration_date":"2017-07-21","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU","product_subscription":"Konnect Enterprise","support_plan":"None"},"signature":"24cc21223633044c15c300be19cacc26ccc5aca0dd9a12df8a7324a1970fe304bc07b8dcd7fb08d7b92e04169313377ae3b550ead653b951bc44cd2eb59f6beb","version":"1"}}'
   ```

   応答: 

   ```json
   {
     "created_at": 1500595200,
     "id": "30b4edb7-0847-4f65-af90-efbed8b0161f",
     "payload": "{\"license\":{\"payload\":{\"admin_seats\":\"1\",\"customer\":\"Example Company, Inc\",\"dataplanes\":\"1\",\"license_creation_date\":\"2017-07-20\",\"license_expiration_date\":\"2017-07-21\",\"license_key\":\"00141000017ODj3AAG_a1V41000004wT0OEAU\",\"product_subscription\":\"Konnect Enterprise\",\"support_plan\":\"None\"},\"signature\":\"24cc21223633044c15c300be19cacc26ccc5aca0dd9a12df8a7324a1970fe304bc07b8dcd7fb08d7b92e04169313377ae3b550ead653b951bc44cd2eb59f6beb\",\"version\":\"1\"}}",
     "updated_at": 1500595200
   }
   ```

{% endnavtab %}
{% endnavtabs %}

ライセンスの追加または更新後に、{{site.base_gateway}} ノードを[再起動](/gateway/{{page.release}}/reference/cli/#kong-restart)します。

詳細とオプションについては、
[Admin API `licenses`エンドポイントリファレンス](/gateway/latest/licenses/examples)を参照してください。

{% endnavtab %}
{% navtab Filesystem %}

データベースを使用したデプロイメントまたは DB レスデプロイメントの {{site.base_gateway}} にライセンスファイルを提供できます。各ノードのライセンスを手動で維持する必要があるため、このメソッドをハイブリッドモードで使用することは推奨されません。

ライセンスデータが有効なJSONと見なされるためには、まっすぐの引用符が
含まれている必要があります（`'`と`"`で、`’`や`“`ではありません）。

1. {{site.base_gateway}}をインストールしたファイルシステム上のホームディレクトリに `license.json` ファイルを安全にコピーします。

   ```sh
   $ scp license.json <system_username id="sl-md0000000">@<server id="sl-md0000000">:~
   ```

2. 次に、ライセンス ファイルを再度コピーします。今回は`/etc/kong`ディレクトリにコピーします。

   ```sh
   $ scp license.json /etc/kong/license.json
   ```

   {{site.base_gateway}}は、この場所で有効なライセンスを探します。

{% endnavtab %}
{% navtab Environment variable (JSON) %}

`KONG_LICENSE_DATA`環境変数を使用して、データベースを使用したデプロイメントまたはDBレスデプロイメントの{{site.base_gateway}}にライセンスを適用できます。各ノードのライセンスを手動で維持する必要があるため、このメソッドをハイブリッドモードで使用することは推奨されません。

ライセンスデータが有効なJSONと見なされるためには、まっすぐの引用符が
含まれている必要があります（`'`と`"`で、`’`や`“`ではありません）。

1. 次のコマンドを実行し、ライセンスキーを変数にエクスポートします。ライセンスキーはお使いのものに置き換えてください。

   {:.note}
   > 
   > **注:** 以下のライセンスは一例です。以下の形式を使用する必要がありますが、内容は各自で提供してください。

   ```bash
   $ export KONG_LICENSE_DATA='{"license":{"signature":"LS0tLS1CRUdJTiBQR1AgTUVTU0FHRS0tLS0tClZlcnNpb246IEdudVBHIHYyCgpvd0did012TXdDSFdzMTVuUWw3dHhLK01wOTJTR0tLWVc3UU16WTBTVTVNc2toSVREWk1OTFEzVExJek1MY3dTCjA0ek1UVk1OREEwc2pRM04wOHpNalZKVHpOTE1EWk9TVTFLTXpRMVRVNHpTRXMzTjA0d056VXdUTytKWUdNUTQKR05oWW1VQ21NWEJ4Q3NDc3lMQmorTVBmOFhyWmZkNkNqVnJidmkyLzZ6THhzcitBclZtcFZWdnN1K1NiKzFhbgozcjNCeUxCZzdZOVdFL2FYQXJ0NG5lcmVpa2tZS1ozMlNlbGQvMm5iYkRzcmdlWFQzek1BQUE9PQo9b1VnSgotLS0tLUVORCBQR1AgTUVTU0FHRS0tLS0tCg=","payload":{"customer":"Test Company Inc","license_creation_date":"2017-11-08","product_subscription":"Kong Enterprise","admin_seats":"5","support_plan":"None","license_expiration_date":"2017-11-10","license_key":"00141000017ODj3AAG_a1V41000004wT0OEAU"},"version":1}}'
   ```

2. {{site.base_gateway}} コンテナを起動するときに、ライセンスを `docker run` コマンドの一部として含めます。

   {% if_version gte:2.6.x lte:2.8.x %}
   {:.note}
   > 
   > **注:** これは単なるスニペットです。完全な動作例については、[Docker に {{site.base_gateway}} をインストールする](/gateway/{{page.release}}/install-and-run/docker/)手順を参照してください。

   {% endif_version %}
   {% if_version gte:3.0.x %}
   {:.note}
   > 
   > **注:** これは単なるスニペットです。完全な動作例については、[Docker に {{site.base_gateway}} をインストールする](/gateway/{{page.release}}/install/docker/)手順を参照してください。

   {% endif_version %}

   ```bash
   docker run -d --name kong-gateway \
    --network=kong-net \
    ...
    -e KONG_LICENSE_DATA \
    kong/kong-gateway:{{page.releases_hash[page.version-index].ee-version}}-alpine
   ```

{% endnavtab %}
{% navtab Environment variable (file path) %}

`KONG_LICENSE_PATH`環境変数を使用して、データベースを使用したデプロイメントまたはDBレスデプロイメントの{{site.base_gateway}}にライセンスを適用できます。各ノードのライセンスを手動で維持する必要があるため、このメソッドをハイブリッドモードで使用することは推奨されません。

{{site.base_gateway}} コンテナを起動するときに、ライセンスを `docker run` コマンドの一部として含めます。ローカルファイルシステム上のファイルへのパスを Docker コンテナ内のディレクトリにマウントし、コンテナからファイルを表示できるようにします。

{% if_version gte:2.6.x lte:2.8.x %}
{:.note}
> 
> **注:** これは単なるスニペットです。完全な動作例については、[Docker に {{site.base_gateway}} をインストールする](/gateway/{{page.release}}/install-and-run/docker)手順を参照してください。

{% endif_version %}
{% if_version gte:3.0.x %}
{:.note}
> 
> **注:** これは単なるスニペットです。完全な動作例については、[Docker に {{site.base_gateway}} をインストールする](/gateway/{{page.release}}/install/docker)手順を参照してください。

{% endif_version %}

```bash
docker run -d --name kong-gateway \
 --network=kong-net \
 ...
 -v "$(pwd)/kong-license/:/kong-license/" \
 -e "KONG_LICENSE_PATH=/kong-license/license.json" \
 kong/kong-gateway:{{page.releases_hash[page.version-index].ee-version}}-alpine
```

{% endnavtab %}
{% endnavtabs %}

