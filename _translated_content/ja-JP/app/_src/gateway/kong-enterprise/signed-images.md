---
title: "署名付き Kong イメージの署名の検証"
badge: "enterprise"
---
{{site.ee_product_name}} 3\.5\.0\.2 以降、Docker コンテナイメージは `cosign` を使用して署名され、署名は Docker Hub リポジトリに公開されるようになりました。

このガイドでは、署名された{{site.ee_product_name}} Dockerコンテナイメージの署名を2つの異なる方法で検証する手順を説明します。

* アノテーションを利用せずに画像を検証するための最小限の例。
* 信頼性を向上させるため、オプションのアノテーションを活用した完全な例

最小限の例では、Dockerの詳細、GitHubリポジトリ名、GitHubワークフローファイル名のみが必要です。

完全な例では、最小限の例と同じ詳細に加えて、検証するオプションの注釈も必要です。

|                 ショートハンド                 |         説明         |         値の例         |
|-----------------------------------------|--------------------|---------------------|
| `<repo id="sl-md0000000">`              | Githubリポジトリ        | `kong-ee`           |
| `<workflow filename id="sl-md0000000">` | Githubワークフローのファイル名 | `release.yml`       |
| `<workflow name id="sl-md0000000">`     | Githubワークフロー名      | `Package & Release` |

KongはGithub Actionsを使用してビルドとリリースを行うため、GithubのOIDCアイデンティティを使用して画像に署名します。そのため、これらの詳細の多くがGithubに関連しています。

例
---

### 前提条件

どちらの例でも、次のことを行う必要があります。

1. `cosign`がインストールされていることを確認します。

2. 必要な画像の詳細を収集します。

3. `COSIGN_REPOSITORY`環境変数を設定します。

   ```sh
   export COSIGN_REPOSITORY=kong/notary
   ```

{:.important .no-icon}
> 
> Githubの所有者は大文字と小文字が区別されます（`Kong/kong-ee`と`kong/kong-ee` ）。

### 最小限の例

`cosign verify ...` コマンドを実行します。

```sh
cosign verify \
   <image id="sl-md0000000">:<tag id="sl-md0000000">@sha256:<digest id="sl-md0000000"> \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='https://github.com/Kong/<repo id="sl-md0000000">/.github/workflows/<workflow filename id="sl-md0000000">'
```

プレースホルダーの代わりにサンプル値を使用した同じ例を次に示します。

```sh
cosign verify \
   'kong/kong-gateway:3.6.0.0-ubuntu@sha256:2f4d417efee8b4c26649d8171dd0d26e0ca16213ba37b7a6b807c98a4fd413e8' \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='https://github.com/Kong/kong-ee/.github/workflows/release.yml'
```

`cosign`検証が完了すると、コマンドは`0`で終了します。

```sh
...
echo $?
0
```

### 完全な例

```sh
cosign verify \
   <image id="sl-md0000000">:<tag id="sl-md0000000">@sha256:<digest id="sl-md0000000"> \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='https://github.com/Kong/<repo id="sl-md0000000">/.github/workflows/<workflow filename id="sl-md0000000">' \
   -a repo='Kong/<repo id="sl-md0000000">' \
   -a workflow='<workflow name id="sl-md0000000">'
```

プレースホルダーの代わりにサンプル値を使用した同じ例を次に示します。

```sh
cosign verify \
   'kong/kong-gateway:3.6.0.0-ubuntu@sha256:2f4d417efee8b4c26649d8171dd0d26e0ca16213ba37b7a6b807c98a4fd413e8' \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='https://github.com/Kong/kong-ee/.github/workflows/release.yml' \
   -a repo='Kong/kong-ee' \
   -a workflow='Package & Release'
```

