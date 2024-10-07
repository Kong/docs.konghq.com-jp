---
title: "署名付き Kong イメージのビルド来歴を確認"
badge: "enterprise"
---
KongはDockerコンテナイメージのビルドの来歴を生成します。これは、Docker Hubリポジトリに公開された来歴証明と`cosign` / `slsa-verifier`を使用して検証できます。

このガイドでは、署名された {{site.ee_product_name}} Docker コンテナイメージのビルドの出所を 2 つの異なる方法で検証する手順を説明します。

* アノテーションを利用せずに画像を検証するための最小限の例。
* 信頼性を向上させるため、オプションのアノテーションを活用した完全な例

最小限の例では、DockerマニフェストダイジェストとGitHubリポジトリ名のみが必要です。

{:.important .no-icon}
> 
> ビルドの起源の検証には Docker マニフェストダイジェストが必要です。マニフェストダイジェストは、特定のディストリビューションのプラットフォーム固有のイメージダイジェストとは異なる場合があります。

完全な例では、最小限の例と同じ詳細に加えて、検証するオプションの注釈も必要です。

|                ショートハンド                 |         説明         |         値の例         |
|----------------------------------------|--------------------|---------------------|
| `<repo id="sl-md0000000">`             | GitHubリポジトリ        | `kong-ee`           |
| `<workflow name id="sl-md0000000">`    | GitHubワークフロー名      | `Package & Release` |
| `<workflow trigger id="sl-md0000000">` | GitHub ワークフロー トリガー | `workflow_dispatch` |

Kong はビルドとリリースに GitHub Actions を使用するため、コンテナ イメージのビルドの由来を生成するために GitHub の OIDC ID も使用します。そのため、これらの詳細の多くは GitHub に関連しています。

例
---

### 前提条件

どちらの例でも、次のことを行う必要があります。

1. `cosign` / `slsa-verifier` がインストールされていることを確認します。

2. `regctl`がインストールされていることを確認します。

3. 必要な画像の詳細を収集します。

4. `regctl`を使用して画像の`<manifest_digest id="sl-md0000000">`を解析します。

   ```sh
   regctl manifest digest <image id="sl-md0000000">:<tag id="sl-md0000000">
   ```

{% if_version gte:3.7.x %}

5. `COSIGN_REPOSITORY`環境変数を設定します。

   ```sh
   export COSIGN_REPOSITORY=kong/notary
   ```

{% endif_version %}

{:.important .no-icon}
> 
> GitHubの所有者は大文字と小文字が区別されます（`Kong/kong-ee`と`kong/kong-ee` ）。

### 最小限の例

#### Cosignの使用

`cosign verify-attestation ...`コマンドを実行します。

```sh
cosign verify-attestation \
   <image id="sl-md0000000">:<tag id="sl-md0000000">@sha256:<manifest_digest id="sl-md0000000"> \
   --type='slsaprovenance' \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='^https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v[0-9]+.[0-9]+.[0-9]+$'
```

同じ例で、プレースホルダーの代わりにサンプル値を使用したバージョンを以下に示します。

```sh
cosign verify-attestation \
   'kong/kong-gateway:3.6.0.0-ubuntu@sha256:2f4d417efee8b4c26649d8171dd0d26e0ca16213ba37b7a6b807c98a4fd413e8' \
   --type='slsaprovenance' \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='^https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v[0-9]+.[0-9]+.[0-9]+$'
```

`cosign`検証が完了すると、コマンドは`0`で終了します。

```sh
...
echo $?
0
```

#### slsa\-verifier を使用する

`slsa-verifier verify-image ...`コマンドを実行します。

```sh
slsa-verifier verify-image \
   <image id="sl-md0000000">:<tag id="sl-md0000000">@sha256:<manifest_digest id="sl-md0000000"> \
   --print-provenance \
   --source-uri 'github.com/Kong/<repo id="sl-md0000000">'
```

同じ例で、プレースホルダーの代わりにサンプル値を使用したバージョンを以下に示します。

```sh
slsa-verifier verify-image \
   'kong/kong-gateway:3.6.0.0-ubuntu@sha256:2f4d417efee8b4c26649d8171dd0d26e0ca16213ba37b7a6b807c98a4fd413e8' \
   --print-provenance \
   --source-uri 'github.com/Kong/kong-ee'
```

成功した場合、コマンドは「検証済みの SLSA の起源」を出力します。

```sh
...
PASSED: Verified SLSA provenance
```

### 完全な例

#### Cosignの使用

`cosign verify-attestation ...`コマンドを実行します。

```sh
cosign verify-attestation \
   <image id="sl-md0000000">:<tag id="sl-md0000000">@sha256:<manifest_digest id="sl-md0000000"> \
   --type='slsaprovenance' \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='^https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v[0-9]+.[0-9]+.[0-9]+$' \
   --certificate-github-workflow-repository='Kong/<repo id="sl-md0000000">' \
   --certificate-github-workflow-name='<workflow name id="sl-md0000000">'
```

同じ例で、プレースホルダーの代わりにサンプル値を使用したバージョンを以下に示します。

```sh
cosign verify-attestation \
   'kong/kong-gateway:3.6.0.0-ubuntu@sha256:2f4d417efee8b4c26649d8171dd0d26e0ca16213ba37b7a6b807c98a4fd413e8' \
   --type='slsaprovenance' \
   --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
   --certificate-identity-regexp='^https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v[0-9]+.[0-9]+.[0-9]+$' \
   --certificate-github-workflow-repository='Kong/kong-ee' \
   --certificate-github-workflow-name='Package & Release'
```

#### slsa\-verifier を使用する

`slsa-verifier verify-image ...`コマンドを実行します。

```sh
slsa-verifier verify-image \
   <image id="sl-md0000000">:<tag id="sl-md0000000">@sha256:<manifest_digest id="sl-md0000000"> \
   --print-provenance \
   --source-uri 'github.com/Kong/<repo id="sl-md0000000">' \
   --build-workflow-input official="true"
```

同じ例で、プレースホルダーの代わりにサンプル値を使用したバージョンを以下に示します。

```sh
slsa-verifier verify-image \
   'kong/kong-gateway:3.6.0.0-ubuntu@sha256:2f4d417efee8b4c26649d8171dd0d26e0ca16213ba37b7a6b807c98a4fd413e8' \
   --print-provenance \
   --source-uri 'github.com/Kong/kong-ee' \
   --build-workflow-input official="true"
```

