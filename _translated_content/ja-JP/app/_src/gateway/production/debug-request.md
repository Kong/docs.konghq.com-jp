---
title: "デバッグリクエスト"
content_type: "explanation"
---

{{site.base_gateway}}管理者は、特定のリクエストに関するタイミング情報をオンデマンドで収集することにより、リクエストをデバッグできます。リクエストのデバッグは安全なトークンを使用してトリガーされ、結果のデータは `X-Kong-Request-Debug-Output` という名前の応答ヘッダーで返されます。

リクエストのデバッグでは、次の分析情報が提供されます。

* プラグイン、DNS 解決、ロードバランシングなどのさまざまな {{site.base_gateway}} コンポーネントで使用した時間。
* これらのプロセス中に試行されたドメイン名などの文脈情報。

{:.note}
> 
> **注：** この機能はライブデバッグ用です。タイミングを含むヘッダーのJSONスキーマは決して静的と見なすべきではなく、常に変更される可能性があります。

リクエストのデバッグを有効にする
----------------

リクエストのデバッグはデフォルトで有効になっており、 [`kong.conf`](/gateway/{{page.release}}/reference/configuration/)では次の構成になっています。

    request_debug = on | off # enable or disable request debugging
    request_debug_token <token id="sl-md0000000"> # Set debug token explicitly. Otherwise, it will be generated randomly when Kong starts, restarts, and reloads. 

デバッグトークン（`request-debug-token`）を使用すると、権限のある人だけがデバッグリクエストを発行できるため、機能の乱用を防ぐことができます。

デバッグトークンは次の場所にあります。

* **{{site.base_gateway}} エラーログ:** {{site.base_gateway}} の起動、再起動、またはリロード時に、デバッグトークンがエラーログ（通知レベル）に記録されます。検索を容易にするために、ログ行には `[request-debug]` プレフィックスが付けられます。
* **ファイルシステム：** デバッグトークンは`{prefix}/.request_debug_token`にあるファイルにも保存され、{{site.base_gateway}}の起動時、再起動時、または再読み込み時に更新されます。

デバッグリクエストの構成
------------

リクエストをデバッグするには、次のリクエストヘッダーを追加します。

* 少なくとも、 `X-Kong-Request-Debug`ヘッダーを設定する必要があります。
* リクエストがループバックアドレス以外から送信された場合は、`X-Kong-Request-Debug-Token` ヘッダーも設定する必要があります。

### X\-Kong\-Request\-Debug ヘッダー

`X-Kong-Request-Debug` ヘッダーが `*` に設定されている場合、現在のリクエストのタイミング情報が収集され、エクスポートされます。

{{site.ee_product_name}}で、収集される時間情報の範囲をフィルタリングするために、フィルターをカンマで区切ったリストで指定することもできます。各フィルターは時間情報を収集するフェーズを指定します。サポートされるフィルターは以下のとおりです。

* `rewrite`
* `access`
* `balancer`
* `response`
* `header_filter`
* `body_filter`
* `upstream`
* `log`

このヘッダーが存在しないか不明な値を含んでいる場合、タイミング情報は現在のリクエストに対して収集されません。

### X\-Kong\-Request\-Debug\-Log header

`X-Kong-Request-Debug-Log` ヘッダーが true に設定されている場合、タイミング情報はログレベル `notice` で {{site.base_gateway}} エラーログにも記録されます。デフォルトでは、`X-Kong-Request-Debug-Log` ヘッダーは `false` に設定されています。ログラインには、検索を容易にするためのプレフィックス `[request-debug]` が含まれます。

### X\-Kong\-Request\-Debug\-Token ヘッダー

`X-Kong-Request-Debug-Token`は、クライアントを認証し、不正使用を防ぐためにデバッグ要求を行うためのトークンです。ループバックアドレスが生成するデバッグリクエストは、このヘッダーを必要としません。

{% if_version gte:3.5.x %}

### X\-Kong\-Request\-Id header

`X-Kong-Request-Id` ヘッダーには、各クライアントリクエストの一意の識別子が含まれます。これは、アップストリームとダウンストリームの両方でデフォルトで有効になっています。この一意の ID は、特定のリクエストを対応するエラーログと照合するのに役立ち、デバッグに役立ちます。{{site.base_gateway}} がエラーを返す場合、リクエスト ID も {{site.base_gateway}} によって生成されるレスポンスボディに含まれます。また、生成されるすべての {{site.base_gateway}} エラーログには、`request_id: xxx` という形式で同じリクエスト ID が含まれます。

