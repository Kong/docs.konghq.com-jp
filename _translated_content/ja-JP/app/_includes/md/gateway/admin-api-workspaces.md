ワークスペースを指定しないリクエストは、`default`ワークスペースを対象とします。

別のワークスペースをターゲットにするには、エンドポイントの前にワークスペース名または ID を付けます。

```sh
http://localhost:8001/<WORKSPACE_NAME|ID>/<endpoint id="sl-md0000000">
```

たとえば、ワークスペースを指定しない場合は、このリクエストは、`default` ワークスペースからサービスのリストを取得します。

```sh
curl -i -X GET http://localhost:8001/services
```

このリクエストはワークスペース`SRE`からすべてのサービスを取得します。

```sh
curl -i -X GET http://localhost:8001/SRE/services
```

