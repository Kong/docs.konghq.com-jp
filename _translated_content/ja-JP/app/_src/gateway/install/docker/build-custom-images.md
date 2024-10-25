---
title: "独自の Docker イメージをビルドする"
content_type: "how-to"
---
Kong は、 [DockerHub](https://hub.docker.com/r/kong)でホストされている公式 Docker イメージに加えて、ビルド済みの{% if_version lte:3.3.x %} `apk` 、 {% endif_version %} `deb` 、 `rpm`パッケージとして配布されています。

Kong は、本番環境で使用するために [Debian](https://hub.docker.com/r/kong/kong-gateway/tags?page=&page_size=&ordering=&name={{ page.release.tag }}-debian) および [RHEL](https://hub.docker.com/r/kong/kong-gateway/tags?page=&page_size=&ordering=&name={{ page.release.tag }}-rhel) イメージを構築および検証します。{% if_version lte:3.3.x %}[Alpine](https://hub.docker.com/r/kong/kong-gateway/tags?page=&page_size=&ordering=&name={{ page.release.tag }}-alpine)イメージには、プラグイン開発用の `git` などの開発ツールが含まれているため、 **開発目的のみ** で提供されます。{%- endif_version %}

Debian および RHEL イメージは、最小限の依存関係（{{ site.base_gateway }} 3\.0 時点）で構築され、公開される前に自動セキュリティスキャナーでスキャンします。サポートされているイメージで検出された脆弱性は、次に利用可能なパッチリリースで対処されます。

独自のイメージをビルドして、ベースイメージや依存関係をさらにカスタマイズしたい場合は、以下の手順に従ってください。

1. `docker-kong`から[docker\-entrypoint.sh](https://raw.githubusercontent.com/Kong/docker-kong/master/docker-entrypoint.sh)スクリプトをダウンロードし、実行可能にします。

```bash
chmod +x docker-entrypoint.sh
```

1. {{site.base_gateway}} パッケージをダウンロードします。

   * **Debian** : [.deb]({{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/debian/pool/bullseye/main/k/ko/kong-enterprise-edition_{{page.versions.ee}}/kong-enterprise-edition_{{page.versions.ee}}_amd64.deb).
   {% if_version lte:3.0.x -%}
   * **Ubuntu** :[.deb]({{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/ubuntu/pool/focal/main/k/ko/kong-enterprise-edition_{{page.versions.ee}}/kong-enterprise-edition_{{page.versions.ee}}_amd64.deb).
   {% endif_version -%}
   {% if_version gte:3.1.x -%}
   * **Ubuntu** : [.deb]({{ site.links.direct }}/gateway-{{ page.major_minor_version }}/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_{{page.versions.ee}}/kong-enterprise-edition_{{page.versions.ee}}_amd64.deb). {% endif_version -%} {%- comment -%} 古いアルパイン「パッケージ」のすべてが、アルパインパッケージのあり方に関するCloudsmithの定義を満たしているわけではありません。 代わりに「raw」アーティファクトとしてアップロードされるものもあり、別の方法でリンクする必要があります。 このページはlte:2\.8\.xには存在しません。 {%- endcomment -%} {% if_version lte:3.3.x -%}
   {% if_version eq:3.0.x -%}
   * **Alpine** : [.apk.tar.gz]({{ site.links.direct }}/gateway-{{ page.major_minor_version }}/raw/names/kong-enterprise-edition-x86_64/versions/{{page.versions.ee}}/kong-enterprise-edition-{{page.versions.ee}}.x86_64.apk.tar.gz)
   {% endif_version -%}
   {% if_version gte:3.1.x -%}
   * **Alpine** : [.apk]({{ site.links.direct }}/gateway-{{ page.major_minor_version }}/alpine/any-version/main/x86_64/kong-enterprise-edition-{{page.versions.ee}}.apk) {% endif_version -%}
   {% endif_version -%}
   * **RHEL** :[.rpm]({{ site.links.direct }}/gateway-{{ page.major_minor_version }}/rpm/el/8/x86_64/kong-enterprise-edition-{{page.versions.ee}}.el8.x86_64.rpm)

2. `Dockerfile`を作成し、ファイル名の最初の`COPY`を、手順2でダウンロードした{{site.base_gateway}}ファイルの名前に置き換えます。

{% capture dockerfile_run_steps %}COPY docker\-entrypoint.sh /docker\-entrypoint.sh

ユーザー Kong

ENTRYPOINT \["/docker\-entrypoint.sh"\]

EXPOSE 8000 8443 8001 8444 8002 8445 8003 8446 8004 8447

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health

CMD \["kong", "docker\-start"\]{% endcapture %}

{% capture dockerfile %}
{% navtabs codeblock indent %}

{% navtab Debian %}

```dockerfile

FROM debian:bullseye-slim

COPY kong.deb /tmp/kong.deb

RUN set -ex; \
    apt-get update \
    && apt-get install --yes /tmp/kong.deb \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/kong.deb \
    && chown kong:0 /usr/local/bin/kong \
    && chown -R kong:0 /usr/local/kong \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && kong version


{{ dockerfile_run_steps }}
```

{% endnavtab %}

{% navtab Ubuntu %}

```dockerfile

{% if_version lte:3.0.x %}
FROM ubuntu:20.04
{% endif_version %}
{% if_version gte:3.1.x %}
FROM ubuntu:22.04
{% endif_version %}

COPY kong.deb /tmp/kong.deb

RUN set -ex; \
    apt-get update \
    && apt-get install --yes /tmp/kong.deb \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/kong.deb \
    && chown kong:0 /usr/local/bin/kong \
    && chown -R kong:0 /usr/local/kong \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && kong version


{{ dockerfile_run_steps }}
```

{% endnavtab %}

{% navtab RHEL %}

```dockerfile

FROM registry.access.redhat.com/ubi8/ubi:8.1

COPY kong.rpm /tmp/kong.rpm

RUN set -ex; \
    yum install -y /tmp/kong.rpm \
    && rm /tmp/kong.rpm \
    && chown kong:0 /usr/local/bin/kong \
    && chown -R kong:0 /usr/local/kong \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && kong version


{{ dockerfile_run_steps }}
```

{% endnavtab %}

{% if_version lte:3.3.x %}
{% navtab Alpine %}

```dockerfile

FROM alpine:latest

COPY kong.apk.tar.gz /tmp/kong.apk.tar.gz

RUN set -ex; \
    apk add --no-cache --virtual .build-deps tar gzip \
    && tar -C / -xzf /tmp/kong.apk.tar.gz \
    && apk add --no-cache libstdc++ libgcc openssl pcre perl tzdata libcap zlib zlib-dev bash curl ca-certificates \
    && adduser -S kong \
    && addgroup -S kong \
    && mkdir -p "/usr/local/kong" \
    && chown -R kong:0 /usr/local/kong \
    && chown kong:0 /usr/local/bin/kong \
    && chmod -R g=u /usr/local/kong \
    && rm -rf /tmp/kong.tar.gz \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
    && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
    && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
    && apk del .build-deps \
    && kong version


{{ dockerfile_run_steps }}
```

{% endnavtab %}
{% endif_version %}

{% endnavtabs %}
{% endcapture %}

{{ dockerfile | indent }}

1. イメージをビルドします。

   ```bash
   docker build --platform linux/amd64 --no-cache -t kong-image .
   ```

2. イメージが正しくビルドされたことをテストします。

       docker run -it --rm kong-image kong version

3. {{ site.base_gateway }}を実行してトラフィックを処理するには、 [Docker インストール手順](/gateway/latest/install/docker/)に従い、イメージ名をカスタム名に置き換えます。

