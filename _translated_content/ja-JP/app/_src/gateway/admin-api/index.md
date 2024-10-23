---
title: "Admin API"
toc: false
disable_image_expand: true
service_body: "属性名｜説明\n---:| ---\nname<br>*任意* | サービス名。\nretries<br>*任意* | プロキシに失敗したときに実行する再試行回数。デフォルト: 5\nprotocol：アップストリーム（アプリケーション）との通信に使用されるプロトコル。指定できる値は、grpc、grpcs、http、https、tcp、tls、tls_passthrough、udp、ws<span class=\"badge enterprise\"></span>、wss<span class=\"badge enterprise\"></span>。デフォルトは、http。\nhost | アップストリームサーバーのホスト。ホスト値では大文字と小文字が区別されることにご注意ください。\nport | アップストリームサーバポート。デフォルト：80\npath<br>*任意* | アップストリームサーバーへのリクエストに使用されるパス。\nconnect_timeout<br>*任意* | アップストリームサーバーとの接続を確立するためのタイムアウト (ミリ秒単位)。デフォルト: 60000。write_timeout<br>*任意* | アップストリームサーバーにリクエストを送信するための、2つの連続した書き込み操作の間のタイムアウト（ミリ秒）。デフォルト: 60000。read_timeout<br>*任意*｜アップストリームサーバーにリクエストを送信し、返ってくるまでの待機時間（ミリ秒単位）。デフォルト: 60000。tags<br>*任意*｜グループ化とフィルタリングのためにサービスに関連付けられた文字列の任意セットです。\nclient_certificate<br>*任意* | アップストリームサーバーとの TLS ハンドシェイク時にクライアント証明書として使用する証明書。form-encodedでは、client_certificate.id=<client_certificate id>。JSONでは、'&quot;client_certificate&quot;:{&quot;id&quot;:&quot;<client_certificate id>&quot;}を使用します。\ntls_verify<br>*任意* | アップストリームサーバーのTLS証明書の検証を有効にするかどうか。nullに設定すると、Nginxのデフォルトが遵守されます。\ntls_verify_depth<br>*任意* | アップストリームサーバーのTLS証明書を確認する際のチェーンの最大深度。nullに設定すると、Nginx のデフォルトが尊重されます。デフォルト: null。\nca_certificates<br>*任意* | アップストリームサーバーのTLS証明書を検証しながらトラストストアを構築するために使用されるCA 証明書オブジェクト UUID の配列。Nginxのデフォルトが尊重されるときに「null」に設定するとします。NginxのデフォルトのCAリストが指定されておらず、TLS検証が有効になっている場合、アップストリームサーバーとのハンドシェイクは常に失敗します (信頼できるCAがないため)。 form-encodedでは、表記はca_certificates[]=4e3ad2e4-0bc4-4638-8e34-c84a417ba39b&amp;ca_certificates[]=51e77dc2-8f3e-4afa-9d0e-0e3bbbcfd515です。JSONでは、配列を使用します。\nenabled | サービスがアクティブかどうか。falseに設定すると、プロキシは、アタッチされたルートが存在しないかのように動作します (404)。デフォルト: true。\nurl<br>*短縮形属性* | protocol、host、port、pathを一度に設定するための短縮形属性。この属性は書き込み専用です(Admin APIはURLを返しません)。"
service_json: "{\n    \"id\": \"9748f662-7711-4a90-8186-dc02f10eb0f5\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-service\",\n    \"retries\": 5,\n    \"protocol\": \"http\",\n    \"host\": \"example.com\",\n    \"port\": 80,\n    \"path\": \"/some_api\",\n    \"connect_timeout\": 60000,\n    \"write_timeout\": 60000,\n    \"read_timeout\": 60000,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"client_certificate\": {\"id\":\"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b\"},\n    \"tls_verify\": true,\n    \"tls_verify_depth\": null,\n    \"ca_certificates\": [\"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b\", \"51e77dc2-8f3e-4afa-9d0e-0e3bbbcfd515\"],\n    \"enabled\": true\n}\n"
service_data: "\"data\": [{\n    \"id\": \"a5fb8d9b-a99d-40e9-9d35-72d42a62d83a\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-service\",\n    \"retries\": 5,\n    \"protocol\": \"http\",\n    \"host\": \"example.com\",\n    \"port\": 80,\n    \"path\": \"/some_api\",\n    \"connect_timeout\": 60000,\n    \"write_timeout\": 60000,\n    \"read_timeout\": 60000,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"client_certificate\": {\"id\":\"51e77dc2-8f3e-4afa-9d0e-0e3bbbcfd515\"},\n    \"tls_verify\": true,\n    \"tls_verify_depth\": null,\n    \"ca_certificates\": [\"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b\", \"51e77dc2-8f3e-4afa-9d0e-0e3bbbcfd515\"],\n    \"enabled\": true\n}, {\n    \"id\": \"fc73f2af-890d-4f9b-8363-af8945001f7f\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-service\",\n    \"retries\": 5,\n    \"protocol\": \"http\",\n    \"host\": \"example.com\",\n    \"port\": 80,\n    \"path\": \"/another_api\",\n    \"connect_timeout\": 60000,\n    \"write_timeout\": 60000,\n    \"read_timeout\": 60000,\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"],\n    \"client_certificate\": {\"id\":\"4506673d-c825-444c-a25b-602e3c2ec16e\"},\n    \"tls_verify\": true,\n    \"tls_verify_depth\": null,\n    \"ca_certificates\": [\"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b\", \"51e77dc2-8f3e-4afa-9d0e-0e3bbbcfd515\"],\n    \"enabled\": true\n}],\n"
route_body: "属性名｜説明\n---:| ---\nname<br>*ルート名。ルート名は一意である必要があり、大文字と小文字が区別されます。 たとえば、「test」と「Test」という名前の 2 つの異なるルートが存在する場合があります。\nprotocols | このルートが許可するプロトコルの配列受け入れられるプロトコルのリストについては、[ルートオブジェクト](#route-object) セクションをご確認ください。httpsのみに設定すると、HTTP リクエストはアップグレードエラーで応答されます。http のみに設定されている場合、HTTP リクエストと HTTPS リクエストの両方が許可されるという点で、これは基本的に [&quot;http&quot;, &quot;https&quot;] と同じです。デフォルト: [http、https]\nmethods<br>*セミオプション*｜このRouteにマッチするHTTPメソッドのリスト。\nhosts<br>*セミオプション*｜このRouteにマッチするドメイン名のリスト。ホストの値は大文字と小文字が区別されることにご注意ください。フォームエンコードの場合、表記は hosts[]=example.com&amp;hosts[]=foo.test になります。JSON では、配列を使用します。paths<br>*セミオプション*｜このルートにマッチするパスのリスト。form-encodedの場合、paths[]=/foo&amp;paths[]=/barとなります。JSONでは、配列を使用します。パスは正規表現またはプレーンテキストパターンにすることができます。 パスパターンは正規化されたパスと照合され、ほとんどのパーセントエンコードされた文字がデコード、パスフォールディングされ、セマンティクスが保持されます。詳細については、[rfc3986](https://datatracker.ietf.org/doc/html/rfc3986#section-6)をご参照ください。\nheaders<br>*セミオプション* | リクエストにこのルートが存在する場合に一致させる、ヘッダー名で索引付けされた1つ以上の値のリスト。Hostヘッダーはこの属性では使用できません。ホストはhosts属性を使用して指定する必要があります。headersに値が 1 つだけ含まれており、その値が特殊なプレフィックス~*で始まる場合、その値は正規表現として解釈されます。\nhttps_redirect_status_code | プロトコルを除くルートのすべてのプロパティが一致する場合、つまり、リクエストのプロトコルが HTTPS ではなく HTTP である場合に Kong が応答するステータスコード。フィールドが 301、302、307、または 308 に設定されている場合、Kong によってLocationヘッダーが挿入されます。注: この設定は、ルートがHTTPSプロトコルのみを受け入れるように設定されている場合にのみ適用されます。  受け入れられる値は、426、301、302、307、308 です。デフォルト: 426。regex_priority <br>*任意* | いくつかのルートが同時に正規表現を使ってマッチするとき、与えられたリクエストを解決するルートを選択するために使われる数値です。 2 つのルートがパスに一致し、同じregex_priorityを持つ場合、古い方 (created_atが最も低い) が使用されます。非正規表現ルートの優先順位は異なることにご注意ください（長い非正規表現のルートは短いルートよりも優先されます）。デフォルト: 0strip_path | paths のいずれかを介してルートを照合する場合、アップストリームリクエスト URL から一致するプレフィックスを削除します。デフォルト: true。path_handling<br>*任意*｜リクエストをアップストリームに送信するときに、サービスパス、ルートパス、リクエストパスをどのように組み合わせるかを制御します。それぞれの動作の詳細については、上記をご覧ください。  指定できる値は&quot; v0 &quot;、&quot; v1 &quot; です。デフォルト: &quot;v0&quot;。\npreserve_host | hostsドメイン名の1つを介してルートを照合する場合は、アップストリームのリクエストヘッダーのHostリクエストヘッダーを使用してください。falseに設定すると、アップストリームのHostヘッダーはサービスのhostのヘッダーになります。request_buffering | リクエストボディのバッファリングを有効にするかどうか。HTTP 1.1では、チャンク転送エンコーディングでデータを受信するサービスではこれをオフにするのが理にかなっているかもしれません。  デフォルト: true。response_buffering | レスポンス本文のバッファリングを有効にするかどうか。HTTP 1.1では、チャンク転送エンコーディングでデータを送信するサービスではこれをオフにするのが理にかなっているかもしれません。  デフォルト: true。snis<br>*セミオプション*｜ストリームルーティングを使用するときに、このルートにマッチするSNIのリスト。\nsources<br>*セミオプション*｜ストリームルーティングを使用しているときに、この Route にマッチする着信接続の IP ソースのリスト。各エントリは、「ip」（任意で CIDR 範囲表記）および/または「port」フィールドを持つオブジェクトです。\ndestinations<br>*セミオプション*｜ストリームルーティングを使用しているときに、この Route にマッチする着信接続の IP 宛先のリスト。各エントリは、「ip」（任意で CIDR 範囲表記）および/または「port」フィールドを持つオブジェクトです。\nタグ<br>*任意*｜グループ化とフィルタリングのために Route に関連付けられた文字列の任意セット。\nservice <br>*任意* | このルートが関連付けられるサービス。 ルートがトラフィックをプロキシする場所です。form-encodedでは、service.id=<service id>またはservice.name=<service name>です。JSONでは、&quot;service&quot;:{&quot;id&quot;:&quot;<service id>&quot;}または&quot;service&quot;:{&quot;name&quot;:&quot;<service name>&quot;}を使用します。\n"
route_json: "{\n    \"id\": \"d35165e2-d03e-461a-bdeb-dad0a112abfe\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-route\",\n    \"protocols\": [\"http\", \"https\"],\n    \"methods\": [\"GET\", \"POST\"],\n    \"hosts\": [\"example.com\", \"foo.test\"],\n    \"paths\": [\"/foo\", \"/bar\"],\n    \"headers\": {\"x-my-header\":[\"foo\", \"bar\"], \"x-another-header\":[\"bla\"]},\n    \"https_redirect_status_code\": 426,\n    \"regex_priority\": 0,\n    \"strip_path\": true,\n    \"path_handling\": \"v0\",\n    \"preserve_host\": false,\n    \"request_buffering\": true,\n    \"response_buffering\": true,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"service\": {\"id\":\"af8330d3-dbdc-48bd-b1be-55b98608834b\"}\n}\n"
route_data: "\"data\": [{\n    \"id\": \"a9daa3ba-8186-4a0d-96e8-00d80ce7240b\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-route\",\n    \"protocols\": [\"http\", \"https\"],\n    \"methods\": [\"GET\", \"POST\"],\n    \"hosts\": [\"example.com\", \"foo.test\"],\n    \"paths\": [\"/foo\", \"/bar\"],\n    \"headers\": {\"x-my-header\":[\"foo\", \"bar\"], \"x-another-header\":[\"bla\"]},\n    \"https_redirect_status_code\": 426,\n    \"regex_priority\": 0,\n    \"strip_path\": true,\n    \"path_handling\": \"v0\",\n    \"preserve_host\": false,\n    \"request_buffering\": true,\n    \"response_buffering\": true,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"service\": {\"id\":\"127dfc88-ed57-45bf-b77a-a9d3a152ad31\"}\n}, {\n    \"id\": \"9aa116fd-ef4a-4efa-89bf-a0b17c4be982\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-route\",\n    \"protocols\": [\"tcp\", \"tls\"],\n    \"https_redirect_status_code\": 426,\n    \"regex_priority\": 0,\n    \"strip_path\": true,\n    \"path_handling\": \"v0\",\n    \"preserve_host\": false,\n    \"request_buffering\": true,\n    \"response_buffering\": true,\n    \"snis\": [\"foo.test\", [\"example.com\"],\n    \"sources\": [{\"ip\":\"10.1.0.0/16\", \"port\":1234}, {\"ip\":\"10.2.2.2\"}, {\"port\":9123}],\n    \"destinations\": [{\"ip\":\"10.1.0.0/16\", \"port\":1234}, {\"ip\":\"10.2.2.2\"}, {\"port\":9123}],\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"],\n    \"service\": {\"id\":\"ba641b07-e74a-430a-ab46-94b61e5ea66b\"}\n}],\n"
consumer_body: "属性名 | 説明\n---: |---\nusername<br>*セミオプション* | コンシューマ固有のユーザー名。このフィールドまたはcustom_idのいずれかをリクエストとともに送信する必要があります。\ncustom_id<br>*セミオプション*｜コンシューマの既存の一意なIDを格納するフィールド - 既存のデータベースのユーザーとKongをマッピングする上で便利です。このフィールドまたはusernameのいずれかをリクエストとともに送信する必要があります。\ntags<br>*任意*｜グループ化とフィルタリングのためにコンシューマに関連付けられた文字列の任意セット。\n"
consumer_json: "{\n    \"id\": \"ec1a1f6f-2aa4-4e58-93ff-b56368f19b27\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"username\": \"my-username\",\n    \"custom_id\": \"my-custom-id\",\n    \"tags\": [\"user-level\", \"low-priority\"]\n}\n"
consumer_data: "\"data\": [{\n    \"id\": \"a4407883-c166-43fd-80ca-3ca035b0cdb7\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"username\": \"my-username\",\n    \"custom_id\": \"my-custom-id\",\n    \"tags\": [\"user-level\", \"low-priority\"]\n}, {\n    \"id\": \"01c23299-839c-49a5-a6d5-8864c09184af\",\n    \"created_at\": 1422386534,\n    \"username\": \"my-username\",\n    \"custom_id\": \"my-custom-id\",\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"]\n}],\n"
plugin_body: "属性名 | 説明\n---:| ---\n name | 追加するプラグイン名。現在、プラグインは Kong インスタンスごとに個別にインストールする必要があります。\nroute<br>*任意* | 設定された場合、プラグインは指定されたルート経由でリクエストを受け取るときのみ有効になります。使用するルートに関係なくプラグインを有効にするには、設定しないでおきます。デフォルト: 'null'。form-encodedでは、表記は 'route.id<route id>=' または route.name=<route name>。JSONを使用する場合は、&quot;route&quot;:{&quot;id&quot;:&quot;<route id>&quot;}'または'&quot;route&quot;:{&quot;name&quot;:&quot;を使用します。<route name>&quot;} .\n'サービス'<br>*任意* | 設定されている場合、プラグインは、指定されたサービスに属するルートの1つを介してリクエストを受信した場合にのみアクティブになります。一致するサービスに関係なくプラグインがアクティブになるには、未設定のままにします。 デフォルト: 'null'。form-encoded では、表記は 'service.id<service id>=' または 'service.name=<service name>' です。JSONでは、'&quot;service&quot;:{&quot;id&quot;:&quot;<service id>&quot;}'または'&quot;service&quot;:{&quot;name&quot;:&quot;を使用します。<service name>&quot;} .\n「消費者」<br>*任意* | 設定されている場合、プラグインは、指定されたリクエストが認証されているリクエストに対してのみアクティブになります。(一部のプラグインは、この方法で消費者に制限できないことに注意してください。認証されたコンシューマに関係なくプラグインがアクティブになるには、未設定のままにします。デフォルト：'null'。form-encodedでは、表記はconsumer.id=<consumer id>またはconsumer.username=<consumer username>。JSONを使用する場合は、'&quot;consumer&quot;:{&quot;id&quot;:&quot;<consumer id>&quot;}'または&quot;consumer&quot;:{&quot;username&quot;:&quot;<consumer username>&quot;}を使用します。\n「instance_name」<br>*任意* |プラグインインスタンス名。'config' (設定)<br>*任意* | プラグインの設定プロパティは、[Kong Hub](/hub/)のプラグインのドキュメンテーションページにあります。\n「プロトコル」 | このプラグインをトリガーするリクエストプロトコルのリスト。デフォルト値、およびこのフィールドで許可される可能性のある値は、プラグインのタイプによって異なる場合があります。たとえば、ストリームモードでのみ動作するプラグインは、'&quot;tcp&quot;' と '&quot;tls&quot;' のみをサポートします。デフォルト: '[&quot;grpc&quot;, &quot;grpcs&quot;, &quot;http&quot;,'' &quot;https&quot;]'\n'有効' |プラグインが適用されているかどうか。デフォルト: 'true'。\n'タグ'<br>*任意* | グループ化とフィルタリングのためにプラグインに関連付けられた文字列の任意セット。\n「注文」<br>*任意* <span class=\"badge enterprise\"></span> |「アクセス」フェーズ中のプラグインの順序を決定するための別のプラグインへの依存関係について説明します。 <br> --'before': プラグインは、指定されたプラグインまたはプラグインのリストの「前に」に実行されます。 <br> -- 'after': プラグインは指定されたプラグインまたはプラグインのリストの「後で」実行されます。"
plugin_json: "{\n    \"id\": \"ce44eef5-41ed-47f6-baab-f725cecf98c7\",\n    \"name\": \"rate-limiting\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"route\": null,\n    \"service\": null,\n    \"consumer\": null,\n    \"instance_name\": rate-limiting-foo,\n    \"config\": {\"hour\":500, \"minute\":20},\n    \"protocols\": [\"http\", \"https\"],\n    \"enabled\": true,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"ordering\": {\"before\":[\"plugin-name\"]}\n}\n"
plugin_data: "\"data\": [{\n    \"id\": \"02621eee-8309-4bf6-b36b-a82017a5393e\",\n    \"name\": \"rate-limiting\",\n    \"instance_name\": \"rate-limiting-foo\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"route\": null,\n    \"service\": null,\n    \"consumer\": null,\n    \"config\": {\"hour\":500, \"minute\":20},\n    \"protocols\": [\"http\", \"https\"],\n    \"enabled\": true,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"ordering\": {\"before\":[\"plugin-name\"]}\n}, {\n    \"id\": \"66c7b5c4-4aaf-4119-af1e-ee3ad75d0af4\",\n    \"name\": \"rate-limiting\",\n    \"created_at\": 1422386534,\n    \"route\": null,\n    \"service\": null,\n    \"consumer\": null,\n    \"config\": {\"hour\":500, \"minute\":20},\n    \"protocols\": [\"tcp\", \"tls\"],\n    \"enabled\": true,\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"],\n    \"ordering\": {\"after\":[\"plugin-name\"]}\n}],\n"
certificate_body: "属性名 | 説明\n---:| ---\n cert | SSL キーペアのPEMエンコードされた公開証明書チェーン。\nこのフィールドは「参照可能」です。つまり、[シークレット] (/gateway/latest/plan-and-deploy/security/secrets-management/getting-started/) としてvaultに安全に保存できます。リファレンスは [特定の形式]（/gateway/latest/plan-and-deploy/security/secrets-management/reference-format/）に従う必要があります。\nkey | SSL キーペアの PEM エンコードされた秘密鍵。このフィールドは「参照可能」です。つまり、[シークレット] (/gateway/latest/plan-and-deploy/security/secrets-management/getting-started/) としてvaultに安全に保存できます。リファレンスは [特定の形式]（/gateway/latest/plan-and-deploy/security/secrets-management/reference-format/）に従う必要があります。\ncert_alt <br>*任意* | 代替 SSL 鍵ペアの PEM エンコードされた公開証明書チェーン。 これは、RSA と ECDSA の両方のタイプの証明書が利用可能であり、クライアントが ECDSA 証明書のサポートをアドバタイズするときに Kong が ECDSA 証明書を使用してサービスを優先する場合にのみ設定する必要があります。このフィールドは「参照可能」です。つまり、[シークレット] (/gateway/latest/plan-and-deploy/security/secrets-management/getting-started/) としてvaultに安全に保存できます。リファレンスは [特定の形式]（/gateway/latest/plan-and-deploy/security/secrets-management/reference-format/）に従う必要があります。\nkey_alt <br>*任意* | 代替 SSL 鍵ペアの PEM エンコードされた秘密鍵。 これは、RSA と ECDSA の両方のタイプの証明書が利用可能であり、クライアントが ECDSA 証明書のサポートをアドバタイズするときに Kong が ECDSA 証明書を使用してサービスを優先する場合にのみ設定する必要があります。このフィールドは「参照可能」です。つまり、[シークレット] (/gateway/latest/plan-and-deploy/security/secrets-management/getting-started/) としてvaultに安全に保存できます。リファレンスは [特定の形式]（/gateway/latest/plan-and-deploy/security/secrets-management/reference-format/）に従う必要があります。\nタグ<br>*任意* | グループ化とフィルタリングのための証明書に関連付けられた文字列の任意セット。\nSNI<br>*省略形属性* | SNI としてこの証明書に関連付ける 0 個以上のホスト名の配列。 これは便利なパラメータです。内部的には SNI オブジェクトを作成し、それをこの証明書に関連付けます。 この属性を設定するには、この証明書に有効な秘密キーが関連付けられている必要があります。\npassphrase <br>*任意*（Enterpriseのみ）｜暗号化された秘密鍵をKongにロードするには、この属性を使ってパスフレーズを指定します。Kongが秘密鍵を復号化し、データベースに保存します。Kong のデータベース内の秘密鍵やその他の機密情報を暗号化するには、DB 暗号化の使用を検討してください。\n"
certificate_json: "{\n    \"id\": \"7fca84d6-7d37-4a74-a7b0-93e576089a41\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"cert\": \"-----BEGIN CERTIFICATE-----...\",\n    \"key\": \"-----BEGIN RSA PRIVATE KEY-----...\",\n    \"cert_alt\": \"-----BEGIN CERTIFICATE-----...\",\n    \"key_alt\": \"-----BEGIN EC PRIVATE KEY-----...\",\n    \"tags\": [\"user-level\", \"low-priority\"]\n}\n"
certificate_data: "\"data\": [{\n    \"id\": \"d044b7d4-3dc2-4bbc-8e9f-6b7a69416df6\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"cert\": \"-----BEGIN CERTIFICATE-----...\",\n    \"key\": \"-----BEGIN RSA PRIVATE KEY-----...\",\n    \"cert_alt\": \"-----BEGIN CERTIFICATE-----...\",\n    \"key_alt\": \"-----BEGIN EC PRIVATE KEY-----...\",\n    \"tags\": [\"user-level\", \"low-priority\"]\n}, {\n    \"id\": \"a9b2107f-a214-47b3-add4-46b942187924\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"cert\": \"-----BEGIN CERTIFICATE-----...\",\n    \"key\": \"-----BEGIN RSA PRIVATE KEY-----...\",\n    \"cert_alt\": \"-----BEGIN CERTIFICATE-----...\",\n    \"key_alt\": \"-----BEGIN EC PRIVATE KEY-----...\",\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"]\n}],\n"
ca_certificate_body: "属性名 | 説明\n---:| ---\n cert | CAのPEMエンコードされた公開証明書。\ncert_digest <br>*任意* | 公開証明書の SHA256 16進ダイジェスト。\nタグ<br>*任意* | グループ化とフィルタリングのための証明書に関連付けられた文字列の任意セット。\n"
ca_certificate_json: "{\n    \"id\": \"04fbeacf-a9f1-4a5d-ae4a-b0407445db3f\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"cert\": \"-----BEGIN CERTIFICATE-----...\",\n    \"cert_digest\": \"c641e28d77e93544f2fa87b2cf3f3d51...\",\n    \"tags\": [\"user-level\", \"low-priority\"]\n}\n"
ca_certificate_data: "\"data\": [{\n    \"id\": \"43429efd-b3a5-4048-94cb-5cc4029909bb\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"cert\": \"-----BEGIN CERTIFICATE-----...\",\n    \"cert_digest\": \"c641e28d77e93544f2fa87b2cf3f3d51...\",\n    \"tags\": [\"user-level\", \"low-priority\"]\n}, {\n    \"id\": \"d26761d5-83a4-4f24-ac6c-cff276f2b79c\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"cert\": \"-----BEGIN CERTIFICATE-----...\",\n    \"cert_digest\": \"c641e28d77e93544f2fa87b2cf3f3d51...\",\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"]\n}],\n"
sni_body: "属性 | 説明\n---:| ---\n  name  | 指定された証明書に関連付ける SNI 名。\nタグ<br>*任意*｜グループ化とフィルタリングのためにSNIに関連付けられた文字列の任意セット。\ncertificate | SNI ホスト名を関連付ける証明書の ID (UUID)。SNI オブジェクトで使用するには、証明書に有効な秘密鍵が関連付けられている必要があります。フォームがエンコードされている場合、表記はcertificate.id=<certificate id>となります。JSONでは、&quot;certificate&quot;:{&quot;id&quot;:&quot;<certificate id>&quot;}を使用してください。\n"
sni_json: "{\n    \"id\": \"91020192-062d-416f-a275-9addeeaffaf2\",\n    \"name\": \"my-sni\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"certificate\": {\"id\":\"a2e013e8-7623-4494-a347-6d29108ff68b\"}\n}\n"
sni_data: "\"data\": [{\n    \"id\": \"147f5ef0-1ed6-4711-b77f-489262f8bff7\",\n    \"name\": \"my-sni\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"certificate\": {\"id\":\"a3ad71a8-6685-4b03-a101-980a953544f6\"}\n}, {\n    \"id\": \"b87eb55d-69a1-41d2-8653-8d706eecefc0\",\n    \"name\": \"my-sni\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"],\n    \"certificate\": {\"id\":\"4e8d95d4-40f2-4818-adcb-30e00c349618\"}\n}],\n"
upstream_body: "属性名 | 説明\n---: |---\n name  | これはホスト名です。サービスの「ホスト」と同じでなければなりません。\nalgorithm<br>*任意* | 使用するロードバランシングアルゴリズム。受け入れられる値は、&quot;consistent-hashing&quot;、&quot;least-connections&quot;、 &quot;round-robin&quot;、&quot;latency&quot;です。デフォルト：&quot;latency&quot;。\n hash_on <br>*任意* | 使用するハッシュ入力。none を使用すると、ハッシュなしの加重ラウンドロビン方式になります。受け入れられる値は：&quot;none&quot;、&quot;consumer&quot;、&quot;ip&quot;、 &quot; header &quot; 、 &quot;cookie&quot;、&quot;path&quot;、&quot;query_arg&quot;、&quot;uri_capture&quot;。デフォルト: &quot;none&quot;。\nhash_fallback <br>*任意* |  hash_on  がハッシュを返さない場合にハッシュの入力として使用するもの (例.ヘッダーが欠落しているか、コンシューマが識別されていません)。hash_on が cookie に設定されている場合は使用できません。受け入れられる値は：&quot;none&quot;、&quot;consumer&quot;、&quot;ip&quot;、 &quot; header &quot; 、 &quot;cookie&quot;、&quot;path&quot;、&quot;query_arg&quot;、&quot;uri_capture&quot;。デフォルト: &quot;none&quot;。\nhash_on_header <br>*セミオプション*｜ハッシュの入力として値を受け取るヘッダー名。hash_on が header に設定されている場合にのみ必要です。\nhash_fallback_header<br>*セミオプション* | ハッシュの入力として値を受け取るヘッダー名。hash_fallback が header に設定されている場合にのみ必要です。\nhash_on_cookie 。<br>*セミオプション*｜ハッシュの入力として値を取得するcookieの名前。hash_on または hash_fallback が cookie に設定されている場合にのみ必要です。指定された Cookie がリクエストにない場合、Kong は値を生成し、応答に Cookie を設定します。\nhash_on_cookie_path：<br>*セミオプション*｜レスポンスヘッダーに設定するクッキーのパス。hash_on または hash_fallback が cookie に設定されている場合にのみ必要です。デフォルト: '&quot;/&quot;'。hash_on_query_arg：<br>*ハッシュの入力として値を受け取るクエリ文字列の名前。hash_on が query_arg に設定されている場合にのみ必要です。\nhash_fallback_query_arg：<br>*セミオプション* | ハッシュの入力として値を受け取るクエリ文字列の名前。hash_fallback が query_arg に設定されている場合にのみ必要です。\nhash_on_uri_capture <br>*セミオプション* | ハッシュの入力として値を受け取るルート URI キャプチャの名前。hash_on が uri_capture に設定されている場合にのみ必要です。\nhash_fallback_uri_capture を指定します。<br>*セミオプション* | ハッシュの入力として値を受け取るルート URI キャプチャの名前。 hash_fallback が uri_capture に設定されている場合にのみ必要です。\nslots <br>*任意* | ロードバランサーアルゴリズムのスロット数。 algorithmが round-robin に設定されている場合、この設定によってスロットの最大数が決まります。algorithm が consistent-hashingに設定されている場合、この設定によってアルゴリズムの実際のスロット数が決まります。10から65536までの整数を指定します。デフォルト: '10000'。 healthchecks.passive. type <br>*任意* | HTTP/HTTPS ステータスを解釈するパッシブヘルスチェックを行うか、TCP 接続の成功だけをチェックするかを示しています。パッシブチェックでは、 httpとHTTPSオプションは同等です。受け入れられる値は、 &quot;tcp&quot; 、http、 &quot;https&quot; 、 &quot;grpc&quot; 、 &quot;grpcs&quot; です。デフォルトは、http。\n healthchecks.passive.  healthy.http_statuses <br>*任意* | パッシブヘルスチェックによって観測された、プロキシされたトラフィックによって生成されたときの健全性を表す HTTP ステータスの配列。デフォルト: [200、201、202、203、204、205、206、207、208、226、300、301、302、303、304、305、306、307、308] 。form-encodedの場合、http_statuses[]=200&amp;http_statuses[]=201となります。JSONでは、配列を使用します。 healthchecks.passive.  healthy.successes <br>*任意* | プロキシされたトラフィックの成功数（ healthchecks.passive.healthy.http_statusesで定義）パッシブヘルスチェックによって観察されたターゲットが正常であると見なします。デフォルト: 0 healthchecks.passive. unhealthy.http_statuses<br>*任意* | パッシブヘルスチェックによって観測された、プロキシされたトラフィックによって生成されたときの 不健全性を表す HTTP ステータスの配列。 デフォルト: '[429, 500, 503]'。 form-encodedの場合、http_statuses[]=429&amp;http_statuses[]=500となります。JSONでは、配列を使用します。 healthchecks.passive. 'unhealthy.timeouts'<br>*任意* | パッシブヘルスチェックで確認された、ターゲットが正常でないと見なされるプロキシトラフィックのタイムアウト回数。デフォルト: 0 healthchecks.passive. unhealthy.http_failures<br>*任意* | パッシブヘルスチェックによって観測され、ターゲットが正常でないと判断される、プロキシされたトラフィック内の HTTP 失敗の数 (healthchecks.passive.unhealthy.http_statusesで定義)。デフォルト: 0 healthchecks.passive. unhealthy.tcp_failures<br>*任意* | パッシブヘルスチェックによって観測され、ターゲットが正常でないと判断されるプロキシトラフィック内の TCP 失敗数。デフォルト: 0healthchecks.active.https_verify_certificate | HTTPS を使用してアクティブヘルスチェックを実行するときに、リモートホストの SSL 証明書の有効性をチェックするかどうか。デフォルト: true。 healthchecks.active.  healthy.http_statuses <br>*任意* | アクティブヘルスチェックのプローブから返されたときに、健全性を示す 成功とみなす HTTP ステータスの配列。 デフォルト: '[200, 302]'。form-encodedの場合、http_statuses[]=200&amp;http_statuses[]=302となります。JSONでは、配列を使用します。 healthchecks.active.  healthy.successes <br>*任意* | ターゲットを正常と見なすため、アクティブプローブの成功数（healthchecks.active.healthy.http_statusesで定義）デフォルト: 0healthchecks.active.healthy.interval<br>*任意* | 正常なターゲットのアクティブなヘルスチェックの間隔 (秒単位)。値がゼロの場合、正常なターゲットに対してアクティブなプローブを実行しないことを示します。デフォルト: 0 healthchecks.active.unhealthy.http_failures<br>*任意* | ターゲットが異常であると判断されるアクティブプローブ内の HTTP 失敗の数 ( healthchecks.active.unhealthy.http_statuses で定義)。デフォルト: 0 healthchecks.active. unhealthy.http_statuses<br>*任意* | アクティブヘルスチェックのプローブから返されたときに、不健全を示す失敗とみなす HTTP ステータスの配列。デフォルト: [429、404、500、501、502、503、  504、505]。form-encodedの場合、表記はhttp_statuses[]=429&amp;http_statuses[]=404となります。JSONでは、配列を使用します。healthchecks.active.'unhealthy.timeouts'<br>*任意* | ターゲットが正常でないと見なされるアクティブプローブのタイムアウト回数。デフォルト: 0healthchecks.active.unhealthy.tcp_failures<br>*任意* | ターゲットが異常であると判断されるアクティブ プローブでの TCP 失敗の数。デフォルト: 0 healthchecks.active.  unhealthy.interval <br>*任意* | 正常でないターゲットのアクティブなヘルスチェックの間隔 (秒単位)。値がゼロの場合、正常でないターゲットに対してアクティブ プローブを実行しないことを示します。デフォルト: 0healthchecks.active.type<br>*任意* | HTTP または HTTPS を使用してアクティブヘルスチェックを実行するか、単に TCP 接続を試行するか。 受け入れられる値は、 &quot;tcp&quot; 、http、 &quot;https&quot; 、 &quot;grpc&quot; 、 &quot;grpcs&quot; です。デフォルトは、http。\n healthchecks.active. concurrency<br>*任意* | アクティブなヘルスチェックで同時にチェックするターゲットの数。デフォルト: '10'。 healthchecks.active.  headers  <br>*任意* | アクティブなヘルスチェックのプローブとして実行するGET HTTPリクエストで使用する、ヘッダー名で索引付けされた1つ以上の値のリスト。 値は事前にフォーマットしておく必要があります。 healthchecks.active. timeout <br>*任意* | アクティブなヘルスチェックのソケットタイムアウト (秒単位)。デフォルト: '1' healthchecks.active. http_path <br>*任意* | アクティブなヘルスチェックのプローブとして実行する GET HTTP リクエストで使用するパス。 デフォルト: '&quot;/&quot;'。 healthchecks.active. https_sni <br>*任意* | HTTPS を使用してアクティブヘルスチェックを実行するときに SNI (Server Name Identification) として使用するホスト名。これは、ターゲットが IP を使用して構成されている場合に特に役立ち、ターゲットホストの証明書を適切な SNI で検証できます。\n'healthchecks.threshold'<br>*任意* | 上流全体が健康であると見なされるためには、上流のターゲットの重量の最小パーセンテージです。デフォルト: 0tags<br>*任意*｜グループ化とフィルタリングのためにUpstreamに関連付けられた文字列の任意セット。\nhost_header<br>*任意* | Kong 経由でリクエストをプロキシするときに Host ヘッダとして使われるホスト名。\nclient_certificate <br>*任意* | 設定されている場合、上流サーバとのTLSハンドシェイク時にクライアント証明書として使用する証明書。form-encodedの場合、 client_certificate.id=<client_certificate id> という表記になります。JSONの場合は、&quot;client_certificate&quot;:{&quot;id&quot;:&quot;<client_certificate id>&quot;} を使用します。\nuse_srv_name <br>*任意* | 設定すると、バランサは上流の Host プロキシとして SRV ホスト名 (DNS アンサーに SRV レコードがある場合) を使います。\n"
upstream_json: "{\n    \"id\": \"58c8ccbb-eafb-4566-991f-2ed4f678fa70\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-upstream\",\n    \"algorithm\": \"round-robin\",\n    \"hash_on\": \"none\",\n    \"hash_fallback\": \"none\",\n    \"hash_on_cookie_path\": \"/\",\n    \"slots\": 10000,\n    \"healthchecks\": {\n        \"passive\": {\n            \"type\": \"http\",\n            \"healthy\": {\n                \"http_statuses\": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308],\n                \"successes\": 0\n            },\n            \"unhealthy\": {\n                \"http_statuses\": [429, 500, 503],\n                \"timeouts\": 0,\n                \"http_failures\": 0,\n                \"tcp_failures\": 0\n            }\n        },\n        \"active\": {\n            \"https_verify_certificate\": true,\n            \"healthy\": {\n                \"http_statuses\": [200, 302],\n                \"successes\": 0,\n                \"interval\": 0\n            },\n            \"unhealthy\": {\n                \"http_failures\": 0,\n                \"http_statuses\": [429, 404, 500, 501, 502, 503, 504, 505],\n                \"timeouts\": 0,\n                \"tcp_failures\": 0,\n                \"interval\": 0\n            },\n            \"type\": \"http\",\n            \"concurrency\": 10,\n            \"headers\": [{\"x-my-header\":[\"foo\", \"bar\"], \"x-another-header\":[\"bla\"]}],\n            \"timeout\": 1,\n            \"http_path\": \"/\",\n            \"https_sni\": \"example.com\"\n        },\n        \"threshold\": 0\n    },\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"host_header\": \"example.com\",\n    \"client_certificate\": {\"id\":\"ea29aaa3-3b2d-488c-b90c-56df8e0dd8c6\"},\n    \"use_srv_name\": false\n}\n"
upstream_data: "\"data\": [{\n    \"id\": \"4fe14415-73d5-4f00-9fbc-c72a0fccfcb2\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-upstream\",\n    \"algorithm\": \"round-robin\",\n    \"hash_on\": \"none\",\n    \"hash_fallback\": \"none\",\n    \"hash_on_cookie_path\": \"/\",\n    \"slots\": 10000,\n    \"healthchecks\": {\n        \"passive\": {\n            \"type\": \"http\",\n            \"healthy\": {\n                \"http_statuses\": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308],\n                \"successes\": 0\n            },\n            \"unhealthy\": {\n                \"http_statuses\": [429, 500, 503],\n                \"timeouts\": 0,\n                \"http_failures\": 0,\n                \"tcp_failures\": 0\n            }\n        },\n        \"active\": {\n            \"https_verify_certificate\": true,\n            \"healthy\": {\n                \"http_statuses\": [200, 302],\n                \"successes\": 0,\n                \"interval\": 0\n            },\n            \"unhealthy\": {\n                \"http_failures\": 0,\n                \"http_statuses\": [429, 404, 500, 501, 502, 503, 504, 505],\n                \"timeouts\": 0,\n                \"tcp_failures\": 0,\n                \"interval\": 0\n            },\n            \"type\": \"http\",\n            \"concurrency\": 10,\n            \"headers\": [{\"x-my-header\":[\"foo\", \"bar\"], \"x-another-header\":[\"bla\"]}],\n            \"timeout\": 1,\n            \"http_path\": \"/\",\n            \"https_sni\": \"example.com\"\n        },\n        \"threshold\": 0\n    },\n    \"tags\": [\"user-level\", \"low-priority\"],\n    \"host_header\": \"example.com\",\n    \"client_certificate\": {\"id\":\"a3395f66-2af6-4c79-bea2-1b6933764f80\"},\n    \"use_srv_name\": false\n}, {\n    \"id\": \"885a0392-ef1b-4de3-aacf-af3f1697ce2c\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"name\": \"my-upstream\",\n    \"algorithm\": \"round-robin\",\n    \"hash_on\": \"none\",\n    \"hash_fallback\": \"none\",\n    \"hash_on_cookie_path\": \"/\",\n    \"slots\": 10000,\n    \"healthchecks\": {\n        \"passive\": {\n            \"type\": \"http\",\n            \"healthy\": {\n                \"http_statuses\": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308],\n                \"successes\": 0\n            },\n            \"unhealthy\": {\n                \"http_statuses\": [429, 500, 503],\n                \"timeouts\": 0,\n                \"http_failures\": 0,\n                \"tcp_failures\": 0\n            }\n        },\n        \"active\": {\n            \"https_verify_certificate\": true,\n            \"healthy\": {\n                \"http_statuses\": [200, 302],\n                \"successes\": 0,\n                \"interval\": 0\n            },\n            \"unhealthy\": {\n                \"http_failures\": 0,\n                \"http_statuses\": [429, 404, 500, 501, 502, 503, 504, 505],\n                \"timeouts\": 0,\n                \"tcp_failures\": 0,\n                \"interval\": 0\n            },\n            \"type\": \"http\",\n            \"concurrency\": 10,\n            \"headers\": [{\"x-my-header\":[\"foo\", \"bar\"], \"x-another-header\":[\"bla\"]}],\n            \"timeout\": 1,\n            \"http_path\": \"/\",\n            \"https_sni\": \"example.com\"\n        },\n        \"threshold\": 0\n    },\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"],\n    \"host_header\": \"example.com\",\n    \"client_certificate\": {\"id\":\"f5a9c0ca-bdbb-490f-8928-2ca95836239a\"},\n    \"use_srv_name\": false\n}],\n"
target_body: "属性名 | 説明\n---:| ---\n  target  | ターゲットアドレス (IPまたはホスト名) とポート。ホスト名が SRV レコードに解決される場合、 port  値は DNS レコードの値によって上書きされます。\n weight <br>*任意* | 上流のロードバランサーでこのターゲットが得る重み ( 0 - 65535 ). ホスト名が SRV レコードに解決される場合、 weight値は DNS レコードの値によって上書きされます。デフォルト: '100'。タグ<br>*任意*｜グループ化とフィルタリングのためにターゲットに関連付けられた文字列の任意セットです。\n"
target_json: "{\n    \"id\": \"173a6cee-90d1-40a7-89cf-0329eca780a6\",\n    \"created_at\": 1422386534,\n     \"updated_at\": 1422386534,\n    \"upstream\": {\"id\":\"bdab0e47-4e37-4f0b-8fd0-87d95cc4addc\"},\n    \"target\": \"example.com:8000\",\n    \"weight\": 100,\n    \"tags\": [\"user-level\", \"low-priority\"]\n}\n"
target_data: "\"data\": [{\n    \"id\": \"f00c6da4-3679-4b44-b9fb-36a19bd3ae83\",\n    \"created_at\": 1422386534,\n    \"upstream\": {\"id\":\"0c61e164-6171-4837-8836-8f5298726d53\"},\n    \"target\": \"example.com:8000\",\n    \"weight\": 100,\n    \"tags\": [\"user-level\", \"low-priority\"]\n}, {\n    \"id\": \"5027BBC1-508C-41F8-87F2-AB1801E9D5C3\",\n    \"created_at\": 1422386534,\n    \"upstream\": {\"id\":\"68FDB05B-7B08-47E9-9727-AF7F897CFF1A\"},\n    \"target\": \"example.com:8000\",\n    \"weight\": 100,\n    \"tags\": [\"admin\", \"high-priority\", \"critical\"]\n}],\n"
vault_body: "属性名 | 説明\n---: |---\n プレフィックス | このVault構成の一意のプレフィックス（または識別子）。プレフィックスは、他のエンティティでシークレットを参照するときに、正しいVaultの設定と実装をロードするために使用されます。name | 追加されるVaultの名前。現在、Vault 実装はすべての Kong インスタンスにインストールする必要があります。 説明  <br>*任意* | Vaultオブジェクトの説明。\n config <br>*任意* |  Vaultの設定プロパティは、Vaultのドキュメントページにあります。\ntags <br>*任意* | グループ化とフィルタリングのためのVaultに関連する任意の文字列セット。\n"
vault_json: "{\n    \"id\": \"B2A30E8F-C542-49CF-8015-FB674987D1A5\",\n    \"prefix\": \"env\",\n    \"name\": \"env\",\n    \"description\": \"This vault is used to retrieve redis database access credentials\",\n    \"config\": {\"prefix\":\"SSL_\"},\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"tags\": [\"database-credentials\", \"data-plane\"]\n}\n"
vault_data: "\"data\": [{\n    \"id\": \"518BBE43-2454-4559-99B0-8E7D1CD3E8C8\",\n    \"prefix\": \"env\",\n    \"name\": \"env\",\n    \"description\": \"This vault is used to retrieve redis database access credentials\",\n    \"config\": {\"prefix\":\"SSL_\"},\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"tags\": [\"database-credentials\", \"data-plane\"]\n}, {\n    \"id\": \"7C4747E9-E831-4ED8-9377-83A6F8A37603\",\n    \"prefix\": \"env\",\n    \"name\": \"env\",\n    \"description\": \"This vault is used to retrieve redis database access credentials\",\n    \"config\": {\"prefix\":\"SSL_\"},\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"tags\": [\"certificates\", \"critical\"]\n}],\n"
key_body: "属性名｜説明\n---:| ---\nset <br>*キーを関連付けるキーセットの ID (UUID) です。form-encoded では、 set.id=<set id>  または  set.name=<set name> と表記します。JSONの場合は、&quot;set&quot;:{&quot;id&quot;:&quot;<set id> &quot; } または &quot;set&quot;:{&quot;name&quot;:&quot;<set name> &quot; } を使用します。\nname <br>*任意* | 与えられたキーに関連付ける名前。\nkid | キーに一意な識別子。\njwk <br>*任意* | 文字列で表現された JSON ウェブキー。\npem *任意* | キー。<br>*任意* | PEM 形式のキーペア。\nタグ<br>*任意* | グループ化とフィルタリングのためにキーに関連付けられた文字列の任意セット。\n"
key_json: "{\n    \"id\": \"24D0DBDA-671C-11ED-BA0B-EF1DCCD3725F\",\n    \"set\": {\"id\":\"46CA83EE-671C-11ED-BFAB-2FE47512C77A\"},\n    \"name\": \"my-key\",\n    \"kid\": \"42\",\n    \"jwk\": \"{\\\"alg\\\":\\\"RSA\\\",  \\\"kid\\\": \\\"42\\\",  ...}\",\n    \"pem\": {\n        \"private_key\": \"-----BEGIN\",\n        \"public_key\": \"-----BEGIN\"\n    },\n    \"tags\": [\"application-a\", \"public-key-xyz\"],\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534\n}\n"
key_data: "\"data\": [{\n    \"id\": \"57532ECE-6720-11ED-9297-279D4320B841\",\n    \"set\": {\"id\":\"68A64404-6720-11ED-8DE1-47A2EE5E214E\"},\n    \"name\": \"my-key\",\n    \"kid\": \"42\",\n    \"jwk\": \"{\\\"alg\\\":\\\"RSA\\\",  \\\"kid\\\": \\\"42\\\",  ...}\",\n    \"pem\": {\n        \"private_key\": \"-----BEGIN\",\n        \"public_key\": \"-----BEGIN\"\n    },\n    \"tags\": [\"application-a\", \"public-key-xyz\"],\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534\n}, {\n    \"id\": \"76BF58F0-6720-11ED-9341-BF64C05FB0CB\",\n    \"set\": {\"id\":\"B235B4BA-6720-11ED-A3DD-67F2793BCB9E\"},\n    \"name\": \"my-key\",\n    \"kid\": \"42\",\n    \"jwk\": \"{\\\"alg\\\":\\\"RSA\\\",  \\\"kid\\\": \\\"42\\\",  ...}\",\n    \"pem\": {\n        \"private_key\": \"-----BEGIN\",\n        \"public_key\": \"-----BEGIN\"\n    },\n    \"tags\": [\"RSA\", \"ECDSA\"],\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534\n}],\n"
key_set_body: "属性名｜説明\n---:| ---\nname<br>*任意* | 与えられたキーセットに関連付ける名前。\ntags<br>*任意*｜グループ化とフィルタリングのためにキーに関連付けられた文字列の任意セット。\n"
key_set_json: "{\n    \"id\": \"24D0DBDA-671C-11ED-BA0B-EF1DCCD3725F\",\n    \"name\": \"my-key_set\",\n    \"tags\": [\"google-keys\", \"mozilla-keys\"],\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534\n}\n"
key_set_data: "\"data\": [{\n    \"id\": \"46CA83EE-671C-11ED-BFAB-2FE47512C77A\",\n    \"name\": \"my-key_set\",\n    \"tags\": [\"google-keys\", \"mozilla-keys\"],\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534\n}, {\n    \"id\": \"57532ECE-6720-11ED-9297-279D4320B841\",\n    \"name\": \"my-key_set\",\n    \"tags\": [\"production\", \"staging\", \"development\"],\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534\n}],\n"
filter_chain_body: "属性 | 説明\n---:| ---\n  name  | フィルターチェーンの名前。\n'有効' |フィルターチェーンが適用されているかどうか。 デフォルト: true。ルート<br>*任意* | このチェーンが適用されるルート。 フィルター チェーンは、単一のルートまたは単一のサービスに適用する必要があります。デフォルト: 'null'。form-encodedの場合、表記は route.id=<route id>   または  route.name=<route name>、JSONでは、&quot;route&quot;:{&quot;id&quot;:&quot;<route id>&quot;}または&quot;route&quot;:{&quot;id&quot;:&quot;&quot;}<route name>を使用します。\nservice <br> *任意* | このチェーンが適用されるサービス。フィルターチェーンは、単一のルートまたはサービスに適用する必要があります。デフォルト: null 。form-encodedの場合、表記はservice.id=<service id>または service.name=<service name>です。JSONでは、&quot;service&quot;:{&quot;id&quot;:&quot;<service id>&quot;}または&quot;service&quot;:{&quot;name&quot;:&quot;<service name>&quot;}です。\n filters | フィルター定義の配列。各フィルターは、必須の name、任意の config 、およびブール値の enabled 設定を含むオブジェクトです。\n tags <br> *任意* | グループ化とフィルタリングのためのフィルターチェーンに関連付けられた任意の文字列セット。\n"
filter_chain_json: "{\n    \"id\": \"ce44eef5-41ed-47f6-baab-f725cecf98c7\",\n    \"name\": \"my-chain\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"enabled\": true,\n    \"route\": null,\n    \"service\": \"20487393-41ed-47f6-93a8-3407cade2002\",,\n    \"filters\": [\n        {\n            \"name\": \"go-rate-limiting\",\n            \"enabled\": true,\n            \"config\": \"{ \\\"minute\\\": 30 }\"\n        },\n        {\n            \"name\": \"rust-response-transformer\",\n            \"enabled\": true,\n            \"config\": \"{ \\\"remove_header\\\": \\\"X-Example\\\" }\"\n        }\n    ],\n    \"tags\": [\"my-tag\"]\n}\n"
filter_chain_data: "\"data\": [{\n    \"id\": \"ce44eef5-41ed-47f6-baab-f725cecf98c7\",\n    \"name\": \"my-chain\",\n    \"created_at\": 1422386534,\n    \"updated_at\": 1422386534,\n    \"enabled\": true,\n    \"route\": null,\n    \"service\": \"20487393-41ed-47f6-93a8-3407cade2002\",,\n    \"filters\": [\n        {\n            \"name\": \"go-rate-limiting\",\n            \"enabled\": true,\n            \"config\": \"{ \\\"minute\\\": 30 }\"\n        },\n        {\n            \"name\": \"rust-response-transformer\",\n            \"enabled\": true,\n            \"config\": \"{ \\\"remove_header\\\": \\\"X-Example\\\" }\"\n        }\n    ],\n    \"tags\": [\"my-tag\"]\n}]\n"
filters_data: 
    "filters": 
        - 
            "config": "{\"minute\": 3}"
            "enabled": true
            "filter_chain": 
                "id": "0385abc3-f8a7-505b-aeec-9375a298d340"
                "name": null
            "from": "サービス"
            "name": "go-rate-limiting"
        - 
            "config": "{\"remove_header\": \"X-Example\"}"
            "enabled": true
            "filter_chain": 
                "id": "d8e9a640-f8a7-505b-aeec-e657383940d6"
                "name": null
            "from": "ルート"
            "name": "rust-response-transformer"
