---
##
#  WARNING: this file was auto-generated by a script.
#  DO NOT edit this file directly. Instead, send a pull request to change
#  https://github.com/Kong/kong/tree/master/kong/pdk
#  or its associated files
##

title: "kong.vault"
pdk: true
toc: true
source_url: "https://github.com/Kong/kong/tree/master/kong/pdk/vault.lua"
---
このモジュールは、ボールト参照を解決、解析、検証するために使用できます。

kong.vault.is\_reference\(reference\)
----------------------------------------

渡された参照が参照のように見えるかどうかをチェックします。
有効な参照は、'\{[vault://'](vault://')で始まり、'\}'で終わります。

より詳細な検証が必要な場合は、`kong.vault.parse_reference`を使用します

**パラメータ** 

* **reference** （`string`）：チェックする参照

**戻り値** 

* `boolean`：渡された参照が参照に見える場合は`true`、そうでない場合は`false`

**使用法** 

```lua
kong.vault.is_reference("{vault://env/key}") -- true
kong.vault.is_reference("not a reference")   -- false
```

kong.vault.parse\_reference\(reference\)
-------------------------------------------

渡された参照を解析およびデコードし、そのコンポーネントを含むテーブルを返します。

次のリソースがあるとします。

```lua
"{vault://env/cert/key?prefix=SSL_#1}"
```

この関数は、次のテーブルを返します。

```lua
{
  name     = "env",  -- name of the Vault entity or Vault strategy
  resource = "cert", -- resource where secret is stored
  key      = "key",  -- key to lookup if the resource is secret object
  config   = {       -- if there are any config options specified
    prefix = "SSL_"
  },
  version  = 1       -- if the version is specified
}
```

**パラメータ** 

* **reference** （`string`）：解析する参照

**戻り値** 

1. `table|nil`：参照の各コンポーネントを含むテーブル、またはエラーの場合は`nil`

2. `string|nil`：失敗した場合のエラーメッセージ、それ以外の場合は `nil`

**使用法** 

```lua
local ref, err = kong.vault.parse_reference("{vault://env/cert/key?prefix=SSL_#1}") -- table
```

kong.vault.get\(reference\)
-----------------------------

渡された参照を解決し、その値を返します。

**パラメータ** 

* **参照** （`string`）：解決するための参照

**戻り値** 

1. `string|nil`: リファレンスの解決された値

2. `string|nil`：失敗した場合のエラーメッセージ、それ以外の場合は `nil`

**使用法** 

```lua
local value, err = kong.vault.get("{vault://env/cert/key}")
```

kong.vault.update\(options\)
------------------------------

TTLに基づくシークレットローテーションのヘルパー関数。現時点では実験的な機能です。

**パラメータ** 

* **オプション** （`table`）：シークレットと参照を含むオプション \(この関数は入力オプションを修正します\)

**戻り値** 

* `table`：シークレット値が更新されたオプション

**使用法** 

```lua
local options = kong.vault.update({
  cert = "-----BEGIN CERTIFICATE-----...",
  key = "-----BEGIN RSA PRIVATE KEY-----...",
  cert_alt = "-----BEGIN CERTIFICATE-----...",
  key_alt = "-----BEGIN EC PRIVATE KEY-----...",
  ["$refs"] = {
    cert = "{vault://aws/cert}",
    key = "{vault://aws/key}",
    cert_alt = "{vault://aws/cert-alt}",
    key_alt = "{vault://aws/key-alt}",
  }
})

-- or

local options = {
  cert = "-----BEGIN CERTIFICATE-----...",
  key = "-----BEGIN RSA PRIVATE KEY-----...",
  cert_alt = "-----BEGIN CERTIFICATE-----...",
  key_alt = "-----BEGIN EC PRIVATE KEY-----...",
  ["$refs"] = {
    cert = "{vault://aws/cert}",
    key = "{vault://aws/key}",
    cert_alt = "{vault://aws/cert-alt}",
    key_alt = "{vault://aws/key-alt}",
  }
}
kong.vault.update(options)
```

kong.vault.try\(callback, options\)
-------------------------------------

シークレットを自動でローテーションするヘルパー関数。現時点では実験的な機能です。

**パラメータ** 

* **callback** （`function`）: コールバック関数
* **オプション** \( `table` \): 認証情報とリファレンスを含むオプション

**戻り値** 

1. `string|nil`: コールバック関数の戻り値

2. `string|nil`：失敗した場合のエラーメッセージ、それ以外の場合は `nil`

**使用法** 

```lua
local function connect(options)
  return database_connect(options)
end

local connection, err = kong.vault.try(connect, {
  username = "john",
  password = "doe",
  ["$refs"] = {
    username = "{vault://aws/database-username}",
    password = "{vault://aws/database-password}",
  }
})
```

{% if_version eq:3.3.x %}

kong.vault.flush\(\)
----------------------

Valutの設定と参照のLRUキャッシュをフラッシュします。

**使用法** 

```lua
kong.vault.flush()
```

{% endif_version %}

