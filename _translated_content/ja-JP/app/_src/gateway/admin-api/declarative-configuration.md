---
title: "Declarative Configuration (宣言型構成)"
no_version: true
---
Declarative Configuration \(宣言型構成\)
-------------------------------------

エンティティの宣言型構成を{{site.base_gateway}}にロードするには、
2つの方法があります。`declarative_config`
プロパティを介して起動時に実行する方法と、`/config`
エンドポイントを使用したAdmin APIを介して実行時に行う方法です。

宣言型構成を使い始めるには、エンティティ定義を含むファイル（YAMLまたはJSON形式）が必要です。次のコマンドを使用して、サンプルの宣言型構成を生成できます。

    kong config init

現在のディレクトリに、適切な構造と例が含まれた`kong.yml`という名前のファイルが生成されます。

### 宣言型構成の再読み込み

このエンドポイントでは、DB レス Kong を新しい宣言型構成データファイルでリセットできます。以前の内容はすべてメモリから消去され、指定されたファイル内に記述したエンティティがそれに取って代わります。

ファイル形式の詳細については、[宣言型構成](/gateway/{{page.release}}/production/deployment-topologies/db-less-and-declarative-config)ドキュメントを参照してください。
<div class="endpoint post indent">/config</div> 

{:.indent}
属性 \| 説明
\-\-\-:\| \-\-\-
`config`<br> **必須** \| ロードする構成データ（YAMLまたはJSON形式）。

#### リクエストクエリ文字列パラメータ

|                          属性 |                                Description                                |
|----------------------------:|---------------------------------------------------------------------------|
| `check_hash`<br> *optional* | 1に設定すると、Kongは入力構成データのハッシュを前のハッシュと比較します。構成が同一の場合は、再読み込みされず、HTTP 304が返されます。 |

#### Response

    HTTP 200 OK

```json
{
    {
        "services": [],
        "routes": []
    }
}
```

レスポンスには、入力ファイルから解析されたすべてのエンティティの
リストが含まれます。

*** ** * ** ***