---
{:.note .no-icon }
> 
> <span class="badge beta"></span> **{{site.base_gateway}} API 仕様が利用可能になりました。** 

|                         仕様                          |                                                                                                                                              Insomniaのリンク                                                                                                                                               |
|-----------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [EnterpriseベータAPI仕様](/gateway/api/admin-ee/latest/) | <a href="https://insomnia.rest/run/?label=Kong%20Gateway%20Enterprise%203.4&uri=https%3A%2F%2Fraw.githubusercontent.com%2FKong%2Fdocs.konghq.com%2Fmain%2Fapi-specs%2FGateway-EE%2F3.4%2Fkong-ee-3.4.json" target="_blank"><img src="https://insomnia.rest/images/run.svg" alt="Insomniaでの実行"></a>      |
| [オープンソースのベータ版API仕様](/gateway/api/admin-oss/latest/) | <a href="https://insomnia.rest/run/?label=Kong%20Gateway%20Open%20Source%203.4&uri=https%3A%2F%2Fraw.githubusercontent.com%2FKong%2Fdocs.konghq.com%2Fmain%2Fapi-specs%2FGateway-OSS%2F3.4%2Fkong-oss-3.4.json" target="_blank"><img src="https://insomnia.rest/images/run.svg" alt="Insomniaでの実行"></a> |


