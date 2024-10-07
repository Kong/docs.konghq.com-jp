---
title: "バックアップと復元"
content_type: "how-to"
purpose: "Kong Gatewayのバックアップと復元の方法を学ぶ"
---
アップグレードを開始する前に、{{site.base_gateway}}データをバックアップしてください。
Kongでは、{{site.base_gateway}}エンティティのバックアップとして、[データベースネイティブバックアップ](#database-native-backup)と[宣言型バックアップ](#declarative-backup)の2つのバックアップ方法をサポートしています。
データベースネイティブバックアップは、{{site.base_gateway}}データベース全体をバックアップするのに対し、宣言型バックアップは宣言型構成ファイルを管理することで機能します。

復旧の柔軟性を高めるため、両方の方法を使用してデータをバックアップすることをお勧めします。

* データベースネイティブツールは、宣言型ツールに比べて堅牢でかつデータを即座に復元できます。
* データが破損した場合は、まずデータベースレベルの復元を試みてください。そうでない場合は、新しいデータベースをブートストラップし、[宣言型ツール](#declarative-tools-for-backup-and-restore)を使用してエンティティデータ内でロードしてください。

[キーリングマテリアル](#keyring-materials-backup-and-restore)と{{site.base_gateway}}[構成ファイル](#other-files)は別々にバックアップする必要があります。
詳細については、以下のそれぞれのセクションを参照してください。

このガイドで説明するバックアップおよび復元の方法は、一般的な手順です。
インフラ、デプロイメント、ビジネス要件に合わせ、必要に応じて方法を修正してください。

バックアップと復元のための宣言型ツール
-------------------

Kongには、宣言形式の{{site.base_gateway}}エンティティの管理をサポートする[decK](/deck/)と[kong config CLI](/gateway/{{page.release}}/reference/cli/)という2つの宣言型バックアップツールが付属しています。

* **データベースを基盤とするデプロイメント** （従来モードおよびハイブリッドモード）では、これらのツールのいずれかを使用して作成されたバックアップは、追加の保護層として機能します。データベースネイティブのバックアップまたは復元によってデータベースが破損した場合は、宣言型ファイルにフォールバックしてデータを復元できます。

  どちらのツールでも、データベースがデータのエクスポートとインポートに対応している必要があります。これらのツールを使用してデータをインポートまたはエクスポートするには、ユーザーとパスワードが初期化され、データベースがブートストラップされていることを確認してください。
* **DBレスのデプロイメント** では特別なツールは必要ないため、宣言型のツールサポートはありません。宣言型ファイルを手動でバックアップします。

decKは一般的に、kong構成CLIよりも強力です。より多くの機能があり、キャッシュを自動的に無効化し、LRUキャッシュではなくデータベースからエンティティを取得します。さらに、パッチを適用する代わりにエンティティを上書きするので、指定した構成の正確なコピーがデータベースに保存されます。

ただし、decKには次のような制限もあります。

* **可用性** ：decKでは{{site.base_gateway}}がオンラインである必要があります。一方、kong config CLIではオンラインである必要はありません。

* **パフォーマンス** ：decKはエンティティの読み書きにAdmin APIを使用するため、特にエンティティの数が非常に多い場合は、予想以上に時間がかかることがあります。

  この問題は、フラグ`--parallelism`を[`deck gateway sync`](/deck/latest/reference/deck_gateway_sync/)または[`deck gateway diff`](/deck/latest/reference/deck_gateway_diff/)に渡してスレッド数を増やすか、またはdecKの
  [分散構成](/deck/latest/guides/distributed-configuration/)機能を使うことで解決できます。
* **decKが管理するエンティティ** ：decKは、RBACの役割、認証情報、キーリング、ライセンスなどのエンタープライズ専用エンティティを管理しません。これらのセキュリティ関連エンティティは、Admin APIまたはKong Managerを使用して個別に設定してください。
  全リストについては、[decKが管理するエンティティ](/deck/latest/reference/entities/)の参考資料を参照してください。

これらの制限があるため、データベースを使用して展開する場合は、[データベースネイティブな方法](#database-native-backup)を優先することをお勧めします。

Gatewayエンティティのバックアップ
--------------------

### データベースネイティブバックアップ

{{site.base_gateway}}を新しいバージョンにアップグレードする場合は、[`kong migrations`](/gateway/{{page.release}}/reference/cli/#kong-migrations)ユーティリティを使用してデータベースの移行を実行する必要があります。`kong migrations`コマンドは元に戻せません。移行で問題が発生した場合に備えて、アップグレードを開始する前はデータをバックアップすることをお勧めします。

データベースで{{site.base_gateway}}を実行している場合は、データベースネイティブな方法でデータベースを迅速に復旧できるように、生データのデータベースダンプを実行してください。これが、{{site.base_gateway}}をバックアップするお勧めの方法です。

PostgreSQLでは、ユーティリティ`pg_dump`を使用して、 *テキスト* 形式、 *tar* 形式（圧縮なし）、 *またはディレクトリ* 形式（圧縮あり）でデータをダンプできます。以下に例を示します。

```sh
pg_dump -U kong -d kong -F d -f kongdb_backup_20230816
```

特に PostgreSQL インスタンスが{{site.base_gateway}}以外のアプリケーションにもサービスを提供する場合は、CLI オプション`-d`を使用して、エクスポートするデータベース（たとえば、`kong` ）を指定します。

### 宣言型バックアップ

{% navtabs %}
{% navtab Traditional or hybrid mode - decK %}

データベースを活用するデプロイメントでは、予備のバックアップ方法としてdecKの使用をお勧めします。

{:.important}
> 
> この方法で、すべての{{site.base_gateway}}エンティティがバックアップされるわけではありません。この方法をプライマリバックアップとして使用しないでください。

1. decKでデータをバックアップするため、まずは、{{site.base_gateway}}に正常に接続されているか確認します。

   ```sh
   deck gateway ping
   ```

   RBACが有効になっている場合は、CLIオプション`--headers`を使用して
   管理者トークンを指定します。このトークンは、任意のdecKコマンドで指定できます。

   ```sh
   deck gateway ping --headers “Kong-Admin-Token: <password id="sl-md0000000">”
   ```

2. decKを使用して構成をダンプし、結果のファイルを安全な場所に保存します。
   特定のワークスペースまたはすべてのワークスペースを一度にバックアップできます。

   ```sh
   deck gateway dump --all-workspaces -o /path/to/kong_backup.yaml
   ```

   または

   ```sh
   deck gateway dump --workspace it_dept -o /path/to/kong_backup.yaml
   ```

   結果のファイルを安全な場所に保管します。

{% endnavtab %}
{% navtab Traditional or hybrid mode - kong config CLI %}

データベースバックアップのデプロイにおける最終的なフェイルセーフとして、`kong config` CLIを使用してデータベースをバックアップすることもできます。

{:.important}
> 
> この方法はデータベースの最終状態を正確に表していない可能性があるため、プライマリバックアップとして使用しないでください。

```sh
kong config db_export /path/to/kong_backup.yaml
```

これもプライマリバックアップとしては推奨されませんが、冗長性をさらに高めるために使用できます。

{% endnavtab %}
{% navtab DB-less mode %}

DBレスのデプロイメントをバックアップするには、宣言型構成ファイル（デフォルトでは`kong.yml` ）のコピーを作成し、安全な場所に保存します。

宣言型構成ファイルは、[`declarative_config`](/gateway/{{page.release}}/reference/configuration/#declarative_config) ゲートウェイ設定で設定されたパスにあります。

{% endnavtab %}
{% endnavtabs %}

Gatewayエンティティの復元
----------------

### データベースネイティブの復元

データベースネイティブバックアップから{{site.base_gateway}}構成データを復元するには、データベースが準備されていることを確認します。

PostgreSQLの場合:

1. `kong.conf`で、 `pg_user`パラメータを使用してデータベースユーザーを設定します。

       pg_user = kong

2. `kong.conf`で、`pg_database`パラメータを使用してデータベース名を設定します。

       pg_database = kong

3. `migrations`コマンドを使用してデータベースエンティティをブートストラップします。
   詳細については、[`kong migrations` CLIリファレンス](/gateway/{{page.release}}/reference/cli/#kong-migrations)を参照してください。

   ```sh
   kong migrations bootstrap
   ```

4. 以下のように、ユーティリティ`pg_restore`を使用してデータを復元できるようになりました。

   ```sh
   pg_restore -U kong -C -d postgres --if-exists --clean kongdb_backup_20230816/
   ```

### 宣言型の復元

ロールバックする必要がある場合は、{{site.base_gateway}} インスタンスを元のバージョンに戻してください。
宣言的な設定を検証して、それを {{site.base_gateway}} インスタンスに適用します。

{% navtabs %}
{% navtab Traditional or hybrid mode - decK %}

従来モードまたはハイブリッドモードでは、decKを使用してバックアップ状態ファイルから構成を復元します。

1. {{site.base_gateway}}がオンラインであることを確認します。

   ```sh
   deck gateway ping
   ```

2. 宣言型構成を検証します。

   ```sh
   deck gateway validate /path/to/kong_backup.yaml [--online] 
   ```

3. 確認が完了したら、特定のワークスペースまたはすべてのワークスペースを一度に復元します。

   ```sh
   deck gateway sync /path/to/kong_backup.yaml --all-workspaces 
   ```

   または

   ```sh
   deck gateway sync /path/to/kong_backup.yaml --workspace it_dept
   ```

{% endnavtab %}
{% navtab Traditional or hybrid mode - kong config CLI %}

`kong config db_export`を使用して{{site.base_gateway}}データベースをバックアップした場合は、kong構成CLIを使用して、バックアップ宣言型構成ファイルから構成を復元します。

1. 復元する前に、バックアップ構成ファイルを検証します。

   ```sh
   kong config parse /path/to/kong_backup.yaml
   ```

2. エンティティをデータベースにインポートします。

   ```sh
   kong config db_import /path/to/kong_backup.yaml
   ```

3. {{site.base_gateway}}インスタンスを再起動またはリロードします。

   ```sh
   kong restart
   ```

   または

   ```sh
   kong reload
   ```

{% endnavtab %}
{% navtab DB-less mode %}

DBレスモードでは、kong config CLI を使用して、宣言型構成ファイルから構成を復元します。

1. 復元する前に、バックアップ構成ファイルを検証します。

   ```sh
   kong config parse /path/to/kong_backup.yaml
   ```

2. バックアップ構成ファイルを使用して、{{site.base_gateway}}インスタンスを再起動または再読み込みします。

   ```sh
   export KONG_DECLARATIVE_CONFIG=/path/to/kong_backup.yaml; kong restart -c /path/to/kong.conf
   ```

   または

       export KONG_DECLARATIVE_CONFIG=/path/to/kong_backup.yaml; kong reload -c /path/to/kong.conf

   または、宣言型バックアップファイルを`:8001/config`エンドポイントに投稿します。

   ```sh
   curl -sS http://localhost:8001/config?check_hash=1 \
     -F 'config=@/path/to/kong_backup.yaml' ; echo
   ```

{% endnavtab %}
{% endnavtabs %}

キーリング素材のバックアップと復元
-----------------

キーリングとデータ暗号化を有効にしている場合は、キーリング素材を個別にバックアップおよび復元する必要があります。

{:.important}
> 
> **注** ：暗号化キーは必ず安全な場所に保管してください。
> 暗号化キーを紛失すると、暗号化された {{site.base_gateway}} 設定データには永久にアクセスできなくなります。他の方法で回復させることはできません。

技術的な詳細については、[手動バックアップ方法](/gateway/latest/kong-enterprise/db-encryption/#manual-export-and-manual-import)および[自動バックアップ方法](/gateway/latest/kong-enterprise/db-encryption/#automatic-backup-and-manual-recovery)を参照してください。

その他のファイル
--------

次のファイルを手動でバックアップします。

* {{site.base_gateway}}構成ファイルの`kong.conf`。
* キー、証明書、`nginx-kong.conf`、その他のファイルなど、{{site.base_gateway}}プレフィックス内のファイル。
* {{site.base_gateway}}デプロイメント用に作成したその他のファイル。

これらのファイルには{{site.base_gateway}}エンティティが含まれていませんが、これらがないと{{site.base_gateway}}を起動できません。

{:.note}
> 
> **注** : {{site.base_gateway}}がステートレスである商用サービスを構築した場合、つまりAMIコンテナまたはDockerコンテナで構成されるすべてがバージョン管理で定義され、実行中のプラットフォームにプッシュされる場合、{{site.base_gateway}}の構成パラメータを独自の操作方法または安全な方法でバックアップします。

