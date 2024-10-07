---
title: "参照形式"
---
シークレットストアへの参照を記述するには、[URL 構文](https://en.wikipedia.org/wiki/URL)を使用します。

```text
{vault://<vault-backend|entity>/<secret-id id="sl-md0000000">[/<secret-key][/][?query][#version]}
```

### プロトコル/スキーム

```text
{vault://<vault-backend|entity>/<secret-id id="sl-md0000000">[/<secret-key]}
 ^^^^^
```

URL内の`vault`はKongの識別子として使用されます。これを使用してVaultを参照します。

### ホスト/パス

```text
{vault://<vault-prefix id="sl-md0000000">/<secret-id id="sl-md0000000">[/<secret-key]}
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

URLの`host`と`path`は以下のように定義されています。

#### Vaultプレフィックス

Vaultのプレフィックスは、バックエンドの名前でも、作成したVaultエンティティの名前でもかまいません。

例

```text
{vault://env/<secret-id id="sl-md0000000">[/<secret-key]}
         ^^^
```

またはVaultエンティティを使用

```text
{vault://my-env-vault/<secret-id id="sl-md0000000">[/<secret-key]}
         ^^^^^^^^^^^^
```

#### シークレットID

`secret-id`は、Vaultに保存されているシークレットの識別子として使用されます。Vaultは
`string`値（単一のシークレット）、またはシークレット`object`としてのユーザーネームや
パスワードなどの、複数の関連するシークレットのいずれかを返します。

#### 秘密鍵

`secret-key`は、 `secret-id`オブジェクト内のシークレットを識別するために使用されます。

{:.note}
> 
> 秘密鍵が`/`で終わる場合、それは[秘密鍵](#secret-key)ではなく[シークレットID](#secret-id)の一部とみなされます。
> [秘密鍵](#secret-key)と[シークレットID](#secret-id)の違いは、[シークレットID](#secret-id)のみがVault APIに送信されることで、
> [秘密鍵](#secret-key)は処理時にのみ使用されます

### クエリ

クエリ引数は、[Vault接頭辞](/gateway/{{page.release}}/kong-enterprise/secrets-management/reference-format/#vault-prefix)の`key=value`形式で構成オプションを示すために使用されます。

### バージョン

```text
{vault://<vault-backend|entity>/<secret-id id="sl-md0000000">[/<secret-key][/][?query][#version]}
                                                                     ^^^^^^^^
```

Vault URLのフラグメントとして指定されているバージョンは、Vaultバックエンドに保存されているシークレットのバージョン番号を識別します。バージョン管理をサポートするすべてのVaultバックエンドに適用されます。

