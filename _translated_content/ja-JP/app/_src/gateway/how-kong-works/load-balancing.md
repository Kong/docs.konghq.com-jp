---
title: "ロードバランシングリファレンス"
---
Kongが複数のバックエンドサービスに対するリクエストをロードバランシングする方法は、デフォルトであるDNSベースの方法と、アップストリームエンティティを使用する高度なロードバランシングアルゴリズムセットと、複数あります。

DNSロードバランサーはデフォルトで有効になっていますが、ラウンドロビン方式のロードバランシングに限定されます。
`upstream`エンティティには、最小接続数などのより高度なアルゴリズム、コンシステントハッシング、最小レイテンシに加え、ヘルスチェックとサーキットブレーカーがあります。

Refer to the [DNS caveats](#dns-caveats) depending on your infrastructure .

DNSベースのロードバランシング
----------------

名前が複数のIPアドレスを解決する場合、（IPアドレスではなく）ホスト名が含まれた`host`で定義されたすべてのサービスは、DNSベースのロードバランシングを自動的に使用します。

DNSレコード`ttl`設定（有効期間）によって、情報が更新される頻度が決まります。`ttl`を0にすると、すべてのリクエストは独自のDNSクエリを使用して解決されます。もちろん、パフォーマンスの低下は生じますが、更新/変更のレイテンシは非常に低くなります。

重みづけの有無にかかわらず、ホスト名のDNSレコードタイプに応じてラウンドロビンアルゴリズムが使用されます。

### A records

Aレコードには1つ以上のIPアドレスが含まれます。したがって、ホスト名がAレコードに解決される場合、各バックエンドサービスには独自のIPアドレスが必要です。

`weight`情報がないため、すべてのエントリはロードバランサーで均等に重み付けされたものとして扱われ、バランサーは単純なラウンドロビン方式で処理を実行します。

### SRV レコード

SRVレコードには、すべてのIPアドレスの重みとポート情報が含まれています。バックエンドサービスはIPアドレスとポート番号の一意の組み合わせによって識別されます。したがって、1つのIPアドレスで、同じサービスの複数のインスタンスを異なるポートでホストできます。

SRVレコードは`priority`プロパティも特徴です。Kongは優先順位の最も高いエントリのみを
使用し、その他すべてを無視します（SRVレコード内の『優先順位の最も高い』ものは、
実際は最も低い`priority`の値を持つレコードであることに注意してください）。

`weight`情報が利用可能なため、各エントリはロードバランサーで独自の重みを持ち、加重ラウンドロビンを実行します。

同様に、指定されたポート情報は、DNSサーバーからのポート情報によって上書きされます。サービスに属性 `host=myhost.com` と `port=123` があり、`myhost.com` が `127.0.0.1:456` のSRVレコードに解決される場合、ポート `123` が `456` で上書きされるため、リクエストは `http://127.0.0.1:456/somepath` にプロキシされます。

### DNSの注意事項

* Kong はネームサーバーを信頼します。これは、DNSクエリ経由で取得された情報が、設定された値よりも優先されることを意味します。これは主に、`port` と `weight` の情報を保持する SRV レコードに関連します。

* Whenever the DNS record is refreshed a list is generated to handle the
  weighting properly. Try to keep the weights as multiples of each other to keep
  the algorithm performant, e.g., 2 weights of 17 and 31 would result in a structure
  with 527 entries, whereas weights 16 and 32 \(or their smallest relative
  counterparts 1 and 2\) would result in a structure with merely 3 entries. This is
  especially relevant with a very small \(or even 0\) `ttl` value.

* DNS は UDP を通して伝送され、デフォルトの制限は 512 バイトです。返されるエントリが多い場合、DNSサーバーは部分的なデータで応答し、未送信のエントリが他にまだあることを示すトランケートフラグを設定します。Kong を含む DNS クライアントは、エントリの完全なリストを取得するために TCP 上で 2 回目のリクエストを送信します。

* Some nameservers by default do not respond with the truncate flag, but trim the response
  to be under 512 byte UDP size.

  * Consulはその一例です。Consulは、デフォルトの設定では、最初の3つのエントリのみを返し、未送信のエントリが残っていることを示すtruncateフラグを設定しません。Consulには切り捨てフラグを有効にするオプションが含まれています。詳細については、[Consulのドキュメント](https://www.consul.io/docs/agent/options.html#enable_truncate)を参照してください。

* デプロイされたネームサーバーがトランケートフラグを提供していない場合、アップストリームインスタンスのプールはロードに一貫性がない可能性があります。Kongノードはネームサーバーから提供される情報が限られているため、一部のインスタンスを認識していません。これを緩和するには、別のネームサーバーを使用するか、名前の代わりにIPアドレスを使用するか、
  または、使用中のすべてのアップストリームサービスを利用し続けられる余裕のあるKongノードを使用します。

* ネームサーバーが`3 name error`を返した場合、それはKongの有効な応答です。これが予期せぬことである場合、まず正しい名前がクエリされているかどうかを確認し、次にネームサーバーの設定を確認します。

* DNSレコード（AまたはSRV）からのIPアドレスの最初の選択はランダム化されません。したがって、`ttl`が0のレコードを使用する場合、ネームサーバーはレコードエントリをランダム化することが期待されます。

高度なロードバランシング
------------

高度なロードバランシングアルゴリズムは、 `upstream`エンティティを通じて利用できます。

これらのロードバランサーを使用すると、バックエンドサービスの追加と削除は
Kong によって処理され、DNSの更新は必要ありません。Kongはサービスレジストリとして機能します。

ロードバランサーの構成は、 `upstream`エンティティと`target`エンティティを通じて行われます。

* `upstream`：サービス`host`フィールドで使用される'virtual hostname'（例：`weather.v2.service`という`upstream`）は`host=weather.v2.service`の`service`からすべてのリクエストを取得します。`upstream`は、ロードバランシングの動作（およびヘルスチェックとサーキットブレーカー構成として）を決定するプロパティを実行します。

* `target`: バックエンドサービスが存在するポート番号付きのIPアドレスまたはホスト名。
  例：「192\.168\.100\.12:80」など）`target`ごとに、取得する相対的な負荷を示す
  `weight`が追加されます。IPアドレスは、
  IPv4とIPv6の両方の形式で指定できます。

### アップストリーム

Each `upstream` can have many `target` entries attached to it, and requests proxied
to the 'virtual hostname' will be load balanced over the targets.

ターゲットの追加と削除はAdmin APIで単純なHTTPリクエストを使用して実行できます。
この操作は比較的低コストですが、アップストリームの変更自体は、たとえばスロット数が変更する場合はバランサーを再構築する必要があるため、コストが高くなります。

アップストリーム追加と操作に関する詳細情報は、
[Admin APIリファレンス](/gateway/{{page.release}}/admin-api#upstream-object)の `upstream` セクションをご覧ください。

### Target

ターゲットは、バックエンドサービスのインスタンスを識別するポート付きのIPアドレス/ホスト名です。各アップストリームは多くのターゲットを持つことができます。ターゲットの追加と操作に関する詳細な情報は、
[Admin APIリファレンス](/gateway/{{page.release}}/admin-api#target-object)の`target`セクションを参照してください。

非アクティブなエントリーがアクティブなエントリーの10倍以上になると、ターゲットは自動的にクリーンアップされます。クリーニングにはバランサーの再構築が含まれます。
したがって、単にターゲットエントリを追加するよりもコストがかかります。

`target`にはIPアドレスの代わりにホスト名を指定することもできます。その場合は、
名前が解決され、見つかったすべてのエントリは個別にリングバランサーに追加されます。例えば `api.host.com:123`に `weight=100`を加えるなどです。「api.host.com」という名前が2つのIPアドレスを持つAレコードに解決されます。そして両方の
IPアドレスがターゲットとして追加され、それぞれが`weight=100`とポート123を取得します。
**注** ：重みは全体ではなく、各エントリに対して使用されます。

SRVレコードに解決され、さらにDNSレコードからの `port`フィールドと`weight`フィールドも取得され、
指定されたポート`123`と`weight=100`が
上書きされますか？
{:.note}
> 
> **注** ：DNSベースのロードバランシングと同様に、SRVレコード内の最も
> 優先順位の高いエントリ（最も低い値）のみが使用されます。

バランサーはDNSレコードの`ttl`設定を尊重し、有効期限が切れるとネームサーバーにクエリを実行し、バランサーを更新します。

{:.important}
> 
> **例外** ：DNSレコードに`ttl=0`がある場合、ホスト名は指定された重みを持つ単一のターゲットとして追加されます。プロキシリクエストごとに
> このターゲットに対して、ネームサーバーを再度照会します。

### ロードバランサーとの競合

[ターゲット](#target)の段落で説明したように、ターゲットはホスト名として指定できます。
k8s や docker\-compose などのオーケストレーションされた環境では、IP アドレスとポートはほとんどが一時的であり、SRV レコードを使用して適切なバックエンドを見つけて最新の状態を維持する必要があります。

DNS レベルで、多くのインフラツールがロードバランシングタイプの機能も提供できます。
これらのほとんどは、独自のヘルスチェックを持ち、DNSレコードをランダム化するか、あるいは利用可能なピアの小規模なサブセットのみを返すサービスディスカバリツールです。

Kong ロードバランサーと DNS ベースのツールは、競合することがよくあります。ネームサーバーは、クライアントをそのスキームに強制的に従わせるように、最小限の情報しか提供しませんが、その一方で Kong はすべてのバックエンドにロードバランサーとヘルスチェックを適切に設定しようとします。

ご使用の環境で、次を確認してください。

* すべてのレコードがUDP応答に収まらない場合、ネームサーバーは応答に切り捨てフラグを設定します。これにより、KongはTCPを使用した再試行が強制されます。
* TCPクエリはネームサーバーで許可されます。

バランシングアルゴリズム
------------

ロードバランサーは以下のロードバランシングアルゴリズムをサポートします。

* `round-robin`
* `consistent-hashing`
* `least-connections`
{% if_version gte:3.2.x -%}
* `latency` {% endif_version %}

これらのアルゴリズムは、`upstream`エンティティを使用する場合にのみ利用可能です。[高度ロードバランシング](#advanced-load-balancing)を参照してください。

{:.note}
> 
> **注** : これらのアルゴリズムすべてにとって、個々のバックエンドの重みとポートがどのように設定されているかを理解することは重要です。実際の重みとポートがユーザー構成と DNS の結果に基づいて決定される方法については、[ターゲット](#target)の段落を参照してください。

### ラウンドロビン

ラウンドロビンアルゴリズムは加重方式で行われます。DNSベースのロードバランシングと結果は同一になりますが、`upstream`であるため、このケースではヘルスチェックとサーキットブレーカー向けの追加機能を利用できます。

このアルゴリズムを選択する際には、以下を考慮してください。

* リクエストの適切な分布。
* 比較的静的であること（トラフィックの分散に影響を与えるのはDNSの更新または `target` の更新だけであるため）。
* キャッシュヒット率は向上しません。

### コンシステントハッシュ

コンシステントハッシュ アルゴリズムでは、構成可能なクライアント入力を使用してハッシュ値を計算します。このハッシュ値は、特定のバックエンドサーバーに関連付けられます。

一般的な例としては、 `consumer`をハッシュ入力として使用する方法です。このIDは、そのユーザーによるすべてのリクエストに対して同じであるため、同じユーザーが一貫して同じバックエンドサーバーによって処理されることを確実にします。これにより、各サーバーは固定されたユーザーのサブセットのみを処理するためバックエンドでのキャッシュ最適化が可能になり、ユーザーに関連するデータのキャッシュヒット率が向上します。

このアルゴリズムは[ketama原理](https://github.com/RJ/ketama)を実装し、
ハッシュの安定性を最大化して、既知のバックエンドのリストに対する変更による一貫性の損失を最小限に抑えます。

`consistent-hashing`アルゴリズムを使用する場合、ハッシュ入力は`none`、`consumer`、`ip`、`header`、`cookie`のいずれかになります。`none`に設定すると、
`round-robin`スキームが使用され、ハッシュは無効になります。`consistent-hashing`アルゴリズムはプライマリとフォールバックのハッシュ属性をサポートします。プライマリ属性が失敗した場合（例：プライマリが`consumer`に設定され、コンシューマが認証されていない場合）、フォールバック属性が使用されます。これにより、アップストリームキャッシュヒットが最大化されます。

サポートされているハッシュ属性は次のとおりです。

* `none`: `consistent-hashing`を使用せず、代わりに`round-robin`を使用してください \(デフォルト\)。
* `consumer`: Use the Consumer ID as the hash input. If no Consumer ID is available, it will fall back on the Credential ID \(for example, in case of an external authentication mechanism like LDAP\).
* `ip`：発信元IPアドレスをハッシュ入力として使用します。これを使用するときは、 [実際のIPを決定する](/gateway/{{page.release}}/reference/configuration/#real_ip_header)ための構成の設定を確認してください。
* `header`：指定したヘッダーをハッシュ入力として使用します。ヘッダー名は `hash_on_header`または`hash_fallback_header`で指定します。これは、`header`がそれぞれプライマリ属性かフォールバック属性かによります。
* `cookie`: Use a specified cookie with a specified path as the hash input. The cookie name is specified in the `hash_on_cookie` field and the path is specified in the `hash_on_cookie_path` field. If the specified cookie is not present in the request, it will be set by the response. Hence, the `hash_fallback` setting is invalid if `cookie` is the primary hashing mechanism. The generated cookie will have a random UUID value. So the first assignment will be random, but then sticks because it is preserved in the cookie.

コンシステントハッシュバランサーは、単一ノードとの併用およびクラスタ内の両方で機能するように設計されています。
ハッシュベースのアルゴリズムを使用する場合、すべてのノードでまったく同じバランサーレイアウトを構築して、同一機能を確保することが重要です。そのためには、バランサーは予測可能な方法で構築される必要があります。

このアルゴリズムを選択する際には、以下を考慮してください。

* バックエンドのキャッシュヒット率を改善します。
* 均等に分配するために、ハッシュ入力に十分なカーディナリティが必要です（たとえば、可能な値が2つしかないヘッダーでハッシュ化することは意味がありません\)。
* cookieベースのアプローチはブラウザベースのリクエストには有効に機能しますが、うまく機能しますが、マシンツーマシン（M2M）クライアントではcookieを省略することが多いため、その精度は低下します。 
* バランサーのホスト名を次のように使用しないでください。 DNS TTLの精度は2秒しかないため、バランサーはゆっくりと分岐する可能性があるためです。 また、更新は、名前が実際にリクエストされたときに決定されます。これに加えて 一部のネームサーバーがすべてのエントリを返さないという問題は、この問題を複雑化させます。 したがって、Kongクラスタでハッシュアプローチを使用する場合は、 IP アドレス別の `target` エンティティをできれば追加してください。この問題はバランサ の再構築とより高いttl設定によって軽減できます。

### 最小接続

このアルゴリズムは、各バックエンドで処理中のリクエストの数を追跡します。重みは、バックエンドの「接続容量」を計算するために使用されます。リクエストは空き容量が最も大きいバックエンドにルーティングされます。

このアルゴリズムを選択する際には、以下を考慮してください。

* トラフィックが良好に分散されていること。
* キャッシュヒット率は向上しません。
* バックエンドが遅いほど、より多くの接続が開かれるため、よりダイナミックになります。したがって、 新しいリクエストは自動的に他のバックエンドにルーティングされます。

{% if_version gte:3.2.x %}

### レイテンシ

`latency`アルゴリズムはピーク時のEWMA（指数加重移動平均）に基づいています。
これは、バランサーが最も低いレイテンシ（`upstream_response_time`）でバックエンドを
選択することを確実にするものです。使用されているレイテンシメトリクスは、TCP接続から
本文の応答時間までのリクエストサイクル全体です。これは移動平均なので、メトリクスは時間の経過とともに
「衰えて」いきます。

ウェイトは考慮されません。

このアルゴリズムを選択する際には、以下を考慮してください。

* ここで提供されるトラフィックの分散は良好です。ただし、「減退」しているメトリクスを生かすのに十分なベースロードです。
* Websocketやサーバー送信イベント（SSE）のような長時間接続には適していません。
* 常に最適化されるので、非常に動的です。
* 理想的には、これはレイテンシの変動が低い場合に最適に機能します。これは、ほとんどが同様の形状のトラフィックを意味し、バックエンドのワークロードも同様です。たとえば、GraphQLバックエンドが小さくて速いクエリと大きくて遅いクエリを提供すると、レイテンシメトリックの変動が大きくなり、メトリックが歪んでしまいます。
* これを防ぐために、バックエンド容量を適切に設定し、適切なネットワークを確保して、リソースの枯渇を防ぎます。たとえば、2台のサーバーを使用します。1台は近くの小さな容量（低遅延）、もう1つは、遠くの大容量（高遅延）にします。ほとんどのトラフィックは、レイテンシが増加し始めるまで、小さなトラフィックにルーティングされます。ただし、レイテンシが増えるということは、小規模なサーバーがリソース不足に陥っている可能性が高いということを意味します。つまり、この場合、アルゴリズムは小規模なサーバーをリソース不足の状態に保ちますが、これはおそらく効率的ではありません。

{% endif_version %}
