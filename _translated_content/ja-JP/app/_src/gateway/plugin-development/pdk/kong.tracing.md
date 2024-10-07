---
##
#  WARNING: this file was auto-generated by a script.
#  DO NOT edit this file directly. Instead, send a pull request to change
#  https://github.com/Kong/kong/tree/master/kong/pdk
#  or its associated files
##

title: "kong.tracing"
pdk: true
toc: true
source_url: "https://github.com/Kong/kong/tree/master/kong/pdk/tracing.lua"
---
Kongのトレーサーモジュールアプリケーションレベルのトレーシング。

span:finish\(end\_time\_ns\)
--------------------------------

スパンを終了します。終了時間を設定し、スパンを解放します。スパンが終了した後は、スパンテーブルを使用してはなりません。

**パラメータ** 

* **end\_time\_ns** \(`number|nil`\):

**使用法** 

```lua
span:finish()

local time = ngx.now()
span:finish(time * 100000000)
```

span:set\_attribute\(key, value\)
------------------------------------

属性をSpanに設定する

**パラメータ**
{% if_version lte:3.6.x %}

* **キー** \( `string` \):
* **value** \(`string|number|boolean`\):

**使用法** 

```lua
span:set_attribute("net.transport", "ip_tcp")
span:set_attribute("net.peer.port", 443)
span:set_attribute("exception.escaped", true)
```

{% endif_version %}

{% if_version gte:3.7.x %}

* **キー** \( `string` \):
* **value** \(`string|number|boolean|nil`\):

**使用法** 

```lua
span:set_attribute("net.transport", "ip_tcp")
span:set_attribute("net.peer.port", 443)
span:set_attribute("exception.escaped", true)
span:set_attribute("unset.this", nil)
```

{% endif_version %}

span:add\_event\(name, attributes, time\_ns\)
-------------------------------------------------

スパンにイベントを追加します

**パラメータ** 

* **name** （`string`）: イベント名
* **attributes** \(`table|nil`\): Event attributes
* **time\_ns** \(`number|nil`\): イベントタイムスタンプ

span:record\_error\(err\)
----------------------------

Spanにエラーイベントを追加します

**パラメータ** 

* **err** （`string`）： エラー文字列

span:set\_status\(status\)
-----------------------------

スパンにエラーイベントを追加します。
ステータスコード：

* `0` 解除
* `1` ok
* `2` エラー

**パラメータ** 

* **status** \(`number`\): status code

{% if_version lte:3.2.x %}

kong.tracing.new\_span\(\)
-----------------------------

{% endif_version %}
{% if_version gte:3.3.x %}

kong.tracing.active\_span\(\)
--------------------------------

{% endif_version %}

アクティブスパンを取得します
デフォルトでルートスパンを返します

**フェーズ** 

* rewrite、access、header\_filter、response、body\_filter、log、admin\_api

**戻り値** 

* `table`：スパン

{% if_version lte:3.2.x %}

kong.tracing.new\_span\(span\)
---------------------------------

{% endif_version %}
{% if_version gte:3.3.x %}

kong.tracing.set\_active\_span\(span\)
------------------------------------------

{% endif_version %}
アクティブスパンの設定

**フェーズ** 

* rewrite、access、header\_filter、response、body\_filter、log、admin\_api

**パラメータ** 

* **span** \(`table`\)：

{% if_version lte:3.2.x %}

kong.tracing.new\_span\(name,options\)
-----------------------------------------

{% endif_version %}
{% if_version gte:3.3.x %}

kong.tracing.start\_span\(name, options\)
--------------------------------------------

{% endif_version %}
新しいスパンを作成する

**フェーズ** 

* rewrite、access、header\_filter、response、body\_filter、log、admin\_api

**パラメータ** 

* **name** \(`string`\): スパン名
* **options** \(`table`\): TODO\(mayo\)

**戻り値** 

* `table`：スパン

kong.tracing.process\_span\(processor\)
------------------------------------------

バッチ処理スパン。
ソケットはログフェーズでは使用できないので、代わりに `ngx.timer.at` を使用してください

**フェーズ** 

* ログ

**パラメータ** 

* **processor** \(`function`\): スパンをパラメータとして使用できる関数

{% if_version gte:3.3.x %}

kong.tracing:set\_should\_sample\(should\_sample\)
-------------------------------------------------------

すべてのスパンのshould\_sampleの値を更新します。

**パラメータ** 

* **should\_sample** \( `bool` \): サンプルパラメータの値 {% endif_version %}

{% if_version gte: 3.6.x %}

kong.tracing:get\_sampling\_decision\(parent\_should\_sample, sampling\_rate\)
-------------------------------------------------------------------------------------

サンプリングの決定結果の取得

親スパンの sampled フラグが false の場合、または、trace\_id が利用できない場合に、親スパンから記録しない決定を継承するために親ベースのサンプラーを使用します。

それ以外の場合は、確率ベースの should\_sample 決定を適用します。

**パラメータ** 

* **parent\_should\_sample** （`bool`）：受信トレーシングヘッダーから抽出される親スパンのサンプルフラグの値 
* **sampling\_rate** \(`number`\): 確率サンプラーに適用するサンプリングレート

**戻り値** 

* `bool`: このトレースでサンプリングされたもののサンプリング値

{% endif_version %}
