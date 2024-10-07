{:.important}
> 
> **重要:** {{site.base_gateway}}を起動すると、NGINXマスタープロセスは`root`として実行され、ワーカープロセスは、デフォルトでは`kong`として実行されます。これが望ましい動作でない場合は、{{site.base_gateway}}を起動する前に、NGINXマスタープロセスを組み込みの`kong`ユーザーまたはカスタムの非ルートユーザーで実行するように切り替えることができます。{% if_version lte:2.8.x %}
> 詳細については、[非ルートユーザーとしてKongを実行](/gateway/{{include.release}}/plan-and-deploy/kong-user/)を参照してください。{% endif_version %}
> {% if_version gte:3.0.x %}
> 詳細については、[非ルートユーザーとしてKongを実行](/gateway/{{include.release}}/production/running-kong/kong-user/)を参照してください。{% endif_version %}