デバッグヘッダーとデバッグレスポンスヘッダーによって生成されるログ行には、リクエスト ID が含まれています。これを使用すると、ログビューアー UI でデバッグヘッダー出力を検索できます。これは、デバッグ出力が長すぎてレスポンスヘッダーに収まらない場合に特に便利です。[大規模なデバッグ出力の一部切り取り](#truncation-for-large-debugging-output)に示すデバッグリクエストの例では、リクエスト ID を含むログ行の例を確認できます。{% endif_version %}

デバッグリクエストの例
-----------

以下はデバッグリクエストの例です。

```bash
curl http://localhost:8000/example \
    -H "X-Kong-Request-Debug: *" \
    -H "X-Kong-Request-Debug-Token: xxxxxx"
```

レスポンスヘッダーの出力例を次に示します。

```json
{
    "child": {
        "rewrite": {
            "total_time": 0
        },
        "access": {
            "child": {
                "dns": {
                    "child": {
                        "example.com": {
                            "child": {
                                "resolve": {
                                    "total_time": 0,
                                    "cache_hit": true
                                }
                            },
                            "total_time": 0
                        }
                    },
                    "total_time": 0
                },
                "plugins": {
                    "child": {
                        "rate-limiting": {
                            "child": {
                                "176928d4-0949-47c8-8114-19cac8f86aab": {
                                    "child": {
                                        "redis": {
                                            "total_time": 1,
                                            "child": {
                                                "connections": {
                                                    "child": {
                                                        "tcp://localhost:6379": {
                                                            "child": {
                                                                "connect": {
                                                                    "child": {
                                                                        "dns": {
                                                                            "child": {
                                                                                "localhost": {
                                                                                    "child": {
                                                                                        "resolve": {
                                                                                            "total_time": 0,
                                                                                            "cache_hit": true
                                                                                        }
                                                                                    },
                                                                                    "total_time": 0
                                                                                }
                                                                            },
                                                                            "total_time": 0
                                                                        }
                                                                    },
                                                                    "total_time": 0
                                                                }
                                                            },
                                                            "total_time": 0
                                                        }
                                                    },
                                                    "total_time": 0
                                                }
                                            }
                                        }
                                    },
                                    "total_time": 2
                                }
                            },
                            "total_time": 2
                        },
                        "request-transformer": {
                            "child": {
                                "cfd2d953-ad82-453c-9979-b7573f52c226": {
                                    "total_time": 0
                                }
                            },
                            "total_time": 0
                        }
                    },
                    "total_time": 2
                }
            },
            "total_time": 3
        },
        "log": {
            "child": {
                "plugins": {
                    "child": {
                        "http-log": {
                            "child": {
                                "22906259-2963-4c6d-96a1-6d36d21714e3": {
                                    "total_time": 4
                                }
                            },
                            "total_time": 4
                        }
                    },
                    "total_time": 4
                }
            },
            "total_time": 4
        },
        "header_filter": {
            "child": {
                "plugins": {
                    "child": {
                        "response-transformer": {
                            "child": {
                                "dee98076-a58f-490d-8f7b-8523506bf96d": {
                                    "total_time": 1
                                }
                            },
                            "total_time": 1
                        }
                    },
                    "total_time": 1
                }
            },
            "total_time": 1
        },
        "body_filter": {
            "child": {
                "plugins": {
                    "child": {
                        "response-transformer": {
                            "child": {
                                "dee98076-a58f-490d-8f7b-8523506bf96d": {
                                    "total_time": 0
                                }
                            },
                            "total_time": 1
                        }
                    },
                    "total_time": 1
                }
            },
            "total_time": 1
        },
        "balancer": {
            "total_time": 0
        },
        "upstream": {
            "total_time": 152,
            "child": {
                "time_to_first_byte": {
                    "total_time": 151
                },
                "streaming": {
                    "total_time": 1
                }
            }
        }
    },
    "request_id": "0208903e83001d216bee5435dbc5ed25"
}
```

デバッグ出力の例を分析すると、次のことがわかります。

* `total_time`の単位は`millisecond`です。
* `example.host` の DNS 解決がキャッシュされているため、この例では非常に高速です。
* このリクエストではアップストリームに 100 ミリ秒かかりました。
  * {{site.base_gateway}}がアップストリームにリクエストを送信してから{{site.base_gateway}}が最初のバイトを受信するまでの経過時間は20ミリ秒です。
  * アップストリームからの最初のバイトから最後のバイトまでの経過時間は 80 ミリ秒です。

デバッグ出力をフィルタリングすることもできます。

```bash
curl http://localhost:8000/example \
    -H "X-Kong-Request-Debug: upstream" \
    -H "X-Kong-Request-Debug-Token: xxxxxx"
```

次のような結果が返されます。

```json
{
  "request_id": "a1a1530f8ddb6f6f2462916ae002b715",
  "child": {
    "upstream": {
      "total_time": 363,
      "child": {
        "time_to_first_byte": {
          "total_time": 363
        }
      }
    }
  }
}
```

### 大きなデバッグ出力の切り捨て

ダウンストリームのシステムでは、応答ヘッダーにサイズ制限が課せられている場合があり、{{site.base_gateway}}は`X-Kong-Request-Output`が2KBを超えるとをこれを切り捨てます。この切り捨てられた出力は、無条件に`error_log`に記録されます。

```bash
curl http://localhost:8000/large_debugging_output \
    -H "X-Kong-Request-Debug: *" \
    -H "X-Kong-Request-Debug-Token: xxxxxx"
```

次のような結果が返されます。

```json
{
  "request_id": "60ca0a4f8e5e936c43692f49b27d2932",
  "truncated": true,
  "message": "Output is truncated, please check the error_log for full output by filtering with the request_id."
}
```

デバッグリクエストの出力が 3KB を超える場合、`request_id` を識別子として複数の部分に分割されます。

{:.note}
> 
> **注:** デバッグ出力には一貫したパターンがなく、将来変更される可能性があります。自動化されたツールで処理するようには設計されていません。むしろ、人間が読みやすいように意図されていました。
