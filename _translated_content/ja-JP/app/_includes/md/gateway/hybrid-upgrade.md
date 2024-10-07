ハイブリッドモードは、1 つ以上のコントロールプレーン \(CP\) ノードと 1 つ以上のデータプレーン \(DP\) ノードで構成されます。
CP ノードはデータベースを使用して Kong 構成データを保存しますが、DP ノードは必要な情報をすべて CP から取得するため、データベースを使用しません。
推奨されるアップグレードプロセスは、ノード \(CP または DP\) の種類ごとに異なるアップグレードストラテジを組み合わせたものです。

ハイブリッドモードアップグレードにおける主な課題は、CP と DP 間の通信です。
ハイブリッドモードでは CP のマイナー バージョンが DP のマイナー バージョン以上である必要があるため、DP ノードの前に CP ノードをアップグレードする必要があります。
アップグレードは2段階に分けて実施する必要があります。

1. まず、DPノードがAPIリクエストを処理している間に、[従来モード](#traditional-mode)セクションの推奨事項に従ってCPをアップグレードします。
2. 次に、[DBレスモード](#db-less-mode)セクションの推奨事項を使用して、DPノードをアップグレードします。 バージョンの競合を回避するために、新しいDPノードを新しいCPにポイントします。

CP と DP 間のロールのデカップリング機能によって、CP をアップグレードしながら DP ノードが API リクエストを処理できるようになります。
この方法により、ビジネスのダウンタイムが発生しません。

カスタムプラグイン（独自のプラグインまたは {{site.base_gateway}} に付属していないサードパーティ製プラグイン）は、ハイブリッドモードでコントロールプレーン（CP）とデータプレーン（DP）の両方にインストールする必要があります。プラグインは、最初にコントロールプレーン（CP）にインストールし、次にデータプレーン（DP）にインストールします。

ハイブリッドモードデプロイメントのオプション詳細については、次のセクションを参照してください。

#### コントロールプレーン（CP）

CPノードはDPノードよりも先にアップグレードする必要があります。CPノードは管理者専用ロールを提供し、データベースのサポートが必要です。そのため、図2と図3で説明するように、従来のモード（デュアルクラスタまたはインプレース）に指定されたものと同じアップグレードストラテジから選択できます。

[デュアルクラスターストラテジ](/gateway/{{include.release}}/upgrade/dual-cluster/)を使用してCPノードをアップグレードする。

{% mermaid %}
flowchart TD
    DBA[(Current
    database)]
    DBB[(New 
    database)]
    CPX(Current control plane X)
    Admin(No admin 
    write operations)
    CPY(New control plane Y)
    DPX(fa:fa-layer-group Current data plane X nodes)
    API(API requests)

    DBA -.- CPX -."DP connects to either \nCP X...".- DPX
    Admin -.X.- CPX & CPY
    DBB --pg_restore--- CPY -."...OR to CP Y".- DPX
    API--> DPX

    style API stroke:none
    style DBA stroke-dasharray:3
    style CPX stroke-dasharray:3
    style Admin fill:none,stroke:none,color:#d44324
    linkStyle 2,3 stroke:#d44324,color:#d44324
{% endmermaid %}
> 
> *図2: この図は、デュアルクラスタストラテジを使用したCPのアップグレードの流れを示しています。*
> *新しいCP Yは現在のCP Xと一緒にデプロイされ、現在のDPノードXはまだAPIリクエストに対応しています。* 

[インプレース ストラテジ](/gateway/{{include.release}}/upgrade/in-place/)を使用して CP ノードをアップグレードします。

{% mermaid %}
flowchart 
    DBA[(Database)]
    CPX(Current control plane X \n #40;inactive#41;)
    Admin(No admin \n write operations)
    CPY(New control plane Y)
    DPX(fa:fa-layer-group Current data plane X nodes)
    API(API requests)

    DBA -..- CPX -."DP connects to either \nCP X...".- DPX
    Admin -.X.- CPX & CPY
    DBA --"kong migrations up \n kong migrations finish"--- CPY -."...OR to CP Y".- DPX
    API--> DPX

    style API stroke:none
    style CPX stroke-dasharray:3
    style Admin fill:none,stroke:none,color:#d44324
    linkStyle 2,3 stroke:#d44324,color:#d44324
{% endmermaid %}
> 
> *図3：この図は、現在のCP Xが新しいCP Yに直接置き換えられるインプレースストラテジを使用したCPアップグレードの流れを示しています。*
> *データベースは新しいCP Yで再利用され、すべてのノードの移行が完了すると現在のCP Xは停止されます。* 

2つの図から、DPノードXが現在のCPノードXに接続されたままになっているか、または新しいCPノードYに切り替わっていることがわかります。
{{site.base_gateway}}は、CPの新しいマイナーバージョンがDPの古いマイナーバージョンと互換性があることを保証します。
これにより、DPノードXが一時的に新しいCPノードYを指せるようになります。
これにより、必要に応じてアップグレードプロセスを一時停止したり、より長い期間にわたってアップグレードを実行したりできます。

{:.important}
> 
> このセットアップは一時的なものであり、アップグレードプロセス中にのみ使用されます。
> 長期間のプロダクションデプロイメントでは、新しいバージョンのCPノードと古いバージョンのDPノードを組み合わせて実行することはお勧めしません。

CPのアップグレード後、クラスタXを廃止することができます。この作業は、DPアップグレードの最後まで遅らせることができます。

#### データプレーン（DP）

CPノードがアップグレードされたら、DPノードのアップグレードに進むことができます。
DPアップグレードでサポートされている唯一のアップグレードストラテジは、ローリングアップグレードです。
次の図4と5は、それぞれ図2と3に相当しています。

[ローリングアップグレード](/gateway/{{include.release}}/upgrade/rolling-upgrade/)のワークフローを使った[デュアルクラスタストラテジ](/gateway/{{include.release}}/upgrade/dual-cluster/)の使用：

{% mermaid %}
flowchart TD
    DBX[(Current \n database)]
    DBY[(New \n database)]
    CPX(Current control plane X)
    CPY(New control plane Y)
    DPX(Current data planes X)
    DPY(New data planes Y)
    API(API requests)
    LB(Load balancer)
    Admin(No admin \n write operations)
    Admin2(No admin \n write operations)
    
    subgraph A
        Admin -.X.- CPX
        DBX -.- CPX
        DBY --- CPY
        CPX -."Current DP connects to \neither CP X...".- DPX
        Admin2 -.X.- CPY
        CPY -."...OR to CP Y".- DPX
        DPX -.90%..- LB
        CPY --- DPY --10%---- LB
        
    end
    subgraph B
        API --> LB & LB & LB
    end

    linkStyle 0,4 stroke:#d44324,color:#d44324
    linkStyle 8,9 stroke:#b6d7a8
    style CPX stroke-dasharray:3,fill:#eff0f1ff,stroke:#c1c6cdff
    style DPX stroke-dasharray:3
    style DBX stroke-dasharray:3,fill:#eff0f1ff,stroke:#c1c6cdff
    style API stroke:none
    style A stroke:none,color:#fff
    style B stroke:none,color:#fff
    style Admin fill:none,stroke:none,color:#d44324
    style Admin2 fill:none,stroke:none,color:#d44324
{% endmermaid %}
> 
> *図4：デュアルクラスタとローリングストラテジを使用したDPアップグレードの図です。*
> *新しいCP Yは現在のCP Xと一緒にデプロイされ、現在のDPノードXはまだAPIリクエストに対応しています。*
> *画像では、現在のデータベースとCP Xの背景色が白ではなく灰色になっていますが、これは古いCPがすでにアップグレードされ、廃止された可能性があることを示しています。* 

[ローリングアップグレード](/gateway/{{include.release}}/upgrade/rolling-upgrade/)のワークフローを使った[インプレースストラテジ](/gateway/{{include.release}}/upgrade/in-place/)
の使用:

{% mermaid %}
flowchart 
    DBA[(Database)]
    CPX(Current control plane X \n #40;inactive#41;)
    CPY(New control plane Y)
    DPX(Current data planes X)
    DPY(New data planes Y)
    API(API requests)
    LB(Load balancer)
    Admin(No admin \n write operations)
    Admin2(No admin \n write operations)

    subgraph A
        Admin -.X.- CPX
        DBA -.X.- CPX
        DBA --- CPY
        CPX -."Current DP connects to \neither CP X...".- DPX
        Admin2 -.X.- CPY
        CPY -."OR to CP Y".- DPX -.90%..- LB
        CPY --- DPY --10%---- LB 
    end
    subgraph B
        API --> LB & LB & LB
    end

    linkStyle 0,1,4 stroke:#d44324,color:#d44324
    linkStyle 8,9 stroke:#b6d7a8
    style CPX stroke-dasharray:3,fill:#eff0f1ff,stroke:#c1c6cdff
    style DPX stroke-dasharray:3
    style A stroke:none,color:#fff
    style B stroke:none,color:#fff
    style Admin fill:none,stroke:none,color:#d44324
    style Admin2 fill:none,stroke:none,color:#d44324
{% endmermaid %}
> 
> *図5：インプレースとローリングストラテジを使用したDPアップグレードの図です。*
> *この図は、データベースが新しいCP Yによって再使用される一方で、現在のDPノードXがまだAPIリクエストを処理しているところを示しています。*

