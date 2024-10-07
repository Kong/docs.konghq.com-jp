---
title: "コンテナでプラグインを使用する"
content-type: "reference"
---
コンテナとKubernetesでの外部プラグインの使用
---------------------------

外部プラグインサーバーを必要とするプラグインを使用するには、プラグインサーバーとプラグイン自体の両方を {{ site.base_gateway }} コンテナにインストールし、プラグインのソースコードを {{ site.base_gateway }} コンテナにコピーまたはマウントする必要があります。

{:.note}
> 
> **注：** 公式の{{ site.base_gateway }}の画像は、`nobody`ユーザーを実行者として構成されています。カスタム画像を構築する際に、{{ site.base_gateway }}画像にファイルをコピーするには、一時的にユーザーを`root`に設定する必要があります。

これは、{{ site.base_gateway }} イメージにプラグインをマウントする方法を示す Dockerfile の例です。

```dockerfile
FROM kong
USER root
# Example for GO:
COPY your-go-plugin /usr/local/bin/your-go-plugin
# Example for JavaScript:
RUN apk update && apk add nodejs npm && npm install -g kong-pdk
COPY your-js-plugin /path/to/your/js-plugins/your-js-plugin
# Example for Python
# PYTHONWARNINGS=ignore is needed to build gevent on Python 3.9
RUN apk update && \
    apk add python3 py3-pip python3-dev musl-dev libffi-dev gcc g++ file make && \
    PYTHONWARNINGS=ignore pip3 install kong-pdk
COPY your-py-plugin /path/to/your/py-plugins/your-py-plugin
# reset back the defaults
USER kong
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 8000 8443 8001 8444
STOPSIGNAL SIGQUIT
HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health
CMD ["kong", "docker-start"]
```

詳細情報
----

* [Pythonでプラグインを開発](/gateway/latest/plugin-development/pluginserver/python/)
* [Goを使用したプラグインの開発](/gateway/latest/plugin-development/pluginserver/go/)
* [JavaScriptを使用したプラグインの開発](/gateway/latest/plugin-development/pluginserver/javascript/)

