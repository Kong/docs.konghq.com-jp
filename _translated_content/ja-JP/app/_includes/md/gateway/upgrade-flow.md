{% mermaid %}
flowchart TD
    A{Deployment type?} --> B(Traditional mode)
    A{Deployment type?} --> C(Hybrid mode)
    A{Deployment type?} --> D(DB-less mode)
    A{Deployment type?} --> E(Konnect DP)
    B ---> F{Enough hardware to 
    run another cluster?}
    C --> G(Upgrade CP first) & H(Upgrade DP second)
    D ----> K([Rolling upgrade])
    E ----> K
    G --> F
    F ---Yes--->I([Dual-cluster upgrade])
    F ---No--->J([In-place upgrade])
    H ---> K
    click K "/gateway/latest/upgrade/rolling-upgrade/"
    click I "/gateway/latest/upgrade/dual-cluster/"
    click J "/gateway/latest/upgrade/in-place/"
{% endmermaid %}
> 
> *図1：デプロイメントタイプに基づいてアップグレードストラテジを選択します。従来のモードでは、十分なリソースがある場合はデュアルクラスタアップグレードを、十分なリソースがない場合はインプレースアップグレードを選択してください。DBレスモードおよび{{site.konnect_short_name}}DPの場合は、ローリングアップグレードを使用します。ハイブリッドモードの場合、CPには従来のモードストラテジの1つを使用し、DPにはローリングアップグレードを使用します。*