{{site.base_gateway}}には、管理用の **内部** RESTful Admin APIが付属しています。
Admin APIへのリクエストはクラスタ内のどのノードにも送信でき、Kongはすべての
ノード間で構成の一貫性を維持します。

* `8001`は、Admin APIがリッスンするデフォルトのポートです。
* `8444` は、Admin API への HTTPS トラフィックのデフォルトポートです。

このAPIは内部使用向けに設計されており、Kongを完全に制御可能にします。そのためKong環境を設定する際には、このAPIが不当に公開されることがないよう、注意してください。Admin APIを保護する方法については、[このドキュメント](/gateway/{{page.release}}/production/running-kong/secure-admin-api)を参照してください。

*** ** * ** ***

DBレスモード
-------

[DB\-lessモード](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config/)では、Admin APIを使用して新しい宣言型構成を読み込み、現在の構成を検査できます。DB\-lessモードでは、各KongノードのAdmin APIは独立して機能し、特定のKongノードのメモリの状態を反映します。これは、Kongノード間のデータベース連携がないためです。

DB レスモードでは、{{site.base_gateway}}を宣言的に構成します。したがって、Admin APIはほとんど読み取り専用です。実行できるタスクはすべて宣言型構成の処理に関連するものであり、次のものが含まれます。

* [スキーマに照らした構成の検証](#validate-a-configuration-against-a-schema)
* [スキーマに対するプラグイン構成の検証](#validate-a-plugin-configuration-against-the-schema)
* [宣言型構成の再読み込み](#reload-declarative-configuration)
* [ロードバランサー内でのターゲットのヘルスステータスの設定](#set-target-as-healthy)

*** ** * ** ***

宣言型構成
-----

エンティティの宣言型構成を{{site.base_gateway}}に読み込むには、`declarative_config`プロパティを介して起動時に実行する方法と、`/config`エンドポイントを使用してAdmin APIを介して実行時に行う方法の2通りあります。

宣言型構成を使い始めるには、エンティティ定義を含むファイル（YAMLまたはJSON形式）が必要です。次のコマンドを使用して、サンプルの宣言型構成を生成できます。

    kong config init

現在のディレクトリに、適切な構造と例が含まれた`kong.yml`という名前のファイルが生成されます。

### 宣言型構成の再読み込み

このエンドポイントでは、DB\-less Kong を新しい宣言型構成データファイルでリセットできます。以前の内容はすべてメモリから消去され、指定されたファイル内に記述したエンティティがそれに取って代わります。

ファイル形式の詳細については、[宣言型構成](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config)ドキュメントを参照してください。
<div class="endpoint post indent">/config</div> 

{:.indent}
| 属性 | 説明 | 
|---| ---- | 
|`config`<br> **必須** | ロードする構成データ（YAMLまたはJSON形式）。|

#### リクエストのクエリ文字列パラメータ

|                       属性 |                                    説明                                     |
|-------------------------|---------------------------------------------------------------------------|
| `check_hash`<br> *オプション* | 1に設定すると、Kongは入力構成データのハッシュを前のハッシュと比較します。構成が同一の場合は、再読み込みされず、HTTP 304が返されます。 |

#### レスポンス

    HTTP 200 OK

```json
{
    {
        "services": [],
        "routes": []
    }
}
```

レスポンスには、入力ファイルから解析されたすべてのエンティティのリストが含まれます。

*** ** * ** ***

サポートされているコンテンツタイプ
-----------------

Admin APIは、すべてのエンドポイントで3つのコンテンツタイプを受け入れます。

* **application/json** 

ボディが複雑（複雑なプラグイン構成など）な場合に便利で、単に送信したいデータのJSON表現を送信します。例：

```json
{
    "config": {
        "limit": 10,
        "period": "seconds"
    }
}
```

以下は、`test-service` という名前のサービスにルートを追加する例です。

    curl -i -X POST http://localhost:8001/services/test-service/routes \
         -H "Content-Type: application/json" \
         -d '{"name": "test-route", "paths": [ "/path/one", "/path/two" ]}'

* **application/x\-www\-form\-urlencoded** 

基本的なリクエストボディとしては十分に単純であるため、ほとんどの場合はこれを使用します。
ネストされた値を送信する場合、Kongはネストされたオブジェクトがドット付きのキーを使って参照されることを想定しています。
以下に例を示します。

    config.limit=10&config.period=seconds

配列を指定するときは、値を順番に送信するか、角括弧を使用します（角括弧内の番号付けはオプションですが、指定する場合は1インデックスで連続している必要があります）。以下は、`test-service` という名前のサービスに追加されたルートの例です。

    curl -i -X POST http://localhost:8001/services/test-service/routes \
         -d "name=test-route" \
         -d "paths[1]=/path/one" \
         -d "paths[2]=/path/two"

次の2つの例は上記のものと同じですが、明示的ではありません。

    curl -i -X POST http://localhost:8001/services/test-service/routes \
         -d "name=test-route" \
         -d "paths[]=/path/one" \
         -d "paths[]=/path/two"
    
    curl -i -X POST http://localhost:8001/services/test-service/routes \
        -d "name=test-route" \
        -d "paths=/path/one" \
        -d "paths=/path/two"

* **multipart/form\-data** 

URLエンコードと同様に、このコンテンツタイプでは、ドットキーを使用してネストされたオブジェクトを参照します。以下は、LuaファイルをプリファンクションKongプラグインに送信する例です。

    curl -i -X POST http://localhost:8001/services/plugin-testing/plugins \
         -F "name=pre-function" \
         -F "config.access=@custom-auth.lua"

このコンテンツタイプの配列を指定する場合は、配列インデックスを指定する必要があります。次は、`test-service` という名前のサービスに追加されたルートの例です。

    curl -i -X POST http://localhost:8001/services/test-service/routes \
         -F "name=test-route" \
         -F "paths[1]=/path/one" \
         -F "paths[2]=/path/two"

*** ** * ** ***

ワークスペースでのAPIの使用
---------------
{:.badge .enterprise}

{% include_cached /md/gateway/admin-api-workspaces.md %}

*** ** * ** ***

インフォメーションルート
------------

### ノード情報の取得
{:.badge .dbless}

ノードについての一般的な詳細を取得します。
<div class="endpoint get">/</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "hostname": "",
    "node_id": "6a72192c-a3a1-4c8d-95c6-efabae9fb969",
    "lua_version": "LuaJIT 2.1.0-beta3",
    "plugins": {
        "available_on_server": [
            ...
        ],
        "enabled_in_cluster": [
            ...
        ]
    },
    "configuration": {
        ...
    },
    "tagline": "Welcome to Kong",
    "version": "0.14.0"
}
```

* `node_id`: 実行中の Kong ノードを表す UUID。この UUID は Kong が起動するとランダムに生成されるので、ノードは再起動するたびに異なる `node_id` を持つことになります。
* `available_on_server`: ノードにインストールされるプラグインの名前。
* `enabled_in_cluster`: 有効化されている、または設定されているプラグインの名前。つまり、すべての Kong ノードによってデータストアに現在共有されているプラグイン構成です。

*** ** * ** ***

### エンドポイントまたはエンティティの有無の確認
{:.badge .dbless}

`HTTP GET`と似ていますが、本文は返しません。エンドポイントが存在する場合は`HTTP 200` を返し、存在しない場合は`HTTP 404`を返します。他のステータスコードも可能です。
<div class="endpoint head">/<any\-endpoint></div> 

#### レスポンス

    HTTP 200 OK

```http
Access-Control-Allow-Origin: *
Content-Length: 11389
Content-Type: application/json; charset=utf-8
X-Kong-Admin-Latency: 1
```

*** ** * ** ***

### エンドポイント別HTTPメソッドの一覧表示
{:.badge .dbless}

エンドポイントでサポートされているすべての`HTTP`メソッドを一覧表示します。これは、`CORS`プリフライトリクエストでも使用できます。
<div class="endpoint options">/<any\-endpoint></div> 

#### レスポンス

    HTTP 204 No Content

```http
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Methods: GET, HEAD, OPTIONS
Access-Control-Allow-Origin: *
Allow: GET, HEAD, OPTIONS
```

*** ** * ** ***

### 使用可能なエンドポイントの一覧化
{:.badge .dbless}

Admin APIによって提供される使用可能なエンドポイントをすべて一覧表示します。
<div class="endpoint get">/endpoints</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "data": [
        "/",
        "/acls",
        "/acls/{acls}",
        "/acls/{acls}/consumer",
        "/basic-auths",
        "/basic-auths/{basicauth_credentials}",
        "/basic-auths/{basicauth_credentials}/consumer",
        "/ca_certificates",
        "/ca_certificates/{ca_certificates}",
        "/cache",
        "/cache/{key}",
        "..."
    ]
}
```

*** ** * ** ***

### スキーマに対する構成の検証
{:.badge .dbless}

エンティティスキーマに照らして構成の有効性を確認します。
これにより、Admin APIのエンティティエンドポイントにリクエストを送信する前に入力内容をテストできます。

これは、入力の構成が正しいことをチェックするスキーマ検証チェックのみを実行することに注意してください。指定された構成を使用するエンティティエンドポイントへのリクエストは、無効な外部キー関係や、データストアの内容に対する一意性チェックのエラーなど、他の理由で失敗する可能性があります。
<div class="endpoint post">/schemas/\{entity\}/validate</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "message": "schema validation successful"
}
```

*** ** * ** ***

### エンティティスキーマを取得する
{:.badge .dbless}

エンティティのスキーマを取得します。これは、エンティティ受け入れるフィールドを理解するのに役立ち、Kongへのサードパーティ統合を構築するために使用できます。
<div class="endpoint get">/schemas/\{entity name\}</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "fields": [
        {
            "id": {
                "auto": true,
                "type": "string",
                "uuid": true
            }
        },
        {
            "created_at": {
                "auto": true,
                "timestamp": true,
                "type": "integer"
            }
        },
        ...
    ]
}
```

*** ** * ** ***

### プラグインスキーマを取得
{:.badge .dbless}

プラグイン構成のスキーマを取得します。これは、プラグインが受け入れるフィールドを理解するのに役立ち、Kongのプラグインシステムにサードパーティ統合を構築するのに使用できます。
<div class="endpoint get">/schemas/plugins/\{plugin name\}</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "fields": {
        "hide_credentials": {
            "default": false,
            "type": "boolean"
        },
        "key_names": {
            "default": ["apikey"],
            "required": true,
            "type": "array"
        }
    }
}
```

*** ** * ** ***

### スキーマに対するプラグイン構成の検証
{:.badge .dbless}

プラグインエンティティスキーマに対してプラグイン構成の有効性を確認します。これにより、Admin APIのエンティティエンドポイントにリクエストを送信する前に入力内容をテストできます。

これは、入力の構成が正しいことをチェックするスキーマ検証チェックのみを実行することに注意してください。指定された構成を使用するエンティティエンドポイントへのリクエストは、無効な外部キー関係や、データストアの内容に対する一意性チェックのエラーなど、他の理由で失敗する可能性があります。
<div class="endpoint post">/schemas/plugins/validate</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "message": "schema validation successful"
}
```

*** ** * ** ***

### Kongのタイマーのランタイムデバッグ情報の取得
{:.badge .dbless}

[lua\-resty\-timer\-ng](https://github.com/Kong/lua-resty-timer-ng)からランタイム統計データを取得します。
<div class="endpoint post">/timers</div> 

#### レスポンス

    HTTP 200 OK

```json
{   "worker": {
      "id": 0,
      "count": 4,
    },
    "stats": {
      "flamegraph": {
        "running": "@./kong/init.lua:706:init_worker();@./kong/runloop/handler.lua:1086:before() 0\n",
        "elapsed_time": "@./kong/init.lua:706:init_worker();@./kong/runloop/handler.lua:1086:before() 17\n",
        "pending": "@./kong/init.lua:706:init_worker();@./kong/runloop/handler.lua:1086:before() 0\n"
      },
      "sys": {
          "running": 0,
          "runs": 7,
          "pending": 0,
          "waiting": 7,
          "total": 7
      },
      "timers": {
          "healthcheck-localhost:8080": {
              "name": "healthcheck-localhost:8080",
              "meta": {
                  "name": "@/build/luarocks/share/lua/5.1/resty/counter.lua:71:new()",
                  "callstack": "@./kong/plugins/prometheus/prometheus.lua:673:init_worker();@/build/luarocks/share/lua/5.1/resty/counter.lua:71:new()"
              },
              "stats": {
                  "finish": 2,
                  "runs": 2,
                  "elapsed_time": {
                      "min": 0,
                      "max": 0,
                      "avg": 0,
                      "variance": 0
                  },
                  "last_err_msg": ""
              }
          }
      }
    }
}
```

* `worker`：
  * `id`: 現在のNginxワーカープロセスの序数（0から始まります）。
  * `count`: Nginxワーカープロセスの合計数。

* `stats.flamegraph`：文字列エンコードされたタイマー関連のフレームグラフデータ。 [brendangregg/FlameGraph](https://github.com/brendangregg/FlameGraph)を使ってフレームグラフSVGを生成できます。
* `stats.sys`：さまざまなタイプのタイマーの数を一覧表示します。 
  * `running`: 実行中のタイマーの数。
  * `pending`: 保留中のタイマーの数。
  * `waiting`: 有効期限が切れていないタイマーの数。
  * `total`：実行中 \+ 保留中 \+ 待機中。

* `timers.meta`: 作成されたタイマーのプログラム呼び出しスタック。 
  * `name`： 作成タイマーが作成された場所を格納する、自動的に生成された文字列。
  * `callstack`：このタイマーが作成された場所を示すLuaコールスタック文字列。

* `timers.stats.elapsed_time`: タイマーの各実行にかかった時間（秒）の最大値、最小値、平均値、および分散を格納するオブジェクト。
* `timers.stats.runs`: 実行の合計数。
* `timers.stats.finish`：成功した実行の合計数。

注：`flamegraph`、`timers.meta`、`timers.stats.elapsed_time`キーは、Kong の`log_level`構成が`debug`に設定されている場合にのみ使用できます。
詳細については、[lua\-resty\-timer\-ngのドキュメント](https://github.com/Kong/lua-resty-timer-ng#stats)をご覧ください。

*** ** * ** ***

ヘルスルート
------

### ノードステータスの取得
{:.badge .dbless}

ノードに関する使用状況情報とともに、基盤となる nginx プロセスによって処理されている接続、データベース接続のステータス、ノードのメモリ使用量に関する基本的な情報を取得します。

Kongのプロセスを監視する場合、Kongはnginxの上に構築されているため、既存のすべてのnginx監視ツールまたはエージェントを使用できます。
<div class="endpoint get">/status</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "database": {
      "reachable": true
    },
    "memory": {
        "workers_lua_vms": [{
            "http_allocated_gc": "0.02 MiB",
            "pid": 18477
          }, {
            "http_allocated_gc": "0.02 MiB",
            "pid": 18478
        }],
        "lua_shared_dicts": {
            "kong": {
                "allocated_slabs": "0.04 MiB",
                "capacity": "5.00 MiB"
            },
            "kong_db_cache": {
                "allocated_slabs": "0.80 MiB",
                "capacity": "128.00 MiB"
            },
        }
    },
    "server": {
        "total_requests": 3,
        "connections_active": 1,
        "connections_accepted": 1,
        "connections_handled": 1,
        "connections_reading": 0,
        "connections_writing": 1,
        "connections_waiting": 0
    },
    "configuration_hash": "779742c3d7afee2e38f977044d2ed96b"
}
```

* `memory`：メモリ使用量に関するメトリクス。
  * `workers_lua_vms`：Kongノードのすべてのワーカーを含む配列で、各エントリには以下が含まれます。
  * `http_allocated_gc`: `collectgarbage("count")`によって報告された、HTTPサブモジュールのLua仮想マシンの、アクティブなワーカーごとのメモリ使用情報。つまり過去10秒以内にプロキシ呼び出しを受信したワーカーです。
  * `pid`：ワーカーのプロセス識別番号。
  * `lua_shared_dicts`：Kongノード内のすべてのワーカーと共有されるディクショナリに関する情報の配列で、各配列ノードには、特定の共有ディクショナリ専用のメモリ量（`capacity`）と、そのメモリの使用量（`allocated_slabs`）が含まれています。これらの共有ディクショナリには、最近使用されていない（LRU）エビクション機能があるため、完全なディクショナリ（ `allocated_slabs == capacity`の場合）は適切に機能します。ただし、一部のディクショナリ、たとえば、HIT/MISS共有ディクショナリをキャッシュし、サイズを大きくすると、Kongノードの全体的なパフォーマンスに有益です。
  * メモリ使用量の単位と精度は、クエリ文字列の引数`unit`と`scale`を使用して変更できます。
    * `unit`: `b/B`、`k/K`、`m/M`、`g/G`のいずれかで、それぞれ、byte、kibibyte、mebibyte、gibibyteで結果を返します。「byte」がリクエストされた場合、応答のメモリ値は文字列ではなく数値型になります。デフォルトは`m`です。
    * `scale`：値が、人間が読めるメモリ文字列（「byte」以外の単位）で指定されている場合の小数点の右側の桁数。デフォルトは`2`です。次の方法で、共有ディクショナリのメモリ使用量を4桁の精度（kibibyte単位）で取得します。`GET /status?unit=k&scale=4`

* `server`: nginx HTTP/Sサーバーについてのメトリクス。
  * `total_requests`: クライアントリクエストの合計数。
  * `connections_active`: 待機中の接続を含む 、現在アクティブなクライアントの接続数。
  * `connections_accepted`：受け入れられたクライアントの接続の合計数。
  * `connections_handled`：処理された接続の合計数。通常、リソース制限に達していないかぎり、パラメータ値は受け入れられる値と同じです。
  * `connections_reading`：Kongがリクエストヘッダーを読み取っている現在の接続数。
  * `connections_writing`：nginxがレスポンスをクライアントに 書き戻している現在の接続数。
  * `connections_waiting`：リクエストを待機しているアイドル状態のクライアント接続の現在の数。

* `database`：データベースに関するメトリクス。
  * `reachable`：データベース接続の状態を反映する ブール値。このフラグは、データベース自体の正常性を 反映して **いない** ことに注意してください。

* `configuration_hash`: 現在の構成のハッシュ。このフィールドは、Kong ノードが DB レスモードまたはデータプレーン（DP）で実行されている場合にのみ返されます。特別な戻り値「00000000000000000000000000000000」は、Kong に現在有効な構成が読み込まれていないことを意味します。

*** ** * ** ***

タグ
---

タグは、Kongのエンティティに関連付けられた文字列です。

タグには、ほぼすべてのUTF\-8文字を含めることができますが、次の例外があります。

* `,`および`/`は、「and」と「or」を含むタグをフィルタリングするために予約されているため、タグでは使用できません。
* 印刷不能なASCII文字（スペース文字など）は使用できません。

ほとんどのコアエンティティは、作成時または編集時に`tags`属性で *タグ付け* できます。

タグは、`?tags`クエリ文字列パラメータを介して、コアエンティティをフィルタリングするのにも使用できます。

たとえば、通常は以下を実行してすべてのサービスのリストを取得します。

    GET /services

`example`タグが付けられたすべてのサービスを一覧表示するには、次のようにします。

    GET /services?tags=example

動揺に、タグされた`example` *および* `admin`のみを取得するよう、サービスをフィルタリングする場合は、以下を実行します。

    GET /services?tags=example,admin

最後に、`example` *または* `admin`とタグ付けされたサービスをフィルタリングする場合は、次を使用できます。

    GET /services?tags=example/admin

注意点：

* 次を使用して、1 度のリクエストで最大 5 つのタグを同時にクエリできます: `,` または `/`
* 演算子の混在はサポートされていません。同じクエリ文字列で`,`と`/`を混在させようとすると、エラーが発生します。
* コマンドラインから使用する場合、一部の文字を引用符で囲んだり、エスケープする必要がある場合があります。
* `tags` によるフィルタリングは、外部キー関係エンドポイントではサポートされていません。例えば、 `tags` パラメータは、次のようなリクエストでは無視されます。`GET /services/foo/routes?tags=a,b`
* `offset`パラメータは、`tags`パラメータが変更または削除された場合、動作が保証されません

### すべてのタグの一覧表示
{:.badge .dbless}

システム内のすべてのタグのページ分割されたリストを返します。

エンティティの一覧は、単一のエンティティタイプに限定されません。タグでタグ付けされたすべてのエンティティがこの一覧に表示されます。

エンティティに複数のタグが付けられている場合、そのエンティティの`entity_id`が結果の一覧に複数回表示されます。同様に、複数のエンティティに同じタグが付けられている場合、そのタグはこの一覧の複数の項目に表示されます。
<div class="endpoint get">/tags</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    {
      "data": [
        { "entity_name": "services",
          "entity_id": "acf60b10-125c-4c1a-bffe-6ed55daefba4",
          "tag": "s1",
        },
        { "entity_name": "services",
          "entity_id": "acf60b10-125c-4c1a-bffe-6ed55daefba4",
          "tag": "s2",
        },
        { "entity_name": "routes",
          "entity_id": "60631e85-ba6d-4c59-bd28-e36dd90f6000",
          "tag": "s1",
        },
        ...
      ],
      "offset": "c47139f3-d780-483d-8a97-17e9adc5a7ab",
      "next": "/tags?offset=c47139f3-d780-483d-8a97-17e9adc5a7ab",
    }
}
```

*** ** * ** ***

### タグ別にエンティティIDを一覧表示
{:.badge .dbless}

指定タグでタグ付けされているエンティティを返します。

エンティティの一覧は、単一のエンティティタイプに限定されません。タグでタグ付けされたすべてのエンティティがこの一覧に表示されます。
<div class="endpoint get">/tags/\{tags\}</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    {
      "data": [
        { "entity_name": "services",
          "entity_id": "c87440e1-0496-420b-b06f-dac59544bb6c",
          "tag": "example",
        },
        { "entity_name": "routes",
          "entity_id": "8a99e4b1-d268-446b-ab8b-cd25cff129b1",
          "tag": "example",
        },
        ...
      ],
      "offset": "1fb491c4-f4a7-4bca-aeba-7f3bcee4d2f9",
      "next": "/tags/example?offset=1fb491c4-f4a7-4bca-aeba-7f3bcee4d2f9",
    }
}
```

*** ** * ** ***

ルートのデバッグ
--------

### すべてのコントロールプレーン（CP）ノードのノードログレベルを設定する

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

ハイブリッド（CP/DP）クラスタにデプロイされているすべてのコントロールプレーン（CP）ノードのログレベルを変更します。

使用可能な値の一覧については、[http://nginx.org/en/docs/ngx\_core\_module.html\#error\_log](http://nginx.org/en/docs/ngx_core_module.html#error_log)
を参照してください。

本番環境でノードのログレベルを `debug` に変更する場合は、ディスクがすぐにいっぱいになる可能性があるため、注意が必要です。デバッグログが終了したら、すぐに`notice`などの上位レベルに戻してください。

現在、DP および DB レスノードのログレベルを変更することはできません。

{{site.ee_product_name}}を使用する場合、このエンドポイントは[RBACで保護](/gateway/latest/admin-api/rbac/reference/#add-a-role-endpoint-permission)できます

{{site.ee_product_name}}を使用する場合、ログレベルの変更は[監査ログ](/gateway/latest/kong-enterprise/audit-log/)に反映されます。

ログレベルの変更は、新しく生成されたワーカーを含めて、ノードのすべてのNginxワーカーに伝達されます。

現在、ユーザーがクラスタ全体のログレベルを動的に変更する場合、新しいノードがクラスタに参加すると、新しいノードは以前クラスタ全体に対して動的に設定されたログレベルではなく、以前のログレベルで実行されます。これを回避するには、起動の `kong.conf` 設定を [KONG\_LOG\_LEVEL](/gateway/latest/reference/configuration/#log_level) に設定して、新しいノードが適切なレベルで開始することを確認します。
<div class="endpoint put indent">/debug/cluster/control\-planes\-nodes/log\-level/\{log\_level\}</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "message": "log level changed"
}
```

*** ** * ** ***

### すべてのノードのノードログレベルの設定

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

クラスタ内のすべてのノードのログレベルを変更します。

使用可能な値の一覧については、[http://nginx.org/en/docs/ngx\_core\_module.html\#error\_log](http://nginx.org/en/docs/ngx_core_module.html#error_log)
を参照してください。

本番環境でノードのログレベルを `debug` に変更する場合は、ディスクがすぐにいっぱいになる可能性があるため、注意が必要です。デバッグログの取得が終了したら、すぐに `notice` などの上位レベルに戻してください。

現在、DP および DB レスノードのログレベルを変更することはできません。

{{site.ee_product_name}}を使用する場合、このエンドポイントは[RBACで保護](/gateway/latest/admin-api/rbac/reference/#add-a-role-endpoint-permission)できます

{{site.ee_product_name}}を使用する場合、ログレベルの変更は[監査ログ](/gateway/latest/kong-enterprise/audit-log/)に反映されます。

ログレベルの変更は、新しく生成されたワーカーを含めて、ノードのすべてのNginxワーカーに伝達されます。

現在、ユーザーがクラスタ全体のログレベルを動的に変更する場合、新しいノードがクラスタに参加すると、新しいノードは以前クラスタ全体に対して動的に設定されたログレベルではなく、以前のログレベルで実行されます。これを回避するには、起動の `kong.conf` 設定を [KONG\_LOG\_LEVEL](/gateway/latest/reference/configuration/#log_level) に設定して、新しいノードが適切なレベルで開始することを確認します。
<div class="endpoint put indent">/debug/node/log\-level/\{log\_level\}</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "message": "log level changed"
}
```

*** ** * ** ***

### ノードのノードログレベルの取得
{:.badge .dbless}

ノードの現在のログレベルを取得します。

可能な戻り値のリストについては、[http://nginx.org/en/docs/ngx\_core\_module.html\#error\_log](http://nginx.org/en/docs/ngx_core_module.html#error_log)を参照してください。
<div class="endpoint get">/debug/node/log\-level</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "message": "log level: debug"
}
```

*** ** * ** ***

### 単一ノードのログレベルの設定

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

ノードのログレベルを変更します。

使用可能な値の一覧については、[http://nginx.org/en/docs/ngx\_core\_module.html\#error\_log](http://nginx.org/en/docs/ngx_core_module.html#error_log)
を参照してください。

本番環境でノードのログレベルを `debug` に変更する場合は、ディスクがすぐにいっぱいになる可能性があるため、注意が必要です。デバッグログが終了したら、すぐに`notice`などの上位レベルに戻してください。

現在、DP および DB レスノードのログレベルを変更することはできません。

{{site.ee_product_name}}を使用する場合、このエンドポイントは[RBACで保護](/gateway/latest/admin-api/rbac/reference/#add-a-role-endpoint-permission)できます

{{site.ee_product_name}}を使用する場合、ログレベルの変更は[監査ログ](/gateway/latest/kong-enterprise/audit-log/)に反映されます。

ログレベルの変更は、新しく生成されたワーカーを含めて、ノードのすべてのNginxワーカーに伝達されます。
<div class="endpoint put indent">/debug/node/log\-level/\{log\_level\}</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "message": "log level changed"
}
```

*** ** * ** ***

サービスオブジェクト
----------

サービスエンティティは、その名のとおり、独自のアップストリームサービスのそれぞれを抽象化したものです。サービスの例としては、データ変換マイクロサービス、課金 API などがあります。

サービスの主な属性はURL（Kongがトラフィックをプロキシする場所）で、
単一の文字列として設定することも、`protocol`、`host`、
`port`、および`path`を個別に指定することもできます。

サービスはルートに関連付けられます（1つのサービスに複数のルートが関連付けられる場合があります）。ルートはKongのエントリポイントであり、クライアントリクエストに一致するルールを定義します。ルートが一致すると、Kongは、リクエストを関連付けられたサービスにプロキシします。Kongがトラフィックをプロキシする方法の詳細については、[プロキシリファレンス](/gateway/{{page.release}}/how-kong-works/routing-traffic/)を参照してください。

サービスは、[タグ付けすることも、タグでフィルタリングすること](#tags)もできます。

```json

{{ page.service_json }}
```

### サービスの追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### サービスを作成する

<div class="endpoint post indent">{{ prefix }}/services</div> 

##### 特定の証明書に関連付けられたサービスの作成

<div class="endpoint post indent">/certificates/\{certificate name or id\}/services</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate name or id`<br> **必須** \| 新しく作成されたサービスに関連付ける証明書の一意の識別子または `name` 属性。

#### リクエストボディ


{{ page.service_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.service_json }}
```

*** ** * ** ***

### サービスの一覧表示
{:.badge .dbless}

##### すべてのサービスの一覧化

<div class="endpoint get indent">{{ prefix }}/services</div> 

##### 特定の証明書に関連付けられたサービスの一覧表示

<div class="endpoint get indent">/certificates/\{certificate name or id\}/services</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate name or id`<br> **必須** \| サービスを取得する証明書の一意の識別子または`name`属性。 このエンドポイントを使用すると、指定された証明書に関連付けられたサービスのみがリストされます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.service_data }}
    "next": "http://localhost:8001/services?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### サービスの取得
{:.badge .dbless}

##### サービスの取得

<div class="endpoint get indent">{{ prefix }}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 取得するサービスの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたサービスを取得

<div class="endpoint get indent">/certificates/\{certificate id\}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 取得する証明書の一意の識別子。
`service name or id`<br> **必須** \| 取得するサービスの一意の識別子 **または** 名前。

##### 特定のルートに関連付けられたサービスの取得

<div class="endpoint get indent">/routes/\{route name or id\}/service</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 取得するサービスに関連付けられたルートの一意の識別子 **または** 名前。

##### 特定のプラグインに関連付けられたサービスの取得

<div class="endpoint get indent">/plugins/\{plugin id\}/service</div> 

{:.indent}
属性 \| 説明
\-\-\-: \|\-\-\-
`plugin id`<br> **必須** \| 取得するサービスに関連付けられたプラグインの一意の識別子。

#### レスポンス

    HTTP 200 OK

```json

{{ page.service_json }}
```

*** ** * ** ***

### サービスの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### サービスの更新

<div class="endpoint patch indent">/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 更新するサービスの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたサービスを更新

<div class="endpoint patch indent">/certificates/\{certificate id\}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 更新する証明書の一意の識別子。
`service name or id`<br> **必須** \| 更新するサービスの一意の識別子 **または** 名前。

##### 特定のルートに関連付けられたサービスの更新

<div class="endpoint patch indent">/routes/\{route name or id\}/service</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 更新するサービスに関連付けられたルートの一意の識別子 **または** 名前。

##### 特定のプラグインに関連付けられたサービスの更新

<div class="endpoint patch indent">/plugins/\{plugin id\}/service</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 更新するサービスに関連付けられたプラグインの一意の識別子。

#### リクエストボディ


{{ page.service_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.service_json }}
```

*** ** * ** ***

### サービスを更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### サービスを作成または更新

<div class="endpoint put indent">{{ prefix }}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 作成または更新するサービスの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたサービスの作成または更新

<div class="endpoint put indent">/certificates/\{certificate id\}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 作成または更新する証明書の一意の識別子。
`service name or id`<br> **必須** \| 更新するサービスの一意の識別子 **または** 名前。

##### 特定のルートに関連付けられたサービスの作成または更新

<div class="endpoint put indent">/routes/\{route name or id\}/service</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 作成または更新するサービスに関連付けられたルートの一意の識別子 **または** 名前。

##### 特定のプラグインに関連付けられたサービスの作成または更新

<div class="endpoint put indent">/plugins/\{plugin id\}/service</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 作成または更新するサービスに関連付けられたプラグインの一意の識別子。

#### リクエストボディ


{{ page.service_body }}

リクエストされたリソースの下に、本文で指定された定義のサービスを挿入（または置換）します。サービスは、`name or id`属性によって識別されます。

`name or id` 属性が UUID の構造を持つ場合、挿入/置換されるサービスは、その `id` によって識別されます。それ以外の場合は、その `name` で識別されます。

`id`をURL内およびボディ内のどちらにも指定せずに新しいサービスを作成する場合、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### サービスの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### サービスの削除

<div class="endpoint delete indent">{{ prefix }}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 削除するサービスの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたサービスの削除

<div class="endpoint delete indent">/certificates/\{certificate id\}/services/\{service name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 削除する証明書の一意の識別子。
`service name or id`<br> **必須** \|削除するサービスの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

ルートオブジェクト
---------

ルートエンティティは、クライアントリクエストに一致するルールを定義します。各ルートはサービスに関連付けられており、1つのサービスには複数のルートが関連付けられている場合があります。特定のルートに一致するすべてのリクエストは、関連するサービスにプロキシされます。

ルートとサービスの組み合わせ（およびそれらの間の関係の分離）により、強力なルーティングメカニズムが確立されます。このメカニズムを使用すると、インフラストラクチャのさまざまなアップストリームサービスにつながるきめ細かいエントリポイントをKongで定義できます。

Route によって照合されるプロトコルに適用される照合ルールが少なくとも 1 つ必要です。ルートと一致するように構成されたプロトコル（`protocols` フィールドで定義）に応じて、次のアトリビュートのうち少なくとも1つを設定する必要があります。

* `http` の場合、少なくとも `methods`、`hosts`、`headers` または `paths` のいずれか 1 つ。
* `https` の場合、`methods`、`hosts`、`headers`、`paths`、または `snis` のいずれか 1 つ以上。
* `tcp`の場合、少なくとも`sources`または`destinations`のいずれか1つ。
* `tls` の場合、少なくとも `sources`、`destinations`、または `snis` のいずれか 1 つ。
* `tls_passthrough` の場合は `snis` を設定します。
* `grpc` の場合、少なくとも `hosts`、`headers`、または `paths` のいずれか 1 つ。
* `grpcs` の場合、少なくとも `hosts`、`headers`、`paths` または `snis` のいずれか 1 つ。
* `ws`の場合、少なくとも`hosts` 、 `headers` 、または`paths`のいずれか 1 つ。<span class="badge enterprise"></span>
* `wss`の場合、少なくとも`hosts`、`headers`、`paths`、または`snis`のいずれか1つ。<span class="badge enterprise"></span>

ルートが`tls`プロトコルと`tls_passthrough`プロトコルの両方を同時に持つことはできません。

3\.0\.x リリースでは、新しいルーター実装である `atc-router`が導入されました。
ルーターは以下を追加します:

* Kongの構成変更時のルーター再構築時間の短縮
* リクエストをルーティングする際のランタイムパフォーマンスの向上
* 10,000ルートでP99レイテンシを1\.5秒から0\.1秒に短縮

ルーターの詳細については、以下を確認してください。

[式を使用してルートを構成する](/gateway/{{page.release}}/key-concepts/routes/expressions)
[式言語リファレンス](/gateway/{{page.release}}/reference/expressions-language/)

#### パス処理アルゴリズム

{:.note}
> 
> **注** : パス処理アルゴリズム v1 は Kong 3\.0 で非推奨になりました。Kong3\.0 以降、`router_flavor` が
> `expressions` に設定されると、`route.path_handling` は設定できず、パス処理の動作は
> `"v0"` になります。`router_flavor` が `traditional_compatible` に設定されると、パス処理の
> 動作は `route.path_handling` の値に関係なく `"v0"` になります。`router_flavor` = `traditional` のみ、
> `path_handling` `"v1'` の動作をサポートします。

`"v0"`はKong 0\.x、2\.x、3\.xで使われている動作です。`service.path` 、 `route.path` 、リクエストパスを次のように扱います。
URL の *セグメント* 。 常にスラッシュでそれらを結合します。 サービスパス `/s`、ルートパス `/r` が与えられました
とリクエストパス `/re`、連結されたパスは `/s/re` になります。 結果のパスが単一のスラッシュの場合、
それ以上の変換は行われません。 それより長い場合は、末尾のスラッシュは削除されます。

`"v1"` は、Kong 1\.x で使用される動作です。`service.path` は *プレフィックス* として扱われ、リクエストパスとルートパスの頭のスラッシュは無視されます。サービスパスが `/s`、ルートパスが `/r`、リクエストパスが `/re`
である場合、連結されたパスは `/sre` となります。

どちらのバージョンのアルゴリズムも、パスを組み合わせるときに「二重スラッシュ」を検出し、それらをシングルスラッシュに置き換えます。

次の表は、パス処理バージョン、ストリップパス、およびリクエストの可能な組み合わせを示しています。

| `service.path` | `route.path` | `request` | `route.strip_path` | `route.path_handling` |  リクエストパス   |  アップストリームパス  |
|----------------|--------------|-----------|--------------------|-----------------------|------------|--------------|
| `/s`           | `/fv0`       | `req`     | `false`            | `v0`                  | `/fv0/req` | `/s/fv0/req` |
| `/s`           | `/fv0`       | `blank`   | `false`            | `v0`                  | `/fv0`     | `/s/fv0`     |
| `/s`           | `/fv1`       | `req`     | `false`            | `v1`                  | `/fv1/req` | `/sfv1/req`  |
| `/s`           | `/fv1`       | `blank`   | `false`            | `v1`                  | `/fv1`     | `/sfv1`      |
| `/s`           | `/tv0`       | `req`     | `true`             | `v0`                  | `/tv0/req` | `/s/req`     |
| `/s`           | `/tv0`       | `blank`   | `true`             | `v0`                  | `/tv0`     | `/s`         |
| `/s`           | `/tv1`       | `req`     | `true`             | `v1`                  | `/tv1/req` | `/s/req`     |
| `/s`           | `/tv1`       | `blank`   | `true`             | `v1`                  | `/tv1`     | `/s`         |
| `/s`           | `/fv0/`      | `req`     | `false`            | `v0`                  | `/fv0/req` | `/s/fv0/req` |
| `/s`           | `/fv0/`      | `blank`   | `false`            | `v0`                  | `/fv0/`    | `/s/fv01/`   |
| `/s`           | `/fv1/`      | `req`     | `false`            | `v1`                  | `/fv1/req` | `/sfv1/req`  |
| `/s`           | `/fv1/`      | `blank`   | `false`            | `v1`                  | `/fv1/`    | `/sfv1/`     |
| `/s`           | `/tv0/`      | `req`     | `true`             | `v0`                  | `/tv0/req` | `/s/req`     |
| `/s`           | `/tv0/`      | `blank`   | `true`             | `v0`                  | `/tv0/`    | `/s/`        |
| `/s`           | `/tv1/`      | `req`     | `true`             | `v1`                  | `/tv1/req` | `/sreq`      |
| `/s`           | `/tv1/`      | `blank`   | `true`             | `v1`                  | `/tv1/`    | `/s`         |

ルートは[タグ付けすることも、タグでフィルタリングすること](#tags)もできます。

```json

{{ page.route_json }}
```

### ルートの追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### ルートの作成

<div class="endpoint post indent">{{ prefix }}/routes</div> 

##### 特定のサービスに関連付けられたルートの作成

<div class="endpoint post indent">{{ prefix }}/services/\{service name or id\}/routes</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 新しく作成されたルートに関連付けるサービスの一意の識別子または`name`属性。

#### リクエストボディ


{{ page.route_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.route_json }}
```

*** ** * ** ***

### ルートの一覧化
{:.badge .dbless}

##### すべてのルートの一覧表示

<div class="endpoint get indent">{{ prefix }}/routes</div> 

##### 特定のサービスに関連付けられたルートの一覧表示

<div class="endpoint get indent">{{ prefix }}/services/\{service name or id\}/routes</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| ルートを取得するサービスの一意の識別子または`name`属性。このエンドポイントを使用すると、指定されたサービスに関連付けられたルートのみが一覧表示されます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.route_data }}
    "next": "http://localhost:8001/routes?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### ルートの取得
{:.badge .dbless}

##### ルートの取得

<div class="endpoint get indent">{{ prefix }}/routes/\{route name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 取得するルートの一意の識別子 **または** 名前。

##### 特定のサービスに関連付けられたルートを取得

<div class="endpoint get indent">{{ prefix }}/services/\{サービス名またはID\}/routes/\{ルート名またはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`service name or id`<br> **必須** \| 取得するサービスの一意の識別子 **または** 名前。`route name or id`<br> **必須** \| 取得するルートの一意の識別子 **または** 名前。

##### 特定のプラグインに関連付けられたルートの取得

<div class="endpoint get indent">/plugins/\{plugin id\}/route</div> 

{:.indent}
属性 \| 説明
\-\-\-: \|\-\-\-
`plugin id` <br> **必須** \| 取得するルートに関連付けられたプラグインの一意の識別子。

#### レスポンス

    HTTP 200 OK

```json

{{ page.route_json }}
```

*** ** * ** ***

### ルートの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### ルートの更新

<div class="endpoint patch indent">/routes/\{route name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 更新するルートの一意の識別子 **または** 名前。

##### 特定のサービスに関連付けられたルートの更新

<div class="endpoint patch indent">/services/\{service name or id\}/routes/\{route name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 更新するサービスの一意の識別子 **または** 名前。
`route name or id`<br> **必須** \| 更新するルート一意の識別子 **または** 名前。

##### 特定のプラグインに関連付けられたルートの更新

<div class="endpoint patch indent">/plugins/\{plugin id\}/route</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 更新するルートに関連付けられたプラグインの一意の識別子。

#### リクエストボディ


{{ page.route_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.route_json }}
```

*** ** * ** ***

### ルートの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### ルートの作成または更新

<div class="endpoint put indent">{{ prefix }}/routes/\{route name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 作成または更新するルートの一意の識別子 **または** 名前。

##### 特定のサービスに関連付けられたルートを作成または更新

<div class="endpoint put indent">{{ prefix }}/services/\{サービス名またはID\}/routes/\{ルート名またはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 作成または更新するサービスの一意の識別子 **または** 名前。
`route name or id`<br> **必須** \| 作成または更新するルートの一意の識別子 **または** 名前。

##### 特定のプラグインに関連付けられたルートの作成または更新

<div class="endpoint put indent">/plugins/\{plugin id\}/route</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 作成または更新するルートに関連付けられたプラグインの一意の識別子。

#### リクエストボディ


{{ page.route_body }}

リクエストされたリソースの下に、本文で指定された定義のルートを
挿入（または置換）します。ルートは `name or id` 属性によって識別されます。

`name or id` 属性が UUID の構造を持つ場合、挿入または置換されるルートは、その `id` によって識別されます。それ以外の場合は、その `name` で識別されます。

`id`を指定せずに（URLでも本文でも）新しいルートを作成すると、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### ルートの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### ルートの削除

<div class="endpoint delete indent">{{ prefix }}/routes/\{route name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`route name or id`<br> **必須** \| 削除するルートの一意の識別子 **または** 名前。

##### 特定のサービスに関連付けられたルートの削除

<div class="endpoint delete indent">/services/\{service name or id\}/routes/\{route name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 削除するサービスの一意の識別子 **または** 名前。`route name or id`<br> **必須** \| 削除するルートの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

コンシューマオブジェクト
------------

コンシューマオブジェクトは、サービスのコンシューマ、つまりユーザーを表します。Kongをプライマリデータストアとして利用するか、コンシューマリストをデータベースとマッピングして、Kongと既存のプライマリデータストア間の一貫性を保つことができます。

コンシューマは、[タグ付けすることも、タグでフィルタリングすることも](#tags)できます。

```json

{{ page.consumer_json }}
```

### コンシューマを追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### コンシューマの作成

<div class="endpoint post indent">{{ prefix }}/consumers</div> 

#### リクエストボディ


{{ page.consumer_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.consumer_json }}
```

*** ** * ** ***

### コンシューマの一覧表示
{:.badge .dbless}

##### すべてのコンシューマの一覧表示

<div class="endpoint get indent">{{ prefix }}/consumers</div> 

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.consumer_data }}
    "next": "http://localhost:8001/consumers?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### コンシューマの取得
{:.badge .dbless}

##### コンシューマの取得

<div class="endpoint get indent">{{ prefix }}/consumers/\{consumer username or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 取得するコンシューマの一意の識別子 **または** ユーザー名。

##### 特定のプラグインに関連付けられたコンシューマを取得

<div class="endpoint get indent">/plugins/\{plugin id\}/consumer</div> 

{:.indent}
属性 \| 説明
\-\-\-: \|\-\-\-
`plugin id` <br> **必須** \| 取得するコンシューマに関連付けられているプラグインの一意の識別子。

#### レスポンス

    HTTP 200 OK

```json

{{ page.consumer_json }}
```

*** ** * ** ***

### コンシューマの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### コンシューマの更新

<div class="endpoint patch indent">/consumers/\{consumer username or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 更新するコンシューマの一意の識別子 **または** ユーザー名。

##### 特定のプラグインに関連付けられたコンシューマを更新する

<div class="endpoint patch indent">/plugins/\{plugin id\}/consumer</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 更新するコンシューマに関連付けられたプラグインの一意の識別子。

#### リクエストボディ


{{ page.consumer_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.consumer_json }}
```

*** ** * ** ***

### コンシューマの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### コンシューマの作成または更新

<div class="endpoint put indent">{{ prefix }}/consumers/\{consumer username or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 作成または更新するコンシューマの一意の識別子 **または** ユーザー名。

##### 特定のプラグインに関連付けられたコンシューマの作成または更新

<div class="endpoint put indent">/plugins/\{plugin id\}/consumer</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 作成または更新するコンシューマに関連付けられたプラグインの一意の識別子。

#### リクエストボディ


{{ page.consumer_body }}

要求されたリソースの下のコンシューマを、本文で指定された定義に挿入（または置換）します。コンシューマは`username or id`属性で識別されます。

`username or id` 属性が UUID の構造を持つ場合、挿入/置換されるコンシューマは、その `id` によって識別されます。それ以外の場合は、その `username` で識別されます。

`id`を指定せずに（URLでも本文でも）新しいコンシューマを作成すると、自動生成されます。

URLに `username` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### コンシューマの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### コンシューマの削除

<div class="endpoint delete indent">{{ prefix }}/consumers/\{consumer username or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 削除するコンシューマの一意の識別子 **または** ユーザー名。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

プラグインオブジェクト
-----------

プラグインエンティティは、HTTP リクエスト/レスポンスのライフサイクル中に実行されるプラグイン構成を表します。これは、認証や Rate Limiting など、Kong の背後で実行されるサービスに機能を追加する方法です。インストール方法と各プラグインが受け取る値の詳細については、[Kong Hub](/hub/) を参照してください。

プラグイン構成をサービスに追加すると、サービスに対してクライアントがリクエストをするごとに、そのプラグインが実行されます。特定のコンシューマに対してプラグインを異なる値に調整する必要がある場合、`service` と `consumer` のフィールドでサービスとコンシューマの両方を指定した別のプラグインインスタンスを作成することによって、それが可能となります。

プラグインは[タグ付けすることも、タグでフィルタリングすることもできます](#tags)。

```json

{{ page.plugin_json }}
```

詳細については、以下の[優先順位](#precedence)セクションを参照してください。

#### 優先順位

プラグインは常に1回だけ、リクエストごとに1回だけ実行されます。ただし、
実行される際の構成は、プラグインが構成されているエンティティによって
異なります。

プラグインは、さまざまなエンティティ、エンティティの組み合わせ、またはグローバルにも構成できます。これは、たとえばほとんどのリクエストに対しては特定の方法でプラグインを設定したいけれど、 *認証されたリクエスト* に対しては多少異なる動作にしたい場合に役立ちます。

したがって、プラグインが異なる構成の異なるエンティティに適用された場合は、プラグインを実行する優先順位があります。特定のプラグインに構成されたエンティティの数は、そのプラグインの優先度に直接相関します。プラグインに構成されたエンティティの数が多いほど、優先順位が高くなります。複数のエンティティに構成されたプラグインの完全な優先順位は以下のとおりです。

{% if_version gte:3.4.x %}

1. コンシューマ、ルート、サービスの組み合わせで構成されたプラグイン。 （コンシューマは、リクエストが認証される必要があることを意味します）。
2. ConsumerGroup、サービス、およびルートの組み合わせで構成されたプラグイン。 \(ConsumerGroup は、リクエストが認証される必要があることを意味します\)。
3. コンシューマとルートの組み合わせで構成されたプラグイン。 （コンシューマは、リクエストが認証される必要があることを意味します）。
4. コンシューマとサービスの組み合わせで構成されたプラグイン。
5. コンシューマグループとルートで構成されたプラグイン。
6. コンシューマグループとサービスで構成されたプラグイン。
7. ルートとサービスに設定されたプラグイン。
8. コンシューマ上に構成されるプラグイン。
9. ConsumerGroupに構成されたプラグイン。
10. ルートに設定されたプラグイン。
11. サービス上に構成されるプラグイン。
12. グローバルに構成されたプラグイン。

{% endif_version %}

{% if_version lte:3.3.x %}

1. ルート、サービス、コンシューマの組み合わせで構成されたプラグイン。 （コンシューマは、リクエストが認証される必要があることを意味します）。
2. ルートとコンシューマの組み合わせで構成されたプラグイン。（コンシューマは、リクエストが認証される必要があることを意味します）。
3. サービスとコンシューマの組み合わせで構成されたプラグイン。（コンシューマは、リクエストが認証される必要があることを意味します）。
4. ルートとサービスの組み合わせに設定されたプラグイン。
5. コンシューマで構成されたプラグイン。（コンシューマは、リクエストが認証される必要があることを意味します）。
6. ルートに設定されたプラグイン。
7. サービス上に構成されるプラグイン。
8. グローバルに実行するように構成されたプラグイン。

{% endif_version %}

**例** : `rate-limiting` プラグインが、サービス（プラグイン構成A）、およびコンシューマ（プラグイン構成B）に対して（異なる構成で）2回適用された場合、このコンシューマを認証するリクエストは、プラグイン構成Bを実行しAを無視します。ただし、このコンシューマを認証しないリクエストは、プラグイン構成Aを実行するようフォールバックします。構成Bが無効になっている場合（その `enabled` フラグが `false` に設定されている場合）、構成Aは構成Bに一致するリクエストに適用されることに注意してください。

### プラグインを追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### プラグインの作成

<div class="endpoint post indent">{{ prefix }}/plugins</div> 

##### 特定のルートに関連付けられたプラグインを作成

<div class="endpoint post indent">{{ prefix }}/routes/\{route name or id\}/plugins</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 新しく作成されたプラグインに関連付けるルートの一意の識別子または`name`属性。

##### 特定のサービスに関連付けられたプラグインの作成

<div class="endpoint post indent">{{ prefix }}/services/\{service name or id\}/plugins</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 新しく作成されたプラグインに関連付けるサービスの一意の識別子または`name`属性。

##### 特定のコンシューマに関連付けられたプラグインの作成

<div class="endpoint post indent">{{ prefix }}/consumers/\{consumer name or id\}/plugins</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer name or id`<br> **必須** \| 新しく作成されたプラグインに関連付けるコンシューマの一意の識別子または`name`属性。

#### リクエストボディ


{{ page.plugin_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.plugin_json }}
```

*** ** * ** ***

### プラグインの一覧表示
{:.badge .dbless}

##### すべてのプラグインの一覧

<div class="endpoint get indent">{{ prefix }}/plugins</div> 

##### 特定のルートに関連付けられたプラグインの一覧表示

<div class="endpoint get indent">{{ prefix }}/routes/\{route name or id\}/plugins</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| プラグインを取得するルートの一意の識別子または`name`属性。このエンドポイントを使用すると、指定されたルートに関連付けられたプラグインのみが一覧表示されます。

##### 特定のサービスに関連付けられたプラグインの一覧化

<div class="endpoint get indent">{{ prefix }}/services/\{service name or id\}/plugins</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| プラグインを取得するサービスの一意の識別子または`name`属性。このエンドポイントを使用すると、指定されたサービスに関連付けられたプラグインのみが一覧表示されます。

##### 特定のコンシューマに関連付けられたプラグインの一覧化

<div class="endpoint get indent">{{ prefix }}/consumers/\{consumer name or id\}/plugins</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer name or id`<br> **必須** \| プラグインを取得するコンシューマの一意の識別子または `name` 属性。このエンドポイントを使用すると、指定されたコンシューマに関連付けられたプラグインのみが一覧化されます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.plugin_data }}
    "next": "http://localhost:8001/plugins?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### プラグインの取得
{:.badge .dbless}

##### プラグインの取得

<div class="endpoint get indent">{{ prefix }}/plugins/\{plugin id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 取得するプラグインの一意の識別子。

##### 特定のルートに関連付けられたプラグインの取得

<div class="endpoint get indent">{{ prefix }}/routes/\{ルート名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 取得するルートの一意の識別子 **または** 名前。`plugin id`<br> **必須** \| 取得するプラグインの一意の識別子。

##### 特定のサービスに関連付けられたプラグインを取得

<div class="endpoint get indent">{{ prefix }}/services/\{サービス名またはID）\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 取得するサービスの一意の識別子 **または** 名前。
`plugin id`<br> **必須** \| 取得するプラグインの一意の識別子。

##### 特定のコンシューマに関連付けられたプラグインの取得

<div class="endpoint get indent">{{ prefix }}/consumers/\{コンシューマのユーザー名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 取得するコンシューマの一意の識別子 **または** ユーザー名。
`plugin id`<br> **必須** \| 取得するプラグインの一意の識別子。

#### レスポンス

    HTTP 200 OK

```json

{{ page.plugin_json }}
```

*** ** * ** ***

### プラグインの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### プラグインの更新

<div class="endpoint patch indent">/plugins/\{plugin id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 更新するプラグインの一意の識別子。

##### 特定のプラグインに関連付けられたサービスを更新

<div class="endpoint patch indent">/routes/\{route name or id\}/plugins/\{plugin id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 更新するルートの一意の識別子 **または** 名前。`plugin id`<br> **必須** \| 更新するプラグインの一意の識別子。

##### 特定のサービスに関連付けられたプラグインの更新

<div class="endpoint patch indent">/services/\{サービス名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 更新するサービスの一意の識別子 **または** 名前。
`plugin id`<br> **必須** \| 更新するプラグインの一意の識別子。

##### 特定のコンシューマに関連付けられたプラグインの更新

<div class="endpoint patch indent">/consumers/\{consumer username or id\}/plugins/\{plugin id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 更新するコンシューマの一意の識別子 **または** ユーザー名。
`plugin id`<br> **必須** \| 更新するプラグインの一意の識別子。

#### リクエストボディ


{{ page.plugin_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.plugin_json }}
```

*** ** * ** ***

### プラグインを更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### プラグインの作成または更新

<div class="endpoint put indent">{{ prefix }}/plugins/\{plugin id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 作成または更新するプラグインの一意の識別子。

##### 特定のルートに関連付けられたプラグインを作成または更新

<div class="endpoint put indent">{{ prefix }}/routes/\{ルート名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 作成または更新するルートの一意の識別子 **または** 名前。
`plugin id`<br> **必須** \| 作成または更新するプラグインの一意の識別子。

##### 特定のサービスに関連付けられたプラグインの作成または更新

<div class="endpoint put indent">{{ prefix }}/services/\{サービス名またはID）\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 作成または更新するサービスの一意の識別子 **または** 名前。
`plugin id`<br> **必須** \| 作成または更新するプラグインの一意の識別子。

##### 特定のコンシューマに関連付けられたプラグインの作成または更新

<div class="endpoint put indent">{{ prefix }}/consumers/\{コンシューマのユーザー名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 作成または更新するコンシューマの一意の識別子 **または** ユーザー名。
`plugin id`<br> **必須** \| 作成または更新するプラグインの一意の識別子。

#### リクエストボディ


{{ page.plugin_body }}

ボディで指定された定義を使って、指定されたリソースの下にプラグインを挿入（または置換）します。プラグインは `name or id` 属性によって識別されます。

`name or id` 属性が UUID の構造を持つ場合、挿入または置換されるプラグインは、その `id` によって識別されます。それ以外の場合は、その `name` で識別されます。

`id`を指定せずに（URLでも本文でも）新しいプラグインを作成すると、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### プラグインの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### プラグインの削除

<div class="endpoint delete indent">{{ prefix }}/plugins/\{plugin id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`plugin id`<br> **必須** \| 削除するプラグインの一意の識別子。

##### 特定のルートに関連付けられたプラグインの削除

<div class="endpoint delete indent">{{ prefix }}/routes/\{ルート名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`route name or id`<br> **必須** \| 削除するルートの一意の識別子 **または** 名前。`plugin id`<br> **必須** \| 削除するプラグインの一意の識別子。

##### 特定のサービスに関連付けられたプラグインの削除

<div class="endpoint delete indent">{{ prefix }}/services/\{サービス名またはID）\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 削除するサービスの一意の識別子 **または** 名前。
`plugin id`<br> **必須** \| 削除するプラグインの一意の識別子。

##### 特定のコンシューマに関連付けられたプラグインの削除

<div class="endpoint delete indent">{{ prefix }}/consumers/\{コンシューマのユーザー名またはID\}/plugins/\{プラグインID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`consumer username or id`<br> **必須** \| 削除するコンシューマの一意の識別子 **または** ユーザー名。
`plugin id`<br> **必須** \| 削除するプラグインの一意の識別子。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### 有効なプラグインの取得
{:.badge .dbless}

Kongノードにインストールされたすべてのプラグインのリストを取得します。
<div class="endpoint get">/plugins/enabled</div> 

#### レスポンス

    HTTP 200 OK

```json
{
    "enabled_plugins": [
        "jwt",
        "acl",
        "cors",
        "oauth2",
        "tcp-log",
        "udp-log",
        "file-log",
        "http-log",
        "key-auth",
        "hmac-auth",
        "basic-auth",
        "ip-restriction",
        "request-transformer",
        "response-transformer",
        "request-size-limiting",
        "rate-limiting",
        "response-ratelimiting",
        "aws-lambda",
        "bot-detection",
        "correlation-id",
        "datadog",
        "galileo",
        "ldap-auth",
        "loggly",
        "statsd",
        "syslog"
    ]
}
```

*** ** * ** ***

証明書オブジェクト
---------

証明書オブジェクトは公開証明書を表し、オプションで対応する秘密鍵と組み合わせることができます
。これらのオブジェクトはKongが暗号化されたリクエストのSSL/TLSの終了を処理するために使用されるか、クライアント/サービスのピア証明書を検証する際の信頼できるCAストアとして使用されます。証明書はオプションでSNIオブジェクトに関連付けて、証明書/キーペアを1つ以上のホスト名に結びつけます。

メインの証明書に加えて中間証明書が必要な場合は、次の順序に従って1つの文字列になるように連結する必要があります。メインの証明書が一番上、その後に中間証明書が続きます。

証明書は、[タグ付けすることも、タグでフィルタリングすることもできます](#tags)。

```json

{{ page.certificate_json }}
```

### 証明書の追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### 証明書の作成

<div class="endpoint post indent">{{ prefix }}/certificates</div> 

#### リクエストボディ


{{ page.certificate_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.certificate_json }}
```

*** ** * ** ***

### 証明書を一覧表示
{:.badge .dbless}

##### すべての証明書の一覧表示

<div class="endpoint get indent">{{ prefix }}/certificates</div> 

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.certificate_data }}
    "next": "http://localhost:8001/certificates?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### 証明書の取得
{:.badge .dbless}

##### 証明書の取得

<div class="endpoint get indent">{{ prefix }}/certificates/\{certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 取得する証明書の一意の識別子。

##### 特定のアップストリームに関連付けられた証明書の取得

<div class="endpoint get indent">/upstreams/\{upstream name or id\}/client\_certificate</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 取得する証明書に関連付けられたアップストリームの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json

{{ page.certificate_json }}
```

*** ** * ** ***

### 証明書を更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### 証明書を更新

<div class="endpoint patch indent">/certificates/\{certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 更新する証明書の一意の識別子。

##### 特定のアップストリームに関連付けられた証明書の更新

<div class="endpoint patch indent">/upstreams/\{upstream name or id\}/client\_certificate</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 更新する証明書に関連付けられたアップストリームの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.certificate_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.certificate_json }}
```

*** ** * ** ***

### 証明書の更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### 証明書の作成または更新

<div class="endpoint put indent">{{ prefix }}/certificates/\{certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 作成または更新する証明書の一意の識別子。

##### 特定のアップストリームに関連付けられた証明書の作成または更新

<div class="endpoint put indent">/upstreams/\{upstream name or id\}/client\_certificate</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 作成または更新する証明書に関連付けられたアップストリームの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.certificate_body }}

要求されたリソースの下の証明書を、本文で指定された定義に挿入（または置換）します。証明書は、`name or id`属性によって識別されます。

`name or id` 属性が UUID の構造を持つ場合、挿入/置換される証明書は、その `id` によって識別されます。それ以外の場合は、その `name` で識別されます。

`id`は、新しい証明書の作成時に（URLにもボディにも）指定されない場合は、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### 証明書の削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### 証明書の削除

<div class="endpoint delete indent">{{ prefix }}/certificates/\{certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 削除する証明書の一意の識別子。

##### 特定のアップストリームに関連付けられた証明書の削除

<div class="endpoint delete indent">/upstreams/\{upstream name or id\}/client\_certificate</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 削除する証明書に関連付けられたアップストリームの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

CA証明書オブジェクト
-----------

CA 証明書オブジェクトは、信頼された CA を表します。これらのオブジェクトは、クライアントまたはサーバー証明書の有効性を検証するために Kong によって使用されます。

CA 証明書は[タグ付けすることも、タグでフィルタリングすること](#tags)もできます。

```json

{{ page.ca_certificate_json }}
```

### CA証明書の追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### CA証明書を作成する

<div class="endpoint post indent">{{ prefix }}/ca\_certificates</div> 

#### リクエストボディ


{{ page.ca_certificate_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.ca_certificate_json }}
```

*** ** * ** ***

### CA証明書の一覧表示
{:.badge .dbless}

##### すべてのCA証明書の一覧表示

<div class="endpoint get indent">{{ prefix }}/ca\_certificates</div> 

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.ca_certificate_data }}
    "next": "http://localhost:8001/ca_certificates?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### CA証明書を取得する
{:.badge .dbless}

##### CA証明書を取得する

<div class="endpoint get indent">{{ prefix }}/ca\_certificates/\{ca\_certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`ca_certificate id`<br> **必須** \| 取得するCA証明書の一意の識別子。

#### レスポンス

    HTTP 200 OK

```json

{{ page.ca_certificate_json }}
```

*** ** * ** ***

### CA証明書を更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### CA証明書を更新

<div class="endpoint patch indent">/ca\_certificates/\{ca\_certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`ca_certificate id`<br> **必須** \| 更新する CA 証明書の一意の識別子。

#### リクエストボディ


{{ page.ca_certificate_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.ca_certificate_json }}
```

*** ** * ** ***

### CA 証明書の更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### CA証明書の作成または更新

<div class="endpoint put indent">{{ prefix }}/ca\_certificates/\{ca\_certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`ca_certificate id`<br> **必須** \| 作成または更新するCA証明書の一意の識別子。

#### リクエストボディ


{{ page.ca_certificate_body }}

ボディで指定された定義を使用して、CA証明書をリクエストされたリソースに挿入（または置換）します。CA 証明書は、`name or id`属性を介して識別されます。

`name or id`属性がUUIDの構造を持つ場合、挿入/置換されるCA証明書は、その`id`によって識別されます。それ以外の場合は、その`name`で識別されます。

`id`をURL内およびボディ内のどちらにも指定せずに新しいCA証明書を作成する場合、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### CA証明書を削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### CA証明書を削除

<div class="endpoint delete indent">{{ prefix }}/ca\_certificates/\{ca\_certificate id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`ca_certificate id`<br> **必須** \| 削除するCA証明書の一意の識別子。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

SNIオブジェクト
---------

SNIオブジェクトは、ホスト名と証明書を多対1でマッピングします。
つまり、1つの証明書オブジェクトに複数のホスト名を関連付けることができます。Kongは、SSLリクエストを受信すると、Client HelloのSNIフィールドを使用して、証明書に関連付けられたSNIに基づいて証明書オブジェクトを検索します。

SNIは[タグ付けすることも、タグでフィルタリングすること](#tags)もできます。

```json

{{ page.sni_json }}
```

### SNIの追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### SNIの作成

<div class="endpoint post indent">{{ prefix }}/snis</div> 

##### 特定の証明書に関連付けられた SNI の作成

<div class="endpoint post indent">{{ prefix }}/certificates/\{certificate name or id\}/snis</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate name or id`<br> **必須** \| 新しく作成された SNI に関連付ける証明書の一意の識別子または`name`属性。

#### リクエストボディ


{{ page.sni_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.sni_json }}
```

*** ** * ** ***

### SNIの一覧表示
{:.badge .dbless}

##### すべてのSNIの一覧表示

<div class="endpoint get indent">{{ prefix }}/snis</div> 

##### 特定の証明書に関連付けられたSNIの一覧表示

<div class="endpoint get indent">{{ prefix }}/certificates/\{certificate name or id\}/snis</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate name or id`<br> **必須** \| SNIを取得する証明書の一意の識別子または`name`属性。このエンドポイントを使用すると、指定された証明書に関連付けられたSNIのみがリストされます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.sni_data }}
    "next": "http://localhost:8001/snis?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### SNIの取得
{:.badge .dbless}

##### SNIの取得

<div class="endpoint get indent">{{ prefix }}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`sni name or id`<br> **必須** \| 取得する SNI の一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたSNIの取得

<div class="endpoint get indent">{{ prefix }}/certificates/\{certificate id\}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 取得する証明書の一意の識別子。
`sni name or id`<br> **必須** \| 取得するSNIの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json

{{ page.sni_json }}
```

*** ** * ** ***

### SNIの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### SNIの更新

<div class="endpoint patch indent">/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`sni name or id`<br> **必須** \| 更新するSNIの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたSNIを更新

<div class="endpoint patch indent">/certificates/\{certificate id\}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 更新する証明書の一意の識別子。
`sni name or id`<br> **必須** \| 更新する SNI の一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.sni_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.sni_json }}
```

*** ** * ** ***

### SNIの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### SNIの作成または更新

<div class="endpoint put indent">{{ prefix }}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`sni name or id`<br> **必須** \| 作成または更新する SNI の一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたSNIを作成または更新する

<div class="endpoint put indent">{{ prefix }}/certificates/\{certificate id\}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-：\|\-\-\-
`certificate id`<br> **必須** \| 作成または更新する証明書の一意の識別子。
`sni name or id`<br> **必須** \| 作成または更新するSNIの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.sni_body }}

リクエストされたリソースの下に、本文で指定された定義のSNIを挿入（または置換）します。SNIは、`name or id`属性によって識別されます。

`name or id`属性がUUID構造をとる場合、挿入されるまたは置き換えられるSNIは、その`id`によって識別されます。それ以外の場合は、その`name`で識別されます。

`id` を URL 内およびボディ内のどちらにも指定せずに新しいSNIを作成する場合、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### SNIの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### SNIの削除

<div class="endpoint delete indent">{{ prefix }}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`sni name or id`<br> **必須** \|削除するSNIの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられた SNI の削除

<div class="endpoint delete indent">{{ prefix }}/certificates/\{certificate id\}/snis/\{sni name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 削除する証明書の一意の識別子。
`sni name or id`<br> **必須** \| 削除する SNI の一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

アップストリームオブジェクト
--------------

アップストリームオブジェクトは仮想ホスト名を表し、複数のサービス（ターゲット）にわたる受信リクエストのロードバランシングに使用できます。たとえば、`host` が `service.v1.xyz` であるサービスオブジェクトの `service.v1.xyz` という名前のアップストリームなどです。このサービスに対するリクエストは、アップストリーム内で定義されたターゲットにプロキシされます。

アップストリームには[ヘルスチェッカー](/gateway/{{page.release}}/how-kong-works/health-checks)も含まれており、それを使うとリクエストを処理できるかどうかに基づいてターゲットを有効または無効にすることができます。ヘルスチェッカーの構成は、アップストリームオブジェクトに格納され、そのすべてのターゲットに適用されます。

アップストリームは[タグ付けすることも、タグでフィルタリングすることも](#tags)できます。

```json

{{ page.upstream_json }}
```

### アップストリームの追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### アップストリームの作成

<div class="endpoint post indent">{{ prefix }}/upstreams</div> 

##### 特定の証明書に関連付けられたアップストリームの作成

<div class="endpoint post indent">/certificates/\{certificate name or id\}/upstreams</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate name or id`<br> **必須** \| 新しく作成されたアップストリームに関連付ける証明書の一意の識別子または`name`属性。

#### リクエストボディ


{{ page.upstream_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.upstream_json }}
```

*** ** * ** ***

### アップストリームを一覧表示
{:.badge .dbless}

##### すべてのアップストリームの一覧表示

<div class="endpoint get indent">{{ prefix }}/upstreams</div> 

##### 特定の証明書に関連付けられたアップストリームの一覧表示

<div class="endpoint get indent">/certificates/\{certificate name or id\}/upstreams</div> 

{:.indent}
属性 \| 説明
\-\-\-：\|\-\-\-
`certificate name or id`<br> **必須** \| アップストリームを取得する証明書の一意の識別子または`name`属性。このエンドポイントを使用すると、指定された証明書に関連付けられたアップストリームのみが一覧表示されます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.upstream_data }}
    "next": "http://localhost:8001/upstreams?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### アップストリームの取得
{:.badge .dbless}

##### アップストリームの取得

<div class="endpoint get indent">{{ prefix }}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 取得するアップストリームの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたアップストリームを取得

<div class="endpoint get indent">/certificates/\{certificate id\}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 取得する証明書の一意の識別子。
`upstream name or id` <br> **必須** \| 取得するアップストリームの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json

{{ page.upstream_json }}
```

*** ** * ** ***

### アップストリームの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### アップストリームの更新

<div class="endpoint patch indent">/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 更新するアップストリームの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたアップストリームの更新

<div class="endpoint patch indent">/certificates/\{certificate id\}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 更新する証明書の一意の識別子。
`upstream name or id`<br> **必須** \| 更新するアップストリームの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.upstream_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.upstream_json }}
```

*** ** * ** ***

### アップストリームの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### アップストリームの作成または更新

<div class="endpoint put indent">{{ prefix }}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 作成または更新するアップストリームの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたアップストリームの作成または更新

<div class="endpoint put indent">/certificates/\{certificate id\}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 作成または更新する証明書の一意の識別子。
`upstream name or id`<br> **必須** \| 作成または更新するアップストリームの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.upstream_body }}

要求されたリソースの下のアップストリームを挿入（または置換）します
本文に定義が指定されています。 アップストリームは、 `name or id` 属性によって識別されます。

`name or id` 属性がUUIDの構造を持つ場合、挿入/置換されるアップストリームは、その`id`によって識別されます。それ以外の場合は、その`name`で識別されます。

`id`は、新しいアップストリームの作成時に（URLにもボディにも）指定されない場合は、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### アップストリームの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### アップストリームの削除

<div class="endpoint delete indent">{{ prefix }}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| 削除するアップストリームの一意の識別子 **または** 名前。

##### 特定の証明書に関連付けられたアップストリームの削除

<div class="endpoint delete indent">/certificates/\{certificate id\}/upstreams/\{upstream name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`certificate id`<br> **必須** \| 削除する証明書の一意の識別子。
`upstream name or id`<br> **必須** \| 削除するアップストリームの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### ノードのアップストリームヘルスを表示
{:.badge .dbless}

特定のアップストリームのすべてのターゲットのヘルスステータスまたは
特定のKongノードの観点から見たアップストリーム全体を表示します。ノード固有の情報であるため、同じリクエストを
Kongクラスターの異なるノードに行うと、異なる結果が生成される場合があります。
たとえば、Kongクラスタの特定のノードで、ネットワークに問題が生じた場合、
一部のターゲットに接続できないことがあります。
こうしたターゲットは、そのノードによって不健全としてマークされます（トラフィックをこのノードから
、正常に到達できる他のターゲットに接続します）が、
他のすべてのKongノード \(そのターゲットの使用に問題はない）にとっては健全です。

レスポンスの`data`フィールドには、Targetオブジェクトの配列が含まれます。
各ターゲットのヘルスは`health`フィールドに返されます。

* DNS の問題が原因でバランサーでターゲットをアクティブ化できない場合、そのステータスは `DNS_ERROR` と表示されます。
* アップストリーム構成で[ヘルスチェック](/gateway/{{page.release}}/how-kong-works/health-checks)が有効になっていない場合、アクティブなターゲットのヘルスステータスは`HEALTHCHECKS_OFF`と表示されます。
* ヘルスチェックが有効になっていて、ターゲットが正常であると判断された場合、 自動または[手動](#set-target-as-healthy)により、ステータスが`HEALTHY`と表示されます。これは、このターゲットが現在、このアップストリームのロードバランサーの実行に含まれていることを意味します。
* アクティブまたはパッシブのヘルスチェック（サーキットブレーカー）によって、または[手動](#set-target-as-unhealthy)によってターゲットが無効になっている場合、そのステータスは `UNHEALTHY` と表示されます。ロードバランサーは、このアップストリーム経由でこのターゲットにトラフィックを送信してはいません。

リクエストクエリのパラメータである`balancer_health`が`1`に
設定されている場合、レスポンスの`data`フィールドはアップストリーム自体を参照します。そしてその`health`
属性は、`healthchecks.threshold`フィールドに基づき、アップストリームの
すべてのターゲットの状態によって定義されます。
<div class="endpoint get indent">/upstreams/\{名前またはID\}/health</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`name or id`<br> **必須** \| ターゲットヘルスを表示するアップストリームの一意の識別子 **または** 名前。

#### リクエストのクエリ文字列パラメータ

|                            属性 |                                      説明                                       |
|------------------------------:|-------------------------------------------------------------------------------|
| `balancer_health`<br> *オプション* | 1に設定すると、Kongはアップストリーム自体のヘルスステータスを返します。`healthchecks.threshold`プロパティを参照してください。 |

#### レスポンス

    HTTP 200 OK

```json
{
    "total": 2,
    "node_id": "cbb297c0-14a9-46bc-ad91-1d0ef9b42df9",
    "data": [
        {
            "id": "18c0ad90-f942-4098-88db-bbee3e43b27f",
            "health": "HEALTHY",
            "data": {
                "dns": "ttl=0, virtual SRV",
                "addresses": [
                    {
                        "weight": 100,
                        "ip": "127.0.0.1",
                        "port": 20000,
                        "health": "HEALTHY"
                    }
                ],
                "weight": {
                    "unavailable": 0,
                    "available": 100,
                    "total": 100
                },
                "port": 20000,
                "host": "127.0.0.1",
                "nodeWeight": 100
            },
            "weight": 100,
            "target": "127.0.0.1:20000",
            "created_at": 1485524883980,
            "upstream": {
                "id": "07131005-ba30-4204-a29f-0927d53257b4"
            },
            "tags": null
        },
        {
            "id": "6c6f34eb-e6c3-4c1f-ac58-4060e5bca890",
            "health": "UNHEALTHY",
            "data": {
                "dns": "ttl=0, virtual SRV",
                "addresses": [
                    {
                        "weight": 100,
                        "ip": "127.0.0.1",
                        "port": 20002,
                        "health": "UNHEALTHY"
                    }
                ],
                "weight": {
                    "unavailable": 100,
                    "available": 0,
                    "total": 100
                },
                "port": 20002,
                "host": "127.0.0.1",
                "nodeWeight": 100
            },
            "weight": 100,
            "target": "127.0.0.1:20002",
            "created_at": 1485524914883,
            "upstream": {
                "id": "07131005-ba30-4204-a29f-0927d53257b4"
            },
            "tags": null
        }
    ]
}
```

`balancer_health=1` の場合:

    HTTP 200 OK

```json
{
    "data": {
        "health": "HEALTHY",
        "id": "07131005-ba30-4204-a29f-0927d53257b4"
    },
    "next": null,
    "node_id": "cbb297c0-14a9-46bc-ad91-1d0ef9b42df9"
}
```

*** ** * ** ***

ターゲットオブジェクト
-----------

ターゲットは、バックエンドサービスのインスタンスを識別するポート付きのIPアドレス/ホスト名です。
どのアップストリームも多数のターゲットを持つことができ、ターゲットは動的に追加、変更、または削除できます。
変更は即座に有効になります。

ターゲットを無効にするには、`weight=0`を使用して新しいターゲットを投稿します。
あるいは、便利な`DELETE`メソッドを使用して同じことを実現します。

現在のターゲットオブジェクトの定義は、最新の`created_at`を持つものです。

ターゲットは[タグ付けすることも、タグでフィルタリングすることも](#tags)できます。

```json

{{ page.target_json }}
```

### ターゲットの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

ターゲットを更新します。
<div class="endpoint patch indent">/upstreams/\{アップストリームの名前またはID\}/targets/\{ホスト:ポートまたはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| ターゲットを更新するアップストリームの一意の識別子 **または** 名前。`host:port or id`<br> **必須** \| 更新するターゲットのホスト/ポートの組み合わせ要素、または既存のターゲットエントリの`id`。

#### レスポンス

    HTTP 201 Created

*** ** * ** ***

### ターゲットの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

ロードバランサーからターゲットを削除します。
<div class="endpoint delete indent">{{ prefix }}/upstreams/\{upstream name or id\}/targets/\{host:port or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| ターゲットを削除するアップストリームの一意の識別子 **または** 名前。
`host:port or id`<br> **必須** \| 削除するターゲットのホスト：ポートの組み合わせ要素、または既存のターゲットエントリの`id`。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### ターゲットアドレスを正常と設定

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

ロードバランサーのターゲットによって解決された個々のアドレスの現在のヘルスステータスを、Kong クラスタ全体で「正常」と設定します。

このエンドポイントは、以前にアップストリームの[ヘルスチェッカー](/gateway/{{page.release}}/how-kong-works/health-checks)によって無効にされたターゲットによって解決されたアドレスを、手動で再度有効にするために使用できます。アップストリームは正常なノードにのみリクエストを転送するので、この呼び出しは、Kongにこのアドレスの使用を再開するよう指示します。

これにより、Kongノードのすべてのワーカーで実行されているヘルスチェッカーの
ヘルスカウンターがリセットされ、クラスタ全体のメッセージをブロードキャストするため、
「正常」なステータスがKongクラスタ全体に伝達されます。

注: Kong がハイブリッドモードで実行している場合、この API は使用できません。
<div class="endpoint put indent">/upstreams/\{upstream name or id\}/targets/\{target or id\}/\{address\}/healthy</div> 

{:.indent}
属性 \| 説明
\-\-\-：\|\-\-\-
`upstream name or id`<br> **必須** \| アップストリームの一意の識別子 **または** 名前。
`target or id`<br> **必須** \| 正常として設定するターゲットのホスト/ポートの組み合わせ要素、または既存のターゲットエントリの`id`。
`address`<br> **必須** \| 正常として設定するアドレスのホスト/ポートの組み合わせ要素。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### ターゲットアドレスを異常として設定

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

ロードバランサーのターゲットによって解決された個々のアドレスの現在のヘルスステータスを、Kongクラスタ全体で「異常」と設定します。

このエンドポイントを使用すると、アドレスを手動で無効にし、リクエストへの応答を停止させることができます。アップストリームは正常なノードにのみリクエストを転送するので、この呼び出しは、Kongにこのアドレスのスキップを開始するよう指示します。

この呼出は、Kongノードのすべてのワーカーで実行されているヘルスチェッカーのヘルスカウンターをリセットし、「正常でない」ステータスが全体のKongクラスタに伝播されるようクラスタ全体のメッセージをブロードキャストします。

[アクティブなヘルスチェックは](/gateway/{{page.release}}/how-kong-works/health-checks/#active-health-checks)、異常がない場合は引き続き実行されます
住所。 アクティブなヘルスチェックが有効になっていて、プローブが検出する場合は注意してください
アドレスが実際に正常であれば、自動的に再び有効になります。
ターゲットをバランサーから完全に削除するには、[ターゲットを削除する必要があります
代わりにターゲット](#delete-target)。

注: Kong がハイブリッドモードで実行している場合、この API は使用できません。
<div class="endpoint put indent">/upstreams/\{upstream name or id\}/targets/\{target or id\}/unhealthy</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| アップストリームの一意の識別子 **または** 名前。
`target or id`<br> **必須** \| 異常として設定するターゲットのホスト/ポートの組み合わせ要素、または既存のターゲットエントリの`id`。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### 目標を正常に設定
{:.badge .dbless}

Kongクラスタ全体で、ロードバランサーのターゲットの現在のヘルスステータスを「正常（healthy）」に設定します。これにより、このターゲットによって解決されるすべてのアドレスに「正常（healthy）」ステータスが設定されます。

このエンドポイントは、以前にアップストリームの[ヘルスチェッカー](/gateway/{{page.release}}/how-kong-works/health-checks)によって無効にされたターゲットを手動で再度有効にするために使用できます。
アップストリームは正常なノードにのみリクエストを転送するので、この呼び出しは、このターゲットの使用を再開するようKongに指示します。

これにより、Kongノードのすべてのワーカーで実行されているヘルスチェッカーの
ヘルスカウンターがリセットされ、クラスタ全体のメッセージをブロードキャストするため、
「正常」なステータスがKongクラスタ全体に伝達されます。

注: Kong がハイブリッドモードで実行している場合、この API は使用できません。
<div class="endpoint put indent">/upstreams/\{upstream name or id\}/targets/\{target or id\}/healthy</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| アップストリームの一意の識別子 **または** 名前。
`target or id`<br> **必須** \| 正常として設定するターゲットのホスト/ポートの組み合わせ要素、または既存のターゲットエントリの`id`。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### ターゲットを異常に設定
{:.badge .dbless}

Kongクラスタ全体で、ロードバランサー内のターゲットの現在のヘルスステータスを「異常（unhealthy）」に設定します。これにより、このターゲットによって解決されるすべてのアドレスに「異常（unhealthy）」ステータスが設定されます。

このエンドポイントを使用すると、ターゲットを手動で無効にし、リクエストへの
応答を停止させることができます。アップストリームは正常なノードにのみリクエストを転送するので、
この呼び出しは、Kongにこのターゲットをスキップするよう指示します。

この呼出は、Kongノードのすべてのワーカーで実行されているヘルスチェッカーのヘルスカウンターをリセットし、「正常でない」ステータスが全体のKongクラスタに伝播されるようクラスタ全体のメッセージをブロードキャストします。

[アクティブヘルスチェックは、](/gateway/{{page.release}}/how-kong-works/health-checks/#active-health-checks)不健康な状態に対しても実行され続けます。
ターゲット。 アクティブなヘルスチェックが有効になっていて、プローブが検出する場合は注意してください
ターゲットが実際に正常であれば、自動的に再び有効になります。
ターゲットをバランサーから完全に削除するには、[ターゲットを削除する必要があります
代わりにターゲット](#delete-target)。

注: Kong がハイブリッドモードで実行している場合、この API は使用できません。
<div class="endpoint put indent">/upstreams/\{upstream name or id\}/targets/\{target or id\}/unhealthy</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`upstream name or id`<br> **必須** \| アップストリームの一意の識別子 **または** 名前。
`target or id`<br> **必須** \| 異常として設定するターゲットのホスト/ポートの組み合わせ要素、または既存のターゲットエントリの`id`。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### すべてのターゲットの一覧表示
{:.badge .dbless}

アップストリームのすべてのターゲットを一覧表示します。同じターゲットに対する複数のターゲットオブジェクトが返され、特定のターゲットの変更履歴が表示される場合があります。最新の `created_at` を持つターゲットオブジェクトが現在の定義です。
<div class="endpoint get indent">/upstreams/\{名前またはID\}/targets/all</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`name or id`<br> **必須** \| ターゲットを一覧表示するアップストリームの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json
{
    "total": 2,
    "data": [
        {
            "created_at": 1485524883980,
            "id": "18c0ad90-f942-4098-88db-bbee3e43b27f",
            "target": "127.0.0.1:20000",
            "upstream_id": "07131005-ba30-4204-a29f-0927d53257b4",
            "weight": 100
        },
        {
            "created_at": 1485524914883,
            "id": "6c6f34eb-e6c3-4c1f-ac58-4060e5bca890",
            "target": "127.0.0.1:20002",
            "upstream_id": "07131005-ba30-4204-a29f-0927d53257b4",
            "weight": 200
        }
    ]
}
```

*** ** * ** ***

Vaultsオブジェクト
------------

Vaultオブジェクトは、さまざまなVaultコネクタを構成するために使用されます。Vaultの例としては、環境変数、HashiCorp Vault、AWS Secrets Managerなどがあります。

Vaultを構成すると、他のエンティティでシークレットを参照できるようになります。たとえば、エンティティ内に証明書とキーを保存する代わりに、Vaultに格納される証明書とキーへの参照を保存できます。
これにより、シークレットと構成が適切に分離され、シークレットの拡散を防止できます。

{% if_version gte:3.4.x %}シークレットのローテーションは[TTL](/gateway/latest/kong-enterprise/secrets-management/advanced-usage)を使用して管理できます。{% endif_version %}

Vault は、[タグ付けすることも、タグでフィルタリングすることも](#tags)できます。

```json

{{ page.vault_json }}
```

### Vault の追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### Vault の作成

<div class="endpoint post indent">{{ prefix }}/vaults</div> 

#### リクエストボディ


{{ page.vault_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.vault_json }}
```

*** ** * ** ***

### Vaultsを一覧表示
{:.badge .dbless}

##### すべての Vault の一覧化

<div class="endpoint get indent">{{ prefix }}/vaults</div> 

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.vault_data }}
    "next": "http://localhost:8001/vaults?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### Vaultを取得
{:.badge .dbless}

##### Vaultを取得

<div class="endpoint get indent">{{ prefix }}/vaults/\{vault prefix or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`vault prefix or id`<br> **必須** \| 取得するVaultの一意の識別子 **または** プレフィックス。

#### レスポンス

    HTTP 200 OK

```json

{{ page.vault_json }}
```

*** ** * ** ***

### Vaultの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### Vaultの更新

<div class="endpoint patch indent">/vaults/\{vault prefix or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`vault prefix or id`<br> **必須** \| 更新するVaultの一意の識別子 **または** プレフィックス。

#### リクエストボディ


{{ page.vault_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.vault_json }}
```

*** ** * ** ***

### Vaultの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### Vaultの作成または更新

<div class="endpoint put indent">{{ prefix }}/vaults/\{vault prefix or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`vault prefix or id`<br> **必須** \|作成または更新する Vault の一意の識別子 **または** プレフィックス。

#### リクエストボディ


{{ page.vault_body }}

リクエストされたリソースの下に、本文で指定された定義のVaultを
挿入（または置換）します。Vaultは、`prefix or id`属性によって識別されます。

`prefix or id`属性がUUIDの構造を持つ場合、挿入/置換されるVaultは、
その`id`によって識別されます。それ以外の場合は、その`prefix`で識別されます。

`id`をURLおよびボディのどちらにも指定せずに新しいVaultを作成する場合、Vaultは自動生成されます。

URLに `prefix` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### Vaultを削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### Vaultを削除

<div class="endpoint delete indent">{{ prefix }}/vaults/\{vault prefix or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`vault prefix or id`<br> **必須** \| 削除するVaultの一意の識別子 **または** プレフィックス。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

キーオブジェクト
--------

キーオブジェクトは、さまざまな形式での非対称キーの表現を保持します。
KongまたはKongプラグインが、特定の操作を行うために特定の公開鍵または秘密鍵を必要とする場合、このエンティティを使用できます。

キーにはタグを付けることも[、タグでフィルタリングすることもできます](#tags)。

```json

{{ page.key_json }}
```

### キーの追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーを作成

<div class="endpoint post indent">/keys</div> 

##### 特定のキーセットに関連付けられたキーの作成

<div class="endpoint post indent">/key\-sets/\{key\_set name or id\}/keys</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key_set name or id`<br> **必須** \| 新しく作成されたキーに関連付けるキーセットの一意の識別子または`name`属性。

#### リクエストボディ


{{ page.key_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.key_json }}
```

*** ** * ** ***

### キーの一覧化
{:.badge .dbless}

##### すべてのキーの一覧化

<div class="endpoint get indent">/keys</div> 

##### 特定のキーセットに関連付けられたキーの一覧表示

<div class="endpoint get indent">/key\-sets/\{key\_set name or id\}/keys</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key_set name or id`<br> **必須** \| キーを取得するキーセットの一意の識別子または`name`属性。このエンドポイントを使用すると、指定されたキーセットに関連付けられたキーのみが一覧表示されます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.key_data }}
    "next": "http://localhost:8001/keys?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### キーの取得
{:.badge .dbless}

##### キーの取得

<div class="endpoint get indent">/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 取得するキーの一意の識別子 **または** 名前。

##### 特定のキーセットに関連付けられたキーの取得

<div class="endpoint get indent">/key\-sets/\{key\_set name or id\}/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key_set name or id`<br> **必須** \| 取得するキーセットの一意の識別子 **または** 名前。
`key name or id`<br> **必須** \| 取得するキーの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json

{{ page.key_json }}
```

*** ** * ** ***

### キーの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーの更新

<div class="endpoint patch indent">/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 更新するキーの一意の識別子 **または** 名前。

##### 特定のキーセットに関連付けられたキーの更新

<div class="endpoint patch indent">/key\-sets/\{key\_set name or id\}/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`key_set name or id`<br> **必須** \| 更新するキーセットの一意の識別子 **または** 名前。`key name or id`<br> **必須** \| 更新するキーの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.key_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.key_json }}
```

*** ** * ** ***

### キーの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーの作成または更新

<div class="endpoint put indent">/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 作成または更新するキーの一意の識別子 **または** 名前。

##### 特定のキーセットに関連付けられたキーの作成または更新

<div class="endpoint put indent">/key\-sets/\{key\_set name or id\}/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`key_set name or id`<br> **必須** \| 作成または更新するキーセットの一意の識別子 **または** 名前。`key name or id`<br> **必須** \| 作成または更新するキーの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.key_body }}

ボディで指定された定義を使って、指定されたリソースの下にキーを挿入（または置換）します。キーは `name or id` 属性によって識別されます。

`name or id`属性がUUID構造をとる場合、挿入されるまたは置き換えられるキーは、その`id`によって識別されます。それ以外の場合は、その`name`で識別されます。

`id` を URL 内およびボディ内のどちらにも指定せずに新しいキーを作成する場合、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### キーを削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーを削除

<div class="endpoint delete indent">/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 削除するキーの一意の識別子 **または** 名前。

##### 特定のキーセットに関連付けられたキーの削除

<div class="endpoint delete indent">/key\-sets/\{key\_set name or id\}/keys/\{key name or id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key_set name or id`<br> **必須** \| 削除するキーセットの一意の識別子 **または** 名前。
`key name or id`<br> **必須** \| 削除するキーの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

キーセットエンティティ
-----------

キーセットオブジェクトは、さまざまな形式での非対称キーオブジェクトのコレクションを保持します。このエンティティを使用すると、目的別にキーを論理的にグループ化できます。

キーセットは、[タグ付けすることも、タグでフィルタリング](#tags)することもできます。

```json

{{ page.key_set_json }}
```

### キーセットの追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーセットの作成

<div class="endpoint post indent">/key\-sets</div> 

#### リクエストボディ


{{ page.key_set_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.key_set_json }}
```

*** ** * ** ***

### キーセットの一覧化
{:.badge .dbless}

##### すべてのキーセットの一覧表示

<div class="endpoint get indent">/key\-sets</div> 

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.key_set_data }}
    "next": "http://localhost:8001/key-sets?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```

*** ** * ** ***

### キーセットの取得
{:.badge .dbless}

##### キーセットの取得

<div class="endpoint get indent">/key\-sets/\{key\_setの名前またはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key_set name or id`<br> **必須** \| 取得するキーセットの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json

{{ page.key_set_json }}
```

*** ** * ** ***

### キーセットの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーセットの更新

<div class="endpoint patch indent">/key\-sets/\{key\_setの名前またはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`key_set name or id`<br> **必須** \| 更新するキーセットの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.key_set_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.key_set_json }}
```

*** ** * ** ***

### キーセットの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーセットの作成または更新

<div class="endpoint put indent">/key\-sets/\{key\_setの名前またはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`key_set name or id`<br> **必須** \| 作成または更新するキーセットの一意の識別子 **または** 名前。

#### リクエストボディ


{{ page.key_set_body }}

ボディで指定された定義に従って、指定されたリソースに対してキーセットを挿入（または置換）します。キーセットは `name or id` 属性によって識別されます。

`name or id` 属性が UUID の構造を持つ場合、挿入/置換されるキーセットは、その `id` によって識別されます。それ以外の場合は、その `name` で識別されます。

`id`は、新しいキーセットの作成時に（URLにもボディにも）指定されない場合は、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### キーセットの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### キーセットの削除

<div class="endpoint delete indent">/key\-sets/\{key\_setの名前またはID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key_set name or id`<br> **必須** \| 削除するキーセットの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### 特定のキーセットに関連付けられたキーを一覧表示
{:.badge .dbless}

指定したキーセット内のすべてのキーを一覧表示します。

##### 特定のキーに関連付けられたキーセットの取得

<div class="endpoint get indent">/keys/\{キーの名前またはID\}/set</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 取得するキーセットに関連付けられたキーの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 200 OK

```json
{
  "data": [
    {
      "id": "46CA83EE-671C-11ED-BFAB-2FE47512C77A",
      "name": "my-key_set",
      "tags": ["google-keys", "mozilla-keys"],
      "created_at": 1422386534,
      "updated_at": 1422386534
  }, {
      "id": "57532ECE-6720-11ED-9297-279D4320B841",
      "name": "my-key_set",
      "tags": ["production", "staging", "development"],
      "created_at": 1422386534,
      "updated_at": 1422386534
  }]
}
```

*** ** * ** ***

### キーセット内のキーの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

キーセット内のキーを更新します

##### 特定のキーに関連付けられたキーセットの更新

<div class="endpoint patch indent">/keys/\{キーの名前またはID\}/set</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 更新するキーセットに関連付けられたキーの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 201 Created

*** ** * ** ***

### キーセット内にキーを作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

キーを作成します

##### 特定のキーに関連付けられたキーセットの作成または更新

<div class="endpoint put indent">/keys/\{キーの名前またはID\}/set</div> 

{:.indent}
属性 \| 説明
\-\-\-：\|\-\-\-
`key name or id`<br> **必須** \| 作成または更新するキーセットに関連付けられたキーの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 201 Created

*** ** * ** ***

### キーセット内のキーの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

このキーセットに関連付けられているキーの削除

##### 特定のキーに関連付けられたキーセットの削除

<div class="endpoint delete indent">/keys/\{キーの名前またはID\}/set</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`key name or id`<br> **必須** \| 削除するキーセットに関連付けられたキーの一意の識別子 **または** 名前。

#### レスポンス

    HTTP 204 No Content

{% if_version gte:3.4.x %}

フィルターチェーン オブジェクト
----------------

[フィルターチェーン](/gateway/{{page.release}}/reference/wasm/#filter_chain)は、特定の[サービス](#services)または[ルート](#routes)への各リクエストに対して実行される1つ以上のWebAssembly[フィルター](/gateway/{{page.release}}/reference/wasm/#filter)を表すデータベースエンティティであり、それぞれに固有の構成があります。

フィルターチェーンは[タグ付けすることも、タグでフィルタリングすることも](#tags)できます。

```json

{{ page.filter_chain_json }}
```

詳細については、リファレンスドキュメントの[Wasmセクション](/gateway/{{page.release}}/reference/wasm/)を参照してください。

### フィルターチェーンを追加

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### フィルターチェーンの作成

<div class="endpoint post indent">{{ prefix }}/filter\-chains</div> 

##### 特定のルートに関連付けられたフィルターチェーンの作成

<div class="endpoint post indent">{{ prefix }}/routes/\{ルート名またはID\}/filter\-chains</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 新しく作成されたフィルターチェーンに関連付けるルートの一意の識別子または`name`属性。

##### 特定のサービスに関連付けられたフィルターチェーンの作成

<div class="endpoint post indent">{{ prefix }}/services/\{service name or id\}/filter\-chains</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 新しく作成されたフィルターチェーンに関連付けるサービスの一意の識別子または`name`属性。

#### リクエストボディ


{{ page.filter_chain_body }}

#### レスポンス

    HTTP 201 Created

```json

{{ page.filter_chain_json }}
```

*** ** * ** ***

### フィルターチェーンの一覧表示
{:.badge .dbless}

##### すべてのフィルターチェーンの一覧化

<div class="endpoint get indent">{{ prefix }}/filter\-chains</div> 

##### 特定のルートに関連付けられたフィルターチェーンの一覧化

<div class="endpoint get indent">{{ prefix }}/routes/\{ルート名またはID\}/filter\-chains</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| フィルターチェーンを取得するルートの一意の識別子または `name` 属性。このエンドポイントを使用すると、指定されたルートに関連付けられたフィルターチェーンのみが一覧化されます。

##### 特定のサービスに関連付けられたフィルターチェーンの一覧表示

<div class="endpoint get indent">{{ prefix }}/services/\{service name or id\}/filter\-chains</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| フィルターチェーンを取得するサービスの一意の識別子または `name` 属性。このエンドポイントを使用すると、指定されたサービスに関連付けられたフィルターチェーンのみが一覧化されます。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.filter_chain_data }}
}
```

*** ** * ** ***

### フィルターチェーンの取得
{:.badge .dbless}

##### フィルターチェーンの取得

<div class="endpoint get indent">{{ prefix }}/filter\-chains/\{フィルターチェーンのID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`filter chain id`<br> **必須** \| 取得するフィルターチェーンの一意の識別子。

##### 特定のルートに関連付けられたフィルターチェーンを取得

<div class="endpoint get indent">{{ prefix }}/routes/\{route name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 取得するルートの一意の識別子 **または** 名前。`filter chain id`<br> **必須** \| 取得するフィルターチェーンの一意の識別子。

##### 特定のサービスに関連付けられたフィルターチェーンの取得

<div class="endpoint get indent">{{ prefix }}/services/\{service name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 取得するサービスの一意の識別子 **または** 名前。`filter chain id`<br> **必須** \| 取得するフィルターチェーンの一意の識別子。

#### レスポンス

    HTTP 200 OK

```json

{{ page.filter_chain_json }}
```

*** ** * ** ***

### フィルターチェーンの更新

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### フィルターチェーンの更新

<div class="endpoint patch indent">/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`filter chain id`<br> **必須** \| 更新するフィルターチェーンの一意の識別子。

##### 特定のルートに関連付けられたフィルターチェーンを更新

<div class="endpoint patch indent">/routes/\{route name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 更新するルートの一意の識別子 **または** 名前。`filter chain id`<br> **必須** \| 更新するフィルターチェーンの一意の識別子。

##### 特定のサービスに関連付けられたフィルターチェーンを更新

<div class="endpoint patch indent">/services/\{service name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-：\|\-\-\-
`service name or id`<br> **必須** \| 更新するサービスの一意の識別子 **または** 名前。
`filter chain id`<br> **必須** \| 更新するフィルターチェーンの一意の識別子。

#### リクエストボディ


{{ page.filter_chain_body }}

#### レスポンス

    HTTP 200 OK

```json

{{ page.filter_chain_json }}
```

*** ** * ** ***

### フィルターチェーンの更新または作成

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### フィルターチェーンを作成または更新

<div class="endpoint put indent">{{ prefix }}/filter\-chains/\{フィルターチェーンのID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`filter chain id`<br> **必須** \| 作成または更新するフィルターチェーンの一意の識別子。

##### 特定のルートに関連付けられたフィルターチェーンを作成または更新

<div class="endpoint put indent">{{ prefix }}/routes/\{route name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 作成または更新するルートの一意の識別子 **または** 名前。
`filter chain id`<br> **必須** \| 作成または更新するフィルターチェーンの一意の識別子。

##### 特定のサービスに関連付けられたフィルターチェーンの作成または更新

<div class="endpoint put indent">{{ prefix }}/services/\{service name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 作成または更新するサービスの一意の識別子 **または** 名前。
`filter chain id`<br> **必須** \| 作成または更新するフィルターチェーンの一意の識別子。

#### リクエストボディ


{{ page.filter_chain_body }}

ボディで指定された定義を使って、指定されたリソースの下にフィルターチェーンを挿入（または置換）します。フィルターチェーンは、 `name or id` 属性によって識別されます。

`name or id`属性がUUIDの構造を持つ場合、挿入/置換されるフィルターチェーンは、その`id`によって識別されます。それ以外の場合は、その`name`で識別されます。

`id`は、新しいフィルターチェーンの作成時に（URLにもボディにも）指定されない場合は、自動生成されます。

URLに `name` を指定して、リクエストボディに異なるものを指定することはできないことに注意してください

#### レスポンス

    HTTP 200 OK

POST応答とPATCH応答を参照してください。

*** ** * ** ***

### フィルターチェーンの削除

{:.note}
> 
> **注** : このAPIはDBレスモードでは利用できません。

##### フィルターチェーンの削除

<div class="endpoint delete indent">{{ prefix }}/filter\-chains/\{フィルターチェーンのID\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`filter chain id`<br> **必須** \| 削除するフィルターチェーンの一意の識別子。

##### 特定のルートに関連付けられたフィルターチェーンの削除

<div class="endpoint delete indent">{{ prefix }}/routes/\{route name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\|\-\-\-
`route name or id`<br> **必須** \| 削除するルートの一意の識別子 **または** 名前。`filter chain id`<br> **必須** \| 削除するフィルターチェーンの一意の識別子。

##### 特定のサービスに関連付けられたフィルターチェーンの削除

<div class="endpoint delete indent">{{ prefix }}/services/\{service name or id\}/filter\-chains/\{filter chain id\}</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`service name or id`<br> **必須** \| 削除するサービスの一意の識別子 **または** 名前。
`filter chain id`<br> **必須** \| 削除するフィルターチェーンの一意の識別子。

#### レスポンス

    HTTP 204 No Content

*** ** * ** ***

### ルートに適用されたリストフィルター
{:.badge .dbless}

サービスフィルターチェーンとルートフィルターチェーンから順に適用されるフィルターを組み合わせ、特定のルートの完全なフィルター実行計画を調べることができます。チェーンおよび個々のフィルターはどちらも `enabled` boolean属性で個別に有効または無効にできるため、有効なフィルター（これはアクティブな実行プランです）、無効なフィルター、すべてのフィルターを一覧表示するための個別のエンドポイントがあります。

詳細については、[フィルターの実行動作](/gateway/{{page.release}}/reference/wasm/#filter_execution_behavior)に関するリファレンスドキュメントを参照してください。

##### 特定のルートに適用された有効なフィルターの一覧表示

<div class="endpoint get indent">{{ prefix }}/routes/\{route name or id\}/filters/enabled</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 関連付けられたフィルターを一覧表示するルートの一意の識別子または `name` 属性。

##### 特定のルートに適用された無効なフィルターを一覧表示する

<div class="endpoint get indent">{{ prefix }}/routes/\{route name or id\}/filters/disabled</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 関連付けられたフィルターを一覧表示するルートの一意の識別子または `name` 属性。

##### 特定のルートに適用されるすべてのフィルターの一覧表示

<div class="endpoint get indent">{{ prefix }}/routes/\{route name or id\}/filters/all</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`route name or id`<br> **必須** \| 関連付けられたフィルターを一覧表示するルートの一意の識別子または `name` 属性。

#### レスポンス

    HTTP 200 OK

```json
{

{{ page.filters_data }}
}
```

{% endif_version %}

*** ** * ** ***

