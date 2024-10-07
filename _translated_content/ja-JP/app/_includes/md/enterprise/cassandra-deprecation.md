{% if include.length == "short" %}

Cassandraのサポートは[非推奨になり、削除される予定であるため](/gateway/{{include.release}}/reference/configuration/#cassandra-settings)。{{site.base_gateway}}にCasssandraを使用することは推奨されません。

{% else %}

{:.warning .no-icon}
> 
> **非推奨の警告:** {{site.base_gateway}}のバックエンド データベースとしての Cassandra
> は非推奨です。Cassandraのサポートは将来のリリースで削除される予定です。
> <br>
> Cassandra の削除目標は{{site.base_gateway}} 3\.4 リリースです。
> {{site.base_gateway}} 3\.0リリース以降、追加予定の新機能は
> カサンドラではサポートされていません。

{% endif %}

