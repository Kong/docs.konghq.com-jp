---
title: "Kong Gatewayをハイブリッドモードでデプロイする"
---
前提条件
----

ハイブリッドモードのデプロイメントを開始するには、まずコントロールプレーン（CP）ノードとしてTLSを使用する

{{site.base_gateway}}のインスタンスをインストールします。詳細については、
[インストールドキュメント](/gateway/{{page.release}}/install/)
を参照してください。

このトピックでは、後続のデータプレーン（DP）インスタンスについて説明します。

{:.note}
> 
> **注：** Kubernetes でのハイブリッドモードのデプロイメントについては、
> `kong/charts`リポジトリの「[ハイブリッドモード](https://github.com/Kong/charts/blob/main/charts/kong/README.md#hybrid-mode)」を参照してください。

証明書/キーペアを生成
-----------

ハイブリッドモードでは、TLS相互認証ハンドシェイク（mTLS）が認証に使用されるので、
実際の秘密鍵は、ネットワークで転送されることはなく、
CPノードとDPノード間の通信は安全です。

ハイブリッドモードを使用する前に、証明書/キーペアが必要です。

{{site.base_gateway}}には、証明書/キーペアを処理するための 2 つのモードがあります。

* **共有モード:** （デフォルト）Kong CLI を使用して証明書/キーペアを生成し、次にそのコピーをノード間で配布します。証明書/キーペアは、CPノードとDPノードの両方で共有されます。
* **PKIモード：** 中央認証局が署名した証明書を提供します （CA）。Kongは、両方が同じCAからのものであるかどうかを確認して検証します。これにより、プライベート鍵の転送に関連するリスクを排除します。

{:.warning}
> 
> **警告:** DPノードとCPノードの間にTLS対応プロキシがある場合は、
> PKIモードを使用し、`kong.conf`で `cluster_server_name` を
> CPホスト名に設定する必要があります。共有モードは使用しないでください。共有モードはTLSサーバー名に非標準の値を使用するため、
> SNIに依存してトラフィックをルーティングするTLS対応プロキシを
> 混乱させます。

これらのモードで使用されるプロパティの内訳については、[構成リファレンス](#configuration-reference)を参照してください。

{% navtabs %}
{% navtab Shared mode %}
{:.warning}
> 
> **警告:** 秘密鍵を保護してください。秘密鍵ファイルには、クラスタに属する Kong ノードだけがアクセスできるようにしてください。鍵が漏洩した場合は、すべての CPノードと DPノードの証明書と鍵を再生成して交換する必要があります。

1. 既存の {{site.base_gateway}} インスタンスで、証明書/キーペアを作成します。

   ```bash
   kong hybrid gen_cert
   ```

   これにより、`cluster.crt`と`cluster.key`ファイルが生成され、現在のディレクトリに
   保存されます。デフォルトでは、証明書/キーペアは3年間有効ですが、
   `--days`オプションで調整できます。使用方法の詳細については、`kong hybrid --help`を
   参照してください。
2. `cluster.crt`ファイルと`cluster.key`ファイルを、すべてのKong CPおよびDPノード上の
   同じディレクトリ（例： `/cluster/cluster` ）にコピーします。
   キーファイルに適切な権限を設定して、Kongのみが読み取れるようにします。

{% endnavtab %}
{% navtab PKI mode %}

PKI モードでは、ハイブリッドクラスタは中央認証局（CA）が署名した証明書を使用できます。

このモードでは、コントロールプレーン（CP）とデータプレーン（DP）に同じ`cluster_cert`と`cluster_cert_key`を使用する必要はありません。
代わりにKongは、両方のCAが同じかどうかを確認して検証します。

Kongが稼働するホストでCA証明書を準備します。

{% navtabs %}
{% navtab CA Certificate Example %}
通常、CA 証明書は次のようになります。

    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                5d:29:73:bf:c3:da:5f:60:69:da:73:ed:0e:2e:97:6f:7f:4c:db:4b
            Signature Algorithm: ecdsa-with-SHA256
            Issuer: O = Kong Inc., CN = Hybrid Root CA
            Validity
                Not Before: Jul  7 12:36:10 2020 GMT
                Not After : Jul  7 12:36:40 2023 GMT
            Subject: O = Kong Inc., CN = Hybrid Root CA
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:df:49:9f:39:e6:2c:52:9f:46:7a:df:ae:7b:9b:
                        87:1e:76:bb:2e:1d:9c:61:77:07:e5:8a:ba:34:53:
                        3a:27:4c:1e:76:23:b4:a2:08:80:b4:1f:18:7a:0b:
                        79:de:ea:8c:23:94:e6:2f:57:cf:27:b4:0a:52:59:
                        90:2c:2b:86:03
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
            X509v3 extensions:
                X509v3 Key Usage: critical
                    Certificate Sign, CRL Sign
                X509v3 Basic Constraints: critical
                    CA:TRUE
                X509v3 Subject Key Identifier:
                    8A:0F:07:61:1A:0F:F4:B4:5D:B7:F3:B7:28:D1:C5:4B:81:A2:B9:25
                X509v3 Authority Key Identifier:
                    keyid:8A:0F:07:61:1A:0F:F4:B4:5D:B7:F3:B7:28:D1:C5:4B:81:A2:B9:25
    
        Signature Algorithm: ecdsa-with-SHA256
             30:45:02:20:68:3c:d1:f3:63:a2:aa:b4:59:c9:52:af:33:b7:
             3f:ca:3a:2b:1c:9d:87:0c:c0:47:ff:a2:c4:af:3e:b0:36:29:
             02:21:00:86:ce:d0:fc:ba:92:e9:59:16:1c:c3:b2:11:11:ed:
             01:5d:16:49:d0:f9:0c:1d:35:0d:40:ba:19:98:31:76:57

{% endnavtab %}

{% navtab Certificate on CP %}
以下は、コントロールプレーン（CO）の証明書の例です。

    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                18:cc:a3:6b:aa:77:0a:69:c6:d5:ff:12:be:be:c0:ac:5c:ff:f1:1e
            Signature Algorithm: ecdsa-with-SHA256
            Issuer: CN = Hybrid Intermediate CA
            Validity
                Not Before: Jul 31 00:59:29 2020 GMT
                Not After : Oct 29 00:59:59 2020 GMT
            Subject: CN = control-plane.kong.yourcorp.tld
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:f8:3a:a9:d2:e2:79:19:19:f3:1c:58:a0:23:60:
                        78:04:1f:7e:e2:bb:60:d2:29:50:ad:7c:9b:8e:22:
                        1c:54:c2:ce:68:b8:6c:8a:f6:92:9d:0c:ce:08:d3:
                        aa:0c:20:67:41:32:18:63:c9:dd:50:31:60:d6:8b:
                        8d:f9:7b:b5:37
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
            X509v3 extensions:
                X509v3 Key Usage: critical
                    Digital Signature, Key Encipherment, Key Agreement
                X509v3 Extended Key Usage:
                    TLS Web Server Authentication
                X509v3 Subject Key Identifier:
                    70:C7:F0:3B:CD:EB:8D:1B:FF:6A:7C:E0:A4:F0:C6:4C:4A:19:B8:7F
                X509v3 Authority Key Identifier:
                    keyid:16:0D:CF:92:3B:31:B0:61:E5:AB:EE:91:42:B9:60:56:0A:88:92:82
    
                X509v3 Subject Alternative Name:
                    DNS:control-plane.kong.yourcorp.tld, DNS:alternate-control-plane.kong.yourcorp.tld
                X509v3 CRL Distribution Points:
    
                    Full Name:
                      URI:https://crl-service.yourcorp.tld/v1/pki/crl
    
        Signature Algorithm: ecdsa-with-SHA256
             30:44:02:20:5d:dd:ec:a8:4f:e7:5b:7d:2f:3f:ec:b5:40:d7:
             de:5e:96:e1:db:b7:73:d6:84:2e:be:89:93:77:f1:05:07:f3:
             02:20:16:56:d9:90:06:cf:98:07:87:33:dc:ef:f4:cc:6b:d1:
             19:8f:64:ee:82:a6:e8:e6:de:57:a7:24:82:72:82:49

{% endnavtab %}

{% navtab Certificate on DP %}
データプレーン（DP）の証明書の例を次に示します。

    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                4d:8b:eb:89:a2:ed:b5:29:80:94:31:e4:94:86:ce:4f:98:5a:ad:a0
            Signature Algorithm: ecdsa-with-SHA256
            Issuer: CN = Hybrid Intermediate CA
            Validity
                Not Before: Jul 31 00:57:01 2020 GMT
                Not After : Oct 29 00:57:31 2020 GMT
            Subject: CN = kong-dp-ce39edecp.service
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:19:51:80:4c:6d:8c:a8:05:63:42:71:a2:9a:23:
                        34:34:92:c6:2a:d3:e5:15:6e:36:44:85:64:0a:4c:
                        12:16:82:3f:b7:4c:e1:a1:5a:49:5d:4c:5e:af:3c:
                        c1:37:e7:91:e2:b5:52:41:a0:51:ac:13:7b:cc:69:
                        93:82:9b:2f:e2
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
            X509v3 extensions:
                X509v3 Key Usage: critical
                    Digital Signature, Key Encipherment, Key Agreement
                X509v3 Extended Key Usage:
                    TLS Web Client Authentication
                X509v3 Subject Key Identifier:
                    25:82:8C:93:85:35:C3:D6:34:CF:CB:7B:D6:14:97:46:84:B9:2B:87
                X509v3 Authority Key Identifier:
                    keyid:16:0D:CF:92:3B:31:B0:61:E5:AB:EE:91:42:B9:60:56:0A:88:92:82
                X509v3 CRL Distribution Points:
    
                    Full Name:
                      URI:https://crl-service.yourcorp.tld/v1/pki/crl
    
        Signature Algorithm: ecdsa-with-SHA256
             30:44:02:20:65:2f:5e:30:f7:a4:28:14:88:53:58:c5:85:24:
             35:50:25:c9:fe:db:2f:72:9f:ad:7d:a0:67:67:36:32:2b:d2:
             02:20:2a:27:7d:eb:75:a6:ee:65:8b:f1:66:a4:99:32:56:7c:
             ad:ca:3a:d5:50:8f:cf:aa:6d:c2:1c:af:a4:ca:75:e8

{% endnavtab %}
{% endnavtabs %}
> 
> **注:** CP および DP の証明書には `TLS Web Server Authentication` と `TLS Web Client Authentication` が、それぞれ X509v3 拡張キー使用法拡張機能として含まれる必要があります。

Kong は DP 証明書の CommonName（CN）を検証しません。任意の値を取ることができます。

{% endnavtab %}
{% endnavtabs %}

コントロールプレーンを設定する
---------------

次に、コントロールプレーンノードに `control_plane`のロールを与えて、
証明書とキーの場所を指定するように証明書/キーパラメータを設定します。

{% navtabs %}
{% navtab Using Docker %}

1. Docker コンテナで、次の環境変数を設定します。

   `shared`証明書モードの場合は、次を使用します。

   ```bash
   KONG_ROLE=control_plane
   KONG_CLUSTER_CERT=/<path-to-file id="sl-md0000000">/cluster.crt
   KONG_CLUSTER_CERT_KEY=/<path-to-file id="sl-md0000000">/cluster.key
   ```

   `pki`証明書モードの場合は、次を使用します。

   ```bash
   KONG_ROLE=control_plane
   KONG_CLUSTER_MTLS=pki
   KONG_CLUSTER_CA_CERT=/<path-to-file id="sl-md0000000">/ca-cert.pem
   KONG_CLUSTER_CERT=/<path-to-file id="sl-md0000000">/control-plane.crt
   KONG_CLUSTER_CERT_KEY=/<path-to-file id="sl-md0000000">/control-plane.key
   ```

   ノードのロールを `control_plane`に設定すると、このノードはデフォルトではデータプレーン接続をポート `0.0.0.0:8005`でリッスンし、テレメトリデータを`0.0.0.0:8006`でリッスンします。コントロールプレーン上のこれらのポートは、設置されているすべてのファイアウォールを通して、制御対象となるすべてのデータプレーンからアクセスできる必要があります。

   PKIモードの場合、`KONG_CLUSTER_CA_CERT` は `KONG_CLUSTER_CERT` および `KONG_CLUSTER_CERT_KEY` のルートCA証明書を指定します。この証明書は、中間 CA ではなく、ルート CA 証明書である必要があります。Kong では、ルート CA とクラスタ証明書の間で最大 3 階層の中間 CA を使用できます。

   コントロールプレーンがリッスンするポートを変更する必要がある場合は、以下を設定します。

   ```bash
   KONG_CLUSTER_LISTEN=0.0.0.0:<port id="sl-md0000000">
   KONG_CLUSTER_TELEMETRY_LISTEN=0.0.0.0:<port id="sl-md0000000">
   ```

2. 次に、Kongを起動します。すでに起動している場合は Kongをリロードします。

   ```bash
   kong start
   ```

   ```bash
   kong reload
   ```

{% endnavtab %}
{% navtab Using kong.conf %}

1. `kong.conf` で次の構成パラメータを設定します。

   `shared`証明書モードの場合は、次を使用します。

   ```bash
   role = control_plane
   cluster_cert = /<path-to-file id="sl-md0000000">/cluster.crt
   cluster_cert_key = /<path-to-file id="sl-md0000000">/cluster.key
   ```

   `pki`証明書モードの場合は、次を使用します。

   ```bash
   role = control_plane
   cluster_mtls = pki
   cluster_ca_cert = /<path-to-file id="sl-md0000000">/ca-cert.pem
   cluster_cert = /<path-to-file id="sl-md0000000">/control-plane.crt
   cluster_cert_key = /<path-to-file id="sl-md0000000">/control-plane.key
   ```

   ノードのロールを `control_plane`に設定すると、このノードはデフォルトではデータプレーン接続をポート `0.0.0.0:8005`でリッスンし、テレメトリデータを`0.0.0.0:8006`でリッスンします。コントロールプレーン上のこれらのポートは、設置されているすべてのファイアウォールを通して、制御対象となるすべてのデータプレーンからアクセスできる必要があります。

   PKIモードの場合、`cluster_ca_cert`は
   `cluster_cert`および`cluster_cert_key`のルートCA証明書を指定します。この証明書はルートCA証明書である必要があり、
   中間CAではありません。Kongでは、ルートCAとクラスタ証明書の間で
   最大3レベルの中間CAを
   使用できます。

   コントロールプレーンがリッスンするポートを変更する必要がある場合は、以下を設定します。

   ```bash
   cluster_listen=0.0.0.0:<port id="sl-md0000000">
   cluster_telemetry_listen=0.0.0.0:<port id="sl-md0000000">
   ```

2. 設定を有効にするためにKongを再起動します。

   ```bash
   kong restart
   ```

{% endnavtab %}
{% endnavtabs %}

コントロールプレーン（CP）には中央構成を
保存するためのデータベースが必要なことに注意してください。ただし、データプレーンノードがデータベースに
アクセスする必要はありません。ロードバランシングと冗長性を提供するには、同じバックエンドデータベースを指していれば、複数のコントロールプレーンノードを実行することができます。

{:.note}
> 
> **注意：** コントロールプレーンノードはプロキシには使用できません。

### （任意）データプレーン証明書の失効チェック

KongがPKIモードとハイブリッドモードで動作している場合、コントロールプレーンは、接続しているデータプレーン証明書の失効ステータスをオプションで確認するように構成できます。

サポートされている方法は、オンライン証明書ステータスプロトコル（OCSP）レスポンダー経由です。
発行されたデータプレーン証明書には、コントロールプレーンからアクセスできるOCSPレスポンダーのURIを参照する、証明機関情報アクセス拡張機能が含まれている必要があります。

OCSPチェックを有効にするには、コントロールプレーンの`cluster_ocsp`構成を次のいずれかの値に設定します。

* `on`: コントロールプレーンレーンとの接続を確立するためには、OCSP失効チェックが有効で、データプレーンが失効チェックに合格する必要があります。これは、OCSP拡張のない証明書や到達不能なOCSPレスポンダも接続の確立を妨げることを意味します。
* `off`: OCSP失効チェックが無効になっています（デフォルト）。
* `optional`: OCSP失効チェックが試行されます。ただし、OCSPレスポンダURIはデータプレーンで提供された証明書内に見つからないか、OCSPレスポンダとの通信に失敗した場合は、データプレーンは引き続き通過が許可されます。断かい

OCSP チェックは、受信データプレーン
ノードによって提供される証明書に対して、コントロールプレーンでのみ実行されることに注意してください。`cluster_ocsp` 設定はデータプレーンノードには影響しません。
`cluster_oscp` は、データプレーンからコントロールプレーンに確立されたすべてのハイブリッドモード接続に影響します。

データプレーン（DP）のインストールと起動
---------------------

コントロールプレーンが稼働しているので、データプレーンノードを接続してトラフィックの処理を開始できます。

このステップでは、すべてのデータプレーンノードに `data_plane`のロールを割り当てて、それらをコントロールプレーンに向け、証明書/キーパラメータを証明書とキーの位置に向け、データベースが無効であることを確実にします。

さらに、`cluster_cert`発行の証明書（`shared`モード）または`cluster_ca_cert`（`pki`モード）が[`lua_ssl_trusted_certificate`](/gateway/{{page.release}}/reference/configuration/#lua_ssl_trusted_certificate)内の信頼できるチェーンに自動的に追加されます。

{:.important}
> 
> **重要:** データプレーンノードは、宣言型構成に似たフォーマットを介してコントロールプレーン（CP）から更新を受け取ります。そのため、 Kong が正常に起動するには、`database` を `off` に設定する必要があります。

データプレーン（DP）ノードが構成を処理する方法の詳細については、
[DPノードの起動シーケンス](#dp-node-start-sequence)を参照してください。

{% navtabs %}
{% navtab Using Docker %}

1. [Dockerインストールドキュメント](/gateway/{{page.release}}/install/docker/)を参照し、次の手順に従ってください。

   1. [{{site.base_gateway}} をダウンロードしてください](/gateway/{{page.release}}/install/docker/)。
   2. [Dockerネットワークを作成します](/gateway/{{page.release}}/install/docker/#install-gateway-in-db-less-mode)。

   {:.warning}
   > 
   > **警告:** このノードでデータベースを起動または作成しないでください。
2. 次の設定でデータプレーン（DP）コンテナを起動します。

   `shared`証明書モードの場合は、次を使用します。

{% capture shared-mode-cp %}
{% navtabs codeblock %}
{% navtab Kong Gateway %}

```bash
docker run -d --name kong-dp --network=kong-net \
-e "KONG_ROLE=data_plane" \
-e "KONG_DATABASE=off" \
-e "KONG_PROXY_LISTEN=0.0.0.0:8000" \
-e "KONG_CLUSTER_CONTROL_PLANE=control-plane.<admin-hostname id="sl-md0000000">.com:8005" \
-e "KONG_CLUSTER_TELEMETRY_ENDPOINT=control-plane.<admin-hostname id="sl-md0000000">.com:8006" \
-e "KONG_CLUSTER_CERT=/<path-to-file id="sl-md0000000">/cluster.crt" \
-e "KONG_CLUSTER_CERT_KEY=/<path-to-file id="sl-md0000000">/cluster.key" \
-e "KONG_CLUSTER_DP_LABELS=deployment:cloud1,region:us-east-1" \
--mount type=bind,source="$(pwd)"/cluster,target=<path-to-keys-and-certs id="sl-md0000000">,readonly \
-p 8000:8000 \
kong/kong-gateway:{{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
docker run -d --name kong-dp --network=kong-net \
-e "KONG_ROLE=data_plane" \
-e "KONG_DATABASE=off" \
-e "KONG_PROXY_LISTEN=0.0.0.0:8000" \
-e "KONG_CLUSTER_CONTROL_PLANE=control-plane.<admin-hostname id="sl-md0000000">.com:8005" \
-e "KONG_CLUSTER_TELEMETRY_ENDPOINT=control-plane.<admin-hostname id="sl-md0000000">.com:8006" \
-e "KONG_CLUSTER_CERT=/<path-to-file id="sl-md0000000">/cluster.crt" \
-e "KONG_CLUSTER_CERT_KEY=/<path-to-file id="sl-md0000000">/cluster.key" \
-e "KONG_CLUSTER_DP_LABELS=deployment:cloud1,region:us-east-1" \
--mount type=bind,source="$(pwd)"/cluster,target=<path-to-keys-and-certs id="sl-md0000000">,readonly \
-p 8000:8000 \
kong:{{page.versions.ce}}
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ shared-mode-cp | indent | replace: " </code>", "</code>" }}

    For `pki` certificate mode, use:

{% capture pki-mode-cp %}
{% navtabs codeblock %}
{% navtab Kong Gateway %}

```bash
docker run -d --name kong-dp --network=kong-net \
-e "KONG_ROLE=data_plane" \
-e "KONG_DATABASE=off" \
-e "KONG_PROXY_LISTEN=0.0.0.0:8000" \
-e "KONG_CLUSTER_CONTROL_PLANE=control-plane.<admin-hostname id="sl-md0000000">.com:8005" \
-e "KONG_CLUSTER_TELEMETRY_ENDPOINT=control-plane.<admin-hostname id="sl-md0000000">.com:8006" \
-e "KONG_CLUSTER_MTLS=pki" \
-e "KONG_CLUSTER_SERVER_NAME=control-plane.kong.yourcorp.tld" \
-e "KONG_CLUSTER_CERT=data-plane.crt" \
-e "KONG_CLUSTER_CERT_KEY=/<path-to-file id="sl-md0000000">/data-plane.crt" \
-e "KONG_CLUSTER_CA_CERT=/<path-to-file id="sl-md0000000">/ca-cert.pem" \
-e "KONG_CLUSTER_DP_LABELS=deployment:cloud1,region:us-east-1" \
--mount type=bind,source="$(pwd)"/cluster,target=<path-to-keys-and-certs id="sl-md0000000">,readonly \
-p 8000:8000 \
kong/kong-gateway:{{page.versions.ee}}
```

{% endnavtab %}
{% navtab Kong Gateway (OSS) %}

```bash
docker run -d --name kong-dp --network=kong-net \
-e "KONG_ROLE=data_plane" \
-e "KONG_DATABASE=off" \
-e "KONG_PROXY_LISTEN=0.0.0.0:8000" \
-e "KONG_CLUSTER_CONTROL_PLANE=control-plane.<admin-hostname id="sl-md0000000">.com:8005" \
-e "KONG_CLUSTER_TELEMETRY_ENDPOINT=control-plane.<admin-hostname id="sl-md0000000">.com:8006" \
-e "KONG_CLUSTER_MTLS=pki" \
-e "KONG_CLUSTER_SERVER_NAME=control-plane.kong.yourcorp.tld" \
-e "KONG_CLUSTER_CERT=data-plane.crt" \
-e "KONG_CLUSTER_CERT_KEY=/<path-to-file id="sl-md0000000">/data-plane.crt" \
-e "KONG_CLUSTER_CA_CERT=/<path-to-file id="sl-md0000000">/ca-cert.pem" \
-e "KONG_CLUSTER_DP_LABELS=deployment:cloud1,region:us-east-1" \
--mount type=bind,source="$(pwd)"/cluster,target=<path-to-keys-and-certs id="sl-md0000000">,readonly \
-p 8000:8000 \
kong:{{page.versions.ce}}
```

{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ pki-mode-cp | indent | replace: " </code>", "</code>" }}

    Where:
    
    `--name` and `--network`
    : The tag of the {{site.base_gateway}} image that you're using, and the Docker network it communicates on.
    
    `KONG_CLUSTER_CONTROL_PLANE`
    : Sets the address and port of the control plane (port `8005` by default).
    
    `KONG_DATABASE`
    : Specifies whether this node connects directly to a database.
    
    `<path-to-file id="sl-md0000000">` and `target=<path-to-keys-and-certs id="sl-md0000000">`
    : Are the same path, pointing to the location of the `cluster.key` and
    `cluster.crt` files.
    
    `KONG_CLUSTER_SERVER_NAME`
    : Specifies the SNI (Server Name Indication
    extension) to use for data plane connections to the control plane through
    TLS. When not set, data plane will use `kong_clustering` as the SNI.
    
    : You can also optionally use `KONG_CLUSTER_TELEMETRY_SERVER_NAME`
      to set a custom SNI for telemetry data. If not set, it defaults to
      `KONG_CLUSTER_SERVER_NAME`.
    
    `KONG_CLUSTER_TELEMETRY_ENDPOINT`
    : Optional setting, needed for telemetry gathering. Not available in open-source deployments.
    
    `KONG_CLUSTER_DP_LABELS`
    : Optional setting, used to configure data plane labels.
    
    You can also choose to encrypt or disable the data plane configuration
    cache with some additional settings:

1. 必要に応じて、同じ設定を使用して後続のデータプレーンを起動します。

{% endnavtab %}
{% navtab Using kong.conf %}

1. [お使いのプラットフォーム](/gateway/{{page.release}}/install/)のドキュメントを確認して、手順 1 と 2 の指示に従ってKong をダウンロード 
{{site.base_gateway}} およびインストール **のみ** を行います。

   {:.note}
   > 
   > **注:** Docker については、上記の **Docker** タブを参照してください。Kubernetesについては、
   > `kong/charts`リポジトリ内の
   > [ハイブリッドモードのドキュメント](https://github.com/Kong/charts/blob/main/charts/kong/README.md#hybrid-mode)を参照してください。

   {:.warning}
   > 
   > このノードでデータベースを開始または作成しないでください。
2. `kong.conf` で次の構成パラメータを設定します。

   `shared`証明書モードの場合は、次を使用します。

   ```bash
   role = data_plane
   database = off
   proxy_listen = 0.0.0.0:8000
   cluster_control_plane = control-plane.<admin-hostname id="sl-md0000000">.com:8005
   cluster_telemetry_endpoint = control-plane.<admin-hostname id="sl-md0000000">.com:8006
   cluster_cert = /<path-to-file id="sl-md0000000">/cluster.crt
   cluster_cert_key = /<path-to-file id="sl-md0000000">/cluster.key
   cluster_dp_labels = deployment:cloud1,region:us-east-1
   ```

   `pki`証明書モードの場合は、次を使用します。

   ```bash
   role = data_plane
   database = off
   proxy_listen = 0.0.0.0:8000
   cluster_control_plane = control-plane.<admin-hostname id="sl-md0000000">.com:8005
   cluster_telemetry_endpoint = control-plane.<admin-hostname id="sl-md0000000">.com:8006
   cluster_mtls = pki
   cluster_server_name = control-plane.kong.yourcorp.tld
   cluster_cert = /<path-to-file id="sl-md0000000">/data-plane.crt
   cluster_cert_key = /<path-to-file id="sl-md0000000">/data-plane.crt
   cluster_ca_cert = /<path-to-file id="sl-md0000000">/ca-cert.pem
   cluster_dp_labels = deployment:cloud1,region:us-east-1
   ```

   設定は以下のとおりです。

   `cluster_control_plane`
   : コントロールプレーンのアドレスとポートを設定します（デフォルトではポート`8005`）。

   `database`
   : このノードがデータベースに直接接続するかどうかを指定します。

   `<path-to-file id="sl-md0000000">`
   : `cluster.key` と `cluster.crt` のファイルの場所を指定します。

   `cluster_server_name`
   : SNI（Server Name Indication extension）を指定し、TLS を介してコントロールプレーンへのデータプレーン接続に使用します。設定されていない場合、
   データプレーンはSNIとして`kong_clustering`を使用します。

   ：オプションで`cluster_telemetry_server_name`を使用して、
   テレメトリデータのカスタムSNIを設定することもできます。設定されていない場合、デフォルトは 
   `cluster_server_name`です。

   `cluster_telemetry_endpoint`: テレメトリの収集に必要なオプションの設定です。オープンソースのデプロイメントでは使用できません。

   `cluster_dp_labels`: データプレーンラベルの設定に使用されるオプション設定です。

   さらに、いくつか設定を追加して、データプレーン（DP）構成キャッシュを暗号化または無効にすることもできます。
3. 設定を有効にするためにKongを再起動します。

   ```bash
   kong restart
   ```

{% endnavtab %}
{% endnavtabs %}

ノードが接続されていることを確認する
------------------

コントロールプレーンの[Cluster Status API](/gateway/api/admin-ee/latest/#/clustering) を使用してデータプレーンを監視します。以下の内容が提供されます。

* ノードの名前
* ノードが最後にコントロールプレーンと同期した時刻
* 各データプレーン（DP）上で現在実行されている構成のバージョン

先ほど起動したCPノードとDPノードが接続されているかどうかを確認するには、
コントロールプレーンで次のコマンドを実行します。

```bash
curl -i -X GET http://localhost:8001/clustering/data-planes
```

出力には、クラスタ内の接続されているすべてのデータプレーン（DP）インスタンスが表示されます。

{% if_version gte:3.6.x %}

```json
{
    "data": [
        {
            "ip": "172.24.0.10",
            "updated_at": 1689266492,
            "config_hash": "595214af5fb356cc569313184c64d9b7",
            "sync_status": "normal",
            "version": "3.3.0.0",
            "id": "10424658-f139-476c-b74f-e0a6e6ec9402",
            "hostname": "kongDP1",
            "ttl": 1209593,
            "last_seen": 1689266492,
            "labels": {
                "deployment": "cloud1",
                "region": "us-east-1"
            },
            "cert_details": {
                "expiry_timestamp": 1897136778,
            }
        },
        {
            "ip": "172.24.0.11",
            "updated_at": 1689266472,
            "config_hash": "595214af5fb356cc569313184c64d9b7",
            "sync_status": "normal",
            "version": "3.3.0.0",
            "id": "3487f520-4f52-4ee5-ad6f-50756822f0c5",
            "hostname": "kongDP2",
            "ttl": 1209572,
            "last_seen": 1689266472,
            "labels": {
                "deployment": "cloud2",
                "region": "us-west-2"
            },
            "cert_details": {
                "expiry_timestamp": 1897136778,
            }
        }
    ],
    "next": null
}
```

{% endif_version %}

{% if_version eq:3.5.x %}

```json
{
    "data": [
        {
            "ip": "172.24.0.10",
            "updated_at": 1689266492,
            "config_hash": "595214af5fb356cc569313184c64d9b7",
            "sync_status": "normal",
            "version": "3.3.0.0",
            "id": "10424658-f139-476c-b74f-e0a6e6ec9402",
            "hostname": "kongDP1",
            "ttl": 1209593,
            "last_seen": 1689266492,
            "labels": {
                "deployment": "cloud1",
                "region": "us-east-1"
            }
        },
        {
            "ip": "172.24.0.11",
            "updated_at": 1689266472,
            "config_hash": "595214af5fb356cc569313184c64d9b7",
            "sync_status": "normal",
            "version": "3.3.0.0",
            "id": "3487f520-4f52-4ee5-ad6f-50756822f0c5",
            "hostname": "kongDP2",
            "ttl": 1209572,
            "last_seen": 1689266472,
            "labels": {
                "deployment": "cloud2",
                "region": "us-west-2"
            }
        }
    ],
    "next": null
}
```

{% endif_version %}

{% if_version gte:3.3.x lte:3.4.x %}

```json
{
    "data": [
        {
            "ip": "172.24.0.10",
            "updated_at": 1689266492,
            "config_hash": "595214af5fb356cc569313184c64d9b7",
            "sync_status": "normal",
            "version": "3.3.0.0",
            "id": "10424658-f139-476c-b74f-e0a6e6ec9402",
            "hostname": "kongDP1",
            "ttl": 1209593,
            "last_seen": 1689266492
        },
        {
            "ip": "172.24.0.11",
            "updated_at": 1689266472,
            "config_hash": "595214af5fb356cc569313184c64d9b7",
            "sync_status": "normal",
            "version": "3.3.0.0",
            "id": "3487f520-4f52-4ee5-ad6f-50756822f0c5",
            "hostname": "kongDP2",
            "ttl": 1209572,
            "last_seen": 1689266472
        }
    ],
    "next": null
}
```

{% endif_version %}
{% if_version lte:3.2.x %}

    {
        "data": [
            {
                "config_hash": "a9a166c59873245db8f1a747ba9a80a7",
                "hostname": "data-plane-2",
                "id": "ed58ac85-dba6-4946-999d-e8b5071607d4",
                "ip": "192.168.10.3",
                "last_seen": 1580623199,
                "status": "connected"
            },
            {
                "config_hash": "a9a166c59873245db8f1a747ba9a80a7",
                "hostname": "data-plane-1",
                "id": "ed58ac85-dba6-4946-999d-e8b5071607d4",
                "ip": "192.168.10.4",
                "last_seen": 1580623200,
                "status": "connected"
            }

{% endif_version %}

リファレンス
------

### DPノードの起動シーケンス

DPノードとして設定すると、{{site.base_gateway}}は次の順序で構成を処理します。

1. **構成キャッシュ** ：ローカル設定キャッシュ`dbless.lmdb`が`kong_prefix`パスに存在する場合 （デフォルトは`/usr/local/kong`）、DPノードはそれを構成としてロードします。
2. **`declarative_config` 存在する** : 構成キャッシュがなく、`declarative_config` パラメータが設定されている場合、DPノードは指定されたファイルをロードします。
3. **空の構成:** 利用可能な構成キャッシュまたは宣言型構成ファイルが無い場合、ノードは空の構成で開始します。この状態の場合、すべてのリクエストに対して 404 が返されます。
4. **CPノードへの接続** ：すべての場合において、DPノードはCPノードに接続して最新の構成を取得します。 成功すると、ローカル構成キャッシュ（`dbless.lmdb`）に保存されます。

### 構成リファレンス

ハイブリッドモードで{{site.base_gateway}}を構成するには、次の構成プロパティを使用します。

|                                                                                         パラメータ                                                                                         |                                                                                                     説明                                                                                                      | CPまたはDP {:width=10%:} |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------|
| [`role`](/gateway/{{page.release}}/reference/configuration/#role)<br> *必須*                                                                                                                      | {{site.base_gateway}}インスタンスがコントロールプレーン（CP）かデータプレーン（DP）かを決定します。有効な値は`control_plane`または`data_plane`です。                                                                                                                       | 両方             |
| [`cluster_listen`](/gateway/{{page.release}}/reference/configuration/#cluster_listen) <br> *オプション* <br><br> **デフォルト:** `0.0.0.0:8005`                                                           | コントロールプレーンが着信データプレーン接続をリッスンするアドレスとポートのリスト。このポートは常にTLS相互認証（mTLS）暗号化で保護されます。データプレーンノードでは無視されます。                                                                                                               | CP             |
| [`proxy_listen`](/gateway/{{page.release}}/reference/configuration/#proxy_listen)<br> *必須*                                                                                                      | プロキシサーバーがHTTP/HTTPSトラフィックをリッスンするアドレスとポートをコンマで区切ったリスト。コントロールプレーン（CP）ノードでは無視されます。                                                                                                                             | DP             |
| [`cluster_telemetry_listen`](/gateway/{{page.release}}/reference/configuration/#cluster_telemetry_listen) <span class="badge enterprise" > <br> *Optional* <br><br> **Default:** `0.0.0.0:8006` | コントロールプレーン（CP）がデータプレーン（DP）のテレメトリデータををリッスンするアドレスとポートのリスト。このポートは常にTLS相互認証（mTLS）暗号化で保護されます。データプレーンノードでは無視されます。                                                                                                 | CP             |
| [`cluster_telemetry_endpoint`](/gateway/{{page.release}}/reference/configuration/#cluster_telemetry_endpoint) <span class="badge enterprise" > <br> *Required for Enterprise deployments*       | データプレーン（DP）がテレメトリデータをコントロールプレーン（CP）に送信するために使用するポート。コントロールプレーンノードでは無視されます。                                                                                                                                   | DP             |
| [`cluster_control_plane`](/gateway/{{page.release}}/reference/configuration/#cluster_control_plane)<br> *必須*                                                                                    | データプレーン（DP）ノードがコントロールプレーン（CP）に接続するために使用するアドレスとポート。コントロールプレーン（CP）ノード上の[`cluster_listen`](/gateway/{{page.release}}/reference/configuration/#cluster_listen) プロパティを使用して構成されたポートを指している必要があります。コントロールプレーン（CP）ノードでは無視されます。 | DP             |
| [`cluster_mtls`](/gateway/{{page.release}}/reference/configuration/#cluster_mtls) <br> *オプション* <br><br> **デフォルト:** `shared`                                                                     | `shared`または`pki`のいずれか。ハイブリッドモードでCP/DP mTLSに共有証明書/キーペアを使用するか、PKIモードを使用するかを示します。mTLSモードの違いについては、以下のセクションを参照してください。                                                                                            | 両方             |

以下のプロパティは、`shared` モードと `pki` モードで使用方法が異なります。

|                                                                             パラメータ                                                                             |                                            説明                                             |  共有モード{:width=12%:}   |                                              PKIモード {:width=30%:}                                              |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|----------------|---------------------------------------------------------------------------------------------------------|
| [`cluster_cert`](/gateway/{{page.release}}/reference/configuration/#cluster_cert)と[`cluster_cert_key`](/gateway/{{page.release}}/reference/configuration/#cluster_cert_key)は *必須* | CP/DPノード間の mTLS に使用される証明書/キーペア。                                                           | CP/DPノード間でも同様。 | `cluster_ca_cert` によって指定された CA から生成された、各ノードの一意の証明書。                                                     |
| [`cluster_ca_cert`](/gateway/{{page.release}}/reference/configuration/#cluster_ca_cert)<br> *PKI モードで必須*                                                                | `cluster_cert` の検証に使用される PEM 形式の信頼されたCA証明書ファイル。                                           | *無視*           | CP/DP ノード間で同じの、`cluster_cert` の検証に使用される CA 証明書。 *必須*                                                    |
| [`cluster_server_name`](/gateway/{{page.release}}/reference/configuration/#cluster_server_name)<br> *PKI モードで必須*                                                        | DPノードmTLSハンドシェイクによって提示されるSNI。                                                             | *無視*           | PKIモードでDPノードは、CPによって提示された証明書内のコモンネーム（CN）またはサブジェクト別名（SAN）が`cluster_server_name`値と一致することも確認します。           |
| [`cluster_telemetry_server_name`](/gateway/{{page.release}}/reference/configuration/#cluster_telemetry_server_name) <span class="badge enterprise" >                    | DPノードの mTLS ハンドシェイクによって提示されるテレメトリ SNI。指定されていない場合は、`cluster_server_name` に設定されたSNIが使用されます。 | *無視*           | PKIモードでDPノードは、CPによって提示された証明書内のコモンネーム（CN）またはサブジェクト別名（SAN）が`cluster_telemetry_server_name`値と一致することも確認します。 |

次の手順
----

これで、コントロールプレーンを使用してクラスタの管理を開始できます。一度
すべてのインスタンスが設定されたら、コントロールプレーンでAdmin APIを通常どおり使用します。
これらの変更は、データプレーンノードで数秒以内に自動的に同期および
更新されます。

